---
title: OOM-Metaspace
permalink: OOM-Metaspace
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:45:30
---

## 标题
java.lang.OutOfMemoryError:Metaspace

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/metaspace

## Overview
Java应用程序仅仅被允许使用有限的内存。当应用启动的时候，你的应用就确定了可以使用的内存量。为了让事情更复杂，Java内存分为下面的图中可以看到的不同的地区：

![metaspace](oom-metaspace.png)

这些区域包括permgen区域，都在JVM启动时，被设置了。如果你没有设置这些，那么将会采取平台默认值。

当Metaspace的内存空间耗尽之后，将会触发*The java.lang.OutOfMemoryError: Metaspace*.

## What is causing it?

如果你不是一个java新手，你可能更熟悉另一概念被称为永久代的Java内存管理。从Java8开始，java内存模型发生了显著变化，一个新的叫Metaspace的内存区域被引入了，同时Permgen被移除了。这个改变被归为以下几个原因，归为但不限定于：
- 永久带的区域需求无法预测，它总是会产生情况，要么内存不足，产生错误: java.lang.OutOfMemoryError: Permgen，要么供应过于充足，产生了很多浪费。 
- GC性能改进，启动并发类数据重新分配不会产生GC暂停和明显的迭代在Metadata中
- 支持如G1并行类卸载的进一步优化

所以如果你很熟悉永久代，这些你所有知道的可以作为背景--在Java8之前无论任何被分配在永久带的内容（类的名称，字段，作为字节码的方法，常量池，JIT优化等）--现在被分配在了Metaspace。

正如你看到的，Metaspace大小要求都依赖于加载的类的数量以及此类声明的大小。这是显而易见的，**当在Metaspace空间加载太多的类或者太大的类，就会产生java.lang.OutOfMemoryError: Metaspace错误。**
>main cause for the java.lang.OutOfMemoryError: Metaspace is: either too many classes or too big classes being loaded to the Metaspace

## Give me an example

正如前几个章节解释的，Metaspace的使用是和类的数量有很强的关联的在JVM中。以下展示一个最粗暴的例子：
````
public class Metaspace {
	static javassist.ClassPool cp = javassist.ClassPool.getDefault();

	public static void main(String[] args) throws Exception{
		for (int i = 0; ; i++) { 
			Class c = cp.makeClass("eu.plumbr.demo.Generated" + i).toClass();
		}
	}
}
````
这个源码就是循环生成类。所有这些生成的类定义最后都消耗着Metaspace空间。这些复杂的类都是javassist库生成的。

这个代码将会保持持续生成新的类，并在Metaspace中加载这些类定义，知道空间被占满，并且抛出错误*java.lang.OutOfMemoryError: Metaspace*.当在Mac OS X中采取Java 1.8.0_05版本启动时，带有参数*-XX:MaxMetaspaceSize=64m*大概会生成70000个类。

注：新参数（MaxMetaspaceSize）用于限制本地内存分配给类元数据的大小。如果没有指定这个参数，元空间会在运行时根据需要动态调整。

## What is the solution?

在面临OutOfMemoryError错误时，Metaspace是第一个解决方案应该是显而易见的。如果应用程序在内存耗尽元空间区域你应该增加元空间的大小。改变你的应用程序启动并增加以下配置：
```
-XX:MaxMetaspaceSize=512m
```
上述Metaspace的配置示例指示JVM在抛出错误OutOfMemoryError时，将空间处理成512MB。

另一个解决方案是更简单。你可以删除Metaspace上所有的参数移除这个限制。注意这样操作之后，你将引入比较重的内存交换或者达到本地分配失败的情况。

在今晚结束之前，警告下--通常情况它会发生，通过使用上述建议“quick fixes”会隐藏这个错误*java.lang.OutOfMemoryError: Metaspace*，并不能解决根本问题。如果您的应用程序中内存泄漏的内存或只是一些不合理的加载到元空间上述解决方案实际上不会有什么助益，它只会推迟问题。
