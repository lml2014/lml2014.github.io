---
title: OOM-All
permalink: OOM-All
categories: [OOM]
tags: [Java,OOM]
date: 2019-07-15 20:47:48
---

## 标题
java.lang.OutOfMemoryError

## The 8 symptoms that surface them(8个表面症状)

The many thousands of java.lang.OutOfMemoryErrors that I’ve met during my career all bear one of the below eight symptoms. This handbook explains what causes a particular error to be thrown, offers code examples that can cause such errors, and gives you solution guidelines for a fix. The content is all based on my own experience.
(平时遇到了很多的OOM错误，以下是常见的8种情况，我总结了常见错误原因和例子)

Nikita Salnikov-Tarnovski  
Plumbr Co-Founder and VP of Engineering

1. java.lang.OutOfMemoryError:Java heap space
2. java.lang.OutOfMemoryError:GC overhead limit exceeded
3. java.lang.OutOfMemoryError:Permgen space
4. java.lang.OutOfMemoryError:Metaspace
5. java.lang.OutOfMemoryError:Unable to create new native thread
6. java.lang.OutOfMemoryError:Out of swap space?
7. java.lang.OutOfMemoryError:Requested array size exceeds VM limit
8. Out of memory:Kill process or sacrifice child