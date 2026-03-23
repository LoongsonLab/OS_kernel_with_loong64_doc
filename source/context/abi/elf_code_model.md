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


1. 正常代码模型（Normal code model）

此模型下：

- *数据的访问地址跨度*范围为：``[(PC & ~0xfff)-2GiB-0x800, (PC & ~0xfff)+2GiB-0x800)``    
可以跨越4GB。

指令模式如下所示：

```
00:  pcalau12i $t0, %pc_hi20(data) # 汇编指令代码
     0: R_LARCH_PCALA_HI20 data    # 可重定位类型

04:  ld.w $a0, $t0, %pc_lo12(data) # 汇编指令代码
     4: R_LARCH_PCALA_LO12 data    # 可重定位类型
```

我们反汇编上面的C代码，结果如下：

```
bash> loongarch64-linux-gnu-objdump -ald hello
  1c:	1a00000c 	pcalau12i   	$t0, 0
  20:	02c0018c 	addi.d      	$t0, $t0, 0
  24:	0284000d 	addi.w      	$t1, $zero, 256
  28:	2700018d 	stptr.d     	$t1, $t0, 0
```


- *函数的跳转地址跨度*范围为：``[PC-128MiB, PC+128MiB-4]``，相对PC跨越256MB内存空间。

指令模式如下所示：

```
00:  bl %plt(printf)
     0: R_LARCH_B26 printf
```

我们反汇编上面的C代码，结果如下：

```
  38:	00150185 	or          	$a1, $t0, $zero
  3c:	1a000004 	pcalau12i   	$a0, 0
  40:	02c00084 	addi.d      	$a0, $a0, 0
  44:	54000000 	bl          	0	# 44 <L0^A+0x18> # 跳转到printf函数
```

2. 中等代码模型（Medium code model）

此模型下：

- *数据的访问地址跨度*范围为：``[(PC & ~0xfff)-2GiB-0x800, (PC & ~0xfff)+2GiB-0x800)``    
可以跨越4GB。

指令模式如下所示：

```
00:  pcalau12i $t0, %pc_hi20(data) # 汇编指令代码
     0: R_LARCH_PCALA_HI20 data    # 可重定位类型

04:  ld.w $a0, $t0, %pc_lo12(data) # 汇编指令代码
     4: R_LARCH_PCALA_LO12 data    # 可重定位类型
```

我们反汇编上面的C代码，结果如下：

```
bash> loongarch64-linux-gnu-objdump -ald hello
  1c:	1a00000c 	pcalau12i   	$t0, 0
  20:	02c0018c 	addi.d      	$t0, $t0, 0
  24:	0284000d 	addi.w      	$t1, $zero, 256
  28:	2700018d 	stptr.d     	$t1, $t0, 0
```

**注意： 数据访问和上述的Normal模式一样。**


- *函数的跳转地址跨度*范围为：``[PC-128GiB-0x20000, PC+128GiB-0x20000-4]``，相对PC跨越256GB内存空间。

指令模式如下所示：

```
00:  pcaddu18i $ra, %call36(printf) # 汇编指令代码
     0: R_LARCH_CALL36 printf       # 可重定位类型

04:  jirl $ra, $ra, 0
```

我们反汇编上面的C代码，结果如下：

```
   120000740:	1a000004 	pcalau12i   	$a0, 0
   120000744:	02dda084 	addi.d      	$a0, $a0, 1896
   120000748:	1e000001 	pcaddu18i   	$ra, 0
   12000074c:	4ffdb821 	jirl        	$ra, $ra, -584    # 跳转到printf函数
```


3. 极限代码模型（Extreme code model）

此模型下：

- *数据的访问地址跨度*范围为：整个64位地址空间。

指令模式如下所示（加载）：

```
00: pcalau12i $t1, %pc_hi20(data)       # 汇编指令代码
    0: R_LARCH_PCALA_HI20 data
04: addi.d $t0, $zero, %pc_lo12(data)   # 汇编指令代码
    4: R_LARCH_PCALA_LO12 data
08: lu32i.d $t0, %pc64_lo20(data)       # 汇编指令代码
    8: R_LARCH_PCALA64_LO20 data
0c: lu52i.d $t0, $t0, %pc64_hi12(data)  # 汇编指令代码
    c: R_LARCH_PCALA64_HI12 data
10: ldx.w $a0, $t1, $t0                 # 汇编指令代码
```

我们反汇编上面的C代码，结果如下：

```
bash> loongarch64-linux-gnu-objdump -ald hello
  1c:	1a00000d 	pcalau12i   	$t1, 0
  20:	02c0000c 	addi.d      	$t0, $zero, 0
  24:	1600000c 	lu32i.d     	$t0, 0
  28:	0300018c 	lu52i.d     	$t0, $t0, 0
  2c:	0284000e 	addi.w      	$t2, $zero, 256
  30:	381c31ae 	stx.d       	$t2, $t1, $t0
```

- *函数的跳转地址跨度*范围为：整个64位地址空间。

指令模式如下所示：

```
00: pcalau12i $t1, %pc_hi20(foo)      # 汇编指令代码
    0: R_LARCH_PCALA_HI20 foo
04: addi.d $t0, $zero, %pc_lo12(foo)  # 汇编指令代码
    4: R_LARCH_PCALA_LO12 foo
08: lu32i.d $t0, %pc64_lo20(foo)      # 汇编指令代码
    8: R_LARCH_PCALA64_LO20 foo
0c: lu52i.d $t0, $t0, %pc64_hi12(foo) # 汇编指令代码
    c: R_LARCH_PCALA64_HI12 foo
10: add.d $t0, $t0, $t1               # 汇编指令代码
14: jirl $ra, $t0, 0                  # 汇编指令代码
```

我们反汇编上面的C代码，结果如下：

```
  60:	1a00000d 	pcalau12i   	$t1, 0
  64:	02c0000c 	addi.d      	$t0, $zero, 0
  68:	1600000c 	lu32i.d     	$t0, 0
  6c:	0300018c 	lu52i.d     	$t0, $t0, 0
  70:	380c31ac 	ldx.d       	$t0, $t1, $t0
  74:	4c000181 	jirl        	$ra, $t0, 0# 跳转到printf函数
```


:::{tip}
1. 对于GCC，默认的-mcmodel=normal，使用参数-mcmodel=medium和-mcmodel=extreme分别采用medium和extreme模式。

2. 对于应用于链接器时，ld有可能会进行优化，也就是我们所说的relax。想要查看原始的没有优化的，则可以参数--no-relax（链接器参数）

```bash
loongarch64-linux-musl-gcc code_model.c -o code_model.extreme.elf -O0 -g -mcmodel=extreme -Wl,--no-relax
loongarch64-linux-musl-objdump -alD -M no-aliases code_model.extreme.elf > code_model.extreme.elf.s
```

3. 其他GCC支持的-mcmodel的参数，比如``large, tiny, tiny-static``目前龙架构暂时不支持。

:::








