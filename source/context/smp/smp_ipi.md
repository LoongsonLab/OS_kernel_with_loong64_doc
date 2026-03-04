# IPI中断

CSR.ESTAT寄存器中的IS[12]，支持核间中断IPI，用于通知当前处理器核有其他处理器核产生了IPI中断请求。

下面我们主要以3A6000处理器为例。

3A6000 为每个处理器核都实现了 8 个核间中断寄存器（IPI）以支持多核 BIOS 启    
动和操作系统运行时在处理器核之间进行中断和通信。

3A6000支持两种不同的访问方式，一种是地址访问模式，另一种是为了支持处
理器寄存器空间的直接私有访问。


处理器核间中断相关的寄存器及其功能描述如下表所示:

|名称|读写权限 |描述|
|-|-|-|
|IPI_Status | R | 32 位状态寄存器，任何一位有被置 1 且对应位使能情况下，<br> 处理器核 INT4中断线被置位。|
|IPI_Enable| RW | 32 位使能寄存器，控制对应中断位是否有效|
|IPI_Set| W |  32 位置位寄存器，往对应的位写 1，则对应的 STATUS 寄存器位被置 1|
|IPI_Clear|W | 32 位清除寄存器，往对应的位写 1，则对应的 STATUS 寄存器位被清 0|
|MailBox0|RW | 缓存寄存器，供启动时传递参数使用，按 64 或者 32 位的 <br> uncache 方式进行访问。|
|MailBox01 | RW |缓存寄存器，供启动时传递参数使用，按 64 或者 32 位的 <br> uncache 方式进行访问。|
|MailBox02 | RW | 缓存寄存器，供启动时传递参数使用，按 64 或者 32 位的 <br> uncache 方式进行访问。|
|MailBox03 | RW | 缓存寄存器，供启动时传递参数使用，按 64 或者 32 位的 <br> uncache 方式进行访问。|


## 按地址访问模式(MMIO)

对于龙芯 3A6000，上述的寄存器可以使用基地址 0x1fe00000 进行访问，也可以使用配置    
寄存器指令（IOCSR）进行访问。具体寄存器说明和地址见下面的表格。

由于龙芯 3A6000 中包括 8 个逻辑核，连续的两个逻辑核号对应一个物理核，在龙芯3      
号空间规则下，8 个逻辑核被划分为两个结点。当使用地址进行访问时，各个内部结点的基      
地址需要在地址的[17:16]上加上内部结点号。例如，核4至核7的基地址为 0x1fe10000，      
核0至核3的基地址为 0x1fe00000。     

-------------

**0 号处理器核的核间中断与通信寄存器列表如下所示：**

|名称|偏移地址|权限|描述|
|-|-|-|-|
|Core0_IPI_Status | 0x1000 |R  | 0 号处理器核的 IPI_Status 寄存器|
|Core0_IPI_Enalbe | 0x1004 |RW | 0 号处理器核的 IPI_Enalbe 寄存器|
|Core0_IPI_Set    | 0x1008 |W  | 0 号处理器核的 IPI_Set 寄存器|
|Core0_IPI_Clear  | 0x100c |W  | 0 号处理器核的 IPI_Clear 寄存器|
|Core0_MailBox0   | 0x1020 |RW | 0 号处理器核的 IPI_MailBox0 寄存器|
|Core0_MailBox1   | 0x1028 |RW | 0 号处理器核的 IPI_MailBox1 寄存器|
|Core0_MailBox2   | 0x1030 |RW | 0 号处理器核的 IPI_MailBox2 寄存器|
|Core0_MailBox3   | 0x1038 |RW | 0 号处理器核的 IPI_MailBox3 寄存器|


-------------
**1 号处理器核的核间中断与通信寄存器列表如下所示：**

|名称|偏移地址|权限|描述|
|-|-|-|-|
|Core1_IPI_Status | 0x1100 |R  | 1 号处理器核的 IPI_Status 寄存器|
|Core1_IPI_Enalbe | 0x1104 |RW | 1 号处理器核的 IPI_Enalbe 寄存器|
|Core1_IPI_Set    | 0x1108 |W  | 1 号处理器核的 IPI_Set 寄存器|
|Core1_IPI_Clear  | 0x110c |W  | 1 号处理器核的 IPI_Clear 寄存器|
|Core1_MailBox0   | 0x1120 |RW | 1 号处理器核的 IPI_MailBox0 寄存器|
|Core1_MailBox1   | 0x1128 |RW | 1 号处理器核的 IPI_MailBox1 寄存器|
|Core1_MailBox2   | 0x1130 |RW | 1 号处理器核的 IPI_MailBox2 寄存器|
|Core1_MailBox3   | 0x1138 |RW | 1 号处理器核的 IPI_MailBox3 寄存器|



-------------
**2 号处理器核的核间中断与通信寄存器列表如下所示：**

|名称|偏移地址|权限|描述|
|-|-|-|-|
|Core2_IPI_Status | 0x1200 |R  | 2 号处理器核的 IPI_Status 寄存器|
|Core2_IPI_Enalbe | 0x1204 |RW | 2 号处理器核的 IPI_Enalbe 寄存器|
|Core2_IPI_Set    | 0x1208 |W  | 2 号处理器核的 IPI_Set 寄存器|
|Core2_IPI_Clear  | 0x120c |W  | 2 号处理器核的 IPI_Clear 寄存器|
|Core2_MailBox0   | 0x1220 |RW | 2 号处理器核的 IPI_MailBox0 寄存器|
|Core2_MailBox1   | 0x1228 |RW | 2 号处理器核的 IPI_MailBox1 寄存器|
|Core2_MailBox2   | 0x1230 |RW | 2 号处理器核的 IPI_MailBox2 寄存器|
|Core2_MailBox3   | 0x1238 |RW | 2 号处理器核的 IPI_MailBox3 寄存器|



-------------
**3 号处理器核的核间中断与通信寄存器列表如下所示：**

|名称|偏移地址|权限|描述|
|-|-|-|-|
|Core3_IPI_Status | 0x1300 |R  | 3 号处理器核的 IPI_Status 寄存器|
|Core3_IPI_Enalbe | 0x1304 |RW | 3 号处理器核的 IPI_Enalbe 寄存器|
|Core3_IPI_Set    | 0x1308 |W  | 3 号处理器核的 IPI_Set 寄存器|
|Core3_IPI_Clear  | 0x130c |W  | 3 号处理器核的 IPI_Clear 寄存器|
|Core3_MailBox0   | 0x1320 |RW | 3 号处理器核的 IPI_MailBox0 寄存器|
|Core3_MailBox1   | 0x1328 |RW | 3 号处理器核的 IPI_MailBox1 寄存器|
|Core3_MailBox2   | 0x1330 |RW | 3 号处理器核的 IPI_MailBox2 寄存器|
|Core3_MailBox3   | 0x1338 |RW | 3 号处理器核的 IPI_MailBox3 寄存器|


:::{tip}
上述的地址描述是基于统一内存地址排布的，核0至核3的基地址为 0x1fe00000，    
核4至核7的基地址为 0x1fe10000。
:::


## 按配置寄存器指令模式(IOCSR)

龙芯 3A6000 中新增了处理器核直接的寄存器访问指令，可以通过私有空间对配置
寄存器进行访问。为了更方便地使用核间中断寄存器。主要是通过IOCSR相关指令操作。

**当前处理器核核间中断与通信寄存器列表**

|名称|偏移地址|权限|描述|
|-|-|-|-|
|perCore_IPI_Status | 0x1000  | R  | 当前处理器核的 IPI_Status 寄存器  |
|perCore_IPI_Enalbe | 0x1004  | RW | 当前处理器核的 IPI_Enalbe 寄存器  |
|perCore_IPI_Set    | 0x1008  | W  | 当前处理器核的 IPI_Set 寄存器     |
|perCore_IPI_Clear  | 0x100c  | W  | 当前处理器核的 IPI_Clear 寄存器   |
|perCore_MailBox0   | 0x1020  | RW | 当前处理器核的 IPI_MailBox0 寄存器|
|perCore_MailBox1   | 0x1028  | RW | 当前处理器核的 IPI_MailBox1 寄存器|
|perCore_MailBox2   | 0x1030  | RW | 当前处理器核的 IPI_MailBox2 寄存器|
|perCore_MailBox3   | 0x1038  | RW | 当前处理器核的 IPI_MailBox3 寄存器|


为了向其它核发送核间中断请求及 MailBox 通信，通过以下寄存器进行访问。

**处理器核核间通信寄存器**

|名称|偏移地址|权限|描述|
|-|-|-|-|
|IPI_Send | 0x1040 | WO | 32 位中断分发寄存器: <br>[31] 等待完成标志，置 1 时会等待中断生效;<br>[30:26] 保留;<br>[25:16] 处理器核号;<br>[15:5] 保留;<br>[4:0] 中断向量号，对应 IPI_Status 中的向量;|
|Mail_Send | 0x1048 | WO | 64 位 MailBox 缓存寄存器: <br>[63:32] MailBox 数据;<br>[31] 等待完成标志，置 1 时会等待写入生效;<br>[30:27] 写入数据的 mask，每一位表示 32 位写数据<br>对应的字节不会真正写入目标地址，如 1000b 表示写<br>入第 0-2 字节，0000b 则 0-3 字节全部写入<br>[26] 保留<br>[25:16] 处理器核号<br>[15:5] 保留<br>[4:2] MailBox 号: <br>0 - MailBox0 低 32 位;<br>1 - MailBox0 高 32 位;<br>2 - MailBox1 低 32 位;<br>3 - MailBox1 高 32 位;<br>4 - MailBox2 低 32 位;<br>5 - MailBox2 高 32 位;<br>6 - MailBox3 低 32 位;<br>7 - MailBox4 高 32 位; <br>[1:0] 保留;|
|FREQ_Send | 0x1058 | WO |32 位频率使能寄存器;<br>[31] 等待完成标志，置 1 时会等待设置生效;<br>[30:27] 写入数据的 mask，每一位表示 32 位写数据;<br>对应的字节不会真正写入目标地址，如 1000b 表示写;<br>入第 0-2 字节，0000b 则 0-3 字节全部写入;<br>[26] 保留;<br>[25:16] 处理器核号;<br>[15:5] 保留;<br>[4:0] 写入对应的处理器核私有频率配置寄存器。<br>IOCSR[0x1050]|


:::{warning}
需要注意的是，由于 Mail_Send 寄存器一次只可以发送 32 位的数据，当发送 64 位数据    
时必须拆**分为两次发送**。因此，目标核在等待 Mail_Box 内容时，需要通过其它的软件手段    
来确保传输的完整性。例如，发送完 Mail_Box 数据之后，通过核间中断来表示已经发送完    
成。
:::




## 多核中断处理流程

1. 首先源处理器核，准备数据，发送到目标的MailBox寄存器。

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

如上所示，首先向Mail_Send写入相应的值，通过IOCSR指令，写入到地址空间中。


根据Mail_Send的定义:
- MailSend[63:32]: 存放的是发送的32位数据。
- MailSend[31]: 等待完成的标志，置1时会等待写入生效。上述例子中，IOCSR_MBUF_SEND_BLOCKING的值为``1<<31``。
- MailSend[25:16]: 是写入目的CPU的MailBox寄存器，比如为0，是写入CPU-0；值为1，就是写入CPU-1。上述
  例子中，直接将``cpu<<16``写入。
- MailSend[4:2]:是写入目标CPU的哪一个MailBox，具体的分类方法如下：

```c
#define  IOCSR_MBUF_SEND_BOX_LO(box)	(box << 1)
#define  IOCSR_MBUF_SEND_BOX_HI(box)	((box << 1) + 1)
```

对应于下面：
|MailSend[4:2]的值|对应写入的目标MailBox内容（32位）|
|- | -|
|0 | MailBox0 低 32 位|
|1 | MailBox0 高 32 位|
|2 | MailBox1 低 32 位|
|3 | MailBox1 高 32 位|
|4 | MailBox2 低 32 位|
|5 | MailBox2 高 32 位|
|6 | MailBox3 低 32 位|
|7 | MailBox4 高 32 位|

------------------


:::{tip}
上述``iocsr_write64(val, LOONGARCH_IOCSR_MBUF_SEND)``的执行结果，会根据Mail_Send值具体的定义，     
写入目标寄存器的MailBox寄存器中，也就是上面描述的perCore_MailBox0，perCore_MailBox1， perCore_MailBox2      
perCore_MailBox3。
:::

:::{warning}
上述对Mail_Send的操作，只是将内容的值写入目标CPU的MailBox中，但是并不会打断目标CPU的当前执行的指令！
:::

2. 源处理器核向目标处理器核通过IPI_Send发送IPI中断。

```c
static void ipi_write_action(int cpu, u32 action)
{
	uint32_t val;

	val = IOCSR_IPI_SEND_BLOCKING | action;
	val |= (cpu << IOCSR_IPI_SEND_CPU_SHIFT);
	iocsr_write32(val, LOONGARCH_IOCSR_IPI_SEND);
}
```

根据IPI_Send的定义:
- IPISend[31]: 等待完成的标志，置1时会等待写入生效。上述例子中，IOCSR_IPI_SEND_BLOCKING的值为``1<<31``。
- IPISend[25:16]: 是写入目的CPU的IPI_Status寄存器，比如为0，是写入CPU-0；值为1，就是写入CPU-1。上述
  例子中，直接将``cpu<<16``写入。
- MailSend[4:0]:中断向量号，对应于目标CPU中IPI_Status中的向量（注意是32位）。      
  比如，向MailSend[4:0]写入0，对应于目标CPU中IPI_Status[0]=1;      
  向MailSend[4:0]写入3，对应于目标CPU中IPI_Status[3]=1;


下面是Linux中，LoongArch的SMP定义的几个向量类型，相当于定义了IPI_Status寄存器中，每位对应的含义。

```c
#define ACTION_BOOT_CPU	0
#define ACTION_RESCHEDULE	1
#define ACTION_CALL_FUNCTION	2
#define ACTION_IRQ_WORK		3
#define ACTION_CLEAR_VECTOR	4
#define SMP_BOOT_CPU		BIT(ACTION_BOOT_CPU)
#define SMP_RESCHEDULE		BIT(ACTION_RESCHEDULE)
#define SMP_CALL_FUNCTION	BIT(ACTION_CALL_FUNCTION)
#define SMP_IRQ_WORK		BIT(ACTION_IRQ_WORK)
#define SMP_CLEAR_VECTOR	BIT(ACTION_CLEAR_VECTOR)
```

:::{tip}
上述``iocsr_write32(val, LOONGARCH_IOCSR_IPI_SEND)``的执行结果，会根据IPI_Send值具体的定义，     
写入目标寄存器的IPI_Status寄存器中。至于会不会产生中断，还是要根据配置情况。
:::


3. 目标处理器核根据配置选项，看是否产生IPI中断。

具体是否产生中断的判断是： 

```verilog
IPI <= |(IPI_Status[31:0] & IPI_Enable[31:0])
```
如果对应IPI_Status的使能位IPI_Enable也置1了，就会产生相应的IPI中断，反之，就算IPI_Status置位了，但是IPI_Enable
没有使能，也是不会产生IPI的异常。

(loongson_ipi_interrupt_routine)=

4. 产生中断异常后，进入异常处理程序，目标处理器核根据自身当前的IPI_Status来决定执行什么样的操作。

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

5. 执行完相应的IPI历程外，还需要清除对应的在IPI_Status中的置位。
```c
static u32 ipi_read_clear(int cpu)
{
	u32 action;

	/* Load the ipi register to figure out what we're supposed to do */
	action = iocsr_read32(LOONGARCH_IOCSR_IPI_STATUS);
	/* Clear the ipi register to clear the interrupt */
	iocsr_write32(action, LOONGARCH_IOCSR_IPI_CLEAR);
	wbflush();

	return action;
}
```

主要是想IPI_Clear对应的位写1，就会将IPI_Status对应的位的1清除。


6. 如果当前处理器核想主动的发起IPI中断，也是可行的，主要是通过向IPI_Set寄存器的对应位写入1。

```c
iocsr_write32(action, LOONGARCH_IOCSR_IPI_SET);
```

后续的处理方式和前面的设置是一样的。
