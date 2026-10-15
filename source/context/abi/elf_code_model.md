# 代码模型(Code Model)

LoongArch作为一个精简指令集，它的内存范围地址受限于指令中的编码位宽。

为了在不同情况下实现内存访问，我们定义了多种代码模型，这些模型包含一系列指令，并具有必要的寻址能力和性能开销。

一般来说，更宽的寻址范围需要更多的指令，并带来更高的开销。   
如果代码模型能够合理的访问的内存空间，则可以提升应用程序的性能并减小其体积。


下面我们主要介绍LoongArch上目前支持的三种代码模型。

我们还是以下面的代码为例，解释所示支持的代码情况。假设有代码

```c
#include <stdio.h>

long global_val = 0;

int main(int argc, char const *argv[])
{
	global_val = 0x100;

	printf("GLobal Value: %lx,\n", global_val);
	return 0;
}
```

代码模型最核心的问题，就是如何加载符号的地址，比如上述代码中，

数据``global_val``和函数``printf``都是全局的符号。


## 三种代码模型及其边界

代码模型是编译器和链接器共同遵守的寻址约定，而不是“某段反汇编必然长成的样子”。寄存器分配、优化级别、PIC/PIE、PLT、链接松弛（relax）以及工具链版本都会改变最终指令；应以重定位和链接后的地址范围验证结果，而不是照抄某次构建的机器码。

LoongArch ELF ABI 定义了 normal、medium、extreme 三种模型；GCC 也以 `-mcmodel=` 选择它们。当前 GCC 文档指出默认值可由 GCC 配置时的 `--with-cmodel=` 改变，因此构建系统应**显式**指定模型，或以 `gcc -Q --help=target` 检查实际默认值。[^laelf-code-model][^gcc-cmodel]

|模型|数据访问范围|函数调用范围|适用场景|
|---|---|---|---|
|`normal`|以 PC 为基准的 4 GiB 窗口：`[(PC & ~0xfff)-2 GiB-0x800, (PC & ~0xfff)+2 GiB-0x800)`|`[PC-128 MiB, PC+128 MiB-4]`|代码和数据布局紧凑的程序。|
|`medium`|与 `normal` 相同|`[PC-128 GiB-0x20000, PC+128 GiB-0x20000-4]`|数据仍在 4 GiB PC-relative 窗口内，但调用目标跨度更大。|
|`extreme`|完整 64 位地址空间|完整 64 位地址空间|链接布局不能满足前两种范围时；指令数和代码体积更高。|

### Normal 代码模型

数据访问使用 `R_LARCH_PCALA_HI20` 和 `R_LARCH_PCALA_LO12` 构成 PC-relative 地址；32 位全局对象的典型加载序列如下。`$t0`、`$a0` 只是 ABI 示例寄存器，编译器实际选择可以不同。

```
00:  pcalau12i $t0, %pc_hi20(g1)
     0: R_LARCH_PCALA_HI20  g1
04:  ld.w      $a0, $t0, %pc_lo12(g1)
     4: R_LARCH_PCALA_LO12  g1
```

对可通过 PLT 调用的外部函数，典型直接调用重定位为：

```
00:  bl %plt(foo)
     0: R_LARCH_B26 foo
```

### Medium 代码模型

`medium` 的数据访问范围与 `normal` 相同；它只把可直接调用的 PC-relative 范围扩大到 256 GiB。典型调用序列使用 `R_LARCH_CALL36`：

```
00:  pcaddu18i $ra, %call36(foo)
     0: R_LARCH_CALL36 foo
04:  jirl      $ra, $ra, 0
```

这并不表示任意数据也可以与当前 PC 相距 256 GiB。若数据超出其 PC-relative 窗口，应调整链接布局或使用 `extreme`，而不是仅把 `normal` 改为 `medium`。

### Extreme 代码模型

`extreme` 为数据和调用目标构造完整的 64 位地址。ABI 要求以下四条地址构造指令保持相邻：链接器需要据此定位 `pcalau12i` 的 PC，并对后两条指令应用重定位。不要在这四条指令间插入调试代码、调度指令或手写标签。

典型的数据加载序列为：

```
00:  pcalau12i $t1, %pc_hi20(g1)
     0: R_LARCH_PCALA_HI20   g1
04:  addi.d     $t0, $zero, %pc_lo12(g1)
     4: R_LARCH_PCALA_LO12   g1
08:  lu32i.d    $t0, %pc64_lo20(g1)
     8: R_LARCH_PCALA64_LO20 g1
0c:  lu52i.d    $t0, $t0, %pc64_hi12(g1)
     c: R_LARCH_PCALA64_HI12 g1
10:  ldx.w      $a0, $t1, $t0
```

函数调用在相同的四条地址构造指令之后，将偏移与基址相加再执行 `jirl`：

```
10:  add.d $t0, $t0, $t1
14:  jirl  $ra, $t0, 0
```

GCC 将 `-mcmodel=extreme` 与 `-fplt` 或 `-mexplicit-relocs=none` 视为不兼容；构建命令应显式使用 `-fno-plt` 和 `-mexplicit-relocs=always`（或工具链支持的等价非 `none` 设置）。[^gcc-cmodel]

### 可复现实验与验证

以下示例将源文件和构建产物分开，并查看对象文件中的重定位；它不会假定某个优化版本的确切机器码。链接布局必须实际超出相应范围时，才会测试到溢出/模型选择的边界。

```c
/* code_model.c */
int g1;
extern void foo(void);

int read_g1(void) { return g1; }
void call_foo(void) { foo(); }
```

```bash
CC=loongarch64-linux-gnu-gcc
READELF=loongarch64-linux-gnu-readelf
OBJDUMP=loongarch64-linux-gnu-objdump

for model in normal medium extreme; do
    if [ "$model" = extreme ]; then
        "$CC" -O0 -fno-pic -fno-pie -fno-plt \
          -mexplicit-relocs=always -mcmodel="$model" \
          -c code_model.c -o "code_model.${model}.o"
    else
        "$CC" -O0 -fno-pic -fno-pie \
          -mexplicit-relocs=always -mcmodel="$model" \
          -c code_model.c -o "code_model.${model}.o"
    fi
    "$READELF" -rW "code_model.${model}.o"
    "$OBJDUMP" -dr -M no-aliases "code_model.${model}.o"
done
```

比较三组输出时，应检查 `normal`/`medium` 的数据重定位是否相同、`medium` 调用是否出现 `R_LARCH_CALL36`，以及 `extreme` 是否使用完整的相邻地址构造序列。发布可执行文件前还应对最终 ELF 执行 `readelf -rW -lW` 和 `objdump -dr`；链接器的 relax 可能使最终序列短于对象文件。为了观察未松弛的链接结果，可向链接器传递 `-Wl,--no-relax`，但这只用于诊断，不能替代目标地址范围的验证。

[^laelf-code-model]: [LoongArch ELF ABI：Code Models](https://github.com/loongson/la-abi-specs/blob/release/laelf.adoc)；本文的范围、重定位名称和 extreme 相邻要求以该规范为准。
[^gcc-cmodel]: [GCC LoongArch Options：`-mcmodel`](https://gcc.gnu.org/onlinedocs/gcc/LoongArch-Options.html)；编译器支持情况及默认值以所使用 GCC 版本的文档为准。






