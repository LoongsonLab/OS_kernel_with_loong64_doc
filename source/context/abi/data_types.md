# C语言与机器类型


## 数据类型

下面是LP64对应的数据模型，涉及到的ABI类型：lp64d，lp64f，lp64s

|标量类型|机器类型|对应的位宽|
|-|-|-|
|bool / _Bool                   |Unsigned byte                | 8-bits |
|unsigned char / char           |Unsigned / signed byte       | 8-bits |
|unsigned short / short         |Unsigned / signed half-word  | 16-bits |
|unsigned int / int             |Unsigned / signed word       | 32-bits |
|unsigned long / long           |Unsigned / signed double-word| 64-bits |
|unsigned long long / long long |Unsigned / signed double-word| 64-bits |
|pointer types                  |64-bit data pointer          | 64-bits |
|_Float16                       |Half precision (IEEE754)     | 16-bits |
|__bf16                         |Half precision (bfloat16)    | 16-bits |
|float                          |Single precision (IEEE754)   | 32-bits |
|double                         |Double precision (IEEE754)   | 64-bits |
|long double                    |Quadruple precision (IEEE754)| 128-bits|


下面是ILP32对应的数据模型，涉及到的ABI类型：ilp32d，ilp32f，ilp32s

|标量类型|机器类型|对应的位宽|
|-|-|-|
|bool / _Bool                   |Unsigned byte                | 8-bits |
|unsigned char / char           |Unsigned / signed byte       | 8-bits |
|unsigned short / short         |Unsigned / signed half-word  | 16-bits |
|unsigned int / int             |Unsigned / signed word       | 32-bits |
|unsigned long / long           |Unsigned / signed word       | 32-bits |
|unsigned long long / long long |Unsigned / signed double-word| 64-bits |
|pointer types                  |32-bit data pointer          | 32-bits |
|_Float16                       |Half precision (IEEE754)     | 16-bits |
|__bf16                         |Half precision (bfloat16)    | 16-bits |
|float                          |Single precision (IEEE754)   | 32-bits |
|double                         |Double precision (IEEE754)   | 64-bits |
|long double                    |Quadruple precision (IEEE754)| 128-bits|






## 栈帧的排布

进程执行过程中会在内存中为其分配栈空间。栈是我们最常用到的存放可读写临时数据的区域。   
通常我们会使用栈来管理函数运行过程中的返回地址、参数和局部变量等信息。     
栈的大小在程序编译后已经确定。


<!-- ```{image} ../../img/stackframe_Layout.drawio.svg
:alt: StackFrame
:class: bg-primary
:scale: 150 %
:align: center
``` -->

<!-- ```{image} ../../img/stackframe_Layout.drawio.svg
:alt: StackFrame
:scale: 150 %
:align: left
``` -->

![StackFrame](../../img/stackframe_Layout.drawio.svg)

---------------------------------------------
栈的大小需求在程序编译成可执行文件后，就已经确定。但是栈空间的分配还是    
由操作系统来动态的分配。程序被解析加载后，各个段，比如可执行段.text被加载到     
确定的地址上，这部分区域是可执行的。数据段，比如.data/.bss等，也会被     
立即分配内存，然后供程序执行时，直接访问。

但是对于栈的分配，内核在加载进程后，会分配一定的内存空间用于进程的一些    
基本信息的保存，其次是环境变量和参数等内容。之后才是正在的栈的开始地址，     
（如果有随机化，还要设置随机化的空间，使得栈的指针也是随机化的）。

具体进程的用户地址空间如下所示：

![Process_StackFrame](../../img/process_stack.drawio.svg)

---------------------------------------------

### 示例代码

本小节我们根据实际的C代码，来探索龙架构下的函数栈帧的排布问题。

ftoa函数是C语言标准库中的一个函数，用于将浮点数转换为字符串。其原型如下：

``char *ftoa(double value, int precision);``

下面的代码是我们截取了部分代码，作为说明栈的排布情况。

```c
// output the specified string in reverse, taking care of any zero-padding
static size_t _out_rev(out_fct_type out, char* buffer, size_t idx, 
	                   size_t maxlen, const char* buf, size_t len, 
	                   unsigned int width, unsigned int flags)
{
  const size_t start_idx = idx;

  // pad spaces up to given width
  if (!(flags & FLAGS_LEFT) && !(flags & FLAGS_ZEROPAD)) {
    for (size_t i = len; i < width; i++) {
      out(' ', buffer, idx++, maxlen);
    }
  }

  // reverse string
  while (len) {
    out(buf[--len], buffer, idx++, maxlen);
  }

  // append pad spaces up to given width
  if (flags & FLAGS_LEFT) {
    while (idx - start_idx < width) {
      out(' ', buffer, idx++, maxlen);
    }
  }

  return idx;
}
```

我们先分析_out_rev函数的栈帧布局。

```
_out_rev():
   c:	02fe8063 	addi.d      	$sp, $sp, -96
  10:	29c14076 	st.d        	$fp, $sp, 80
  14:	29c12077 	st.d        	$s0, $sp, 72
  18:	29c0e079 	st.d        	$s2, $sp, 56
  1c:	29c0c07a 	st.d        	$s3, $sp, 48
  20:	29c0a07b 	st.d        	$s4, $sp, 40
  24:	29c0807c 	st.d        	$s5, $sp, 32
  28:	29c0607d 	st.d        	$s6, $sp, 24
  2c:	29c0407e 	st.d        	$s7, $sp, 16
  30:	29c0207f 	st.d        	$s8, $sp, 8
  34:	29c16061 	st.d        	$ra, $sp, 88
  38:	29c10078 	st.d        	$s1, $sp, 64

  3c:	03400d6c 	andi        	$t0, $a7, 0x3
  40:	001500da 	move        	$s3, $a2
  44:	001500bb 	move        	$s4, $a1
  48:	00150099 	move        	$s2, $a0
  4c:	001500fc 	move        	$s5, $a3
  50:	0015011d 	move        	$s6, $a4

  54:	00150137 	move        	$s0, $a5
  58:	00150156 	move        	$fp, $a6
  5c:	0340097e 	andi        	$s7, $a7, 0x2

  60:	001500df 	move        	$s8, $a2
  64:	4000cd80 	beqz        	$t0, 204	# 130 <L0^A>
  68:	03400000 	nop

  6c:	0010feff 	add.d       	$s8, $s0, $s8
  70:	03400000 	nop
  74:	03400000 	nop
  78:	03400000 	nop
  7c:	0011dfe6 	sub.d       	$a2, $s8, $s0
  80:	02fffef7 	addi.d      	$s0, $s0, -1

  84:	38005fa4 	ldx.b       	$a0, $s6, $s0
  88:	00150387 	move        	$a3, $s5
  8c:	00150365 	move        	$a1, $s4
  90:	001503f8 	move        	$s1, $s8
  94:	4c000321 	jirl        	$ra, $s2, 0

  98:	47ffe6ff 	bnez        	$s0, -28	# 7c <L0^A>
  9c:	400043c0 	beqz        	$s7, 64	# dc <L0^A>
  a0:	0011ebfa 	sub.d       	$s3, $s8, $s3
  a4:	00df02d7 	bstrpick.d  	$s0, $fp, 0x1f, 0x0
  a8:	6c003757 	bgeu        	$s3, $s0, 52	# dc <L0^A>
  ac:	03400000 	nop
  b0:	03400000 	nop
  b4:	03400000 	nop

  b8:	00150306 	move        	$a2, $s1
  bc:	00150387 	move        	$a3, $s5
  c0:	00150365 	move        	$a1, $s4
  c4:	02808004 	li.w        	$a0, 32
  c8:	02c0075a 	addi.d      	$s3, $s3, 1
  cc:	02c00718 	addi.d      	$s1, $s1, 1
  d0:	4c000321 	jirl        	$ra, $s2, 0
  d4:	6bffe757 	bltu        	$s3, $s0, -28	# b8 <L0^A>
  d8:	03400000 	nop

  dc:	28c16061 	ld.d        	$ra, $sp, 88
  e0:	28c14076 	ld.d        	$fp, $sp, 80
  e4:	28c12077 	ld.d        	$s0, $sp, 72
  e8:	28c0e079 	ld.d        	$s2, $sp, 56
  ec:	28c0c07a 	ld.d        	$s3, $sp, 48
  f0:	28c0a07b 	ld.d        	$s4, $sp, 40
  f4:	28c0807c 	ld.d        	$s5, $sp, 32
  f8:	28c0607d 	ld.d        	$s6, $sp, 24

  fc:	28c0407e 	ld.d        	$s7, $sp, 16
 100:	28c0207f 	ld.d        	$s8, $sp, 8
 104:	00150304 	move        	$a0, $s1
 108:	28c10078 	ld.d        	$s1, $sp, 64
 10c:	02c18063 	addi.d      	$sp, $sp, 96
 110:	4c000020 	ret
```

栈分布如下所示：

![StackFrame-Case1](../../img/stackframe_Layout-example-1.drawio.svg)

这是没有使能优化的情况下的栈帧情况，可以看出函数中间使用的静态寄存器和``ra``和``fp``     
都是按照固定的排布来保存的。

----------------------------------

下面是_ftoa函数，传递的参数有结构体，整形变量以及浮点数。在函数中还有临时的变量等       
我们看具体的栈中分配情况（在程序编译后，当前函数的栈分配已经确定）。

```c
// internal ftoa for fixed decimal floating point
static size_t _ftoa(out_fct_type out, char* buffer, size_t idx, 
	                size_t maxlen, double value, unsigned int prec, 
	                unsigned int width, unsigned int flags)
{
  char buf[PRINTF_FTOA_BUFFER_SIZE];
  size_t len  = 0U;
  double diff = 0.0;

  // powers of 10
  static const double pow10[] = { 1, 10, 100, 1000, 10000, 100000, 
                                 1000000, 10000000, 100000000, 1000000000 };

  // test for special values
  if (value != value)
    return _out_rev(out, buffer, idx, maxlen, "nan", 3, width, flags);
  if (value < -DBL_MAX)
    return _out_rev(out, buffer, idx, maxlen, "fni-", 4, width, flags);
  if (value > DBL_MAX)
    return _out_rev(out, buffer, idx, maxlen, (flags & FLAGS_PLUS) ? "fni+" : "fni", (flags & FLAGS_PLUS) ? 4U : 3U, width, flags);

  // 省略部分代码

  if (diff > 0.5) {
    ++frac;
    // handle rollover, e.g. case 0.99 with prec 1 is 1.0
    if (frac >= pow10[prec]) {
      frac = 0;
      ++whole;
    }
  }
  else if (diff < 0.5) {
  }
  else if ((frac == 0U) || (frac & 1U)) {
    // if halfway, round up if odd OR if last digit is 0
    ++frac;
  }

  // do whole part, number is reversed
  while (len < PRINTF_FTOA_BUFFER_SIZE) {
    buf[len++] = (char)(48 + (whole % 10));
    if (!(whole /= 10)) {
      break;
    }
  }

  // 省略部分代码

  if (len < PRINTF_FTOA_BUFFER_SIZE) {
    if (negative) {
      buf[len++] = '-';
    }
    else if (flags & FLAGS_PLUS) {
      buf[len++] = '+';  // ignore the space if the '+' exists
    }
    else if (flags & FLAGS_SPACE) {
      buf[len++] = ' ';
    }
  }

  return _out_rev(out, buffer, idx, maxlen, buf, len, width, flags);
}

```

下面是对上面函数_ftoa的反汇编，编译使用-O0，没有使用其他的编译优化。

```
0000000000000000 <_ftoa>:
_ftoa():
   0:	02fd8063 	addi.d      	$sp, $sp, -160
   4:	29c26061 	st.d        	$ra, $sp, 152
   8:	29c24076 	st.d        	$fp, $sp, 144
   c:	02c28076 	addi.d      	$fp, $sp, 160

0000000000000010 <L0^A>:
  10:	29fe62c4 	st.d        	$a0, $fp, -104
  14:	29fe42c5 	st.d        	$a1, $fp, -112
  18:	29fe22c6 	st.d        	$a2, $fp, -120
  1c:	29fe02c7 	st.d        	$a3, $fp, -128
  20:	2bfde2c0 	fst.d       	$fa0, $fp, -136
  24:	0015010c 	move        	$t0, $a4
  28:	0015012e 	move        	$t2, $a5
  2c:	0015014d 	move        	$t1, $a6
  30:	29bdd2cc 	st.w        	$t0, $fp, -140
  34:	001501cc 	move        	$t0, $t2
  38:	29bdc2cc 	st.w        	$t0, $fp, -144
  3c:	001501ac 	move        	$t0, $t1
  40:	29bdb2cc 	st.w        	$t0, $fp, -148
  44:	29ffa2c0 	st.d        	$zero, $fp, -24
  48:	29ff22c0 	st.d        	$zero, $fp, -56
  4c:	293f9ec0 	st.b        	$zero, $fp, -25
  50:	2bbde2c0 	fld.d       	$fa0, $fp, -136
  54:	0114a801 	movgr2fr.d  	$fa1, $zero
  58:	0c218400 	fcmp.slt.d  	$fcc0, $fa0, $fa1
  5c:	48001c00 	bceqz       	$fcc0, 28	# 78 <.L2>
  60:	0280040c 	li.w        	$t0, 1
  64:	293f9ecc 	st.b        	$t0, $fp, -25
  68:	0114a801 	movgr2fr.d  	$fa1, $zero
  6c:	2bbde2c0 	fld.d       	$fa0, $fp, -136
  70:	01030020 	fsub.d      	$fa0, $fa1, $fa0
  74:	2bfde2c0 	fst.d       	$fa0, $fp, -136

  // 省去部分代码

0000000000000560 <.L39>:
 560:	28bdb2cc 	ld.w        	$t0, $fp, -148
 564:	0340218c 	andi        	$t0, $t0, 0x8
 568:	0040818c 	slli.w      	$t0, $t0, 0x0
 56c:	40002180 	beqz        	$t0, 32	# 58c <.L37>
 570:	28ffa2cc 	ld.d        	$t0, $fp, -24
 574:	02c0058d 	addi.d      	$t1, $t0, 1
 578:	29ffa2cd 	st.d        	$t1, $fp, -24
 57c:	02ffc18c 	addi.d      	$t0, $t0, -16
 580:	0010d98d 	add.d       	$t1, $t0, $fp
 584:	0280800c 	li.w        	$t0, 32
 588:	293ec1ac 	st.b        	$t0, $t1, -80

000000000000058c <.L37>:
 58c:	24ff6ecd 	ldptr.w     	$t1, $fp, -148
 590:	24ff72cc 	ldptr.w     	$t0, $fp, -144
 594:	02fe82ce 	addi.d      	$t2, $fp, -96
 598:	001501ab 	move        	$a7, $t1
 59c:	0015018a 	move        	$a6, $t0
 5a0:	28ffa2c9 	ld.d        	$a5, $fp, -24
 5a4:	001501c8 	move        	$a4, $t2
 5a8:	28fe02c7 	ld.d        	$a3, $fp, -128
 5ac:	28fe22c6 	ld.d        	$a2, $fp, -120
 5b0:	28fe42c5 	ld.d        	$a1, $fp, -112
 5b4:	28fe62c4 	ld.d        	$a0, $fp, -104
 5b8:	54000000 	bl          	0	# 5b8 <.L37+0x2c>
 5bc:	0015008c 	move        	$t0, $a0
 5c0:	00150184 	move        	$a0, $t0
 5c4:	28c26061 	ld.d        	$ra, $sp, 152

00000000000005c8 <L0^A>:
 5c8:	28c24076 	ld.d        	$fp, $sp, 144
 5cc:	02c28063 	addi.d      	$sp, $sp, 160
 5d0:	4c000020 	ret
```

栈分布如下所示：

![StackFrame-Case2](../../img/stackframe_Layout-example-2.drawio.svg)

我们可以清楚的在图上看出变量在栈中的位置。

排布规则也是按照上节的调用规约。



## 优化对栈帧的影响

加优化选项-O2后，反汇编的代码如下所示：

```
000000000000000c <_ftoa>:
_ftoa():
   c:	0114a801 	movgr2fr.d  	$fa1, $zero
  10:	02ff4063 	addi.d      	$sp, $sp, -48
  14:	0c218400 	fcmp.slt.d  	$fcc0, $fa0, $fa1
  18:	29c08076 	st.d        	$fp, $sp, 32
  1c:	29c0a061 	st.d        	$ra, $sp, 40
  20:	02c0c076 	addi.d      	$fp, $sp, 48

0000000000000024 <L0^A>:
  24:	00150132 	move        	$t6, $a5
  28:	4803e100 	bcnez       	$fcc0, 992	# 408 <.L85>
  2c:	00150013 	move        	$t7, $zero
  30:	03400000 	nop

0000000000000034 <.L2>:
  34:	0350014d 	andi        	$t1, $a6, 0x400
  38:	0280180f 	li.w        	$t3, 6
  3c:	0013350c 	maskeqz     	$t0, $a4, $t1
  40:	0013b5ed 	masknez     	$t1, $t3, $t1
  44:	00150009 	move        	$a5, $zero
  48:	02ff42ce 	addi.d      	$t2, $fp, -48
  4c:	0280240f 	li.w        	$t3, 9
  50:	0280c011 	li.w        	$t5, 48
  54:	0015358c 	or          	$t0, $t0, $t1
  58:	02808010 	li.w        	$t4, 32
  5c:	03400000 	nop
  60:	03400000 	nop
  64:	03400000 	nop

0000000000000068 <.L6>:
  68:	6c001dec 	bgeu        	$t3, $t0, 28	# 84 <.L7>
  6c:	02c00529 	addi.d      	$a5, $a5, 1
  70:	0010a5cd 	add.d       	$t1, $t2, $a5
  74:	293ffdb1 	st.b        	$t5, $t1, -1
  78:	02bffd8c 	addi.w      	$t0, $t0, -1
  7c:	5fffed30 	bne         	$a5, $t4, -20	# 68 <.L6>
  80:	03400000 	nop

0000000000000084 <.L7>:
  84:	011a8803 	ftintrz.w.d 	$fa3, $fa0
  88:	00df018f 	bstrpick.d  	$t3, $t0, 0x1f, 0x0
  8c:	011d2061 	ffint.d.w   	$fa1, $fa3
  90:	01030401 	fsub.d      	$fa1, $fa0, $fa1
  94:	1a000010 	pcalau12i   	$t4, 0
  98:	02c00210 	addi.d      	$t4, $t4, 0
  9c:	00410def 	slli.d      	$t3, $t3, 0x3
  a0:	38343e04 	fldx.d      	$fa4, $t4, $t3
  a4:	1a00000d 	pcalau12i   	$t1, 0
  a8:	2b8001a2 	fld.d       	$fa2, $t1, 0
  ac:	01051021 	fmul.d      	$fa1, $fa1, $fa4
  b0:	0114b46d 	movfr2gr.s  	$t1, $fa3
  b4:	0c238440 	fcmp.sle.d  	$fcc0, $fa2, $fa1
  b8:	48017d00 	bcnez       	$fcc0, 380	# 234 <L0^A>
  bc:	011aa822 	ftintrz.l.d 	$fa2, $fa1
  c0:	0114b84f 	movfr2gr.d  	$t3, $fa2
  c4:	600191e0 	bltz        	$t3, 400	# 254 <.L12>
  c8:	03400000 	nop

00000000000000cc <.L88>:
  cc:	0114a9e2 	movgr2fr.d  	$fa2, $t3
  d0:	1a000010 	pcalau12i   	$t4, 0
  d4:	011d2842 	ffint.d.l   	$fa2, $fa2
  d8:	01030821 	fsub.d      	$fa1, $fa1, $fa2
  dc:	2b800202 	fld.d       	$fa2, $t4, 0
  e0:	0c218440 	fcmp.slt.d  	$fcc0, $fa2, $fa1
  e4:	4801a000 	bceqz       	$fcc0, 416	# 284 <.L77>
  e8:	03400000 	nop

// 省略了部分汇编代码

000000000000046c <L0^A>:
 46c:	28c08076 	ld.d        	$fp, $sp, 32
 470:	02c0c063 	addi.d      	$sp, $sp, 48

0000000000000474 <L0^A>:
 474:	4c000020 	ret
 478:	03400000 	nop
 47c:	03400000 	nop
 480:	03400000 	nop
 484:	03400000 	nop
 488:	03400000 	nop
 48c:	03400000 	nop
 490:	03400000 	nop

0000000000000494 <L0^A>:
 494:	0340214c 	andi        	$t0, $a6, 0x8
 498:	43fd619f 	beqz        	$t0, -672	# 1f8 <.L37>
 49c:	0010d92c 	add.d       	$t0, $a5, $fp
 4a0:	0280800d 	li.w        	$t1, 32
 4a4:	293f418d 	st.b        	$t1, $t0, -48
 4a8:	02c00529 	addi.d      	$a5, $a5, 1
 4ac:	53fd4fff 	b           	-692	# 1f8 <.L37>
 4b0:	03400000 	nop
 4b4:	03400000 	nop
 4b8:	03400000 	nop
 4bc:	03400000 	nop
 4c0:	03400000 	nop
 4c4:	03400000 	nop
 4c8:	03400000 	nop

00000000000004cc <.L90>:
 4cc:	034005f0 	andi        	$t4, $t3, 0x1
 4d0:	43fc4a1f 	beqz        	$t4, -952	# 118 <.L15>
 4d4:	02c005ef 	addi.d      	$t3, $t3, 1
 4d8:	53fdc3ff 	b           	-576	# 298 <.L93>
 4dc:	03400000 	nop

00000000000004e0 <.L38>:
 4e0:	00df024c 	bstrpick.d  	$t0, $t6, 0x1f, 0x0
 4e4:	6bfe292c 	bltu        	$a5, $t0, -472	# 30c <.L35>
 4e8:	53fd13ff 	b           	-752	# 1f8 <.L37>
 4ec:	03400000 	nop

00000000000004f0 <.L87>:
 4f0:	43fce65f 	beqz        	$t6, -796	# 1d4 <.L33>
 4f4:	43ff267f 	beqz        	$t7, -220	# 418 <.L34>
 4f8:	02bffe52 	addi.w      	$t6, $t6, -1
 4fc:	00df024c 	bstrpick.d  	$t0, $t6, 0x1f, 0x0
 500:	6bfe0d2c 	bltu        	$a5, $t0, -500	# 30c <.L35>
 504:	5ffce131 	bne         	$a5, $t5, -800	# 1e4 <.L36>
 508:	53fcf3ff 	b           	-784	# 1f8 <.L37>
```

栈分布如下所示：

![StackFrame-Case2](../../img/stackframe_Layout-example-2-O2.drawio.svg)

可以看出，在使用优化选项后，除了一些必须保存的``ra``,``fp``等，      
**内部临时的变量的位宽如果是能够在寄存器中放得下，就将临时变量提升到寄存器中**      
避免将这些变量保存在内存中，而频繁访存（访存相对于其他指令的执行来说，是很耗时的），导致性能降低。


## longjump和setjump

`setjmp` 和 `longjmp` 是 C 语言中用于实现**非本地跳转**（Non-Local Jump）的核心函数，允许程序直接跳转到之前保存的执行点，突破传统函数调用栈的限制。     

---

### 工作机制
1. **`setjmp`：保存执行环境**  
   - **功能**：将当前程序的上下文（包括栈指针、寄存器状态等）保存到 `jmp_buf` 类型的变量中，并首次调用时返回 `0`。  
   - **语法**：  
     ```c
     int setjmp(jmp_buf env);
     ```  
   - **返回值规则**：  
     - 直接调用时返回 `0`。  
     - 通过 `longjmp` 跳转回来时返回非零值（由 `longjmp` 的第二个参数指定）。

2. **`longjmp`：恢复执行环境**  
   - **功能**：从 `jmp_buf` 恢复之前保存的上下文，强制程序跳转回 `setjmp` 的调用位置，并修改其返回值。  
   - **语法**：  
     ```c
     void longjmp(jmp_buf env, int val);
     ```  
   - **特性**：  
     - 不返回正常流程，而是“伪装”成从 `setjmp` 返回。  
     - 若 `val` 为 `0`，`setjmp` 返回 `1`（避免与直接调用混淆）。

---

### 核心用途
#### 跨函数异常处理 
   - **场景**：在深层嵌套的函数调用中，若某层发生错误，可直接跳转到顶层的错误处理代码，无需逐层返回。  
   - **示例**：  
     ```c
     #include <setjmp.h>
     #include <stdio.h>

     jmp_buf env;

     void func3() {
         printf("Error in func3, jumping back...\n");
         longjmp(env, 1);  // 跳转回 setjmp 处
     }

     void func2() {
         func3();
     }

     void func1() {
         func2();
     }

     int main() {
         if (setjmp(env) == 0) {
             func1();  // 正常流程
         } else {
             printf("Error handled in main.\n");  // 异常处理
         }
         return 0;
     }
     ```  
     **输出**：  
     ```
     Error in func3, jumping back...
     Error handled in main.
     ```

#### 资源清理与状态恢复 
   - **场景**：在信号处理（如 `SIGINT`）或协程切换时，跳转到预定义的清理代码，避免资源泄漏。  
   - **示例**：  
     ```c
     #include <signal.h>
     #include <setjmp.h>
     #include <stdio.h>

     jmp_buf sig_env;

     void handler(int sig) {
         printf("Signal %d received, jumping back...\n", sig);
         longjmp(sig_env, 1);
     }

     int main() {
         if (setjmp(sig_env) == 0) {
             signal(SIGINT, handler);  // 捕获 Ctrl+C
             printf("Press Ctrl+C to test...\n");
             while (1);  // 无限循环等待信号
         } else {
             printf("Resumed after signal handling.\n");
         }
         return 0;
     }
     ```

#### 协作式多任务与状态机 
   - **场景**：手动控制程序流程，实现轻量级协程或状态切换。  
   - **示例**：  
     ```c
     #include <setjmp.h>
     #include <stdio.h>

     jmp_buf state_env;
     int current_state = 0;

     void state1() {
         printf("State 1: Transitioning to State 2...\n");
         current_state = 2;
         longjmp(state_env, 1);
     }

     void state2() {
         printf("State 2: Transitioning to State 1...\n");
         current_state = 1;
         longjmp(state_env, 1);
     }

     int main() {
         if (setjmp(state_env) == 0) {
             current_state = 1;
             state1();
         } else {
             printf("Current State: %d\n", current_state);
             if (current_state == 1) state2();
         }
         return 0;
     }
     ```

---

### 关键特性与限制
#### 1. **优势**  
   - **跨函数跳转**：突破 `goto` 的函数内限制，直接跳转至任意保存的上下文。  
   - **高效性**：相比异常处理机制（如 C++ 的 `try-catch`），无运行时开销。

#### 2. **风险与限制**  
   - **资源泄漏**：跳转会绕过局部变量析构和栈展开（Stack Unwinding），可能导致内存泄漏。  
   - **栈帧失效**：若跳转点所在的函数已返回，`longjmp` 将导致未定义行为。  
   - **可维护性差**：滥用会导致代码逻辑混乱，增加调试难度。

#### 3. **与现代异常处理的对比**  
   | **特性**       | `setjmp`/`longjmp`                     | C++ 异常处理                     |
   |----------------|---------------------------------------|----------------------------------|
   | **类型安全**   | 无类型检查，需手动处理返回值           | 支持类型安全的 `catch` 块        |
   | **资源管理**   | 无法自动释放栈上资源                   | 通过析构函数自动清理（RAII）     |
   | **可读性**     | 代码跳转路径不直观                     | 异常传播路径清晰                 |

---



`setjmp` 和 `longjmp` 的底层实现相对比较简单。下面是相关的汇编代码。


longjmp核心函数：

```asm
.global _longjmp
.global longjmp
.type   _longjmp,@function
.type   longjmp,@function
_longjmp:
longjmp:
	ld.d    $ra, $a0, 0
	ld.d    $sp, $a0, 8
	ld.d    $r21,$a0, 16
	ld.d    $fp, $a0, 24
	ld.d    $s0, $a0, 32
	ld.d    $s1, $a0, 40
	ld.d    $s2, $a0, 48
	ld.d    $s3, $a0, 56
	ld.d    $s4, $a0, 64
	ld.d    $s5, $a0, 72
	ld.d    $s6, $a0, 80
	ld.d    $s7, $a0, 88
	ld.d    $s8, $a0, 96
#ifndef __loongarch_soft_float
	fld.d   $fs0, $a0, 104
	fld.d   $fs1, $a0, 112
	fld.d   $fs2, $a0, 120
	fld.d   $fs3, $a0, 128
	fld.d   $fs4, $a0, 136
	fld.d   $fs5, $a0, 144
	fld.d   $fs6, $a0, 152
	fld.d   $fs7, $a0, 160
#endif
	sltui   $a0, $a1, 1
	add.d   $a0, $a0, $a1  # a0 = (a1 = 0) ? 1 : a1
	jr      $ra
```

setjmp核心函数：

```asm
.global __setjmp
.global _setjmp
.global setjmp
.type __setjmp,@function
.type _setjmp,@function
.type setjmp,@function
__setjmp:
_setjmp:
setjmp:
	st.d    $ra, $a0, 0
	st.d    $sp, $a0, 8
	st.d    $r21,$a0, 16
	st.d    $fp, $a0, 24
	st.d    $s0, $a0, 32
	st.d    $s1, $a0, 40
	st.d    $s2, $a0, 48
	st.d    $s3, $a0, 56
	st.d    $s4, $a0, 64
	st.d    $s5, $a0, 72
	st.d    $s6, $a0, 80
	st.d    $s7, $a0, 88
	st.d    $s8, $a0, 96
#ifndef __loongarch_soft_float
	fst.d   $fs0, $a0, 104
	fst.d   $fs1, $a0, 112
	fst.d   $fs2, $a0, 120
	fst.d   $fs3, $a0, 128
	fst.d   $fs4, $a0, 136
	fst.d   $fs5, $a0, 144
	fst.d   $fs6, $a0, 152
	fst.d   $fs7, $a0, 160
#endif
	move    $a0, $zero
	jr      $ra

```

可以看出，setjmp函数的核心是保存必要的寄存器：
- ``ra``返回寄存器
- ``sp``栈指针寄存器
- ``r21`` 保留寄存器。为了保持兼容，用户有可能定义特殊用途。
- ``fp/s9`` 帧寄存器或者s9寄存器
- ``s0-s8`` 静态寄存器，被调用者保存的寄存器



