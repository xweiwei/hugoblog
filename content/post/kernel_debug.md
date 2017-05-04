+++
css = []
date = "2017-05-04T22:03:15+08:00"
description = ""
draft = false
highlight = true
scripts = []
tags = ["linux", "debug"]
title = "Android内核调试"
index = true
+++


在分析Linux的内核CVE，或者是编写相关的POC，有时候可能结果与预期不太符合，这个时候就需要查看内核的运行过程。


# KGDB


# 编译内核

首先是去`https://source.android.com/source/building-kernels`下载对应的内核源代码。

msm对应大部分的nexus以及pixel手机
`
git clone https://android.googlesource.com/kernel/msm.git
`

下面以nexus5x(bullhead)为例子,生成一个.config
```
$ export ARCH=arm64
$ export CROSS_COMPILE=aarch64-linux-android-
$ cd msm
$ git checkout android-msm-bullhead-3.10-nougat
$ make bullhead_defconfig

```

然后配置.config

```
$ make menuconfig 
```


主要开启
```
[*] Kernel hacking 
     [*] Compile the kernel with debug info 
     [*] KGDB: kernel debugging with remote gdb —>
[*] Enable dynamic printk() call support 

```

编译得到的arch/arm64/boot/Image



# GDB调试

模拟器启动
`emulator -verbose -show-kernel -netfast -avd -avd Pixel_API_25 -kernel arch/arm64/boot/Image -qemu -gdb tcp::1234,ipv4 -S`

-verbose -show-kernel选项可以看到内核的详细输出，-no-window -no-audio选项可以不启动界面，-qumu -s -S选项可以启动调试监听让内核启动时在端口1234等待。

利用目标平台的gdb去加载vmlinux
```
aarch64-linux-android-gdb ./vmlinux
target remote localhost:1234
```

内核已经跑起来了，这个时候直接c就可以继续执行了。


# 优化问题
但是在编译的时候默认是-O2优化，所以查看变量时会出现`optimized out`
```
(gdb) p copied
$1 = <optimized out>

```


用-O0的选项重新编译内核。
> gdb调试程序的时候打印变量值会出现<value optimized out> 情况,可以在gcc编译的时候加上 -O0参数项,意思是不进行编译优化,调试的时候就会顺畅了,运行流程不会跳来跳去的,发布项目的时候记得不要在使用 -O0参数项,gcc 默认编译或加上-O2优化编译会提高程序运行速度. 

但是在修改成-O0之后，编译内核代码会出现问题

```
In file included from ./arch/arm64/include/asm/bitops.h:19:0,
                 from include/linux/bitops.h:36,
                 from arch/arm64/mm/context.c:20:
In function '__xchg',
    inlined from 'flush_context' at arch/arm64/mm/context.c:58:10:
include/linux/compiler.h:437:38: error: call to '__compiletime_assert_67' declared with attribute error: BUILD_BUG failed
  _compiletime_assert(condition, msg, __compiletime_assert_, __LINE__)
                                      ^
include/linux/compiler.h:420:4: note: in definition of macro '__compiletime_assert'
    prefix ## suffix();    \
    ^
include/linux/compiler.h:437:2: note: in expansion of macro '_compiletime_assert'
  _compiletime_assert(condition, msg, __compiletime_assert_, __LINE__)
  ^
include/linux/bug.h:50:37: note: in expansion of macro 'compiletime_assert'
 #define BUILD_BUG_ON_MSG(cond, msg) compiletime_assert(!(cond), msg)
                                     ^
include/linux/bug.h:84:21: note: in expansion of macro 'BUILD_BUG_ON_MSG'
 #define BUILD_BUG() BUILD_BUG_ON_MSG(1, "BUILD_BUG failed")
                     ^
./arch/arm64/include/asm/cmpxchg.h:67:3: note: in expansion of macro 'BUILD_BUG'
   BUILD_BUG();
   ^
In function '__xchg',
    inlined from 'check_and_switch_context' at arch/arm64/mm/context.c:153:9:
include/linux/compiler.h:437:38: error: call to '__compiletime_assert_67' declared with attribute error: BUILD_BUG failed
  _compiletime_assert(condition, msg, __compiletime_assert_, __LINE__)
                                      ^
include/linux/compiler.h:420:4: note: in definition of macro '__compiletime_assert'
    prefix ## suffix();    \
    ^
include/linux/compiler.h:437:2: note: in expansion of macro '_compiletime_assert'
  _compiletime_assert(condition, msg, __compiletime_assert_, __LINE__)
  ^
include/linux/bug.h:50:37: note: in expansion of macro 'compiletime_assert'
 #define BUILD_BUG_ON_MSG(cond, msg) compiletime_assert(!(cond), msg)
                                     ^
include/linux/bug.h:84:21: note: in expansion of macro 'BUILD_BUG_ON_MSG'
 #define BUILD_BUG() BUILD_BUG_ON_MSG(1, "BUILD_BUG failed")
                     ^
./arch/arm64/include/asm/cmpxchg.h:67:3: note: in expansion of macro 'BUILD_BUG'
   BUILD_BUG();

```

内核中有大量的代码是根据优化的，所以关闭所有优化会出现问题。
这里指定-O(-O1)

> 另外开始编译内核之前请注意Linux源码目录下Makefile中的优化选项，默认的Linux内核的编译都以-O2的优化级别进行。在这个优化级别之下，编译器要对内核中的某些代码的执行顺序进行改动，所以在调试时会出现程序语句执行顺序与代码中的顺序不一致的情况。可以把Makefile中的-O2选项改为-O,但不可去掉-O，不然编译不过:
```
ifdef CONFIG_CC_OPTIMIZE_FOR_SIZE
KBUILD_CFLAGS   += -Os
else
#KBUILD_CFLAGS   += -O2
KBUILD_CFLAGS   += -O
endif

```
[gdb 关于value optimized out](http://dsl000522.blog.sohu.com/180439264.html)


编译kernel不使用-O2的补丁
[build kernel without -O2 option](https://sourceware.org/ml/gdb/2010-12/msg00009.html)

# 参考

[Android Linux内核编译调试](http://www.joenchen.com/archives/1093)

[内核调试](http://freemandealer.github.io/2014/11/29/debug-android-kernel/)

[KGDB + Eclipse debug Kernel on i.mx6 Android](http://fatalfeel.blogspot.com/2013/09/kgdb-eclipse-debug-kernel-on-imx6.html)

[Practical Android Debugging Via KGDB](http://blog.trendmicro.com/trendlabs-security-intelligence/practical-android-debugging-via-kgdb/)

[用 kGDB 调试 Linux 内核](http://tinylab.org/kgdb-debugging-kernel/)

[Enabling KGDB for Android](http://bootloader.wikidot.com/android:kgdb)

[Android内核编译调试](https://geneblue.github.io/2016/01/27/Android%E5%86%85%E6%A0%B8%E7%BC%96%E8%AF%91%E8%B0%83%E8%AF%95/)

[KGDB 在ARM平台上的交叉编译和使用](http://blog.hibeiyu.com/archives/432)


