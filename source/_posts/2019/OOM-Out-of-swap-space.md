---
title: OOM-Out of swap space
permalink: OOM-Out of swap space
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:50:41
---

## 标题
java.lang.OutOfMemoryError:Out of swap space?

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/out-of-swap-space

java应用程序在启动时，就被限制使用一定量的内存。这个限制是通过几个指定的参数-Xmx和其他类似的启动的参数。在另一种场景中，JVM所要求的总内存大于可用的物理内存，操作系统启动时将替换硬盘内容到内存。

![out-of-swap-space](out-of-swap-space.png)

错误*java.lang.OutOfMemoryError: Out of swap space*表明交换空间也耗尽，由于缺乏物理内存和交换空间的新尝试分配失败。

## What is causing it?

错误*java.lang.OutOfmemoryError: Out of swap space*将会被抛出，当JVM在堆中的字节分配失败时，并且本地的堆将会耗尽。这个信息表明了分配的大小（bytes大小）是失败的，并且是由于内存请求的原因。

这个问题发生在这种场景就是Java进程已经开始了交换，但是java垃圾收集的回收并不在一个不合适的场景。现在的GC算法并不是很好，但当面对延迟所引起的问题交换，GC中断倾向于增大这个级别，这个级别是大多数的应用程序所不能容忍的。

错误*java.lang.OutOfMemoryError: Out of swap space*经常有操作系统级别的错误导致，例如：
- 操作系统配置了一个不足的swap空间。
- 操作系统的其他程序消耗了所有的内存资源。

有时候也可能是应用程序的错误归因于本地泄漏，例如，应用程序不断的申请内存但是缺不释放这个操作系统的内存。

## What is the solution?
为了克服这个问题，你有一下几种方式。最简单和最常用的方式就是去增大swap空间。这意味是平台相关的。例如，在Linux上，你可以输入以下几个命令，创建并且达到一个新的640MB的swapfile。
```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```
你应该晓得，由于垃圾收集器清扫内存内容，这使得一般的Java进程的交换不可用。运行中的垃圾收起算法在交换分配区能够增加这个GC暂定好几个数量级，因此，你应该多思考下是否使用最简单的办法处理这种情况。

如果您的应用程序部署的旁边有"吵闹的邻居"，即需要JVM去争夺资源，你应该隔离这些服务分离(虚拟)的机器。

然而在大多数情况下，你唯一真正可行的选择是要么机器升级到包含更多的内存或优化应用程序以减少内存占用。当你转向优化这条路时，一个开始的好方法是使用内存转储分析器来检测分配的内存。


