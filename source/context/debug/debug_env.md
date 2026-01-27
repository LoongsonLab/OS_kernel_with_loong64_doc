# LoongArch最小debug环境搭建

## 制作Linux RamDisk

请参考文档第一章，安装 QEMU 。

请参考文档第一章，编译 Linux Kernel 。

使用以下命令，在 QEMU 中启动 Linux Kernel。
``` bash
#	replace your real path with {/path/to/qemu}.
{/path/to/qemu}/bin/qemu -M virt -cpu la464 -m 4G -smp 1 -nographic -kernel ./vmlinux -serial mon:stdio
```

## 如何在 QEMU 中调试

QEMU 常用的启动参数，如下所示:

1. 设备类型: -machine/-M

在qemu中，不同的指令集的模拟器会编译成不同的可执行文件，可以运行在不同的平台上，可使用 -machine/-M 指定模拟器运行的设备信息。
```shell
$ qemu -M ?

Supported machines are:
none                 empty machine
virt                 Loongson-3A5000 LS7A1000 machine (default)

```
2. 内存大小: -m

参数-m 1G就是指定虚拟机内部的内存大小为1GB，使用说明如下:

```shell
$ qemu -m ?

qemu-system-loongarch64: -m ?: Parameter 'size' expects a non-negative number below 2^64
Optional suffix k, M, G, T, P or E means kilo-, mega-, giga-, tera-, peta-
and exabytes, respectively.
```

3. 核心数: -smp

现代cpu往往是对称多核心的，因此通过添加启动参数 -smp 8 可以指定虚拟机核心数为 8。

```shell
$ qemu -smp ?

smp-opts options:
  books=<num>
  clusters=<num>
  cores=<num>
  cpus=<num>
  dies=<num>
  drawers=<num>
  maxcpus=<num>
  sockets=<num>
  threads=<num>
```

4. 关闭图像输出: -nographic

参数关闭了图像输出模式，QEMU 运行时不再弹出新窗口，信息输入输出，通过 serial 串口，在终端显示交互。

5. 调试参数: -s -S

参数选项用于建立 gdb 服务。

其中，-s 选项会让 QEMU 在 TCP 端口 1234 监听来自 gdb 的传入连接，-S 选项会使 QEMU 在启动后，不立即运行 guest ，而是等待主机 gdb 发起连接。

6. 6.指定镜像: -kernel

熟悉上述参数后，可在 QEMU 时添加对应参数，进入调试模式。

首先需要在启动 Linux Kernel 时加入额外参数。

参数指定传入 QEMU 的内核的镜像文件，一般是ELF文化，也可以是uImage等。

``` bash
#	replace your real path with {/path/to/qemu}.
{/path/to/qemu}/bin/qemu -M virt -cpu la464 -m 4G -smp 1 -nographic -kernel ./vmlinux -serial mon:stdio -S -s
```

打开一个新终端，并在终端中运行以下命令:

``` bash
gdb ./vmlinux
```

进入 gdb 程序后，使用以下命令，与 QEMU 进行连接。
``` bash
(gdb)target remote localhost:1234
```

:::{note}
端口 1234 是 QEMU 默认的 gdb 连接端口，特殊情况下，当该端口不可用时，可使用 -gdb tcp::xxxx 来代替 QEMU 启动参数 -s ，其中 xxxx 代表可使用的新端口。

同时，在 gdb 启动时候，使用 target remote localhost:xxxx 来与 QEMU 建立连接，同样，xxxx 代表可使用的新端口，并与 QEMU 启动时配置的新端口相同。
:::

## 设置断点

在 gdb 界面，可以使用以下命令设置断点:
``` shell
#	replace _start with all function name which you need
(gdb)b _start
#	or input any va 
(gdb)b *0x9000000000200000
```
输入命令，使 QEMU 开始运行 guest :
``` shell
(gdb)continue
```
当 PC 运行到对应的函数、或对应地址，QEMU 将暂停运行。