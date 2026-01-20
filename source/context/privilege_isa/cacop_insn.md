# CACOP指令

格式: `cacop code,rj,si12`

操作:

CACOP 指令主要用于 Cache 的初始化以及 Cache 一致性维护。

通用寄存器`rj`的值加上符号扩展后的12位立即数`si12`，将得到`CACOP`指令所用的虚拟地址`VA`，其将用于指示被操作`Cache`行的位置。

`CACOP`指令访问那个`Cache`，以及对该`Cache`进行哪种操作，由指令中5比特的`code`决定。

`code[2:0]`指示操作的`Cache`对象，`code[4:3]`指示操作类型。

`code[2:0]`指示的缓存对象与`CPUCFG10`中标识的`Cache`顺序相同。

例如，当`CPUCFG10`= 0x02C3D 时，`code[2:0]` = 0 表示对一级私有指令缓存的操作，`code[2:0]` = 1 表示对一级私有数据缓存的操作，`code[2:0]` = 2 表示对二级私有混合缓存的操作，`code[2:0]` = 3 表示对三级共享混合缓存的操作。

`code[4:3]` = 0 用于`Cache`初始化（存储标签），将指定`Cache`行的标签置为全 0 。假设要访问的`Cache`有 (1 << `Way`) 路，每路有 (1 << `Index`) 个缓存行，每个缓存行大小为 (1 << `0ffset`) 字节，那么，采用直接地址索引方法意味着，指令操作该`Cache`的第`VA[Way-1:0]` 路的第`VA[Index+offset - 1:0ffset]`个行。

`code[4:3]` = 1 表示通过直接地址索引来维护缓存一致性（索引失效/失效并回写）。维护一致性的操作是对指定`Cache`执行无效并写回的操作。
如果操作针对指令`Cache`，则仅执行失效操作，而不需要将`Cache`行中的数据写回。
数据写回到哪一级存储中，由`Cache`层次结构的具体实现以及各级之间的包含或互斥关系决定。
对于数据`Cache`或混合`Cache`，是否仅在`Cache`行为脏时回写数据由实现决定。

`code[4:3]` = 2 表示通过查询索引来维护`Cache`一致性（命中失效/失效和写回）。
所谓的查询索引方法将`CACOP`指令的虚拟地址`VA`当作普通加载指令来访问要操作的`Cache`。
如果命中，则对命中的`Cache`行进行操作，否则不做任何操作。
由于此查询过程可能涉及虚实地址的转换，因此在这种情况下，`CACOP`指令可能会触发与 `TLB`相关的异常。不过，由于`CACOP`指令是对`Cache`行进行操作，所以在此情况下无需考虑地址是否对齐。

`code[4:3]` = 3 是自定义缓存操作的一种实现方式，并未在架构规范中明确给出其功能定义。


在`Linux 6.10`中，`Cacop`指令常用于内核清空`Cache`操作。

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