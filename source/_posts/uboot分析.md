---
title: uboot分析
date: 2019-08-18 17:04:23
categories: 嵌入式
---

uboot初始化的第一阶段(硬件相关)：

1、设置SVC模式

2、关看门狗

3、屏蔽中断

4、初始化SDRAM

5、设置栈

6、设置时钟

7、将代码从flash拷贝到SDRAM中

8、清BSS段

9、调用start_armboot