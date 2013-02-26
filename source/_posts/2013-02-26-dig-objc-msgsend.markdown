---
layout: post
title: "深入分析 objc_msgSend"
date: 2013-02-26 13:35
comments: true
categories: 
- objective-c
- analyze 
- objc_msgSend
---

在Objective-C中，所有的`[receiver message]`都会转换为`objc_msgSend(receiver, @selector(message));`（[Objective-C Runtime](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html)）。所以相比c/c++这势必是有性能影响，下面就分析objc_msgSend源码看看这个cost到底有多大，并给出在特殊情况下改进方案。

<!-- More -->

在[「objc_msgSend() Tour」](http://www.friday.com/bbum/2009/12/18/objc_msgsend-part-1-the-road-map/) 的系列文章中，已经通过对objc_msgSend汇编源码进行的详细分析，总结出objc_msgSend的处理流程：

>1. Check for ignored selectors (GC) and short-circuit.
>2. Check for nil target.
>		* If nil & nil receiver handler configured, jump to handler
>		* If nil & no handler (default), cleanup and return.
>3. Search the class’s method cache for the method IMP(use **hash** to find&store method in cache)
>		* If found, jump to it.
>		* Not found: lookup the method IMP in the class itself corresponding its hierarchy chain.
>			* If found, load it into cache and  jump to it.
>			* If not found, jump to forwarding mechanism.

在另外一个博客中[「Obj-C Optimization：Method and function call innards」](http://www.mulle-kybernetik.com/artikel/Optimization/opti-3.html)通过分析运行时的汇编代码，给出了更加直观的objc_msgSend运行流程。而且该大神在[「Obj-C Optimization: The faster objc_msgSend」](http://www.mulle-kybernetik.com/artikel/Optimization/opti-9.html)给出了objc_msgSend实现的c语言版本，这个理解起来就更加容易了：

{% gist 5037737 %}

所以第一次调用某个method的时候，会需要运行500多行指令去寻找method并添加到cache；但是之后只需要运行30多行指令，通过hash方法直接在cache中找到相应method。所以可以认为：「**objc_msgSend的cost大概在30多行指令**」。这个cost是十分小的，对于现代CPU来说毛毛雨都不算，所以不用为objc_msgSend带来的cost操碎了心。

但是即使再微小的cost一旦累计多了，也可能带来很大的耗时，特别是一些常常会被调用的核心代码。所以必要时可以这样优化，通过`methodForSelector` 直接获得selector的IMP，将Objective-C method转换为c调用。于是我写了个小测试：

{% gist 5037927 %}

测试结果为：Objective-C Call Cost: *527.266ms*；C Call Cost:*330.790ms*

从上面测试表明，C调用方式比OC调用方式将近快了2倍。如果运行速度成为了程序的瓶颈，采取上面的方法给程序提速不失为一个不错的选择。