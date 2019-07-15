---
title: OOM-Kill process
permalink: OOM-Kill process
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:08:33
---

## 标题
java.lang.OutOfMemoryError:Kill process or sacrifice child

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/kill-process-or-sacrifice-child

## overview
为了理解这种错误，我们需要恢复基本的操作系统。正如我们所知道的，操作系统都是由一系列的进程处理的。这些处理程序被几个主要的kernel jobs控制运行。其中一个进程叫做“Out of memory killer"进程，这是我们所感兴趣的。

当内存特别低的时候，kernel job就会杀掉你的进程。当这个条件出发时，Out of memory killer就会激活并杀掉一个进程。被选中的进程是来自于一个启发式的程序中，它会对所有的进程进行排分然后选择最差的得分去杀掉。`Out of memory: Kill process or sacrifice child`因此是不同于其他的java的OOM异常的。它不是由JVM触发的，也不是代理的，而是一个内置在操作系统内核中的安全网络。

![oom-example](kill-process-or-sacrifice-child.png)

`Out of memory: kill process or sacrifice child error`发生是由于所有可用虚拟内容（包括swap）被过度消耗，这会使得操作系统进入一定的风险状态。在这种情况下，Out of memory killer就会杀掉此进程。

## What is causing it?
默认情况下，Linux内核允许进程请求更多的内存，而不是系统中当前可用的内存。考虑到大多数进程从来没有真正使用它们分配的所有内存，这在全世界都有意义。与这种方法最简单的比较是宽带运营商。他们向所有消费者出售100MIT下载承诺，远远超过其网络中存在的实际带宽。该赌注再次基于用户不会同时使用他们分配的下载限制的事实。因此，一个10GBIT链路可以成功地服务于我们的简单数学允许的100个用户。

这种方法的副作用是可见的，在某些情况下，你的程序正在耗尽系统内存的路径。这可能导致非常低的内存条件，其中没有页面可以被分配来处理。你可能会遇到这样的情况，甚至根本没有一个根基帐户不能杀死违规的任务。为了防止这种情况，杀手激活，并识别流氓进程被杀死。

你可以阅读更多的内容关于“Out of memory killer” [this article from RedHat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/performance_tuning_guide/s-memory-captun)

现在我们有了这些上下文内容了，那么你怎么知道什么时候触发“killer”并在凌晨5点叫醒你？常见的一个激活操作触发器是隐藏在操作系统配置中。当您在/proc/sys／vm/overcommit_memory路径中，您将得到第一个提示——这里指定的值指示是否允许所有的malloc()调用成功。请注意，proc文件系统中的参数路径取决于受更改影响的系统。

过度提交配置即允许分配越来越多的内存用于这个无赖进程，这最终会触发“Out of memory killer”来完成它想要做的事情。

## Give me an example
编译并且运行以下的代码
```
package eu.plumbr.demo;

public class OOM {

public static void main(String[] args){
	java.util.List<int[]> l = new java.util.ArrayList();
	for (int i = 10000; i < 100000; i++) {
			try {
				l.add(new int[100_000_000]);
			} catch (Throwable t) {
				t.printStackTrace();
			}
		}
	}
}
```
你将会在系统日志中见到如下的内容（/var/log/kern.log in our example）
```
Jun  4 07:41:59 plumbr kernel: [70667120.897649] Out of memory: Kill process 29957 (java) score 366 or sacrifice child
Jun  4 07:41:59 plumbr kernel: [70667120.897701] Killed process 29957 (java) total-vm:2532680kB, anon-rss:1416508kB, file-rss:0kB
```
你需要去调整swapfile和heap尺寸。在我们的情况中，我们需要使用2g的内空间通过修改-Xmx2g并且配置如下的swap配置：
```
swapoff -a 
dd if=/dev/zero of=swapfile bs=1024 count=655360
mkswap swapfile
swapon swapfile
```

## What is the solution?
有几种方法来处理这种情况。克服这个问题的第一个也是最直接的方法是将系统迁移到具有更多内存的实例。

其他的可能性包括对OOM杀手进行微调，在几个小实例中水平地缩放负载或减少应用程序的内存需求。

一个我们不愿意推荐的解决方案是涉及增加交换空间。当你回忆起Java是垃圾收集语言时，这个解决方案似乎已经不那么赚钱了。在物理内存中运行时，现代GC算法是有效的，但是当处理交换分配时，效率是被捶打的。交换可以在几个数量级内增加GC暂停的长度，所以在跳转到这个解决方案之前，你应该三思而后行。
