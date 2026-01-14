# TLB指令

`TLB`指令，仅在`PLV0` 特权等级下才能执行。

可参考《龙芯架构参考手册卷一》 4.2.4 章节。

## 软件重填相关指令

### TLBSRCH

指令格式: `tlbsrch`

`tlbsrch`指令实现`TLB`表项查找。

执行该指令，前置需要使用`CSR`相关指令，将查询信息索引配置到`CSR.ASID`和`CSR.TLBEHI`中，应将件使用`CSR.ASID`和`CSR.TLBEHI`的信息，去查询`TLB`。
如果有命中项，那么将命中项的索引值写入到`CSR.TLBIDX`的`Index`域，同时将 `CSR.TLBIDX`的`NE`位置为 0；如果没有命中项，那么将`CSR.TLBIDX`的`NE`位置为 1。

其中，`TLB`中表项的索引值规则，是从0开始依次递增编号，从`STLB`到`MTLB`，从`STLB`的第 0 路第 0 行至最后一行，然后从第 1 路第 0 行至最后一行，直至最后一路最后一行。
`MTLB`从第 0 行至最后一行。

### TLBRD

指令格式: `tlbrd`

`tlbrd`指令实现`TLB`表项读取。

执行该指令，前置需要使用`CSR`相关指令，将查询信息配置到`CSR.TLBIDX`中。
如果指定位置处是一个有效`TLB`表项，那么将该`TLB`项的页表信息写入到`CSR.TLBEHI`、`CSR.TLBELO0`、`CSR.TLBELO1`和`CSR.TLBIDX.PS`中，且将`CSR.TLBIDX`的`NE`位置为 0。
如果指定位置处是一个无效`TLB`项，需将`CSR.TLBIDX`的`NE`位置为 1，且建议对读出内容进行屏蔽保护，如`CSR.ASID.ASID`、`CSR.TLBEHI`、`CSR.TLBELO0`、`CSR.TLBELO1`和`CSR.TLBIDX.PS`都不更新或全置为 0。

如果访问使用的索引值超出`TLB`范围，处理器行为不确定。

### TLBWR

指令格式: `tlbwr`

`tlbwr`指令实现TLB表项写入。

执行该指令，前置需要使用`CSR`指令，将页表项信息写入到相关`CSR`，包括`CSR.TLBEHI`、`CSR.TLBELO0`、`CSR.TLBELO1`和`CSR.TLBIDX.PS`。并根据`CSR.TLBIDX`的`Index`域，确定写入`TLB`表项位置。

同时，如果`CSR.TLBRERA.IsTLBR` = 1，即处于`TLB`重填例外处理过程中，则填入表项是一个有效项。否则，需要依旧`CSR.TLBIDX.NE`位，判断写入的`TLB`表项是否有效。如果`CSR.TLBIDX.NE` = 1，那么填入表项是一个无效表项，仅当`CSR.TLBIDX.NE` = 0，`TLB`才会被填入一个有效表项。

### TLBFILL

指令格式: `tlbfill`

`tlbfill`指令实现TLB表项填入。

被填入的页表项信息，来自于`CSR.TLBEHI`、`CSR.TLBELO0`、`CSR.TLBELO1`和`CSR.TLBIDX.PS`。如此时`CSR.TLBRERA.IsTLBR` = 1，即处于`TLB`重填例外处理过程中，则填入表项是一个有效项。否则，需要依旧`CSR.TLBIDX.NE`位，判断写入的`TLB`表项是否有效。如果`CSR.TLBIDX.NE` = 1，那么填入表项是一个无效表项，仅当`CSR.TLBIDX.NE` = 0，`TLB`才会被填入一个有效表项。

页表项填入时，首先根据被填入页表项的页大小来决定写入`STLB`或`MTLB`。

当被填入的页表项页大小与`STLB`配置的页大小(`CSR.STLBPS`)相等时，会填入`STLB`，否则填入`MTLB`。填入对应`TLB`的位置，由硬件决定。

### TLBCLR

指令格式: `tlbclr`

`tlbclr`指令实现指定的`TLB`表项清除,以维持`TLB`与内存之间页表数据的一致性。

执行`tlbclr`指令，前置需要使用`CSR`相关指令，将无效表项索引填入`CSR.TLBIDX`与`CSR.ASID`。根据`CSR.TLBIDX`的`Index`域，将对应位置且满足`G=0`且`ASID`等于`CSR.ASID.ASID`条件的表项无效。

### TLBFLUSH

指令格式: `tlbflush`

`tlbflush`指令实现TLB表项无效。

执行`tlbflush`指令，需要根据`CSR.TLBIDX`，找到对应表项。

如果对应表项在`MTLB`，则将`MTLB`中所有页表项无效掉。如果落在`STLB`范围，则将`STLB`中对应表项所在路中的页表项无效掉。

### INVTLB

指令格式: `invtlb op,rj,rk`

`invtlb`指令用于无效 TLB 中的内容，以维持 TLB 与内存之间页表数据的一致性。

指令源操作数中，`op`是 5 比特立即数，指示操作类型。

通用寄存器`rj`中，[9:0] 位存放无效操作所需的`ASID`。如果操作类型不需要`ASID`，应将该`rj`设置为`r0`。

通用寄存器`rk`存放无效操作所需要的虚拟地址，如果操作类型不需要虚拟地址，应将该`rk`设置为`r0`。

## 页表查找指令

### LDDIR

指令格式: `lddir rd,rj,level`

`lddir`指令用于访问目录项。

指令中，8 比特立即数`level`用于指示当前访问的页表级数。`rj`寄存器则可能为一个页表项或当前级数的页表基址地址。
如果`rj[6]`为 0，表明`rj`为第`level`级页表的基址的物理地址，指令会根据当前`CSR.TLBRBADV`访问第`level`级页表，取回下一级页表的基址，写入到通用寄存器`rd`中。
如果`rj[6]`为 1，表明`rj`为一个大页页表项。再考虑`rj[14:13]`。如果`rj[14:13]`为 0 ，将`rj[14:13]`位替换为`level[1:0]`后， 整体写入到`rd`；否则将rj直接写入到`rd`中。

### LDPTE

指令格式: `ldpte rd,rj,level`

`ldpte`该指令用于访问页表项。
立即数`seq`用于指示访问的偶数页还是奇数页。访问偶数页时结果将写入`CSR.TLBRELO0`，访问奇数页时结果将写入`CSR.TLBRELO1`。
如果`rj[6]`为0，表明`rj`为 PTE 该级页表基址的物理地址，指令会根据`CSR.TLBRBADV`访问页表，写回CSR；否则，将`rj[14:13]`替换为对应级数，写回 CSR。

## 应用示例

在linux kernel 6.10 中，由软件实现的`TLB`重填过程，结合使用了上述指令。

在`arch/loongarch/mm/tlbex.S`中，用汇编实现了重填过程。
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
上述函数首先使用`lddir`指令，遍历页表的目录项，再使用`ldpte`指令，读取页表项，最终，使用`tlbfill`实现`TLB`填入。