---
title: stm32外部晶振故障检测
date: 2022-06-26 22:35:42
categories: 嵌入式
tags:
---

## 背景 ## 
&emsp;&emsp;在可靠性需求中，需要考虑在外部晶振失效的情况下，能够将系统的晶振源切换为内部的振荡器，让系统至少能够维持能够发出晶振失效告警的状态，如果能让系统保持正常系统正常运行那是最好。

## 方法 ##
&emsp;&emsp;在stm32中，有Clock security system (CSS)的机制，当外部晶振出现问题，比如晶振停振、短时间内出现大的频率偏差就会触发(fae说的，没有找到具体的触发机制文档)，当触发时会系统时钟源会从HSE或者HSE为源的PLL切换到HSI，并进入NMI.(non-maskable interrupt)中断。

![](../images/stm32外部晶振故障检测/cubemx1.PNG)
&emsp;&emsp;当在cubemx中配置Enable CSS后，会在NMI中断中跳转到HAL_RCC_NMI_IRQHandler函数中，可以在该函数中进行时钟的重新配置，将PLL Source Mux的源调整为HSI，并将M\N\P的系数进行调整，使系统时钟与之前保持一致。之后系统会以内部时钟的形式继续运行。经过实测can通讯串口通讯不会受到影响，切换会花费多少时间还需测试。

![](../images/stm32外部晶振故障检测/css_datasheet.jpg)

![](../images/stm32外部晶振故障检测/css_datasheet2.PNG)

## 切换时间测量 ##
&emsp;&emsp;测量切换时间可以使用stm32的MCO(microcontroller clock output)功能，将PLL时钟输出到引脚上，测量该引脚时钟消失的时间就可以得到切换时间。
&emsp;&emsp;实际操作时，示波器没法在时钟消失的时候进行触发(频率触发没搞明白用法)，配置一个GPIO为低电平，在NMI的中断函数中改为高电平，通过上升沿触发的方式来触发示波器。
![](../images/stm32外部晶振故障检测/scope_4.bmp)
从图中可以看出切换时间约为128us的时间。


