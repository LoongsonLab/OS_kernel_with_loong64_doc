============
SMP多核系统
============

本章节涉及LoongArch的SMP，多核系统的内容。

学习完本章节，你会掌握基本的技能，学会如下的技能：
  - 了解LoongArch架构对于多核的实现情况
  - 熟悉目前龙架构的CPU中，关于多核相关的寄存器
  - 熟悉IPI_Status，IPI_Enable，IPI_Set，IPI_Clear，IPI_MailBox等
  - 熟悉龙架构CPU如何发送IPI中断以及如何相应
  - 熟悉龙架构CPU中IPI_Status如何将中断传递给核内中断
  - 学习Linux内核中，龙架构SMP的初始化流程
  - 学习IOCSR指令与多核通信的内容
  - 学习Linux中对于多核页表刷新的内容

我们会以代码例子来说明如何进行多核之间的通信，远程调用等。可以自己根据所学，自己手写多核之间的处理代码。

.. toctree::
  :maxdepth: 1

  smp.md
  smp_ipi.md
  smp_iocsr.md
  smp_init.md
  smp_linux_loongarch.md
  
