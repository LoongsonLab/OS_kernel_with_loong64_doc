# ELF头 (Header)

ELF（Executable and Linkable Format）是类Unix系统中广泛使用的可执行文件、        
目标代码和共享库的标准格式。其设计目标是支持跨平台、跨架构的二进制兼容性，      
同时兼顾灵活性和高效性。[^ELF_CONTENTS]

下面我们主要看一下ELF的头部Header，下面示例是C语言的结构定义。

```c

/* 32-bit ELF base types. */
typedef __u32	Elf32_Addr;
typedef __u16	Elf32_Half;
typedef __u32	Elf32_Off;
typedef __s32	Elf32_Sword;
typedef __u32	Elf32_Word;

/* 64-bit ELF base types. */
typedef __u64	Elf64_Addr;
typedef __u16	Elf64_Half;
typedef __s16	Elf64_SHalf;
typedef __u64	Elf64_Off;
typedef __s32	Elf64_Sword;
typedef __u32	Elf64_Word;
typedef __u64	Elf64_Xword;
typedef __s64	Elf64_Sxword;

#define EI_NIDENT 16

typedef struct {
        unsigned char   e_ident[EI_NIDENT];
        Elf32_Half      e_type;
        Elf32_Half      e_machine;
        Elf32_Word      e_version;
        Elf32_Addr      e_entry;
        Elf32_Off       e_phoff;
        Elf32_Off       e_shoff;
        Elf32_Word      e_flags;
        Elf32_Half      e_ehsize;
        Elf32_Half      e_phentsize;
        Elf32_Half      e_phnum;
        Elf32_Half      e_shentsize;
        Elf32_Half      e_shnum;
        Elf32_Half      e_shstrndx;
} Elf32_Ehdr;

typedef struct {
        unsigned char   e_ident[EI_NIDENT];
        Elf64_Half      e_type;
        Elf64_Half      e_machine;
        Elf64_Word      e_version;
        Elf64_Addr      e_entry;
        Elf64_Off       e_phoff;
        Elf64_Off       e_shoff;
        Elf64_Word      e_flags;
        Elf64_Half      e_ehsize;
        Elf64_Half      e_phentsize;
        Elf64_Half      e_phnum;
        Elf64_Half      e_shentsize;
        Elf64_Half      e_shnum;
        Elf64_Half      e_shstrndx;
} Elf64_Ehdr;
```


1. e_ident
与机器无关的数据，解释文件的内容。如下所示：

|Name	|Value	|Purpose |内容|
|-|-|-|-|
|EI_MAG0	    |0	|File identification|文件标志符号，值为0x7f|
|EI_MAG1	    |1	|File identification|文件标志符号，值为'E'|
|EI_MAG2	    |2	|File identification|文件标志符号，值为'L'|
|EI_MAG3	    |3	|File identification|文件标志符号，值为'F'|
|EI_CLASS	    |4	|File class|表明文件的类别||
|EI_DATA	    |5	|Data encoding||
|EI_VERSION	    |6	|File version||
|EI_OSABI	    |7	|Operating system/ABI identification||
|EI_ABIVERSION	|8	|ABI version||
|EI_PAD	        |9	|Start of padding bytes||
|EI_NIDENT	    |16	|Size of e_ident[]||

- EI_CLASS：指的是文件类型，下面三个值，

|Name	        |Value	|Meaning|
|-|-|-|
|ELFCLASSNONE	|0	    |Invalid class|
|ELFCLASS32	    |1	    |32-bit objects|
|ELFCLASS64	    |2	    |64-bit objects|

 - ELFCLASS32： 32位架构
 - ELFCLASS64： 64位架构


2. e_type

表明当前文件的类型，如下显示：

|Name|	Value|	Meaning|
|-|-|-|
|ET_NONE	|0	    |No file type|
|ET_REL	    |1	    |Relocatable file|
|ET_EXEC	|2	    |Executable file|
|ET_DYN	    |3	    |Shared object file|
|ET_CORE	|4	    |Core file|
|ET_LOOS	|0xfe00	|Operating system-specific|
|ET_HIOS	|0xfeff	|Operating system-specific|
|ET_LOPROC	|0xff00	|Processor-specific|
|ET_HIPROC	|0xffff	|Processor-specific|

3. e_machine

表明当前ELF文件的机器架构类型。

在龙架构中，值等于**EM_LOONGARCH**，十进制:258，十六进制:0x102

4. e_version
标志此文件的版本。支持两个： 
- EV_NONE，0，表明是个无效的版本
- EV_CURRENT，1，表示当前的版本（Current）


5. e_entry
程序入口虚拟地址，如果没有，则设置成0。

6. e_phoff
这个字段表示程序头表在文件的偏移（按照字节）

7. e_shoff
这个字段表示段头表在文件的偏移（按照字节）

8. e_flags[^LoongArch_ABI2.5]
架构相关的标志。

|Bit 31-8 | Bit 7-6  |Bit 5-3     |Bit 2-0|
|-|-|-|-|
|(reserved) | ABI version|ABI extension |Base ABI Modifier|


下面我们主要看这些位的含义。

首先我们看e_flags[2:0]\(也就是Base ABI Type):

|名称|EL_CLASS类型| e_flags[2:0]的值| 描述|
|-|-|-|-|
|lp64s   | ELFCLASS64 |  0x1 | 使用**64位**通用寄存器和栈来<br>传递函数，数据模型是LP64<br> 其中long和指针都是64位，<br> 而 int类型是32位 |
|lp64f   | ELFCLASS64 |  0x2 | 使用**64位**通用寄存器，<br>以及**32位**浮点寄存器和栈来<br>传递函数，数据模型是LP64<br> 其中long和指针都是64位，<br> 而 int类型是32位 |
|lp64d   | ELFCLASS64 |  0x3 | 使用**64位**通用寄存器，<br>以及**64位**浮点寄存器和栈来<br>传递函数，数据模型是LP64<br> 其中long和指针都是64位，<br> 而 int类型是32位 |
|ilp32s  | ELFCLASS32 |  0x1 | 使用**32位**通用寄存器和栈来<br>传递函数，数据模型是ILP32<br> 其中long、指针和int类型是32位 |
|ilp32f  | ELFCLASS32 |  0x2 | 使用**32位**通用寄存器，<br>以及**32位**浮点寄存器和栈来<br>传递函数，数据模型是ILP32<br> 其中long、指针和int类型是32位 |
|ilp32d  | ELFCLASS32 |  0x3 | 使用**32位**通用寄存器，<br>以及**64位**浮点寄存器和栈来<br>传递函数，数据模型是ILP32<br> 其中long、指针和int类型是32位 | 

---------------

其中e_flags[5:3]\(就是ABI extension):

|名称  | e_flags[5:3]的值| 描述 |
|-|-|-|
|base | 0x0             | No extra ABI features.|
|     | 0x1 - 0x7       | 保留(reserved)|

---------------

其中e_flags[7:6]\(就是ABI version):

|ABI 版本| 值    | 描述|
|-|-|-|
|v0     |  0x0  | 重定位类型是基于栈式的操作的ABI|
|v1     |  0x1  | **和v0不兼容**，支持立即数直接写入立即数槽，而不是像栈式的那样进行计算|
|       | 0x2，0x3| 保留后续使用（Reserved.）|


:::{tip}
**v0**我们称之为旧世界的ABI，也叫做ABI1.0，主要的区别是重定位的计算是按照栈式的计算方式。       
是龙架构刚开始使用的程序接口。

**v1**我们称之为新世界的ABI，有时也称为ABI2.0。这两套ABI是不兼容的。
:::


9. 下面我们使用Linux的命令查看ELF的头表信息等。

```bash
bash> file main.loongarch64.elf 

# hello: ELF 64-bit LSB executable, LoongArch, version 1 (SYSV), statically linked, not stripped

```

使用``readelf -h main.loongarch64.elf``可查看ELF头信息：

```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           LoongArch
  Version:                           0x1
  Entry point address:               0x120000114
  Start of program headers:          64 (bytes into file)
  Start of section headers:          66088 (bytes into file)
  Flags:                             0x43, DOUBLE-FLOAT, OBJ-v1
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         3
  Size of section headers:           64 (bytes)
  Number of section headers:         9
  Section header string table index: 8

```

其中Flags的值是0x43，也就是： 

- e_flags[7:6] = 2'b01，使用的v1；
- e_flags[5:3] = 3'b000，默认，无扩展；
- e_flags[2:0] = 3'b011，ELFCLASS64，也就是使用lp64d ABI；




[^ELF_CONTENTS]:https://www.sco.com/developers/gabi/latest/contents.html
[^LoongArch_ABI2.5]:https://github.com/loongson/la-abi-specs


