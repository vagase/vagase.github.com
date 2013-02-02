---
layout: post
title: NSRunLoop概述和原理
categories:
- iOS
- CFRunLoop
- Cocoa
- Cocoa消息机制
- iOS
- iOS消息机制
- NSRunloop
- 什么是NSRunLoop
status: publish
type: post
published: true
comments: true
---
##1.什么是NSRunLoop？
我们会经常看到这样的代码：  

{% codeblock lang:objc %}
- (IBAction)start:(id)sender
{
	pageStillLoading = YES;
	[NSThread detachNewThreadSelector:@selector(loadPageInBackground:)toTarget:self withObject:nil];
	[progress setHidden:NO];
	while (pageStillLoading) {
		[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
	}
	[progress setHidden:YES];
}
{% endcodeblock %}  

这段代码很神奇的，因为他会“暂停”代码运行，而且程序运行不会因为这里有一个while循环而受到影响。在[progress setHidden:NO]执行之后，整个函数想暂停了一样停在循环里面，等loadPageInBackground里面的操作都完成了以后才让[progress setHidden:YES]运行。这样做就显得简介，而且逻辑很清晰。如果你不这样做，你就需要在loadPageInBackground里面表示load完成的地方调用[progress setHidden:YES]，显得代码不紧凑而且容易出错。  

<!-- More -->

那么具体什么是NSRunLoop呢？其实NSRunLoop的本质是一个消息机制的处理模式。如果你对vc++编程有一定了解，在windows中，有一系列很重要的函数SendMessage，PostMessage，GetMessage，这些都是有关消息传递处理的API。但是在你进入到Cocoa的编程世界里面，我不知道你是不是走的太快太匆忙而忽视了这个很重要的问题，Cocoa里面就没有提及到任何关于消息处理的API，开发者从来也没有自己去关心过消息的传递过程，好像一切都是那么自然，像大自然一样自然？在Cocoa里面你再也不用去自己定义WM_COMMAD_XXX这样的宏来标识某个消息，也不用在switch-case里面去对特定的消息做特别的处理。难道是Cocoa里面就没有了消息机制？答案是否定的，只是Apple在设计消息处理的时候采用了一个更加高明的模式，那就是RunLoop。

##2. NSRunLoop工作原理
接下来看一下NSRunLoop具体的工作原理，首先是官方文档提供的说法，看图： 
 
{% img /myimages/NSRunLoop-Infrastructure.png %}  

通过所有的“消息”都被添加到了NSRunLoop中去，而在这里这些消息并分为“input source”和“Timer source” 并在循环中检查是不是有事件需要发生，如果需要那么就调用相应的函数处理。为了更清晰的解释，我们来对比VC++和iOS消息处理过程。

VC++中在一切初始化都完成之后程序就开始这样一个循环了（代码是从户sir mfc程序设计课程的slides中截取）：  
{% codeblock lang:objc %}
int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR	lpCmdLine,int	nCmdShow){
...
	while (GetMessage(&amp;msg, NULL, 0, 0)){
		if (!TranslateAccelerator(msg.hwnd, hAccelTable, &amp;msg)){
			TranslateMessage(&amp;msg);
			DispatchMessage(&amp;msg);
		}
	}
}
{% endcodeblock %}

可以看到在GetMessage之后就去分发处理消息了，而iOS中main函数中只是调用了UIApplicationMain，那么我们可以介意猜出UIApplicationMain在初始化完成之后就会进入这样一个情形：  
{% codeblock lang:objc %}
int UIApplicationMain(...){
	...
	while(running){
		[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
	}
	...
}
{% endcodeblock %}

所以在UIApplicationMain中也是同样在不断处理runloop才是的程序没有退出。刚才的我说了NSRunLoop是一种更加高明的消息处理模式，他就高明在对消息处理过程进行了更好的抽象和封装，这样才能是的你不用处理一些很琐碎很低层次的具体消息的处理，在NSRunLoop中每一个消息就被打包在input source或者是timer source中了，当需要处理的时候就直接调用其中包含的相应对象的处理函数了。所以对外部的开发人员来讲，你感受到的就是，把source/timer加入到runloop中，然后在适当的时候类似于[receiver action]这样的事情发生了。甚至很多时候，你都没有感受到整个过程前半部分，你只是感觉到了你的某个对象的某个函数调用了。比如在UIView被触摸时会用touchesBegan/touchesMoved等等函数被调用，也许你会想，“该死的，我都不知道在那里被告知有触摸消息，这些处理函数就被调用了！？”所以，消息是有的，只是runloop已经帮你做了！为了证明我的观点，我截取了一张debug touchesBegan的call stack，有图有真相：  

{% img /myimages/runloop-callstack.png %}

现在会过头来看看刚才的那个会“暂停”代码的例子，有没有更加深入的认识了呢？
