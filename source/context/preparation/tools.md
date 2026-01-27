# 工具链说明

本章简要介绍，在为龙芯架构开发系统内核时，使用的编译工具链。

其中，工具链具体原理与详细说明，请以文档第二章为主。

## GCC

1、龙芯生态的新世界与旧世界。

龙芯目前有两套生态, 社区常常使用新世界和旧世界进行区分。旧世界兼容`MIPS`生态，新世界增加了更多的LoongArch架构特性，两个世界并不兼容。一般而言，开发者需要使用一整套同一个世界的工具链。

以下是新旧世界下，一些常见软件、开发工具的版本对比：

:::{list-table} 
:widths: 30 20 20
:header-rows: 0

*  -  Linux
   -  4.19
   -  >= 5.19
*  -  binutils
   -  2.31
   -  >= 2.38
*  -  gcc
   -  8.3
   -  >= 12.1
*  -  glibc
   -  2.28
   -  >= 12.1
*  -  LLVM
   -  8
   -  >= 16
*  -  Node.js
   -  14.16.1
   -  >= 18
*  -  Go
   -  1.15,1.18
   -  >= 1.19
*  -  Rust
   -  1.41,1.58
   -  >= 1.71
:::

2、编译器选项

编译器支持两类命令行选项。

-  基础选项:选择编译目标的基本配置，包括 -march -mabi -mtune;
-  扩展选项:对基础选项或基础选项默认值的配置进行增量调整。

基础选项内容与可选值如下：

:::{list-table} 基础选项
:widths: 20 30 30
:header-rows: 1

*  - **选项**
   - **可用值**
   - **描述**
*  - -march=
   - native,loongarch64,la464
   - 设定可用的指令集和寄存器范围
*  - -mabi=
   - lp64d,lp64f,lf64s,ilp32d,ilp32f,ilp32s
   - 选择基础ABI类型
*  - -mtune=
   - native,loongarch64,la464
   - 设定魏家沟相关的性能调优参数
:::

扩展选项内容与可选值如下：

:::{list-table} 基础选项
:widths: 20 30 30
:header-rows: 1

*  - **选项**
   - **可用值**
   - **描述**
*  - -msoft-float
   - 
   - 禁止使用浮点指令，ABI使用软浮点类型(后缀为s)
*  - -msingle-float
   - 
   - 允许使用32位浮点指令，ABI使用32位浮点类型(后缀为f)
*  - -mdouble-float
   - 
   - 允许使用32位和64位浮点指令，ABI使用64位浮点类型(后缀为d)
*  - -mfpu=
   - 64,32,0,none
   - 选择可用的浮点指令和浮点寄存器范围。
:::

3、musl和glibc

针对不同的ABI，其支持的 C 库，与对应脚骨标识符，如下所示：

:::{list-table} 
:widths: 20 20 20
:header-rows: 1

*  - **ABI类型/扩展特性**
   - **C库**
   - **架构标识符**
*  - lp64d/base
   - glibc
   - loongarch64-linux-gnu
*  - lp64f/base
   - glibc
   - loongarch64-linux-gnuf32
*  - lp64s/base
   - glibc
   - loongarch64-linux-gnusf
*  - lp64d/base
   - musl,libc
   - loongarch64-linux-musl
*  - lp64f/base
   - musl,libc
   - loongarch64-linux-muslf32
*  - lp64s/base
   - musl,libc
   - loongarch64-linux-muslsf
:::

## LLVM

LLVM 自 16.0.0 版本开始，以正式后端（official target）的级别实现了对 LoongArch 指令集架构的完善支持。

## Rust

Rust 自 1.71.0 版本开始，实现对龙架构（LoongArch）指令集的原生支持。

目前 Rust 支持的 LoongArch Linux 目标平台，包括：loongarch64-unknown-linux-gnu 与 loongarch64-unknown-linux-musl ， 支持原生构建和交叉构建，可通过 rustup 管理使用。


# 模拟器

## QEMU

QEMU 是一个通用且开源的机器模拟器和虚拟化程序，可以运行为特定机器（例如 Loongson 3A5000 ）编写的操作系统和程序，在不同的机器（例如 x86 PC）上运行。

同事，通过使用动态翻译，QEMU 可以使用 Xen 或 KVM 等其他虚拟机监控程序来利用 CPU 扩展（HVM）进行虚拟化。

这样，运行操作系统和程序时，QEMU 通过在主机 CPU 上直接执行客户机代码来实现接近原生的性能，达到良好性能。

QEMU 提供全系统仿真和用户模式仿真两种模式。

-  全系统仿真：QEMU 模拟一个完整的系统，包括一个或多个处理器和各种外围设备。它更准确但速度较慢。
-  用户模式仿真：QEMU 能够通过利用主机系统资源，运行不同架构编译的 Linux 可执行文件。

可使用以下命令进行安装:

``` shell
// install Dependency
sudo apt-get install libglib2.0-0 libglib2.0-dev
sudo apt install libpixman-1-0 libpixman-1-dev
sudo apt install flex
sudo apt install bison ninja-build
sudo apt install gcc g++
// clone Repository
git clone https://github.com/qemu/qemu.git
// install
cd qemu/
mkdir build
cd build
../configure
// sudo if necesarry
make
make install
```
即可运行 QEMU 。


## LA_EMU

LA_EMU 为 LoongArch64 模拟器，支持整型、浮点、向量指令集。模拟器能够启动 Linux kernel，并运行大型测试。

可使用以下命令进行安装:
``` bash
// clone Repository
git clone https://github.com/Open-ChipHub/LA_EMU.git
// compile
make
```
编译完成后，在 `build/`目录下，生成两个可执行程序：`la_emu_kernel`与`la_emu_user`，分别支持系统模式仿真和用户模式仿真。


LA_EMU 运行时，支持的参数选项包括:
``` bash
la_emu_kernel -m n[G] -k kernel
-m Memory size(kernel mode)
-k Kernel vmlinux or checkpoint directory(kernel mode)
-d Log info, support: exec,cpu,fpu,int
-D Log file
-c Check item, support: tlb_mhit
-z Determined events
-g Enable gdbserver
-w Force enable hardware page table walker
```

系统模式下，LA_EMU 常用运行命令如下:
``` bash
./build/la_emu_kernel -w -z -n -m 8 -k ~/linux/vmlinux
```
该命令可在 LA_EMU 系统模式下，启动 Linux kernel。


# 操作系统编译

本章以多个开源操作系统为示例，展示LoongArch架构的操作系统编译详细步骤，并在`QEMU`上启动。

首先准备开发环境。可使用以下命令安装交叉编译器:
```shell
wget https://github.com/loongson/build-tools/releases/download/2023.08.08/x86_64-cross-tools-loongarch64-gcc-libc.tar.xz
tar xvf ./x86_64-cross-tools-loongarch64-gcc-libc.tar.xz
// set env
export CROSS_COMPILE=loongarch64-unknown-linux-gnu-
// replace your real path of cross compiler with {/path/of/cross/compile}
export PATH={/path/of/cross/compile}/bin:$PATH
```
使用以下命令检查软件安装情况:
``` shell
loongarch64-unknown-linux-gnu-gcc --version
// print version on terminal.
```

## LoongNix

## UOS

## Deepin

1. 下载 Deepin 安装镜像，可使用以下命令:
``` shell
wget https://mirrors.hust.edu.cn/deepin-cd/25.0.10/loong64/deepin-desktop-community-25.0.10-loong64.iso
```
2. 创建虚拟机磁盘。

首先创建一个 100GB 大小的磁盘，可使用以下命令:
``` shell
/emulator/qemu/bin/qemu-img create -f qcow2 ./deepin-v23.qcow2 100G
```
运行一下命令，启动系统:
``` shell
~/emulator/qemu/bin/qemu-system-loongarch64 -m 8G -cpu la464-loongarch-cpu -machine virt -smp 4 -bios ../QEMU_EFI.fd -serial stdio -device virtio-gpu-pci -net nic -device virtio-blk-pci,drive=drive-virtio-disk0 -drive id=drive-virtio-disk0,if=none,format=raw,file=deepin-v23.qcow2 -device virtio-scsi-pci,id=scsi0 -drive id=drive-scsi0-cdrom0,if=none,format=raw,readonly=on,file=deepin-desktop-community-25.0.10-loong64.iso -device scsi-cd,bus=scsi0.0,drive=drive-scsi0-cdrom0
```
``` shell

```

## AOSC(安同)

1. 在终端中运行该指令下载安同os镜像:

```bash
$ wget https://releases.aosc.io/os-loongarch64/installer/aosc-os_installer_20251206_loongarch64.iso
```

2. 创建映像文件

在虚拟机终端中运行如下命令，建立一个 20GiB 大小的硬盘映像文件 aosc.img 供qemu挂载使用   
```bash
$ sudo qemu-img create -f raw aosc.img 20G
```

3. 为映像文件建立分区

随后为 aosc.img 建立分区
```bash  
$ sudo fdisk aosc.img

Welcome to fdisk (util-linux 2.37.2).   
Changes will remain in memory only, until you decide to write them.   
Be careful before using the write command.   
   
Device does not contain a recognized partition table.   
Created a new DOS disklabel with disk identifier 0x0f513192.

Command (m for help): n   
Partition type   
   p   primary (0 primary, 0 extended, 4 free)   
   e   extended (container for logical partitions)   
Select (default p): p   
Partition number (1-4, default 1): 1   
First sector (2048-41943039, default 2048):    
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943039, default 41943039): 
   
Created a new partition 1 of type 'Linux' and of size 20 GiB.
   
Command (m for help): w   
The partition table has been altered.   
Syncing disks.
```   
打印目前的分区表   
```bash  
$ fdisk -l aosc.img

Disk aosc.img: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe25655f6

Device     Boot Start      End  Sectors Size Id Type
aosc.img1        2048 41943039 41940992  20G 83 Linux
```   
4. 创建Makefile文件

创建一个Makefile文件   
`$ vim Makefile`   
在其中编写   
```makefile
QEMU  = /your/qemu/path/qemu-system-loongarch64
MEM   = 4G
CORES = 4
CPU   = la464
MACH  = virt
BIOS  = /your/EFI.fd/path/QEMU_EFI.fd

DISK  = aosc.img
# ISO名字可能会随着版本变化而改变，更改为最新下载版本的名字即可
ISO   = aosc-os_installer_20251206_loongarch64.iso

.PHONY: run run-hd create clean

run:
        $(QEMU) -m $(MEM) -smp $(CORES) -cpu $(CPU) -machine $(MACH) -bios $(BIOS) \
                -serial stdio -device virtio-gpu-pci -net nic -net user \
                -device virtio-blk-pci,drive=drive-virtio-disk0 \
                -drive id=drive-virtio-disk0,if=none,format=raw,file=$(DISK) \
                -device virtio-scsi-pci,id=scsi0 \
                -drive id=drive-scsi0-cdrom0,if=none,format=raw,readonly=on,file=$(ISO) \
                -device scsi-cd,bus=scsi0.0,drive=drive-scsi0-cdrom0

run-hd:
        $(QEMU) -m $(MEM) -smp $(CORES) -cpu $(CPU) -machine $(MACH) -bios $(BIOS) \
                -serial stdio -device virtio-gpu-pci -net nic -net user \
                -drive file=$(DISK),format=raw,if=virtio

create:
        qemu-img create -f raw $(DISK) 20G

clean:
        rm -f $(DISK)
```   
5. 使用qemu运行安同os   
此时文件夹下有三个文件，分别是：  
```
aosc.img      
aosc-os_installer_20251206_loongarch64.iso   
Makefile
```   
使用`sudo make run`命令运行   
屏幕上会弹出安同os的界面  
如果无法在安同os界面操作，那就返回到终端中进行操作选择  

```{image} ../../_static/aosc_language.png
:alt: aosc 选择配置界面
:class: bg-primary
:scale: 50 %
:align: center
```

第一个界面选择 中文（简体）  

```{image} ../../_static/aosc_grub.png
:alt: aosc 选择启动项界面
:class: bg-primary
:scale: 50 %
:align: center
```

第二个界面选择 试用AOSC OS桌面版   
稍等片刻即可进入安同os桌面  

```{image} ../../_static/aosc_desktop.png
:alt: aosc 桌面版图片
:class: bg-primary
:scale: 50 %
:align: center
```

## Linux kernel

linux内核版本推荐6.10。

首先，基于busybox制作最小根文件系统。

可使用以下命令获取对应文件。
``` shell
wget https://github.com/Open-ChipHub/LoongArch-SDK/raw/refs/heads/main/rootfs/rootfs.cpio.gz
mv ./rootfs.cpio.gz /path/to/your/linux/directory/
```

也可使用以下命令，重新制作。

首先获取busybox。
```shell
# 下载tar包
wget https://busybox.net/downloads/busybox-1.33.0.tar.bz2
tar -xjf ./busybox-1.33.0.tar.bz2
cd busybox-1.33.0
# 或从git仓库下载
git clone git://git.busybox.net/busybox
cd busybox
```
使用以下命令进行编译。
``` shell
# 安装依赖
sudo apt install libncurses5-dev ncurses-devel
make arch=loongarch CROSS_COMPILE=loongarch64-linux-gnu- defconfig
make arch=loongarch CROSS_COMPILE=loongarch64-linux-gnu- menuconfig
# 根据需求进行配置
make -j$(nproc)
# 生成到新建目录rootfs
make install CONFIG_PREFIX={/path/to/rootfs}
```
在rootfs目录下，进行以下命令。
``` shell
cd {/path/to/rootfs}
mkdir -p proc sys dev etc/init.d
vim etc/init.d/rcS
```
修改etc/init.d/rcS为以下内容
``` shell
mount -t proc none /proc
mount -t sysfs nont /sys
exec /bin/sh
```
进行以下命令。
``` shell
chmod +x etc/init.d/rcS
ln -sf bin/busybox init
# 打包为initramfs
find . | cpio -o -H newc | gzip > ../rootfs.cpio.gz
```

接下来编译linux kernel

可使用以下命令进行获取。
``` shell
# 下载tar.gz包
wget https://www.kernel.org/pub/linux/kernel/v6.x/linux-6.10.tar.gz
tar zxvf linux-6.10.tar.gz -C {/path/of/linux/}
cd linux-6.10
# 或从git仓库下载
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git checkout v6.10
```
使用以下命令，配置内核对应的设备数文件，与内核配置文件。
```shell
# 可手动从github仓库下载该文件，或使用以下命令
wget https://raw.githubusercontent.com/Open-ChipHub/LoongArch-SDK/refs/heads/main/dts/labcore-sim.dts
mv ./labcore-sim.dts {path/to/linux}/arch/loongarch/boot/dts/
wget https://raw.githubusercontent.com/Open-ChipHub/LoongArch-SDK/refs/heads/main/linux/labcore_defconfig
mv ./labcore_defconfig {path/to/linux}/.config
```

使用以下命令，对linux kernel进行编译。
``` shell
cp {path/to/rootfs/}/rootfs.cpio.gz ./
# 请确保交叉编译器环境变量设置正确
make ARCH=loongarch CROSS_COMPILE=loongarch64-linux-gnu- menuconfig
```
在`[General setup]`选项中，选择`[Initial RAM filesystem and RAM disk (initramfs/initrd) support]`。在`[Initramfs source file(s)]`中输入`[rootfs.cpio.gz]`。


返回上一页，在`[Kernel type and options]`选项中，选中`[Enable built-in dtb in kernel]`，在`[Source file for built-in dtb]`中输入`[labcore-sim]`。


保存并退出。

使用以下命令，进行编译。
``` shell
# 请确保交叉编译器环境变量设置正确
make ARCH=loongarch CROSS_COMPILE=loongarch64-linux-gnu- -j$(nproc)
```
部分编译过程以及结果如下所示。

## 如何自己上手编译一个发行版
   以“勇豹”(Yongbao)为例：https://github.com/sunhaiyong1978/Yongbao.git

