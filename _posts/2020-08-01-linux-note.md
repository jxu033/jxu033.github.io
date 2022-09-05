---
title: "Linux note"
layout: post
date: 2020-08-01 22:44
tag:
- linux
star: true
category: blog
author: jiaqixu
description: note for linux
---

### 目录
* TOC
{:toc}


### fstab

`/etc/fstab` 是用来存放文件系统的静态信息的文件。其位于`/etc/`目录下，可以通过`vim /etc/fstab`来修改。
当系统启动的时候，系统会自动地从这个文件读取信息，并且会自动将此文件中指定的文件系统挂载到指定的目录。

![image](/assets/images/blog/linux-fstab.png)

1. **file system**: 用来指定你要挂在的文件系统的设备名称或者块信息，也可以是远程的文件系统
2. **mount point**: 挂载点，也就是自己找一个或者创建一个dir,然后把文件系统挂载到这个目录上，然后就可以从这个目录中访问要挂载的文件系统<br>
对于swap分区，这个域应该填写：none，表示没有挂载点。
3. **type**: 用来指定文件系统的类型
4. **options**：用来填写设置选项，各个选项用逗号隔开。由于选项非常多，而这里篇幅有限，所以不再作详细介绍，如需了解，请用 命令 man mount 来查看。但在这里有个非常重要的关键字需要了解一下：defaults，它代表包含了选项rw,suid,dev,exec,auto,nouser和 async。
5. **dump**。此处为1的话，表示要将整个里的内容备份；为0的话，表示不备份。
6. **pass**。这里用来指定如何使用fsck来检查硬盘

其他相关命令
```text
ls -l /dev/disk/by-uuid # 输出指向可用卷的链接，软链接名称为对应的UUID
fdisk -l # 可以显示出所有挂载和未挂载的分组，但不显示文件系统类型
lsblk -f # 可以查看未挂载的文件系统类型
```

### 免sudo使用docker
将指定用户加入docker组即可
```text
sudo groupadd docker #添加docker组
sudo gpasswd -a ${USER} docker # 将用户加入docker组
sudo service docker restart #重启docker服务
newgrp - docker #切换当前会话到新group或者重启X会话
```

### 给用户admin权限，免密使用sudo
```text
adduser jiaqi
passwd jiaqi(输出密码)
usermod -aG wheel jiaqi #将jiaqi用户加入wheel组
user -a jiaqi -G guan # 将jiaqi用户加入guan组
sudo visudo # 打开visudo文件，输入jiaqi ALL=(ALL) NOPASSWD:ALL (注意，如果用户是%wheel那就是所有的admin了)
```

### github ssh 拉取
一个公钥只允许放在一个github repo的depoly key上或者是SSH and GPG keys;<br>
deploy key和SSH and GPG keys的关系是房间和房子的关系

```text
假设现在有2个repo, 我们想通过ssh 拉取；
可以生成2个public keys，将每一个对应到一个repo的deploy key上；
全部设置好之后，可以通过ssh -T git@github.com查看，是否验证通过。
这时会发现只有一个能拉取，另外一个还是说没有权限，原因是默认使用第一个public key；
想要2个repo都可以ssh 拉取，需要将其中一个进行如下配置：
~/.ssh/config :
Host github2.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_ul-aidp
  
同时在.git/config中修改(也可以使用git remote set-url origin git@github2.com:guandata/ul-aidp.git)：
[remote "origin"]
	url = git@github2.com:guandata/ul-aidp.git
```

### jenkins修改启动用户后，无法启动
```
# 以centos为例
日志文件路径 /var/log/jenkins/jenkins.log
配置文件路径：/etc/sysconfig/jenkins
Jenkins_home目录：包含各种metadata信息(e.g. plugins, users, config.xml)，可在配置文件中重新设置

# 在jenkins配置文件中修改JENKINS_USER后，启动jenkins的时候就会以该用户启动，为保证其能且能正常启动，需要修改一些文件夹权限(可以通过启动时日志给出的信息知道得知)
1. chown -R jiaqi:jenkins /var/log/jenkins
2. chown -R jaiqi:jenkins /var/cache/jenkins
修改完成后，通过sudo service jenkins restart即可成功启动
```

### 网卡















### iptables和filewalld
https://www.linuxprobe.com/chapter-08.html


### ping, telnet, netstat，arp
```
ping(Packet Internet Groper)
目的：测试源主机与目的主机的连通性。
原理：Ping命令工作在OSI(Open System Interconnection Reference Model开放式系统互联模型)的第三层网络层。Ping命令会发送ICMP(Internet控制报文协议, 它是TCP/IP协议簇的一个子协议，用于在IP主机、路由器之间传递控制消息，控制消息是指网络通不通，主机是否可达 、路由是否可用等网络本身的消息) 数据包到目的主机，然后等待从目的主机接收回复的数据包，当目的主机接收到这个数据包时，为源主机发送回复数据包。

telnet(teletype network)
目的：可以用来测试远程机器或本地机器上某端口是否开放
原理：telnet是应用层协议，是Internet远程登录服务的标准协议和主要形式。登录的账号密码与数据铭文传输，不加密。
e.g. telnet <IP> <Port>

netstat(network statistics)
目的：查看所有网络连接，端口使用情况，占用端口的进程等等
常用命令: netstat -lnp | grep <port>

arp(address resolution protocol)
目的： 用于实现从IP地址到MAC地址的映射，即询问目标IP对应的MAC地址。
原理： https://zhuanlan.zhihu.com/p/28771785
```

### tcp和udp
