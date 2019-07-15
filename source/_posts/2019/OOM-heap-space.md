---
title: OOM-heap space
permalink: OOM-heap space
tags: [Java,OOM]
date: 2019-07-15 20:03:36
categories: [OOM]
---

## 标题 
java.lang.OutOfMemoryError:Java heap space

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/java-heap-space

## Overview
Java程序仅仅被使用有限的内存空间。这个空间在程序被启动时指定了。为了让事情更复杂，java内存被分割成2个区域。这些区域被称为堆空间和永久代(用于永久代)  
![head space](https://plumbr.io/wp-content/uploads/2014/04/java-lang-outofmemoryerror-java-heap-space.png)

这些区域的大小被设置了在JVM启动时，并且能够通过JVM参数-**Xmx**和-**XX:MaxPermSize**定制。如果没有明确指定这些值，那么将使用平台特有的默认值。

`java.lang.OutOfMemoryError: Java heap space error`将会被触发，**当应用程序试图添加更多的数据放进堆空间区域，但没有足够的空间给它**。
>when the application attempts to add more data into the heap space area, but there is not enough room for it

注意可能有很多可用的物理内存，但是每当JVM到达堆大小的限制时，都会抛出异常`java.lang.OutOfMemoryError: Java heap space error`

## What is causing it?
抛出`java.lang.OutOfMemoryError: Java heap space error`大多数时候原因都是很简单的--你尝试去将XXL大小的应用程序装入S大小的堆空间。意思就是--应用程序需要更多堆空间去正常运作。还有其他导致OutOfMemoryError错误信息的原因更复杂并且由于程序错误：
- **使用/数据的容量峰值** 应用程序被设计为处理一定量的用户或特定数量的数据，当用户或数据的数量突然暴涨并且穿过预期阈值。那么这些正常程序在峰值操作时就会导致触发` java.lang.OutOfMemoryError: Java heap space error`
- **内存泄漏** 特定类型的编程错误会导致你的应用程序不断消耗更多的内存，每一次这个泄漏内存的功能被使用就会导致更多的内存对象进入到堆空间中。随着时间推移，泄漏的对象占用所有可用的Java堆空间并且会触发常见的错误`java.lang.OutOfMemoryError: Java heap space`错误。

## Give me an example
- 普通的例子  
第一个例子是一个真实的例子--以下的java代码尝试去分配2M的Integer数组。当你完成它并且使用12M大小的java堆内存（java -Xmx12m OOM）启动时，它将会抛出错误`java.lang.OutOfMemoryError: Java heap space`，使用13M的java堆内存就会运行的很好。
```
class OOM {
  static final int SIZE=2*1024*1024;
  public static void main(String[] a) {
    int[] i = new int[SIZE];
   }
}
```

- 内存泄漏的例子  
第二个例子是一个更常见的内存泄漏的例子。在java中，当开发者创建和使用新对象时，例如*new Integer(5)*，他们并不需要关心是否分配了内存--JVM会处理它们。在应用程序的整个声明周期，JVM会定期检查哪些对象的内存正在被使用哪些不再使用了。不再使用的内存会被发现，并且内存会回收它再次利用，这个过程就叫做垃圾收集。在JVM收集中这个相对应的模版就叫做垃圾收集器（GC）.

Java的自动内存管理依赖于GC定期查找未使用的对象并将它们删除，简单点说，**内存泄漏在java中的场景就是当一些对象不再被应用使用了但是GC却无法识别回收他们**。由于这些未使用的对象无限期的停留在堆空间，最后就导致了抛出异常：java.lang.OutOfMemoryError: Java heap space error.

以下就构建了一个内存泄漏的例子
```
class KeylessEntry {
 
   static class Key {
      Integer id;
 
      Key(Integer id) {
         this.id = id;
      }
 
      @Override
      public int hashCode() {
         return id.hashCode();
      }
   }
 
   public static void main(String[] args) {
      Map m = new HashMap();
      while (true)
         for (int i = 0; i < 10000; i++)
            if (!m.containsKey(new Key(i)))
               m.put(new Key(i), "Number:" + i);
   }
}
```
当你运行上面的代码时候，你可能期望他永远没有任何问题。假如这个缓存解决方案仅仅将Map夸大到10000元素，那么所有的key都将被保存于hashMap中。然而现实中，这些key都将会加入到缓存中，这个类没有合适的equals()实现在hashCode()之后。

因此，随着时间的推移，这些含有内存泄漏的代码不断缓存结果最终就会消耗大量的java堆空间了。当这些泄漏的内存填充满了可用的堆空间并且GC无法清理它的时候，就会抛出异java.lang.OutOfMemoryError: Java heap space

解决方案很简单--实现equals()方法类似于下面这种，那么程序会很顺利的。但在找到这些原因之前，你需要浪费大量的脑细胞了。
```
@Override
public boolean equals(Object o) {
   boolean response = false;
   if (o instanceof Key) {
      response = (((Key)o).id).equals(this.id);
   }
   return response;
}
```

## What is the solution?
有时候，你向JVM申请的内存数量是不够容纳你的应用所需要的。这种情况下，你应该分配更多的内存空间--看文章最后了解如何实现它。

然后在许多场景中，提供更多的内存空间将不会解决这个问题。例如，如果你的应用程序包含了内存泄漏，添加更多的内存空间仅仅会延长java.lang.OutOfMemoryError: Java heap space错误的抛出。增加Java堆空间的数量也会增加GC暂停影响您的应用程序的吞吐量或延迟的长度。

如果你想要根本的解决java堆问题而不是掩盖这个问题，你需要找出你的代码那一部分分配了最多的内存。换句话说，你需要回到如下问题：
1. 哪个对象占据了最大的内存空间？
2. 这些对象在源代码中哪里被分配了？

为了解决这些问题，你需要留个自己一些充足的时间（--看一看下面项目构建的列表）。这是一个解决问题的大纲，可以帮你解决这个问题。
- 获取到安全许可为了得到你的JVM中的堆空间dump文件。“Dumps”是堆空间的快照包含了你需要分析的内容。这些快照包含了机密信息，例如密码，卡号等等。因此你需要安全许可才能得到dump文件。
- 在适当的时候获取dump文件。在你获取一些dump文件时，如果是在错误的时间上，你将会获取一堆的噪音文件，几乎毫无用处。另一方面，每次获取dump文件时JVM可能完全的处于冻结状态，因此不要过多的获取这些dump文件不然你的用户将会面临性能问题。
- 找到可以加载dump文件的机器。当你的JVM需要排查8G以上的堆文件时，你需要超过8G以上内存的机器才能去分析堆内容。使用dump分析文件去分析（我们推荐Eclipse MAT，但也有其他更好的替代品）
- 检测堆中的最大消费者的GC根的路径。这里有篇[文章](https://plumbr.io/blog/memory-leaks/solving-outofmemoryerror-dump-is-not-a-waste)来介绍如何处理.这是对初学者来说特别艰难，但实际上会让你了解结构和导航。
- 接下来，您需要找出源代码中的潜在危险大量对象被分配。如果你有好的应用程序源代码知识你可以做这几个搜索。

或者，我们推荐[Plumbr, the only Java monitoring solution with automatic root cause detection](https://plumbr.io/)。在其他的性能问题中，它也会一并捕获所有的java错误，并自动向你呈现这些信息，这些信息都是关于内存饥饿的数据结构。

Plumbr负责收集必要的数据在后台–这包括有关堆使用情况的相关数据（只有对象布局图，没有实际的数据）。同时也包含了一些你不能找到的java的堆数据。它也对这些数据做了必要处理--当JVM发生java.lang.OutOfMemoryError错误时。以下是一个Plumbr分析的附带的java.lang.OutOfMemoryError例子。

![outofmemeoryerror-analyzed](https://plumbr.io/wp-content/uploads/2015/08/outofmemoryerror-analyzed.png)

无需任何工具的情况下，你可以看到：
- 那个对象消费了大量的内存（271个com.example.map.impl.PartitionContainer实例消费了总248MB内存中的173MB）
- 这些对象被分配在了哪里（大多数的对象被分配在了MetricManagerImpl当中，line 304）
- 哪些是当前对象完整的引用（完整的引用链GC根）

配备此信息，您可以缩放到根本原因，并确保数据结构调整到正常水平，在这里他们会非常适合你的内存池。

但是，当你从内存分析或从阅读Plumbr报告得出的结论是，内存使用是合法的，源代码没有任何问题，那么您需要允许JVM更多Java堆空间才能正常运行。在这种情况下，改变您的JVM启动配置和添加(或增加价值如果存在)如下：
```
-Xmx1024m
```

上面的配置将使Java堆空间的1024MB的应用。您可以使用g或G GB、m或M MB，k或K为KB。例如以下所有的都是等于说，最大的堆空间是1GB：
```
  java -Xmx1073741824 com.mycompany.MyClass
  java -Xmx1048576k com.mycompany.MyClass
  java -Xmx1024m com.mycompany.MyClass
  java -Xmx1g com.mycompany.MyClass
```
