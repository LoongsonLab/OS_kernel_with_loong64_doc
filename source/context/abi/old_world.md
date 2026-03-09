# 旧世界 [^old-and-new-worlds]

[这部分内容主要来自于“怎么龙了吗”阅读材料](https://areweloongyet.com/docs/old-and-new-worlds)

LoongArch 有两套不兼容的软件体系，习惯上大家把它们叫作「旧世界」和「新世界」。
龙芯中科的材料中也有ABI1.0、ABI2.0的提法（目前所见的表述均未在 ABI 与数字之间加空格）。

如果您目前在龙架构电脑上使用 Loongnix、麒麟或者 UOS 这些系统，几个月或一两年之后，一定会有一次全系统升级。

如果您不升级，那么本身外界如何变化也与您无关。


如果您升级，那么升级之后您应该也感受不到使用上的差别，这其实就是「移民新世界」了。

如果您目前在龙架构电脑上使用 Arch、Gentoo 等等这些系统，那么您已经是新世界住民了，这一切也与您无关。

会遇到困扰的目前来看只有：

您使用 Loongnix、麒麟或者 UOS 这些系统，但自行编译了一些要用的软件。      
在未来那次全系统升级之后，您自行编译的软件应该不再能工作，需要重新编译或从系统包管理器安装。

**旧世界**是指最早在龙芯中科内部适配的、随着 LoongArch 公开一并发布的那个 LoongArch 软件生态。

**新世界**是指龙芯中科与社区同仁一道，以典型开源社区协作模式打造的，完全开源的 LoongArch 软件生态。

两个世界的产生是龙芯中科对 LoongArch 采取了秘密开发、突然全盘推出的商业策略，      
由于未能预见到这一版工作有些地方不得不做不兼容修改，而使客户和自身不得不面对的无奈后果。      

按照目前的趋势和一些公开消息，未来旧世界将逐渐消亡。

从龙芯 3A6000 一代产品起，相关产品的出厂配套固件都已达到兼容新、旧世界的状态，      
但就 2024 年 2 月初的现状而言，可能发行版方面（Loongnix 及其他商业发行版）仍需更长时间才能完成迁移，    
因此未能赶上 3A6000 的正式发布。

新旧世界的说法仅仅被用来区分两个不兼容的 LoongArch 生态。      
MIPS 型号的龙芯既不是新世界也不是旧世界。      
一般只会说「MIPS 时代的龙芯」（the MIPS-era Loongson）怎么怎么样。    

「旧世界」、「新世界」的名词形式英译即为「the old world」、「the new world」。   
作形容词时一般以连字符连接前后部分即「old-world」、「new-world」。    
如果在一段话中频繁使用，有时也会用「OW」、「NW」的缩写形式。    


[^old-and-new-worlds]: 具体可查看阅读材料[咱们龙了吗？](https://areweloongyet.com/docs/old-and-new-worlds)


# 新旧世界的区分

如果符合以下任一条件，你就在用**旧世界**：

* 系统是麒麟、Loongnix、UOS 其中之一
* 内核版本以 4.19 开头
* 有 WPS 用但没有安装过 `libLoL` 等旧世界兼容方案

如果一条都没中，你就在用**新世界**。


可以使用 `file` 工具方便地检查一个二进制程序属于哪个世界。
假设你想检查 `someprogram` 这个文件，就执行 `file someprogram`，如果输出的行含有这些字样：

```
interpreter /lib64/ld.so.1, for GNU/Linux 4.15.0
```

就表明这是一个旧世界程序。

相应地，如果输出的行含有这些字样：

```
interpreter /lib64/ld-linux-loongarch-lp64d.so.1, for GNU/Linux 5.19.0
```

就表明这是一个新世界程序。

以上的判断都适用于系统 libc 为 glibc 且动态链接的程序。如果程序是静态链接的，便没有 interpreter 信息；      
如果程序是 Go 语言的或者使用了 musl 作为 C 库，那么文件里就没有对应到 `for GNU/Linux` 这部分信息的标记。      
这种时候试着运行一下就可以了，「异世界」的程序几乎没有可能正常启动。     


## 新世界与旧世界的区别

**源码开放程度不一样**

新世界都是开源代码，而旧世界的部分底层代码由于知识产权等原因始终没有开放，尽管其中也有一部分后来放出了。      
比方说旧世界的 binutils、gcc 在最初发布之后过了几个月有了完整源码，Linux 源码直到 2023 年才有，      
但 GSGPU 的 shader 编译器源码就始终没有。      

放出的源码基本也比较少有完整的 Git 提交历史，因此不便基于它二次修改或者将其移植到上游新版本。

**可用的发行版不一样**      

由于外界拿不到旧世界的完整源码，旧世界发行版只有几个商业公司能做。       
社区制作的发行版都属于新世界。     

目前已知的旧世界发行版（移植）有：（按英文名字母顺序排序）

* 麒麟 (Kylin)
* Loongnix
* UOS

目前已知的新世界发行版（移植）有：（按英文名字母顺序排序）

* [ALT Linux](https://www.altlinux.org/Ports/loongarch64)
* [AOSC OS](https://aosc.io/zh-cn)
* [CLFS 手册与成品](https://github.com/sunhaiyong1978/CLFS-for-LoongArch)
* [Debian](https://wiki.debian.org/Ports/loong64)
* [Fedora LoongArch Remix](https://github.com/fedora-remix-loongarch/releases-info)
* [Gentoo](https://wiki.gentoo.org/wiki/Project:LoongArch)
* [Loong Arch Linux](https://github.com/loongarchlinux)
* [Slackware](https://github.com/shipujin/slackware-loongarch64)
* [Yongbao](https://github.com/sunhaiyong1978/Yongbao)


**软件版本不一样**      

旧世界的基础组件版本主要跟随当初移植时基于的 Debian 或 RHEL 大版本。因为商业公司不一定有优先级（或者能力）去关心跟进新版本的事情，       
所以旧世界的基础组件版本几乎不会有大的更新。
视具体用户场景和开发、部署习惯而定，有时候这是个好事，有时候很糟心。          

以下是一些常见软件、开发工具在两个世界的版本对比：

|软件|旧世界版本|新世界版本|
|----|----------|----------|
|Linux|4.19|&ge; 5.19，常见 &ge; 6.1|
|binutils|2.31|&ge; 2.38，常见 &ge; 2.40|
|gcc|8.3|&ge; 12.1，常见 &ge; 13.1|
|glibc|2.28|&ge; 2.36|
|LLVM|8|&ge; 16|
|Node.js|14.16.1|&ge; 18|
|Go|1.15、1.18、1.19|&ge; 1.19|
|Rust|1.41、1.58|&ge; 1.71|



