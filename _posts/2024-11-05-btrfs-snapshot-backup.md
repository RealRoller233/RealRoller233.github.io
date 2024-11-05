---
layout: post
title: "btrfs snapshot backup"
subtitle: "关于btrfs子卷恢复的经历"
date: 2024-11-05
author: "Real_Roller"
header-img: "img/post-bg-2015.jpg"
tags: []
---
- 概述
```bash
sudo btrfs subvolume snapshot -r @ @ro
sudo btrfs subvolume snapshot -r @home @home-ro
sudo btrfs send /@ro | sudo  btrfs receive /mnt
sudo mount /dev/sdX /mnt
sudo btrfs send /@home-ro | sudo btrfs receive /mnt
```
创建只读子卷并且传到移动硬盘或者其他主机上
如果是其他主机,可采取sshfs方式,或者使用
```bash
sudo btrfs send /mnt/snapshot | ssh user@remote_ip "sudo btrfs receive /mnt/remote_btrfs"
```
此前可以ssh到目标主机上输入`sudo -v`来避免sudo密码问题
将各处路径替换成所需路径,比如/tmp/backup
```bash
sshfs user@host:/path /mnt
```
当需要恢复的时候
```bash
sudo btrfs send xxx | sudo btrfs receive xxx
sudo btrfs subvolume snapshot xxx xxx
sudo btrfs subvolume delete snapshot-ro
```
之后按照安装arch的方式进行处理
```bash
sudo mount -o subvol=@ /dev/sdX /mnt
genfstab
```
- 个人经历

作者误认为社团的机器被捐赠者收回,于是将ssd中的所有子卷全部导出,并且使用dd毁灭了原来的系统
转移子卷到的机器和社团机器通过内网路由器连接,可以ssh访问

作者处理的办法
### attempt1
```bash
sudo -v #避免sudo密码问题
sudo btrfs send @subvol | ssh user@remote_ip "sudo btrfs receive /mnt" #machine1
sudo btrfs restore /mnt/@subvol /dev/sdX #machine2
```
此时报错不能在同一个设备上restore
### attempt2
```bash
mkdir /tmp/backup 
sudo mount /dev/sdX1 /tmp/backup
sudo btrfs send /mnt/@subvol | sudo btrfs receive /tmp/backup
sudo umount /mnt
sudo btrfs restore /tmp/backup/@subvol /dev/sdX
```
此时失败,大概是数据传输问题
### attempt3
```bash
sudo btrfs send /tmp/backup/@subvol |sudo btrfs receive /mnt
sudo btrfs subvolume snapshot /mnt/@subvol /mnt/@subvol
```
最后一次成功
可以理解subvol为一种特殊文件夹,可以通过btrfs send和receive来传输
```

