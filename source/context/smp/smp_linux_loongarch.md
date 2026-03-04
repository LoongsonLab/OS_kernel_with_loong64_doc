# Linux中SMP的处理（与架构相关）

本章节涉及除SMP初始化之外其他有关的一些细节问题，比如多核之间的同步，页表的处理等。

## 多核的同步

龙架构没有明确规定多核实现的技术规范。目前实现的多核CPU都支持硬件的Cache一致性协议，因此不需要手动的进行同步。     
不同的芯片可能存在细微的差异，具体可查看相应的处理器手册。



## 多核的页表处理

由于可能有多个CPU同时访问一个进程的页表，因此对于页表，需要特殊的处理，每次刷新基本都要通知所有的相关CPU。


1. 下面是单个CPU的情况下，对于页表的刷新操作。

```c
#define flush_tlb_all()			local_flush_tlb_all()
#define flush_tlb_mm(mm)		local_flush_tlb_mm(mm)
#define flush_tlb_range(vma, vmaddr, end)	local_flush_tlb_range(vma, vmaddr, end)
#define flush_tlb_kernel_range(vmaddr, end)	local_flush_tlb_kernel_range(vmaddr, end)
#define flush_tlb_page(vma, page)	local_flush_tlb_page(vma, page)
#define flush_tlb_one(vaddr)		local_flush_tlb_one(vaddr)
```

具体的实现如下：

```c
oid local_flush_tlb_all(void)
{
	invtlb_all(INVTLB_CURRENT_ALL, 0, 0);
}

void local_flush_tlb_user(void)
{
	invtlb_all(INVTLB_CURRENT_GFALSE, 0, 0);
}

void local_flush_tlb_kernel(void)
{
	invtlb_all(INVTLB_CURRENT_GTRUE, 0, 0);
}

void local_flush_tlb_one(unsigned long page)
{
	page &= (PAGE_MASK << 1);
	invtlb_addr(INVTLB_ADDR_GTRUE_OR_ASID, 0, page);
}


static __always_inline void invtlb_all(u32 op, u32 info, u64 addr)
{
	__asm__ __volatile__(
		"invtlb %0, $zero, $zero\n\t"
		:
		: "i"(op)
		: "memory"
		);
}

static __always_inline void invtlb_addr(u32 op, u32 info, u64 addr)
{
	__asm__ __volatile__(
		"invtlb %0, $zero, %1\n\t"
		:
		: "i"(op), "r"(addr)
		: "memory"
		);
}

```

其实际情况就是调用invtlb指令，进行操作。见章节[<u>页表刷新的指令</u>](#pagetable_flush_inst)内容描述。


----------------

2. 多核的页表刷新由于存在多个副本的可能性。主要还是因此各个CPU内的TLB并不会做一致性的侦听协议。   
因此需要手动的刷新。


```c
static void flush_tlb_all_ipi(void *info)
{
	local_flush_tlb_all();
}

void flush_tlb_all(void)
{
	on_each_cpu(flush_tlb_all_ipi, NULL, 1);
}

static void flush_tlb_mm_ipi(void *mm)
{
	local_flush_tlb_mm((struct mm_struct *)mm);
}

void flush_tlb_mm(struct mm_struct *mm)
{
	if (atomic_read(&mm->mm_users) == 0)
		return;		/* happens as a result of exit_mmap() */

	preempt_disable();

	if ((atomic_read(&mm->mm_users) != 1) || (current->mm != mm)) {
		on_each_cpu_mask(mm_cpumask(mm), flush_tlb_mm_ipi, mm, 1);
	} else {
		unsigned int cpu;

		for_each_online_cpu(cpu) {
			if (cpu != smp_processor_id() && cpu_context(cpu, mm))
				cpu_context(cpu, mm) = 0;
		}
		local_flush_tlb_mm(mm);
	}

	preempt_enable();
}

struct flush_tlb_data {
	struct vm_area_struct *vma;
	unsigned long addr1;
	unsigned long addr2;
};

static void flush_tlb_range_ipi(void *info)
{
	struct flush_tlb_data *fd = info;

	local_flush_tlb_range(fd->vma, fd->addr1, fd->addr2);
}

void flush_tlb_range(struct vm_area_struct *vma, unsigned long start, unsigned long end)
{
	struct mm_struct *mm = vma->vm_mm;

	preempt_disable();
	if ((atomic_read(&mm->mm_users) != 1) || (current->mm != mm)) {
		struct flush_tlb_data fd = {
			.vma = vma,
			.addr1 = start,
			.addr2 = end,
		};

		on_each_cpu_mask(mm_cpumask(mm), flush_tlb_range_ipi, &fd, 1);
	} else {
		unsigned int cpu;

		for_each_online_cpu(cpu) {
			if (cpu != smp_processor_id() && cpu_context(cpu, mm))
				cpu_context(cpu, mm) = 0;
		}
		local_flush_tlb_range(vma, start, end);
	}
	preempt_enable();
}

static void flush_tlb_kernel_range_ipi(void *info)
{
	struct flush_tlb_data *fd = info;

	local_flush_tlb_kernel_range(fd->addr1, fd->addr2);
}

void flush_tlb_kernel_range(unsigned long start, unsigned long end)
{
	struct flush_tlb_data fd = {
		.addr1 = start,
		.addr2 = end,
	};

	on_each_cpu(flush_tlb_kernel_range_ipi, &fd, 1);
}

static void flush_tlb_page_ipi(void *info)
{
	struct flush_tlb_data *fd = info;

	local_flush_tlb_page(fd->vma, fd->addr1);
}

void flush_tlb_page(struct vm_area_struct *vma, unsigned long page)
{
	preempt_disable();
	if ((atomic_read(&vma->vm_mm->mm_users) != 1) || (current->mm != vma->vm_mm)) {
		struct flush_tlb_data fd = {
			.vma = vma,
			.addr1 = page,
		};

		on_each_cpu_mask(mm_cpumask(vma->vm_mm), flush_tlb_page_ipi, &fd, 1);
	} else {
		unsigned int cpu;

		for_each_online_cpu(cpu) {
			if (cpu != smp_processor_id() && cpu_context(cpu, vma->vm_mm))
				cpu_context(cpu, vma->vm_mm) = 0;
		}
		local_flush_tlb_page(vma, page);
	}
	preempt_enable();
}
```

其中不会直接调用local_*系列的函数，而是将其作为参数传入。


我们以``flush_tlb_all``函数为例说明。实际调用的是``on_each_cpu``函数。

```c
/*
 * Call a function on all processors
 */
static inline void on_each_cpu(smp_call_func_t func, void *info, int wait)
{
	on_each_cpu_cond_mask(NULL, func, info, wait, cpu_online_mask);
}
```
on_each_cpu_cond_mask最终还是调用``smp_call_function_many_cond``函数。

```c
static void smp_call_function_many_cond(const struct cpumask *mask,
					smp_call_func_t func, void *info,
					unsigned int scf_flags,
					smp_cond_func_t cond_func)
{

//...
	if (run_remote) {
		cfd = this_cpu_ptr(&cfd_data);
		cpumask_and(cfd->cpumask, mask, cpu_online_mask);
		__cpumask_clear_cpu(this_cpu, cfd->cpumask);
		cpumask_clear(cfd->cpumask_ipi);
		for_each_cpu(cpu, cfd->cpumask) {
			call_single_data_t *csd = per_cpu_ptr(cfd->csd, cpu);
			//...
			if (wait)
				csd->node.u_flags |= CSD_TYPE_SYNC;
			csd->func = func;
			csd->info = info;
			//...
			if (llist_add(&csd->node.llist, &per_cpu(call_single_queue, cpu))) {
				__cpumask_set_cpu(cpu, cfd->cpumask_ipi);
				nr_cpus++;
				last_cpu = cpu;
			}
		}
		if (nr_cpus == 1)
			send_call_function_single_ipi(last_cpu);
		else if (likely(nr_cpus > 1))
			send_call_function_ipi_mask(cfd->cpumask_ipi);
	}

//...
	// 如果是本地的CPU，则直接调用func，不用启用远程的smp_call
	if (run_local && (!cond_func || cond_func(this_cpu, info))) {
		unsigned long flags;

		local_irq_save(flags);
		csd_do_func(func, info, NULL);
		local_irq_restore(flags);
	}

```

首先将需要执行的函数func添加到每个CPU的远程执行smp_call_function_queue队列中。    
具体可以参看函数``generic_smp_call_function_single_interrupt``。

然后通过IPI，``send_call_function_ipi_mask``通知远程的CPU执行相应的函数。


```c
static __always_inline void
send_call_function_ipi_mask(struct cpumask *mask)
{
	trace_ipi_send_cpumask(mask, _RET_IP_,
			       generic_smp_call_function_single_interrupt);
	arch_send_call_function_ipi_mask(mask);
}
```

```c
static inline void arch_send_call_function_ipi_mask(const struct cpumask *mask)
{
	mp_ops.send_ipi_mask(mask, ACTION_CALL_FUNCTION);
}
```

最终调用的则是loongson_send_ipi_mask，通过ipi_write_action通知远程CPU进行``ACTION_CALL_FUNCTION``操作。

```c
static void ipi_write_action(int cpu, u32 action)
{
	uint32_t val;

	val = IOCSR_IPI_SEND_BLOCKING | action;
	val |= (cpu << IOCSR_IPI_SEND_CPU_SHIFT);
	iocsr_write32(val, LOONGARCH_IOCSR_IPI_SEND);
}
```

```c
static void loongson_send_ipi_mask(const struct cpumask *mask, unsigned int action)
{
	unsigned int i;

	for_each_cpu(i, mask)
		ipi_write_action(cpu_logical_map(i), (u32)action);
}
```

最后每个CPU执行的都是下面的函数(local_*的函数，也就是invtlb指令)：

```c
local_flush_tlb_all()
local_flush_tlb_mm(mm)
local_flush_tlb_range(vma, vmaddr, end)
local_flush_tlb_kernel_range(vmaddr, end)
local_flush_tlb_page(vma, page)
local_flush_tlb_one(vaddr)
```

**不同于单个CPU的是，他们会通知每个在线的CPU，同时进行此刷新操作。**
