# TLB 与页表指令

虚实地址映射，是操作系统更加高效、安全实现内存管理的基础，而 TLB 则是处理器中存放映射关系的临时缓存，用于加速虚实地址转换过程。

LoongArch 架构下，页表组织映射方式、TLB 的结构与管理，请查看手册《龙芯架构参考手册卷一》5 章节。

为提高性能，优化管理，LoongArch 架构提供了页表和 TLB 相关指令支持。

需要注意的是，页表和 TLB 相关指令，仅在 PLV0 特权等级下才能执行。

## TLB 相关指令

TLB 相关指令用于 TLB 维护，包括 TLB 中表项查找、读取、写入、无效化等操作。

### TLBSRCH

格式: `tlbsrch`

操作:

	CST.TLBIDX.NE = 1
	for (i <- tlb_num)	#	遍历 TLB
		if ( CSR.TLBEHI == TLB[i].VPPN && CSR.ASID == TLB[i].ASID)	#	如果 VPPN 与 ASID 匹配
			CSR.TLBIDX.INDEX = i
			CSR.TLBIDX.NE = 0

TLBSRCH 指令实现 TLB 表项查找。

执行该指令，前置需要使用 CSR 相关指令，将查询信息索引配置到 CSR.ASID 和 CSR.TLBEHI 中，其中 CSR.ASID 存放查询表项时，对应进程编号，CSR.TLBEHI 中保存虚拟地址的 [VALEN - 1:13] 位。

硬件将使用 CSR.ASID 和 CSR.TLBEHI 的信息，去遍历 TLB 中的所有表项。

如果有命中项，那么将命中项的索引值写入到 CSR.TLBIDX 的 Index 域，同时将 CSR.TLBIDX 的 NE 位置为 0；如果没有命中项，那么将 CSR.TLBIDX 的 NE 位置为 1。

其中，TLB 中表项的索引值规则，是从0开始依次递增编号，从 STLB 到 MTLB ，从 STLB 的第 0 路第 0 行至最后一行，然后从第 1 路第 0 行至最后一行，直至最后一路最后一行。
MTLB 从第 0 行至最后一行。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_tlbsrch`函数。
:::

### TLBRD

格式: `tlbrd`

操作:

	index = CSR.TLBIDX.INDEX
	if ( TLB[index].V == 1)		#	如果表项有效
		CSR.TLBEHI = TLB[index].VPPN
		CSR.TLBLO0 = TLB[index].ENTRY0
		CSR.TLBLO1 = TLB[index].ENTRY1
		CSR.TLBIDX.PS = TLB[index].PS
		CSR.TLBIDX.NE = 0
	else
		CSR.TLBIDX.NE = 1

TLBRD 指令实现 TLB 表项读取。

执行该指令，前置需要使用 CSR 相关指令，将查询信息配置到 CSR.TLBIDX 中。

如果指定位置处是一个有效 TLB 表项，那么将该 TLB 项的页表信息写入到 CSR.TLBEHI 、 CSR.TLBELO0 、 CSR.TLBELO1 和 CSR.TLBIDX.PS 中，且将 CSR.TLBIDX 的 NE 位置为 0。

如果指定位置处是一个无效 TLB 项，需将 CSR.TLBIDX 的 NE 位置为 1，且建议对读出内容进行屏蔽保护，如 CSR.ASID.ASID 、 CSR.TLBEHI 、 CSR.TLBELO0 、 CSR.TLBELO1 和 CSR.TLBIDX.PS 都不更新或全置为 0。

其中，TLB 中表项的索引值规则，是从0开始依次递增编号，从 STLB 到 MTLB ，从 STLB 的第 0 路第 0 行至最后一行，然后从第 1 路第 0 行至最后一行，直至最后一路最后一行。
MTLB 从第 0 行至最后一行。

如果访问使用的索引值超出 TLB 范围，处理器行为不确定。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_tlbrd`函数。
:::

### TLBWR

格式: `tlbwr`

操作:

	#	确定位置索引并写入
	index = CSR.TLBIDX.INDEX
	TLB[index].VPPN 	= CSR.TLBEHI
	TLB[index].ENTRY0 	= CSR.TLBLO0
	TLB[index].ENTRY1 	= CSR.TLBLO1
	TLB[index].PS 		= CSR.TLBIDX.PS
	#	判断是否有效
	if ( CSR.TLBRERA.IsTLBR == 1 || CSR.TLBIDX.NE == 0)
		TLB[index].V = 1
	else
		TLB[index].V = 0


TLBWR 指令实现 TLB 表项写入。

执行该指令，前置需要使用 CSR 指令，将页表项信息写入到相关 CSR ，包括 CSR.TLBEHI 、 CSR.TLBELO0 、 CSR.TLBELO1 和 CSR.TLBIDX.PS 。硬件根据 CSR.TLBIDX 的 Index 域，确定写入 TLB 表项位置。

同时，如果 CSR.TLBRERA.IsTLBR  = 1，即处于 TLB 重填例外处理过程中，则填入表项是一个有效项。否则，需要依旧 CSR.TLBIDX.NE 位，判断写入的 TLB 表项是否有效。如果 CSR.TLBIDX.NE  = 1，那么填入表项是一个无效表项，仅当 CSR.TLBIDX.NE  = 0， TLB 才会被填入一个有效表项。

其中，TLB 中表项的索引值规则，是从0开始依次递增编号，从 STLB 到 MTLB ，从 STLB 的第 0 路第 0 行至最后一行，然后从第 1 路第 0 行至最后一行，直至最后一路最后一行。
MTLB 从第 0 行至最后一行。

如果访问使用的索引值超出 TLB 范围，处理器行为不确定。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_tlbwr`函数。
:::

### TLBFILL

格式: `tlbfill`

操作:

	#	获取填入位置索引
	index = SelectIndex(CSR.TLBIDX.PS,CSR.STLBPS)
	#	填入索引对应位置表项
	TLB[index].VPPN 	= CSR.TLBEHI
	TLB[index].ENTRY0 	= CSR.TLBLO0
	TLB[index].ENTRY1 	= CSR.TLBLO1
	TLB[index].PS 		= CSR.TLBIDX.PS
	#	判断是否有效
	if ( CSR.TLBRERA.IsTLBR == 1 || CSR.TLBIDX.NE == 0)
		TLB[index].V = 1
	else
		TLB[index].V = 0	

TLBFILL 指令实现 TLB 表项填入。

被填入的页表项信息，来自于 CSR.TLBEHI 、 CSR.TLBELO0 、 CSR.TLBELO1 和 CSR.TLBIDX.PS 。如此时 CSR.TLBRERA.IsTLBR  = 1，即处于 TLB 重填例外处理过程中，则填入表项是一个有效项。否则，需要依旧 CSR.TLBIDX.NE 位，判断写入的 TLB 表项是否有效。如果 CSR.TLBIDX.NE  = 1，那么填入表项是一个无效表项，仅当 CSR.TLBIDX.NE  = 0， TLB 才会被填入一个有效表项。

页表项填入时，首先根据被填入页表项的页大小来决定写入 STLB 或 MTLB 。

当被填入的页表项页大小与 STLB 配置的页大小( CSR.STLBPS )相等时，会填入 STLB ，否则填入 MTLB 。填入对应 TLB 的位置，由硬件决定。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_tlbfill`函数。
:::

### TLBCLR

格式: `tlbclr`

操作:

	index = CSR.TLBIDX.INDEX
	if ( TLB[index].ASID == CSR.ASID && TLB[index].G == 0)
		TLB[index].V = 0

TLBCLR 指令实现指定的 TLB 表项清除,以维持 TLB 与内存之间页表数据的一致性。

执行 TLBCLR 指令，前置需要使用 CSR 相关指令，将无效表项索引填入 CSR.TLBIDX 与 CSR.ASID 。根据 CSR.TLBIDX 的 Index 域，将对应位置且满足 G=0 且 ASID 等于 CSR.ASID.ASID 条件的表项无效。

### TLBFLUSH

格式: `tlbflush`

TLBFLUSH 指令实现 TLB 表项无效。

执行 TLBFLUSH 指令，需要根据 CSR.TLBIDX ，找到对应表项。

如果对应表项在 MTLB ，则将 MTLB 中所有页表项无效掉。如果落在 STLB 范围，则将 STLB 中对应表项所在路中的页表项无效掉。

### INVTLB

格式: `invtlb op,rj,rk`

INVTLB 指令用于无效 TLB 中的内容，以维持 TLB 与内存之间页表数据的一致性。

指令源操作数中，op 是 5 比特立即数，指示操作类型，具体类型及其操作说明，可以在手册《龙芯架构参考手册卷一》4.2.4.7 章节中查看。

通用寄存器 rj 中，[9:0] 位存放无效操作所需的 ASID 。如果操作类型不需要 ASID ，应将该 rj 设置为 r0 。

通用寄存器 rk 存放无效操作所需要的虚拟地址，如果操作类型不需要虚拟地址，应将该 rk 设置为 r0 。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_invtlb_all`函数。
:::

## 页表查找指令

### LDDIR

格式: `lddir rd,rj,level`

操作:

	#	获取需查找的 VA
	badvaddr = CSR.TLBRBADV
	#	当前级页表的宽度与基址
	dir_base = CSR.PWCL.BASE
	dir_width = CSR.PWCL.WIDTH
	#	查找下一级页表
	if ( rj[6] == 0 )
		#	通过 VA 获取下一级页表的索引
		index = (badvaddr >> dir_base) & ((1 << dir_width) - 1);
		next_base = rj[6] | (index << 8)
		#	从内存中读取下一级页表项
		GPR[rd] = RAM_LOAD(next_base)


LDDIR 指令用于访问目录项。

指令中，8 比特立即数 level 用于指示要当前访问的页表级数。 level = 1 对应 CSR.PWCL 中对应的 PT ，level = 2 对应 CSR.PWCL 中对应的 Dir1 ，level = 3 对应 CSR.PWCL 中对应的 Dir2 ，level = 4 对应 CSR.PWCL 中对应的 Dir3 。

具体页表级数对应关系，可以在手册《龙芯架构参考手册卷一》5.4.5 章节中查看。

rj 寄存器则可能为一个页表项或当前级数的页表基址地址。
如果 rj[6] 为 0，表明 rj 为第 level 级页表的基址的物理地址，指令会根据当前 CSR.TLBRBADV 访问第 level 级页表，取回下一级页表的基址，写入到通用寄存器 rd 中。
如果 rj[6] 为 1，表明 rj 为一个大页页表项。再考虑 rj[14:13] 。如果 rj[14:13] 为 0 ，将 rj[14:13] 位替换为 level[1:0] 后， 整体写入到 rd ；否则将rj直接写入到 rd 中。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_lddir`函数。
:::

### LDPTE

格式: `ldpte rj,seq`

操作:

	#	获取需查找的 VA
	badvaddr = CSR.TLBRBADV
	#	当前级页表的宽度与基址
	dir_base = CSR.PWCL.BASE
	dir_width = CSR.PWCL.WIDTH
	#	查找下一级页表
	if ( rj[6] == 0 )
		#	通过 VA 获取下一级页表的索引
		index = (badvaddr >> dir_base) & ((1 << dir_width) - 1);
		#	奇数项基址
		next_base_0 = rj[6] | (index << 8)
		#	从内存中读取下一级页表项
		CSR.TLBRELO1 = RAM_LOAD(next_base_0)
		#	偶数项基址
		next_base_1 = rj[6] | ((index + 1) << 8)
		#	从内存中读取下一级页表项
		CSR.TLBRELO0 = RAM_LOAD(next_base_1)

LDPTE 指令用于访问页表项。

立即数 seq 用于指示访问的偶数页还是奇数页。访问偶数页时结果将写入 CSR.TLBRELO0 ，访问奇数页时结果将写入 CSR.TLBRELO1 。
如果 rj[6] 为0，表明 rj 为 PTE 该级页表基址的物理地址，指令会根据 CSR.TLBRBADV 访问页表，写回CSR；否则，将 rj[14:13] 替换为对应级数，写回 CSR。

:::{tip}
指令实现原理，可参考模拟器 LA_EMU 的`loongarch64/tlb_helper.c`中`helper_ldpte`函数。
:::

## 应用示例

在linux 6.10 中，由软件实现的`TLB`重填过程，结合使用了上述指令。

在`arch/loongarch/mm/tlbex.S`中，用汇编实现了软件重填过程。
``` asm
SYM_CODE_START(handle_tlb_refill)
	UNWIND_HINT_UNDEFINED
	csrwr		t0, LOONGARCH_CSR_TLBRSAVE
	csrrd		t0, LOONGARCH_CSR_PGD
	lddir		t0, t0, 3
#if CONFIG_PGTABLE_LEVELS > 3
	lddir		t0, t0, 2
#endif
#if CONFIG_PGTABLE_LEVELS > 2
	lddir		t0, t0, 1
#endif
	ldpte		t0, 0
	ldpte		t0, 1
	tlbfill
	csrrd		t0, LOONGARCH_CSR_TLBRSAVE
	ertn
SYM_CODE_END(handle_tlb_refill)
```
上述函数首先使用 LDDIR 指令，遍历页表的目录项，再使用 LDPTE 指令，读取页表项，最终，使用 TLBFILL 实现 TLB 填入。