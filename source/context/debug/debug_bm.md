
# 裸机运行环境搭建

教程提供 C 语言源代码到可直接在QEMU上运行的裸机可执行文件的完整编译、链接和运行环境，无需操作系统支持。

## 运行环境与开发工具

1, QEMU 安装

请参考本文档 1.3.1 章节的安装教程，安装 QEMU 。

2, 交叉编译器配置

请参考本文档 1.4 章节的安装教程，配置交叉编译器。

## 基于picolibc库的运行环境

### 获取和编译picolibc库

1, 获取 picolibc 源码。

``` shell
# 克隆klibc源码
git clone https://github.com/picolibc/picolibc.git
cd picolibc
```
2, 创建一个交叉编译配置文件 cross-loongarch64.txt，输入内容。
``` shell
# cross-loongarch64.txt
[binaries]
c = 'loongarch64-unknown-linux-gnu-gcc'
ar = 'loongarch64-unknown-linux-gnu-ar'
as = 'loongarch64-unknown-linux-gnu-as'
ld = 'loongarch64-unknown-linux-gnu-ld'
strip = 'loongarch64-unknown-linux-gnu-strip'

[host_machine]
system = 'none'          # 裸机系统
cpu_family = 'loongarch'
cpu = 'loongarch64'
endian = 'little'

[properties]
c_args = ['-march=loongarch64', '-mabi=lp64d']
link_args = ['-nostdlib']
needs_exe_wrapper = true
```
3, 运行以下命令，开始构建。
``` shell
meson setup build-loongarch64 \
  --cross-file cross-loongarch64.txt \
  --prefix=$(pwd)/picolibc-install
cd build-loongarch64
ninja
ninja install
```
在 picolibc-install/lib 路径下，查看到 lib.c.a, libg.a, libm.a, libnosys.a 等文件，说明构建成功。

4, 创建用户程序，以 hello-world 程序为例。
``` c
/* hello.c */
#include <stdio.h>

int main(void) {
    printf("Hello, LoongArch!\n");
    return 0;
}
```
5, 创建链接脚本 linker.ld 。
``` shell
/* linker.ld */
OUTPUT_ARCH(loongarch64)
ENTRY(_start)

SECTIONS {
    /* QEMU virt机器的RAM起始地址 */
    . = 0x9000000000200000;
    
    .text : {
        *(.text.entry)
        *(.text*)
        *(.rodata*)
        *(.srodata*)
    }
    
    .data : {
        *(.data*)
        *(.sdata*)
    }
    
    .bss : {
        *(.bss*)
        *(.sbss*)
        *(COMMON)
    }
    
    .stack : {
        . = ALIGN(16);
        stack_bottom = .;
        . = . + 0x4000;  /* 16KB stack */
        stack_top = .;
    }
    
    /DISCARD/ : {
        *(.comment)
        *(.note*)
        *(.debug*)
        *(.eh_frame*)
    }
}
```
6, 创建启动代码 ctr.S 。
``` assemble
# crt.S
.section .text.entry
.global _start

_start:
    la $sp, stack_top
    
    b    main
    
.fill 4096,1,0
stack_top:
	.space 64
```

7, 创建系统调用封装 syscall.c 。
``` c
// syscalls.c
#include <sys/stat.h>
#include <stdint.h>
#include <stdio.h>

void _exit(int status) {
    while(1);
}

#define UART_BASE 0x1fe001e0
#define UART_TX   (UART_BASE + 0x00)
#define UART_LSR  (UART_BASE + 0x05)
#define UART_LSR_TX_READY 0x20

static int uart_putc(char c,FILE *file) {
    while ((*((volatile uint8_t*)(UART_LSR)) & UART_LSR_TX_READY) == 0);
    
    *((volatile uint8_t*)(UART_TX)) = c;
    
    if (c == '\n') {
        while ((*((volatile uint8_t*)(UART_LSR)) & UART_LSR_TX_READY) == 0);
        *((volatile uint8_t*)(UART_TX)) = '\r';
    }
	(void) file;
	return 0;
}

__attribute__((weak)) int _close(int fd) { return -1; }
__attribute__((weak)) off_t _lseek(int fd, off_t offset, int whence) { return 0; }
__attribute__((weak)) int _fstat(int fd, struct stat *buf) { buf->st_mode = S_IFCHR; return 0; }
__attribute__((weak)) int _isatty(int fd) { return (fd == 1); }

static FILE __stdout = FDEV_SETUP_STREAM(uart_putc, NULL, NULL, _FDEV_SETUP_WRITE);

FILE *const stdout = &__stdout;
```
8, 创建编译和运行脚本 Makefile 。
``` Makefile
CROSS_TOOL=loongarch64-unknown-linux-gnu-
GCC=$(CROSS_TOOL)gcc
LD=$(CROSS_TOOL)ld
AS=$(CROSS_TOOL)as

compile:
	${AS} -mabi=lp64d -c crt.S -o crt.o
	${GCC} -c syscall.c -o syscall.o -I./picolibc/picolibc-install/include -march=loongarch64 -mabi=lp64d -O2 -Wall
	${GCC} -c hello.c -o hello.o -I./picolibc/picolibc-install/include -march=loongarch64 -mabi=lp64d -O2 -Wall
	${LD} crt.o hello.o syscall.o -I./picolibc/picolibc-install/include -L./picolibc/picolibc-install/lib -lc -lg -lnosys -T linker.ld -o hello.elf --no-dynamic-linker -static

clean:
	rm *.o
	rm *.elf

run:
	qemu-system-loongarch64 -M virt -m 8G -kernel hello.elf -nographic -serial mon:stdio
```

9, 执行一下命令，完成编译与运行
``` shell
make compile
#	在当前目录下生成hello.elf
#	修改脚本中 qemu-system-loongarch64 程序的路径为本机路径
#	修改脚本中 -kernel hello.elf 为本机生成的 elf 文件
make run
```
即可在终端中查看 QEMU 打印信息。

用户可基于此框架，添加自定义用户程序，并在 QEMU 上运行测试。