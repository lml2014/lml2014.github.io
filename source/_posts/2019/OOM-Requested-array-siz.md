---
title: OOM-Requested array siz
permalink: OOM-Requested array siz
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:53:17
---

## 标题
java.lang.OutOfMemoryError:Requested array size exceeds VM limit

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/requested-array-size-exceeds-vm-limit

## OverView
在你的应用程序分配内存时，java有个最大的数组长度限制。确切的来说，它限制的是特定的平台，一般是1到2.1亿个元素。

![Requested array size](outofmemoryerror-array-size-exceeds-vm-limit.png)

当你面对`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`异常时，它意味着应用程序奔溃了，这个错误是应用程序尝试去申请一个大数组超过了java虚拟机能够支持的限度。

## What is causing it?
这个错误是由JVM本地代码抛出的错误。它发生在你申请一个数组内存之前，当JVM执行特定平台的检查时：在这个平台中，无论分配好的数据结构是否是可用的。在你最初的思考中，这个错误是不常见的。

你几乎很少面对这个错误的原因是Java数组是由int索引的。这个最大的位置int是2^31-1=2,147,483,647。并且在特定平台中，这个限制能够被关闭--例如，在我64位的MB Pro的java 1.7版本中，我能够初始化这个数组尺寸到2,147,483,645或者Integer.MAX_VALUE-2个元素。

增加这个数组的长度1位到Integer.MAX_VALUE-1的结果就是OutOfMemoryError。
```
Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit
```
但是这个限制可能不会超过这个高度--再32位的Linux带有OpenJDK6中，你将会得到`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`错误当你分配一个数组带有1.1亿的元素时。为了理解特定环境的限制，运行下一章中描述的小测试程序。

## Give me an example
运行如下的代码将会得到错误：
```
for (int i = 3; i >= 0; i--) {
	try {
		int[] arr = new int[Integer.MAX_VALUE-i];
		System.out.format("Successfully initialized an array with %,d elements.\n", Integer.MAX_VALUE-i);
	} catch (Throwable t) {
		t.printStackTrace();
	}
}
```
该示例迭代四次，并在每个回合中初始化长基元数组。这个程序试图初始化的数组的大小随着每次迭代增长一个，最后到达整数。现在，当用热点7启动64位Mac OS X上的代码片段时，应该得到与以下类似的输出：
```
java.lang.OutOfMemoryError: Java heap space
	at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
java.lang.OutOfMemoryError: Java heap space
	at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at eu.plumbr.demo.ArraySize.main(ArraySize.java:8)
```
最后2次发生了异常`java.lang.OutOfMemoryError: Requested array size exceeds VM limit`，还有常见的`java.lang.OutOfMemoryError: Java heap space message`.它发生是因为你尝试在8G的内存冲分配2^31-1个int原始空间，这些内存空间是少于默认的JVM内容的。

这个错误也论证了为什么这个错误这么少见--为了看到JVM限制array size，你需要分配array的大小在正确的范围介于平台和限制和Integer.MAX_INT之间。当我们的例子是运行在Mac OS X的64位Hotspot 7机器时，这里只有2个array的length限制：Integer.MAX_INT-1和Integer.MAX_INT

## What is the solution?
这个错误会在以下情况下发生：
1. 你的数组增长过快，并且拥有的大小在平台限制和Integer.MAX_INT之间
2. 你故意去分配数组的长度让他超过2^31-1个元素。

在第一种情况中，检查你的代码去确认你真的需要数组长度这么大。或许你可以减少数组的长度。或者分割你的数组到更小的数据块中，当你需要这个数据时，分批次的加载这些数据信息。

在第二种情况中，要记得java的数组是int索引的。因此，当使用标准的数据结构时，你不能分配超过2^31-1个元素。事实上，你在编译阶段就会得到错误信息“error: integer number too large”来阻止你。

如果你真正需要这么大的数据，你应该重新思考下你的选项。你可以加载你需要的数据在更小的批次数据中，这样仍然可以使用java弓箭。或者你使用其他的标准工具。另一种方式是查看sun.misc.Unsafe的实现。这个允许你分配直接分配内存就像C语言一样。





