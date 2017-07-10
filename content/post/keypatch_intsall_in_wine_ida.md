+++
css = []
date = "2017-07-10T12:41:59+08:00"
description = ""
draft = false
highlight = true
metaimage = "/path/to/img.png"
metavideo = "/path/to/video.mp4"
nodisqus = false
notoc = false
scripts = []
sharebuttons = true
tags = []
title = "Wine 上的IDA安装keypatch plugin"

+++

# keypatch

[Keypatch](http://www.keystone-engine.org/keypatch/)是基于keystone-engine在IDA上的汇编插件，可以直接用汇编字符串修改原来的指令。

# 在wine上安装


## 安装keystone python module

从[Download](http://www.keystone-engine.org/download/)下载最新的binary，这里都下32位的binary.
直接运行安装文件就行，勾选python2.7的版本


## 安装keypatch

```
cd ${WINE_HOME}/drive_c/Program Files (x86)/IDA 6.8/plugins
wget "https://raw.githubusercontent.com/keystone-engine/keypatch/master/keypatch.py"
```

## 使用
快捷键**`Crtl+ALT+K`**
然后在Assembly上填入相应的指令既可以修改。

