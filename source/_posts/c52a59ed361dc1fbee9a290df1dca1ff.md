---
title: 编译原理学习一，去除代码中的注释
date: 2020/03/28 16:24:14
abbrlink: @@abbrlink
categories:
- 编程
tags:
- 编译原理
- 编译
- 去除
- 注释
- 代码
- 原理
- 学习
---
## 前言
开始学习编译原理了耶～
关于编译原理的所有练习，按照老规矩，还是用我最喜欢的C#语言来实现，运行在.NetCore平台上～
关于这个系列的所有代码已经上传到github了，项目主页：
>[https://github.com/Deali-Axy/CompilerConstructionLearning](https://github.com/Deali-Axy/CompilerConstructionLearning)


## 本次题目
>对C或C++等高级程序设计语言编写的源程序中的//注释和/*…*/注释进行删除，保留删除后的源程序。要求以文件形式进行保存。

## 思路分析
>程序主要功能就是消除已经编写好的源程序中的注释。在源程序中注释有两种形式，一种是单行注释，用“//”表示，另一种是多行注释，用“/*…*/”表示。针对这两种形式，程序中用了if..else..语句加以判断，并做出相应的处理。在这里还有可能出现另一种情况，上述两种注释符号可能出现在引号中，出现在引号中的注释符号并没有注释功能，因此在引号中出现的注释符号不应该被消除。所以，这次编写的程序将要分三种情况分析。

#### 第一种情况，单行注释：
```
if (ch != temp)
{
    // 这里就是单行注释
    ofile.put(ch);
    ch = ifile.get();
}
```
或者
```
if (ch != temp)
{
    /* 这里就是单行注释 */
    ofile.put(ch);
    ch = ifile.get();
}
```

#### 第二种情况，块注释：
```
if (ifile.fail() || ofile.fail())
{
    cerr << "open file fail\n";
    return EXIT_FAILURE;
    /*返回值EXIT_FAILURE（在cstdlib库中定义）,用于向操作系统报*
    告打开文件失败*/
}
```

#### 第三种情况，行后注释：
```
ifile.close(); // 关闭文件
ofile.close();
cout << "/////*////ret/rtr////";
system("pause");
return 0;
```

#### 还有一个关键的注意点
可以看到这一行
```
cout << "/////*////ret/rtr////";
```
这个字符串用双引号包起来的代码中有很多斜杠，所以要避免将这些斜杠识别为注释。
这里我用的方法是在处理注释前先把包含注释符号的字符串替换掉，等注释删除之后，再添加回去。

## 实现代码
注释写得很详细啦，配合上面的思路分析，我就不再继续分析代码了～
```
var sReader = new StreamReader(filePath);
var newSource = "";
var inBlock = false;
var replaceFlag = false;
var tempLine = ""; // 用于保存被替换的特殊行代码
while (!sReader.EndOfStream)
{
    var line = sReader.ReadLine();
    if (line.Length == 0) continue; // 去除空行

    var quotationPattern = "^(.*?)\".*//.*\"";
    var quotationResult = Regex.Match(line, quotationPattern);
    if (quotationResult.Success)
    {
        System.Console.WriteLine("替换特殊代码，双引号中包裹注释斜杠");
        tempLine = quotationResult.Groups[0].Value;
        replaceFlag = true;
        line = Regex.Replace(line, quotationPattern, REPLACEMENT);
    }

    // 单行注释
    if (line.Trim().StartsWith(@"//"))
        continue;
    if (line.Trim().StartsWith(@"/*") && line.EndsWith(@"*/"))
        continue;

    // 注释块
    if (Regex.Match(line.Trim(), @"^/\*").Success)
        inBlock = true;
    if (Regex.Match(line.Trim(), @"\*/$").Success)
    {
        inBlock = false;
        continue;
    }

    // 行后注释
    // 使用非贪婪模式(.+?)匹配第一个//
    var pattern = @"^(.*?)//(.*)";
    // var pattern = @"[^(.*?)//(.*)]|[^(.*?)/\*(.*)\*/]";
    var result = Regex.Match(line, pattern);
    if (result.Success)
    {
        System.Console.WriteLine("发现行后注释：{0}", result.Groups[2]);
        line = result.Groups[1].Value;
    }

    // 还原被替换的代码
    if (replaceFlag)
    {
        System.Console.WriteLine("还原特殊代码");
        line = line.Replace(REPLACEMENT, tempLine);
        replaceFlag = false;
    }

    if (inBlock) continue;
    newSource += line + Environment.NewLine;
}

var outputPath = "output/exp1.src";
System.Console.WriteLine("去除注释完成，创建新文件。");
using (var sWriter = new StreamWriter(outputPath))
{
    sWriter.Write(newSource);
}
System.Console.WriteLine("操作完成！文件路径：{0}", outputPath);
```

## 结果测试
#### 源文件
```
#include <iostream>
#include <fstream>
#include <iomanip>
#include <cstdlib>
using namespace std;

int main()
{
    cout << '/';
    ifstream ifile; //建立文件流对象
    ofstream ofile;
    ifile.open("f:\\上机实验题\\C++\\ConsoleApplication2\\ConsoleApplication2\\源.cpp"); //打开F盘根目录下的fileIn.txt文件
    ofile.open("f:\\上机实验题\\C++\\ConsoleApplication2\\ConsoleApplication2\\源.obj");
    if (ifile.fail() || ofile.fail())
    { //测试打开操作是否成功
        cerr << "open file fail\n";
        return EXIT_FAILURE;
        /*返回值EXIT_FAILURE（在cstdlib库中定义）,用于向操作系统报*
    告打开文件失败*/
    }
    char ch;
    ch = ifile.get(); //进行读写操作
    while (!ifile.eof())
    {
        if (ch == 34)
        {                   //双引号中若出现“//”，双引号中的字符不消除
            char temp = ch; //第一个双引号
            ofile.put(ch);
            ch = ifile.get();
            while (!ifile.eof())
            {
                if (ch != temp)
                { //寻找下一个双引号
                    ofile.put(ch);
                    ch = ifile.get();
                }
                else
                {
                    ofile.put(ch);
                    break;
                }
            }
            ch = ifile.get();
            continue; //双引号情况结束，重新新一轮判断
        }
        if (ch == 47)
        { //出现第一个斜杠
            char temp2 = ch;
            ch = ifile.get();
            if (ch == 47)
            { //单行注释情况
                ch = ifile.get();
                while (!(ch == '\n'))
                    ch = ifile.get();
            }
            else if (ch == '*')
            { //多行注释情况
                while (1)
                {
                    ch = ifile.get();
                    while (!(ch == '*'))
                        ch = ifile.get();
                    ch = ifile.get();
                    if (ch == 47)
                        break;
                }
                ch = ifile.get();
            }
            else
            {
                ofile.put(temp2); //temp2保存第一个斜杠，当上述两种情况都没有时，将此斜杠输出
            }
            //ch = ifile.get();
        }
        //cout << ch << endl;
        ofile.put(ch);    //将字符写入文件流对象中
        ch = ifile.get(); //从输入文件对象流中读取一个字符
    }
    ifile.close(); //关闭文件
    ofile.close();
    cout << "/////*////ret/rtr////";
    system("pause");
    return 0;
}
```

#### 处理后的结果
```
#include <iostream>
#include <fstream>
#include <iomanip>
#include <cstdlib>
using namespace std;
int main()
{
    cout << '/';
    ifstream ifile; 
    ofstream ofile;
    ifile.open("f:\\上机实验题\\C++\\ConsoleApplication2\\ConsoleApplication2\\源.cpp"); 
    ofile.open("f:\\上机实验题\\C++\\ConsoleApplication2\\ConsoleApplication2\\源.obj");
    if (ifile.fail() || ofile.fail())
    { 
        cerr << "open file fail\n";
        return EXIT_FAILURE;
    }
    char ch;
    ch = ifile.get(); 
    while (!ifile.eof())
    {
        if (ch == 34)
        {                   
            char temp = ch; 
            ofile.put(ch);
            ch = ifile.get();
            while (!ifile.eof())
            {
                if (ch != temp)
                { 
                    ofile.put(ch);
                    ch = ifile.get();
                }
                else
                {
                    ofile.put(ch);
                    break;
                }
            }
            ch = ifile.get();
            continue; 
        }
        if (ch == 47)
        { 
            char temp2 = ch;
            ch = ifile.get();
            if (ch == 47)
            { 
                ch = ifile.get();
                while (!(ch == '\n'))
                    ch = ifile.get();
            }
            else if (ch == '*')
            { 
                while (1)
                {
                    ch = ifile.get();
                    while (!(ch == '*'))
                        ch = ifile.get();
                    ch = ifile.get();
                    if (ch == 47)
                        break;
                }
                ch = ifile.get();
            }
            else
            {
                ofile.put(temp2); 
            }
        }
        ofile.put(ch);    
        ch = ifile.get(); 
    }
    ifile.close(); 
    ofile.close();
    cout << "/////*////ret/rtr////";
    system("pause");
    return 0;
}
```

## 完整代码
[https://github.com/Deali-Axy/CompilerConstructionLearning/blob/master/code/exp/exp1/Exp1.cs](https://github.com/Deali-Axy/CompilerConstructionLearning/blob/master/code/exp/exp1/Exp1.cs)

参考资料：
- 正则表达式元字符：[https://www.runoob.com/regexp/regexp-metachar.html](https://www.runoob.com/regexp/regexp-metachar.html)
- JavaScript去除注释：[https://segmentfault.com/a/1190000015611632](https://segmentfault.com/a/1190000015611632)


## 欢迎交流
交流问题请在微信公众号后台留言，每一条信息我都会回复哈~
- 微信公众号：画星星高手
- 打代码直播间：[https://live.bilibili.com/11883038](https://live.bilibili.com/11883038)
- 知乎：[https://www.zhihu.com/people/dealiaxy](https://www.zhihu.com/people/dealiaxy)
- 专栏：[https://zhuanlan.zhihu.com/deali](https://zhuanlan.zhihu.com/deali)
- 简书：[https://www.jianshu.com/u/965b95853b9f](https://www.jianshu.com/u/965b95853b9f)
