# 前言

1. 为什么有这个仓库

我们旨在为参与龙芯架构生态的开发者，提供基本的技术指南。

该文档内容包括：基础指令手册、扩展指令说明、开发工具链、调试接口、模拟器。

2. 主要是针对的用户

该文档面向所有龙芯架构生态的参与者与爱好者。

3. 本文的主要内容是什么

该文档的章节包括：

- 工具链
- 特权态指令
- 内存管理
- 中断与异常系统
- SMP多核系统
- 程序二进制接口
- 调试方法与技巧
- 平台相关
- SIMD与向量指令
- LVZ与虚拟化扩展


主要的维护人信息：

:::{list-table} 
:widths: 20 30
:header-rows: 0

*	- 李亚伟
	- liyawei@loongson.cn
*	- 王宇吉
	- wangyuji@loongson.cn
:::

内容报错，与意见反馈，可通过以下方式进行。

- Github仓库提Issue。
- 邮件反馈。
- 技术交流群。


## 更新说明
跟新的内容与上次比较
按照版本号：主版本号.次版本号.修订版本号，比如 1.1.1020，建议修订版本号以时间为准

0.1.0107：
	- 增加了...
	- 增加了...

## 仓库地址

一些常用的仓库地址列出，以及说明:

1、Linux内核以及发行版仓库地址。

-	[linux kernel](https://github.com/torvalds/linux.git):开源操作系统
-	[Yongbao](https://github.com/sunhaiyong1978/Yongbao.git):开源操作系统

2、工具链仓库地址。
-	[Cross_tools](https://github.com/loongson/build-tools.git):交叉编译器。

3、模拟器仓库地址。

-	[QEMU](https://github.com/qemu/qemu.git):开源多平台虚拟机软件。
-	[LA_EMU](https://github.com/Open-ChipHub/LA_EMU.git):LoongArch64模拟器。

4、手册与文档地址。

-	[龙芯架构参考手册卷一](https://www.loongson.cn/uploads/images/2023041918133323805.%E9%BE%99%E8%8A%AF%E6%9E%B6%E6%9E%84%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C%E5%8D%B7%E4%B8%80_r1p03.pdf):基础指令参考手册