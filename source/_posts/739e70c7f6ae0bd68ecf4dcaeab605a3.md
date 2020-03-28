---
title: 修改_Inotify-Watches-Limit_提高idea性能
date: 2020/03/28 16:24:14
abbrlink: c15c9e90c2ee3834
categories:
- 杂七杂八
tags:
- 修改
- 性能
- 提高
---
## 官网说明
>For an intelligent IDE, it is essential to be in the know about any external changes in files it is working with - e.g. changes made by VCS, or build tools, or code generators etc. For that reason, IntelliJ platform spins background process to monitor such changes. The method it uses is platform-specific, and on Linux, it is the [Inotify](http://en.wikipedia.org/wiki/Inotify) facility.
>
>Inotify requires a "watch handle" to be set for each directory in the project. Unfortunately, the default limit of watch handles may not be enough for reasonably sized projects, and reaching the limit will force IntelliJ platform to fall back to recursive scans of directory trees.
>
>To prevent this situation it is recommended to increase the watches limit (to, say, 512K)

大意就是说idea运行的时候有一个后台进程在不断的扫描项目文件夹里面是否有文件变动，这个技术在Linux系统上是使用 `Inotify` 特性实现的，但是Linux系统有一个 `watch handle limit`，简单说就是监视大小限制， 一般来说这个大小限制都比我们的项目所需要的小，所以idea就要经常主动去扫描项目目录，而不能利用系统特性，导致变卡。

解决这个问题的办法就是我们修改一下系统配置，提高这个限制的大小。

## 解决方法
>Add the following line to either /etc/sysctl.conf file or a new *.conf file (e.g. idea.conf) under /etc/sysctl.d/ directory:
>```bash
>fs.inotify.max_user_watches = 524288
>```
>Then run this command to apply the change:
>```bash
>sudo sysctl -p --system
>```


## 官网说明
https://confluence.jetbrains.com/display/IDEADEV/Inotify+Watches+Limit

## About
{% asset_img 8869373-901590e019f6f85b.png %}

---------------
Learn more on my WeChat Official Account：DealiAxy
Every post was in my blog：[blog.deali.cn](http://blog.deali.cn)
