# Kernel与异常

这里所说的异常是指包括了中断和例外，在需要区分处会特别的说明，其他情况都按照异常来说。


(expection_entry_selection)=
## 异常的入口地址选择

LoongArch的异常配置比较灵活，可满足不同的使用场景。全局的中断使能我们上节已经说明，除此之外，我们
需要明确异常的处理入口。LoongArch通过配置CSR.ECFG.VS位，来决定处理的入口偏移问题。


:::{note}
CSR.ECFG.VS：配置例外和中断入口的间距。
- 当 VS =0 时，所有例外和中断的入口地址是同一个。
- 当 VS!=0 时，各例外和中断之间的入口地址间距是 2{sup}``VS`` 条指令。

**TLB重填例外**和**机器错误**例外有**独立的入口基址**，所以二者的例外入口不受 VS域的影响。
:::

下面我们详细的说明我们常见的两种方式。

### “统一式”的入口地址

“统一式”指所有的异常和中断都是使用同一个入口地址，也就是来自CSR.EENTRY寄存器。此时需要配置
``CSR.ECFG.VS = 0``。

如何区分中断和异常：

 - 中断： 
   此时读取 CSR.ESTAT.Ecode域，如果``Ecode = 0`` 则表明是处于中断。      
   再读取CSR.ESTAT.IS[12:0]，如IS[12:0]！=0，则对应的位有了中断，然后再具体的处理对应中断。
 
 - 异常：
   读取 CSR.ESTAT.Ecode域，如果``Ecode != 0`` 则表明是处于异常，     
   此时可根据下面的例外代码确认具体的异常类型。

|Ecode|EsubCode|例外代号|例外类型|
|-|-|-|-|
|0x1||PIL|load 操作页无效例外|
|0x2||PIS|store 操作页无效例外|
|0x3||PIF|取指操作页无效例外|
|0x4||PME|页修改例外|
|0x5||PNR|页不可读例外|
|0x6||PNX|页不可执行例外|
|0x7||PPI|页特权等级不合规例外|
|0x8|0|ADEF|取指地址错例外|
|0x8|1|ADEM|访存指令地址错例外|
|0x9||ALE|地址非对齐例外|
|0xA||BCE|边界检查错例外|
|0xB||SYS|系统调用例外|
|0xC||BRK|断点例外|
|0xD||INE|指令不存在例外|
|0xE||IPE|指令特权等级错例外|
|0xF||FPD|浮点指令未使能例外|
|0x10||SXD | 128 位向量扩展指令未使能例外|
|0x11||ASXD | 256 位向量扩展指令未使能例外|
|0x12|0|FPE|基础浮点指令例外|
|0x12|1|VFPE|向量浮点指令例外|
|0x13|0|WPEF|取指监测点例外|
|0x13|1|WPEM|load/store 操作监测点例外|
|0x14||BTD|二进制翻译扩展指令未使能例外|
|0x15||BTE|二进制翻译相关例外|
|0x16||GSPR|客户机敏感特权资源例外|
|0x17||HVC|虚拟机监控调用例外|
|0x18|0|GCSC|客户机 CSR 软件修改例外|
|0x18|1|GCHC|客户机 CSR 硬件修改例外|
|0x1A-0x3E|||保留编码|  


:::{caution}
统一式异常和中断入口都来自于CSR.EENTRY，因此需要处理程序判断是来自于中断，还是来自于异常。

如果来自中断，则判断Ecode=0，再跳转到中断处理程序。

如果来自异常，则判断Ecode！=0，具体再判断Ecode和EsubCode确定是那个异常，然后再通过     
异常表，跳转到各自异常的执行程序中去处理！
:::

统一式异常和中断入口处理起来比较简单，但是对于所有的中断和异常都进入一个入口，因此需要花费
指令判断异常类型。因此中断和异常处理程序比较烦躁，要进行各自判断。


### “分离式”的入口地址

“分离式”指所有的异常和中断都有各自的入口地址，入口的基址来自CSR.EENTRY寄存器。

``计算方式为： 入口地址 = CSR.EENTRY + 页内偏移``

此时需要配置``CSR.ECFG.VS != 0``。


此时异常和中断的入口地址如下所示：

:::{note}
入口地址 = CSR.EENTRY + ecode * 2{sup}``CSR.ECFG.VS+2``

各例外和中断之间的**入口地址间距是 2{sup}``VS`` 条指令**。


VS配置的大小，具体看Kernel设计的情况。比如，Linux中，将VS=7；而有些嵌入式则设置为1，此时入口间距是2条指令。
:::

注意：

**异常Ecode的值**按照下面的方式计算:

|Ecode|EsubCode|例外代号|例外类型|
|-|-|-|-|
|0x1||PIL|load 操作页无效例外|
|0x2||PIS|store 操作页无效例外|
|0x3||PIF|取指操作页无效例外|
|0x4||PME|页修改例外|
|0x5||PNR|页不可读例外|
|0x6||PNX|页不可执行例外|
|0x7||PPI|页特权等级不合规例外|
|0x8|0|ADEF|取指地址错例外|
|0x8|1|ADEM|访存指令地址错例外|
|0x9||ALE|地址非对齐例外|
|0xA||BCE|边界检查错例外|
|0xB||SYS|系统调用例外|
|0xC||BRK|断点例外|
|0xD||INE|指令不存在例外|
|0xE||IPE|指令特权等级错例外|
|0xF||FPD|浮点指令未使能例外|
|0x10||SXD | 128 位向量扩展指令未使能例外|
|0x11||ASXD | 256 位向量扩展指令未使能例外|
|0x12|0|FPE|基础浮点指令例外|
|0x12|1|VFPE|向量浮点指令例外|
|0x13|0|WPEF|取指监测点例外|
|0x13|1|WPEM|load/store 操作监测点例外|
|0x14||BTD|二进制翻译扩展指令未使能例外|
|0x15||BTE|二进制翻译相关例外|
|0x16||GSPR|客户机敏感特权资源例外|
|0x17||HVC|虚拟机监控调用例外|
|0x18|0|GCSC|客户机 CSR 软件修改例外|
|0x18|1|GCHC|客户机 CSR 硬件修改例外|
|0x1A-0x3E|||保留编码|  

--------------

**中断Ecode的值**按照下面的方式计算:

|Ecode(十六进制)|Ecode(十进制)|例外代号|例外类型|
|-|-|-|-|
|0x40| 64 | INT_SWI0 | SWI0对应的中断（ESTAT.IS[0]）|
|0x41| 65 | INT_SWI1 | SWI1对应的中断（ESTAT.IS[1]）|
|0x42| 66 | INT_HWI0 | HWI0对应的中断（ESTAT.IS[2]）|
|0x43| 67 | INT_HWI1 | HWI1对应的中断（ESTAT.IS[3]）|
|0x44| 68 | INT_HWI2 | HWI2对应的中断（ESTAT.IS[4]）|
|0x45| 69 | INT_HWI3 | HWI3对应的中断（ESTAT.IS[5]）|
|0x46| 70 | INT_HWI4 | HWI4对应的中断（ESTAT.IS[6]）|
|0x47| 71 | INT_HWI5 | HWI5对应的中断（ESTAT.IS[7]）|
|0x48| 72 | INT_HWI6 | HWI6对应的中断（ESTAT.IS[8]）|
|0x49| 73 | INT_HWI7 | HWI7对应的中断（ESTAT.IS[9]）|
|0x4A| 74 | INT_PMI  | 性能计数器中断（ESTAT.IS[10]）|
|0x4B| 75 | INT_TI   |   Timer中断 （ESTAT.IS[11]）|
|0x4C| 76 | INT_IPI  |   核间中断   （ESTAT.IS[12]）|


## 异常的初始化

这里我们讨论上述两种方式的初始化，以及需要主要的事项。

不管是分离式还是同一的方式，中断的全局使能(CSR.CRMD.IE)、局部(CSR.ECFG.LIE)都是一样的，      
他们只是入口地址不一样而已，其他都是相同的！

下面我们以例子列举初始化的过程，以及需要注意的！

### 统一式的初始化

初始化流程如下：
 - 设置异常处理通用代码，主要是按照上面的说明，区分中断和异常，分别处理。
 - 配置``CSR.ECFG.VS = 0``，表示使用统一式的中断处理。
    :::{note}
	``CSR.ECFG.VS``默认为0，也可以不用配置，表示使用同一的入口。
	:::

 - 配置中断的相应的局部使能位：
   ```
   CSR.ECFG.LIE[12:0]对应于CSR.ESTAT.IS[12:0]，如果想要使能哪一位中断源，则将其对应的LIE[x]位置1。
   ```
 - 将异常处理程序的地址写入CSR.EENTRY。发生异常时，从这里开始执行。
 	
 	:::{warning}
 	CSR.EENTRY写入的时候，**必须按照4K页对齐**，也就是说，异常处理程序必须4K页对齐！
 	:::

 - 通过写CSR.CRMD.IE来控制全局的中断使能。



:::{warning}
**例外没有使能位控制位**，如果发生则会执行相应的代码！
:::


#### 异常处理程序的例子

下面是一个内核的异常处理过程。

```asm
// 将所有寄存器的值恢复（除SP寄存器指针）
.macro POP_GENERAL_REGS
    ld.d   $ra, $sp, 8*1
    ld.d   $tp, $sp, 8*2
    ld.d   $a0, $sp, 8*4
    ld.d   $a1, $sp, 8*5
    ld.d   $a2, $sp, 8*6
    ld.d   $a3, $sp, 8*7
    ld.d   $a4, $sp, 8*8
    ld.d   $a5, $sp, 8*9
    ld.d   $a6, $sp, 8*10
    ld.d   $a7, $sp, 8*11
    ld.d   $t0, $sp, 8*12
    ld.d   $t1, $sp, 8*13
    ld.d   $t2, $sp, 8*14
    ld.d   $t3, $sp, 8*15
    ld.d   $t4, $sp, 8*16
    ld.d   $t5, $sp, 8*17
    ld.d   $t6, $sp, 8*18
    ld.d   $t7, $sp, 8*19
    ld.d   $t8, $sp, 8*20
    ld.d   $r21,$sp, 8*21
    ld.d   $fp, $sp, 8*22
    ld.d   $s0, $sp, 8*23
    ld.d   $s1, $sp, 8*24
    ld.d   $s2, $sp, 8*25
    ld.d   $s3, $sp, 8*26
    ld.d   $s4, $sp, 8*27
    ld.d   $s5, $sp, 8*28
    ld.d   $s6, $sp, 8*29
    ld.d   $s7, $sp, 8*30
    ld.d   $s8, $sp, 8*31
.endm

// 将所有寄存器的值保存（除SP寄存器指针）
.macro PUSH_GENERAL_REGS
    st.d   $ra, $sp, 8*1
    st.d   $tp, $sp, 8*2
    st.d   $a0, $sp, 8*4
    st.d   $a1, $sp, 8*5
    st.d   $a2, $sp, 8*6
    st.d   $a3, $sp, 8*7
    st.d   $a4, $sp, 8*8
    st.d   $a5, $sp, 8*9
    st.d   $a6, $sp, 8*10
    st.d   $a7, $sp, 8*11
    st.d   $t0, $sp, 8*12
    st.d   $t1, $sp, 8*13
    st.d   $t2, $sp, 8*14
    st.d   $t3, $sp, 8*15
    st.d   $t4, $sp, 8*16
    st.d   $t5, $sp, 8*17
    st.d   $t6, $sp, 8*18
    st.d   $t7, $sp, 8*19
    st.d   $t8, $sp, 8*20
    st.d   $r21,$sp, 8*21
    st.d   $fp, $sp, 8*22
    st.d   $s0, $sp, 8*23
    st.d   $s1, $sp, 8*24
    st.d   $s2, $sp, 8*25
    st.d   $s3, $sp, 8*26
    st.d   $s4, $sp, 8*27
    st.d   $s5, $sp, 8*28
    st.d   $s6, $sp, 8*29
    st.d   $s7, $sp, 8*30
    st.d   $s8, $sp, 8*31
.endm
```

```
.section .text
.balign 4096
.global exception_entry_base
exception_entry_base:
    csrwr   $sp, KSAVE_KSP
    bnez    $sp, .Ltrap_entry

    csrrd   $sp, KSAVE_KSP
    addi.d  $sp, $sp, -{trapframe_size}

.Ltrap_entry:
    PUSH_GENERAL_REGS

    csrrd   $t0, KSAVE_KSP
    csrwr   $r0, KSAVE_KSP
    csrrd   $t1, LA_CSR_PRMD
    csrrd   $t2, LA_CSR_ERA
    st.d    $t0, $sp, 3
    st.d    $t1, $sp, 32    // prmd
    st.d    $t2, $sp, 33    // era

    move    $a0, $sp

    andi    $t1, $t1, 0x3
    bnez    $t1, .Lexit_user

    la.abs  $ra, .Ltrap_return
    b       loongarch64_trap_handler

.Lexit_user:
    ld.d    $sp, $a0, 0
    st.d    $r0, $a0, 0
    ld.d    $ra, $sp, 0
    ld.d    $tp, $sp, 1
    ld.d    $fp, $sp, 2
    ld.d    $r21,$sp, 3
    ld.d    $s0, $sp, 4
    ld.d    $s1, $sp, 5
    ld.d    $s2, $sp, 6
    ld.d    $s3, $sp, 7
    ld.d    $s4, $sp, 8
    ld.d    $s5, $sp, 9
    ld.d    $s6, $sp, 10
    ld.d    $s7, $sp, 11
    ld.d    $s8, $sp, 12

    addi.d  $sp, $sp, 14 * 8
    ret

.global enter_user
enter_user:
    addi.d  $sp, $sp, -14 * 8
    st.d    $ra, $sp, 0
    st.d    $tp, $sp, 1
    st.d    $fp, $sp, 2
    st.d    $r21,$sp, 3
    st.d    $s0, $sp, 4
    st.d    $s1, $sp, 5
    st.d    $s2, $sp, 6
    st.d    $s3, $sp, 7
    st.d    $s4, $sp, 8
    st.d    $s5, $sp, 9
    st.d    $s6, $sp, 10
    st.d    $s7, $sp, 11
    st.d    $s8, $sp, 12

    st.d    $sp, $a0, 0
    move    $sp, $a0
    csrwr   $a0, KSAVE_KSP

.Ltrap_return:
    ld.d    $t0, $sp, 32    // prmd
    ld.d    $t1, $sp, 33    // era
    csrwr   $t0, LA_CSR_PRMD
    csrwr   $t1, LA_CSR_ERA

    POP_GENERAL_REGS
    ld.d    $sp, $sp, 3

    ertn
```


### 分离式的初始化

初始化流程如下：
 - 设置异常处理通用代码，这个时候和统一方式不同的是，每个例外都有各自对应的历程，     
   而中断则一般使用一个统一的代码处理逻辑。

   比如Linux中LoongArch相关的异常例外都是这种方式组织的。

   但是也可以写成一个，多个不同的例外都使用同一个也是可以的。

 - 配置``CSR.ECFG.VS ！= 0``，表示使用分离式的中断处理。
    :::{note}
	``CSR.ECFG.VS``配置成多少，表示每个例外的可以放2{sup}``VS`` 条指令
	:::

 - 配置中断的相应的局部使能位：
   ```
   CSR.ECFG.LIE[12:0]对应于CSR.ESTAT.IS[12:0]，如果想要使能哪一位中断源，则将其对应的LIE[x]位置1。
   ```
 
 - 申请一个4K页对齐的内存空间，我们作为异常的基址EVaddr。

 - 将上面申请的EVaddr地址写入CSR.EENTRY。发生异常时，用于参与计算入口地址。计算方法按照
   :::{note}
   入口地址 = CSR.EENTRY + ecode * 2{sup}``CSR.ECFG.VS+2``
   :::
 	
 - 将我们的**例外的处理函数以及中断的处理函数指令片段**，复制到以EVaddr为Base地址相应的入口处。

   :::{warning}
   拷贝指令时，不能超过2{sup}``VS+2``个字节，如果超过有可能操作例外程序的覆盖，因此在      
   组织汇编代码的历程时，一定要控制代码指令条数。可以使用绝对跳转指令``la.abs  sym``       
   这种方式来处理。 
   :::


 - 通过写CSR.CRMD.IE来控制全局的中断使能。
 

#### 异常处理程序的例子

下面是一个Linux内核的异常处理过程。


```
SYM_CODE_START(handle_vint)
	UNWIND_HINT_UNDEFINED
	BACKUP_T0T1
	SAVE_ALL
	la_abs	t1, __arch_cpu_idle
	LONG_L	t0, sp, PT_ERA
	/* 32 byte rollback region */
	ori	t0, t0, 0x1f
	xori	t0, t0, 0x1f
	bne	t0, t1, 1f
	LONG_S	t0, sp, PT_ERA
1:	move	a0, sp
	move	a1, sp
	la_abs	t0, do_vint
	jirl	ra, t0, 0
	RESTORE_ALL_AND_RET
SYM_CODE_END(handle_vint)
```

处理中断的函数是do_vint，具体如下：

```c
asmlinkage void noinstr do_vint(struct pt_regs *regs, unsigned long sp)
{
	register int cpu;
	register unsigned long stack;
	irqentry_state_t state = irqentry_enter(regs);

	cpu = smp_processor_id();

	if (on_irq_stack(cpu, sp))
		handle_loongarch_irq(regs);
	else {
		stack = per_cpu(irq_stack, cpu) + IRQ_STACK_START;

		/* Save task's sp on IRQ stack for unwinding */
		*(unsigned long *)stack = sp;

		__asm__ __volatile__(
		"move	$s0, $sp		\n" /* Preserve sp */
		"move	$sp, %[stk]		\n" /* Switch stack */
		"move	$a0, %[regs]		\n"
		"bl	handle_loongarch_irq	\n"
		"move	$sp, $s0		\n" /* Restore sp */
		: /* No outputs */
		: [stk] "r" (stack), [regs] "r" (regs)
		: "$a0", "$a1", "$a2", "$a3", "$a4", "$a5", "$a6", "$a7", "$s0",
		  "$t0", "$t1", "$t2", "$t3", "$t4", "$t5", "$t6", "$t7", "$t8",
		  "memory");
	}

	irqentry_exit(regs, state);
}
```

-----------------

下面这是处理其他异常，比如ADE, ALE, BP, FPE, FPU, LSX等异常的流程。

```
.macro	BUILD_HANDLER exception handler prep
.align	5
	SYM_CODE_START(handle_\exception)
	UNWIND_HINT_UNDEFINED
666:
	BACKUP_T0T1
	SAVE_ALL
	build_prep_\prep
	move	a0, sp
	la_abs	t0, do_\handler
	jirl	ra, t0, 0
668:
	RESTORE_ALL_AND_RET
	SYM_CODE_END(handle_\exception)
.pushsection	".data", "aw", %progbits
	SYM_DATA(unwind_hint_\exception, .word 668b - 666b)
.popsection
.endm

	BUILD_HANDLER ade ade badv
	BUILD_HANDLER ale ale badv
	BUILD_HANDLER bce bce none
	BUILD_HANDLER bp bp none
	BUILD_HANDLER fpe fpe fcsr
	BUILD_HANDLER fpu fpu none
	BUILD_HANDLER lsx lsx none
	BUILD_HANDLER lasx lasx none
	BUILD_HANDLER lbt lbt none
	BUILD_HANDLER ri ri none
	BUILD_HANDLER watch watch none
	BUILD_HANDLER reserved reserved none	/* others */
```

---------


下面代码段时，Linux中将各个例外处理函数的拷贝过程。
```c

#define VECSIZE 0x200

static inline void setup_vint_size(unsigned int size)
{
	unsigned int vs;

	vs = ilog2(size/4);

	if (vs == 0 || vs > 7)
		panic("vint_size %d Not support yet", vs);

	csr_xchg32(vs<<CSR_ECFG_VS_SHIFT, CSR_ECFG_VS, LOONGARCH_CSR_ECFG);
}

void per_cpu_trap_init(int cpu) {

	// ......
	setup_vint_size(VECSIZE);

	configure_exception_vector();
	// ......
}

/* Install CPU exception handler */
void set_handler(unsigned long offset, void *addr, unsigned long size)
{
	memcpy((void *)(eentry + offset), addr, size);
	local_flush_icache_range(eentry + offset, eentry + offset + size);
}



/* Interrupt numbers */
#define INT_SWI0	0	/* Software Interrupts */
#define INT_SWI1	1
#define INT_HWI0	2	/* Hardware Interrupts */
#define INT_HWI1	3
#define INT_HWI2	4
#define INT_HWI3	5
#define INT_HWI4	6
#define INT_HWI5	7
#define INT_HWI6	8
#define INT_HWI7	9
#define INT_PCOV	10	/* Performance Counter Overflow */
#define INT_TI		11	/* Timer */
#define INT_IPI		12
#define INT_NMI		13
#define INT_AVEC	14

/* ExcCodes corresponding to interrupts */
#define EXCCODE_INT_NUM		(INT_AVEC + 1)
#define EXCCODE_INT_START	64
#define EXCCODE_INT_END		(EXCCODE_INT_START + EXCCODE_INT_NUM - 1)

void __init trap_init(void)
{
	long i;

	/* Set interrupt vector handler */
	for (i = EXCCODE_INT_START; i <= EXCCODE_INT_END; i++)
		set_handler(i * VECSIZE, handle_vint, VECSIZE);

	/* Set exception vector handler */
	for (i = EXCCODE_ADE; i <= EXCCODE_BTDIS; i++)
		set_handler(i * VECSIZE, exception_table[i], VECSIZE);

	cache_error_setup();

	local_flush_icache_range(eentry, eentry + 0x400);
}

```

```c
unsigned long eentry;
unsigned long tlbrentry;

long exception_handlers[VECSIZE * 128 / sizeof(long)] __aligned(SZ_64K);

static void configure_exception_vector(void)
{
	eentry    = (unsigned long)exception_handlers;
	tlbrentry = (unsigned long)exception_handlers + 80*VECSIZE;

	csr_write64(eentry, LOONGARCH_CSR_EENTRY);
	csr_write64(eentry, LOONGARCH_CSR_MERRENTRY);
	csr_write64(tlbrentry, LOONGARCH_CSR_TLBRENTRY);
}
```


TLB相关的处理历程也是在函数``setup_tlb_handler``中拷贝到了目的入口处。
```c
static void setup_tlb_handler(int cpu) {
	/* The tlb handlers are generated only once */
	if (cpu == 0) {
		memcpy((void *)tlbrentry, handle_tlb_refill, 0x80);
		local_flush_icache_range(tlbrentry, tlbrentry + 0x80);

		for (int i = EXCCODE_TLBL; i <= EXCCODE_TLBPE; i++)
			set_handler(i * VECSIZE, exception_table[i], VECSIZE);
	}
}
```


上述代码的eentry是分配的4KB对齐的内存空间，在``trap_init``初始化的时候，将各自的
处理代码通过函数``set_handler``复制到了各自的入口处。


## 异常发生时，CPU做了什么

当触发普通例外时，处理器硬件会进行如下操作：
  - ❖ 将 CSR.CRMD 的 PLV、IE 分别存到 CSR.PRMD 的 PPLV、PIE 中，然后将 CSR.CRMD 的 PLV 置       
	  为 0，IE 置为 0；
  
  - ❖ 对于支持 Watch 功能的实现，还要将 CSR.CRMD 的 WE 存到 CSR.PRMD 的 PWE 中，然后将       
      CSR.CRMD 的 WE 置为 0；
  
  - ❖ 将触发例外指令的 PC 值记录到 CSR.ERA 中；
  
  - ❖ 跳转到例外入口处取指。       
      当软件执行 ERTN 指令从普通例外执行返回时，处理器硬件会完成如下操作：
  
  - ❖ 将 CSR.PRMD 中的 PPLV、PIE 值恢复到 CSR.CRMD 的 PLV、IE 中；
  
  - ❖ 对于支持 Watch 功能的实现，还要将 CSR.PRMD 中的 PWE 值恢复到 CSR.CRMD 的 WE 中；       
  
  - ❖ 跳转到 CSR.ERA 所记录的地址处取指。      
      针对上述硬件实现，软件在例外处理过程的中途如果需要开启中断，需要保存 CSR.PRMD 中的 PPLV、      
      PIE 等信息，并在例外返回前，将所保存的信息恢复到 CSR.PRMD 中。



# 特殊异常的处理

下面我们详细介绍下，LoongArch上几个特殊的异常或者中断的使用方法。

## 定时器中断

LoongArch相关的定时器的状态控制寄存器如下所示：

**TID**: 定时器编号

处理器中每个定时器都有一个唯一可识别的编号，由软件配置在该寄存器中。每个定时器也同时唯一       
对应着一个计时器，当软件使用 RDTIME 指令读取计时器数值时，一并返回的计时器 ID 号也就是与之对应      
的定时器编号。      

|位|名字|读写|描述|
|-|-|-|-|
|31:0|TID|RW|定时器编号。软件可配置。处理器核复位期间，硬件可以将其复位成与 CSR.CPUID 中<br>CoreID 相同的值。|

---------------


**TCFG**: 定时器配置

该寄存器是软件配置定时器的接口。定时器的有效位数由实现决定，因此该寄存器中 InitVal 域的位
宽也将随之变化。    

|位|名字|读写|描述|
|-|-|-|-|
|0|En|RW|定时器使能位。仅当该位为 1 时，定时器才会进行倒计时自减，并在减为 0 值时置起<br>定时中断信号。|
|1|Periodic|RW|定时器循环模式控制位。若该位为 1，定时器在倒计时自减至 0 时，在置起定时中断<br>信号的同时，还会自动定时器重新装载成 InitVal 域中配置的初始值，然后再下一个<br>时钟周期继续自减。若该位为 0，定时器在倒计时自减至 0 时，将停止计数直至软件<br>再次配置该定时器。
|n-1:2|InitVal|RW|定时器倒计时自减计数的初始值。要求该初始值必须是 4 的整倍数。硬件将自动在该<br>域数值的最低位补上两比特 0 后再使用。|
|GRLEN-1:n|0|R|只读恒为 0，写被忽略。|

---------------


**TVAL**: 定时器数值

软件可通过读取该寄存器来获知定时器当前的计数值。定时器的有效位数由实现决定，因此该寄存器     
中 TimeVal 域的位宽也将随之变化   

|位|名字|读写|描述|
|-|-|-|-|
|n-1:0|TimeVal|R|当前定时器的计数值。|
|GRLEN-1:n|0|R|只读恒为 0，写被忽略。|

---------------


**CNTC**: 计时器补偿

软件可以通过配置该寄存器来对计时器的读出值进行修正，最终的读出值为：计时器原始计数值+计时     
器补偿值。需知配置该寄存器不可直接改变计时器的计数值。  

|位|名字|读写|描述|
|-|-|-|-|
|GRLEN-1:0|Compensation|RW|软件可配置的计时器补偿值。|

---------------


**TICLR**: 定时中断清除

软件通过对该寄存器位 0 写 1 来清除定时器置起的定时中断信号。

|位|名字|读写|描述|
|-|-|-|-|
|0|CLR|W1|当对该 bit 写值 1 时，将清除时钟中断标记。该寄存器读出结果总为 0。|

---------------

下面是Linux中，定时器的一些处理函数，比如周期的方式，或者oneshot的方式等！
```c
static irqreturn_t constant_timer_interrupt(int irq, void *data)
{
	int cpu = smp_processor_id();
	struct clock_event_device *cd;

	/* Clear Timer Interrupt */
	write_csr_tintclear(CSR_TINTCLR_TI);
	cd = &per_cpu(constant_clockevent_device, cpu);
	cd->event_handler(cd);

	return IRQ_HANDLED;
}

static int constant_set_state_oneshot(struct clock_event_device *evt)
{
	unsigned long timer_config;

	raw_spin_lock(&state_lock);

	timer_config = csr_read64(LOONGARCH_CSR_TCFG);
	timer_config |= CSR_TCFG_EN;
	timer_config &= ~CSR_TCFG_PERIOD;
	csr_write64(timer_config, LOONGARCH_CSR_TCFG);

	raw_spin_unlock(&state_lock);

	return 0;
}

static int constant_set_state_periodic(struct clock_event_device *evt)
{
	unsigned long period;
	unsigned long timer_config;

	raw_spin_lock(&state_lock);

	period = const_clock_freq / HZ;
	timer_config = period & CSR_TCFG_VAL;
	timer_config |= (CSR_TCFG_PERIOD | CSR_TCFG_EN);
	csr_write64(timer_config, LOONGARCH_CSR_TCFG);

	raw_spin_unlock(&state_lock);

	return 0;
}
```

需要注意的是，定时器中断通过向CSR.TICLR寄存器写``1``来清除定时中断信号！
```c
/* Clear Timer Interrupt */
	write_csr_tintclear(CSR_TINTCLR_TI);
```


## 系统调用

系统调用（System Call）是用户空间程序与内核空间交互的唯一合法接口，允许应用程序       
请求内核执行特权操作（如文件读写、进程控制等）。其核心作用包括：     

- 资源访问控制：用户程序无法直接操作硬件或内核数据，需通过系统调用委托内核完成。
- 抽象硬件差异：提供统一的接口（如open/read），屏蔽底层设备差异。
- 安全隔离：通过权限校验（如UID/GID）防止非法访问。

LoongArch使用``syscall 0``产生系统调用异常。


```
SYM_CODE_START(handle_sys)
	UNWIND_HINT_UNDEFINED
	la_abs	t0, handle_syscall
	jr	t0
SYM_CODE_END(handle_sys)

```

```
SYM_CODE_START(handle_syscall)
	UNWIND_HINT_UNDEFINED
	csrrd		t0, PERCPU_BASE_KS
	la.pcrel	t1, kernelsp
	add.d		t1, t1, t0
	move		t2, sp
	ld.d		sp, t1, 0

	addi.d		sp, sp, -PT_SIZE
	cfi_st		t2, PT_R3
	cfi_rel_offset	sp, PT_R3
	st.d		zero, sp, PT_R0
	csrrd		t2, LOONGARCH_CSR_PRMD
	st.d		t2, sp, PT_PRMD
	csrrd		t2, LOONGARCH_CSR_CRMD
	st.d		t2, sp, PT_CRMD
	csrrd		t2, LOONGARCH_CSR_EUEN
	st.d		t2, sp, PT_EUEN
	csrrd		t2, LOONGARCH_CSR_ECFG
	st.d		t2, sp, PT_ECFG
	csrrd		t2, LOONGARCH_CSR_ESTAT
	st.d		t2, sp, PT_ESTAT
	cfi_st		ra, PT_R1
	cfi_st		a0, PT_R4
	cfi_st		a1, PT_R5
	cfi_st		a2, PT_R6
	cfi_st		a3, PT_R7
	cfi_st		a4, PT_R8
	cfi_st		a5, PT_R9
	cfi_st		a6, PT_R10
	cfi_st		a7, PT_R11
	csrrd		ra, LOONGARCH_CSR_ERA
	st.d		ra, sp, PT_ERA
	cfi_rel_offset	ra, PT_ERA

	cfi_st		tp, PT_R2
	cfi_st		u0, PT_R21
	cfi_st		fp, PT_R22

	SAVE_STATIC
	UNWIND_HINT_REGS

#ifdef CONFIG_KGDB
	li.w		t1, CSR_CRMD_WE
	csrxchg		t1, t1, LOONGARCH_CSR_CRMD
#endif

	move		u0, t0
	li.d		tp, ~_THREAD_MASK
	and		tp, tp, sp

	move		a0, sp
	bl		do_syscall

	RESTORE_ALL_AND_RET
SYM_CODE_END(handle_syscall)
```

通过执行do_syscall函数，来处理具体的系统调用！

----------------

用户态使用下面的内敛汇编进入系统调用。

```c
#define SYSCALL_CLOBBERLIST \
	"$t0", "$t1", "$t2", "$t3", \
	"$t4", "$t5", "$t6", "$t7", "$t8", "memory"

static inline long __syscall0(long n)
{
	register long a7 __asm__("$a7") = n;
	register long a0 __asm__("$a0");

	__asm__ __volatile__ (
		"syscall 0"
		: "=r"(a0)
		: "r"(a7)
		: SYSCALL_CLOBBERLIST);
	return a0;
}

static inline long __syscall1(long n, long a)
{
	register long a7 __asm__("$a7") = n;
	register long a0 __asm__("$a0") = a;

	__asm__ __volatile__ (
		"syscall 0"
		: "+r"(a0)
		: "r"(a7)
		: SYSCALL_CLOBBERLIST);
	return a0;
}

static inline long __syscall2(long n, long a, long b)
{
	register long a7 __asm__("$a7") = n;
	register long a0 __asm__("$a0") = a;
	register long a1 __asm__("$a1") = b;

	__asm__ __volatile__ (
		"syscall 0"
		: "+r"(a0)
	        : "r"(a7), "r"(a1)
		: SYSCALL_CLOBBERLIST);
	return a0;
}
```




## 非对齐访问

1. 所有取指操作的访存地址必须 4 字节边界对齐，否则将触发取指地址错例外（ADEF）（**这个是错误，一般都是终止进程**）。

2. 除了原子访存指令、整数边界检查访存指令和浮点数边界检查访存指令外，其余的 load/store 访存指令        
可以实现为允许访存地址不对齐。（**这是架构允许的**）

不过，在一个允许访存地址不对齐的实现中，系统态软件可以通过配置        
CSR.MISC 中的 ALCL0~ALCL3 控制位，在 PLV0~PLV3 特权等级下，对这些 load/store 访存指令也进行地        
址对齐检查。对于需要进行地址对齐检查的访存指令，如果其访问的地址不是自然对齐1的，将触发地址非        
对齐例外（ALE）。

:::{note}

除了以下指令：
- 原子访存指令
- 整数边界检查访存指令
- 浮点数边界检查访存指令

其余的Load/Store指令都支持非对齐访问。手册不强制实现！

但是在以下考虑成本的芯片中，也可以不实现硬件的非对齐访问。

处理的方式为：当检查访存指令为非对齐访问时，抛出ALE地址非对齐异常，
然后使用软件模拟此条访存指令的行为。
:::

模拟非对齐访问的执行过程如下：
 - 检查load/store指令的地址是否对齐，如果不对齐则抛出ALE异常。
 - 此时当前的指令PC保存在CSR.ERA中，非对齐的地址保存在CSR.BADV中。
 - 然后进入非对齐访问的处理函数。
 - 读取当前的指令通过CSR.ERA，返回软件译码分析当前的指令INSN。
 - 然后获得当前出错指令的需要加载或者存储的字节数。
 - 如果是load操作，从内存中按照字节读出内存，然后陪凑成目标的值，然后写入到目标寄存器。
 - 如果是store操作，则将寄存器的值，按照字节负责到指定的CSR.BADV内存中。
 - 然后CSR.ERA+4，执行下一条指令。
 - 最好使用指令era返回。

更加详细的代码可访问Linux内核源码``arch/loongarch/kernel/unaligned.c``。
