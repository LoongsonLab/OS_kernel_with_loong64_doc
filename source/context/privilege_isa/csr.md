# CSR寄存器

控制状态寄存器 CSR 是 CPU 内部一组特殊的、具有特定功能的寄存器，是操作系统内核与 CPU 硬件进行控制、状态监控和配置交互的核心硬件接口。

与通用寄存器不同，CSR 数量有限，并且每个 CSR 都有明确的专用功能，可以用来实现执行流控制，管理中断与异常，实现性能检测等功能。

因此，CSR 有独立的地址编号，并以此编号作为查找 CSR 的索引地址——基本寻址单位为一个 CSR 宽度。依靠不同的 CSR ，操作系统才能实现软件的隔离保护、调度并发，以及中断响应。

CSR 大致分为两类:

- 控制类。操作系统通过写入这些 CSR 来命令 CPU 改变其行为或配置。例如：写入 CSR.CRMD 的 IE 位来开启或关闭全局中断。

- 状态类。CPU 硬件通过更新这些 CSR 来报告其内部状态或事件，操作系统通过读取它们来了解情况。例如：当发生例外时，CPU 会将异常原因代码自动写入 CSR.ESTAT 的 ECode 字段，并将出错地址写入 CSR.BADV。

另外，访问 CSR 需要特殊的指令，如 LoongArch 中的 CSRWR（写）、CSRRD（读）、CSRXCHG（交换），这保证了只有拥有足够特权级（如操作系统内核）才能操作它们，是安全性的基石，文档将在 3.2 章详细介绍这些特权指令。

LoongArch架构包含的状态控制寄存器 CSR ，其具体含义如下。

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

 CSR 具体含义可参考《龙芯架构参考手册卷一》7 章节。

# CSR指令

CSR 指令用于软件访问和修改 CSR 。

这组指令仅在 PLV0 特权等级下才能访问。仅有一个例外情况，当 CSR.MISC 中的 RPCNTL1/RPCNTL2/RPCNTL3 配置为1时，可以在 PLV1/PLV2/PLV3 特权等级下执行 CSRRD 指令，读取性能计数器。

## CSRRD指令

格式: `csrrd 	rd,csr_num`

操作: 

	GPR[rd] <- CSR[csr_num]

CSRRD 指令将指定 CSR 的值写入到通用寄存器 rd 中。

其中，csr_num 是 14 比特立即数，用来描述目标 CSR 的编号，其对应关系可查看 3.1 章节对应关系表格。举例来说，当 csr_num 值为 8 时，在表格中查询地址为 8 的表格项，找到其对应名称为 BADI ，表明该指令的目标 CSR 为 BADI ，指令为获取出错指令信息。

所有 CSR 寄存器的位宽可能是 32 位宽，或者与架构中的通用寄存器 GR 等宽，因此 CSR 指令不区分位宽。
在 LA32 架构下，所有 CSR 寄存器都是 32 位宽。在 LA64 架构下，定义中宽度固定为 32 位的 CSR ，需要符号扩展后写入到通用寄存器 rd 中。

当 CSRRD 指令访问一个架构中未定义或硬件未实现的 CSR 时，操作可能返回任意值。

:::{note}
在模拟器 LA_EMU 中可查看 CSRRD 指令的模拟实现。

在`LA_EMU/loongarch64/interpreter.c`中，可以查看如下函数实现:
``` C
uint64_t helper_read_csr(CPULoongArchState *env, int csr_index) {
    uint64_t old_v = 0;
    switch (csr_index) {
        case LOONGARCH_CSR_CRMD           :old_v = env->CSR_CRMD; break;
        case LOONGARCH_CSR_PRMD           :old_v = env->CSR_PRMD; break;
        //	...
        default:fprintf(stderr, "NOT IMPLEMENTED %s %x\n", __func__, csr_index);
    }
    return old_v;
}

static bool trans_csrrd(CPULoongArchState *env, arg_csrrd *restrict a) {
    CHECK_PLV(0);
    env->gpr[a->rd] = helper_read_csr(env, a->csr);
    switch (a->csr)
    cpu_set_pc(env, env->pc + 4);
    return true;
}
```
在模拟实现 CSRRD 指令时，函数首先调用`CHECK_PLV(0)`，检查运行环境是否满足特权级 PLV0 。然后，根据 csr_num 值，调用`helper_read_csr`函数，从指定 CSR 读取数值，并写回到对应目的寄存器中。程序执行到下一条指令。
:::

:::{tip}
***在哪里会用到这条指令？***

CSRRD 指令，常见于操作系统判断应硬件的状态，一般读取状态类 CSR 。比如，读取 0x20 地址的 CSR ，可以获取 CPUID ，读取 0x5 ，可以获取 ESTAT 值，该值可以表示展示 CPU 的异常状态，提供信息，方便操作系统判断下一步处理。
:::

## CSRWR指令

格式: `csrwr 	rd,csr_num`

操作:

	temp <- GPR[rd]
	GPR[rd] <- CSR[csr_num]
	CSR[csr_num] <- temp

CSRWR 指令将通用寄存器 rd 中的旧值，写入到指定 CSR 中，同时，将指定 CSR 的旧值，更新到通用寄存器 rd 中。

其中，csr_num 是 14 比特立即数，用来描述目标 CSR 的编号，其对应关系可查看 3.1 章节对应关系表格。

所有 CSR 寄存器的位宽可能是 32 位宽，或者与架构中的通用寄存器 GR 等宽，因此 CSR 指令不区分位宽。
在 LA32 架构下，所有 CSR 寄存器都是 32 位宽。在 LA64 架构下，定义中宽度固定为 32 位的 CSR ，需要符号扩展后写入到通用寄存器 rd 中。

当 CSRWR 指令访问一个架构中未定义或硬件未实现的 CSR 时，写操作不会修改通用寄存器和 CSR 的值。

:::{note}
在模拟器 LA_EMU 中可查看 CSRWR 指令的模拟实现。
在`LA_EMU/loongarch64/interpreter.c`中，可以查看如下函数实现:
``` C
uint64_t mask_write(uint64_t old, uint64_t new, uint64_t mask) {
    return (old & ~mask) | (new & mask);
}

uint64_t helper_write_csr(CPULoongArchState *env, int csr_index, uint64_t new_v, uint64_t mask) {
    uint64_t old_v = 0;
    switch (csr_index) {
        case LOONGARCH_CSR_CRMD           :old_v = env->CSR_CRMD; env->CSR_CRMD = mask_write(env->CSR_CRMD, new_v, mask & LOONGARCH_CSR_CRMD_WMASK); break;
        case LOONGARCH_CSR_PRMD           :old_v = env->CSR_PRMD; env->CSR_PRMD = mask_write(env->CSR_PRMD, new_v, mask & LOONGARCH_CSR_PRMD_WMASK); break;
        //	...
        default:	fprintf(stderr, "NOT IMPLEMENTED %s %x\n", __func__, csr_index);
    return old_v;
}

static bool trans_csrwr(CPULoongArchState *env, arg_csrwr *restrict a) {
    CHECK_PLV(0);
    env->gpr[a->rd] = helper_write_csr(env, a->csr, env->gpr[a->rd], -1);
    cpu_set_pc(env, env->pc + 4);
    return true;
}
```
在模拟实现 CSRWR 指令时，函数首先调用`CHECK_PLV(0)`，检查运行环境是否满足特权级 PLV0 。然后，根据 csr_num 值，调用`helper_write_csr`函数，将 rd 寄存器中数值写入指定 CSR ，并将 CSR 原值写回到 rd 寄存器中。程序执行到下一条指令。

此处`mask_write`作为掩码处理函数，掩码被设置为全 1 ，即保持 rd 寄存器原值不变。

:::

:::{tip}
***为什么 CSRWR 指令，要将 CSR 的旧值，写回到寄存器 rd 中？在哪里会用到这条指令？***

CSRWR 指令，快捷实现了 CSR 值与寄存器 rd 值的交换。在操作系统进行用户进程与内核切换时，有时需要将用户栈寄存器 SP 的值到 CSR ，同时将 CSR 中保存的内核栈替换到寄存器 SP 中，可使用一条 CSRWR 指令，实现数值快速交换。
:::

## CSRXCHG指令

格式:	`csrxchg 	rd,rj,csr_num`

操作:

	tmp = GPR[rd]
	GPR[rd] = CSR[csr_num]
	CSR[csr_num] = (tmp & GPR[rj]) | (CSR[csr_num] & ~GPR[rj])


CSRXCHG 指令根据通用寄存器 rj 中存放的写掩码信息，将通用寄存器 rd 中的旧值，写入到指定 CSR 中对应写掩码为 1 的那些比特，该 CSR 中的其余比特保持不变，同时，将指定 CSR 的旧值，更新到通用寄存器 rd 中。

其中，csr_num 是 14 比特立即数，用来描述指定 CSR 的编号，其对应关系可查看 3.1 章节对应关系表格。

所有 CSR 寄存器的位宽可能是 32 位宽，或者与架构中的通用寄存器 GR 等宽，因此 CSR 指令不区分位宽。
在 LA32 架构下，所有 CSR 寄存器都是 32 位宽。在 LA64 架构下，定义中宽度固定为 32 位的 CSR ，需要符号扩展后写入到通用寄存器 rd 中。

当 CSRXCHG 指令访问一个架构中未定义或硬件未实现的 CSR 时，写操作不会修改通用寄存器和 CSR 的值。

:::{note}
在模拟器 LA_EMU 中可查看 CSRXCHG 指令的模拟实现。
在`LA_EMU/loongarch64/interpreter.c`中，可以查看如下函数实现:
``` C
uint64_t mask_write(uint64_t old, uint64_t new, uint64_t mask) {
    return (old & ~mask) | (new & mask);
}

uint64_t helper_write_csr(CPULoongArchState *env, int csr_index, uint64_t new_v, uint64_t mask) {
    uint64_t old_v = 0;
    switch (csr_index) {
        case LOONGARCH_CSR_CRMD           :old_v = env->CSR_CRMD; env->CSR_CRMD = mask_write(env->CSR_CRMD, new_v, mask & LOONGARCH_CSR_CRMD_WMASK); break;
        case LOONGARCH_CSR_PRMD           :old_v = env->CSR_PRMD; env->CSR_PRMD = mask_write(env->CSR_PRMD, new_v, mask & LOONGARCH_CSR_PRMD_WMASK); break;
        //	...
        default:	fprintf(stderr, "NOT IMPLEMENTED %s %x\n", __func__, csr_index);
    return old_v;
}

static bool trans_csrxchg(CPULoongArchState *env, arg_csrxchg *restrict a) {
    CHECK_PLV(0);
    env->gpr[a->rd] = helper_write_csr(env, a->csr, env->gpr[a->rd], env->gpr[a->rj]);
    cpu_set_pc(env, env->pc + 4);
    return true;
}
```
在模拟实现 CSRXCHG 指令时，函数首先调用`CHECK_PLV(0)`，检查运行环境是否满足特权级 PLV0 。然后，调用`mask_write`掩码处理函数，将寄存器 rd 与 rj 值进行掩码处理，计算得到要写入值。其次，根据 csr_num 值，调用`helper_write_csr`函数，将写入值写入指定 CSR ，并将 CSR 原值写回到 rd 寄存器中。程序执行到下一条指令。

此处作为掩码处理函数，掩码由寄存器 rj 决定。
:::

:::{tip}
***为什么 CSRXCHG 指令，要设置掩码写入？在哪里会用到这条指令？***

CSRXCHG 指令，快捷实现了 CSR 值的部分写入。

CSR 寻址最小单位为一个 CSR 值宽度，同时，CSR 值中可能每一位都有独特的状态与控制逻辑。

如果，操作系统只想改变某个硬件状态，同时不改变其他硬件状态，可以使用 CSRXCHG 指令实现。

例如，CSR.CRMD 包含多种硬件的状态信息，如果只想打开硬件中断使能，即使 CSR.CRMD[2] 位为 1 ，同时，其他位保持不变，可使用以下汇编指令:
``` asm
	li.d 		t0,0x4
	li.d 		t1,0x4
	csrxchg 	t0,t1,0x0
```
:::

# IOCSR指令

除了描述 CPU 内部状态信息的一组 CSR 寄存器外，还有另外一组控制状态寄存器 IOCSR，用于描述 CPU 外部、系统总线上的设备配置与控制空间。

与 CSR 类似，IOCSR 也有独立的编号，对应也有独立的寻址空间。该地址空间与 CSR 空间、内存地址空间并行。但 IOCSR 的地址空间更大，采用直接映射方式，其物理地址等于逻辑地址。同时，寻址基本单位为字节，所有数据在 IOCSR 空间中采用小尾端存储格式。

与 CSR 访问对比，作为 CPU 总线访问，IOCSR 访问的速度要慢一个量级。

IOCSR 寄存器通常可以被多个处理器核同时访问。多个处理器核上 IOCSR 访问指令的执行满足顺序一致性条件。

与 CSR 类似，访问 IOCSR 需要特殊的指令，如 LoongArch 中的 IOCSR{RD/WR}.{B/H/W/D} ，这保证了只有拥有足够特权级（如操作系统内核）才能操作它们，是安全性的基石，本章详细介绍这些特权指令。

## IOCSRRD指令

格式:	`iocsrrd.b rd,rj`
		`iocsrrd.h rd,rj`
		`iocsrrd.w rd,rj`
		`iocsrrd.d rd,rj`

操作:

	GPR[rd] = SignExtend(IOCSR[rj][ 7:0],GRLEN)
	GPR[rd] = SignExtend(IOCSR[rj][15:0],GRLEN)
	GPR[rd] = SignExtend(IOCSR[rj][31:0],GRLEN)
	GPR[rd] = IOCSR[rj][63:0]

IOCSRRD.{B/H/W/D} 指令，以寄存器 rj 中数值为指定地址，从 IOCSR 空间处读取字节/半字/字/双字长度的数据，如位宽小于 GRLEN ,则进行符号扩展后，将结果写入到寄存器 rd 中。

IOCSRRD.D 和指令只出现在 LA64 架构中。

## IOCSRWR指令

格式:	`iocsrwr.b rd,rj`
		`iocsrwr.h rd,rj`
		`iocsrwr.w rd,rj`
		`iocsrwr.d rd,rj`

操作:

	IOCSR[rj][ 7:0] = GPR[rd][ 7:0]
	IOCSR[rj][15:0] = GPR[rd][15:0]
	IOCSR[rj][31:0] = GPR[rd][31:0]
	IOCSR[rj][63:0] = GPR[rd][63:0]

IOCSRWR.{B/H/W/D} 指令将寄存器 rd 中的 [7:0]/[15:0]/[31:0]/[63:0] 位数据，以寄存器 rj 中数值为指定地址，写入到 IOCSR 空间处。

IOCSRWR.D 指令只出现在 LA64 架构中。

:::{tip}
***在哪里会用到这组指令？***

IOCSRWR 指令，可以替代原有的地址映射配置寄存器的方式，实现了操作系统对 CPU 外设的管理和控制。这样访问配置可以不经过硬件的 Load/Store 处理单元，缓解硬件流水线压力，同时保证了指令的原子性。

例如，在 3A5000 芯片中，想要读取 CORE0 扩展 IO 中断状态(如查询串口、键盘等外设的中断是否有使能)，可通过 访问 IOCSR 空间内 0x1800 地址(可查询 3A5000 芯片寄存器手册)，进行查询。
:::