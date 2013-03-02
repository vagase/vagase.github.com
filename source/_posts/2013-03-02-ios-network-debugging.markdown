---
layout: post
title: "iOS Network Debugging"
date: 2013-03-02 13:35
comments: true
categories:
- iOS
- Networking 
- Debug
- RVI
- PonyDebugger 
- Charles

---
鉴于用Instruments测试Network比较废柴，网络上不断涌现出更加优秀的解决方案，汇总如下：
<!--More-->
1. Proxy  
Mac插网线上网，然后通过Wifi共享网络给iPhone，在Mac上可以用任何流量监控工具查看iPhone的网络流量。如果查看HTTP协议流量，个人比较喜欢[HttpScoop](http://www.tuffcode.com/)；如果还需要查看HTTPS协议，可以使用[CharlesProxy]("http://www.charlesproxy.com/")，这里有篇很好的教程：[Using Charles Proxy to examine iOS apps](http://blog.cloudfour.com/using-charles-proxy-to-examine-ios-apps/)

2. [RVI(Remote Virtual Interface)](http://blog.manbolo.com/2013/02/22/analysing-ios-app-network-performances-on-cellularwifi)  
用RVI的好处就是可以测试iPhone在蜂窝移动数据下的网络情况。

3. [PonyDebugger](http://corner.squareup.com/2012/08/ponydebugger-remote-debugging.html)  
Square出品，通过浏览器可以测试设备的网络、CoreData、View Hierarchy情况，但是需要在Code中安装PonyDebugger的lib。

4. [iOS 6 Network Link Conditioner](http://jeffreysambells.com/2012/09/22/network-link-conditioner)  
虽然地球人都知道，但是这玩意儿确实没多大用。

最后，推荐订阅:[iOS Dev Weekly](http://iosdevweekly.com/)
