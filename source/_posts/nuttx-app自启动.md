---
title: nuttx app自启动
date: 2025-02-04 22:45:10
categories: 嵌入式
tags: nuttx
---
nuttx app可以通过启动脚本init.d来实现开机自动运行的功能。
系统版本基于release12.6，后续的版本这个功能好像改了名字。

首先需要在menuconfig中使能
Auto-mount etc baked-in ROMFS image功能.

其次需要使用./tools/mkromfsimg.sh来生成 etc_romfs.c文件。
nuttx目录下创建两个模板文件rc.sysinit.template和rcS.template。(这俩模板应该是对应启动时不一样的执行位置，具体没仔细看)

在rcS.template中写入需要启动执行的指令。比如我自己写的app：serialTest &

&符号表示后台执行。


使用方式为
```
./tools/mkromfsimg.sh build/
```
其中bulid为编译目录。这个脚本的使用要想要基于一些编译相关的东西，具体的还要看下其实现

再把生成的etc_romfs.c文件放入对应的BSP中，我这边所使用的bsp目录是nuttx/boards/arm/stm32/stm32f411-minimum/src

编译后下载。即可在系统启动后自动运行serialTest的相关程序。
