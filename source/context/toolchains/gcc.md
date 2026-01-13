# GCC

GCC（GNU Compiler Collection）是全球知名的**自由软件编译器套件**，最初为GNU操作系统开发，现已成为跨平台软件开发的核心工具之一。其核心功能是支持多种编程语言的编译与优化，广泛应用于操作系统、嵌入式系统、高性能计算等领域。


## GCC的相关概念

### GCC的基本定义与起源
GCC全称为GNU Compiler Collection（GNU编译器集合），由GNU项目开发，遵循GNU通用公共许可证（GPL）发布。它的起源可追溯至1984年，当时理查德·斯托曼（Richard Stallman）为开发GNU系统（一套完全自由的操作系统），需要一个能编译C语言的编译器。最初的GCC仅支持C语言，命名为“GNU C Compiler”，但随着需求扩展，逐渐增加了对C++、Fortran、Ada、Go等多种语言的支持，因此更名为“GNU Compiler Collection”（GNU编译器集合）。[^gnu_gcc]

[^gnu_gcc]: 具体可查看[官网:https://gcc.gnu.org/](https://gcc.gnu.org/)

### GCC支持的语言与平台
GCC支持**多种编程语言**，包括：  
- 系统级语言：C、C++、Fortran、Ada；  
- 面向对象语言：Objective-C、Objective-C++；  
- 现代语言：Go、D；  
- 其他：Java（通过GCJ编译器）、Mercury（逻辑编程）等。

在**平台支持**方面，GCC具备极强的跨平台能力，可运行于：  
- 操作系统：Linux、macOS、Windows（通过MinGW或Cygwin）、Unix（如Solaris、HP-UX）；  
- 硬件架构：x86、x86-64、ARM、MIPS、PowerPC、RISC-V、LoongArch等。


### GCC的核心功能与特点
GCC的核心功能是**将源代码转换为可执行文件**，其工作流程分为四个阶段：  
1. **预处理**：处理宏定义（`#define`）、头文件包含（`#include`）等，生成`.i`文件；  
2. **编译**：将预处理后的代码转换为汇编语言（`.s`文件）；  
3. **汇编**：将汇编代码转换为机器码（`.o`文件，目标文件）；  
4. **链接**：将目标文件与库文件（如C标准库`libc`）链接，生成可执行文件。

GCC的特点包括：  
- **自由软件**：遵循GPL许可证，允许用户自由使用、修改和分发；  
- **高度可定制**：支持多种优化选项（如`-O2`、`-O3`），可根据需求调整编译策略；  
- **多语言支持**：通过不同的前端（Frontend）处理不同语言，后端（Backend）生成统一的机器码；  
- **跨平台性**：支持几乎所有主流操作系统和硬件架构，是嵌入式系统开发的首选编译器。


### GCC的最新进展
截至2025年，GCC的最新版本为**15.1.0**（2025年5月发布），其主要更新包括：  
- **支持C++23特性**：如模块化标准库（`import std;`），简化了标准库的使用；  
- **性能优化**：改进了代码生成效率，提升了程序的运行速度；  
- **新架构支持**：增加了对RISC-V、LoongArch等新兴架构的支持。


### GCC的应用场景
GCC的应用极其广泛，主要包括：  
- **操作系统开发**：Linux内核、GNU工具的编译均依赖GCC；  
- **嵌入式系统**：ARM、MIPS等架构的嵌入式设备（如手机、路由器）的固件开发；  
- **高性能计算**：科学计算、人工智能（如TensorFlow、PyTorch）的底层代码编译；  
- **桌面应用**：Windows、macOS平台上的C/C++应用程序开发（通过MinGW或Cygwin）。


## 交叉编译器

交叉编译（Cross Compilation）是一种在一种计算机平台（宿主机）上生成另一种不同平台（目标机）可执行程序的技术。其核心在于解决目标平台资源受限或环境不兼容时的开发难题，广泛应用于嵌入式系统、跨平台应用开发等领域。

交叉编译指：在一个平台（Host 架构）上生成另一种平台（Target 架构）可运行的二进制。

> Host 架构：当前运行编译工具链的架构；

> Target 架构：生成的二进制将运行的目标架构；

比如，我们现在在X86_64的机器上使用GCC编译一个C语言程序，让它能够在ARM的机器上能够运行。
此时Host就是X86_64，Target就是ARM。

:::{tip}
我们经常在编译具有configure配置的项目时，当你输入```./configure --help```显示如下：  

System types:  
  --build=BUILD     configure for building on BUILD [guessed]  
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]  
  --target=TARGET   configure for building compilers for TARGET [HOST]  

其中，在大多数情况下，build和host会自动推测，但是target是需要我们手动输入的，比如：  
--build=x86_64-cross-linux-gnu     
--host=x86_64-cross-linux-gnu    
--target=loongarch64-unknown-linux-gnu     

此时，会生成LoongArch有关的二进制文件，但是工具链是运行在x86上。
:::

如果没有特殊的说明，后续我们都是以交叉编译环境为例，即工具链运行在X86_64上面，生产的可执行文件是针对LoongArch架构的，
进行相关的内容介绍。

## 常见的类别组合
我们本节以GCC为例，介绍常用的几种组合。

gcc Target Triplet的格式如下所示[^osdev_gcc]，   
> Machine-Vendor-OperatingSystem

比如Machine对于的是架构，如，X86_64, arm, aarch64, riscv, mips, loongarch64等等

```loongarch64-linux-gnu-gcc```指的是，LoongArch64架构，vender是空（也可以是unknown），OS是Linux，使用的是
glibc库环境。

[^osdev_gcc]: 具体可参考[Target Triplet](https://wiki.osdev.org/Target_Triplet)

我们在在使用

下面我经常使用的组合以及说明：


:::{list-table} LoongArch GCC Target组合
:header-rows: 1

*   - **组合（前缀）**
    - **说明（Description）**
*   - loongarch64-unknown-elf
    - 这是最简单的，适用于嵌入式的组合，其中使用的c库大都是newlib等嵌入是!
*   - loongarch64-unknown-none
    - 和上面的loongarch64-unknown-elf基本差别不大[^gcc_target_none]!
*   - loongarch64-linux-gnu
    - 这是我们常见的组合，OS是Linux，使用的GlibC库，支持生成浮点指令等。 
*   - loongarch64-linux-gnusf
    - OS是Linux，使用的GlibC库，不支持生成浮点指令。
*   - loongarch64-linux-musl
    - 这也是我们常见的组合，OS是Linux，只不过C库使用的是musl C库，支持生成浮点指令等。 
*   - loongarch64-linux-muslsf
    - OS是Linux，C库使用的是musl C库，不支持生成浮点指令等。 
:::

[^gcc_target_none]: 目前在LoongArch平台使用较少，后续会增加此类组合。

下面我们简单的使用helloworld来简单的说明交叉编译器的使用，以及初级查看生成的可执行文件。

至于工具链的下载和使用，我们后续章节有更加详细的使用说明，这里只是做一个演示。

```c
#include <stdio.h>
int main(int argc, char const *argv[])
{
   printf("Hello LoongArch\n");
   return 0;
}
```


编译成一个使用musl库，禁止使用生成浮点指令，动态的ELF可执行文件，命令如下：

```bash
loongarch64-linux-muslsf-gcc main.c -o mainsf.elf
```

我们可以使用file命令查看生成的可执行文件的类型情况：

```bash
shell> file mainsf.elf
mainsf.elf: ELF 64-bit LSB executable, LoongArch, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-loongarch64-sf.so.1, not stripped
```

注意此时动态ELF的解释器是```/lib/ld-musl-loongarch64-sf.so.1```

如果我们使用打开浮点指令生成的交叉编译器时：

```bash
shell> loongarch64-linux-musl-gcc main.c -o main.elf
shell> file main.elf
main.elf: ELF 64-bit LSB executable, LoongArch, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-loongarch64.so.1, not stripped
```

注意此时动态ELF的解释器是```/lib/ld-musl-loongarch64.so.1```


:::{caution}
/lib/ld-musl-loongarch64-sf.so.1链接器和/lib/ld-musl-loongarch64.so.1链接器不能混用，一个Linux环境中可以同时存在。
:::

我们可以使用链接器来判断这个***动态的可执行文件ELF***是否是支持浮点指令的，但是我们还有更加稳妥的更加常用的方式
使用工具```readelf```

查看禁止生成浮点指令的可执行文件：
```bash
shell> loongarch64-linux-musl-readelf mainsf.elf

#显示如下：
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2\'s complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           LoongArch
  Version:                           0x1
  Entry point address:               0x120000540
  Start of program headers:          64 (bytes into file)
  Start of section headers:          67672 (bytes into file)
  Flags:                             0x41, SOFT-FLOAT, OBJ-v1
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         8
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23

```

查看支持生成浮点指令的可执行文件：
``` bash 
shell> loongarch64-linux-musl-readelf main.elf
#显示如下：
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2\'s complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           LoongArch
  Version:                           0x1
  Entry point address:               0x120000540
  Start of program headers:          64 (bytes into file)
  Start of section headers:          67672 (bytes into file)
  Flags:                             0x43, DOUBLE-FLOAT, OBJ-v1
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         8
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23

```

上述两者在生成的```Flags:```字段，会显示不同的提示，

- 软浮点（禁止生成浮点指令）Flags： 0x41, SOFT-FLOAT, OBJ-v1
- 软浮点（支持生成浮点指令）Flags： 0x43, DOUBLE-FLOAT, OBJ-v1


## 如何编译交叉编译器

本小节会教大家如何从gcc等源码编译我们使用的交叉编译器。

### 如何编译loongarch64-linux-gnu[sf]

配置说明：

:::{list-table} 相关编译工具链说明
:header-rows: 1
*   - **版本说明**
    - **下载链接**

*   - Linux Kernel头文件版本为6.7
    - [https://mirrors.edge.kernel.org/pub/linux/kernel/v6.x/](https://mirrors.edge.kernel.org/pub/linux/kernel/v6.x/)

*   - BINUTILS版本： binutils-2.45
    - [https://ftp.gnu.org/gnu/binutils/](https://ftp.gnu.org/gnu/binutils/)

*   - GMP版本： gmp-6.3.0
    - [https://ftp.gnu.org/gnu/gmp/](https://ftp.gnu.org/gnu/gmp/)

*   - MPFR版本： mpfr-4.2.2
    - [https://ftp.gnu.org/gnu/mpfr/](https://ftp.gnu.org/gnu/mpfr/)

*   - MPC版本： mpc-1.3.1
    - [https://ftp.gnu.org/gnu/mpc](https://ftp.gnu.org/gnu/mpc/)

*   - GLIBC版本： glibc-2.42
    - [https://ftp.gnu.org/gnu/glibc/](https://ftp.gnu.org/gnu/glibc/)

*   - GLIBC版本： gcc-15.2.0
    - [https://ftp.gnu.org/gnu/gcc/gcc-15.2.0](https://ftp.gnu.org/gnu/gcc/gcc-15.2.0)

*   - GDB版本： gdb-16.3
    - [https://ftp.gnu.org/gnu/gdb/](https://ftp.gnu.org/gnu/gdb/)

:::

下面逐一演示工具链的制作过程。   
首先我们定义几个常用的变量（GCC支持生成浮点指令）, 下面显示的命令规则是在Makefile中：
```Makefile
SYSDIR=$(shell pwd)
CROSS_HOST=$(echo $MACHTYPE | sed "s/$(echo $MACHTYPE | cut -d- -f2)/cross/") #x86_64-cross-linux-gnu
CROSS_TARGET=loongarch64-linux-gnu #表示我们的目标组合
MABI=lp64d
BUILD64=-mabi=lp64d
CROSS_BUILD_DIR=cross-tools-all-202612 #用于安装交叉编译器的目录在：$(SYSDIR)/$(CROSS_BUILD_DIR)
```

#### linux header的安装
``` Makefile
LINUX_DIR := linux-6.7
linux_header:
   cd $(LINUX_DIR) && make mrproper
   cd $(LINUX_DIR) && make ARCH=loongarch INSTALL_HDR_PATH=dest headers_install
   cd $(LINUX_DIR) && find dest/include -name '.*' -delete
   cd $(LINUX_DIR) && mkdir -pv ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/usr/include
   cd $(LINUX_DIR) && cp -rv dest/include/* ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/usr/include
```

编译完成后会在```cross-tools-all-202612/sysroot/usr/include```安装相关的头文件等

#### binutils的编译和安装
``` Makefile
BINUTILS  := binutils-2.45
binutils:
   cd $(BINUTILS) && rm -rf gdb* libdecnumber readline sim
   cd $(BINUTILS) && rm -rf tools-build && mkdir tools-build
   cd $(BINUTILS)/tools-build && CC=gcc AR=ar AS=as \
   ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) --build=${CROSS_HOST} --host=${CROSS_HOST} \
                    --target=${CROSS_TARGET} --with-sysroot=${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot --disable-nls \
                    --disable-static --enable-64-bit-bfd
   cd $(BINUTILS)/tools-build && make configure-host ${JOBS}
   cd $(BINUTILS)/tools-build && make ${JOBS}
   cd $(BINUTILS)/tools-build && make install-strip
   cd $(BINUTILS)/tools-build && cp -v ../include/libiberty.h ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/usr/include

```

#### GMP的编译和安装
```Makefile
GMP     := gmp-6.3.0
gmp:
   cd $(GMP) && rm -rf build && mkdir -p build
   cd $(GMP)/build && ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) --enable-cxx --disable-static
   cd $(GMP)/build && make ${JOBS}
   cd $(GMP)/build && make install
```

#### MPFR的编译和安装
```Makefile
MPFR    := mpfr-4.2.2
mpfr:
   cd $(MPFR) && rm -rf build && mkdir -p build
   cd $(MPFR)/build && ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) --disable-static --with-gmp=${SYSDIR}/$(CROSS_BUILD_DIR)
   cd $(MPFR)/build && make ${JOBS}
   cd $(MPFR)/build && make install
```

#### MPC的编译和安装
```Makefile
MPC     := mpc-1.3.1
mpc:
   cd $(MPC) && rm -rf build && mkdir -p build
   cd $(MPC)/build && ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) --disable-static --with-gmp=${SYSDIR}/$(CROSS_BUILD_DIR)
   cd $(MPC)/build && make ${JOBS}
   cd $(MPC)/build && make install
```

#### 简易版GCC的编译和安装
制作交叉编译器中的GCC，第一次编译交叉工具链的GCC需要采用精简方式进行编译和安装，否则会因为缺少目标系统的C库而导致部分内容编译链接失败
```Makefile
GCC       := gcc-15.2.0
gcc-simp:
   cd $(GCC) && rm -rf tools-build && mkdir tools-build
   cd $(GCC)/tools-build && AR=ar LDFLAGS="-Wl,-rpath,${SYSDIR}/$(CROSS_BUILD_DIR)/lib" \
   ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) --build=${CROSS_HOST} --host=${CROSS_HOST} \
      --target=${CROSS_TARGET} --disable-nls \
      --with-mpfr=${SYSDIR}/$(CROSS_BUILD_DIR) --with-gmp=${SYSDIR}/$(CROSS_BUILD_DIR) \
      --with-mpc=${SYSDIR}/$(CROSS_BUILD_DIR) \
      --with-sysroot=${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot \
      --disable-decimal-float --disable-libgomp --disable-libitm \
      --disable-libsanitizer --disable-libquadmath --enable-threads=posix \
      --disable-target-zlib --with-system-zlib --enable-checking=release \
      --enable-tls --enable-initfini-array --enable-__cxa_atexit \
      --disable-libgcc \
      --with-simd=none \
      --enable-default-pie \
      --enable-languages=c,c++,fortran,objc,obj-c++,lto
   cd $(GCC)/tools-build && make all-gcc all-target-libgcc ${JOBS}
   cd $(GCC)/tools-build && make install-strip-gcc install-strip-target-libgcc
```

#### GlibC的编译和安装
```Makefile
GLIBC     := glibc-2.42
glibc:
   cd $(GLIBC) && sed -i "s@5.15.0@4.15.0@g" sysdeps/unix/sysv/linux/loongarch/configure{,.ac}
   cd $(GLIBC) && rm -rf build-64 && mkdir -v build-64
   cd $(GLIBC)/build-64 && BUILD_CC="gcc" CC="${CROSS_TARGET}-gcc ${BUILD64} -mstrict-align" \
        CXX="${CROSS_TARGET}-gcc ${BUILD64} -mstrict-align" \
        AR="${CROSS_TARGET}-ar" RANLIB="${CROSS_TARGET}-ranlib" \
        ../configure --prefix=/usr --host=${CROSS_TARGET} --build=${CROSS_HOST} \
                    --libdir=/usr/lib64 --libexecdir=/usr/lib64/glibc \
                    --with-binutils=${SYSDIR}/$(CROSS_BUILD_DIR)/bin \
                    --with-headers=${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/usr/include \
                    --enable-stack-protector=strong \
                    --enable-add-ons \
                    --enable-crypt \
                    --disable-werror libc_cv_slibdir=/usr/lib64 \
                    --enable-kernel=4.15
   cd $(GLIBC)/build-64 && make ${JOBS}
   cd $(GLIBC)/build-64 && make DESTDIR=${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot install
   cd $(GLIBC)/build-64 && cp -v ../nscd/nscd.conf ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/etc/nscd.conf
   cd $(GLIBC)/build-64 && mkdir -pv ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/var/cache/nscd
   cd $(GLIBC)/build-64 && install -v -Dm644 ../nscd/nscd.tmpfiles \
                     ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/usr/lib/tmpfiles.d/nscd.conf
   cd $(GLIBC)/build-64 && install -v -Dm644 ../nscd/nscd.service \
                        ${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot/usr/lib/systemd/system/nscd.service
```

#### 完整版GCC的编译和安装
```Makefile
GCC       := gcc-15.2.0
gcc:
   cd $(GCC) && rm -rf tools-build-all && mkdir tools-build-all
   cd $(GCC)/tools-build-all && AR=ar LDFLAGS="-Wl,-rpath,${SYSDIR}/$(CROSS_BUILD_DIR)/lib" \
      ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) --build=${CROSS_HOST} \
                   --host=${CROSS_HOST} --target=${CROSS_TARGET} \
                   --with-sysroot=${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot --with-mpfr=${SYSDIR}/$(CROSS_BUILD_DIR) \
                   --with-gmp=${SYSDIR}/$(CROSS_BUILD_DIR) --with-mpc=${SYSDIR}/$(CROSS_BUILD_DIR) \
                   --enable-__cxa_atexit --enable-threads=posix --with-system-zlib \
                   --enable-libstdcxx-time --enable-checking=release \
                   --disable-libssp \
                   --with-simd=none \
                   --enable-default-pie \
                   --enable-languages=c,c++,fortran,objc,obj-c++,lto
   cd $(GCC)/tools-build-all && make ${JOBS}
   cd $(GCC)/tools-build-all && make install-strip
```
此时安装的GCC会替换掉我们在上面安装的简易版GCC。

#### GDB的编译和安装
```Makefile
GDB       := gdb-16.3
gdb:
   rm -rf $(GDB)/build
   mkdir -p $(GDB)/build
   cd $(GDB)/build && ../configure --prefix=${SYSDIR}/$(CROSS_BUILD_DIR) \
                      --build=${CROSS_HOST} \
                     --host=${CROSS_HOST} --target=${CROSS_TARGET} \
                     --with-sysroot=${SYSDIR}/$(CROSS_BUILD_DIR)/sysroot --enable-64-bit-bfd
   cd $(GDB)/build && make ${JOBS}
   cd $(GDB)/build && make DESTDIR=${SYSDIR}/${CROSS_BUILD_DIR} install
```


#### 特殊说明
LoongArch的GCC上游在版本13以后，再编译GCC的时候，会默认生成向量指令，即产生参数```-msimd=lsx```，
我们可以使用-v来查看验证
```bash
loongarch64-linux-gnu-gcc main.c -o main -v
```

生成的详细命令参数如下（省略了不必要的部分）
```bash
gcc version 14.1.0 (GCC) 
COLLECT_GCC_OPTIONS='-o' 'main' '-v' '-mabi=lp64d' '-march=la64v1.0' '-mfpu=64' '-msimd=none' '-mcmodel=normal' '-mtune=generic'
 /home/airxs/local/gcc-14.1.0-loongarch64-linux-gnu/bin/../libexec/gcc/loongarch64-linux-gnu/14.1.0/cc1 -quiet -v -iprefix /home/airxs/local/gcc-14.1.0-loongarch64-linux-gnu/bin/../lib/gcc/loongarch64-linux-gnu/14.1.0/ -isysroot /home/airxs/local/gcc-14.1.0-loongarch64-linux-gnu/bin/../sysroot main.c -quiet -dumpbase main.c -dumpbase-ext .c -mabi=lp64d -march=la64v1.0 -mfpu=64 -msimd=none -mcmodel=normal -mtune=generic -version -o /tmp/ccaWuA6Z.s
```

如果想在编译GCC的时候，关闭默认向量指令的生成，可以使用参数
```bash
--with-simd=none
```
比如上面我们在编译的时候，默认关闭了向量的生成。

:::{caution}
--with-simd=none它只是在编译在**默认情况下**关闭生成向量指令，但是还是可以使用命令参数```-msimd=```来
手动的指定可以产生128位向量。

我们用上面使用```--with-simd=none```生成的交叉编译器，来生成具有向量指令的可执行文件：
```shell
loongarch64-linux-musl-gcc main.c -o main -msimd=lsx -v
```

反汇编时，我们可以看见使用了```vld和vst```等向量指令。
:::


### 如何编译loongarch64-linux-musl[sf]

基于musl-c库的交叉编译器编译顺序和上述的基本一致，我们为了方便，将其做了封装，可以直接使用编译仓库。

1. 克隆musl-cross-make仓库
```bash 
git clone https://github.com/lyw19b/musl-cross-make.git
```
2. 复制配置文件
```bash
cd musl-cross-make && cp config.mak.loongarch64 config.mak
```

3. 修改配置文件，指令需要编译的工具链版本
```bash
#修改配置文件 config.mak
BINUTILS_VER = 2.25.1
GCC_VER = 15.2.0
GDB_VER = 16.3
MUSL_VER = 1.2.5
GMP_VER = 6.3.0
MPC_VER = 1.3.1
MPFR_VER = 4.2.2
LINUX_VER = 6.7
```
如果不指定版本，会使用默认的版本编译。

4. 定制配置参数
按照需求设置，如果不设置按照默认的参数编译。
```bash
# COMMON_CONFIG 编译各个工具链时都用的配置，比如HOST编译器参数等
COMMON_CONFIG += CFLAGS="-g0 -Os" CXXFLAGS="-g0 -Os" LDFLAGS="-s"

# GCC_CONFIG 特定针对GCC编译时的参数。
GCC_CONFIG += --disable-libquadmath --disable-decimal-float
GCC_CONFIG += --disable-libitm
GCC_CONFIG += --disable-fixed-point
GCC_CONFIG += --disable-lto

GCC_CONFIG += --enable-languages=c,c++ --enable-checking=release

```

5. 选择支持浮点还是禁止浮点指令
```bash
# 修改配置文件 config.mak

# 1. 如果时禁止浮点使用下面的TARGET 
TARGET = loongarch64-linux-muslsf

# 2. 如果使用支持生成浮点指令的编译器使用下面的TARGET
TARGET = loongarch64-linux-musl
```

编译之后，会在output中生成我们需要的交叉编译器，可以在out/bin/测试是否正常
```bash
shell> ./bin/loongarch64-linux-musl-gcc -v
# Using built-in specs.
# COLLECT_GCC=./bin/loongarch64-linux-musl-gcc
# COLLECT_LTO_WRAPPER=/home/airxs/user/toolchains/musl-cross-make/loongarch64-linux-musl-gcc-15.2.0/bin/../libexec/gcc/loongarch64-linux-musl/15.2.0/lto-wrapper
# Target: loongarch64-linux-musl
# Configured with: ../src_gcc/configure --enable-languages=c,c++ --with-simd=none --enable-languages=c,c++ --enable-checking=release --disable-bootstrap --disable-assembly --disable-werror --target=loongarch64-linux-musl --prefix= --libdir=/lib --disable-multilib --with-sysroot=/sysroot --enable-tls --disable-libmudflap --disable-libsanitizer --disable-gnu-indirect-function --disable-libmpx --enable-initfini-array --enable-libstdcxx-time=rt --with-build-sysroot=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_sysroot AR_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/ar AS_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/gas/as-new LD_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/ld/ld-new NM_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/nm-new OBJCOPY_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/objcopy OBJDUMP_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/objdump RANLIB_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/ranlib READELF_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/readelf STRIP_FOR_TARGET=/home/airxs/user/toolchains/musl-cross-make/build/local/loongarch64-linux-musl/obj_binutils/binutils/strip-new --build=x86_64-pc-linux-gnu --host=x86_64-pc-linux-gnu
# Thread model: posix
# Supported LTO compression algorithms: zlib zstd
# gcc version 15.2.0 (GCC) 
```





### crtn, crti, crtbegin, crtend有什么区别
在 Linux 系统中，`crti.o`、`crtn.o`、`crtbegin.o` 和 `crtend.o` 是 C/C++ 运行时库（CRT）的核心组件，负责程序启动和终止时的初始化与清理工作。它们的区别主要体现在功能定位、作用阶段和链接顺序上：

---

#### 功能定位与作用阶段
| **文件**       | **功能定位**                                                                 | **作用阶段**                     | **关联段（Section）**          |
|----------------|-----------------------------------------------------------------------------|----------------------------------|-------------------------------|
| **crti.o**     | 定义初始化框架的**入口**（Prologue），设置 `.init` 段的起始代码，用于全局构造函数注册 | 程序启动时，`main()` 之前        | `.init` 段开头                |
| **crtn.o**     | 定义终止框架的**出口**（Epilogue），设置 `.fini` 段的结束代码，用于全局析构函数注册   | 程序退出时，`main()` 之后        | `.fini` 段结尾                |
| **crtbegin.o** | 为 C++ 全局构造函数提供支持，将 `.ctors` 段中的构造函数列表插入到 `.init` 段末尾     | 全局构造函数注册阶段             | `.ctors` 段（或 `.init_array`）|
| **crtend.o**   | 为 C++ 全局析构函数提供支持，将 `.dtors` 段中的析构函数列表插入到 `.fini` 段开头      | 全局析构函数注册阶段             | `.dtors` 段（或 `.fini_array`）|

---

#### 具体职责对比
##### 1. **crti.o 与 crtn.o**
- **crti.o**  
  - 提供 `.init` 段的**起始代码**，负责调用 `__init` 函数（由编译器生成），该函数会遍历 `.ctors` 段中的构造函数指针并执行。  
  - 包含平台相关的初始化代码（如设置 CPU 特性、初始化 TLS 等）。  

- **crtn.o**  
  - 提供 `.fini` 段的**结束代码**，负责调用 `__fini` 函数，该函数会逆序执行 `.dtors` 段中的析构函数指针。  
  - 确保程序退出前释放全局资源（如关闭文件描述符、释放静态内存池）。  

##### 2. **crtbegin.o 与 crtend.o**
- **crtbegin.o**  
  - 在链接时插入到 `.init` 段末尾，收集所有 C++ 全局对象的构造函数地址，形成 `.ctors` 段的函数指针列表。  
  - 支持 C++ 静态对象的初始化（如 `static MyClass obj;`）。  

- **crtend.o**  
  - 在链接时插入到 `.fini` 段开头，收集所有 C++ 全局对象的析构函数地址，形成 `.dtors` 段的函数指针列表。  
  - 确保析构函数按与构造函数**相反的顺序**执行（逆序析构规则）。  

---

#### 链接顺序与依赖关系
在标准 Linux 平台下，链接器（`ld`）的默认顺序为：  
```bash
crt1.o → crti.o → crtbegin.o → 用户目标文件 → 系统库 → crtend.o → crtn.o
```
- **crt1.o**：包含入口符号 `_start`，负责调用 `__libc_start_main` 初始化 libc 并跳转到 `main()`。  
- **crtbegin.o/crtend.o**：由 GCC 提供，仅参与 C++ 全局对象的构造/析构流程，与 C 语言无关。  
- **crti.o/crtn.o**：由 glibc 提供，是所有 C/C++ 程序启动和终止的必要组件。  

---

#### 特殊变体与场景
| **文件**       | **变体**          | **适用场景**                                                                 |
|----------------|-------------------|-----------------------------------------------------------------------------|
| **crtbegin.o** | `crtbeginS.o`     | 用于生成位置无关代码（PIC）的共享库（如 `.so` 文件）。             |
| **crtend.o**   | `crtendS.o`       | 共享库的终止代码，确保动态加载时析构函数正确注册。                   |
| **crt1.o**     | `Scrt1.o`         | 生成位置无关可执行文件（PIE）时替代 `crt1.o`。                     |

---

#### 实际示例分析
以 C++ 程序为例：  
```cpp
// 全局对象构造函数
__attribute__((constructor)) void init() { /* ... */ }
// 全局对象析构函数
__attribute__((destructor)) void cleanup() { /* ... */ }

int main() { return 0; }
```
- **链接过程**：  
  1. `crtbegin.o` 将 `init` 函数地址写入 `.ctors` 段。  
  2. `crtend.o` 将 `cleanup` 函数地址写入 `.dtors` 段。  
  3. `crti.o` 的 `.init` 段代码遍历 `.ctors` 并调用所有构造函数。  
  4. `crtn.o` 的 `.fini` 段代码逆序遍历 `.dtors` 并调用所有析构函数。  

---

#### 跨平台差异
- **Windows**：类似机制由 `crt0.obj` 和 `crtlib.lib` 实现，但文件命名和段管理方式不同。  
- **嵌入式系统**：可能使用精简版 `crt0.s`，省略动态链接支持。  

通过理解这些文件的分工，开发者可以更精准地控制程序的初始化流程（如自定义构造函数顺序）或优化资源管理。



## 常用的参数与选项

本小节主要涉及与GCC相关的扩展内容，在调试查看可执行文件，或者理解操作系统底层的时很有帮助。

### GCC常见的参数
我们可以使用```man gcc```来查看详细的参数以及说明。

还可以使用[在线文档: GCC Option Summary](https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html)

其次，我们如果想要查看具体的更架构相关的参数，可以使用
```bash
loongarch64-linux-musl-gcc --target-help
```
```
The following options are target specific:
  --param=loongarch-vect-issue-info=<1,64> Indicate how many non memory access vector instructions can be issued per cycle,
                              it's used in unroll factor determination for autovectorizer.  The default value is 4.
  --param=loongarch-vect-unroll-limit=<1,64> Used to limit unroll factor which indicates how much the autovectorizer may
                              unroll a loop.  The default value is 6.
  -G<number>                  Put global and static data smaller than <number> bytes into a special section (on some targets).
  -mabi=BASEABI               Generate code that conforms to the given BASEABI.
  -maddr-reg-reg-cost=        -maddr-reg-reg-cost=COST  Set the cost of ADDRESS_REG_REG to the value calculated by COST.
  -mandroid                   Generate code for the Android platform.
  -mannotate-tablejump        Annotate table jump instruction (jr {reg}) to correlate it with the jump table.
  -march=PROCESSOR            Generate code for the given PROCESSOR ISA.
  -mbionic                    Use Bionic C library.
  -mbranch-cost=COST          Set the cost of branches to roughly COST instructions.
  -mcheck-zero-division       Trap on integer divide by zero.
  -mcmodel=                   Specify the code model.
  -mcond-move-float           Conditional moves for float are enabled.
  -mcond-move-int             Conditional moves for integral are enabled.
  -mdirect-extern-access      Avoid using the GOT to access external symbols.
  -mdiv32                     Support div.w[u] and mod.w[u] instructions with inputs not sign-extended.
  -mdouble-float              Allow hardware floating-point instructions to cover both 32-bit and 64-bit operations.
  -mexplicit-relocs           Use %reloc() assembly operators (for backward compatibility).  Same as -mexplicit-relocs=.
  -mexplicit-relocs=          Use %reloc() assembly operators.
  -mfpu=FPU                   Generate code for the given FPU.
  -mfpu=0                     Same as -mfpu=none.
  -mfrecipe                   Support frecipe.{s/d} and frsqrte.{s/d} instructions.
  -mfused-madd                Same as -ffp-contract=fast (or, in negated form, -ffp-contract=off).  Uses of this option are
                              diagnosed.
  -mglibc                     Use GNU C library.
  -mlam-bh                    Support am{swap/add}[_db].{b/h} instructions.
  -mlamcas                    Support amcas[_db].{b/h/w/d} instructions.
  -mlasx                      Enable LoongArch Advanced SIMD Extension (LASX, 256-bit).
  -mld-seq-sa                 Do not need load-load barriers (dbar 0x700).
  -mlsx                       Enable LoongArch SIMD Extension (LSX, 128-bit).
  -mmax-inline-memcpy-size=SIZE Set the max size of memcpy to inline, default is 1024.
  -mmemcpy                    Prevent optimizing block moves, which is also the default behavior of -Os.
  -mmusl                      Use musl C library.
  -mrecip                     Generate approximate reciprocal divide and square root for better throughput.  Same as -mrecip=.
  -mrecip=                    Control generation of reciprocal estimates.
  -mrelax                     Take advantage of linker relaxations to reduce the number of instructions required to
                              materialize symbol addresses.
  -msimd=SIMD                 Generate code for the given SIMD extension.
  -msingle-float              Restrict the use of hardware floating-point instructions to 32-bit operations.
  -msoft-float                Prevent the use of all hardware floating-point instructions.
  -mstrict-align              Do not generate unaligned memory accesses.
  -mtls-dialect=              Specify TLS dialect.
  -mtune=PROCESSOR            Generate optimized code for PROCESSOR.
  -muclibc                    Use uClibc C library.

  Base ABI types for LoongArch:
    lp64d lp64f lp64s

  LoongArch ARCH presets:
    abi-default la464 la64v1.0 la64v1.1 la664 loongarch64 native

  The code model option names for -mcmodel:
    extreme large medium normal tiny tiny-static

  The code model option names for -mexplicit-relocs:
    always auto none

  FPU types of LoongArch:
    32 64 none

  SIMD extension levels of LoongArch:
    lasx lsx none

  The possible TLS dialects:
    desc trad

  LoongArch TUNE presets:
    generic la464 la664 loongarch64 native

Assembler options
=================

Use "-Wa,OPTION" to pass "OPTION" to the assembler.

LARCH options:
  -mthin-add-sub    Convert a pair of R_LARCH_ADD32/64 and R_LARCH_SUB32/64 to
           R_LARCH_32/64_PCREL as much as possible
           The option does not affect the generation of R_LARCH_32_PCREL
           relocations in .eh_frame
  -mignore-start-align    Ignore .align if it is at the start of a section. This option
           can't be used when partial linking (ld -r).

Linker options
==============

Use "-Wl,OPTION" to pass "OPTION" to the linker.

elf64loongarch: 
  -z pack-relative-relocs     Pack relative relocations
  -z nopack-relative-relocs   Do not pack relative relocations (default)
```


### PIC与PIE的区别是什么

PIC（Position Independent Code，位置无关代码）和PIE（Position Independent Executable，位置无关可执行文件）是两种与内存地址无关的技术，但它们的应用场景和实现方式存在显著差异。以下是两者的核心区别及技术细节：

---

#### 定义与核心目标
| **维度**       | **PIC**                          | **PIE**                          |
|----------------|----------------------------------|----------------------------------|
| **全称**       | Position Independent Code        | Position Independent Executable  |
| **核心目标**   | 生成共享库（`.so`文件），允许多个进程共享同一份代码 | 生成可执行文件，使其加载地址随机化以提高安全性 |
| **主要应用**   | 动态链接库（如C/C++的`.so`文件） | 可执行文件（如ELF格式的Linux程序） |

---

#### 技术实现差异
##### 1. **代码生成方式**
- **PIC**  
  - 通过**全局偏移表（GOT）**和**过程链接表（PLT）**实现间接寻址，避免使用绝对地址。  
  - 代码段（`.text`）中的函数调用和全局变量访问通过GOT/PLT间接解析。  

- **PIE**  
  - 在PIC基础上进一步要求**代码段可加载到任意地址**，且必须包含`main`函数。  
  - 与PIC共享GOT/PLT机制，但链接时需通过`-pie`选项强制生成可执行文件。  
  - 示例编译命令：  
    ```bash
    gcc -fPIE -pie -o program program.c
    ```

##### **编译与链接选项**
| **选项**       | **PIC**                          | **PIE**                          |
|----------------|----------------------------------|----------------------------------|
| **编译选项**   | `-fPIC`（生成PIC代码）           | `-fPIE`（生成PIE代码）           |
| **链接选项**   | 无需特殊选项                     | `-pie`（生成位置无关可执行文件） |
| **对象文件用途**| 可用于共享库或PIE可执行文件      | 仅用于PIE可执行文件              |

---

#### 内存布局与加载机制
| **维度**       | **PIC（共享库）**                | **PIE（可执行文件）**            |
|----------------|----------------------------------|----------------------------------|
| **加载地址**   | 随机化（ASLR启用时）             | 随机化（ASLR启用时）             |
| **代码段属性** | 只读（`.text`不可修改）          | 可执行（包含`main`函数入口）     |
| **数据段共享** | 多个进程共享同一份代码段         | 每个进程独立数据段               |

---

#### 安全性对比
| **攻击类型**   | **PIC（共享库）**                | **PIE（可执行文件）**            |
|----------------|----------------------------------|----------------------------------|
| **缓冲区溢出** | 仅保护共享库代码，不影响主程序   | 完全随机化主程序地址，提升防御   |
| **ROP攻击**    | 依赖GOT/PLT的间接跳转，可能被利用| ROP链构造难度更高                |
| **ASLR支持**   | 必须启用ASLR才能生效             | 必须启用ASLR才能生效             |

---

#### 性能影响
| **维度**       | **PIC**                          | **PIE**                          |
|----------------|----------------------------------|----------------------------------|
| **间接寻址开销** | 较高（需频繁访问GOT/PLT）        | 较低（部分优化可减少GOT访问）    |
| **代码体积**   | 较大（需额外GOT/PLT表）          | 较小（与PIC共享优化策略）        |
| **启动时间**   | 无显著影响                       | 可能因地址随机化增加少量时间     |

---

#### 典型使用场景
| **场景**       | **PIC**                          | **PIE**                          |
|----------------|----------------------------------|----------------------------------|
| **共享库开发** | 必须使用（如`.so`文件）          | 不适用                           |
| **可执行文件** | 可选（非必须）                   | 必须启用（对抗漏洞攻击）         |
| **动态加载**   | 支持（如`dlopen`加载插件）       | 不支持                           |

---

#### 特殊注意事项
1. **符号导出限制**  
   - PIC共享库需明确导出符号（通过`-export-dynamic`），而PIE可执行文件默认隐藏符号。  
2. **多模块交互**  
   - PIC代码可被多个进程共享，但PIE代码每个进程独立加载，内存占用更高。  
3. **编译器兼容性**  
   - 某些架构（如ARM）要求PIC和PIE编译选项必须显式指定，否则默认生成位置相关代码。

---

#### 总结对比表
| **特性**       | **PIC**                          | **PIE**                          |
|----------------|----------------------------------|----------------------------------|
| **核心用途**   | 共享库开发                       | 可执行文件安全加固               |
| **地址随机化** | 依赖ASLR                         | 强制启用ASLR                     |
| **代码共享**   | 多进程共享                       | 单进程独占                       |
| **编译选项**   | `-fPIC`                          | `-fPIE -pie`                     |
| **安全等级**   | 中等（保护库代码）               | 高（保护主程序）                 |

通过合理选择PIC和PIE，开发者可以在性能、安全性和兼容性之间取得平衡。例如，共享库必须使用PIC，而面向安全的关键程序（如浏览器、数据库）应启用PIE以防御内存攻击。

---

### 与LoongArch相关的选项说明

下面我们将介绍与LoongArch相关的参数，涉及到GCC编译，但不包括链接阶段。[^gcc_loongarch_options]

---
-march=arch-type:
   为特定的arch-type生成相关的指令代码，
   - native： 由编译器自行检测处理器类型
   - loongarch64： 通用的64位LoongArch处理器
   - la464：基于LA464的处理器，带有LSX, LASX指令集
   - la664：基于LA664的处理器，带有LSX, LASX指令集和LoongArch v1.1所有指令集
   - la64v1.0： LoongArch64 ISA 版本 v1.0
   - la64v1.1： LoongArch64 ISA 版本 v1.1
   - la32v1.0： LoongArch32 ISA 版本 v1.0
   - la32rv1.0： LoongArch32 精简 ISA 版本 v1.0
---

-mtune=tune-type:
   为类型tune-type产生特定的优化代码，
   - native： 由编译器自行检测处理器类型
   - generic： 通用的LoongArch处理器
   - loongarch64： 通用的64位LoongArch处理器
   - la464：基于LA464的处理器
   - la664：基于LA664的处理器
   - loongarch32： 通用的32位LoongArch处理器
---

-mabi=base-abi-type:
   为类型为base-abi-type的调用规约(Calling Convention)产生代码，
   - lp64d： 使用64位通用寄存器，32/64位浮点寄存器传递参数，
             数据模型是LP64，指的是int是32位，long int和指针都是64位。
   - lp64f： 使用64位通用寄存器，32位浮点寄存器传递参数，
             数据模型是LP64，指的是int是32位，long int和指针都是64位。
   - lp64s： 使用64位通用寄存器，没有浮点寄存器传递参数，
             数据模型是LP64，指的是int是32位，long int和指针都是64位。
---

-mfpu=fpu-type:
   为指定的fpu-type类型产生代码，
   - 64： 允许32、64位的浮点操作指令。
   - 32： 允许32位的浮点操作指令。
   - 'none'或者'0':禁止产生浮点指令。 
---

后续待补充。。。


[^gcc_loongarch_options]: https://gcc.gnu.org/onlinedocs/gcc/LoongArch-Options.html



### C/C++ 预处理器内嵌的宏定义
5. 与我们LoongArch相关的定义，比如__longarch_等等，全部收集下
   最好能把代码也放出来，或者说是引用也放出来。

:::{list-table} 与LoongArch相关的C/C++预处理器内嵌的宏定义
:header-rows: 1

*   - **名称**
    - **可能的值**
    - **描述说明**

*   - Albatross
    - 2.99
    - On a stick!

*   - `__loongarch__`
    - `1`
    - Defined if the target is LoongArch.

*   - `__loongarch_grlen`
    - `32` `64`
    - Bit-width of general purpose registers.

*   - `__loongarch_frlen`
    - `0` `32` `64`
    - Bit-width of floating-point registers (`0` if there is no FPU).

*   - `__loongarch_arch`
    - `"loongarch64"`  `"la464"` `"la664"` `"la64v1.0"` `"la64v1.1"` `"la32v1.0"` `"la32rv1.0"`
    - Target ISA preset as specified by `-march=`.
If `-march=` is not present, an implementation-defined default value should be
used. If `-march=native` is enabled (user-specified or the default value),
the result is automatically detected by the compiler.

*   - `__loongarch_tune`
    - `"generic"` `"loongarch64"` `"la464"` `"la664"` `"loongarch32"`
    - Processor model as specified by `-mtune` or its default value.
If `-mtune=native` is enabled (either explicitly given or set with
`-march=native`), the result is automatically detected by the compiler.

*   - `__loongarch_lp64`
    - `1` or undefined
    - Defined if ABI uses the LP64 data model and 64-bit GPRs for parameter passing.

*   - `__loongarch_ilp32`
    - `1` or undefined
    - Defined if ABI uses the ILP32 data model and 32-bit GPRs for parameter passing.

*   - `__loongarch_hard_float`
    - `1` or undefined
    - Defined if floating-point/extended ABI type is `single` or `double`.

*   - `__loongarch_soft_float`
    - `1` or undefined
    - Defined if floating-point/extended ABI type is `soft`.

*   - `__loongarch_single_float`
    - `1` or undefined
    - Defined if floating-point/extended ABI type is `single`.

*   - `__loongarch_double_float`
    - `1` or undefined
    - Defined if floating-point/extended ABI type is `double`.

*   - `__loongarch_sx`
    - `1` or undefined
    - Defined if the compiler enables the `lsx` ISA extension.

*   - `__loongarch_asx`
    - `1` or undefined
    - Defined if the compiler enables the `lasx` ISA extension.

*   - `__loongarch_simd_width`
    - `128` `256` or undefined
    - The maximum SIMD bit-width enabled by the compiler.
(`128` for `lsx`, and `256` for `lasx`)

*   - `__loongarch_frecipe`
    - `1` or undefined
    - Defined if `-mfrecipe` is enabled.

*   - `__loongarch_div32`
    - `1` or undefined
    - Defined if `-mdiv32` is enabled.

*   - `__loongarch_lam_bh`
    - `1` or undefined
    - Defined if `-mlam-bh` is enabled.

*   - `__loongarch_lamcas`
    - `1` or undefined
    - Defined if `-mlamcas` is enabled.

*   - `__loongarch_ld_seq_sa`
    - `1` or undefined
    - Defined if `-mld-seq-sa` is enabled.

*   - `__loongarch_version_major`
    - `1` or undefined
    - The minimally required LoongArch ISA version (major) to run the compiled program.
Undefined if no such version is known to the compiler.

*   - `__loongarch_version_minor`
    - `0` `1` or undefined
    - The minimally required LoongArch ISA version (minor) to run the compiled program.
Undefined if and only if `__loongarch_version_major` is undefined.
:::
