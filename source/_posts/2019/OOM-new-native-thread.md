---
title: OOM-new native thread
permalink: OOM-new native thread
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:49:14
---

## 标题
java.lang.OutOfMemoryError:Unable to create new native thread

## 前言  
本文翻译自 https://plumbr.io/outofmemoryerror/unable-to-create-new-native-thread

## Overview

java应用程序天生就是多线程的。这个意思就是说用java编写的程序看起来同一时刻可以做好几个事情。例如--即时只有在一个处理器上--当你从一个窗口内容拖动到另一个时，后台的音乐播放并没有停止，因为你可以在此时执行好几个操作。

一个思考线程的方式是你把它当做工人给他们任务去执行。当你只有一个工人的时候，他只能去做一个任务。但当你有很多工人时，他们可以同时完成几个命令。

现在，正如现实世界的工人一样，JVM的线程需要一些空间去执行这些他们被分配处理的工作。当这里更多的线程超过了他们在内存的空间时，我们就产生了一个基础的问题：

![new-native-thead](unable-to-create-new-thead.png)

错误信息*java.lang.OutOfMemoryError: Unable to create new native thread*意思是**Java应用程序已达到它能启动多少线程的限制。**
>Java application has hit the limit of how many Threads it can launch

## What is causing it?

当JVM要求一个OS提供一个新线程的时候，你有一个机会去展示这个错误*java.lang.OutOfMemoryError: Unable to create new native thread*.只要底层的OS无法分配一个新的线程，那么就将会引发OutOfMemoryError的抛出。本地线程的限制是十分依赖于平台的。因此我们建议可以通过运行以下的测试例子来测试。但是，一般来说，*java.lang.OutOfMemoryError: Unable to create new native thread*抛出的情况会经过以下这几个阶段：

1. 在JVM中当应用程序申请一个新的java线程。
2. JVM本地代码代理了这个请求在OS操作系统上去创建一个新的线程。
3. OS操作系统尝试创建一个新的线程，这需要内存分配给线程。
4. 因为32位Java进程耗尽了它的内存地址空间大小，操作系统会拒绝任何本机内存分配。--（2-4）GB进程大小被击中--操作系统的虚拟内存已完全耗尽。
5. *java.lang.OutOfMemoryError: Unable to create new native thread error*此时就会抛出。

## Give me an example

以下代码创建并且开启线程在循环中。当运行这个代码的时候，操作系统会限制的非常快，并且*java.lang.OutOfMemoryError: Unable to create new native thread message*将会展示：
```
while(true){
    new Thread(new Runnable(){
        public void run() {
            try {
                Thread.sleep(10000000);
            } catch(InterruptedException e) { }        
        }    
    }).start();
}
```

这个精确的线程限制是依赖于平台的，例如在Windows,Linux和Mac OS X呈现：
- 64-bit Mac OS X 10.9,Java 1.7.0_45-JVM在创建了#2031线程之后挂了
- 64-bit Ubuntu Linux, Java 1.7.0_45-JVM在创建了#31893线程之后挂了 
- 64-bit Windows 7, Java 1.7.0_45-由于由操作系统使用不同的线程模型,似乎没有引发此错误在这个特殊的平台。在线程是@250000进程之后仍然存活，即使交换文件已增加到10GB，该应用程序正面临极端性能问题。

所以确保你知道的机器极限可以通过调用一个小的测试，然后找出什么时候将会触发*java.lang.OutOfMemoryError: Unable to create new native thread*错误。

## What is solution?

偶尔你可以绕过这个限制通过增加OS的限制。例如，如果你已经限制了程序的数量。JVM可以在用户空间引发这个检查，并且这个限制还会增大：
```
[root@dev ~]# ulimit -a
core file size          (blocks, -c) 0
--- cut for brevity ---
max user processes              (-u) 1800
```

通常来说不会的，程序错误时OoutOfMemoryError将会表明新的线程无法创建。当在您的应用程序中生成成千上万的线程时，那么有些东西已经发生了可怕的错误--这里将不会有很多应用程序受益于这样一个巨大的线程的数量。

解决这个问题的一个方法是开启一个任务线程去dumps这些东西，并去了解这个和场景。这通常需要花费部分时间。
