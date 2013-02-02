---
layout: post
title: NSLog warning:"format not a string literal and no format arguments"
categories:
- iOS
- NSLog
- warning
- Xcode
status: publish
type: post
published: true
comments: true
---
我觉得这个问题已经烦躁了我很久很久，因为自打我学习Objective c的时候，写这样一句是没有任何问题的：
{% codeblock lang:objc %}
NSString *name = @"Mimi";
NSLog(name);
{% endcodeblock %}

而如今升级到SnowLeopard，用上Xcode 3.2之后，如果这么写总是会有一个“format not a string literal and no format arguments”另人烦躁的warning, 虽然比较恶心，但是还是看看为什么会有这个warning再说。

其实一切都是为了安全着想，加入你写了下面的一段程序：

{% codeblock lang:objc %}
NSString *nameFormat = @"%@ %@";
NSString *firstName = @"Jon";
NSString *lastName = @"Hess %@";
NSString *name = [NSString stringWithFormat:nameFormat, firstName, lastName];
NSLog(name)；
{% endcodeblock %}

那么就相当于  
`NSLog(@"Jon Hess %@");`

这样自然程序运行是有问题的！

<!-- More -->

所以为了避免这么无辜而且隐蔽的错误，xcode添加了类型检查。

但是如果你觉得你不需要xcode为你操这些心，方便才是王道的话，你可以在xcode里面将GCC 4.2-Warnings中的Typecheck Calls to printf/scanf选项去掉就可以解决问题：

{% img /myimages/nslog_type_check.png 400 400 %}
