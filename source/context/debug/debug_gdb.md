
# GDB的使用


## 交叉编译环境中的GDB

在 x86 环境下配置 LoongArch 架构 QEMU，可参考文档第一章。

在 x86 环境下配置交叉编译器，可参考文档第一章。

编译好操作系统后，可使用以下命令，启动 QEMU :

``` bash
#	replace your real path with {/path/to/qemu}.
{/path/to/qemu}/bin/qemu-system-loongarch64 -M virt -cpu la464 -m 4G -smp 1 -nographic -kernel ./vmlinux -serial mon:stdio
```
可使用以下命令，进入 gdb 调试模式:

``` bash
#	replace your real path with {/path/to/qemu}.
{/path/to/qemu}/bin/qemu-system-loongarch64 -M virt -cpu la464 -m 4G -smp 1 -nographic -kernel ./vmlinux -serial mon:stdio -s -S 
```

交叉编译器的 gdb 与 QEMU 连接， 使用以下命令:
``` bash
loongarch64-unknown-linux-gnu-gdb ./vmlinux
```

gdb 内部命令，与原生 LoongArch 上 QEMU 连接 gdb 命令保持一致。

## 常见的GDB命令示例

1. 设置断点 break/b

设置断点可使用 break 命令，或使用简写 b 。断点位置由函数名或虚拟地址确定。

可使用以下命令，查询已设置的断点。
``` shell
(gdb) info breakpoint
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x900000000020006c in main at kernel/main.c:16
	breakpoint already hit 1 time
```
可使用以下命令，删除已设置的断点。
``` shell
(gdb) delete breakpoints 1
(gdb) info breakpoint     
No breakpoints, watchpoints, tracepoints, or catchpoints.
```

2. 打印寄存器信息

可使用以下命令，打印寄存器信息:
``` shell
(gdb)info registers  # or: i r
(gdb) info registers 
r0             0x0                 0
r1             0x9000000000200058  0x9000000000200058 <spin>
r2             0x0                 0x0
r3             0x9000000000308d00  0x9000000000308d00 <stack0+4080>
r4             0x1000              4096
r5             0x1                 1
r6             0x200               512
r7             0x0                 0
r8             0x0                 0
r9             0x0                 0
r10            0x0                 0
r11            0x0                 0
r12            0x8                 8
r13            0x0                 0
r14            0x0                 0
r15            0x0                 0
r16            0x0                 0
r17            0x0                 0
r18            0x0                 0
r19            0x0                 0
r20            0x0                 0
r21            0x0                 0
r22            0x9000000000308d10  0x9000000000308d10 <uart_tx_lock>
r23            0x0                 0
r24            0x0                 0
r25            0x0                 0
r26            0x0                 0
r27            0x0                 0
r28            0x0                 0
r29            0x0                 0
r30            0x0                 0
r31            0x0                 0
orig_a0        0x0                 0
pc             0x900000000020006c  0x900000000020006c <main+16>
badv           0x0                 0x0

```
可指定寄存器，进行查询:
``` shell
(gdb) info registers r4
r4             0x1000              4096
```
3. 查看内存数值

可使用以下命令，打印虚拟内存对应地址的数值:
``` shell
(gdb) print /x *0x90000000002016a4
$7 = 0x4c000020
```
4. 查看堆栈信息

可使用以下命令，查看当前调用堆栈:
``` shell
(gdb) bt   
#0  initlock (lk=lk@entry=0x9000000000308d10 <uart_tx_lock>, name=name@entry=0x900000000020b018 "uart") at kernel/spinlock.c:14
#1  0x9000000000200158 in uartinit () at kernel/uart.c:77
#2  0x9000000000201f54 in consoleinit () at kernel/console.c:187
#3  0x90000000002000a0 in main () at kernel/main.c:17
```
可使用以下命令，切换到其他堆栈处。
``` shell
(gdb) frame 0
#0  initlock (lk=lk@entry=0x9000000000308d10 <uart_tx_lock>, name=name@entry=0x900000000020b018 "uart") at kernel/spinlock.c:14
14	  lk->name = name;
(gdb) frame 1
#1  0x9000000000200158 in uartinit () at kernel/uart.c:77
77	  initlock(&uart_tx_lock, "uart");
```
5. 单步执行

gdb 模式下，使用 next 或 step 指令，均可实现程序单步执行。

- next 命令。如何当前行包含函数调用，next 命令会执行整个函数调用，然后停到函数调用后的下一行。该命令不会进入到函数内部，而是直接执行完函数。

- step 命令。如果当前行包含函数调用，step 会进入函数内部，开始调试函数内部的代码。该指令会逐行执行函数内部的指令。

6. 退出调试

可使用以下命令，退出 gdb 调试:
``` shell
(gdb) exit
A debugging session is active.

	Inferior 1 [process 1] will be detached.

Quit anyway? (y or n) y
Detaching from program: kernel, process 1
Ending remote debugging.
```

## 调试指令常见失效状况

- 单步调试，在执行 ertn 等特权指令后，guest 程序会连续运行，调试失效。
- 内存访问指令，在遇到页表切换会，容易失效。




