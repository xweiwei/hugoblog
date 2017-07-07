+++
css = []
date = "2017-07-07T12:25:09+08:00"
description = ""
draft = true
highlight = true
metaimage = "/path/to/img.png"
metavideo = "/path/to/video.mp4"
nodisqus = false
notoc = false
scripts = []
sharebuttons = true
tags = []
title = "Android 反混淆的研究"

+++


# Android obfuscation

这里说的主要是在NDK上开发的混淆。
不同与java的混淆概念，proguard处理后的结果与其说是混淆，不如说是只是为了压缩吧。替换有语义的类名，变量名为无语义的字母，去掉没有用到的代码，但是代码的执行流程还是存在的，并不会打乱原有的逻辑块。

NDK上的混淆就比较不同，基本扰乱了代码的逻辑块。
在混淆方面，主要是通过在llvm上增加obfuscate pass。


# 解混淆

虽然是混淆了，但是混淆也是由代码写的，经过混淆出来的汇编码还是有特征的，找到特征后还是很可以在一定程度上还原出逻辑块的流程。

目前看来比较高大上的方式就是利用符号执行的方式，找到从开始到结束流程块的路径，然后通过修改关键的跳转，使得逻辑块复原。

在android上的话，可以利用angr这个二进制分析框架，因为这个支持arm以及其它架构指令集。

另外就是直接的通过特征，找到逻辑块的路径。

