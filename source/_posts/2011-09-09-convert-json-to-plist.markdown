---
layout: post
title: jtop - Convert JSON to Plist
categories:
- Mac OS X
- convert
- convert json to plist
- json
- jtop
- plist
- tool
- 工具
status: publish
type: post
published: true
comments: true
---

JSON格式很好，但是可阅读性相对较差，而且在mac下也没有很好的专门针对JSON的编辑器，一般都用文本编辑器，所以看起来非常累。

恰好，mac下的plist文件格式如果用系统自带的properties list editor（现在整合到Xcode4中了）打开阅读性非常好。于是自己写了一个小工具将JSON转换为plist文件输出。这样看起来就非常爽了～

两者阅读效果对比截图如下：

{% img /myimages/jtop.png 400 300 %}

Github地址：[https://github.com/vagase/jtop](https://github.com/vagase/jtop)