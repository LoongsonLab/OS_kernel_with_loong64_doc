=============
特权态指令
=============

本章节主要涉及 LoongArch 架构下的特权态指令，主要包含以下内容:

* 状态寄存器。介绍了 LoongArch 架构下包含的所有状态寄存器
* 状态寄存器指令。介绍了 LoongArch 架构下针对状态寄存器的特权态指令
* 配置寄存器指令。介绍了 LoongArch 架构下读取 CPU 配置寄存器的特权态指令
* 缓存指令。介绍了 LoongArch 架构下针对 Cache 操作的特权态指令
* TLB 指令。介绍了 LoongArch 架构下针对 TLB 操作的特权态指令
* 页表指令。介绍了 LoongArch 架构下用于页表维护的特权态指令
* 例外与异常指令。介绍了 LoongArch 架构下处理例外和异常状态的特权态指令

文档会结合样例，解释上述特权态指令的实现原理与使用场景，加深理解。

.. toctree::
  :maxdepth: 1

  csr.md
  cacop_insn.md
  tlb_insn.md
  misc_insn.md
