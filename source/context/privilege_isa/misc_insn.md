# ERTN指令

指令格式: `ertn`

ERTN 指令用于从例外处理程序中返回。需要将例外前模式信息进行恢复，并返回到例外前对应地址。

根据例外类型，ERTN 指令有不同的操作步骤。

如果所处理的例外是一般例外，则将 CSR.PRMD 中的 PPLV ， PIE ， PWE 等信息更新到 CSR.CRMD 中，同时跳转到例外返回地址 CSR.ERA 处，开始取值。

操作:

	#	恢复现场
	CSR.CRMD.PLV 	= CSR.PRMD.PPLV
	CSR.CRMD.IE 	= CSR.PRMD.PIE
	CSR.CRMD.WE 	= CSR.PRMD.PWE
	#	指令执行流跳转
	PC 		= CSR.ERA


如果所处理的例外是 Debug 例外，将 CSR.DBG 中的 DS 位置 0 ，同时跳转到例外返回地址 CSR.DERA 处，开始取值。

操作:

	#	恢复现场
	CSR.DBG.DS 		= 0
	#	指令执行流跳转
	PC 		= CSR.DERA


如果所处理的例外是 Error 例外，则将 CSR.MERRCTL 中的 PPLV ， PIE ， PWE ， PDA ， PPG ， PDCAF ， PDCAM 等信息更新到 CSR.CRMD 中,同时跳转到例外返回地址 CSR.MERRERA 处，开始取值。

操作:

	#	恢复现场
	CSR.CRMD.PLV 	= CSR.MERRCTL.PPLV
	CSR.CRMD.IE 	= CSR.MERRCTL.PIE
	CSR.CRMD.WE 	= CSR.MERRCTL.PWE
	CSR.CRMD.DA 	= CSR.MERRCTL.PDA
	CSR.CRMD.PG 	= CSR.MERRCTL.PPG
	CSR.CRMD.DCAF 	= CSR.MERRCTL.PDCAF
	CSR.CRMD.DCAM 	= CSR.MERRCTL.PDCAM
	#	指令执行流跳转
	PC 		= CSR.MERRERA

如果所处理的例外是 TLB 重填例外，则将 CSR.TLBRSAVE 中的 PPLV ， PIE ， PWE 等信息更新到 CSR.CRMD 中，将 CSR.CRMD 的 DA 设为 0 ， PG 设为 1 ，同时跳转到例外返回地址 CSR.TLBRERA 处，开始取值。

操作:

	#	恢复现场
	CSR.CRMD.PLV 	= CSR.TLBRSAVE.PPLV
	CSR.CRMD.IE 	= CSR.TLBRSAVE.PIE
	CSR.CRMD.WE 	= CSR.TLBRSAVE.PWE
	CSR.CRMD.DA 	= 0
	CSR.CRMD.PG 	= 1
	#	指令执行流跳转
	PC 		= CSR.TLBRERA

在 RISCV 指令集中，从异常处理程序返回，需要根据特权级，区分使用`mret/eret/hret`等指令。

与之对照，LoongArch 架构下只使用`ertn`一条指令，硬件根据状态寄存器，自动完成特权级切换等相关工作，相比之下，软件设计可更加简洁。

同时，LoongArch 架构为 TLB 重填实现了专门优化，提高性能。

# IDLE指令

格式: `idle level`

IDLE 指令，可使处理器核停止取值，进入等待状态，直到被中断唤醒，或被复位。

从停止状态被中断唤醒后，处理器核执行的第一条指令，为 IDLE 指令的下一条指令。


# SysCALL指令

格式: `syscall code`

操作:

	#	保存现场
	CSR.PRMD.PPLV 	= CSR.CRMD.PLV
	CSR.PRMD.PIE	= CSR.CRMD.IE
	CSR.PRMD.PWE	= CSR.CRMD.WE
	CSR.ERA 	= PC
	#	指令执行流跳转
	if ( CSR.ECFG.VS == 0 )
		#	所有例外共用入口，由软件根据 ESTAT.ECode 判断例外类型
		pc 	= CSR.EENTRY
	else
		#	按例外编码进入分离式入口
		pc  = CSR.EENTRY + (CSR.ESTAT.ECode << (CSR.ECFG.VS + 2))

SYSCALL 指令将立即无条件触发系统调用例外。

指令编码中的 `code` 是一个 15 比特立即数字段，但它**不决定例外入口地址**，也不等同于 Linux 的系统调用号。若软件确有使用该字段的需要，例外处理程序必须从 `CSR.ERA` 指向的 `syscall` 指令自行取指并解码；硬件不会把它作为向量索引或自动写入通用的系统调用参数寄存器。

系统调用例外的入口地址，采用"入口页号 | 页内偏移"的计算方式，为入口页号与页内偏移的按位或运算。

其中，系统调用例外入口的入口页号，来自 CSR.EENTRY 。

其页内偏移由例外编码 `CSR.ESTAT.ECode` 与 `CSR.ECFG.VS` 决定，而不是由 `syscall code` 决定。对于系统调用例外，`ECode` 是硬件规定的系统调用例外编码。

当 CSR.ECFG.VS = 0 时，所有例外使用 `CSR.EENTRY` 指向的同一入口；软件通过 CSR.ESTAT 中的 `ECode`、`IS` 等字段判断具体的例外或中断类型。

当 CSR.ECFG.VS != 0 时，硬件跳转到 `CSR.EENTRY + (ECode << (VS + 2))`；等价地，偏移为 `ECode × 2^(VS + 2)` 字节。此时每个 `ECode` 有独立的入口槽位，但入口程序仍可能需要继续区分中断号等信息。软件分配入口页时，必须保证全部可能的槽位均位于 `EENTRY` 要求的对齐范围内。

在 Linux 的 LoongArch 系统调用 ABI 中，系统调用号通过通用寄存器 `$a7`（`r11`）传递，通常最多六个参数依次通过 `$a0`–`$a5`（`r4`–`r9`）传递，返回值通过 `$a0` 返回。Linux 的系统调用入口因此读取保存的通用寄存器现场，而不是使用 `syscall` 指令的 `code` 字段分派系统调用。

进行系统调用时，硬件也会进行如下操作:

- 将 CSR.CRMD 中的 PLV ， IE 分别保存到 CSR.PRMD ，然后置为 0；
- 将触发系统调用例外的 PC 值保存到 CSR.ERA 中；
- 跳转到系统调用例外入口，开始取指。

:::{tip}
有时候会发现 syscall 跳转地址不正确？

查看 CSR.EENTRY 状态寄存器，可以发现，CSR.EENTRY[11:0] 为保留位，不可写，读结果恒为 0 。

所以，当指定的系统调用例外处理函数的地址，没有对齐(末 12 位不全为 0)，写入 CSR.EENTRY 的例外处理函数地址并不正确，导致 CPU 无法正确跳转。
:::
