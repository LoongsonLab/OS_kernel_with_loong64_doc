=======
工具链
=======

本章节主要会涉及一些常用的与LoongArch相关的工具链。这些工具链会在操作系统的设计过程中起到至关重要的作用。
因此，学习和掌握这些工具的相关技能，能够更好的理解操作系统底层的构建方式。

.. rubric:: 说明

交叉工具链不限定运行在某一种宿主机架构上；它可以运行在 x86_64、AArch64 或其他受支持的宿主机上，并为 LoongArch 生成目标程序。阅读本章时，应始终区分下面三个角色：

* **build**：正在执行 `configure`/`make` 的机器；通常与 host 相同。
* **host**：构建完成后的编译器、链接器和调试器实际运行的机器。
* **target**：编译器生成代码所运行的机器。本章的 Linux 示例通常是 `loongarch64-linux-gnu`。

GNU 三元组通常写作 ``machine-vendor-os``。例如 ``loongarch64-linux-gnu`` 中，``loongarch64`` 是目标 CPU 架构，vendor 可以省略或写作 ``unknown``，``linux-gnu`` 表示面向 Linux/GNU 用户态。三元组描述目标平台，但**不能单独完整决定 ABI**：还应通过 ``-mabi=lp64d``、``-mabi=lp64f`` 或 ``-mabi=lp64s`` 等选项选择调用约定和浮点 ABI，并确保所有对象、C 库和动态加载器一致。

下图概括从宿主机到目标进程的关系：

.. code-block:: text

   宿主机（build/host，例如 x86_64 或 AArch64）
          │  运行 loongarch64-linux-gnu-gcc / ld / gdb
          ▼
   交叉工具链 ── --sysroot ──> sysroot/
                                  ├── usr/include/   Linux UAPI 与 C 库头文件
                                  ├── usr/lib*/      libc、crt 对象、其他目标库
                                  └── lib*/          动态加载器与运行时库
          │  -mabi=…、-march=…、链接脚本、库搜索路径
          ▼
   LoongArch ELF（含 ABI 标记和 PT_INTERP）
          │  部署到目标根文件系统
          ▼
   LoongArch Linux 内核 ──> PT_INTERP 指定的动态加载器 ──> C 库与应用程序

``sysroot`` 是宿主机上模拟目标根文件系统的目录；它只用于编译和链接时查找目标头文件、启动对象和库，并不是自动部署到板卡或模拟器的根文件系统。动态链接 ELF 的 ``PT_INTERP`` 项指定运行时加载器路径（例如不同 libc/ABI 对应的 ``ld.so``）；该路径必须在**目标**根文件系统中存在，且与 ELF 的 ABI 匹配。

学习完本章节，你会掌握基本的技能，学会如下的技能：
  
  - 了解GNU GCC编译器的相关知识，包括常用的参数介绍，以及内嵌的定义等。
  - 学会如何在受支持的宿主机上自行编译面向 LoongArch 目标机的交叉编译器，并区分 build、host 与 target。
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
