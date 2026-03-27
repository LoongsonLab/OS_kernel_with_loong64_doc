================
程序二进制接口
================

本章节主要描述有关LoongArch的程序二进制接口（Application Binary Interface）。

学习完本章节，你会掌握基本的技能，学会如下的技能：
  - 了解LoongArch的新旧世界
  - 熟悉新世界的ABI标准
  - 熟悉通用寄存器的使用规约
  - 了解LoongArch的一些ABI变体，比如lp64s，lp64f，lp64d等
  - 子程序调用的细节
  - 函数调用的栈的排布情况
  - C语言数据类型与机器数据类型的对应
  - 有LoongArch ELF相关的内容：
  - ELF Header   
  - 重定位类型(Relocation Types)   
  - 代码模型(Code Models)  

.. toctree::
  :maxdepth: 1

  old_world.md
  new_world.md
  abi.md
  data_types.md
  elf_header.md
  elf_relocation.md
  elf_code_model.md
  elf_load.md
