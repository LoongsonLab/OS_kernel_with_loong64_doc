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

:::{tip}
本文中的交叉工具链构建命令以 **GCC 15.2.0、Binutils 2.45、glibc 2.42、Linux UAPI 6.7 和 GDB 16.3** 为验证基线，记录日期为 **2026-10-15**。这不是“最新版本”声明；GCC 的发布版本和 LoongArch 选项会持续变化。使用其他版本时，应先执行 ``$CC -Q --help=target``、``$CC -v`` 和 ``$CC -dumpmachine``，并重新验证生成 ELF 的 ABI 标记、动态加载器和目标 CPU 兼容性。
:::

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

如果没有特殊说明，后续示例采用“宿主机运行交叉工具链、目标机运行 LoongArch ELF”的环境；宿主机可以是 x86_64、AArch64 或其他受支持架构，并不限定为 x86_64。

## 目标三元组、C 库与 ABI

三元组描述操作系统和 C 库生态，不能单独完整决定 ABI；还必须检查 ``-mabi``、sysroot、启动对象、库和动态加载器是否属于同一 ABI。

GNU target triplet 通常写作 ``machine-vendor-os``。例如 ``loongarch64-linux-gnu`` 中，``loongarch64`` 是目标 CPU 架构，vendor 可以省略或写作 ``unknown``，``linux-gnu`` 表示 Linux/GNU 用户态。[^osdev_gcc]

|目标前缀示例|C 库/运行环境|适用说明|
|-|-|-|
|``loongarch64-linux-gnu-``|glibc Linux 用户态|常见的通用 Linux 交叉工具链前缀。|
|``loongarch64-linux-musl-``|musl Linux 用户态|面向 musl 根文件系统；头文件、库和动态加载器必须来自同一 sysroot。|
|``loongarch64-unknown-elf-``|裸机或嵌入式运行时|通常搭配 newlib 或自建运行时，不包含 Linux 动态加载器。|
|``loongarch64-unknown-none-``|裸机运行时|是否提供 C 库、启动文件和链接脚本取决于具体工具链发行版。|

``loongarch64-linux-gnusf`` 和 ``loongarch64-linux-muslsf`` 曾被部分工具链发行版用作软浮点运行时的**发行包前缀**。它们不是只要在任意 GNU 三元组末尾加 ``sf`` 就自动得到的标准 ABI 选择方式；应以该工具链发行版提供的 sysroot、C 库、动态加载器名称和文档为准。

LoongArch LP64 基础 ABI 由 ``-mabi=`` 选择，且会影响对象文件的调用约定和 ELF ABI 标记：

|ABI|参数传递与浮点约定|运行时要求|
|-|-|-|
|``lp64d``|LP64 数据模型；使用 64 位通用寄存器，双精度浮点 ABI 可使用浮点寄存器传递参数。|需要与 ``lp64d`` 匹配的 C 库、启动文件和动态加载器；这是多数通用 LoongArch64 Linux 环境的常见选择。|
|``lp64f``|LP64 数据模型；单精度浮点 ABI 使用浮点寄存器传递参数。|需要完整的 ``lp64f`` sysroot；不能与 ``lp64d`` 或 ``lp64s`` 对象/库混合链接。|
|``lp64s``|LP64 数据模型；浮点参数不通过浮点寄存器传递。|需要完整的 ``lp64s``（常称软浮点）sysroot；仅关闭某个源文件的浮点指令并不会把 ``lp64d`` C 库变成 ``lp64s``。|

软浮点 ABI 的现实可用性取决于发行版和工具链是否同时提供 ``lp64s`` 的 libc、crt 对象、libgcc 和动态加载器；不少通用发行版只提供 ``lp64d`` 用户态。因此，在选用 ``lp64f`` 或 ``lp64s`` 前，先检查工具链支持的 ABI、sysroot 中的库和目标根文件系统，而不是仅凭编译器文件名判断。

[^osdev_gcc]: 具体可参考[Target Triplet](https://wiki.osdev.org/Target_Triplet)。

### 最小 ABI 检查

以下示例不会假定动态加载器的固定路径，而是直接检查工具链和生成 ELF。至于工具链构建和 sysroot 准备，请参见后续小节。

```c
#include <stdio.h>
int main(int argc, char const *argv[])
{
   printf("Hello LoongArch\n");
   return 0;
}
```


```bash
CC=loongarch64-linux-gnu-gcc
READELF=loongarch64-linux-gnu-readelf

"$CC" -dumpmachine
"$CC" --print-sysroot
"$CC" -Q --help=target | grep -E '^-mabi=|^-march=|^-mstrict-align'

"$CC" -mabi=lp64d hello.c -o hello.lp64d
"$READELF" -hA -lW hello.lp64d
```

输出中的 ``Machine`` 应为 LoongArch；ELF header 的 ``Flags``/attributes 应与 ``-mabi`` 一致；若生成的是动态 ELF，程序头中的 ``Requesting program interpreter`` 必须指向目标根文件系统实际提供的加载器。将 ``-mabi=lp64d`` 替换为 ``lp64f`` 或 ``lp64s`` 前，先确认编译、链接和目标机运行均成功。


## 如何编译交叉编译器

本小节会教大家如何从gcc等源码编译我们使用的交叉编译器。

### 如何构建 `loongarch64-linux-gnu` 的 LP64 工具链

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

*   - GCC版本： gcc-15.2.0
    - [https://ftp.gnu.org/gnu/gcc/gcc-15.2.0](https://ftp.gnu.org/gnu/gcc/gcc-15.2.0)

*   - GDB版本： gdb-16.3
    - [https://ftp.gnu.org/gnu/gdb/](https://ftp.gnu.org/gnu/gdb/)

:::

下面给出一个 GNU/Linux 交叉工具链的最小、可复现流程。它假定所有源码压缩包已经按上表所列版本解压到同一工作目录；下载后应先核对发布方提供的 SHA256/SIGNATURE。不要把构建目录放在源码树中，也不要通过修改 glibc 源码来绕过版本检查。

下面的命令在 Bash 中执行。`BUILD` 是构建和运行工具链的宿主机三元组，`TARGET` 是生成代码的目标机三元组；二者不能互换。示例默认产生 `lp64d` 工具链，若目标是软浮点 ABI，应将 `TARGET`、GCC 配置和后续 sysroot 成套替换，并单独验证，不能只改变一个 `-mabi` 参数。

```bash
set -euo pipefail

TOP="$(pwd)"                         # 仅包含已验证的源码包和本次构建产物
BUILD="$(gcc -dumpmachine)"           # 例如 x86_64-pc-linux-gnu
TARGET="loongarch64-linux-gnu"
PREFIX="$TOP/cross-tools-all-202612"
SYSROOT="$PREFIX/sysroot"
JOBS="${JOBS:-$(nproc)}"

LINUX_SRC="$TOP/linux-6.7"
BINUTILS_SRC="$TOP/binutils-2.45"
GCC_SRC="$TOP/gcc-15.2.0"
GLIBC_SRC="$TOP/glibc-2.42"
GDB_SRC="$TOP/gdb-16.3"
GMP_SRC="$TOP/gmp-6.3.0"
MPFR_SRC="$TOP/mpfr-4.2.2"
MPC_SRC="$TOP/mpc-1.3.1"
HOST_DEPS="$TOP/host-tools"

mkdir -p "$PREFIX" "$SYSROOT/usr/include" "$TOP/build" "$HOST_DEPS"
export PATH="$PREFIX/bin:$PATH"
```

`$TOP` 中的版本号必须与实际下载的源码目录一致。每次重建请使用新的、为空的 `$TOP/build/<component>` 目录；若要删除旧构建目录，先用 `pwd` 和 `test -d` 确认目标目录，再由用户显式删除，避免在文档中提供面向源码树的递归删除命令。

#### 1. 构建 GCC 的宿主依赖

GCC 本身在宿主机上运行，需要 GMP、MPFR 和 MPC。以下步骤将它们安装到独立的 `$HOST_DEPS`，随后两个 GCC 阶段均显式使用这一前缀；这比依赖宿主机中不明版本的开发包更容易复现。

```bash
mkdir "$TOP/build/gmp"
cd "$TOP/build/gmp"
"$GMP_SRC/configure" --prefix="$HOST_DEPS" --enable-cxx --disable-static
make -j"$JOBS"
make install

mkdir "$TOP/build/mpfr"
cd "$TOP/build/mpfr"
"$MPFR_SRC/configure" --prefix="$HOST_DEPS" --disable-static \
  --with-gmp="$HOST_DEPS"
make -j"$JOBS"
make install

mkdir "$TOP/build/mpc"
cd "$TOP/build/mpc"
"$MPC_SRC/configure" --prefix="$HOST_DEPS" --disable-static \
  --with-gmp="$HOST_DEPS" --with-mpfr="$HOST_DEPS"
make -j"$JOBS"
make install
```

#### 2. 安装 Linux UAPI 头文件

这一步安装的是面向用户态的 UAPI 头文件，并不编译内核，也不应运行 `make mrproper` 破坏已有的内核配置。

```bash
make -C "$LINUX_SRC" ARCH=loongarch \
  INSTALL_HDR_PATH="$SYSROOT/usr" headers_install
find "$SYSROOT/usr/include" -name '.*' -type f -delete
```

完成后，头文件位于 `$SYSROOT/usr/include`。内核头文件版本必须满足所选 glibc 的最低要求；这个要求由 glibc 发布版本决定，不能通过 `sed` 修改 glibc 的 configure 文件来降低。

#### 3. 构建并安装 Binutils

```bash
mkdir "$TOP/build/binutils"
cd "$TOP/build/binutils"
"$BINUTILS_SRC/configure" \
  --build="$BUILD" --host="$BUILD" --target="$TARGET" \
  --prefix="$PREFIX" --with-sysroot="$SYSROOT" \
  --disable-nls --disable-werror --enable-64-bit-bfd
make -j"$JOBS"
make install
```

#### 4. 构建 stage-1 GCC 与 libgcc

第一阶段只需要 C 编译器和 `libgcc`，不能依赖尚未安装的目标 C 库。因此使用 `--without-headers`；同时**不要**传递 `--disable-libgcc`，因为本步骤正是要构建 `all-target-libgcc`。

```bash
mkdir "$TOP/build/gcc-stage1"
cd "$TOP/build/gcc-stage1"
"$GCC_SRC/configure" \
  --build="$BUILD" --host="$BUILD" --target="$TARGET" \
  --prefix="$PREFIX" --with-sysroot="$SYSROOT" \
  --with-gmp="$HOST_DEPS" --with-mpfr="$HOST_DEPS" --with-mpc="$HOST_DEPS" \
  --with-newlib --without-headers --disable-nls \
  --enable-languages=c --disable-shared --disable-threads \
  --disable-libatomic --disable-libgomp --disable-libquadmath \
  --disable-libsanitizer --disable-libssp --disable-libstdcxx \
  --disable-multilib --with-abi=lp64d --with-simd=none
make -j"$JOBS" all-gcc all-target-libgcc
make install-gcc install-target-libgcc
```

#### 5. 构建并安装 glibc 到 sysroot

glibc 必须在源码树外构建。`--enable-kernel` 要选择所支持发行版的实际最低 Linux 内核版本；下例保留 `5.15` 仅作示例，修订该值时应同时更新兼容性矩阵和测试环境。

```bash
mkdir "$TOP/build/glibc"
cd "$TOP/build/glibc"
BUILD_CC=gcc \
CC="$TARGET-gcc --sysroot=$SYSROOT -mabi=lp64d -mstrict-align" \
AR="$TARGET-ar" RANLIB="$TARGET-ranlib" \
"$GLIBC_SRC/configure" \
  --build="$BUILD" --host="$TARGET" --prefix=/usr \
  --libdir=/usr/lib64 --with-headers="$SYSROOT/usr/include" \
  --enable-kernel=5.15 --disable-werror
make -j"$JOBS"
make install_root="$SYSROOT" install
```

#### 6. 构建完整 GCC

第二阶段使用已经具备 Linux 头文件和 glibc 的 sysroot。示例只启用 C/C++；Fortran、LTO、sanitizer 等运行库应根据 sysroot 是否满足其依赖单独启用和测试。

```bash
mkdir "$TOP/build/gcc-final"
cd "$TOP/build/gcc-final"
"$GCC_SRC/configure" \
  --build="$BUILD" --host="$BUILD" --target="$TARGET" \
  --prefix="$PREFIX" --with-sysroot="$SYSROOT" \
  --with-gmp="$HOST_DEPS" --with-mpfr="$HOST_DEPS" --with-mpc="$HOST_DEPS" \
  --enable-languages=c,c++ --enable-threads=posix \
  --enable-shared --enable-__cxa_atexit --disable-multilib \
  --disable-libsanitizer --disable-werror \
  --with-abi=lp64d --with-simd=none
make -j"$JOBS"
make install
```

这会覆盖 stage-1 的 GCC 驱动程序；stage-1 和 final GCC 使用不同构建目录，因此其配置和日志可以分别追溯。

#### 7. 可选：构建 GDB

GDB 是宿主机程序，应使用 `--host="$BUILD"`，而 `--target` 仍然指向调试对象的目标架构。安装时不要再使用 `DESTDIR`，否则会把 `$PREFIX` 重复拼接。

```bash
mkdir "$TOP/build/gdb"
cd "$TOP/build/gdb"
"$GDB_SRC/configure" \
  --build="$BUILD" --host="$BUILD" --target="$TARGET" \
  --prefix="$PREFIX" --with-sysroot="$SYSROOT" \
  --disable-werror
make -j"$JOBS"
make install
```

#### 8. 安装后验证

验证必须同时确认目标三元组、sysroot、ABI 和生成 ELF 的机器类型；仅看到 GCC 版本号不足以证明工具链正确。

```bash
"$TARGET-gcc" -dumpmachine
"$TARGET-gcc" --print-sysroot
printf 'int main(void) { return 0; }\n' > "$TOP/build/hello.c"
"$TARGET-gcc" -mabi=lp64d "$TOP/build/hello.c" -o "$TOP/build/hello"
"$TARGET-readelf" -hA "$TOP/build/hello"
"$TARGET-readelf" -l "$TOP/build/hello" | grep 'Requesting program interpreter'
"$TARGET-gdb" --version
```

预期 `-dumpmachine` 输出 `loongarch64-linux-gnu`，ELF 头的 `Machine` 为 LoongArch；动态链接程序路径、`e_flags` 和 attributes 则应与所选 glibc、ABI 和 binutils 版本相匹配。将上述输出及各源码包的 SHA256 保存为构建记录，才能在后续排查 ABI 问题时重现环境。


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


### 如何构建 `loongarch64-linux-musl` 工具链

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

5. 选择目标前缀与 ABI
```bash
# 修改配置文件 config.mak

# musl Linux 用户态的目标前缀
TARGET = loongarch64-linux-musl
```

``TARGET`` 用于选择目标系统和 musl 运行时，不应用它代替 ``-mabi``。如果所用的 musl-cross-make 分支明确提供 ``loongarch64-linux-muslsf``，它表示该分支打包的软浮点运行时；只有在该分支同时提供 ``lp64s`` 兼容的 musl、crt 对象、libgcc、动态加载器和 sysroot 时才能使用。构建结束后必须以 ``-Q --help=target``、``--print-sysroot`` 和 ``readelf -hA -lW`` 验证实际 ABI 与加载器，而不能只根据前缀推断。

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


### PIC、PIE、静态链接与 sysroot

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
  - 生成适合位置无关可执行文件的代码；程序是否包含 `main` 与 PIE 属性无关。
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
| **动态加载**   | 支持（如`dlopen`加载插件）       | PIE 程序同样可以使用动态加载     |

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
| **地址随机化** | 依赖ASLR                         | 为主程序参与 ASLR 提供条件       |
| **代码共享**   | 多进程共享                       | 单进程独占                       |
| **编译选项**   | `-fPIC`                          | `-fPIE -pie`                     |
| **安全等级**   | 中等（保护库代码）               | 高（保护主程序）                 |

通过合理选择PIC和PIE，开发者可以在性能、安全性和兼容性之间取得平衡。共享库通常使用 PIC；PIE 使主程序可参与 ASLR，但实际随机化仍取决于目标内核、加载器和运行时配置。

#### 与 ABI、链接方式和 sysroot 一起验证

``-fPIC`` 用于生成共享对象所需的位置无关代码，链接共享库时配合 ``-shared``；``-fPIE -pie`` 用于生成位置无关可执行文件。``-static`` 请求静态链接，最终 ELF 不应有 ``PT_INTERP``，但 sysroot 必须提供匹配 ABI 的静态 libc 和其他依赖库。``--sysroot=DIR`` 只改变编译、链接时的目标头文件和库搜索根目录，不会把文件部署到目标系统。

```bash
CC=loongarch64-linux-gnu-gcc
READELF=loongarch64-linux-gnu-readelf
SYSROOT=/path/to/loongarch64-sysroot

"$CC" --sysroot="$SYSROOT" -mabi=lp64d -fPIC -shared hello.c -o libhello.so
"$CC" --sysroot="$SYSROOT" -mabi=lp64d -fPIE -pie hello.c -o hello.pie
"$CC" --sysroot="$SYSROOT" -mabi=lp64d -static hello.c -o hello.static
"$CC" --sysroot="$SYSROOT" -mabi=lp64d -mstrict-align hello.c -o hello.strict-align

"$READELF" -h -lW hello.pie
"$READELF" -h -lW hello.static
"$CC" --sysroot="$SYSROOT" -v -E -x c /dev/null
```

验证时，``libhello.so`` 应为共享对象，``hello.pie`` 通常为 ``ET_DYN`` 且带有 ``PT_INTERP``，``hello.static`` 不应有 ``Requesting program interpreter``。最后一条命令显示实际的头文件和库搜索路径，可用于确认没有意外使用宿主机文件。

``-mstrict-align`` 要求编译器不生成非对齐内存访问指令，适用于目标核或外设映射不支持非对齐访问的情形。它不改变 ABI，不能修复无效指针、越界访问或错误的结构布局；开启后应在目标硬件上测试性能和功能。

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

#### `-march` 与 `-mabi` 的兼容规则

* ``-march`` 决定允许生成的 ISA 指令与扩展；选择 ``la64v1.1``、``la464`` 或 ``la664`` 所生成的程序，不能假定能在只支持 LA64 v1.0 或缺少相应扩展的目标 CPU 上运行。
* ``-mabi`` 决定数据模型和跨对象的调用约定。参与同一次链接的应用对象、静态库、启动对象和 libc 必须使用兼容 ABI；即使 ``-march`` 相同，混用 ``lp64d``、``lp64f``、``lp64s`` 仍是不正确的。
* 不同对象可以针对不同的、但目标 CPU 均支持的 ``-march`` 编译；发布程序时应以部署 CPU 的最小 ISA 能力为准。不能只因链接成功就认为目标机器能够执行所有指令。
* ``-mfpu``、``-msoft-float`` 和 ``-msimd`` 控制可生成的指令或扩展，它们不能替代 ``-mabi``。改变 ABI 时必须同时换用对应 sysroot；仅增加或禁止某一类指令时，仍应检查 ABI 是否保持不变。

可通过编译器的实际配置和 ELF 输出验证选择：

```bash
CC=loongarch64-linux-gnu-gcc
READELF=loongarch64-linux-gnu-readelf

"$CC" -Q -march=la64v1.0 -mabi=lp64d --help=target
"$CC" -march=la64v1.0 -mabi=lp64d hello.c -o hello.la64v1.0.lp64d
"$READELF" -hA hello.la64v1.0.lp64d
```

若编译器拒绝某个 ``-march/-mabi`` 组合、链接器报告 ABI 不匹配，或目标根文件系统缺少对应加载器/库，应停止并选择工具链实际提供的组合，而不是强行混合对象文件。

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
