==========
准备工作
==========

本章解释了如何理解该文档，并提供 LoongArch 生态下，基本开发环境的操作指南，便于初步上手操作。       
主要包含以下内容:

* 文档主旨和立意。介绍了文档包含了哪些内容，会对读者提供哪些帮助。
* 开发工具链的简单说明。提供工具链的简单介绍，和配置教程。
* 模拟器支持。提供模拟器的简单介绍，和安装支持。
* 发行版操作系统操作指南。提供多款 LoongArch 架构下发行版镜像使用教程。

可根据本章内容，了解文档整体结构，并通过安装模拟器，配置开发工具链，安装和启动 LoongArch 架构发行版，     
便于更快接触和了解 LoongArch 架构与操作系统。

术语速览
--------

下表给出阅读后续章节所需的核心术语。术语的精确定义、位域和行为以各章节引用的指定版本 ISA/ABI 规范为准。

.. list-table:: 核心术语
   :widths: 20 80
   :header-rows: 1

   * - 术语
     - 简要说明
   * - PLV
     - Privilege Level，特权等级。LoongArch 以 CRMD 等 CSR 中的 PLV 字段表示当前或保存的特权等级；它是访问特权资源和处理例外的基础。
   * - CSR
     - Control and Status Register，控制状态寄存器。处理器通过 CSR 暴露执行模式、例外状态、TLB、定时器等控制与状态信息。
   * - DMW
     - Direct Mapping Window，直接映射窗口。由 DMW0--DMW3 配置的虚拟地址到物理地址直接翻译窗口，常用于内核早期启动或特定内核映射。
   * - TLB refill
     - TLB 重填。当地址翻译所需表项不在 TLB 中时，硬件触发的重填处理过程；可由软件重填，具体实现也可能使用硬件页表遍历能力。
   * - PTW
     - Page Table Walk，页表遍历。依据多级页表从内存查找虚拟地址映射的过程；应与“TLB 重填例外”这一事件区分。
   * - IOCSR
     - I/O Control and Status Register，I/O 控制状态寄存器接口。通过 IOCSR 指令访问实现定义的 I/O/核间通信资源；可用资源随 SoC 而不同。
   * - IPI
     - Inter-Processor Interrupt，核间中断。一个处理器核向另一处理器核发出的中断，常用于调度、停机与 TLB shootdown。
   * - ECFG / ESTAT
     - Exception Configuration / Exception Status，例外配置与例外状态 CSR。前者配置中断使能和例外入口间距等，后者记录例外编码、挂起中断等状态。

引用与勘误
----------

文档的规范基线、适用章节和链接集中列于“前言”的“规范引用基线”表。发现技术错误、版本不一致、链接失效或示例无法复现时，请在 Issue 中提供文档路径和行号、所用硬件/模拟器、工具链与内核版本、完整命令、实际输出和预期结果；维护者会据此复现并修订。

.. toctree::
  :maxdepth: 1

  preface.md
  tools.md
  license.md


