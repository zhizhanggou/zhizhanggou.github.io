---
title: 用samba搭建私有云
date: 2023-05-14 22:31:02
categories: 折腾
tags: 服务器
---
# 用samba搭建私有云
## 1、在ubuntu上安装samba
`sudo apt-get install samba samba-common`

## 2、修改samba配置文件
`sudo vim /etc/samba/smb.conf `
在最下方添加
<pre>
[share]
comment = share folder
browseable = yes
path = /home/abc/data #需要共享的路径
create mask = 0755
directory mask = 0755  
valid users = abc   #用户名称
force user = abc
force group = abc
public = yes
available = yes
writable = yes </pre>

如果要从外网访问，则要修改端口，（运营商一般默认关闭445端口）

在[global]标签下增加下面的语句，端口号为你想设置的端口
`smb ports = 端口号`

## 3、给共享目录添加权限
`sudo chmod 777 /home/abc/data`

## 4、添加用户并设置访问密码
`sudo smbpasswd -a abc`

## 5、重启samba服务器
`sudo service smbd restart`

如果没有修改端口号，则已经可以在局域网内访问了
在windows端打开运行 win+R
输入双反斜杠加IP或者域名就可以访问
`\\192.168.1.11`

## 6、windows端口转发设置
关闭占用445端口的应用
`sc config LanmanServer start= disabled`
`net stop LanmanServer`
开启ip helper服务
`sc config iphlpsvc start= auto`
设置转发
`netsh interface portproxy add v4tov6 listenport=445 listenaddress=127.0.0.1 connectport=替换为端口号 connectaddress=替换为动态域名`
因为我的域名是ipv6的解析，所以用v4tov6
重启电脑后生效。

## 7、windows下通过访问共享文件夹
注意，此时进行了端口转发，所以不是使用smb服务器的地址！
`\\127.0.0.1`
在输入用户名和访问密码后即可登录。

## 8、ios端访问
&emsp;&emsp;用系统自带的服务器连接工具不能指定端口，需要借助第三方软件ES文件浏览器，在新建中选择SMB，然后输入域名、端口、用户名密码就可以访问。