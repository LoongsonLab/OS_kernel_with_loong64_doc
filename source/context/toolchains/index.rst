=======
工具链
=======

本章节主要会涉及一些常用的与LoongArch相关的工具链。这些工具链会在操作系统的设计过程中起到至关重要的作用。
因此，学习和掌握这些工具的相关技能，能够更好的理解操作系统底层的构建方式。

学习完本章节，你会掌握基本的技能，学会如下的技能：
  
  - 了解GNU GCC编译器的相关知识，包括常用的参数介绍，以及内嵌的定义等。
  - 学会如何自己动手编译基于X86_64的交叉编译器。
  - 学会如何使用readelf、objcopy和objdump等工具分析ELF文件。
  - 使用Rust编译器生成LoongArch架构的可执行文件，可以使用架构特定的参数等。
  - 熟悉链接脚本基础的知识，可能独立完成LoongArch架构的链接脚本。
  - 简单的介绍LoongArch的汇编知识，还可以了解进阶版的伪汇编指令时如何产生机器指令的。
  - 了解GLIBC和常见的musl库，以及架构相关的内容。
  - 我们会用汇编代码的例子，讲解常用的几个重要的库函数:memcpy, memchr, memset等。

.. toctree::
  :maxdepth: 1

  gcc.md
  binutils.md
  rust.md
  builtin.md
  assembly.md
  library.md
