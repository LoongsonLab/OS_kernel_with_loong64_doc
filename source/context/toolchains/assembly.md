# 汇编代码与链接脚本

## 与LoongArch相关的汇编指令

本小节主要涉及汇编相关的内容，帮助理解LoongArch底层指令。

LoongArch的汇编指令例子如下所示（下面代码的功能就是一个打印``printf("LoongArch!\n")``）：

```
	.globl	global_val
	.align	3
	.type	global_val, @object
	.size	global_val, 8
global_val:
	.space	8
	
	.globl	global_val_int
	.align	2
	.type	global_val_int, @object
	.size	global_val_int, 4
global_val_int:
	.space	4

	.section	.rodata
	.align	3
.local_strings:
	.ascii	"LoongArch!\000"

# code here...
.text
.align	2
.globl	main
.type	main, @function
main:
	addi.d	$r3,$r3,-32
	
	st.d	$r1,$r3,24
	st.d	$r22,$r3,16

	addi.d	$r22,$r3,32

	or	    $r12,$r4,$r0
	st.d	$r5,$r22,-32
	st.w	$r12,$r22,-20
	
	la.local	$r12,global_val
	addi.w	$r13,$r0,1234			# 0x4d2
	stptr.d	$r13,$r12,0
	
	la.local	$r12,global_val_int
	addi.w	$r13,$r0,1234			# 0x4d2
	stptr.w	$r13,$r12,0
	
	la.local	$r4, .local_strings
	bl	    %plt(puts)
	or	    $r12,$r0,$r0
	or	    $r4,$r12,$r0
	ld.d	$r1,$r3,24

	ld.d	$r22,$r3,16

	addi.d	$r3,$r3,32
	jr	$r1

.LFE0:
	.size	main, .-main
```

:::{note}
LoongArch的汇编指令的寄存器都是以``$``开始的，比如寄存器a0，在汇编中写作``$a0``。

如果见到没有带$符号的，首先查找有没有如下的定义：

```c
#define a0	$r4	/* argument registers, a0/a1 reused as v0/v1 for return value */
#define a1	$r5
#define a2	$r6
#define a3	$r7
#define a4	$r8
#define a5	$r9
```
如果没有上面的汇编定义，直接书写汇编器会报错。（其实是汇编器在预处理时，将符号进行了替换）

之所以出现上面宏定义的方式，是为了书写方便，也和其他架构的书写方式保持一致。其实mips也是采用这样的方式。

```bash
shell>> loongarch64-linux-musl-gcc test_asm_main.S -o test_asm_main.S.o -c
test_asm_main.S: Assembler messages:
test_asm_main.S:15: Error: no match insn: move	a0,a1
```

还需要注意的是：寄存器编号的写法也有两种方式：

- 采用ABI的方式书写，比如下面的汇编指令写法，例如
	```
	addi.d		$a5, $a5, 8
	```

- 按照``r+寄存器编号``的方式书写，例如
	```
	addi.d		$r9, $r9, 8
	```
:::

> LoongArch架构有32个通用寄存器，使用``$r0-$r31``表示，其中``$r0 或者 $zero``总是保持0值，在LA32中，这些寄存器的宽度是32位，
在LA64中，这些寄存器的宽度是64位。其中我们使用``$r1``作为返回地址的寄存器``$ra``，``$r3``作为栈指针``$sp``。


> LoongArch架构有32个浮点寄存器，使用``$f0-$f31``表示。有8个状态标志寄存器，``$fcc0-$fcc7``，每个寄存器的宽度是
1位，存放的是浮点比较指令``fcmp``的结果。另外还有4个浮点状态寄存器，``$fcsr0-$fcsr3``，每个寄存器宽度是32位，其中``$fcsr1-$fcsr3``是状态寄存器``$fcsr0``部分域值的别名。具体可查看[龙芯架构参考手册 卷一：基础架构](https://loongson.cn/uploads/images/2025032109191292796.%E9%BE%99%E6%9E%B6%E6%9E%84%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C%E5%8D%B7%E4%B8%80_r1p11.pdf)。



### 伪汇编指令

1. move指令

```
move  $rd, $rj
```

描述： 将rj寄存器的值复制到rd寄存器中。move是一个伪汇编指令，在LoongArch的指令手册是没有的，他是``or $rd, $rj, $zero``指令的别名(alias)。

比如： `` move $a0, $a1``

我们使用objdump反汇编出它的原始指令。

```bash
$ loongarch64-linux-gnu-gcc test_asm_main.S -o test_asm_main.S.o -c
$ loongarch64-linux-gnu-objdump -alD -M no-aliases,numeric test_asm_main.S.o > test_asm_main.S.o.s
```
其中使用objdump -M no-aliases可以显示原始的指令，如下所示：
```
or          	$r4, $r5, $r0
```


2. li.d/li.w指令

语法：
```
li.w  $rd, imm32
li.d  $rd, imm64
```

描述：``li.w``是将 imm32（32位立即数）加载到rd中，如果imm32查出了32位立即数的范围，汇编器会报错。

``
li.w $a0, 0x123456789abc
``

``
Fatal error: li overflow: hi32:0x1234 lo32:0x56789abc
``

``li.d``是将 imm64（64位立即数）加载到rd中。


实际上汇编器会把``li.w和li.d``伪指令使用指令序列``lu12i.w, ori, lu32i.d和lu52i.d``指令替换。

下面是示例代码 ``li.w $t0, imm32``的展开指令：
```
lu12i.w     $t0, imm32[31:12]
ori         $t0, $t0, imm32[11:0]
```

下面是示例代码 ``li.d $t0, imm64``的展开指令：
```
lu12i.w     $t0, imm64[31:12]
ori         $t0, $t0, imm64[11:0]
lu32i.d     $t0, imm64[51:32]
lu52i.d     $t0, $t0, imm64[63:52]
```

:::{note}

上述代码的指令拆分序列会进行优化，比如``ori  $t0, $t0, imm64[11:0]``
假设我们加载的imm64[11:0]是0的话，这条指令时没有必要执行的。

因此会被优化掉，所以有时候li.d指令反汇编过来并不是严格意义
上对应的四条指令，有时候可能是1-3条等等。

:::


3. jr和ret伪指令

语法：
```
jr  $t0
ret
```

``jr  $t0``是跳转到$t0的地址去执行，它的指令``jirl   $zero, $t0, 0``的别称。

``ret``是函数返回指令，她是指令``jirl $zero, $ra, 0``的别称。


4. call36和tail36

call36的语法是： ``call36  symbol_name``，意思是跳转到symbol_name地址去执行。它的实际反汇编是
```
pcaddu18i  $ra, %call36(symbol_name)
jirl       $ra, $ra, 0
```

tail36的语法是： ``tail36  $rd, symbol_name``，意思是跳转到symbol_name地址去执行, 同时将返回地址保存到$rd, ，它也使用$rd寄存器作为中间保存临时地址。它的实际反汇编是
```
pcaddu18i  $rd, %call36(symbol_name)
jirl       $rd, $rd, 0
```

:::{caution}
tail36需要一个寄存器，而call36默认使用$ra寄存器。``tail36 $ra, sym_name``和``call36  sym_name``是等价的。

需要注意下，这两个伪汇编指令的跳转地址范围
```
 [PC-128GiB-0x20000, PC+128GiB-0x20000-4]
```

:::

5.分支指令伪汇编

```
bgt    $rj, $rd, si18 or symbol
ble    $rj, $rd, si18 or symbol
bgtu   $rj, $rd, si18 or symbol
bleu   $rj, $rd, si18 or symbol
```
上述指令时LoongArch指令手册没有的，但是可以使用别的指令来实现，下面是它具体LoongArch指令的对应关系

|**伪汇编指令**| **具体的含义** | **实际的指令**|
|--|--|--|
|``bgt    $rj, $rd, si18 or symbol`` | if (signed($rj) > signed($rd)) 则跳转到目标地址 | ``blt $rd, $rj, si18 or symbol`` |
|``ble    $rj, $rd, si18 or symbol`` | if (signed($rj) <= signed($rd)) 则跳转到目标地址 | ``bge $rd, $rj, si18 or symbol`` |
|``bgtu    $rj, $rd, si18 or symbol`` | if (unsigned($rj) > unsigned($rd)) 则跳转到目标地址 | ``bltu $rd, $rj, si18 or symbol`` |
|``bleu    $rj, $rd, si18 or symbol`` | if (unsigned($rj)<= unsigned($rd)) 则跳转到目标地址 | ``bgeu $rd, $rj, si18 or symbol`` |

:::{caution}
```
注意上述指令中，$rj, $rd的顺序！
```
:::


```
bltz 	$rd, si18 or symbol
bgtz 	$rd, si18 or symbol
blez 	$rd, si18 or symbol
bgez 	$rd, si18 or symbol
```
上述指令时LoongArch指令手册没有的，但是可以使用别的指令来实现，下面是它具体LoongArch指令的对应关系

|**伪汇编指令**| **具体的含义** | **实际的指令**|
|--|--|--|
|``bltz $rd, si18 or symbol`` | if (signed($rd) > 0) 则跳转到目标地址 | ``blt $rd, $zero, si18 or symbol`` |
|``bgtz $rd, si18 or symbol`` | if (signed($rd) < 0) 则跳转到目标地址 | ``blt $zero, $rd, si18 or symbol`` |
|``blez $rd, si18 or symbol`` | if (signed($rd) <= 0) 则跳转到目标地址 | ``bge $zero, $rd, si18 or symbol`` |
|``bgez $rd, si18 or symbol`` | if (signed($rd) >= 0) 则跳转到目标地址 | ``bge $rd, $zero, si18 or symbol`` |


### 地址加载指令
这个小节主要处理将一个地址加载到寄存器，这里面包含很多的知识点，因此单独作为一个小节来说明。








### 内嵌汇编



### 参考阅读

1. [Assembly Language Programming Guide LoongArch, 英文版](https://github.com/loongson/la-asm-manual/releases/download/release-1.1/la-asm-manual-v1.1.pdf)

## 链接脚本

GNU LD 链接脚本是控制链接器行为的核心配置文件，通过定义内存布局、段分配规则和符号映射，实现对最终可执行文件或库的精确控制。以下是其核心语法和用法详解：

---

### 一、基础语法结构
#### 1. **MEMORY 坽块**  
定义物理内存区域及其属性，格式为：  
```text
MEMORY
{
  名称 (属性) : ORIGIN = 起始地址, LENGTH = 大小
}
```
- **属性**：`r`（可读）、`w`（可写）、`x`（可执行）、`a`（可分配）、`i`（已初始化）等，用 `!` 反转属性。  
- **示例**：  
  ```text
  FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 2M   # Flash存储器，只读可执行
  RAM (rwx)   : ORIGIN = 0x20000000, LENGTH = 128K # RAM，可读写执行
  ```

#### 2. **SECTIONS 块**  
定义段的放置规则，格式为：  
```text
SECTIONS
{
  段名称 : [AT(加载地址)] [ALIGN(对齐值)] {
    内容规则
  } >目标内存区域
}
```
- **关键概念**：  
  - **VMA（Virtual Memory Address）**：运行时地址。  
  - **LMA（Load Memory Address）**：加载地址（默认与 VMA 相同）。  
- **示例**：  
  ```text
  .text : {
    *(.text)       # 合并所有输入文件的 .text 段
    KEEP(*(.init)) # 保留初始化段不被优化
  } >FLASH AT>FLASH # 运行在 Flash，加载地址也为 Flash
  ```

---

### 二、核心命令与操作
#### 1. **段内容控制**
- **通配符匹配**：  
  - `*(.text)`：所有输入文件的 `.text` 段。  
  - `lib*.a:(.text)`：所有 `lib` 开头的库文件的 `.text` 段。  
- **排除规则**：  
  ```text
  *(EXCLUDE_FILE(boot.o) .text)  # 排除 boot.o 的 .text 段
  ```

#### 2. **符号操作**
- **符号赋值**：  
  ```text
  _start = 0x8000000;        # 定义入口地址
  data_start = ADDR(.data);  # 获取 .data 段地址
  ```  
- **PROVIDE 关键字**：  
  ```text
  PROVIDE(etext = .);        # 若未定义 etext，则设为当前地址
  ```

#### 3. **内存对齐与填充**
- **对齐**：  
  ```text
  . = ALIGN(4);              # 对齐到 4 字节边界
  .text : ALIGN(16) { ... }  # 段起始地址对齐到 16 字节
  ```
- **填充**：  
  ```text
  .bss : {
    *(.bss)
    FILL(0x90)               # 用 0x90 填充剩余空间
  }
  ```

#### 4. **动态表达式**
- **SIZEOF(section)**：获取段大小。  
- **ADDR(section)**：获取段起始地址。  
- **NEXT(address)**：返回对齐后的下一个地址。  
  ```text
  .data : {
    *(.data)
    _data_end = . + SIZEOF(.data);  # 计算 .data 结束地址
  }
  ```

---

### 三、高级功能
#### 1. **多内存区域管理**
通过 `MEMORY` 定义多个区域，结合 `>region` 指定段存放位置：  
```text
MEMORY
{
  ROM (rx)  : ORIGIN = 0x0, LENGTH = 32K
  RAM (rwx) : ORIGIN = 0x8000, LENGTH = 128K
}

SECTIONS
{
  .text : { *(.text) } >ROM
  .data : { *(.data) } >RAM AT>ROM  # 数据从 ROM 加载到 RAM
}
```

#### 2. **段叠加与覆盖**
- **OVERLAY**：允许段重叠加载（需配合 `INSERT` 命令）。  
- **PHDRS**：定义程序头表（如 ELF 的加载段）。

#### 3. **错误检查**
- **ASSERT**：条件不满足时终止链接。  
  ```text
  ASSERT(_end <= 0x20000000, "内存溢出")
  ```
- **NOCROSSREFS**：禁止段间交叉引用。  
  ```text
  NOCROSSREFS(.text .data)  # .text 和 .data 不可相互引用
  ```

---

### 四、典型应用场景
#### 1. **嵌入式开发**
- **Flash-RAM 分离**：将代码段（.text）放在 Flash，数据段（.data）从 Flash 加载到 RAM：  
  ```text
  SECTIONS
  {
    .text : { *(.text) } >FLASH
    .data : { *(.data) } >RAM AT>FLASH
    .bss  : { *(.bss) } >RAM
  }
  ```
- **启动代码**：通过 `KEEP(*(.init))` 保留初始化代码。

#### 2. **库文件优化**
- **隐藏符号**：使用 `HIDDEN(symbol)` 限制符号可见性。  
- **强制分配**：`FORCE_COMMON_ALLOCATION` 为未初始化符号分配空间。

#### 3. **安全加固**
- **反调试**：通过 `PHDRS` 修改程序头，隐藏调试信息。  
- **代码混淆**：重命名符号或调整段顺序。

---

### 五、实战示例
#### 示例 1：简单链接脚本
```text
MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 512K
  RAM (rwx)  : ORIGIN = 0x20000000, LENGTH = 128K
}

SECTIONS
{
  .text : {
    KEEP(*(.isr_vector))  # 保留中断向量表
    *(.text)
  } >FLASH

  .data : {
    *(.data)
  } >RAM AT>FLASH

  .bss : {
    *(.bss)
    *(COMMON)
  } >RAM
}
```

#### 示例 2：动态数据加载
```text
SECTIONS
{
  .rodata : {
    *(.rodata)
  } >FLASH

  .data : {
    _data_load = LOADADDR(.data);  # 记录加载地址
    _data_start = ADDR(.data);     # 记录运行地址
    *(.data)
  } >RAM AT>_data_load
}
```

---

### 六、调试与验证
1. **查看段信息**：  
   ```bash
   loongarch64-linux-gnu-objdump -h test_main  # 显示段布局
   ```
   显示如下：
   ```text
   test_main:     file format elf64-loongarch

   Sections:
   Idx Name          Size      VMA               LMA               File off  Algn
     0 .interp       0000001e  0000000120000200  0000000120000200  00000200  2**0
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     1 .hash         00000038  0000000120000220  0000000120000220  00000220  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     2 .gnu.hash     0000001c  0000000120000258  0000000120000258  00000258  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     3 .dynsym       000000d8  0000000120000278  0000000120000278  00000278  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     4 .dynstr       00000090  0000000120000350  0000000120000350  00000350  2**0
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     5 .rela.dyn     00000090  00000001200003e0  00000001200003e0  000003e0  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     6 .rela.plt     00000060  0000000120000470  0000000120000470  00000470  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
     7 .plt          00000060  00000001200004d0  00000001200004d0  000004d0  2**4
                     CONTENTS, ALLOC, LOAD, READONLY, CODE
     8 .text         00000200  0000000120000540  0000000120000540  00000540  2**5
                     CONTENTS, ALLOC, LOAD, READONLY, CODE
     9 .rodata       0000000b  0000000120000740  0000000120000740  00000740  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
    10 .eh_frame_hdr 00000014  000000012000074c  000000012000074c  0000074c  2**2
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
    11 .eh_frame     0000003c  0000000120000760  0000000120000760  00000760  2**3
                     CONTENTS, ALLOC, LOAD, READONLY, DATA
    12 .init_array   00000008  000000012001fdf0  000000012001fdf0  0000fdf0  2**3
                     CONTENTS, ALLOC, LOAD, DATA
    13 .fini_array   00000008  000000012001fdf8  000000012001fdf8  0000fdf8  2**3
                     CONTENTS, ALLOC, LOAD, DATA
    14 .dynamic      000001b0  000000012001fe00  000000012001fe00  0000fe00  2**3
                     CONTENTS, ALLOC, LOAD, DATA
    15 .got          00000040  000000012001ffb0  000000012001ffb0  0000ffb0  2**3
                     CONTENTS, ALLOC, LOAD, DATA
    16 .got.plt      00000030  000000012001fff0  000000012001fff0  0000fff0  2**3
                     CONTENTS, ALLOC, LOAD, DATA
    17 .sdata        00000008  0000000120020020  0000000120020020  00010020  2**3
                     CONTENTS, ALLOC, LOAD, DATA
    18 .bss          00000038  0000000120020028  0000000120020028  00010028  2**3
                     ALLOC
    19 .comment      00000012  0000000000000000  0000000000000000  00010028  2**0
                     CONTENTS, READONLY
    20 .debug_aranges 00000030  0000000000000000  0000000000000000  0001003a  2**0
                     CONTENTS, READONLY, DEBUGGING, OCTETS
    21 .debug_info   000000aa  0000000000000000  0000000000000000  0001006a  2**0
                     CONTENTS, READONLY, DEBUGGING, OCTETS
    22 .debug_abbrev 00000071  0000000000000000  0000000000000000  00010114  2**0
                     CONTENTS, READONLY, DEBUGGING, OCTETS
    23 .debug_line   00000062  0000000000000000  0000000000000000  00010185  2**0
                     CONTENTS, READONLY, DEBUGGING, OCTETS
    24 .debug_str    0000009d  0000000000000000  0000000000000000  000101e7  2**0
                     CONTENTS, READONLY, DEBUGGING, OCTETS
    25 .debug_line_str 0000003b  0000000000000000  0000000000000000  00010284  2**0
                     CONTENTS, READONLY, DEBUGGING, OCTETS
   ```



2. **生成映射文件**：  
   ```bash
   loongarch64-linux-gnu-ld -T linker.ld -M=map.txt main.c
   ```
3. **验证符号地址**：  
   ```bash
   loongarch64-linux-gnu-nm test_main | grep main  # 检查 main 函数地址
   ```
   显示如下：
   ```text
                     U __libc_start_main
	0000000120000704 T main
   ```

---

### 七、总结
GNU LD 链接脚本通过 **MEMORY** 和 **SECTIONS** 两大核心块，结合符号操作、对齐填充和动态表达式，实现了对程序内存布局的完全控制。其典型应用包括嵌入式系统的 Flash-RAM 分离、库文件优化和安全加固。掌握链接脚本是进行底层开发和性能调优的关键技能。


### 以LoongArch为例说明链接脚本

下面是LoongArch平台GNU ld默认的链接脚本，按照上面的说明，我们分析下链接脚本。

```text
/* Script for -z combreloc */
/* Copyright (C) 2014-2025 Free Software Foundation, Inc.
   Copying and distribution of this script, with or without modification,
   are permitted in any medium without royalty provided the copyright
   notice and this notice are preserved.  */

OUTPUT_FORMAT("elf64-loongarch", "elf64-loongarch", "elf64-loongarch")
OUTPUT_ARCH(loongarch)

ENTRY(_start)

SEARCH_DIR("=/loongarch64-linux-musl/lib64"); SEARCH_DIR("=/usr/local/lib64"); SEARCH_DIR("=/lib64"); SEARCH_DIR("=/usr/lib64"); SEARCH_DIR("=/loongarch64-linux-musl/lib"); SEARCH_DIR("=/usr/local/lib"); SEARCH_DIR("=/lib"); SEARCH_DIR("=/usr/lib");

SECTIONS
{
  /* Read-only sections, merged into text segment: */
  PROVIDE (__executable_start = SEGMENT_START("text-segment", 0x120000000));
  . = SEGMENT_START("text-segment", 0x120000000) + SIZEOF_HEADERS;
  /* Place the build-id as close to the ELF headers as possible.  This
     maximises the chance the build-id will be present in core files,
     which GDB can then use to locate the associated debuginfo file.  */
  .note.gnu.build-id  : { *(.note.gnu.build-id) }
  .interp         : { *(.interp) }
  .hash           : { *(.hash) }
  .gnu.hash       : { *(.gnu.hash) }
  .dynsym         : { *(.dynsym) }
  .dynstr         : { *(.dynstr) }
  .gnu.version    : { *(.gnu.version) }
  .gnu.version_d  : { *(.gnu.version_d) }
  .gnu.version_r  : { *(.gnu.version_r) }
  .rela.dyn       :
    {
      *(.rela.init)
      *(.rela.text .rela.text.* .rela.gnu.linkonce.t.*)
      *(.rela.fini)
      *(.rela.rodata .rela.rodata.* .rela.gnu.linkonce.r.*)
      *(.rela.data .rela.data.* .rela.gnu.linkonce.d.*)
      *(.rela.tdata .rela.tdata.* .rela.gnu.linkonce.td.*)
      *(.rela.tbss .rela.tbss.* .rela.gnu.linkonce.tb.*)
      *(.rela.ctors)
      *(.rela.dtors)
      *(.rela.got)
      *(.rela.sdata .rela.sdata.* .rela.gnu.linkonce.s.*)
      *(.rela.sbss .rela.sbss.* .rela.gnu.linkonce.sb.*)
      *(.rela.sdata2 .rela.sdata2.* .rela.gnu.linkonce.s2.*)
      *(.rela.sbss2 .rela.sbss2.* .rela.gnu.linkonce.sb2.*)
      *(.rela.bss .rela.bss.* .rela.gnu.linkonce.b.*)
      PROVIDE_HIDDEN (__rela_iplt_start = .);
      *(.rela.iplt)
      PROVIDE_HIDDEN (__rela_iplt_end = .);
    }
  .rela.plt       :
    {
      *(.rela.plt)
    }
  .relr.dyn : { *(.relr.dyn) }
  /* Start of the executable code region.  */
  .init           :
  {
    KEEP (*(SORT_NONE(.init)))
  } =0x00004003
  .plt            : { *(.plt) }
  .iplt           : { *(.iplt) }
  .text           :
  {
    *(.text.unlikely .text.*_unlikely .text.unlikely.*)
    *(.text.exit .text.exit.*)
    *(.text.startup .text.startup.*)
    *(.text.hot .text.hot.*)
    *(SORT(.text.sorted.*))
    *(.text .stub .text.* .gnu.linkonce.t.*)
    /* .gnu.warning sections are handled specially by elf.em.  */
    *(.gnu.warning)
  } =0x00004003
  .fini           :
  {
    KEEP (*(SORT_NONE(.fini)))
  } =0x00004003
  PROVIDE (__etext = .);
  PROVIDE (_etext = .);
  PROVIDE (etext = .);
  /* Start of the Read Only Data region.  */
  .rodata         : { *(.rodata .rodata.* .gnu.linkonce.r.*) }
  .rodata1        : { *(.rodata1) }
  .sdata2         :
  {
    *(.sdata2 .sdata2.* .gnu.linkonce.s2.*)
  }
  .sbss2          : { *(.sbss2 .sbss2.* .gnu.linkonce.sb2.*) }
  .eh_frame_hdr   : { *(.eh_frame_hdr) *(.eh_frame_entry .eh_frame_entry.*) }
  .eh_frame       : ONLY_IF_RO { KEEP (*(.eh_frame)) *(.eh_frame.*) }
  .sframe         : ONLY_IF_RO { *(.sframe) *(.sframe.*) }
  .gcc_except_table   : ONLY_IF_RO { *(.gcc_except_table .gcc_except_table.*) }
  .gnu_extab   : ONLY_IF_RO { *(.gnu_extab*) }
  /* These sections are generated by the Sun/Oracle C++ compiler.  */
  .exception_ranges   : ONLY_IF_RO { *(.exception_ranges*) }
  /* Various note sections.  Placed here so that they are always included
     in the read-only segment and not treated as orphan sections.  The
     current orphan handling algorithm does place note sections after R/O
     data, but this is not guaranteed to always be the case.  */
  .note.build-id :      { *(.note.build-id) }
  .note.GNU-stack :     { *(.note.GNU-stack) }
  .note.gnu-property :  { *(.note.gnu-property) }
  .note.ABI-tag :       { *(.note.ABI-tag) }
  .note.package :       { *(.note.package) }
  .note.dlopen :        { *(.note.dlopen) }
  .note.netbsd.ident :  { *(.note.netbsd.ident) }
  .note.openbsd.ident : { *(.note.openbsd.ident) }
  /* Start of the Read Write Data region.  */
  /* Adjust the address for the data segment.  We want to adjust up to
     the same address within the page on the next page up.  */
  . = DATA_SEGMENT_ALIGN (CONSTANT (MAXPAGESIZE), CONSTANT (COMMONPAGESIZE));
  /* Exception handling.  */
  .eh_frame       : ONLY_IF_RW { KEEP (*(.eh_frame)) *(.eh_frame.*) }
  .sframe         : ONLY_IF_RW { *(.sframe) *(.sframe.*) }
  .gnu_extab      : ONLY_IF_RW { *(.gnu_extab) }
  .gcc_except_table   : ONLY_IF_RW { *(.gcc_except_table .gcc_except_table.*) }
  .exception_ranges   : ONLY_IF_RW { *(.exception_ranges*) }
  /* Thread Local Storage sections.  */
  .tdata	  :
   {
     PROVIDE_HIDDEN (__tdata_start = .);
     *(.tdata .tdata.* .gnu.linkonce.td.*)
   }
  .tbss		  : { *(.tbss .tbss.* .gnu.linkonce.tb.*) *(.tcommon) }
  .preinit_array    :
  {
    PROVIDE_HIDDEN (__preinit_array_start = .);
    KEEP (*(.preinit_array))
    PROVIDE_HIDDEN (__preinit_array_end = .);
  }
  .init_array    :
  {
    PROVIDE_HIDDEN (__init_array_start = .);
    KEEP (*(SORT_BY_INIT_PRIORITY(.init_array.*) SORT_BY_INIT_PRIORITY(.ctors.*)))
    KEEP (*(.init_array EXCLUDE_FILE (*crtbegin.o *crtbegin?.o *crtend.o *crtend?.o ) .ctors))
    PROVIDE_HIDDEN (__init_array_end = .);
  }
  .fini_array    :
  {
    PROVIDE_HIDDEN (__fini_array_start = .);
    KEEP (*(SORT_BY_INIT_PRIORITY(.fini_array.*) SORT_BY_INIT_PRIORITY(.dtors.*)))
    KEEP (*(.fini_array EXCLUDE_FILE (*crtbegin.o *crtbegin?.o *crtend.o *crtend?.o ) .dtors))
    PROVIDE_HIDDEN (__fini_array_end = .);
  }
  .ctors          :
  {
    /* gcc uses crtbegin.o to find the start of
       the constructors, so we make sure it is
       first.  Because this is a wildcard, it
       doesn't matter if the user does not
       actually link against crtbegin.o; the
       linker won't look for a file to match a
       wildcard.  The wildcard also means that it
       doesn't matter which directory crtbegin.o
       is in.  */
    KEEP (*crtbegin.o(.ctors))
    KEEP (*crtbegin?.o(.ctors))
    /* We don't want to include the .ctor section from
       the crtend.o file until after the sorted ctors.
       The .ctor section from the crtend file contains the
       end of ctors marker and it must be last */
    KEEP (*(EXCLUDE_FILE (*crtend.o *crtend?.o ) .ctors))
    KEEP (*(SORT(.ctors.*)))
    KEEP (*(.ctors))
  }
  .dtors          :
  {
    KEEP (*crtbegin.o(.dtors))
    KEEP (*crtbegin?.o(.dtors))
    KEEP (*(EXCLUDE_FILE (*crtend.o *crtend?.o ) .dtors))
    KEEP (*(SORT(.dtors.*)))
    KEEP (*(.dtors))
  }
  .jcr            : { KEEP (*(.jcr)) }
  .data.rel.ro : { *(.data.rel.ro.local* .gnu.linkonce.d.rel.ro.local.*) *(.data.rel.ro .data.rel.ro.* .gnu.linkonce.d.rel.ro.*) }
  .dynamic        : { *(.dynamic) }
  .got            : { *(.got) *(.igot) }
  . = DATA_SEGMENT_RELRO_END (SIZEOF (.got.plt) >= 16 ? 16 : 0, .);
  .got.plt        : { *(.got.plt) *(.igot.plt) }
  .data           :
  {
    *(.data .data.* .gnu.linkonce.d.*)
    SORT(CONSTRUCTORS)
  }
  .data1          : { *(.data1) }
  /* We want the small data sections together, so single-instruction offsets
     can access them all, and initialized data all before uninitialized, so
     we can shorten the on-disk segment size.  */
  .sdata          :
  {
    *(.sdata .sdata.* .gnu.linkonce.s.*)
  }
  _edata = .;
  PROVIDE (edata = .);
  . = ALIGN(ALIGNOF(NEXT_SECTION));
  __bss_start = .;
  .sbss           :
  {
    *(.dynsbss)
    *(.sbss .sbss.* .gnu.linkonce.sb.*)
    *(.scommon)
  }
  .bss            :
  {
    *(.dynbss)
    *(.bss .bss.* .gnu.linkonce.b.*)
    *(COMMON)
    /* Align here to ensure that in the common case of there only being one
       type of .bss section, the section occupies space up to _end.
       Align after .bss to ensure correct alignment even if the
       .bss section disappears because there are no input sections.
       FIXME: Why do we need it? When there is no .bss section, we do not
       pad the .data section.  */
      . = ALIGN(. != 0 ? 64 / 8 : 1);
  }
  . = ALIGN(64 / 8);
  /* Start of the Large Data region.  */
  . = SEGMENT_START("ldata-segment", .);
  . = ALIGN(64 / 8);
  _end = .;
  PROVIDE (end = .);
  . = DATA_SEGMENT_END (.);
  /* Start of the Tiny Data region.  */
  /* Stabs debugging sections.  */
  .stab          0 : { *(.stab) }
  .stabstr       0 : { *(.stabstr) }
  .stab.excl     0 : { *(.stab.excl) }
  .stab.exclstr  0 : { *(.stab.exclstr) }
  .stab.index    0 : { *(.stab.index) }
  .stab.indexstr 0 : { *(.stab.indexstr) }
  .comment 0 (INFO) : { *(.comment); LINKER_VERSION; }
  .gnu.build.attributes : { *(.gnu.build.attributes .gnu.build.attributes.*) }
  /* DWARF debug sections.
     Symbols in the DWARF debugging sections are relative to the beginning
     of the section so we begin them at 0.  */
  /* DWARF 1.  */
  .debug          0 : { *(.debug) }
  .line           0 : { *(.line) }
  /* GNU DWARF 1 extensions.  */
  .debug_srcinfo  0 : { *(.debug_srcinfo) }
  .debug_sfnames  0 : { *(.debug_sfnames) }
  /* DWARF 1.1 and DWARF 2.  */
  .debug_aranges  0 : { *(.debug_aranges) }
  .debug_pubnames 0 : { *(.debug_pubnames) }
  /* DWARF 2.  */
  .debug_info     0 : { *(.debug_info .gnu.linkonce.wi.*) }
  .debug_abbrev   0 : { *(.debug_abbrev) }
  .debug_line     0 : { *(.debug_line .debug_line.* .debug_line_end) }
  .debug_frame    0 : { *(.debug_frame) }
  .debug_str      0 : { *(.debug_str) }
  .debug_loc      0 : { *(.debug_loc) }
  .debug_macinfo  0 : { *(.debug_macinfo) }
  /* SGI/MIPS DWARF 2 extensions.  */
  .debug_weaknames 0 : { *(.debug_weaknames) }
  .debug_funcnames 0 : { *(.debug_funcnames) }
  .debug_typenames 0 : { *(.debug_typenames) }
  .debug_varnames  0 : { *(.debug_varnames) }
  /* DWARF 3.  */
  .debug_pubtypes 0 : { *(.debug_pubtypes) }
  .debug_ranges   0 : { *(.debug_ranges) }
  /* DWARF 5.  */
  .debug_addr     0 : { *(.debug_addr) }
  .debug_line_str 0 : { *(.debug_line_str) }
  .debug_loclists 0 : { *(.debug_loclists) }
  .debug_macro    0 : { *(.debug_macro) }
  .debug_names    0 : { *(.debug_names) }
  .debug_rnglists 0 : { *(.debug_rnglists) }
  .debug_str_offsets 0 : { *(.debug_str_offsets) }
  .debug_sup      0 : { *(.debug_sup) }
  .gnu.attributes 0 : { KEEP (*(.gnu.attributes)) }
  /DISCARD/ : { *(.note.GNU-stack) *(.gnu_debuglink) *(.gnu.lto_*) *(.gnu_object_only) }
}

```

1. 链接脚本中的注释使用``/* 注释 */``来指定。

```text
/* Script for -z combreloc */
/* Copyright (C) 2014-2025 Free Software Foundation, Inc.
   Copying and distribution of this script, with or without modification,
   are permitted in any medium without royalty provided the copyright
   notice and this notice are preserved.  */
```

-------------------
2. OUTPUT_FORMAT
```
OUTPUT_FORMAT(bfdname)
OUTPUT_FORMAT(default, big, little)
```

OUTPUT_FORMAT指明输出可执行文件的格式。

- 当只有一个参数时，输出目标对象格式。

其输入的参数bfdname，可以使用一下命令查询：
```bash
loongarch64-linux-gnu-objdump -i
```
输出结果如下所示：
```text
BFD header file version (GNU Binutils) 2.43
elf64-loongarch
 (header little endian, data little endian)
  Loongarch64
elf32-loongarch
 (header little endian, data little endian)
  Loongarch64
pei-loongarch64
 (header little endian, data little endian)
  Loongarch64
elf64-little
 (header little endian, data little endian)
  Loongarch64
elf64-big
 (header big endian, data big endian)
  Loongarch64
elf32-little
 (header little endian, data little endian)
  Loongarch64
elf32-big
 (header big endian, data big endian)
  Loongarch64
srec
 (header endianness unknown, data endianness unknown)
  Loongarch64
symbolsrec
 (header endianness unknown, data endianness unknown)
  Loongarch64
verilog
 (header endianness unknown, data endianness unknown)
  Loongarch64
tekhex
 (header endianness unknown, data endianness unknown)
  Loongarch64
binary
 (header endianness unknown, data endianness unknown)
  Loongarch64
ihex
 (header endianness unknown, data endianness unknown)
  Loongarch64
plugin
 (header little endian, data little endian)

            elf64-loongarch elf32-loongarch pei-loongarch64 elf64-little 
Loongarch64 elf64-loongarch elf32-loongarch pei-loongarch64 elf64-little

            elf64-big elf32-little elf32-big srec symbolsrec verilog tekhex 
Loongarch64 elf64-big elf32-little elf32-big srec symbolsrec verilog tekhex

            binary ihex plugin 
Loongarch64 binary ihex ------
```

- 当有三个参数时，``OUTPUT_FORMAT(default, big, little)``用法如下：

如果传递给ld链接器的参数中没有``-EL``和``-EB``时，输入格式就是默认default

如果传入的参数中有``-EB``时，输出格式为big格式

如果传入的参数中有``-EL``时，输出格式为little格式

例如：

```
OUTPUT_FORMAT("elf64-loongarch", "elf64-loongarch", "elf64-loongarch")
```
所有的输出不管传入参数是什么，都输出elf64-loongarch格式，即小端格式：``(header little endian, data little endian)``


```
OUTPUT_FORMAT(elf32-bigmips, elf32-bigmips, elf32-littlemips)
```
表面默认输出时大端模式，如果参数包含``-EB``输出 大端格式elf32-bigmips，如果参数包含``-EL``输出小端格式elf32-littlemips。

-------------------
3. OUTPUT_ARCH

```
OUTPUT_ARCH(bfdarch)
```
表明输出文件的架构类型。


-------------------
4. ENTRY

```
ENTRY(symbol)
```

ENTRY指定可执行文件的入口地址，其输入是个符号。


-------------------
5. SEARCH_DIR

```
SEARCH_DIR(path)
```
SEARCH_DIR给ld链接器指令搜索库文件的路劲，类似于命令行输入``-L path``. SEARCH_DIR可以有过个。

例如：

```text
SEARCH_DIR("=/loongarch64-linux-musl/lib64"); SEARCH_DIR("=/usr/local/lib64"); SEARCH_DIR("=/lib64"); SEARCH_DIR("=/usr/lib64"); SEARCH_DIR("=/loongarch64-linux-musl/lib"); SEARCH_DIR("=/usr/local/lib"); SEARCH_DIR("=/lib"); SEARCH_DIR("=/usr/lib");
```


6. PROVIDE

In some cases, it is desirable for a linker script to define a symbol only if it is referenced and is not defined by any object included in the link. For example, traditional linkers defined the symbol ‘etext’. However, ANSI C requires that the user be able to use ‘etext’ as a function name without encountering an error. The PROVIDE keyword may be used to define a symbol, such as ‘etext’, only if it is referenced but not defined. The syntax is PROVIDE(symbol = expression).

Here is an example of using PROVIDE to define ‘etext’:
```
SECTIONS
{
  .text :
    {
      *(.text)
      _etext = .;
      PROVIDE(etext = .);
    }
}
```

In this example, if the program defines ‘_etext’ (with a leading underscore), the linker will give a multiple definition diagnostic. If, on the other hand, the program defines ‘etext’ (with no leading underscore), the linker will silently use the definition in the program. If the program references ‘etext’ but does not define it, the linker will use the definition in the linker script.

Note - the PROVIDE directive considers a common symbol to be defined, even though such a symbol could be combined with the symbol that the PROVIDE would create. This is particularly important when considering constructor and destructor list symbols such as ‘__CTOR_LIST__’ as these are often defined as common symbols. 