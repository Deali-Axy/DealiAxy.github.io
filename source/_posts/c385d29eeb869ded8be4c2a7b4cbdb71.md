---
title: 全文检索引擎原理以及Lucene简单介绍
date: 2020/03/02 10:27:53
abbrlink: c5a8b36d2c1c67bc
categories:
- 搜索引擎
tags:
- 全文检索
- 引擎
- 原理
- 介绍
---
## 前言

继上次的博客介绍whoosh搜索引擎的，我打算写一个“从零开始编写自己的搜索引擎”系列文章，不过转念一想，我应该先写一篇介绍原理的，然后再开始比较好，于是就有了本文。

我们生活中的数据总体分为两种：**结构化数据**和**非结构化数据**。

- **结构化数据：**指具有固定格式或有限长度的数据，如数据库，元数据等。
- **非结构化数据：**指不定长或无固定格式的数据，如邮件，word文档等。

**非结构化数据又一种叫法叫全文数据。**

按照数据的分类，搜索也分为两种：

- **对结构化数据的搜索**：如对数据库的搜索，用SQL语句。再如对元数据的搜索，如利用windows搜索对文件名，类型，修改时间进行搜索等。
- **对非结构化数据的搜索**：如利用windows的搜索也可以搜索文件内容，Linux下的grep命令，再如用Google和百度可以搜索大量内容数据。

对非结构化数据也即对全文数据的搜索主要有两种方法：

一种是**顺序扫描法(Serial Scanning)：**所谓顺序扫描，比如要找内容包含某一个字符串的文件，就是一个文档一个文档的看，对于每一个文档，从头看到尾，如果此文档包含此字符串，则此文档为我们要找的文件，接着看下一个文件，直到扫描完所有的文件。如利用windows的搜索也可以搜索文件内容，只是相当的慢。假如有一个80G硬盘，如果想在上面找到一个内容包含某字符串的文件，可能需要几个小时的时间。Linux下的grep命令也是这一种方式。这是一种比较原始的方法，但对于小数据量的文件，这种方法还是最直接，最方便的。但是对于大量的文件，这种方法的速度就很慢。

另一种是**全文检索(Full-text Search)**：即先建立索引，再对索引进行搜索。索引是从非结构化数据中提取出之后重新组织的信息。

## 全文检索引擎的基本实现原理

下面这幅图来自《Lucene in action》，但却不仅仅描述了Lucene以及全文检索引擎的一般过程。

{% asset_img 8869373-7c90ae9bfe68c296.png %}

全文检索大体分两个过程，**索引创建(Indexing)和搜索索引(Search)**。

- 索引创建：将现实世界中所有的结构化和非结构化数据提取信息，创建索引的过程。
- 搜索索引：就是得到用户的查询请求，搜索创建的索引，然后返回结果的过程。

## 索引创建

全文检索的索引创建过程一般有以下几步：

#### 第一步：一些要索引的原文档(Document)。

为了方便说明索引创建过程，这里特意用两个文件为例：

- 文件一：Students should be allowed to go out with their friends, but not allowed to drink beer.
- 文件二：My friend Jerry went to school to see his students but found them drunk which is not allowed.

#### 第二步：将原文档传给分次组件(Tokenizer)。

**分词组件(Tokenizer)会做以下几件事情(此过程称为Tokenize)：**

1. **将文档分成一个一个单独的单词。**
2. **去除标点符号。**
3. **去除停词(Stop word)。**

所谓停词(Stop word)就是一种语言中最普通的一些单词，由于没有特别的意义，因而大多数情况下不能成为搜索的关键词，因而创建索引时，这种词会被去掉而减少索引的大小。

英语中挺词(Stop word)如：“the”,“a”，“this”等。

对于每一种语言的分词组件(Tokenizer)，都有一个停词(stop word)集合。

**经过分词(Tokenizer)后得到的结果称为词元(Token)。**

在我们的例子中，便得到以下词元(Token)：

“Students”，“allowed”，“go”，“their”，“friends”，“allowed”，“drink”，“beer”，“My”，“friend”，“Jerry”，“went”，“school”，“see”，“his”，“students”，“found”，“them”，“drunk”，“allowed”。

#### 第三步：将得到的词元(Token)传给语言处理组件(Linguistic Processor)。

语言处理组件(linguistic processor)主要是对得到的词元(Token)做一些同语言相关的处理。

**对于英语，语言处理组件(Linguistic Processor)一般做以下几点：**

1. **变为小写(Lowercase)。**
2. **将单词缩减为词根形式，如"cars"到"car"等。这种操作称为：stemming。**
3. **将单词转变为词根形式，如"drove"到"drive"等。这种操作称为：lemmatization。**

**Stemming 和 lemmatization的异同：**

- 相同之处：Stemming和lemmatization都要使词汇成为词根形式。
- 两者的方式不同：
  - Stemming采用的是“缩减”的方式：“cars”到“car”，“driving”到“drive”。
  - Lemmatization采用的是“转变”的方式：“drove”到“drove”，“driving”到“drive”。
- 两者的算法不同：
  - Stemming主要是采取某种固定的算法来做这种缩减，如去除“s”，去除“ing”加“e”，将“ational”变为“ate”，将“tional”变为“tion”。
  - Lemmatization主要是采用保存某种字典的方式做这种转变。比如字典中有“driving”到“drive”，“drove”到“drive”，“am, is, are”到“be”的映射，做转变时，只要查字典就可以了。
- Stemming和lemmatization不是互斥关系，是有交集的，有的词利用这两种方式都能达到相同的转换。

**语言处理组件(linguistic processor)的结果称为词(Term)。**

在我们的例子中，经过语言处理，得到的词(Term)如下：

“student”，“allow”，“go”，“their”，“friend”，“allow”，“drink”，“beer”，“my”，“friend”，“jerry”，“go”，“school”，“see”，“his”，“student”，“find”，“them”，“drink”，“allow”。

也正是因为有语言处理的步骤，才能使搜索drove，而drive也能被搜索出来。

## 查询索引

当用户输入关键词进行查找的时候，搜索引擎在索引中寻找匹配的索引，并通过索引找到原文档，这就是查询索引的过程，但是这还没完，你以为找到原文档就完事了吗？nope，如果仅仅只有一个或十个文档包含我们查询的字符串，我们的确找到了。然而如果结果有一千个，甚至成千上万个呢？那个才你最想要的文件呢？

这时候就需要根据评分算法计算出每个结果的权重，排序后再给用户展示了~

查询索引一般会经过一下四个步骤：

1. 用户输入查询语句
2. 对查询语句进行词法分析（识别单词和关键字），语法分析（根据查询语句的语法规则来形成一棵语法树），及语言处理（原始语言的进一步加工）
3. 搜索索引，得到符合语法树的文档
4. 根据得到的文档和查询语句的相关性，对结果进行排序

判断词(Term)之间的关系从而得到文档相关性的过程应用一种叫做向量空间模型的算法(Vector Space Model)**

下面仔细分析一下这两个过程：

###### 1. 计算权重(Term weight)的过程。

影响一个词(Term)在一篇文档中的重要性主要有两个因素：

- Term Frequency (tf)：即此Term在此文档中出现了多少次。tf 越大说明越重要。
- Document Frequency (df)：即有多少文档包含此Term。df 越大说明越不重要。

容易理解吗？词(Term)在文档中出现的次数越多，说明此词(Term)对该文档越重要，如“搜索”这个词，在本文档中出现的次数很多，说明本文档主要就是讲这方面的事的。然而在一篇英语文档中，this出现的次数更多，就说明越重要吗？不是的，这是由第二个因素进行调整，第二个因素说明，有越多的文档包含此词(Term), 说明此词(Term)太普通，不足以区分这些文档，因而重要性越低。

这也如我们程序员所学的技术，对于程序员本身来说，这项技术掌握越深越好（掌握越深说明花时间看的越多，tf越大），找工作时越有竞争力。然而对于所有程序员来说，这项技术懂得的人越少越好（懂得的人少df小），找工作越有竞争力。人的价值在于不可替代性就是这个道理。

道理明白了，我们来看看公式：

{% asset_img 8869373-976f02f22da5543d.png %}

{% asset_img 8869373-37a0a9696941004c.png %}

这仅仅只term weight计算公式的简单典型实现，不同的全文检索系统的会有不同的实现，Lucene就与此稍有不同。

###### 2. 判断Term之间的关系从而得到文档相关性的过程，也即向量空间模型的算法(VSM)。

我们把文档看作一系列词(Term)，每一个词(Term)都有一个权重(Term weight)，不同的词(Term)根据自己在文档中的权重来影响文档相关性的打分计算。

于是我们把所有此文档中词(term)的权重(term weight) 看作一个向量。

Document = {term1, term2, …… ,term N}

Document Vector = {weight1, weight2, …… ,weight N}

同样我们把查询语句看作一个简单的文档，也用向量来表示。

Query = {term1, term 2, …… , term N}

Query Vector = {weight1, weight2, …… , weight N}

我们把所有搜索出的文档向量及查询向量放到一个N维空间中，每个词(term)是一维。如图：

{% asset_img 8869373-3d85f0c39246a75f.jpg %}

我们认为两个向量之间的夹角越小，相关性越大。

所以我们计算夹角的余弦值作为相关性的打分，夹角越小，余弦值越大，打分越高，相关性越大。

有人可能会问，查询语句一般是很短的，包含的词(Term)是很少的，因而查询向量的维数很小，而文档很长，包含词(Term)很多，文档向量维数很大。你的图中两者维数怎么都是N呢？

在这里，既然要放到相同的向量空间，自然维数是相同的，不同时，取二者的并集，如果不含某个词(Term)时，则权重(Term Weight)为0。

相关性打分公式如下：

{% asset_img 8869373-79429a682851b58f.png %}

举个例子，查询语句有11个Term，共有三篇文档搜索出来。其中各自的权重(Term weight)，如下表格。

|      | t1   | t2   | t3   | t4   | t5   | t6   | t7   | t8   | t9   | t10  | t11  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| D1   | 0    | 0    | .477 | 0    | .477 | .176 | 0    | 0    | 0    | .176 | 0    |
| D2   | 0    | .176 | 0    | .477 | 0    | 0    | 0    | 0    | .954 | 0    | .176 |
| D3   | 0    | .176 | 0    | 0    | 0    | .176 | 0    | 0    | 0    | .176 | .176 |
| Q    | 0    | 0    | 0    | 0    | 0    | .176 | 0    | 0    | .477 | 0    | .176 |

于是计算，三篇文档同查询语句的相关性打分分别为：

{% asset_img 8869373-7ee898daff887329.png %}

于是文档二相关性最高，先返回，其次是文档一，最后是文档三。

到此为止，我们可以找到我们最想要的文档了。

## Lucene简单介绍

### 索引创建和搜索过程

{% asset_img 8869373-078e62c16f28fdb6.jpg %}

> 1. 索引过程：
>    1) 有一系列被索引文件
>    2) 被索引文件经过语法分析和语言处理形成一系列词(Term)。
>    3) 经过索引创建形成词典和反向索引表。
>    4) 通过索引存储将索引写入硬盘。

> 2. 搜索过程：
>    a) 用户输入查询语句。
>    b) 对查询语句经过语法分析和语言分析得到一系列词(Term)。
>    c) 通过语法分析得到一个查询树。
>    d) 通过索引存储将索引读入到内存。
>    e) 利用查询树搜索索引，从而得到每个词(Term)的文档链表，对文档链表进行交，差，并得到结果文档。
>    f) 将搜索到的结果文档对查询的相关性进行排序。
>    g) 返回查询结果给用户。

### 建立索引

为了对文档进行索引，Lucene 提供了五个基础的类，他们分别是 Document, Field, IndexWriter, Analyzer, Directory。下面我们分别介绍一下这五个类的用途：

**Document**

Document 是用来描述文档的，这里的文档可以指一个 HTML 页面，一封电子邮件，或者是一个文本文件。一个 Document 对象由多个 Field 对象组成的。可以把一个 Document 对象想象成数据库中的一个记录，而每个 Field 对象就是记录的一个字段。

**Field**

Field 对象是用来描述一个文档的某个属性的，比如一封电子邮件的标题和内容可以用两个 Field 对象分别描述。

**Analyzer**

在一个文档被索引之前，首先需要对文档内容进行分词处理，这部分工作就是由 Analyzer 来做的。Analyzer 类是一个抽象类，它有多个实现。针对不同的语言和应用需要选择适合的 Analyzer。Analyzer 把分词后的内容交给 IndexWriter 来建立索引。

**IndexWriter**

IndexWriter 是 Lucene 用来创建索引的一个核心的类，他的作用是把一个个的 Document 对象加到索引中来。

**Directory**

这个类代表了 Lucene 的索引的存储的位置，这是一个抽象类，它目前有两个实现，第一个是 FSDirectory，它表示一个存储在文件系统中的索引的位置。第二个是 RAMDirectory，它表示一个存储在内存当中的索引的位置。

### 搜索文档

利用 Lucene 进行搜索就像建立索引一样也是非常方便的。在上面一部分中，我们已经为一个目录下的文本文档建立好了索引，现在我们就要在这个索引上进行搜索以找到包含某个关键词或短语的文档。Lucene 提供了几个基础的类来完成这个过程，它们分别是呢 IndexSearcher, Term, Query, TermQuery, Hits. 下面我们分别介绍这几个类的功能。

**Query**

这是一个抽象类，他有多个实现，比如 TermQuery, BooleanQuery, PrefixQuery. 这个类的目的是把用户输入的查询字符串封装成 Lucene 能够识别的 Query。

**Term**

Term 是搜索的基本单位，一个 Term 对象有两个 String 类型的域组成。生成一个 Term 对象可以有如下一条语句来完成：Term term = new Term(“fieldName”,”queryWord”); 其中第一个参数代表了要在文档的哪一个 Field 上进行查找，第二个参数代表了要查询的关键词。

**TermQuery**

TermQuery 是抽象类 Query 的一个子类，它同时也是 Lucene 支持的最为基本的一个查询类。生成一个 TermQuery 对象由如下语句完成： TermQuery termQuery = new TermQuery(new Term(“fieldName”,”queryWord”)); 它的构造函数只接受一个参数，那就是一个 Term 对象。

**IndexSearcher**

IndexSearcher 是用来在建立好的索引上进行搜索的。它只能以只读的方式打开一个索引，所以可以有多个 IndexSearcher 的实例在一个索引上进行操作。

**Hits**

Hits 是用来保存搜索的结果的。

## 欢迎与我交流
- 打代码直播间：[https://live.bilibili.com/11883038](https://live.bilibili.com/11883038)
- 微信公众号：DealiAxy
- 知乎：[https://www.zhihu.com/people/dealiaxy](https://www.zhihu.com/people/dealiaxy)
- 博客：[https://blog.deali.cn](https://blog.deali.cn)
- 简书：[https://www.jianshu.com/u/965b95853b9f](https://www.jianshu.com/u/965b95853b9f)
