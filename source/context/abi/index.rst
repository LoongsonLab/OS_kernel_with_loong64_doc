================
程序二进制接口
================

本章节主要描述有关LoongArch的程序二进制接口（Application Binary Interface）。

学习完本章节，你会掌握基本的技能，学会如下的技能：
  - 了解LoongArch的新旧世界
  - 熟悉新世界的ABI标准
  - 熟悉通用、浮点和向量寄存器的使用边界，以及 TLS 线程指针
  - 了解LoongArch的一些ABI变体，比如lp64s，lp64f，lp64d等
  - 子程序调用的参数、可变参数、结构体返回与栈对齐细节
  - 理解栈帧是受编译器、优化与 ABI 影响的示例布局，并使用 DWARF CFI 回溯或展开
  - 区分 C++ 异常展开与 ``setjmp``/``longjmp`` 非局部跳转
  - C语言数据类型与机器数据类型的对应
  - 有LoongArch ELF相关的内容：
  - ELF Header：LoongArch ``e_machine``、``e_flags`` 与对象 ABI
  - 重定位类型(Relocation Types)：``.rela.*``、dynamic tags、GOT/PLT、TLS 与溢出诊断
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
