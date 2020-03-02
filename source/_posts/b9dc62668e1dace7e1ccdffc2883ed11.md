---
title: Apache-Commons-IO-最佳实践
date: 2020/03/02 10:27:52
abbrlink: ae75f61093b6e7b4
categories:
- Java
tags:
- 运维
- 实践
---
本文列举了Java输入输出部分（IO area）的大量“最佳实践”（best practices）。

## java.io.File
通常你需要去处理文件或文件名时，有许多会出错的情况：
- 一个类可在Unix运行但不能在Windows运行，反之亦然。
- 由于双路径分隔符（`path separator`）或缺失路径分隔符（`path separator`）导致文件名无效。
- (在Windows上的)符合通用命名标准（`UNC`）的文件名无法在开发人员开发的（`home-grown`）工具方法中运行
- 等等

上述是字符串（`String`）文件名不运行的准确原因。使用java.io.File类代为处理上述情况将更合适。因此我们的最佳实践（`best practice`）推荐使用java.io.File类代替字符串（`String`）处理文件名以避免依赖具体的平台（`platform`）。

`Commons-io 1.1` 包括一个专用于处理文件名的类-文件名工具类（`FilenameUtils`）。它处理了许多上述文件名的问题，但是我们仍然推荐，你尽可能地使用java.io.File对象来处理。

让我们看一个例子。

```java
public static String getExtension(String filename) {
   int index = filename.lastIndexOf('.');
   if (index == -1) {
     return "";
   } else {
     return filename.substring(index + 1);
   }
 }
```
简单吗？对，但如果开发人员传入（`pass`）一个全路径（`full path`）字符串而不仅仅是一个文件名会发生什么？考虑以下情况，传入完全符合要求的路径：`"C:\Temp\documentation.new\README"`。上面定义的方法（`method`）可能返回（`return`） `"new\README"`-完全不是你想要的结果。

请使用 `java.io.File` 代替字符串（`String`）处理文件名。该类所提供的功能是经过良好测试的（`well tested`）。在文件工具类（`FileUtils`）中你将找到关于 `java.io.File` 的其他有用的工具方法（`function`）。

例如： 

```java
String tmpdir = "/var/tmp";
 String tmpfile = tmpdir System.getProperty("file.separator") "test.tmp";
 InputStream in = new java.io.FileInputStream(tmpfile);
```
改为： 
```java
File tmpdir = new File("/var/tmp");
 File tmpfile = new File(tmpdir, "test.tmp");
 InputStream in = new java.io.FileInputStream(tmpfile);
```

## 输入输出流缓冲（`Buffering streams`）

输入输出性能非常依赖缓冲策略（`buffering strategy`）。一般而言，缓冲512或1024大小字节来读取数据块（`packets`）是非常迅速的，因为这个大小与数据块（`packets`）在硬盘（`harddisk`）文件系统（`file system`）或文件缓存（`file system cache`）中的大小非常匹配。而且当你仅仅读取一点字节时，时间性能就会显著下降（`drop`）。

当读取或写入输入输出流时，特别是当访问文件的时候，确保你正确地缓冲输入输出流。只需要使用缓冲字节输入流（`BufferedInputStream`）修饰（`decorate`）你的文件字节输入流（`FileInputStream`）即可
>译注：Java中的输入输出流采用了`修饰模式`，或叫`装饰模式`：

```java
InputStream in = new java.io.FileInputStream(myfile);
 try {
   in = new java.io.BufferedInputStream(in);
   
   in.read(.....
 } finally {
   IOUtils.closeQuietly(in);
 }
```

**注意，你不能缓存一个已经缓冲了的输入输出流。**
一些组件，例如XML解析器（`XML parser`）会使用他们自己的字节输入流（`InputStream`）缓存修饰（`decorate`），你传递一个缓存输入输出流到XML解析器（`XML parser`）也没多大影响但会拖慢（`slow down`）你的代码性能。如果你用我们的复制工具类（`CopyUtils`）或输入输出工具类（`IOUtils`），你就不需要额外缓冲输入输出流，将这些代码当作已经在复制过程中缓存了即可。检查`Javadoc`可以获取更多信息。其他如在你写入一个字节数组输出流（`ByteArrayOutputStream`）到内存（`memory`）的情况，是不需要缓冲的。

>*本文由侯骏雄授权发表。*
>翻译：侯骏雄（QQ：2442278700）
>博客：http://blog.csdn.net/houjx114

## About
{% asset_img 8869373-901590e019f6f85b.png %}
---------------
了解更多有趣的操作请关注我的微信公众号：DealiAxy
每一篇文章都在我的博客有收录：[blog.deali.cn](http://blog.deali.cn)
