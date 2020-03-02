---
title: 内存不够用？在Linux上使用swapfile配置交换空间
date: 2020/03/02 10:27:53
abbrlink: dbd5b4751612ccae
categories:
- Linux
tags:
- Linux
- 内存
- 交换
- 不够
- 配置
- 空间
- 使用
---
## 前言
几个月前我还兴高采烈写了篇文章说给笔记本升级了内存配置，从乞丐版的4G一跃成为16G高富帅，然而，这几个月来我备受电脑死机的折磨，我现在有理由怀疑我可能买了假的内存条，于是没有办法，只能换回原厂的4G内存条，~~（然后准备赚部台式机）~~，**问题不大。**
换上4G内存条之后效果显著，只开了一个谷歌浏览器，内存占用就已经到了80%了，JB家的IDE是决计不敢再用了，可是这样还是难以满足系统对内存空间渴求，这时候就只能配置交换空间了。*（也就是虚拟内存）*

## Swap概念
Swap分区（也称交换分区）是硬盘上的一个区域，被指定为操作系统可以临时存储数据的地方，这些数据不能再保存在RAM中。 基本上，这使您能够增加服务器在工作“内存”中保留的信息量，但有一些注意事项，主要是当RAM中没有足够的空间容纳正在使用的应用程序数据时，将使用硬盘驱动器上的交换空间。
写入磁盘的信息将比保存在RAM中的信息慢得多，但是操作系统更愿意将应用程序数据保存在内存中，并使用交换旧数据。 总的来说，当系统的RAM耗尽时，将交换空间作为回落空间可能是一个很好的安全网，可防止非SSD存储系统出现内存不足的情况。

## 检查系统信息
在开始之前，我们可以检查系统是否已经有一些可用的交换空间，可能有多个交换文件或交换分区，但通常应该是足够的。我们可以通过如下的命令来查看系统是否有交换分区：
```bash
sudo swapon --show
```
如果没有任何结果或者没有任何显示，说明系统当前没有可用的交换空间。free命令用来查看空闲的内存空间，其中包括交换分区的空间。
```bash
free -h
```

## 检查硬盘驱动器分区上的可用空间
为swap分配空间的最常见方式是使用专门用于具体某个任务的单独分， 但是，改变分区方案并不是一定可行的，我们只是可以轻松地创建驻留在现有分区上的交换文件。

在开始之前，我们应该通过输入以下命令来检查当前磁盘的使用情况：
```bash
df -h
```

## 创建swap文件
由于这个系统我已经用了很久了，并且磁盘上并没有空闲的分区可以用，所以我选择手动创建一个swapfile来充当交换空间。

据说交换空间的大小一般是内存的两倍，我现在只有4g的内存空间，于是当然要创建一个8G大小的交换空间了。
使用以下命令创建swapfile
```bash
sudo fallocate -l 8G /swapfile
```
经过测试，OpenSuSE系统要使用以下命令才能成功创建swapfile
```bash
sudo dd if=/dev/zero of=/swapfile count=4096 bs=1MiB
```

使用以下命令查看是否正确创建。
```bash
ls -lh /swapfile
```
结果应该类似下面这样：
```bash
-rw-r--r-- 1 root root 8.0G Apr 26 17:04 /swapfile
```

## 修改swapfile权限
```bash
sudo chmod 600 /swapfile
```
查看效果
```bash
ls -lh /swapfile
```

结果应该类似下面这样：
```bash
-rw------- 1 root root 8.0G Apr 26 17:04 /swapfile
```

## 激活交换空间
```bash
sudo mkswap /swapfile
sudo swapon /swapfile
```

之后使用以下命令查看使用成功开启交换空间：
```bash
sudo swapon --show
```

结果类似下面这样：
```bash
NAME      TYPE SIZE USED PRIO
/swapfile file   8G   0B   -1
```

## 添加到fstab
这样每次开机系统就会自动吧swapfile挂载为交换空间。
首先请自行备份`fstab`文件。
然后把以下配置添加到`fstab`文件末尾。
```bash
/swapfile none swap sw 0 0
```

或者直接使用以下命令：
```bash
sudo cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab 
```


## About
{% asset_img 8869373-901590e019f6f85b.png %}

---------------
了解更多有趣的操作请关注我的微信公众号：DealiAxy
每一篇文章都在我的博客有收录：[blog.deali.cn](http://blog.deali.cn)
