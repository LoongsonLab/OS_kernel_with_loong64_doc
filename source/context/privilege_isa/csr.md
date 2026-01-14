# CSR寄存器

LoongArch架构包含的状态控制寄存器(CSR)如下。

:::{list-table} 状态控制寄存器
:widths: 15 20 15
:header-rows: 1

*   - **地址**
    - **全名**
    - **缩写名称**
*   - 0x0
    - 当前模式信息
    - CRMD
*   - 0x1
    - 例外前模式信息
    - PRMD
*   - 0x2
    - 扩展部件使能
    - EUEN
*   - 0x3
    - 杂项控制
    - MISC
*	- 0x4
	- 例外配置
	- ECFG
*	- 0x5
	- 例外状态
	- ESTAT
*	- 0x6
	- 例外返回地址
	- ERA
*	- 0x7
	- 出错虚拟地址
	- BADV
*	- 0x8
	- 出错指令
	- BADI
*	- 0xC
	- 例外入口点地址
	- EENTRY
*	- 0x10
	- TLB 索引
	- TLBIDX
*	- 0x11
	- TLB 表项高位
	- TLBEHI
*	- 0x12
	- TLB 表项低位 0
	- TLBELO0
*	- 0x13
	- TLB 表项低位 1
	- TLBELO1
*	- 0x18
	- 地址空间标识符
	- ASID
*	- 0x19
	- 低半地址空间的全局目录地址
	- PGDL
*	- 0x1A
	- 高半地址空间的全局目录地址
	- PGDH
*	- 0x1B
	- 全局目录地址
	- PGD
*	- 0x1C
	- 页面遍历控制低半部分
	- PWCL
*	- 0x1D
	- 页面遍历控制高半部分
	- PWCH
*	- 0x1E
	- STLB 页大小
	- STLBPS
*	- 0x1F
	- 缩减虚地址配置
	- RVACFG
*	- 0x20
	- CPU 标识符
	- CPUID
*	- 0x21
	- 特权资源配置信息 1
	- PRCFG1
*	- 0x22
	- 特权资源配置信息 2
	- PRCFG2
*	- 0x23
	- 特权资源配置信息 3
	- PRCFG3
*	- 0x30+n (0≤n≤15)
	- 数据保存
	- SAVEn
*	- 0x40
	- 定时器编号
	- TID
*	- 0x41
	- 定时器配置
	- TCFG
*	- 0x42
	- 定时器值
	- TVAL
*	- 0x43
	- 计时器补偿
	- CNTC
*	- 0x44
	- 定时器中断清除
	- TICLR
*	- 0x60
	- LLBit 控制
	- LLBCTL
*	- 0x80
	- 实现相关控制 1
	- IMPCTL1
*	- 0x81
	- 实现相关控制 2
	- IMPCTL2
*	- 0x88
	- TLB 重填例外入口地址
	- TLBRENTRY
*	- 0x89
	- TLB 重填例外出错虚拟地址
	- TLBRBADV
*	- 0x8A
	- TLB 重填例外返回地址
	- TLBRERA
*	- 0x8B
	- TLB 重填例外数据保存
	- TLBRSAVE
*	- 0x8C
	- TLB 重填例外表项低位 0
	- TLBRELO0
*	- 0x8D
	- TLB 重填例外表项低位 1
	- TLBRELO1
*	- 0x8E
	- TLB 重填例外表象高位
	- TLBEHI
*	- 0x8F
	- TLB 重填例外前模式信息
	- TLBRPRMD
*	- 0x90
	- 机器错误控制
	- MERRCTL
*	- 0x91
	- 机器错误信息 1
	- MERRINFO1
*	- 0x92
	- 机器错误信息 2
	- MERRINFO2
*	- 0x93
	- 机器错误例外入口地址
	- MERRENTRY
*	- 0x94
	- 机器错误例外返回地址
	- MERRERA
*	- 0x95
	- 机器错误例外数据保存
	- MERRSAVE
*	- 0x98
	- 高速缓存标签
	- CTAG
*	- 0xa0
	- 消息中断状态 0
	- MSGIS0
*	- 0xa1
	- 消息中断状态 1
	- MSGIS1
*	- 0xa2
	- 消息中断状态 2
	- MSGIS2
*	- 0xa3
	- 消息中断状态 3
	- MSGIS3
*	- 0xa4
	- 消息中断请求
	- MSGIR
*	- 0xa5
	- 消息中断使能
	- MSGIE
*	- 0x180+n (0≤n≤3)
	- 直接映射配置窗口 n
	- DMWn
*	- 0x200+2n (0≤n≤31)
	- 性能监测配置 n
	- PMCFGn
*	- 0x201+2n (0≤n≤31)
	- 性能监测计数器 n
	- PMCNTn
*	- 0x300
	- load/store 监视点整体控制
	- MWPC
*	- 0x301
	- load/store 监视点整体状态
	- MWPS
*	- 0x310+8n (0≤n≤7)
	- load/store 监视点 n 配置 1
	- MWPnCFG1
*	- 0x311+8n (0≤n≤7)
	- load/store 监视点 n 配置 2
	- MWPnCFG2
*	- 0x312+8n (0≤n≤7)
	- load/store 监视点 n 配置 3
	- MWPnCFG3
*	- 0x313+8n (0≤n≤7)
	- load/store 监视点 n 配置 4
	- MWPnCFG4
*	- 0x380
	- 取指监视点整体控制
	- FWPC
*	- 0x381
	- 取指监视点整体状态
	- FWPS
*	- 0x390+8n (0≤n≤7)
	- 取指监视点 n 配置 1
	- FWPnCFG1
*	- 0x391+8n (0≤n≤7)
	- 取指监视点 n 配置 2
	- FWPnCFG2
*	- 0x392+8n (0≤n≤7)
	- 取指监视点 n 配置 3
	- FWPnCFG3
*	- 0x393+8n (0≤n≤7)
	- 取指监视点 n 配置 4
	- FWPnCFG4
*	- 0x500
	- 调试寄存器
	- DBG
*	- 0x501
	- 调试例外返回地址
	- DERA
*	- 0x502
	- 调试数据保存
	- DSAVE
:::

`CSR`具体含义可参考《龙芯架构参考手册卷一》 7 章节。

# CSR指令

`CSR`指令用于软件访问`CSR`。

这组指令仅在`PLV0`特权等级下才能访问。

仅有一个例外情况，当`CSR.MISC`中的`RPCNTL1/RPCNTL2/RPCNTL3`配置为1时，可以在`PLV1/PLV2/PLV3`特权等级下执行`CSRRD`指令，读取性能计数器。


## CSRRD指令

指令格式:	`csrrd 	rd,csr_num`

`CSRRD`指令将指定`CSR`的值写入到通用寄存器`rd`中。

## CSRWR指令

指令格式:	`csrwr 	rd,csr_num`

`CSRWR`指令将通用寄存器`rd`中的旧值，写入到指定`CSR`中，同时，将指定`CSR`的旧值，更新到通用寄存器`rd`中。

## CSRXCHG指令

指令格式:	`csrxchg 	rd,rj,csr_num`

`CSRXCHG`指令根据通用寄存器`rd`中存放的写掩码信息，将通用寄存器`rd`中的旧值，写入到指定`CSR`中对应写掩码为1的那些比特，该`CSR`中的其余比特保持不变，同时，将指定`CSR`的旧值，更新到通用寄存器`rd`中。

所有`CSR`寄存器采用独立的寻址空间，`CSR`指令中，`CSR`的寻址值来自于指令中的14比特立即数`csr_num`，寻址单位是一个`CSR`寄存器，即 0 号`CSR`的`csr_num`是 0 ，1 号`CSR`的`csr_num`是 1 ，

所有`CSR`寄存器的位宽可能是32位宽，或者与架构中的通用寄存器`GR`等宽，因此`CSR`指令不区分位宽。
在`LA32`架构下，所有`CSR`寄存器都是32位宽。在`LA64`架构下，定义中宽度固定为 32 位的`CSR`，需要符号扩展后写入到通用寄存器`rd`中。

当`CSR`访问指令访问一个架构中未定义或硬件未实现的`CSR`时，读操作可能返回任意值(推荐返回全 0 值)，写操作不会修改处理器的任何软件可见状态。

# IOCSR指令

`IOCSR{RD/WR}.{B/H/W/D}`指令用于访问`IOCSR`。

所有`IOCSR`寄存器采用独立的寻址空间，寻址基本单位为字节。所有数据在`IOCSR`空间中采用小尾端存储格式。

`IOCSR`空间采用直接地址映射方式，物理地址直接等于逻辑地址。

`IOCSR`寄存器通常可以被多个处理器核同时访问。多个处理器核上`IOCSR`访问指令的执行满足顺序一致性条件。


## IOCSRRD指令

指令格式:	`iocsrrd.b rd,rj`
		`iocsrrd.h rd,rj`
		`iocsrrd.w rd,rj`
		`iocsrrd.d rd,rj`

`IOCSRRD.{B/H/W/D}`指令中`IOCSR`空间的指定地址处读取字节/半字/字/双字长度的数据，进行符号扩展后，写入到通用寄存器`rd`中。

`IOCSRRD.{B/H/W/D}`指令中的`IOCSR`地址来自于通用寄存器`rj`。

其中，`IOCSRRD.D`和指令只出现在`LA64`架构中。

## IOCSRWR指令

指令格式:	`iocsrwr.b rd,rj`
		`iocsrwr.h rd,rj`
		`iocsrwr.w rd,rj`
		`iocsrwr.d rd,rj`

`IOCSRWR.{B/H/W/D}`指令将通用寄存器`rd`中的[7:0]/[15:0]/[31:0]/[63:0]位数据写入到`IOCSR`空间的指定地址开始处。

`IOCSRWR.{B/H/W/D}`指令中的`IOCSR`地址来自于通用寄存器`rj`。

其中，IOCSRWR.D`指令只出现在`LA64`架构中。