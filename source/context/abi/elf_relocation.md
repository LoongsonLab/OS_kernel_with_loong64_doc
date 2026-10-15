# ELF 可重定位

重定位（Relocation）是计算机程序在内存中动态调整代码和数据地址的过程，     
将程序中的逻辑地址（编译时生成的地址）转换为虚拟地址（内存中的实际地址）的过程，   
确保程序在不同内存位置运行时仍能正确执行。

本章节主要会讲解有关龙架构相关的可重定位类型及其具体的操作。


## 可重定位类型（Relocation Type）

下面是所有关于LoongArch的可重定位的类型[^loongarch_reloc_type]。

```c
/* LoongArch specific dynamic relocations */
#define R_LARCH_NONE		0
#define R_LARCH_32		1
#define R_LARCH_64		2
#define R_LARCH_RELATIVE	3
#define R_LARCH_COPY		4
#define R_LARCH_JUMP_SLOT	5
#define R_LARCH_TLS_DTPMOD32	6
#define R_LARCH_TLS_DTPMOD64	7
#define R_LARCH_TLS_DTPREL32	8
#define R_LARCH_TLS_DTPREL64	9
#define R_LARCH_TLS_TPREL32	10
#define R_LARCH_TLS_TPREL64	11
#define R_LARCH_IRELATIVE	12
#define R_LARCH_TLS_DESC32	13
#define R_LARCH_TLS_DESC64	14

/* Reserved for future relocs that the dynamic linker must understand.  */

/* used by the static linker for relocating .text.  */
#define R_LARCH_MARK_LA  20
#define R_LARCH_MARK_PCREL  21
#define R_LARCH_SOP_PUSH_PCREL  22
#define R_LARCH_SOP_PUSH_ABSOLUTE  23
#define R_LARCH_SOP_PUSH_DUP  24
#define R_LARCH_SOP_PUSH_GPREL  25
#define R_LARCH_SOP_PUSH_TLS_TPREL  26
#define R_LARCH_SOP_PUSH_TLS_GOT  27
#define R_LARCH_SOP_PUSH_TLS_GD  28
#define R_LARCH_SOP_PUSH_PLT_PCREL  29
#define R_LARCH_SOP_ASSERT  30
#define R_LARCH_SOP_NOT  31
#define R_LARCH_SOP_SUB  32
#define R_LARCH_SOP_SL  33
#define R_LARCH_SOP_SR  34
#define R_LARCH_SOP_ADD  35
#define R_LARCH_SOP_AND  36
#define R_LARCH_SOP_IF_ELSE  37
#define R_LARCH_SOP_POP_32_S_10_5  38
#define R_LARCH_SOP_POP_32_U_10_12  39
#define R_LARCH_SOP_POP_32_S_10_12  40
#define R_LARCH_SOP_POP_32_S_10_16  41
#define R_LARCH_SOP_POP_32_S_10_16_S2  42
#define R_LARCH_SOP_POP_32_S_5_20  43
#define R_LARCH_SOP_POP_32_S_0_5_10_16_S2  44
#define R_LARCH_SOP_POP_32_S_0_10_10_16_S2  45
#define R_LARCH_SOP_POP_32_U  46

/* used by the static linker for relocating non .text.  */
#define R_LARCH_ADD8  47
#define R_LARCH_ADD16  48
#define R_LARCH_ADD24  49
#define R_LARCH_ADD32  50
#define R_LARCH_ADD64  51
#define R_LARCH_SUB8  52
#define R_LARCH_SUB16  53
#define R_LARCH_SUB24  54
#define R_LARCH_SUB32  55
#define R_LARCH_SUB64  56
#define R_LARCH_GNU_VTINHERIT  57
#define R_LARCH_GNU_VTENTRY  58

/* reserved 59-63 */

#define R_LARCH_B16 64
#define R_LARCH_B21 65
#define R_LARCH_B26 66
#define R_LARCH_ABS_HI20 67
#define R_LARCH_ABS_LO12 68
#define R_LARCH_ABS64_LO20 69
#define R_LARCH_ABS64_HI12 70
#define R_LARCH_PCALA_HI20 71
#define R_LARCH_PCALA_LO12 72
#define R_LARCH_PCALA64_LO20 73
#define R_LARCH_PCALA64_HI12 74
#define R_LARCH_GOT_PC_HI20 75
#define R_LARCH_GOT_PC_LO12 76
#define R_LARCH_GOT64_PC_LO20 77
#define R_LARCH_GOT64_PC_HI12 78
#define R_LARCH_GOT_HI20 79
#define R_LARCH_GOT_LO12 80
#define R_LARCH_GOT64_LO20 81
#define R_LARCH_GOT64_HI12 82
#define R_LARCH_TLS_LE_HI20 83
#define R_LARCH_TLS_LE_LO12 84
#define R_LARCH_TLS_LE64_LO20 85
#define R_LARCH_TLS_LE64_HI12 86
#define R_LARCH_TLS_IE_PC_HI20 87
#define R_LARCH_TLS_IE_PC_LO12 88
#define R_LARCH_TLS_IE64_PC_LO20 89
#define R_LARCH_TLS_IE64_PC_HI12 90
#define R_LARCH_TLS_IE_HI20 91
#define R_LARCH_TLS_IE_LO12 92
#define R_LARCH_TLS_IE64_LO20 93
#define R_LARCH_TLS_IE64_HI12 94
#define R_LARCH_TLS_LD_PC_HI20 95
#define R_LARCH_TLS_LD_HI20 96
#define R_LARCH_TLS_GD_PC_HI20 97
#define R_LARCH_TLS_GD_HI20 98
#define R_LARCH_32_PCREL 99
#define R_LARCH_RELAX 100
/* 101 reserved (R_LARCH_DELETE was removed from the ABI) */
#define R_LARCH_ALIGN 102
#define R_LARCH_PCREL20_S2 103
/* 104 reserved (R_LARCH_CFA was removed from the ABI) */
#define R_LARCH_ADD6 105
#define R_LARCH_SUB6 106
#define R_LARCH_ADD_ULEB128 107
#define R_LARCH_SUB_ULEB128 108
#define R_LARCH_64_PCREL 109
#define R_LARCH_CALL36 110
#define R_LARCH_TLS_DESC_PC_HI20 111
#define R_LARCH_TLS_DESC_PC_LO12 112
#define R_LARCH_TLS_DESC64_PC_LO20 113
#define R_LARCH_TLS_DESC64_PC_HI12 114
#define R_LARCH_TLS_DESC_HI20 115
#define R_LARCH_TLS_DESC_LO12 116
#define R_LARCH_TLS_DESC64_LO20 117
#define R_LARCH_TLS_DESC64_HI12 118
#define R_LARCH_TLS_DESC_LD 119
#define R_LARCH_TLS_DESC_CALL 120
#define R_LARCH_TLS_LE_HI20_R 121
#define R_LARCH_TLS_LE_ADD_R 122
#define R_LARCH_TLS_LE_LO12_R 123
#define R_LARCH_TLS_LD_PCREL20_S2 124
#define R_LARCH_TLS_GD_PCREL20_S2 125
#define R_LARCH_TLS_DESC_PCREL20_S2 126
```



[^loongarch_reloc_type]: https://sourceware.org/git/?p=glibc.git;a=blob;f=elf/elf.h;h=46a01281cb0fb5322d5124f0443c11dea4d5b721;hb=HEAD


重定位的类型主要分为两大类：

- 静态重定位（Static Relocation）

  主要是由链接器使用（ld），将各个的.o文件合并成一个可执行文件时使用。

- 动态重定位（Dynamic Relocation）

  主要是程序在加载或者执行过程中(loader)，将各个需要重定位的符号解析，按照     
  类型定义，重新获得地址。此过程主要是运行过程中处理。


(reloc_type_dynamic)=
## 动态链接（dynamic link）

DSO（Dynamic Shared Object）是Linux/Unix系统中动态链接的共享库文件，以.so为扩展名。   
其核心特性是代码共享与地址无关性，允许多个进程共享同一份代码段，但数据段独立。

本小节主要介绍与龙架构相关的关于动态链接过程中需要的重定位类型。

主要有下面

```c
/* LoongArch specific dynamic relocations */
#define R_LARCH_NONE		0
#define R_LARCH_32		1
#define R_LARCH_64		2
#define R_LARCH_RELATIVE	3
#define R_LARCH_COPY		4
#define R_LARCH_JUMP_SLOT	5
#define R_LARCH_TLS_DTPMOD32	6
#define R_LARCH_TLS_DTPMOD64	7
#define R_LARCH_TLS_DTPREL32	8
#define R_LARCH_TLS_DTPREL64	9
#define R_LARCH_TLS_TPREL32	10
#define R_LARCH_TLS_TPREL64	11
#define R_LARCH_IRELATIVE	12
#define R_LARCH_TLS_DESC32	13
#define R_LARCH_TLS_DESC64	14

/* Reserved for future relocs that the dynamic linker must understand.  */
```
其中值15-19的保留为将来扩展使用。

我们下面一一介绍这些类型。

- **R_LARCH_32 类型**

使用``readelf -r main``可以得到下面的信息：

```
Relocation section '.rela.dyn' at offset 0x4b8 contains 6 entries:
  Offset     Info    Type            Sym.Value  Sym. Name + Addend
 0003001c  00000301 R_LARCH_32        00000000   deregister
```

表示此条可重定位需要做的操作是：

``*relocation_address = runtime_symbol_address + addend``

addend经常为0，有时也不为0，主要看编译时决定。

对应上面的显示就是指：在offset为0x7fd8的位置，写入符号deregister运行时的地址+addend

R_LARCH_32出现在32位的ELF可执行文件中。

- **R_LARCH_64 类型**

同上面的一致，但是所有的地址都是64位的。

```
Relocation section '.rela.dyn' at offset 0x4b8 contains 6 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000007fd8  000600000002   R_LARCH_64     0000000000000000     deregister
```

表示此条可重定位需要做的操作是：

``*relocation_address(64位地址) = runtime_symbol_address(64位地址) + addend``

- **R_LARCH_RELATIVE 类型**

表示对加载地址的运行时修复。

```
Relocation section '.rela.dyn' at offset 0x4b8 contains 6 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000007df0  000000000003 R_LARCH_RELATIVE                     788
000000007df8  000000000003 R_LARCH_RELATIVE                     730
000000008020  000000000003 R_LARCH_RELATIVE                     8020
```

表示此条可重定位需要做的操作是：

``*relocation_address(64位地址) = 对象文件加载到内存中的基址(64位地址) + addend``

假设当前的lib.so或者是（ELF 64-bit LSB pie executable, LoongArch），    
此时向0x7df0的地址，写入加载时的地址加上0x788的值。


- **R_LARCH_COPY 类型**

主要应运于可执行文件中内存的拷贝。

表示此条可重定位需要做的操作是：

``memcpy(relocation_address, runtime_symbol_address, sizeof(symbol))``


- **R_LARCH_JUMP_SLOT 类型**

主要应运于运行时PLT(Procedure Linkage Table)的支持。

表示此条可重定位需要做的操作是：

``*relocation_address(64位地址) = runtime_symbol_address(64位地址) + addend``

和R_LARCH_64的区别是R_LARCH_JUMP_SLOT用于PLT的函数符号处理。


- **R_LARCH_IRELATIVE 类型**
它获得目标地址是通过本地间接函数的返回结果。

如下面的例子所示：

```c
// 不同实现
int add_generic(int a, int b) { return a + b; }
int add_optfun(int a, int b) { return a + b; }

int flags = 0;

int add(int a, int b) __attribute__((ifunc("add_resolver")));

// 解析器函数
void *add_resolver() {
    if (flags) {
        return &add_optfun;  // 使用 优化版本
    } else {
        return &add_generic; // 通用版本
    }
}


int main(int argc, char const *argv[])
{
    int a = 100, b = 101;
    int res = 0;
    res = add(a, b);
    return 0;
}
```

```
bash> readelf -r ifunc

Relocation section '.rela.dyn' at offset 0x498 contains 7 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000007df0  000000000003 R_LARCH_RELATIVE                     768
000000007df8  000000000003 R_LARCH_RELATIVE                     710
000000008020  000000000003 R_LARCH_RELATIVE                     8020
000000007fd8  000600000002 R_LARCH_64        0000000000000000 _ITM_deregisterTM[...] + 0
000000007fe0  000900000002 R_LARCH_64        0000000000000000 _ITM_registerTMCl[...] + 0
000000007fe8  000a00000002 R_LARCH_64        0000000000000000 __cxa_finalize@GLIBC_2.36 + 0
000000008018  00000000000c R_LARCH_IRELATIVE                    800

Relocation section '.rela.plt' at offset 0x540 contains 3 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000008000  000700000005 R_LARCH_JUMP_SLOT 0000000000000000 __libc_start_main@GLIBC_2.36 + 0
000000008008  000800000005 R_LARCH_JUMP_SLOT 0000000000000000 abort@GLIBC_2.36 + 0
000000008010  000a00000005 R_LARCH_JUMP_SLOT 0000000000000000 __cxa_finalize@GLIBC_2.36 + 0

```

在函数被加载后，ld.so会先通过执行``add_resolver``函数来确定到底实行那个add函数。

## `.rela.*`、动态标签与运行时链接

LoongArch ELF 使用带显式 addend 的 `Elf*_Rela` 表项；因此在 Linux 用户态对象中，
常见的是 `.rela.dyn` 和 `.rela.plt`，而不是把 addend 隐含在被修改位置的 `.rel.*`。
`.rela.dyn` 通常承载 `R_LARCH_RELATIVE`、数据符号和 TLS 相关修复；`.rela.plt`
通常承载函数调用的 `R_LARCH_JUMP_SLOT`。这只是常见 GNU 链接布局，具体节名、顺序
和地址不是 ABI 常量。

动态段使用的是通用 ELF dynamic tags；当前 LoongArch ELF ABI 没有要求应用程序依赖
额外的 LoongArch 专有 dynamic tag。排查动态链接时，至少应关联检查下表的 tag 与
重定位节：

|Dynamic tag|关联内容|排查目的|
|-|-|-|
|`DT_RELA`、`DT_RELASZ`、`DT_RELAENT`|`.rela.dyn` 的地址、总大小和表项大小|确认动态加载器的常规重定位表|
|`DT_JMPREL`、`DT_PLTRELSZ`、`DT_PLTREL`|PLT 重定位表（通常显示为 `.rela.plt`）|确认外部函数延迟/立即绑定入口|
|`DT_PLTGOT`|GOT/PLT 所用的全局偏移表|关联 PLT 槽与 GOT 地址|
|`DT_SYMTAB`、`DT_STRTAB`|动态符号表和字符串表|解析 relocation 中的符号索引|
|`DT_NEEDED`、`DT_RUNPATH`|依赖库与搜索路径|解释符号最终从何处解析|

下面是一个动态 PIE 的 `readelf` 实际输出格式（不同 binutils、链接顺序和 ASLR 前的
链接地址会改变数值；应以本机构建的输出为准）：

```
$ readelf -dW app | rg 'NEEDED|RELA|JMPREL|PLTGOT'
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x0000000000000003 (PLTGOT)             0x0000000000020000
 0x0000000000000007 (RELA)               0x00000000000004b8
 0x0000000000000008 (RELASZ)             168 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x0000000000000017 (JMPREL)             0x0000000000000560
 0x0000000000000002 (PLTRELSZ)           72 (bytes)
 0x0000000000000014 (PLTREL)             RELA

$ readelf -rW app
Relocation section '.rela.dyn' at offset 0x4b8 contains 7 entries:
    Offset             Info             Type               Symbol's Value  Symbol's Name + Addend
0000000000020020  0000000000000003 R_LARCH_RELATIVE                          0000000000000768
0000000000020040  0006000000000002 R_LARCH_64           0000000000000000 __cxa_finalize + 0

Relocation section '.rela.plt' at offset 0x560 contains 3 entries:
    Offset             Info             Type               Symbol's Value  Symbol's Name + Addend
0000000000020000  0007000000000005 R_LARCH_JUMP_SLOT    0000000000000000 __libc_start_main + 0
```

可用同一组命令检查自己的产物：

```bash
loongarch64-linux-gnu-readelf -dWrW app
loongarch64-linux-gnu-objdump -drwC -j .plt -j .got -j .got.plt app
```

## GOT、PLT 与 TLS

GOT 保存由动态加载器修复的地址或 TLS 访问所需的条目；PLT 是外部函数调用的跳板，
其相应 GOT 槽经 `R_LARCH_JUMP_SLOT` 填充。位置无关代码访问全局数据时，静态链接器
可在 `.text` 中留下 `R_LARCH_GOT_PC_HI20`/`R_LARCH_GOT_PC_LO12` 等重定位；调用外部
函数则可见 `%plt(symbol)` 或动态表中的 `R_LARCH_JUMP_SLOT`。不要把“出现 GOT”简单
等同于“必须通过 PLT 调用”：数据访问和函数调用的路径不同。

TLS 模型同样应从重定位类型判断，而不是只看变量是否带 `__thread`：

|模型|编译/静态链接时常见重定位|动态加载时可能看到的重定位|
|-|-|-|
|LE|`R_LARCH_TLS_LE_*`|通常无需动态 TLS 表项|
|IE|`R_LARCH_TLS_IE_*`|`R_LARCH_TLS_TPREL32/64`|
|GD/LD|`R_LARCH_TLS_GD_*`、`R_LARCH_TLS_LD_*`|`R_LARCH_TLS_DTPMOD32/64`、`R_LARCH_TLS_DTPREL32/64`|
|TLS descriptor|`R_LARCH_TLS_DESC_*`|`R_LARCH_TLS_DESC32/64`|

例如，`readelf -rW libtls.so` 中的以下两行表明动态 TLS-GD 表项（实际偏移、符号版本
和 32/64 位后缀取决于目标 ABI）：

```
0000000000020030  0005000000000007 R_LARCH_TLS_DTPMOD64 0000000000000000 tls_counter + 0
0000000000020038  0005000000000009 R_LARCH_TLS_DTPREL64 0000000000000000 tls_counter + 0
```

## 重定位溢出：诊断而非截断

立即数字段和 PC 相对跳转都有可表示范围。链接器发现 `S + A - PC` 或其它计算结果
超出该 relocation 的范围时必须报错，不能静默截断。例如 `R_LARCH_B26` 要求 28 位
有符号、4 字节对齐的 PC 相对位移；medium 代码模型的 `R_LARCH_CALL36` 则要求 38 位
有符号、4 字节对齐的范围，且 `pcaddu18i + jirl` 两条指令必须相邻。

GNU ld 的实际错误文本会包含 relocation 名称和目标符号，典型形式如下：

```
ld: caller.o: relocation truncated to fit: R_LARCH_B26 against symbol `far_function'
ld: caller.o: relocation truncated to fit: R_LARCH_CALL36 against symbol `far_function'
```

遇到此类错误时，先以 `readelf -rW` 和 `objdump -drwC` 定位产生 relocation 的指令，
再检查链接脚本、代码模型（normal/medium/extreme）、`-fPIC` 与是否应经 GOT/PLT 间接
访问。不要通过修改机器码立即数来“修复”溢出。


## 静态链接（static link）

静态类型的部分如下表所示：

|值|ELF 重定位类型| 用法|描述|
|-|-|-|-|
|64 | R_LARCH_B16 | 18-bit PC-relative jump, ``%b16(symbol)``|(\*(uint32_t \*)PC)[25:10] = (S+A-PC)[17:2]|
|65 | R_LARCH_B21 | 23-bit PC-relative jump, ``%b21(symbol)``|(\*(uint32_t \*)PC)[4:0] = (S+A-PC)[22:18],<br>(\*(uint32_t \*) PC)[25:10] = (S+A-PC)[17:2]|
|66 | R_LARCH_B26 | 28-bit PC-relative jump, ``%plt(symbol)``|(\*(uint32_t \*)PC)[9:0] = (S+A-PC)[27:18], <br> (\*(uint32_t \*)PC)[25:10] = (S+A-PC)[17:2]|
|67 | R_LARCH_ABS_HI20 | [31：12] bits of 32/64-bit absolute address, <br> ``%abs_hi20(symbol)`` |(\*(uint32_t \*) PC)[24:5] = (S+A)[31:12]|
|68 | R_LARCH_ABS_LO12| [11:0] bits of 32/64-bit absolute address, <br>``%abs_lo12(symbol)`` | (\*(uint32_t \*) PC)[21:10] = (S+A)[11:0]|
|69 | R_LARCH_ABS64_LO20 | [51:32] bits of 64-bit absolute address, <br> ``%abs64_lo20(symbol)`` |(\*(uint32_t \*) PC)[24:5] = (S+A)[51:32]
|70 | R_LARCH_ABS64_HI12 | [63:52] bits of 64-bit absolute address, <br> ``%abs64_hi12(symbol)`` |(\*(uint32_t \*) PC)[21:10] = (S+A) [63:52]
|71 | R_LARCH_PCALA_HI20 | [31:12] bits of 32/64-bit PC-relative offset, <br> ``%pc_hi20(symbol)`` |(\*(uint32_t \*) PC)[24:5] = <br>(((S+A+0x800) & ~0xfff) - (PC & ~0xfff)) [31:12]|
|72 | R_LARCH_PCALA_LO12 | [11:0] bits of 32/64-bit address, <br> ``%pc_lo12(symbol)`` | (\*(uint32_t \*) PC)[21:10] = (S+A)[11:0]
|73 | R_LARCH_PCALA64_LO20    | [51:32] bits of 64-bit PC-relative offset, <br> ``%pc64_lo20(symbol)`` |(\*(uint32_t \*) PC) [24:5] = <br>(((S+A+0x8000'0000 + (((S+A) & 0x800) ? <br>(0x1000-0x1'0000'0000) : 0)) <br>& ~0xfff) - (PC-8 & ~0xfff))[51:32]|
|74 | R_LARCH_PCALA64_HI12 | [63:52] bits of 64-bit PC-relative offset, <br>``%pc64_hi12(symbol)`` |(\*(uint32_t \*) PC) [21:10] = <br>(((S+A+0x8000'0000 + (((S+A) & 0x800) ? <br>(0x1000-0x1'0000'0000) : 0)) & ~0xfff) <br>- (PC-12 & ~0xfff))[63:52]|
|75 | R_LARCH_GOT_PC_HI20 | [31:12] bits of 32/64-bit PC- relative offset to GOT entry, <br>``%got_pc_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = <br>(((GOT+G) & ~0xfff) - (PC & ~0xfff)) [31:12]|
|76 | R_LARCH_GOT_PC_LO12 | [11:0] bits of 32/64-bit GOT entry address,<br>``%got_pc_lo12(symbol)`` | (\*(uint32_t \*) PC) [21:10] = (GOT+G) [11:0]|
|77 | R_LARCH_GOT64_PC_LO20 | [51:32] bits of 64-bit PC-relative offset to GOT entry,<br>``%got64_pc_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = <br> (((GOT+G+0x8000'0000 + (((GOT+G) & 0x800)<br> ? (0x1000-0x1'0000'0000) : 0)) & ~0xfff) <br>- (PC-8 & ~0xfff))[51:32]|
|78 | R_LARCH_GOT64_PC_HI12 | [63:52] bits of 64-bit PC-relative offset to GOT entry,<br>``%got64_pc_hi12(symbol)`` | (\*(uint32_t \*) PC) [21:10] = <br> (((GOT+G+0x8000'0000 + (((GOT+G) & 0x800) <br>? (0x1000-0x1'0000'0000) : 0)) & ~0xfff) <br>- (PC-12 & ~0xfff)) [63:52]|
|79 | R_LARCH_GOT_HI20 | [31:12] bits of 32/64-bit GOT entry absolute address, <br>``%got_hi20(symbol)`` |(\*(uint32_t \*) PC) [24:5] = (GOT+G) [31:12] |
|80 | R_LARCH_GOT_LO12 | [11:0] bits of 32/64-bit GOT entry absolute address,<br>``%got_lo12(symbol)`` | (\*(uint32_t \*) PC) [21:10] = (GOT+G) [11:0]|
|81 | R_LARCH_GOT64_LO20 | [51:32] bits of 64-bit GOT entry absolute address,<br>``%got64_lo20(symbol)`` |(\*(uint32_t \*) PC) [24:5] = (GOT+G) [51:32]|
|82 | R_LARCH_GOT64_HI12 | [63:52] bits of 64-bit GOT entry absolute address,<br>``%got64_hi12(symbol)`` |(\*(uint32_t \*) PC) [21:10] = (GOT+G) [63:52]|
|83 | R_LARCH_TLS_LE_HI20 | [31:12] bits of TLS LE 32/64-bit offset from TP register,<br>``%le_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = T [31:12]|
|84 | R_LARCH_TLS_LE_LO12 | [11:0] bits of TLS LE 32/64-bit offset from TP register,<br>``%le_lo12(symbol)``|(\*(uint32_t \*) PC) [21:10] = T [11:0]|
|85 | R_LARCH_TLS_LE64_LO20 | [51:32] bits of TLS LE 64-bit offset from TP register,<br>``%le64_lo20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = T [51:32]|
|86 | R_LARCH_TLS_LE64_HI12 | [63:52] bits of TLS LE 64-bit offset from TP register,<br>``%le64_hi12(symbol)`` | (\*(uint32_t \*) PC) [21:10] = T [63:52]|
|87 | R_LARCH_TLS_IE_PC_HI20 | [31:12] bits of 32/64-bit PC-relative offset to TLS IE <br>GOT entry, ``%ie_pc_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = (((GOT+IE) & ~0xfff) <br>- (PC & ~0xfff)) [31:12]|
|88 | R_LARCH_TLS_IE_PC_LO12 | [11:0] bits of 32/64-bit TLS IE GOT entry address,<br>``%ie_pc_lo12(symbol)``| (\*(uint32_t \*) PC) [21:10] = (GOT+IE) [11:0]|
|89 | R_LARCH_TLS_IE64_PC_LO20 | [51:32] bits of 64-bit PC-relative offset to TLS IE GOT<br> entry, ``%ie64_pc_lo20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = <br>(((GOT+IE+0x8000'0000 +(((GOT+IE) & 0x800) <br>? (0x1000-0x1'0000'0000) : 0))<br>& ~0xfff) - (PC-8 & ~0xfff))[51:32]|
|90 | R_LARCH_TLS_IE64_PC_HI12 | [63:52] bits of 64-bit PC-relative offset to TLS IE GOT<br>entry, ``%ie64_pc_hi12(symbol)``| (\*(uint32_t \*) PC) [21:10] = <br>(((GOT+IE+0x8000'0000+ (((GOT+IE) & 0x800) <br>? (0x1000-0x1'0000'0000) : 0)) & ~0xfff) <br>- (PC-12 &~0xfff)) [63:52]|
|91 | R_LARCH_TLS_IE_HI20 | [31:12] bits of 32/64-bit TLS IE GOT entry absolute address,<br> ``%ie_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = (GOT+IE) [31:12]|
|92 | R_LARCH_TLS_IE_LO12 | [11:0] bits of 32/64-bit TLS IE GOT entry absolute address,<br> ``%ie_lo12(symbol)`` | (\*(uint32_t \*) PC) [21:10] = (GOT+IE) [11:0]|
|93 | R_LARCH_TLS_IE64_LO20 | [51:32] bits of 64-bit TLS IE GOT entry absolute address,<br>``%ie64_lo20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = (GOT+IE) [51:32] |
|94 | R_LARCH_TLS_IE64_HI12 | [63:52] bits of 64-bit TLS IE GOT entry absolute address,<br>``%ie64_hi12(symbol)`` | (\*(uint32_t \*) PC) [21:10] = (GOT+IE) [63:52]|
|95 | R_LARCH_TLS_LD_PC_HI20 | [31:12] bits of 32/64-bit PC-relative offset to TLS LD GOT<br> entry, ``%ld_pc_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = <br>(((GOT+GD) & ~0xfff) - (PC & ~0xfff)) [31:12]|
|96 | R_LARCH_TLS_LD_HI20 | [31:12] bits of 32/64-bit TLS LD GOT entry absolute address, <br> ``%ld_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = (GOT+GD) [31:12]|
|97 | R_LARCH_TLS_GD_PC_HI20 | [31:12] bits of 32/64-bit PC-relative offset to TLS GD GOT<br>entry, ``%gd_pc_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = <br>(((GOT+GD) & ~0xfff) - (PC & ~0xfff)) [31:12]|
|98 | R_LARCH_TLS_GD_HI20 | [31:12] bits of 32/64-bit TLS GD GOT entry absolute address,<br> ``%gd_hi20(symbol)`` | (\*(uint32_t \*) PC) [24:5] = (GOT+GD) [31:12]|
|99 | R_LARCH_32_PCREL | 32-bit PC relative | (\*(uint32_t \*) PC) = (S+A-PC) [31:0]|
|100| R_LARCH_RELAX | Instruction can be relaxed, <br>paired with a normal <br>relocation at the same address ||


上述描述中，符号的意义如下表所示：
|变量|描述|
|-|-|
|RtAddr | Runtime address of the symbol in the relocation entry |
|PC | The address of the instruction to be relocated|
|B  | Base address of an object loaded into the memory|
|S  | The address of the symbol in the relocation entry|
|A  | Addend field in the relocation entry associated with the symbol|
|GOT| The address of GOT (Global Offset Table)|
|G  | GOT-relative offset of the GOT entry of a symbol. For tls LD/GD symbols, <br> G is always equal to GD.|
|T  | TP-relative offset of a TLS LE/IE symbols|
|IE | GOT-relative offset of the GOT entry of a TLS IE symbol|
|GD | GOT-relative offset of the GOT entry of a TLS LD/GD/DESC symbol. If a symbol is <br> referenced by IE, GD/LD and DESC simultaneously, this symbol has five GOT entries.  <br> The first two are for GD/LD; the next two are for DESC; the last one is for IE.|
|PLT| The address of PLT entry of a function symbol|
|R(S) | The target (it may be a symbol, a GOT entry for a symbol) of an <br>R_LARCH_*_PCADD_HI20 relocation filling the immediate slot for a pcaddu12i <br>instruction at S|

