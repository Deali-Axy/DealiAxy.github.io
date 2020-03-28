---
title: 给你的Deepin系统换主题～
date: 2020/03/28 20:59:32
abbrlink: b151c6b95d825a92
categories:
- Linux
tags:
- 主题
- 系统
---
我现在的桌面：
{% asset_img 8869373-e30de2ae83767563.png %}

因为美国封锁然后华为电脑用上Deepin系统的事情，Deepin系统最近名声大噪，其实Deepin本来的界面就很好看，号称打造最适合国人的系统不是盖的。

不过Deepin有一个坏处，就是默认主题的标题栏太大了…… 太占空间。所以我们要换主题折腾一下。

Deepin的桌面是Qt写的，所以首先要换KDE主题，然后再安排一下GTK3的主题，在Deepin系统设置里面换的主题就是GTK的，只对Gtk程序有效。

首先下载这个主题：[pingguo-aurorae](https://gitlab.com/1314/pingguo-aurorae)

然后解压到`主目录 / .local / share / aurorae / themes/`目录下面，缺了哪个目录就自己创建啦。

然后还需要安装KDE设置这个程序，因为我们要设置Qt程序的主题呀，Deepin系统设置是没办法换Qt程序主题的，如下：

```bash
sudo apt install systemsettings
```

之后打开KDE设置，选 **应用程序风格** ，选一个你喜欢的配色即可

设置之后的效果：

{% asset_img 8869373-1b7ba88392f1b132.png %}
