---
layout: post
title: iOS and Mac OS 调试信息输出（一）
categories:
- iOS
- Apple System Log facility
- ASL
- Debug
- NSLog
- output
- stderr
- System Log
- TN2239
status: publish
type: post
published: true
comments: true
---
##调试信息输出方法介绍
在Apple [Tech Note TN2239：iOS Debugging Magic](http://developer.apple.com/library/ios/#technotes/tn2010/tn2239.html)中提到了程序开发中Debug output 方法：

1. NSLog
2. stderr
3. system log

调试信息的输出主要有方式，一是通过输出到终端输出，二是输出到日志系统。下面讲介绍一下这几种输出调试信息的方式，首先从stderr说起。

####1. stderr （引用自TN2239）
> Many programs, and indeed many system frameworks, print debugging messages to stderr. The destination for this output is ultimately controlled by the program: it can redirect stderr to whatever destination it chooses. However, in most cases a program does not redirect stderr, so the output goes to the default destination inherited by the program from its launch environment. This is typically one of the following:  
<!-- More -->
> 1. If you launch a GUI application as it would be launched by a normal user, the system redirects any messages printed on stderr to the system log. You can view these messages using the techniques described earlier.  

> 2. If you run a program from within Xcode, you can see its stderr output in Xcode's debugger Console window (choose the Console menu item from the Run menu to see this window).  

> Attaching to a running program (using Xcode's Attach to Process menu, or the attach command in GDB) does not automatically connect the program's stderr to your GDB window. You can do this from within GDB using the trick described in the "Seeing stdout and stderr After Attaching" section of Technical Note [TN2030](http://developer.apple.com/technotes/tn/tn2030.html), 'GDB for MacsBug Veterans'.

这样一段代码在真机上跑：  

`NSLog(@"This is message from NSLog");`  
`fprintf(stderr, "This is message from stderr\n");`  

1）如果是通过Xcode调试加载运行这个程序，那么  

在xcode的console中打印如下：   
`2011-03-12 18:52:26.948 Test86[7891:307] This is message from NSLog`  
`This is message from stderr`  
在iPhone的system log中（通过Organizer的console查看）只打印:  
`Sat Mar 12 18:52:26 unknown Test86[7891] &lt;Warning&gt;: This is message from NSLog`  

2）但是如果在iPhone上通过手指触摸启动这个程序，在iPhone的system log中会打印：  
`Sat Mar 12 18:53:38 unknown Test86[7900] &lt;Warning&gt;: This is message from NSLog`  
`Sat Mar 12 18:53:38 unknown UIKitApplication:com.yourcompany.Test86[0x7d60][7900] &lt;Notice&gt;: This is message from stderr`

说明确实stderr在user 自己launch的app中被重定向为system log，而且log的等级为Notice；NSLog的等级为Warning。  


####2. system log
其实system log是unix系统都有采用syslog协议的一个日志系统（[RFC详细讲解了这种协议](http://tools.ietf.org/html/rfc5424)）。  
每条日志是有等级的，主要分为如下等级：  

* Level 0 – “Emergency”
* Level 1 – “Alert”
* Level 2 – “Critical”
* Level 3 – “Error”
* Level 4 – “Warning”
* Level 5 – “Notice”
* Level 6 – “Info”
* Level 7 – “Debug”

在创建好日志之后，通过调用API发送日志信息给一个叫做syslogd的守护进程，然后syslogd根据自己的配置文件（位于`/private/etc/syslog.conf`, [具体怎么配置这篇文章说得很详细](http://study.chyangwa.com/IT/AIX/aixcmds5/syslogd.htm)），最后讲日志保存早自己的系统日志“数据库”里面。有兴趣的可以去打卡这个syslog.conf文件看看，我Mac上的配置文件是这样的：

`*.notice;authpriv,remoteauth,ftp,install,internal.none 	/var/log/system.log`（此处省去若干字。。。）

其中`*.notice`指明了任何等级比notice高的等级都要录入到 `/var/log/system.log` 这个文件中去。

在mac os和ios那么怎样调用  API讲日志发送给系统日志呢？有两种API：

* [syslog API（不要和之前syslog协议混淆](https://developer.apple.com/library/ios/#documentation/System/Conceptual/ManPages_iPhoneOS/man3/syslog.3.html#//apple_ref/doc/man/3/syslog)
* [ASL: Apple System Log facility](https://developer.apple.com/library/ios/#documentation/System/Conceptual/ManPages_iPhoneOS/man3/asl.3.html) 是苹果自己实现的一种可以同syslogd服务器交互，用来替换syslog API的实现。

这里还有一些讲Syslog不错的文章,：

* ["Accesing the iOS system log"](http://www.cocoanetics.com/2011/03/accessing-the-ios-system-log/)  


####3. NSLog
NSLog应该是我们最熟悉的方式，其实也应该是每一个学习Objective C第一句会的语法，然后你对它真正了解多少？NSLog顾名思义，出去namespace NS 就是Log，其主要的功能就是为Cocoa程序编写人员提供一种简单的输入日志的方式。但是我们很多时候都讲其误认为是printf，而且也只是当printf用。如果是这样就太可惜了。

> NSLog is a high-level API for logging which is used extensively by Objective-C code. The exact behaviour of NSLog is surprisingly complex, and has changed significantly over time, making it beyond the scope of this document. However, it's sufficient to know that NSLog prints to stderr, or logs to the system log, or both. So, if you understand those two mechanisms, you can see anything logged via NSLog.[TN2239](http://developer.apple.com/library/ios/#technotes/tn2010/tn2239.html)

看见这个surprisingly complex也许就知道NSLog也许没那么简单了，我讲在另外一篇文章中详细讲解NSLog, [《iOS/Mac OS 调试信息输出（二）之 NSLog没那么简单》](http://vagase.github.com/2011/03/13/ios-osx-log-sys-in-deep-part2/)。
