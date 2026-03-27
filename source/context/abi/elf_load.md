# 程序如何加载-扩展阅读

假设有下面一段C代码，我们将其编译成可执行文件，然后通过分析其加载过程，学习有关LoongArch龙架构    
二进制接口重定位相关的知识。


```c
#include <stdio.h>

long global_val = 0;

int main(int argc, char const *argv[])
{
	global_val = 0x100;

	printf("GLobal Value: %lx,\n", global_val);
	return 0;
}
```

可执行文件的信息如下：

```
bash> readelf -h hello

ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           LoongArch
  Version:                           0x1
  Entry point address:               0x120000540
  Start of program headers:          64 (bytes into file)
  Start of section headers:          91968 (bytes into file)
  Flags:                             0x43, DOUBLE-FLOAT, OBJ-v1
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         8
  Size of section headers:           64 (bytes)
  Number of section headers:         33
  Section header string table index: 32
```

上面显示了ELF可执行文件的程序头相关信息。


```
bash> readelf -l hello

Elf file type is EXEC (Executable file)
Entry point 0x120000540
There are 8 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000120000040 0x0000000120000040
                 0x00000000000001c0 0x00000000000001c0  R      0x8
  INTERP         0x0000000000000200 0x0000000120000200 0x0000000120000200
                 0x000000000000001e 0x000000000000001e  R      0x1
      [Requesting program interpreter: /lib/ld-musl-loongarch64.so.1]
  LOAD           0x0000000000000000 0x0000000120000000 0x0000000120000000
                 0x0000000000000800 0x0000000000000800  R E    0x10000
  LOAD           0x000000000000fdf0 0x000000012001fdf0 0x000000012001fdf0
                 0x0000000000000238 0x0000000000000278  RW     0x10000
  DYNAMIC        0x000000000000fe00 0x000000012001fe00 0x000000012001fe00
                 0x00000000000001b0 0x00000000000001b0  RW     0x8
  GNU_EH_FRAME   0x00000000000007ac 0x00000001200007ac 0x00000001200007ac
                 0x0000000000000014 0x0000000000000014  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x000000000000fdf0 0x000000012001fdf0 0x000000012001fdf0
                 0x0000000000000210 0x0000000000000210  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .hash .gnu.hash .dynsym .dynstr .rela.dyn .rela.plt .plt .text .rodata .eh_frame_hdr .eh_frame 
   03     .init_array .fini_array .dynamic .got .got.plt .sdata .bss 
   04     .dynamic 
   05     .eh_frame_hdr 
   06     
   07     .init_array .fini_array .dynamic .got

```

可以看出其解释器是``/lib/ld-musl-loongarch64.so.1``。


```
bash> readelf -d hello

Dynamic section at offset 0xfe00 contains 22 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000c (INIT)               0x0
 0x000000000000000d (FINI)               0x0
 0x0000000000000019 (INIT_ARRAY)         0x12001fdf0
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x12001fdf8
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x0000000000000004 (HASH)               0x120000220
 0x000000006ffffef5 (GNU_HASH)           0x120000258
 0x0000000000000005 (STRTAB)             0x120000350
 0x0000000000000006 (SYMTAB)             0x120000278
 0x000000000000000a (STRSZ)              146 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x12001fff0
 0x0000000000000002 (PLTRELSZ)           96 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x120000478
 0x0000000000000007 (RELA)               0x1200003e8
 0x0000000000000008 (RELASZ)             144 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x0000000000000000 (NULL)               0x0
```

动态链接的情况如下：

```
bash> readelf -r hello

Relocation section '.rela.dyn' at offset 0x3e8 contains 6 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
00012001ffb8  000200000002 R_LARCH_64        0000000000000000 _init + 0
00012001ffc0  000300000002 R_LARCH_64        0000000000000000 __deregister_fram[...] + 0
00012001ffc8  000400000002 R_LARCH_64        0000000000000000 _ITM_registerTMCl[...] + 0
00012001ffd0  000500000002 R_LARCH_64        0000000000000000 _ITM_deregisterTM[...] + 0
00012001ffe0  000600000002 R_LARCH_64        0000000000000000 _fini + 0
00012001ffe8  000800000002 R_LARCH_64        0000000000000000 __register_frame_info + 0

Relocation section '.rela.plt' at offset 0x478 contains 4 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000120020000  000100000005 R_LARCH_JUMP_SLOT 0000000120000500 printf + 0
000120020008  000300000005 R_LARCH_JUMP_SLOT 0000000000000000 __deregister_fram[...] + 0
000120020010  000700000005 R_LARCH_JUMP_SLOT 0000000120000520 __libc_start_main + 0
000120020018  000800000005 R_LARCH_JUMP_SLOT 0000000000000000 __register_frame_info + 0
```

查看其依赖的动态链接库libc.so，以及.rela.dyn .rela.plt段的地址信息等。



下面我们分析即可执行文件hello的执行过程。


**下面是内核空间的操作**：


**1. ELF可执行文件的加载**

主要的核心函数在linux Kernel中load_elf_binary函数(binfmt_elf.c)

```c
elf_phdata = load_elf_phdrs(elf_ex, bprm->file)
```
加载ELF程序头表，可查看上节的相关描述。


```c
retval = elf_read(interpreter, interp_elf_ex,
				  sizeof(*interp_elf_ex), 0);
```
如果有解释器的话，就先加载解释器到内存中。本例子中解释器是``/lib/ld-musl-loongarch64.so.1``。

```c
if (interpreter) {
	//...
	/* Load the interpreter program headers */
		interp_elf_phdata = load_elf_phdrs(interp_elf_ex,
						   interpreter);

```

设置用户栈的执行信息

```c
retval = setup_arg_pages(bprm, randomize_stack_top(STACK_TOP),
				 executable_stack);
```


**2. 加载ELF可执行程序的数据段和代码段（本例子中是hello可执行程序）**

```c
/* Now we do a little grungy work by mmapping the ELF image into
	   the correct location in memory. */
	for(i = 0, elf_ppnt = elf_phdata;
	    i < elf_ex->e_phnum; i++, elf_ppnt++) {

		// ......
		if (!first_pt_load) {
			elf_flags |= MAP_FIXED;
		} else if (elf_ex->e_type == ET_EXEC) {
			/*
			 * This logic is run once for the first LOAD Program
			 * Header for ET_EXEC binaries. No special handling
			 * is needed.
			 */
			elf_flags |= MAP_FIXED_NOREPLACE;
		} else if (elf_ex->e_type == ET_DYN) {
			//...
		}
		error = elf_load(bprm->file, load_bias + vaddr, elf_ppnt,
				elf_prot, elf_flags, total_size);

```

将各个数据段，代码段加载到各自合适的位置上，具体可[查看代码](https://github.com/torvalds/linux/blob/master/fs/binfmt_elf.c)。


**3. 如果有解释器的话，加载解释器的各个段，（本例子中的解释器为``/lib/ld-musl-loongarch64.so.1``）**

```c
        if (interpreter) {
		     elf_entry = load_elf_interp(interp_elf_ex,
					    interpreter,
					    load_bias, interp_elf_phdata,
					    &arch_state);
		} else {
		     elf_entry = e_entry;
		     if (BAD_ADDR(elf_entry)) {
		     	retval = -EINVAL;
		     	goto out_free_dentry;
		     }
	    }
```
此时的elf_entry的地址是ld.so（ld-musl-loongarch64.so.1）的入口地址。


**4. 将相关的信息复制到用户的栈顶空间中。**

```c
create_elf_tables(struct linux_binprm *bprm, const struct elfhdr *exec,
		unsigned long interp_load_addr,
		unsigned long e_entry, unsigned long phdr_addr)
```

核心的执行函数为create_elf_tables，我们下面分析其执行过程。


产生ELF解释器的信息：

```c
	/* Create the ELF interpreter info */
	elf_info = (elf_addr_t *)mm->saved_auxv;
	/* update AT_VECTOR_SIZE_BASE if the number of NEW_AUX_ENT() changes */
#define NEW_AUX_ENT(id, val) \
	do { \
		*elf_info++ = id; \
		*elf_info++ = val; \
	} while (0)

	//...

	NEW_AUX_ENT(AT_HWCAP, ELF_HWCAP);
	NEW_AUX_ENT(AT_PAGESZ, ELF_EXEC_PAGESIZE);
	NEW_AUX_ENT(AT_CLKTCK, CLOCKS_PER_SEC);
	NEW_AUX_ENT(AT_PHDR, phdr_addr);
	NEW_AUX_ENT(AT_PHENT, sizeof(struct elf_phdr));
	NEW_AUX_ENT(AT_PHNUM, exec->e_phnum);
	NEW_AUX_ENT(AT_BASE, interp_load_addr);
	if (bprm->interp_flags & BINPRM_FLAGS_PRESERVE_ARGV0)
		flags |= AT_FLAGS_PRESERVE_ARGV0;
	NEW_AUX_ENT(AT_FLAGS, flags);
	NEW_AUX_ENT(AT_ENTRY, e_entry);
	NEW_AUX_ENT(AT_UID, from_kuid_munged(cred->user_ns, cred->uid));
	NEW_AUX_ENT(AT_EUID, from_kuid_munged(cred->user_ns, cred->euid));
	NEW_AUX_ENT(AT_GID, from_kgid_munged(cred->user_ns, cred->gid));
	NEW_AUX_ENT(AT_EGID, from_kgid_munged(cred->user_ns, cred->egid));
	NEW_AUX_ENT(AT_SECURE, bprm->secureexec);
	NEW_AUX_ENT(AT_RANDOM, (elf_addr_t)(unsigned long)u_rand_bytes);

```

放置程序的输入参数信息

```c
    /* Now, let's put argc (and argv, envp if appropriate) on the stack */
	if (put_user(argc, sp++))
		return -EFAULT;

    while (argc-- > 0) {
		size_t len;
		if (put_user((elf_addr_t)p, sp++))
			return -EFAULT;
		len = strnlen_user((void __user *)p, MAX_ARG_STRLEN);
		if (!len || len > MAX_ARG_STRLEN)
			return -EINVAL;
		p += len;
	}
```

放置程序的环境相关信息

```c
	/* Populate list of envp pointers back to envp strings. */
	mm->env_end = mm->env_start = p;
	while (envc-- > 0) {
		size_t len;
		if (put_user((elf_addr_t)p, sp++))
			return -EFAULT;
		len = strnlen_user((void __user *)p, MAX_ARG_STRLEN);
		if (!len || len > MAX_ARG_STRLEN)
			return -EINVAL;
		p += len;
	}
	if (put_user(0, sp++))
		return -EFAULT;
```

将上述的NEW_AUX_ENT拷贝到栈顶位置。

```c
	/* Put the elf_info on the stack in the right place.  */
	if (copy_to_user(sp, mm->saved_auxv, ei_index * sizeof(elf_addr_t)))
		return -EFAULT;
```


**5. 将可执行文件的入口地址复制到CSR.ERA寄存器中，用于返回用户空间。**

```c
void start_thread(struct pt_regs *regs, unsigned long pc, unsigned long sp)
{
	unsigned long crmd;
	unsigned long prmd;
	unsigned long euen;

	/* New thread loses kernel privileges. */
	crmd = regs->csr_crmd & ~(PLV_MASK);
	crmd |= PLV_USER;
	regs->csr_crmd = crmd;

	prmd = regs->csr_prmd & ~(PLV_MASK);
	prmd |= PLV_USER;
	regs->csr_prmd = prmd;

	euen = regs->csr_euen & ~(CSR_EUEN_FPEN);
	regs->csr_euen = euen;
	lose_fpu(0);
	lose_lbt(0);
	current->thread.fpu.fcsr = boot_cpu_data.fpu_csr0;

	clear_thread_flag(TIF_LSX_CTX_LIVE);
	clear_thread_flag(TIF_LASX_CTX_LIVE);
	clear_thread_flag(TIF_LBT_CTX_LIVE);
	clear_used_math();
	regs->csr_era = pc;
	regs->regs[3] = sp;
}
```

具体代码可查看[process.c](https://github.com/torvalds/linux/blob/master/arch/loongarch/kernel/process.c)。



上面的五个步骤是内核所做的事情，负责将程序加载到内存中。

--------------------


**下面我们介绍用户空间的处理流程**：

假设我们有解释器，解释器为``ld-musl-loongarch64.so.1``


**6. 内核从特权态返回用户态时，将执行权限交给解释器的入口函数。**

其ld.so的入口指令如下所示：

```c
__asm__(
".text \n"
".global " START "\n"
".type   " START ", @function\n"
START ":\n"
"	move $fp, $zero\n"
"	move $a0, $sp\n"
".weak _DYNAMIC\n"
".hidden _DYNAMIC\n"
"	la.local $a1, _DYNAMIC\n"
"	bstrins.d $sp, $zero, 3, 0\n"
"	b " START "_c\n"
);
```
代码在[这里](https://github.com/ifduyue/musl/blob/master/arch/loongarch64/crt_arch.h)

可以看出，``$a0``寄存器保存着sp栈指针，而``$a1``保存着符号_DYNAMIC的地址。

**_DYNAMIC**符号的地址就是.dynamic段的地址。我们在上面的解析可执行文件的时候，就可以看到

```
bash> readelf -l hello

...
  LOAD           0x0000000000000000 0x0000000120000000 0x0000000120000000
                 0x0000000000000800 0x0000000000000800  R E    0x10000
  LOAD           0x000000000000fdf0 0x000000012001fdf0 0x000000012001fdf0
                 0x0000000000000238 0x0000000000000278  RW     0x10000
  DYNAMIC        0x000000000000fe00 0x000000012001fe00 0x000000012001fe00
                 0x00000000000001b0 0x00000000000001b0  RW     0x8
...
```

比如此时传入的则是地址``0x12001fe00``。

然后跳入``_dlstart_c``去执行。


**7. 执行C语言环境函数（musl libc）**

其解释器的C入口函数为``_dlstart_c``，[在此处可查看源码](https://github.com/ifduyue/musl/blob/master/ldso/dlstart.c)。


解析aux数组，这是内核传入的信息，看上面的内核加载过程。
```c
	for (i=0; i<AUX_CNT; i++) aux[i] = 0;
	for (i=0; auxv[i]; i+=2) if (auxv[i]<AUX_CNT)
		aux[auxv[i]] = auxv[i+1];

```

解析DYNAMIC段的表项。

```c
for (i=0; i<DYN_CNT; i++) dyn[i] = 0;
	for (i=0; dynv[i]; i+=2) if (dynv[i]<DYN_CNT)
		dyn[dynv[i]] = dynv[i+1];

```

获得解释器加载的基址。注意，解释器是个DYN类型的可执行文件，内核会更加情况，加载到一个合适的内存地址。
```c
/* If the dynamic linker is invoked as a command, its load
	 * address is not available in the aux vector. Instead, compute
	 * the load address as the difference between &_DYNAMIC and the
	 * virtual address in the PT_DYNAMIC program header. */
	base = aux[AT_BASE];
```

**8. 解析和修正``.rela.dyn``和``.rela.plt``段的地址信息**

```c
rel = (void *)(base+dyn[DT_REL]);
	rel_size = dyn[DT_RELSZ];
	for (; rel_size; rel+=2, rel_size-=2*sizeof(size_t)) {
		if (!IS_RELATIVE(rel[1], 0)) continue;
		size_t *rel_addr = (void *)(base + rel[0]);
		*rel_addr += base;
	}

	rel = (void *)(base+dyn[DT_RELA]);
	rel_size = dyn[DT_RELASZ];
	for (; rel_size; rel+=3, rel_size-=3*sizeof(size_t)) {
		if (!IS_RELATIVE(rel[1], 0)) continue;
		size_t *rel_addr = (void *)(base + rel[0]);
		*rel_addr = base + rel[2];
	}

	rel = (void *)(base+dyn[DT_RELR]);
	rel_size = dyn[DT_RELRSZ];
	size_t *relr_addr = 0;
	for (; rel_size; rel++, rel_size-=sizeof(size_t)) {
		if ((rel[0]&1) == 0) {
			relr_addr = (void *)(base + rel[0]);
			*relr_addr++ += base;
		} else {
			for (size_t i=0, bitmap=rel[0]; bitmap>>=1; i++)
				if (bitmap&1)
					relr_addr[i] += base;
			relr_addr += 8*sizeof(size_t)-1;
		}
	}
```

```
bash> readelf -r sysroot/lib/ld-musl-loongarch64.so.1

Relocation section '.rela.dyn' at offset 0x13340 contains 97 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
00000010fb88  000000000003 R_LARCH_RELATIVE                     ca558
00000010fb90  000000000003 R_LARCH_RELATIVE                     ca958
00000010fb98  000000000003 R_LARCH_RELATIVE                     caf58
00000010fba0  000000000003 R_LARCH_RELATIVE                     1106a8
00000010fba8  000000000003 R_LARCH_RELATIVE                     1108c0

Relocation section '.rela.plt' at offset 0x13c58 contains 6 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000110000  00f600000005 R_LARCH_JUMP_SLOT 000000000003e4a8 malloc + 0
000000110008  008100000005 R_LARCH_JUMP_SLOT 000000000003e288 calloc + 0
```

上述的偏移Offset，比如0x10fb88是加载的偏移，我们实际加载到内存，需要加上一个偏移。    


也就是说我们需要在``base+0x10fb88``的地址上，进行类型``R_LARCH_RELATIVE``的修正。


而可重定位段的地址信息可以使用下面查看：

```
bash> readelf -d sysroot/lib/ld-musl-loongarch64.so.1 

Dynamic section at offset 0xffdf0 contains 17 entries:
  Tag        Type                         Name/Value
 0x000000000000000c (INIT)               0x2e274
 0x000000000000000d (FINI)               0x2f5f0
 0x0000000000000004 (HASH)               0x190
 0x000000006ffffef5 (GNU_HASH)           0x2b80
 0x0000000000000005 (STRTAB)             0xf828
 0x0000000000000006 (SYMTAB)             0x5d60
 0x000000000000000a (STRSZ)              15121 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000003 (PLTGOT)             0x10fff0
 0x0000000000000002 (PLTRELSZ)           144 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x13c58
 0x0000000000000007 (RELA)               0x13340
 0x0000000000000008 (RELASZ)             2328 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffff9 (RELACOUNT)          82
 0x0000000000000000 (NULL)               0x0

```

其中``0x0000000000000017 (JMPREL)             0x13c58``指的是``.rela.plt``段      
而``0x0000000000000007 (RELA)               0x13340``指的是``.rela.dyn``段


注意此时修正的解释器的可执行文件，也就是ld.so。


**9. 跳转到__dls2去执行。**

```c
	stage2_func dls2;
	GETFUNCSYM(&dls2, __dls2, base+dyn[DT_PLTGOT]);
	dls2((void *)base, sp);
```


**10. 动态链接器执行自身的符号重定位的准备工作。**

```c
__dls2(unsigned char *base, size_t *sp)
{
	//...
	Ehdr *ehdr = __ehdr_start ? (void *)__ehdr_start : (void *)ldso.base;
	ldso.name = ldso.shortname = "libc.so";
	ldso.phnum = ehdr->e_phnum;
	ldso.phdr = laddr(&ldso, ehdr->e_phoff);
	ldso.phentsize = ehdr->e_phentsize;
	search_vec(auxv, &ldso_page_size, AT_PAGESZ);
	kernel_mapped_dso(&ldso);
	decode_dyn(&ldso);
	//...
}
```

**11. 动态链接器ld.so自身的符号重定位操作。核心函数在``reloc_all``。**

```c
static void reloc_all(struct dso *p)
{
	size_t dyn[DYN_CNT];
	for (; p; p=p->next) {
		if (p->relocated) continue;
		decode_vec(p->dynv, dyn, DYN_CNT);
		if (NEED_MIPS_GOT_RELOCS)
			do_mips_relocs(p, laddr(p, dyn[DT_PLTGOT]));
		do_relocs(p, laddr(p, dyn[DT_JMPREL]), dyn[DT_PLTRELSZ],
			2+(dyn[DT_PLTREL]==DT_RELA));
		do_relocs(p, laddr(p, dyn[DT_REL]), dyn[DT_RELSZ], 2);
		do_relocs(p, laddr(p, dyn[DT_RELA]), dyn[DT_RELASZ], 3);
		if (!DL_FDPIC)
			do_relr_relocs(p, laddr(p, dyn[DT_RELR]), dyn[DT_RELRSZ]);

		if (head != &ldso && p->relro_start != p->relro_end) {
			long ret = __syscall(SYS_mprotect, laddr(p, p->relro_start),
				p->relro_end-p->relro_start, PROT_READ);
			if (ret != 0 && ret != -ENOSYS) {
				error("Error relocating %s: RELRO protection failed: %m",
					p->name);
				if (runtime) longjmp(*rtld_fail, 1);
			}
		}

		p->relocated = 1;
	}
}
```

可以看出主要的还是段``dyn[DT_JMPREL]，dyn[DT_REL]，dyn[DT_RELA]``三个段的重定位。


下面我们还是主要以``dyn[DT_JMPREL]``解析为主，其他的都是一样的。


**12. 核心解析函数``do_relocs``**

- 获得可重定位的目标地址reloc_addr，和addend

```c
for (; rel_size; rel+=stride, rel_size-=stride*sizeof(size_t)) {
		if (skip_relative && IS_RELATIVE(rel[1], dso->syms)) continue;
		type = R_TYPE(rel[1]);
		if (type == REL_NONE) continue;
		reloc_addr = laddr(dso, rel[0]);

		if (stride > 2) {
			addend = rel[2];
		} else if (type==REL_GOT || type==REL_PLT|| type==REL_COPY) {
			addend = 0;
		} else if (reuse_addends) {
			/* Save original addend in stage 2 where the dso
			 * chain consists of just ldso; otherwise read back
			 * saved addend since the inline one was clobbered. */
			if (head==&ldso)
				saved_addends[save_slot] = *reloc_addr;
			addend = saved_addends[save_slot++];
		} else {
			addend = *reloc_addr;
		}
```

- 获得符号信息，并且在已有的.so文件中查找相应的符号地址

```c
	sym_index = R_SYM(rel[1]);
		if (sym_index) {
			sym = syms + sym_index;
			name = strings + sym->st_name;
			ctx = type==REL_COPY ? head->syms_next : head;
			def = (sym->st_info>>4) == STB_LOCAL
				? (struct symdef){ .dso = dso, .sym = sym }
				: find_sym(ctx, name, type==REL_PLT);
			if (!def.sym) def = get_lfs64(name);
			if (!def.sym && (sym->st_shndx != SHN_UNDEF
			    || sym->st_info>>4 != STB_WEAK)) {
				if (dso->lazy && (type==REL_PLT || type==REL_GOT)) {
					dso->lazy[3*dso->lazy_cnt+0] = rel[0];
					dso->lazy[3*dso->lazy_cnt+1] = rel[1];
					dso->lazy[3*dso->lazy_cnt+2] = addend;
					dso->lazy_cnt++;
					continue;
				}
				error("Error relocating %s: %s: symbol not found",
					dso->name, name);
				if (runtime) longjmp(*rtld_fail, 1);
				continue;
			}
```

``find_sym``函数会进行具体的函数查找工作，中介涉及hash相关内容。

具体可查看[源代码](https://github.com/ifduyue/musl/blob/master/ldso/dynlink.c)。


- 根据我们上节所描述[动态链接章节](#reloc_type_dynamic)的可重定位信息，来根据相关的操作，进行地址的修正。

```c
#define REL_PLT         R_LARCH_JUMP_SLOT
#define REL_COPY        R_LARCH_COPY
#define REL_DTPMOD      R_LARCH_TLS_DTPMOD64
#define REL_DTPOFF      R_LARCH_TLS_DTPREL64
#define REL_TPOFF       R_LARCH_TLS_TPREL64
#define REL_RELATIVE    R_LARCH_RELATIVE
#define REL_SYMBOLIC    R_LARCH_64
```

```c
switch(type) {
		case REL_OFFSET:
			addend -= (size_t)reloc_addr;
		case REL_SYMBOLIC:
		case REL_GOT:
		case REL_PLT:
			*reloc_addr = sym_val + addend;
			break;
		case REL_USYMBOLIC:
			memcpy(reloc_addr, &(size_t){sym_val + addend}, sizeof(size_t));
			break;
		case REL_RELATIVE:
			*reloc_addr = (size_t)base + addend;
			break;
		case REL_SYM_OR_REL:
			if (sym) *reloc_addr = sym_val + addend;
			else *reloc_addr = (size_t)base + addend;
			break;
		case REL_COPY:
			memcpy(reloc_addr, (void *)sym_val, sym->st_size);
			break;
		case REL_OFFSET32:
			*(uint32_t *)reloc_addr = sym_val + addend
				- (size_t)reloc_addr;
			break;
		case REL_FUNCDESC:
			*reloc_addr = def.sym ? (size_t)(def.dso->funcdescs
				+ (def.sym - def.dso->syms)) : 0;
			break;
		case REL_FUNCDESC_VAL:
			if ((sym->st_info&0xf) == STT_SECTION) *reloc_addr += sym_val;
			else *reloc_addr = sym_val;
			reloc_addr[1] = def.sym ? (size_t)def.dso->got : 0;
			break;
		case REL_DTPMOD:
			*reloc_addr = def.dso->tls_id;
			break;
		case REL_DTPOFF:
			*reloc_addr = tls_val + addend - DTP_OFFSET;
			break;
#ifdef TLS_ABOVE_TP
		case REL_TPOFF:
			*reloc_addr = tls_val + def.dso->tls.offset + TPOFF_K + addend;
			break;
#else
		case REL_TPOFF:
			*reloc_addr = tls_val - def.dso->tls.offset + addend;
			break;
		case REL_TPOFF_NEG:
			*reloc_addr = def.dso->tls.offset - tls_val + addend;
			break;
#endif
		default:
			error("Error relocating %s: unsupported relocation type %d",
				dso->name, type);
			if (runtime) longjmp(*rtld_fail, 1);
			continue;
		}
```

此时链接器的可重定位符号信息已经被修正了。



**13. 处理TLS(Thread Local Storage，线程局部存储)相关的问题。**


```c
/* Setup early thread pointer in builtin_tls for ldso/libc itself to
	 * use during dynamic linking. If possible it will also serve as the
	 * thread pointer at runtime. */
	search_vec(auxv, &__hwcap, AT_HWCAP);
	libc.auxv = auxv;
	libc.tls_size = sizeof builtin_tls;
	libc.tls_align = tls_align;
	if (__init_tp(__copy_tls((void *)builtin_tls)) < 0) {
		a_crash();
	}
```

用于解决全局/静态变量在多线程环境下的共享冲突问题，确保线程内部数据独立访问且不影响其他线程。         
其核心思想是为每个线程分配专属的变量存储空间，使变量仅对当前线程可见，从而避免同步开销和数据竞争。

```c
int __init_tp(void *p)
{
	pthread_t td = p;
	td->self = td;
	int r = __set_thread_area(TP_ADJ(p));
	if (r < 0) return -1;
	if (!r) libc.can_do_threads = 1;
	td->detach_state = DT_JOINABLE;
	td->tid = __syscall(SYS_set_tid_address, &__thread_list_lock);
	td->locale = &libc.global_locale;
	td->robust_list.head = &td->robust_list.head;
	td->sysinfo = __sysinfo;
	td->next = td->prev = td;
	return 0;
}
```

其中在上面函数中初始化线程指针``$tp``，也就是tp会在libc库中被设定和初始化，并不是在内核。

```asm
.global __set_thread_area
.hidden __set_thread_area
.type   __set_thread_area,@function
__set_thread_area:
	move $tp, $a0
	move $a0, $zero
	jr   $ra
```

然后跳转到``__dls3``函数处执行。

**14. 执行__dls3，也就是处理依赖和修正可重定位项，此时是为主程序，也就是我们说的``hello``程序，     
上述的执行过程是个链接器ld.so完成了可重定位。**

处理hello的参数，环境变量，auxv表等。

```c
char **argv_orig = argv;
	char **envp = argv+argc+1;

	/* Find aux vector just past environ[] and use it to initialize
	 * global data that may be needed before we can make syscalls. */
	__environ = envp;
	decode_vec(auxv, aux, AUX_CNT);
	search_vec(auxv, &__sysinfo, AT_SYSINFO);
```

加载主程序app的依赖库（.so）

```c
	/* Load preload/needed libraries, add symbols to global namespace. */
	ldso.deps = (struct dso **)no_deps;
	if (env_preload) load_preload(env_preload);
 	load_deps(&app);
```

其核心就是函数``load_library``，可在[此处](https://github.com/ifduyue/musl/blob/master/ldso/dynlink.c)查看源码

它会递归的加载所有依赖的so文件，并且进行解析符号，重定位等等信息。然后保存在内部的链表中。


**15. 处理主程序（hello）的重定位相关内容**

```c
	/* The main program must be relocated LAST since it may contain
	 * copy relocations which depend on libraries' relocations. */
	reloc_all(app.next);
	reloc_all(&app);
```

核心还是调用上述``do_relocs``函数处理。


**16. 上述完成了，已经完成了可执行文件的基本所需，下面跳转到真正的主程序的入口处**

```c
#define CRTJMP(pc,sp) \
   __asm__ __volatile__( \
	"move $sp, %1; 
	jr %0" 
	: 
	: "r"(pc), "r"(sp) 
	: "memory" )
```

```c
	CRTJMP((void *)aux[AT_ENTRY], argv-1);
	for(;;);
```

此时hello主程序的入口地址保存在``aux[AT_ENTRY]``中，直接跳转过去执行入口函数。


**17. 此时hello的入口函数时_start函数**

```c
__asm__(
".text \n"
".global " START "\n"
".type   " START ", @function\n"
START ":\n"
"	move $fp, $zero\n"
"	move $a0, $sp\n"
".weak _DYNAMIC\n"
".hidden _DYNAMIC\n"
"	la.local $a1, _DYNAMIC\n"
"	bstrins.d $sp, $zero, 3, 0\n"
"	b " START "_c\n"
);
```

而此时的跳转函数是``_start_c``。
```c
void _start_c(long *p)
{
	int argc = p[0];
	char **argv = (void *)(p+1);
	__libc_start_main(main, argc, argv, _init, _fini, 0);
}
```

**18. libc库的初始化函数``__libc_start_main``**

```c
int __libc_start_main(int (*main)(int,char **,char **), int argc, char **argv,
	void (*init_dummy)(), void(*fini_dummy)(), void(*ldso_dummy)())
{
	char **envp = argv+argc+1;

	/* External linkage, and explicit noinline attribute if available,
	 * are used to prevent the stack frame used during init from
	 * persisting for the entire process lifetime. */
	__init_libc(envp, argv[0]);

	/* Barrier against hoisting application code or anything using ssp
	 * or thread pointer prior to its initialization above. */
	lsm2_fn *stage2 = libc_start_main_stage2;
	__asm__ ( "" : "+r"(stage2) : : "memory" );
	return stage2(main, argc, argv);
}
```

具体的代码可查看[此处](https://github.com/ifduyue/musl/blob/master/src/env/__libc_start_main.c)。

```c
void __init_libc(char **envp, char *pn)
{
	size_t i, *auxv, aux[AUX_CNT] = { 0 };
	__environ = envp;
	for (i=0; envp[i]; i++);
	libc.auxv = auxv = (void *)(envp+i+1);
	for (i=0; auxv[i]; i+=2) if (auxv[i]<AUX_CNT) aux[auxv[i]] = auxv[i+1];
	__hwcap = aux[AT_HWCAP];
	if (aux[AT_SYSINFO]) __sysinfo = aux[AT_SYSINFO];
	libc.page_size = aux[AT_PAGESZ];

	if (!pn) pn = (void*)aux[AT_EXECFN];
	if (!pn) pn = "";
	__progname = __progname_full = pn;
	for (i=0; pn[i]; i++) if (pn[i]=='/') __progname = pn+i+1;

	__init_tls(aux);
	__init_ssp((void *)aux[AT_RANDOM]);

	if (aux[AT_UID]==aux[AT_EUID] && aux[AT_GID]==aux[AT_EGID]
		&& !aux[AT_SECURE]) return;

	struct pollfd pfd[3] = { {.fd=0}, {.fd=1}, {.fd=2} };
	int r =
#ifdef SYS_poll
	__syscall(SYS_poll, pfd, 3, 0);
#else
	__syscall(SYS_ppoll, pfd, 3, &(struct timespec){0}, 0, _NSIG/8);
#endif
	if (r<0) a_crash();
	for (i=0; i<3; i++) if (pfd[i].revents&POLLNVAL)
		if (__sys_open("/dev/null", O_RDWR)<0)
			a_crash();
	libc.secure = 1;
}
```

**19. 执行musl libc的阶段2初始化**

```c
static int libc_start_main_stage2(int (*main)(int,char **,char **), int argc, char **argv)
{
	char **envp = argv+argc+1;
	__libc_start_init();

	/* Pass control to the application */
	exit(main(argc, argv, envp));
	return 0;
}
```

```c
void __libc_start_init(void)
{
	do_init_fini(main_ctor_queue);
	if (!__malloc_replaced && main_ctor_queue != builtin_ctor_queue)
		free(main_ctor_queue);
	main_ctor_queue = 0;
}
```

之后，才执行``main(argc, argv, envp)``程序开发者的入口函数``main``函数。

**20. 执行完main之后，执行exit**

```c
_Noreturn void exit(int code)
{
	__funcs_on_exit();
	__libc_exit_fini();
	__stdio_exit();
	_Exit(code);
}
```
退出时，执行libc库相关的一下回收工作。

最后执行系统调用，退出当前的程序。

```c
_Noreturn void _Exit(int ec)
{
	__syscall(SYS_exit_group, ec);
	for (;;) __syscall(SYS_exit, ec);
}
```

此时完成了所有用户空间的执行操作。接下来就是操作系统处理退出的事情。

------------------------

下面的代码又返回到了linux内核空间。


**21. 调用系统调用，进入内核执行do_sys_exit**

```c
SYSCALL_DEFINE1(exit, int, error_code)
{
	do_exit((error_code&0xff)<<8);
}
```

处理信号，回收内存，关闭文件等待操作。具体可看[此处内核源码](https://github.com/torvalds/linux/blob/master/kernel/exit.c)。

