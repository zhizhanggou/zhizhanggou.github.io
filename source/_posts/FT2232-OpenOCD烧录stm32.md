---
title: FT2232+OpenOCD烧录stm32
date: 2025-02-04 21:43:40
categories: 嵌入式
tags: openocd
---
# 环境搭建
+ ubuntu 环境下先进行openocd的安装
```
sudo apt-get install openocd
```

+ 安装依赖
```
apt-get install libusb-1.0-0-dev libftdi-dev
```

# 配置文件
+ 先创建ft2232的配置文件ft2232.cfg (文件名随意)

    ```
    adapter driver ftdi
    ftdi_vid_pid 0x0403 0x6010

    adapter speed 1500
    # ftdi_tdo_sample_edge falling
    transport select swd

    ftdi_layout_init 0x0508 0x0f1b
    ftdi_layout_signal SWD_EN -data 0

    ftdi_layout_signal nSRST -data 0x0010

    ```
  openocd还有很多设备的配置文件，像stlink、ulink、jlink之类的，配置文件路径默认在
  /usr/share/openocd/scripts/interface


+ 芯片相关配置的配置
  芯片相关配置的路径在/usr/share/openocd/scripts/target
  这里需要烧录的芯片是stm32f411ce，所以需要使用配置文件stm32f4x.cfg
  为方便起见，我把ft2232.cfg和stm32f4x.cfg都放在了~/openocd目录下
  
# 烧录脚本
使用bash脚本来实现烧录指令
stm32_flash.sh
```
#!/bin/bash
SOURCE=~/openocd
INTERFACE_CFG=${SOURCE}/ft2232.cfg
TARGET_CFG=${SOURCE}/stm32f4x.cfg

openocd -f ${INTERFACE_CFG}                     \
        -f ${TARGET_CFG}                        \
        -c "init"                               \
        -c "halt"                               \
        -c "flash write_image erase ${1}"       \
        -c "reset"                              \
        -c "exit"
```

加好环境变量与执行权限后这样就可以用如下指令直接烧录固件(hex)。
```
./stm32_flash.sh <固件路径>
```
要烧录其他类型的固件可以修改脚本指令。
比如bin文件的话就加上起始地址，改为flash write_image erase ${1} 0x08000000