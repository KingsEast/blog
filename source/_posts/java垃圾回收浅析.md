---
title: Java垃圾回收浅析
categories:
  - Java
tags: [Java,GC]
copyright: true
reward: true
toc: true
abbrlink: 52a381d
date: 2017-12-31 11:05:37
description:
password:
---

# GC的三种收集方法

## 标记清除

标记清除算法是最基础的收集算法，其他收集算法都是基于这种思想。

标记清除算法分为“标记”和“清除”两个阶段：首先标记出需要回收的对象，标记完成之后统一清除对象。

![vern标记清除算法](http://ouat6a0as.bkt.clouddn.com/vern标记清除算法.jpg)

主要缺点：

1. 效率问题，标记和清除过程效率不高 。
2. 空间问题，标记清除之后会产生大量不连续的内存碎片。

## 标记整理

标记整理，主要用于回收老年代。

标记操作和“标记-清除”算法一致，后续操作不只是直接清理对象，而是在清理无用对象完成后让所有存活的对象都向一端移动，并更新引用其对象的指针。

![vern标记整理算法](http://ouat6a0as.bkt.clouddn.com/vern标记整理算法.jpg)

主要缺点：在标记-清除的基础上还需进行对象的移动，成本相对较高，好处则是不会产生内存碎片。

## 复制算法

复制算法，主要用于回收新生代。

它将可用内存容量划分为大小相等的两块，每次只使用其中的一块。当这一块用完之后，就将还存活的对象复制到另外一块上面，然后在把已使用过的内存空间一次理掉。这样使得每次都是对其中的一块进行内存回收，不会产生碎片等情况，只要移动堆订的指针，按顺序分配内存即可，实现简单，运行高效。

![vern复制算法](http://ouat6a0as.bkt.clouddn.com/vern复制算法.jpg)

主要缺点：内存缩小为原来的一半。

# 分代的垃圾回收策略

分代的垃圾回收策略是基于这样一个事实：不同的对象的生命周期是不一样的。因此，不同生命周期的对象可以采取不同的回收算法，以便提高回收效率。

## 年轻代（Young Generation）

* 所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。
* 新生代内存按照8:1:1的比例分为一个eden区和两个survivor(survivor0,survivor1)区。一个Eden区，两个 Survivor区(一般而言)。大部分对象在Eden区中生成。
  * 回收时先将eden区存活对象复制到一个survivor0区，然后清空eden区；
  * 当这个survivor0区也存放满了时，则将eden区和survivor0区存活对象复制到另一个survivor1区，然后清空eden和这个survivor0区；
  * 此时survivor0区是空的，然后将survivor0区和survivor1区交换，即保持survivor1区为空。 
  * 如此往复循环。
* 当survivor1区不足以存放 eden和survivor0的存活对象时，就将存活对象直接存放到老年代。若是老年代也满了就会触发一次Full GC，也就是新生代、老年代都进行回收
* 新生代发生的GC也叫做Minor GC，MinorGC发生频率比较高(不一定等Eden区满了才触发)

## 年老代（Old Generation）

* 在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
* 内存比新生代也大很多(大概比例是1:2)，当老年代内存满时触发Major GC即Full GC，Full GC发生频率比较低，老年代对象存活时间比较长，存活率标记高。

## 持久代（Permanent Generation）

用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class，例如Hibernate 等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。

> Oracle JDK8的HotSpot VM去掉“持久代”，以“元数据区”（Metaspace）替代之。



<br>

<p id="div-border-top-green"><i>最后要说的是：[博客源码](https://github.com/fakeYanss/blog) ， 欢迎 star</i></p>