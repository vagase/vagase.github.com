---
layout: post
title: 用NSZombieEnabled处理EXC_BAD_ACCESS
categories:
- iOS
- EXC_BAD_ACCESS
- Instruments
- NSZombieEnabled
status: publish
type: post
published: true
comments: true
---
一直有件很头痛的事情就是在debug程序的时候经常出现程序crash然后在console就只是打印了EXC_BAD_ACCESS, 完全不知道问题具体出在哪里。这种情况一看就知道是对象指针出了问题，很大部分时候都是因为再次使用了一个已经完全dealloc的object。对于console的这种不负责任的报错，大家都表示纷纷不给力。其实只要你google一下EXC_BAD_ACCESS，就会得到很多很多的解决方案，这里我搜集了很多我认为讲得不错的文章和大家分享。

* CocoaDev，个人觉得讲Cocoa技术十分专业的网站之一，下面的链接详细讲了讲NSZombieEnable的原理。
	<http://www.cocoadev.com/index.pl?NSZombieEnabled>
* 苹果官方的Mac OS X Debugging Magic, 详细讲述了最为一个高级苹果程序员应该具备的调试技巧。 
	<http://developer.apple.com/library/mac/#technotes/tn2004/tn2124.html>
* 其实还可以在Instruments中开启NSZombie选项，这样就可以在Instruments中直接查看crash时候的call stack了。 
	<http://www.markj.net/iphone-memory-debug-nszombie/>

最后提醒NSZombieEnabled只能在调试的时候使用，千万不要忘记在产品发布的时候去掉，因为NSZombieEnabled不会真正去释放dealloc对象的内存，一直开启后果可想而知，自重！
