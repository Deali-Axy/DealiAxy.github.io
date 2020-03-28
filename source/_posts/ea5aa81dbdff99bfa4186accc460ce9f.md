---
title: 操作系统学习三：进程调度与死锁-以及银行家算法避免死锁--NetCore实现
date: 2020/03/28 20:59:33
abbrlink: a1325c3c41b64eb2
categories:
- 编程
tags:
- 操作系统
- 死锁
- 调度
- 银行家
- 进程
- 避免
- 学习
- 实现
---
## 前言

这是操作系统学习的第三篇啦，关于进程调度有很多内容，操作系统在调度进程的时候最容易遇到的问题就是死锁了，**银行家算法**是一个典型的避免死锁算法。

## 死锁的概念

先来了解一下死锁的基本概念：**一组竞争系统资源或相互通信的进程相互的“永久”阻塞。若无外力作用，这组进程将永远不能继续执行。**

看下面两幅图片，左边是可能产生死锁的状态，四辆汽车（进程）要竞争同一个资源（通过路口），如果系统调度不当，就会陷入死锁状态，如右图*（每辆车占据一个车道（资源），所需车道（资源）被另一辆车占据。）*。

{% asset_img 8869373-1404068f0d4e4cce.png %}


## 产生死锁的原因

产生死锁的原因主要有两点：

- 资源数  <  要求该种资源的进程数（资源竞争）

- 进程的推进顺序不恰当

  如下图：A、B分别代表某种资源，假设都只有一个，进程P先占用了资源A，接着进程Q占用了资源B，后面进程P再想要资源B就拿不到，进程Q想要资源A也拿不到，系统就陷入死锁状态了。

{% asset_img 8869373-0ea9289dfe99ad54.png %}

再看看下面这个图，同样说明了进程推进顺序不恰当导致的结果：

-  (1)、(2) 、(4) 、(5)正常运行
-  (3) 、(6)发生死锁

{% asset_img 8869373-d70abb059cda7c8a.png %}

修改了进程的推进顺序之后，不会陷入死锁，可以正常运行，如下图：

{% asset_img 8869373-66f8da011ee24382.png %}

## 产生死锁的条件

1. **互斥条件**：进程所竞争的资源必须被互斥使用。

*互斥是资源的固有属性，不可禁止。*

2. **请求保持条件**：当前已拥有资源的进程仍能申请新的资源，当被阻塞时，对已获得的资源保持不放。

3. **不剥夺条件**：进程已获得的资源只能在使用完时自行释放，而不能被抢占。

4. **环路条件**：存在一个至少包含两个进程的循环等待链，链中的每个进程都正在等待下一个进程占有的资源。

> - 前面三个条件是必要条件，**环路条件**是必要条件。
>
> - 第一个**互斥**是资源的固有属性，没办法禁止的。
> - 只要破坏后面3个条件中的任意条件，就可以预防或者避免死锁。

## 银行家算法

这是仿照银行发放贷款时采取的控制方式而设计的一种死锁避免算法。其特点是所有客户的信用额度可以超过银行的全部资本。

银行家算法的目标：**所有客户的信用额度之和可以超过银行的全部资本，这就是杠杆。 **

## 主要思想

银行家算法的主要思想如下：

1. 当一个用户对资金的最大的需求量（即信用额度）不超过银行家现有的资金时就可以接纳该用户。
2. 用户可以分期贷款，但贷款的总数不能超过最大需求量。 
3. 当银行家现有的资金不能满足用户的尚需贷款时，对用户的贷款可推迟支付，但总能使用户在有限的时间里得到贷款。
4. 当用户得到所需的全部资金后，一定能在有限的时间里归还所有资金。 （客户信用良好，都能安期还款）

## 举例

> 假定某银行的全部流动资金为10000元。客户A申请4000元的信用额度，客户B申请6000元的信用额度，客户C申请10000元的信用额度，由于都没有超过银行的流动资金，予以批准。客户D申请12000元的信用额度，银行拒绝。假定A、B、C来银行提出下列贷款申请：
>
> - A要求贷款2000元
> - B要贷款4000元
> - C要贷款3000元

银行策略如下：

1. **只要银行还有钱就发放贷款**

   如果采用这种策略，发放贷款后银行剩下1000元，将无法满足任何一个客户的信用额度，从而造成银行呆账。

2. **需要考虑发放贷款后银行面临的风险（如果银行无法满足所有客户的信用额度，将导致无法收回贷款）**

   如果采用这种策略，先发放A和B的贷款后银行剩下4000元，推迟支付C的贷款，剩下的4000元可以满足A或B客户的信用额度。

## 进一步理解

我们把操作系统看作是银行家，操作系统管理的资源相当于是银行家管理的资金，当进程提出资源请求时，系统检查：

- 可利用资源数`Available`：银行流动资金
- 进程最大资源需求数`Max`：信用额度
- 已分配给进程的资源数`Allocation`：贷款
- 进程还将需要的资源数`Need`：信用额度-贷款

**银行家算法运行的流程如下：**

- 进行资源预分配
- 实施安全检测
  - 安全：真正资源分配
  - 不安全：回到预分配前状态

{% asset_img 8869373-3a543e2a1951d39d.png %}

## 算法实现

下面是银行家算法整个实现过程的流程图：

{% asset_img 8869373-b59a343c8c2f357b.png %}

**实现相关数据结构的说明：**

1． 可利用资源向量Available ，它是一个含有m个元素的数组，其中的每一个元素代表一类可利用的资源的数目，其初始值是系统中所配置的该类全部可用资源数目。其数值随该类资源的分配和回收而动态地改变。如果`Available[j]=k`，表示系统中现有j类资源k个。

2． 最大需求矩阵Max，这是一个n×m的矩阵，它定义了系统中n个进程中的每一个进程对m类资源的最大需求。如果`Max[i][j]=k`，表示进程i需要j类资源的最大数目为k。

3． 分配矩阵Allocation，这是一个`n×m`的矩阵，它定义了系统中的每类资源当前分配到每一个进程的资源数。如果`Allocation[i][j]=k`，表示进程i当前已经分到j类资源的数目为k个。`Allocation[i]`表示进程i的分配向量。

4． 需求矩阵Need，这是一个n×m的矩阵，用以表示每个进程还需要的各类资源的数目。如果`Need[i][j]=k`，表示进程i还需要j类资源k个，才能完成其任务。`Need[i]`表示进程i的需求向量。

**上述三个矩阵间存在关系：`Need[i][j]=Max[i][j]-Allocation[i][j]；`**

**算法过程说明**

Request是进程i的请求向量。`Request[j]=k`表示进程i请求分配j类资源k个。当进程i发出资源请求后，系统按下述步骤进行检查：

1． 如果`Request ≤Need[i]`，则转向步骤2；否则，认为出错，因为它所请求的资源数已超过它当前的最大需求量。

2． 如果`Request ≤Available`，则转向步骤3；否则，表示系统中尚无足够的资源满足进程i的申请，进程i必须等待。

3． 系统试探性地把资源分配给进程i，并修改下面数据结构中的数值：

```c++
Available = Available - Request 

Allocation[i]= Allocation[i]+ Request

Need[i]= Need[i] - Request 
```

4． 系统执行安全性算法，检查此次资源分配后，系统是否处于安全状态。如果安全才正式将资源分配给进程i，以完成本次分配；否则，将试探分配作废，恢复原来的资源分配状态，让进程i等待。

## 安全性测试算法

1． 设置两个向量。

`Work`：它表示系统可提供给进程继续运行的各类资源数目，它包含m个元素，开始执行安全性算法时，`Work = Available`

`Finish`：它表示系统是否有足够的资源分配给进程，使之运行完成，开始Finish[i]=false；当有足够资源分配给进程i时，令`Finish[i]=true`

2． 从进程集合中找到一个能满足下述条件的进程。

- ` Finish[i]= = false`
- `Need[i]≤work`

如找到则执行步骤3；否则，执行步骤4；

3． 当进程i获得资源后，可顺利执行直到完成，并释放出分配给它的资源，故应执行

- `Work = work + Allocation[i]`
- `Finish[i]=true`
- `转向步骤2`

4． 若所有进程的`Finish[i]`都为`true`，则表示系统处于安全状态；否则，系统处于不安全状态。

## 实现代码

好啦，关于概念和原理都讲清楚了，接下来贴上代码。

代码里已经写了详细的注释了，虽然是面向过程的写法，但是结构也很清晰。

**首先定义进程类**

```c#
namespace OperatingSystemExperiment.Exp3 {
    public class ProcessExp3 {
        public int Id;

        /// <summary>
        /// 进程最大（各类）资源需求数：信用额度
        /// </summary>
        public int[] Max;

        /// <summary>
        /// 已分配给进程的资源：贷款
        /// </summary>
        public int[] Allocation;

        /// <summary>
        /// 进程还需要的资源：信用额度 - 贷款
        /// </summary>
        public int[] Need;

        public ProcessExp3(int id) => this.Id = id;


        public void EvaluateNeedResource()
        {
            Need = new int[Max.Length];
            for (var i = 0; i < Max.Length; i++) {
                Need[i] = Max[i] - Allocation[i];
            }
        }
    }
}
```

**算法实现代码：**

```c#
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Linq;

namespace OperatingSystemExperiment.Exp3 {
    public class Main {
        private int _resourceClassesCount = 0;
        private int _processCount = 0;

        private List<ProcessExp3> _processes = new List<ProcessExp3>();

        /// <summary>
        /// 系统全部可分配资源：银行流动资金
        /// </summary>
        private List<int> _resource = new List<int>();

        /// <summary>
        /// 系统剩余可分配资源
        /// </summary>
        private List<int> _available = new List<int>();

        private Main()
        {
            // 获取当前系统资源分配状态
            LoadAllAvailableResource();
            LoadProcessMaxResource();

            var continueFlag = true;
            while (continueFlag) {
                mainLoop:
                // 获取各进程已分配资源
                LoadAllocationResource();
                // 评估每个进程还需要的资源
                EvaluateNeedResource();

                // 打印各数据结构当前值
                PrintStatus();
                Console.Write("请输入要操作的进程号：");
                if (!int.TryParse(Console.ReadLine(), out var procId)) {
                    Console.WriteLine("\n请输入数字！");
                    if (QueryExit()) Environment.Exit(0);
                    else goto mainLoop;
                }

                if (procId < 0 || procId >= _processes.Count) {
                    Console.WriteLine("\n不存在进程号为 {0} 的进程！", procId);
                    if (QueryExit()) Environment.Exit(0);
                    else goto mainLoop;
                }

                Console.Write("请输入资源请求向量：");
                var request = Console.ReadLine();
                var requestVector = Array.ConvertAll(request?.Split(' '), int.Parse);

                // 检查资源请求是否合理
                var proc = _processes[procId];
                Console.WriteLine("银行家算法检验中...");
                for (var i = 0; i < requestVector.Length; i++) {
                    if (requestVector[i] > proc.Need[i]) {
                        Console.WriteLine("分配失败！资源类型 {0}，请求数量 {1}，超过进程所需数量 {2}",
                            i, requestVector[i], proc.Need[i]);
                        if (QueryExit()) Environment.Exit(0);
                        else goto mainLoop;
                    }

                    if (requestVector[i] > proc.Max[i]) {
                        Console.WriteLine("分配失败！资源类型 {0}，请求数量 {1}，超过进程最大资源数量 {2}",
                            i, requestVector[i], proc.Need[i]);
                        if (QueryExit()) Environment.Exit(0);
                        else goto mainLoop;
                    }
                }

                // 保存当前状态
                var tempAvailable = new List<int>(_available);
                var tempAllocation = (int[]) proc.Allocation.Clone();
                var tempNeed = (int[]) proc.Need.Clone();
                // 资源预分配
                for (var i = 0; i < _resourceClassesCount; i++) {
                    proc.Allocation[i] += requestVector[i];
                    proc.Need[i] -= requestVector[i];
                }

                if (SecurityEvaluate()) {
                    Console.WriteLine("正在为进程 {0} 分配资源", proc.Id);
                    // 写入资源分配文件
                    var writer = new StreamWriter(Path.Combine(
                        Environment.CurrentDirectory, "Exp3", "input", "Allocation_list.txt"), false);
                    foreach (var p in _processes) {
                        // 使用LinQ语句，构造输出行
                        var line = p.Allocation.Aggregate("", (current, allocation) => current + (allocation + " "));

                        writer.WriteLine(line.Trim());
                    }

                    Console.WriteLine("已经保存新的分配状态！");
                    writer.Close();
                }
                else {
                    Console.WriteLine("恢复试分配前的状态。");
                    // 恢复预分配之前的状态
                    _available = tempAvailable;
                    proc.Allocation = tempAllocation;
                    proc.Need = tempNeed;
                }

                continueFlag = !QueryExit();
            }
        }

        /// <summary>
        /// 安全性算法
        /// </summary>
        private bool SecurityEvaluate()
        {
            var work = _available;
            var finish = new bool[_processCount];
            var found = false; // 判断标志
            var finishCount = 0; // 满足条件的进程数目
            var safeQueue = new List<int>();

            while (finishCount < _processCount) {
                for (var procId = 0; procId < _processCount; procId++) {
                    var proc = _processes[procId];

                    if (!finish[procId]) {
                        for (var resId = 0; resId < work.Count; resId++) {
                            Debug.WriteLine("安全性测试，procId={0} resId=(1)", procId, resId);
                            if (proc.Need[resId] > work[resId]) {
                                Debug.WriteLine("NotFound! procId={0} resId={1}", procId, resId);
                                found = false;
                            }
                            else {
                                Debug.WriteLine("Found! procId={0} resId={1}", procId, resId);
                                found = true;
                            }
                        }
                    }

                    if (found) {
                        // 模拟释放资源
                        for (var t = 0; t < work.Count; t++) {
                            work[t] += proc.Allocation[t];
                        }

                        // 保存进程号
                        finish[procId] = true;
                        finishCount++;
                        // 加入安全队列
                        safeQueue.Add(procId);
                        // 重置状态
                        found = false;
                    }
                }
            }

            Console.WriteLine("安全序列如下：");
            // 打印安全序列
            var output = "";
            foreach (var procId in safeQueue) {
                output += "P" + procId + ",";
            }

            Console.WriteLine(output.TrimEnd(','));


            if (finish.Any(flag => !flag)) {
                Console.WriteLine("未通过安全性测试！");
                return false;
            }

            Console.WriteLine("已经通过安全性测试！");

            return true;
        }

        private bool QueryExit()
        {
            Console.Write("是否退出(y/n)？");
            var option = Console.ReadLine()?.ToLower();
            return option == "y";
        }

        /// <summary>
        /// 评估进程所需资源
        /// </summary>
        private void EvaluateNeedResource()
        {
            foreach (var p in _processes) {
                p.EvaluateNeedResource();
                // 计算系统还剩下多少资源
                for (var i = 0; i < _resourceClassesCount; i++) {
                    _available[i] -= p.Allocation[i];
                }
            }
        }

        /// <summary>
        /// 打印出当前状态
        /// </summary>
        private void PrintStatus()
        {
            Console.WriteLine("-------------------------银行家算法-------------------------");
            Console.WriteLine("系统进程数量：{0}；资源种类数量：{1}", _processCount, _resourceClassesCount);
            Console.WriteLine("可用资源向量 Available：");
            foreach (var i in _resource) {
                Console.Write("{0} ", i);
            }

            Console.WriteLine("\n最大需求矩阵 Max：");
            foreach (var p in _processes) {
                foreach (var t in p.Max) {
                    Console.Write("{0} ", t);
                }

                Console.WriteLine("");
            }

            Console.WriteLine("已分配矩阵 Allocation：");
            foreach (var p in _processes) {
                foreach (var t in p.Allocation) {
                    Console.Write("{0} ", t);
                }

                Console.WriteLine("");
            }

            Console.WriteLine("需求矩阵 Need：");
            foreach (var p in _processes) {
                foreach (var t in p.Need) {
                    Console.Write("{0} ", t);
                }

                Console.WriteLine("");
            }
        }

        /// <summary>
        /// 加载系统所有可用的资源数量
        /// </summary>
        private void LoadAllAvailableResource()
        {
            var reader =
                new StreamReader(Path.Combine(Environment.CurrentDirectory, "Exp3", "input", "Available_list.txt"));
            var line = reader.ReadLine();
            // 获取各类型资源数量
            _resourceClassesCount = int.Parse(line?.Trim());
            line = reader.ReadLine();
            _resource.AddRange(Array.ConvertAll(line?.Split(' '), int.Parse));
            _available.AddRange(Array.ConvertAll(line?.Split(' '), int.Parse));
            reader.Close();
        }

        /// <summary>
        /// 加载所有进程以及他们需要的最大资源
        /// </summary>
        private void LoadProcessMaxResource()
        {
            var reader =
                new StreamReader(Path.Combine(Environment.CurrentDirectory, "Exp3", "input", "Max_list.txt"));
            var line = reader.ReadLine();
            var index = 0;
            _processCount = int.Parse(line?.Trim());
            while (!reader.EndOfStream) {
                line = reader.ReadLine();
                var tempProc = new ProcessExp3(index++) {
                    Max = Array.ConvertAll(line?.Split(' '), int.Parse)
                };
                _processes.Add(tempProc);
            }

            reader.Close();
        }

        /// <summary>
        /// 获取所有进程已经分配的资源
        /// </summary>
        private void LoadAllocationResource()
        {
            var reader = new StreamReader(Path.Combine(
                Environment.CurrentDirectory, "Exp3", "input", "Allocation_list.txt"));
            var index = 0;
            do {
                var line = reader.ReadLine()?.Trim();
                _processes[index++].Allocation = Array.ConvertAll(line?.Split(' '), int.Parse);
            } while (!reader.EndOfStream);

            reader.Close();
        }

        private void EvaluateProcessNeedResource() { }

        public static void Run()
        {
            new Main();
        }
    }
}
```

**PS：这个代码里面没有包括`Program`类，要运行代码的话，需要加上`Program`类，在`Main`方法里调用`Exp3.Main.Run();`**

## 运行结果

```
## 

系统进程数量：5；资源种类数量：3
可用资源向量 Available：
10 5 7 
最大需求矩阵 Max：
7 5 3 
3 2 2 
9 0 2 
2 2 2 
4 3 3 
已分配矩阵 Allocation：
2 3 2 
2 0 0 
3 0 2 
2 1 1 
0 0 2 
需求矩阵 Need：
5 2 1 
1 2 2 
6 0 0 
0 1 1 
4 3 1 
请输入要操作的进程号：0
请输入资源请求向量：1 1 1
银行家算法检验中...
安全序列如下：
P0,P1,P2,P3,P4
已经通过安全性测试！
正在为进程 0 分配资源
已经保存新的分配状态！
```

重新分配之后的状态：

```
-------------------------银行家算法-------------------------
系统进程数量：5；资源种类数量：3
可用资源向量 Available：
10 5 7 
最大需求矩阵 Max：
7 5 3 
3 2 2 
9 0 2 
2 2 2 
4 3 3 
已分配矩阵 Allocation：
3 4 3 
2 0 0 
3 0 2 
2 1 1 
0 0 2 
需求矩阵 Need：
4 1 0 
1 2 2 
6 0 0 
0 1 1 
4 3 1 
```


## 欢迎交流
交流问题请在微信公众号后台留言，每一条信息我都会回复哈~
- 微信公众号：画星星高手
- 打代码直播间：[https://live.bilibili.com/11883038](https://live.bilibili.com/11883038)
- 知乎：[https://www.zhihu.com/people/dealiaxy](https://www.zhihu.com/people/dealiaxy)
- 专栏：[https://zhuanlan.zhihu.com/deali](https://zhuanlan.zhihu.com/deali)
- 简书：[https://www.jianshu.com/u/965b95853b9f](https://www.jianshu.com/u/965b95853b9f)
