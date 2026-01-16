# LoongArch最小debug环境搭建

## 制作Linux RamDisk

请参考文档第一章，安装 QEMU 。

请参考文档第一章，编译 Linux Kernel 。

使用以下命令，在 QEMU 中启动 Linux Kernel。
``` bash
#	replace your real path with {/path/to/qemu}.
{/path/to/qemu}/bin/qemu-system-loongarch64 -M virt -cpu la464 -m 4G -smp 1 -nographic -kernel ./vmlinux -serial mon:stdio
```

## 如何在 QEMU 中调试

在 QEMU 中进入调试模式，首先需要在启动 Linux Kernel 时加入额外参数。

``` bash
#	replace your real path with {/path/to/qemu}.
{/path/to/qemu}/bin/qemu-system-loongarch64 -M virt -cpu la464 -m 4G -smp 1 -nographic -kernel ./vmlinux -serial mon:stdio -S -s
```

打开一个新终端，并在终端中运行以下命令:

``` bash
#	set env
gdb ./vmlinux
```

gdb 成功运行后，

## 设置断点