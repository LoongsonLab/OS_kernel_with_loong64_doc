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
		#	由软件进行类外类型判断
		pc 	= CSR.EENTRY
	else
		#	硬件直接进行偏移跳转
		pc  = CSR.EENTRY | 2^(CSR.ECFG.VS + 2)^ x ecode

SYSCALL 指令将立即无条件触发系统调用例外。

指令的源操作数中， code 为 15 比特立即数，为系统调用例外传递参数。

系统调用例外的入口地址，采用"入口页号 | 页内偏移"的计算方式，为入口页号与页内偏移的按位或运算。

其中，系统调用例外入口的入口页号，来自 CSR.EENTRY 。

其页内偏移，由 CSR.ECFG.VS 与 code 决定。

当 CSR.ECFG.VS = 0 时，所有页内偏移相同，由软件通过 CSR.ESTAT 中的 Ecode ， IS 域的信息来判断具体的例外类型。

当 CSR.ECFG.VS != 0 时，偏移等于 2^(CSR.ECFG.VS + 2)^ x ecode，软件无需判断。此时，软件在分配例外入口基址时，需要确保所有偏移值不会超过地址边界对齐空间。

进行系统调用时，硬件也会进行如下操作:

- 将 CSR.CRMD 中的 PLV ， IE 分别保存到 CSR.PRMD ，然后置为 0；
- 将触发系统调用例外的 PC 值保存到 CSR.ERA 中；
- 跳转到系统调用例外入口，开始取指。
