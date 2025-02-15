title: 搭建跳板机
author: Fighting
tags:
  - linux
  - ssh
  - drawboard
categories:
  - linux
date: 2018-03-12 10:44:00
---
#### 前言

近遇到这样一个问题，我在阿里云室架设了一台服务器，发现外部网元对服务器攻击非常平凡，都知道服务器开放出来的端口越多，对服务器的风险越大。如何来规避这些风险呢，下面具体说明在阿里云上搭建一个跳板机来访问内网服务器，提升服务器的安全。

##### 描述一下目前的机器状况，梳理梳理：

| 机器 | IP | 用户名 | 备注 |
| :-- | :-- | :-- | :-- |
| A | 123.78.150.91 | root | 外网服务器，相当于桥梁的作用，跳板机 |
| B | 172.18.62.123 | root | 目标服务器，处于内网 |
| C | 123.123.123.123 | win | 自己的电脑 |

<!--more-->

##### 解决方法
通俗地说：就是在机器C上使用xshell/mobaXterm软件连接A机器，自动转向到B机器上。

##### 实现前的准备
每台都要安装ssh的客户端，在这里我使用的是centos7，都自带ssh。如果是使用其他版本Linux，请手动Google一下咯。

##### 介绍一下使用到的ssh参数

**反向代理**
`ssh -fCNR`

**正向代理**
`ssh -fCNL`

> 
-f 后台执行ssh指令
-C 允许压缩数据
-N 不执行远程指令
-R 将远程主机(服务器)的某个端口转发到本地端指定机器的指定端口
-L 将本地机(客户机)的某个端口转发到远端指定机器的指定端口
-p 指定远程主机的端口

##### 首先在A上面操作
建立A机器到B机器的反向代理，具体指令为
`ssh -fCNR [B机器IP或省略]:[B机器端口]:[A机器的IP]:[A机器端口] [登陆B机器的用户名@服务器IP]`

在这里我使用了B机器的2222端口，以及A机器的22端口，按照上面的指令就是这样子的操作

```shell
ssh -ngfNTR 2222:172.18.62.123:22 root@123.78.150.91 -o ServerAliveInterval=300
```

检验是否已经启动了可以使用ps aux | grep ssh指令来查看

##### 阿里云打开端口

A服务器要打开2222
B服务器要打开22，仅限A服务器访问

##### 接着在B上面操作

建立B机器的ssh-key

```shell
ssh-keygen
```

然后一直按回车就可以了，将`~/ssh/id_rsa.pub`拷贝给A服务器`~/ssh/authorized_keys`

##### 在B上用autossh建立稳定隧道

centos7上没有默认安装`autossh`的，所以使用一下命令安装

```shell
yum install autossh
```

在B服务器上使用autossh建立稳定隧道

```shell
autossh -p 22 -M 6777 -fNR 2222:127.0.0.1:22 root@123.78.150.91
```

> 命令说明
-M 6777 这个表示是监控端口，检测到这个端口不通会重新连接
2222:127.0.0.1:22  2222表示需要在远程主机上开启的端口，22表示本地的ssh端口
root@123.78.150.91 -p 22 root为vps上面的用户，22为vps ssh端口

**现在就可以在A服务器使用SSH端口为2222访问B服务器了**