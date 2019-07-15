---
title: OOM-GC Overhead Limit Exceeded
permalink: OOM-GC Overhead Limit Exceeded
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:55:05
---

## 标题
OutOfMemoryError: GC Overhead Limit Exceeded

## 前言
本文翻译自 http://www.baeldung.com/java-gc-overhead-limit-exceeded 

## 正文

1. 概述  
当对象不再被使用时，JVM会小心地释放它所占用的内存，这个进行被叫做Garbage Collection（GC）。    
异常`GC Overhead Limit Exceeded error`是`java.lang.OutOfMemoryError`家族的一员，并且很明显地它表示了资源（内存）消耗殆尽。  
在下面的文章中，我们将会寻找是什么原因导致了`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded error`并且如何解决它们。  
2. GC Overhead Limit Exceeded Error  
`OutOfMemoryError`是`java.lang.VirtualMachineError`的子类，当JVM遇到与资源利用有关的问题时，就会抛出这个异常。更确切的说，**当JVM花费了太多时间执行垃圾收集时，这个错误就会发生**并且它回收的堆内存很少。  
按照Java文档的说法，默认情况下，JVM配置如果java进程花费超过98%的时间来做GC但是每次只能回收2%不到的堆内存，那么这个错误就会被抛出。换句话说，这意味着我们的应用程序用尽了几乎所有可用的内存，垃圾收集器花费了太多的时间来清理它，并且多次失败了。  
在这种情况下，用户就会感觉到应用程序特别慢。某些通常亿毫秒为单位完成的操作，此时也需要更长的时间才能完成了。这是因为CPU正在使用它全部的能力在垃圾收集器上，因此不能够执行其他的任务了。  
3. 错误的场景
让我们来看一段运行代码能够抛出'java.lang.OutOfMemoryError: GC Overhead Limit Exceeded.'例如，我们可以完成这段代码，添加key-value的对到一个无结束的循环中。
```
public class OutOfMemoryGCLimitExceed {
    public static void addRandomDataToMap() {
        Map<Integer, String> dataMap = new HashMap<>();
        Random r = new Random();
        while (true) {
            dataMap.put(r.nextInt(), String.valueOf(r.nextInt()));
        }
    }
}
```
当方法被执行时，带有JVM参数：`-Xmx100m -XX:+UseParallelGC`（堆内存设置为100MB并且GC算法采取ParallelGC）我们能够得到错误` java.lang.OutOfMemoryError: GC Overhead Limit Exceeded error`，为了能够更好的理解不同的垃圾收集器算法，我们可以查看Oracle的文档[Java Garbage Collection Basics](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)  
我们也可以快速运行以下的命令在项目根路径得到异常` java.lang.OutOfMemoryError: GC Overhead Limit Exceeded error`
```
mvn exec:exec
```
还需要注意点的是，在某些场景下，我们可能在`GC Overhead Limit Exceeded`错误之前遇到`heap space error`  
4. 解决GC Overhead Limit Exceeded Error
去解决这个应用程序根本的问题是去检查引起任何内存泄漏的代码。  
以下的问题需要去处理：
- 程序中哪个对象占据这大量的堆内存空间？
- 那一部分代码是分配这些对象的？  
我们也可以使用自动化的图形工具例如[JConsole](https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html)来检测袋中包含`java.lang.OutOfMemoryErrors`的性能问题。  
最后的办法是通过改变JVM的启动配置来增加堆的大小。例如为这个应用程序分配1G的堆内存空间。
```
java -Xmx1024m com.xyz.TheClassName
```
然后，如果应用程序代码中真正存在内存泄漏，这种方法将不会解决问题。相反，我们将会延期这个错误。因此，更可取的是要彻底重新评估应用程序的内存使用。  
5. 结论
在这个指导中，我们已经明确了`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded `异常，并且知道了它背后的原因。  
一如既往，可以找到与本文相关的源代码(Over on GitHub)[https://github.com/eugenp/tutorials/tree/master/core-java]
