# SMP的初始化

本章节还是以Linux初始化为主，其他的Kernel SMP初始化差异不大。

## CPU-0的初始化

在多核的处理器系统中，复位后只有CPU0核从复位向量地址开始执行(在LoongArch上是0x1C000000)。其他的CPU都    
处于暂停状态，不像RISC-V所有的CPU都开始从同一个地址开始执行。龙架构的CPU0负责完成主要的初始化工作，等到    
基础初始化工作完成后，CPU0会通过核间IPI中断通知其他的CPU来时执行。完成各自CPU必要的初始化工作。

注意CPU0执行的是一条初始化主线，其他CPU执行的另外的相同的初始化主线，有CPU0触发其他CPU初始化。

下面我们主要看其他CPU（逻辑处理器核）的初始化流程。

1. CPU0为唤醒其他CPU的准备工作。


```c
// File: arch/loongarch/kernel/smp.c

/*
 * Setup the PC, SP, and TP of a secondary processor and start it running!
 */
void loongson_boot_secondary(int cpu, struct task_struct *idle)
{
	unsigned long entry;

	pr_info("Booting CPU#%d...\n", cpu);

	entry = __pa_symbol((unsigned long)&smpboot_entry);
	cpuboot_data.stack = (unsigned long)__KSTK_TOS(idle);
	cpuboot_data.thread_info = (unsigned long)task_thread_info(idle);

	csr_mail_send(entry, cpu_logical_map(cpu), 0);

	loongson_send_ipi_single(cpu, ACTION_BOOT_CPU);
}
```


```c
/* Send mailbox buffer via Mail_Send */
static void csr_mail_send(uint64_t data, int cpu, int mailbox)
{
	uint64_t val;

	/* Send high 32 bits */
	val = IOCSR_MBUF_SEND_BLOCKING;
	val |= (IOCSR_MBUF_SEND_BOX_HI(mailbox) << IOCSR_MBUF_SEND_BOX_SHIFT);
	val |= (cpu << IOCSR_MBUF_SEND_CPU_SHIFT);
	val |= (data & IOCSR_MBUF_SEND_H32_MASK);
	iocsr_write64(val, LOONGARCH_IOCSR_MBUF_SEND);

	/* Send low 32 bits */
	val = IOCSR_MBUF_SEND_BLOCKING;
	val |= (IOCSR_MBUF_SEND_BOX_LO(mailbox) << IOCSR_MBUF_SEND_BOX_SHIFT);
	val |= (cpu << IOCSR_MBUF_SEND_CPU_SHIFT);
	val |= (data << IOCSR_MBUF_SEND_BUF_SHIFT);
	iocsr_write64(val, LOONGARCH_IOCSR_MBUF_SEND);
};
```

主要的准备工作：
- loongson_boot_secondary传入的参数时每个CPU的ID，和每个CPU空闲的task_struct。
- 将每个CPU的执行第一条指令的入口地址smpboot_entry，通过csr_mail_send发送到每个CPU的MailBox0（除CPU0外）。
- loongson_send_ipi_single向其他CPU发送Send_IPI中断（发送的是0），其他CPU在收到IPI_Status[0]=1后，会进行复位，从传入
  的MailBox0中获得Boot地址，也就是上面的smpboot_entry开始执行。


:::{warning}
其他的CPU发送的Send_IPI中断会一起CPU的CSR.ESTAT.IS[11]置位，进而产生异常处理流程。但是如果发生的IPI_Send的中断向量号为0     
也就是IPI_Status[0]=1的话，不会使得CPU再进入异常，而是进入复位初始化流程。
:::


2. smpboot_entry初始化。

```
SYM_CODE_START(smpboot_entry)

	SETUP_DMWINS	t0
	JUMP_VIRT_ADDR	t0, t1

#ifdef CONFIG_PAGE_SIZE_4KB
	li.d		t0, 0
	li.d		t1, CSR_STFILL
	csrxchg		t0, t1, LOONGARCH_CSR_IMPCTL1
#endif
	/* Enable PG */
	li.w		t0, 0xb0		# PLV=0, IE=0, PG=1
	csrwr		t0, LOONGARCH_CSR_CRMD
	li.w		t0, 0x04		# PLV=0, PIE=1, PWE=0
	csrwr		t0, LOONGARCH_CSR_PRMD
	li.w		t0, 0x00		# FPE=0, SXE=0, ASXE=0, BTE=0
	csrwr		t0, LOONGARCH_CSR_EUEN

	la.pcrel	t0, cpuboot_data
	ld.d		sp, t0, CPU_BOOT_STACK
	ld.d		tp, t0, CPU_BOOT_TINFO

	bl		start_secondary
	ASM_BUG()

SYM_CODE_END(smpboot_entry)
```

流程如下：
- 复位会后还是首先，建立需要的DMW直接地址映射配置窗口，和CPU一样。
- 使能映射地址翻译模式，也就是打开PG
- 通过cpuboot_data传入的内核栈和thread_info分别加载到sp和tp寄存器中。
- 跳转到start_secondary，可是执行C语言环境的初始化工作（栈已经准备完毕）。


3. 执行C语言相关的初始化start_secondary函数

```c
/*
 * First C code run on the secondary CPUs after being started up by
 * the master.
 */
asmlinkage void start_secondary(void)
{
	unsigned int cpu;

	sync_counter();
	cpu = raw_smp_processor_id();
	set_my_cpu_offset(per_cpu_offset(cpu));

	cpu_probe();
	constant_clockevent_init();
	loongson_init_secondary();

	set_cpu_sibling_map(cpu);
	set_cpu_core_map(cpu);

	notify_cpu_starting(cpu);

	/* Notify boot CPU that we're starting */
	complete(&cpu_starting);

	/* The CPU is running, now mark it online */
	set_cpu_online(cpu, true);

	calculate_cpu_foreign_map();

	/*
	 * Notify boot CPU that we're up & online and it can safely return
	 * from __cpu_up()
	 */
	complete(&cpu_running);

	/*
	 * irq will be enabled in loongson_smp_finish(), enabling it too
	 * early is dangerous.
	 */
	WARN_ON_ONCE(!irqs_disabled());
	loongson_smp_finish();

	cpu_startup_entry(CPUHP_AP_ONLINE_IDLE);
}
```

执行的步骤如下：
- cpu_probe，和CPU0一样，进行必要的初始化工作：
  - 内个CPU都有各自的cpuinfo_loongarch结构体。
  - 探测CPU执行类型，是否支持浮点，那些指令集，是指支持UAL非对齐访问，通过cpucfg读取硬件特性等等。
  - 设置异常的ECFG(和CPU0一致)，配置各自的异常寄存器，比如CSR.EENTRY, CSR.REENTRY等。 但是    
    需要注意的是，异常处理流程只初始化一次，由CPU0初始化，而其他的CPU不再初始化，初始化的只是自身    
    逻辑CPU需要的CSR相关寄存器。
  - TLB的初始化也是同样的原则，初始化相关的CSR寄存器，PGDL，PWCTL等。例外函数都是由CPU来初始化。
  - 其次是Cache的初始化，配置和Cache信息，保存在各自的current_cpu_data结构体中。


在CPU0执行```void __init init_IRQ(void)```的时候，如果打开了SMP选项的话，

会执行如下的代码: 
```c
void __init init_IRQ(void)
{

// ...
#ifdef CONFIG_SMP
	mp_ops.init_ipi();
#endif
// ...
}
```

``mp_ops.init_ipi()``最终执行的是loongson_init_ipi函数，也就是初始化IPI中断。

```c
struct smp_ops mp_ops = {
	.init_ipi		= loongson_init_ipi,
	.send_ipi_single	= loongson_send_ipi_single,
	.send_ipi_mask		= loongson_send_ipi_mask,
};
```
------------------


```c
static void loongson_init_ipi(void)
{
	int r, ipi_irq;

	ipi_irq = get_percpu_irq(INT_IPI);
	if (ipi_irq < 0)
		panic("IPI IRQ mapping failed\n");

	irq_set_percpu_devid(ipi_irq);
	r = request_percpu_irq(ipi_irq, loongson_ipi_interrupt, "IPI", &irq_stat);
	if (r < 0)
		panic("IPI IRQ request failed\n");
}
```

首先向申请了ipi_irq中断请求，然后将实际的执行中断函数``loongson_ipi_interrupt``绑定到相应的中断请求上。


```c
static irqreturn_t loongson_ipi_interrupt(int irq, void *dev)
{
	unsigned int action;
	unsigned int cpu = smp_processor_id();

	action = ipi_read_clear(cpu_logical_map(cpu));

	if (action & SMP_RESCHEDULE) {
		scheduler_ipi();
		per_cpu(irq_stat, cpu).ipi_irqs[IPI_RESCHEDULE]++;
	}

	if (action & SMP_CALL_FUNCTION) {
		generic_smp_call_function_interrupt();
		per_cpu(irq_stat, cpu).ipi_irqs[IPI_CALL_FUNCTION]++;
	}

	if (action & SMP_IRQ_WORK) {
		irq_work_run();
		per_cpu(irq_stat, cpu).ipi_irqs[IPI_IRQ_WORK]++;
	}

	if (action & SMP_CLEAR_VECTOR) {
		complete_irq_moving();
		per_cpu(irq_stat, cpu).ipi_irqs[IPI_CLEAR_VECTOR]++;
	}

	return IRQ_HANDLED;
}
```

最终是通过各自CPU的IPI_Status寄存器，来判断是哪一类的中断请求，并且执行相关的函数。

具体的是一下四类的：
```c
#define SMP_RESCHEDULE		BIT(ACTION_RESCHEDULE)
#define SMP_CALL_FUNCTION	BIT(ACTION_CALL_FUNCTION)
#define SMP_IRQ_WORK		BIT(ACTION_IRQ_WORK)
#define SMP_CLEAR_VECTOR	BIT(ACTION_CLEAR_VECTOR)
```

- SMP_RESCHEDULE: 用于发生到目标CPU进行执行流的重调度，交出当前的执行权，切换的下一个执行流。
- SMP_CALL_FUNCTION：源CPU需要目标CPU执行相应的函数SMP IPI回调。
- SMP_IRQ_WORK： 执行IRQ_Work，处理中断开启的紧急任务。
- SMP_CLEAR_VECTOR： 处理MSGIS消息中断相关内容。

-----------------------


4. 执行loongson_init_secondary函数

```c
/*
 * SMP init and finish on secondary CPUs
 */
void loongson_init_secondary(void)
{
	unsigned int cpu = smp_processor_id();
	unsigned int imask = ECFGF_IP0 | ECFGF_IP1 | ECFGF_IP2 |
			     ECFGF_IPI | ECFGF_PMC | ECFGF_TIMER | ECFGF_SIP0;

	change_csr_ecfg(ECFG0_IM, imask);

	iocsr_write32(0xffffffff, LOONGARCH_IOCSR_IPI_EN);

#ifdef CONFIG_NUMA
	numa_add_cpu(cpu);
#endif
	per_cpu(cpu_state, cpu) = CPU_ONLINE;
	cpu_data[cpu].package =
		     cpu_logical_map(cpu) / loongson_sysconf.cores_per_package;
	cpu_data[cpu].core = pptt_enabled ? cpu_data[cpu].core :
		     cpu_logical_map(cpu) % loongson_sysconf.cores_per_package;
	cpu_data[cpu].global_id = cpu_logical_map(cpu);
}
```

主要工作如下：
- 打开CSR.ECFG寄存器的IPI中断使能。
- 使能各自CPU的IPI_EN寄存器，保证其他核间的发送来的IPI都能够执行。


