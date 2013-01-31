---
layout: post
title: NSManagedObject property没有实例变量的模拟
categories:
- iOS
- instance variable
- NSManagedObject
- property
- 没有实例变量
status: publish
type: post
published: true
comments: true
---
趁现在还记忆犹新，而且还小有兴奋，把自己怎么实现RT话题写下来。

这个问题其实是在我学习CoreData的时候就一直存在，在CoreData里面有一个“很酷”的类NSManagedObject，只要是找个类的子类，那么这些类的property只要申明并且在.m里面申明为dynamic就可以，而不需要为property指定特定的实力变量，用起来特别方便。这让我这个经常实用property的人觉得真的是匪夷所思啊。一直在想找个玩意儿到底是采用了什么魔法，才能够导致property和实力变量之间的依赖可以剥离。而且前段时间和Kevin同学一起讨论了一下，但是都表示没有头绪。

最近又刚好温习了一下 ["Objective-C Runtime Programming Guide"](http://developer.apple.com/mac/library/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)这才恍然大悟，于是花了一天的时间就实现出了类似的效果。并且通过这次实践，对动态语言到底有多动态又有了从来没有过的深入认识，这种感觉很过瘾。

<!-- More -->

具体是怎么实现的不想多讲，原理不难，过程也不复杂，自己看源代码去，但是要求对oc runtime要有所了解。最终的效果就是我实现一个叫SmartObject 的类，只要是从找个类继承下来的类就可以想NSManagedObject的子类那样不用给出实力变量就可以使用property了！具体使用如下：

{% codeblock lang:objc %}
#import "SmartObject.h"
@interface TestClass : SmartObject {
}
@property (nonatomic, retain) NSString *title;
@property (nonatomic, retain) NSString *location;
@property (nonatomic, retain) NSDate *date;
@end
{% endcodeblock %}

{% codeblock lang:objc %}
#import "TestClass.h"
@implementation TestClass
@dynamic title;
@dynamic location;
@dynamic date;
@end
{% endcodeblock %}

结果就可以这样使用了：

{% codeblock lang:objc %}
TestClass *_myObject;
...
_myObject = [[TestClass alloc] init];
_myObject.title = @"this is title";
_myObject.location = @"this is location";
_myObject.date = [NSDate date];
...
NSLog(_myObject.title, nil);
NSLog(_myObject.location, nil);
NSLog([_myObject.date description], nil);
{% endcodeblock %}

感觉是不是很酷啊～

---
**Github: <https://github.com/vagase/SmartObject>**
