# CPUCFG 指令

格式: `CPUCFG 	rd,rj`

操作:

	GPR[rd] = SigExtend(ConfigList[GPR[rj]])

CPUCFG 指令主要用于识别处理器的配置信息字。

LoongArch 架构下，指令系统的功能特性的实现情况，都记录在一些列的配置信息字中，CPUCFG 指令执行一次，可以读取一个配置信息字。

寄存器 rj 中存放待访问的配置信息字的编号，指令执行后，所读取的配置信息字写入寄存器 rd 中。

配置信息字有独立的编号空间，一个配置信息字为 32 比特，每个配置信息字又包含一系列配置位，即多比特组成一个域，表达特定含义。

GPUCFG 可访问的配置信息列表，可以在手册《龙芯架构参考手册卷一》2.2.10.5 章节中查看。

指令读取基本单位为 32 比特。在 LA64 架构下，需要将读取结果进行符号扩展后，再写入寄存器 rd 。

CPUCFG 指令访问未定义的配置字，将读回全 0 值，访问已定义配置字中的未定义区域，将读回任意值。

:::{tip}
在哪里用到该指令？

举例说明，在 Linux 6.10 源码中，可以在`./arch/loongarch/kernel/cpu-probe.c`文件中看到:
``` C
#define LOONGARCH_CPUCFG0		0x0
#define LOONGARCH_CPUCFG2		0x2
#define  CPUCFG2_FPVERS			GENMASK(5, 3)

void cpu_probe(void)
{
	//	...
	c->processor_id = read_cpucfg(LOONGARCH_CPUCFG0);
	c->fpu_vers     = (read_cpucfg(LOONGARCH_CPUCFG2) & CPUCFG2_FPVERS) >> 3;
	//	...
}
```
函数 read_cpucfg 利用 CPUCFG 指令，实现配置信息字的读取。

在 Linux Kernel 启动时，会读取硬件的特性配置信息，比如编号为 0x0 的配置信息字，包含处理器标识信息。编号为 0x2 的配置信息字中，[5:3] 的比特位，代表浮点运算标准的版本号。
:::


# CACOP 指令

格式: `cacop code,rj,si12`

操作:

	vaddr = REG[rj] + SigExtend(si12,GRLEN)
	CacheFunc(va,code)

CACOP 指令主要用于 Cache 的初始化以及 Cache 一致性维护。

寄存器 rj 的值加上符号扩展后的12位立即数 si12 ，将得到 CACOP 指令所用的虚拟地址 VA ，其将用于指示被操作 Cache 行的位置。

CACOP 指令访问那个 Cache ，以及对该 Cache 进行哪种操作，由指令中5比特的 code 决定。

code[2:0] 指示操作的 Cache 对象， code[4:3] 指示操作类型。

code[2:0] 指示的缓存对象与 CPUCFG.10 中标识的 Cache 顺序相同。

:::{note}
如何获取CPU Cache 配置信息？进而指定 code[2:0] 操作对象？

可使用 CPUCFG 指令，从特定编号的配置信息字中查询。

	#	加载编号 0x10
	li 	t1,0x10
	#	读取配置信息字
	cpucfg 	t2,t1

编号为 0x10 配置信息字，分别表示了 L1 、L2 、L3 级 Cache 的属性，包括: 是否存在、是否为每个核私有、以及与其他层级 Cache 的包含关系。

例如，当 CPUCFG10.10 = 0x02C3D 时，[3:0] 位的 D 值表示存在 L1 Cache，并且指令、数据缓存分别存在，同时，存在 L2 Cache；[7:4] 位的 3 值，表示 L2 Cache 为私有混合 Cache，且与 L1 Cache 并不是包含关系；[11:8]位的 C 值，表示存在 L3 混合 Cache ；[15:12] 的值 2 表示 L3 Cache 是共享混合 Cache ，且与低层次的 Cache 是包含关系。

因此，code[2:0] = 0 时表示对一级私有指令 Cache 的操作，code[2:0]  = 1 时表示对一级私有数据 Cache 的操作， code[2:0] = 2 表示对二级私有混合 Cache 的操作，code[2:0] = 3 表示对三级共享混合 Cache 的操作。
:::

code[4:3] = 0 用于 Cache 初始化（存储标签），将指定 Cache 行的标签置为全 0 。假设要访问的 Cache 有 (1 << Way ) 路，每路有 (1 << Index ) 个缓存行，每个缓存行大小为 (1 << 0ffset ) 字节，那么，采用直接地址索引方法意味着，指令操作该 Cache 的第 VA[Way + Index + offset - 1:Index + offset] 路的第 VA[Index + offset - 1:0ffset] 个行。

code[4:3] = 1 表示通过直接地址索引来维护缓存一致性（索引失效/写回）。维护一致性的操作是对指定 Cache 执行无效并写回的操作。
如果操作针对指令 Cache ，则仅执行失效操作，并且不需要将 Cache 行中的数据写回。
数据写回到哪一级存储中，由 Cache 层次结构的具体实现以及各级之间的包含或互斥关系决定。
对于数据 Cache 或混合 Cache ，是否仅在 Cache 行为脏时回写数据由实现决定。

code[4:3] = 2 表示通过查询索引来维护 Cache 一致性（命中失效/写回）。
所谓的查询索引方法将 CACOP 指令的虚拟地址 VA 当作普通 Load 指令来访问要操作的 Cache 。
如果命中，则对命中的 Cache 行进行操作，即，如果该 Cache 行为脏，就进行写回操作。如果没有命中，则不做任何操作。
由于此查询过程可能涉及虚实地址的转换，因此在这种情况下， CACOP 指令可能会触发与 TLB 相关的异常。不过，由于 CACOP 指令是对 Cache 行进行操作，所以在此情况下无需考虑地址是否对齐。

code[4:3] = 3 是自定义缓存操作的一种实现方式，并未在架构规范中明确给出其功能定义。

:::{tip}
在哪里用到该指令？

在 Linux 6.10 中，Cacop 指令用于内核清空 Cache 操作。

在`arch/loongarch/include/asm/cacheflush.h`中，通过内联汇编语法，为`Cacop`创建函数并调用。

``` C
#define cache_op(op, addr)						\
	__asm__ __volatile__(						\
	"	cacop	%0, %1					\n"	\
	:								\
	: "i" (op), "ZC" (*(unsigned char *)(addr)))


static inline void flush_cache_line(int leaf, unsigned long addr)
{
	switch (leaf) {
	case Cache_LEAF0:
		cache_op(Index_Writeback_Inv_LEAF0, addr);
		break;
	case Cache_LEAF1:
		cache_op(Index_Writeback_Inv_LEAF1, addr);
		break;
	case Cache_LEAF2:
		cache_op(Index_Writeback_Inv_LEAF2, addr);
		break;
	case Cache_LEAF3:
		cache_op(Index_Writeback_Inv_LEAF3, addr);
		break;
	case Cache_LEAF4:
		cache_op(Index_Writeback_Inv_LEAF4, addr);
		break;
	case Cache_LEAF5:
		cache_op(Index_Writeback_Inv_LEAF5, addr);
		break;
	default:
		break;
	}
}
```
:::