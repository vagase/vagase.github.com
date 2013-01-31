---
layout: post
title: Timer不靠谱
categories:
- iOS
- Cocoa
- NSRunloop
- NSTimer
- Run Loop
- SDL_Timer
- timer
status: publish
type: post
published: true
comments: true
---
有了这两篇文章作为基础,可以很轻松理解这个问题，timer为什么不是一个精确记时工具，不是实时的。

* ["NSRunLoop概述和原理"](http://vagase.github.com/ios-osx-log-sys-in-deep-part1/) 
* ["如何自己动手做timer"](http://vagase.github.com/how-to-make-timer/)

我们把环境都设置在Cocoa中，这里所讲的timer就用NStimer，当然这里的原理适用于其他系统的timer。

因为NSTimer是作为一种timer resource加入到NSRunLoop中去，在当timer的时间累计到规定时间之后就触发timer的action。从这个过程上看来timer应该是很“准时”的，而且现实情况也是这样的，比如一个规定每1s触发的timer绝大多数情况一般也是1s触发一次。但是timer的这种所谓的“准时”千万不要让你产生这样一种幻觉，“timer可以用来作精确的循环控制，比如用来精准控制动画”。

<!-- More -->

timer的不精确性主要是表现在：timer有可能delay或者丢失。具体有下面几种情形：

1. runloop被堵塞了。  

	timer被加入到runloop中去，如果这个runloop堵塞了，举个例子说就是处理runloop的某个resource花了10S钟，而你的timer是1s触发一次，那么这个时候因为runloop被这个10s的任务所堵塞住了，就没有可能去处理timer了，于是按照[“Thread Programming Guide”](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html)原文中的说法就是：
	> “if a timer fires when the run loop is in the middle of executing a handler routine, the timer waits until the next time through the run loop to invoke its handler routine. If the run loop is not running at all, the timer never fires.” 
	所以这种情况下，timer可能被delay甚至是丢失掉。
	
2.  runloop没有run或者run的model timer不支持。  

	加入timer加入的是defaule mode，但是这个时候用户在如[“NSRunLoop概述和原理”](http://vagase.github.com/nsrunloop-in-deep/)中第一段代码中，用的是某个用户自己定于的mode在run这个runloop那么timer的计时就没有被累加。之有当runloop的model支持该timer的时候，该timer计时才会累计。
	
所以timer只是一种非实时控制的，“粗略”地计时的一种工具，在通常我们对实时不太要求的时候timer满足我们的需求，但是如果对实时要求很高，比如游戏中，就得采取一些真正实时的手段来实现了。这里我想起了很早的时候看过的一份有戏代码，其中的动画效果都是由NSTimer来控制的，当时我就石化了，虽然游戏各种动画都还能看，但是明显不是很流畅，而且时快时慢。

结合上面说的，又会过头看看SDL中对timer的实现就实在是太简陋了，不过这种简单的timer系统有的时候反而能够提供很好的实时性。
