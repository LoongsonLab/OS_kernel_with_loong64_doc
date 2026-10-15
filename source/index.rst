目录
==========================================

文档定位与适用范围
------------------

本文档讲解 LoongArch 处理器与操作系统在特权态、内存管理、中断异常、SMP、ABI、工具链和调试方面的交互。它是一份学习与工程参考资料，**不替代** LoongArch ISA/ABI 规范、芯片用户手册、固件规范或 Linux 内核源码。

.. list-table:: 适用范围
   :widths: 22 78
   :header-rows: 1

   * - 项目
     - 范围与边界
   * - 架构
     - 以 LA64 操作系统场景为主。LA32 的 ABI、工具链和 ISA 基础内容仅在相关章节说明；本文不提供完整的 LA32 内核移植指南。
   * - ISA 基线
     - 以《龙芯架构参考手册 卷一：基础架构》r1p11 为主要参照。引用较早手册、特定扩展或特定实现时，正文应以所在章节注明的版本为准。
   * - Linux 基线
     - 内核构建、特权态与平台示例主要以 Linux v6.10 为基线；ABI“新旧世界”一节的部分源码链接使用 Linux v6.6。不同内核版本、配置和发行版的实现可能不同。
   * - 固件与启动环境
     - 覆盖 QEMU ``virt`` 机器配合 EDK2/UEFI 的实验环境，以及采用 UEFI（含新旧世界/BPI 差异）的 LoongArch 平台。具体板卡还必须以其用户手册、设备树或 ACPI 表和固件版本为准。

阅读约定
----------

* **【规范性依据】**：指向指定版本的 LoongArch ISA/ABI 规范、芯片用户手册或其他一手规范的内容。发生冲突时，以被引用的规范及其版本为准。
* **【说明性内容】**：Linux 内核、编译器、QEMU、固件、发行版、反汇编输出和命令示例。这些内容用于解释或复现，受版本、配置、链接布局和硬件实现影响，不应推导为所有 LoongArch 系统的架构强制行为。
* 尚未标出上述标签的历史内容，读者应先依据章节引用和版本信息判断其性质；后续修订将逐章补齐标签与版本基线。

学习路径
--------

* **操作系统移植与内核开发**：准备工作 → 特权态指令 → 内存管理 → 中断与异常系统 → SMP 多核系统 → 平台相关 → 调试方法与技巧。
* **用户态、二进制与工具链开发**：准备工作 → 工具链 → 程序二进制接口 → 调试方法与技巧。需要理解启动链或动态加载时，再阅读 ABI 中的 ELF 加载和新旧世界内容。
* **进阶主题**：在掌握前两条路径后阅读 SIMD 与向量指令、LVZ 与虚拟化扩展。二者目前仍在补充中，不能作为完整实现指南。

.. toctree::
   :maxdepth: 3
   :glob:
   :numbered:

   context/preparation/index.rst
   context/toolchains/index.rst
   context/privilege_isa/index.rst
   context/mmu/index.rst
   context/trap/index.rst
   context/smp/index.rst
   context/abi/index.rst
   context/debug/index.rst
   context/platform/index.rst
   context/simd/index.rst
   context/virt/index.rst
   context/faq/index.rst
