---
title: OOM-GC overhead
permalink: OOM-GC overhead
tags: [Java,OOM]
date: 2019-07-15 19:46:46
categories: [OOM]
---

## 标题 
java.lang.OutOfMemoryError:GC overhead limit exceeded

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/gc-overhead-limit-exceeded

## overview

java运行环境包含了一个内件的Garbage Collection(GC)进程。在许多其他的语言中，开发者需要自己手动分配和释放内存区域这样他们才能够再次利用被释放的内存。

java应用程序从另一方面将仅仅需要分配内存，每当一片内存区域不再使用时，一个叫垃圾收集器的进程就会开始清理这些内存。GC如何发现这块内存区域在[Garbage Collection Handbook](https://plumbr.io/java-garbage-collection-handbook)已经解释了，但是你可以相信GC工作做的非常好。

**当你的应用程序已经用尽了所有可用的内存，而GC多次都没办法清除它时**将会报错`java.lang.OutOfMemoryError: GC overhead limit exceeded error`

## What is causing it?
`java.lang.OutOfMemoryError: GC overhead limit exceeded`这个错误表明了你的应用程序花费了太多的时间在做垃圾回收了但是得到的结果很少。默认情况下，如果JVM花费了总时间的98%以上在做GC，而在GC之后只有少于2%的堆被恢复，JVM就会被配置抛出这个错误来。
>By default the JVM is configured to throw this error if it spends more than 98% of the total time doing GC and when after the GC only less than 2% of the heap is recovered.

![oom-example](https://plumbr.io/wp-content/uploads/2014/04/OOM-example-schema3.png)

当`GC overhead limit`不存在将会发生什么呢？请注意当几个GC周期之后每次只能释放不到2%的内存是`java.lang.OutOfMemoryError: GC overhead limit exceeded`错误将会被抛出，这意味着只有少量的堆能够被GC清理完成并且这些堆会再次立刻被填充上，再次强迫GC去重启清理进程。这就形成了一个恶性循环，CPU会100%运行用于GC并且实际的工作没法做。应用程序端的用户将会面对极端的慢-往常在几毫秒完成的操作将会花费i数分钟才能完成。

因此`java.lang.OutOfMemoryError: GC overhead limit exceeded`消息是一个执行快速失败原则中很好的例子了。

## Give me an example
在以下的例子中，我们能够创建一个`GC overhead limit exceeded`异常通过初始化一个map并且添加key-value对到这个map中在一个无限循环中：
```
class Wrapper {
  public static void main(String args[]) throws Exception {
    Map map = System.getProperties();
    Random r = new Random();
    while (true) {
      map.put(r.nextInt(), "value");
    }
  }
}
```
正如你可能猜测这不能有好结果。 而事实上，当我们发布以下方案：
```
java -Xmx100m -XX:+UseParallelGC Wrapper
```
这个应用程序将会挂掉，当Map调整长度（resize）时，将会抛出一个更通用的异常` java.lang.OutOfMemoryError: Java heap space`信息。当我运行其他垃圾收集算法时，除了`ParallelGC`之外，例如` -XX:+UseConcMarkSweepGC`或者`-XX:+UseG1GC`时，这个错误将会有默认的异常处理程序捕获，并且堆被耗尽时的程度并没有堆栈跟踪，这个堆栈跟踪不能填充这个异常区域。

这些变化是一个非常好的例子，说明了在资源受限的情况下，你不能预测你的程序将要死亡，因此，不要将你的期望基于这些特定的要完成的操作上。

## What is the solution?

作为一个半开玩笑的建议，如果你仅仅想清除`java.lang.OutOfMemoryError: GC overhead limit exceeded`信息，你可以添加如下的代码他将会实现这个：
```
-XX:-UseGCOverheadLimit   //关闭特性
```

我强烈建议不要使用这种选项--代替去修复这个你只是延迟的不可避免的问题：这个应用程序内存运行不足并且需要被修复。加上这些选项将会掩盖原始的错误`java.lang.OutOfMemoryError: GC overhead limit exceeded`并抛出一个通用的异常`java.lang.OutOfMemoryError: Java heap space`

换一个严肃的话题--有时候GC开销错误被触发是因为你给你的JVM分配的堆内存数量不能够适应你程序的需要在这个JVM上，这种情况，你应该加大你的内存容量--看文章的最后，如何去实现它。

然而在许多情况下，提供更多的内存空间并不能解决这个问题。例如，假如你的应用程序有内存泄漏，添加更多的内存将会延期`java.lang.OutOfMemoryError: Java heap space error`错误信息，同时，添加堆内存也会倾向于增加GC暂停影响您的应用程序的吞吐量或延迟的长度。

如果你想解决根本问题与Java堆空间而不是掩蔽症状，您需要弄清您的代码的哪一部分是负责分配最多内存。换句话说，你需要回答这些问题：
1. 哪个对象占用了大量的堆空间
2. 这些对象在代码中哪里被分配了

在这一点上，可能需要消耗你2天时间(或者请参阅下面的自动项目符号列表)。这是一个艰难的过程大纲，它将帮助你回答上述问题：
- 得到从你的JVM-toTroubleshoot（JVM排查）的堆栈dump文件。"Dumps"基本上是堆内容的快照，它可以分析，并包含所有应用程序时保存在内存转储。包括密码、信用卡号码等。
- 将JVM堆内存的信息转储到一个文件中。如果我们在错误的时间采集一些dump文件，这些文件将会含有大量的噪声，几乎毫无用处。在另一方面，每次堆栈丢弃这些JVM完全冻结的线程，不要这种文件太多，免得同事咒骂。(On the other hand, every heap dump “freezes” the JVM entirely, so don’t take too many of them or your end users start swearing.)
- 找到一台可以加载dump的机器。在JVM排查错误中假如你使用了8GB的堆，那你需要在8GB以上的计算机能够分析堆内容。用分析软件打开它(推荐使用**Eclipse MAT**，也有其他好用的替代品)
- 检测堆中最大消费者的GC根路径。这篇[文章](https://plumbr.io/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste)覆盖了如何去处理。别担心，起初你会觉得它很难，之后几天你会得到更好的挖掘。
- 接下来，您需要找出源代码中的潜在危险大量对象被分配。如果你熟悉您的应用程序的源代码，那么您会很快找到这些地方。当你运气不佳时，你可能需要更多的能量饮料协助。

另外，我们建议采取Plumbr（Java监视解决方案与自动检测根本原因）在其他性能问题中，它会捕获所有的*java.lang.OutOfMemoryErrors*错误，并且自动给你内存渴望的数据结构信息。它会收集必要的数据在后台–这包括有关堆使用情况数据(仅对象布局图，没有实际的数据)，还有一些你甚至不能发现的堆文件。它还为你做了必要的数据处理--当JVM遭遇了*java.lang.OutOfMemoryError*，以下是个OutofMemoryError的例子，通过plumbr:
![outofmemoryerror-analyzed](https://plumbr.io/wp-content/uploads/2015/08/outofmemoryerror-analyzed.png)

无需任何的分析工具，你可以看到：
- 哪个对象消费着最大的内存(271个PartitionContainer实例消费了总共248MB内存中的173MB)
- 这些对象在哪里被分配了（大多数的对象被分配在MetricManagerImpl中）
- 当前哪些引用着这些对象（完整引用链到GC根）

具有这些，您可以缩放到的根本原因，并确保数据结构是下调至水平，在这里他们会非常适合你的内存池。

但是，当你从内存分析或从阅读Plumbr报告得出的结论是，内存使用是合法的，没有异议在源代码中，您需要允许JVM更多Java堆空间才能正常运行。在这种情况下，改变您的JVM启动配置和添加(或增加价值如果存在)就启动脚本中的一个参数：
```
java -Xmx1024m com.yourcompany.YourClass
```
在上面的示例中给出了Java进程堆的1GB。作为最适合的值修改为您的JVM。 但是，如果结果与内存溢出错误，JVM仍然死了，你可能仍然无法避免上述手动或Plumbr辅助分析。
