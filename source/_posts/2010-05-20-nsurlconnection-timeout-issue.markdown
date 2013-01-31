---
layout: post
title: NSURLConnection timeout失效问题
categories:
- iOS
- NSURLConnection
- NSURLMutableRequest
- NSURLRequest
- setHTTPBody
- timeout
- timeoutInterval
status: publish
type: post
published: true
comments: true
---
今天在编译3.0的程序时候无意中写了这么一段代码：
{% codeblock lang:objc %}
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:@"..." cachePolicy:NSURLRequestReloadIgnoringLocalAndRemoteCacheData timeoutInterval:15];
NSString *string = @"FUCK GFW THREE DAYS!";
NSData *bodyData = [string dataUsingEncoding:NSUTF8StringEncoding];
...
[request setValue:@"text/xml; charset=utf-8" forHTTPHeaderField:@"Content-Type"];
[request setValue:[NSString stringWithFormat:@"%u", [bodyData length]] forHTTPHeaderField:@"Content-Length"];
[request setHTTPBody: bodyData];
[request setHTTPMethod: @"POST"];
...
NSURLConnection *connection = [NSURLConnection connectionWithRequest:request delegate:self];
{% endcodeblock %}
結果由于本部龟速的网络导致程序迟迟没有反应，一直处于loading，但是很奇怪的是在15s之后程序仍然没有NSURLConnection fail之類的回调，我测试了一下程序最终在一分多钟以后才返回了，也就是说我们设置的timeout失效了。

<!-- More -->

###原因
这个问题只有在3.0以及之后的os中才有的，而且只有在当调用了setHTTPBody之后才会出现timeout失效。这个是苹果公司对URL Loading System的在OS3.0中的一个改动，不过在我看来其实这就是一个bug！在setHTTPBody之后，request的timeout会被改为240s（这个你可以通过NSLog ［request timeoutInterval］查看），苹果开发人员的解释就是通常我们自己设置的太短的timeout其实是没什么作用的，尤其对移动设备上来讲与网络沟通需要的时间往往是比较长的，假如你的timeout是10s，在WWAN的网络环境下，可能才刚刚“bring WWAN Interface up”（不知道怎么翻译，囧）。所以自从OS 3后，如果设置了HTTP body的data，系统就会自动设置一个最低的timeout值，即240s，而且这个值都是不能被改动的，即是你自己再次设置了timeoutInterval，你通过NSLog ［request timeoutInterval］得到的还是240S！！

###解决方案
这里我们就可以自己开启一个timer，然后将timer的interval设置为你想设置给connection的timeout，然后在timer的响应selector中讲connection cancel掉，这样就能够像原来一样在timeout之后cancel connection了。如果想同样像以前一样有- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error这样的回调信息，你自己在timer的selector中手动调用［delegate connection:theConnection didFailWithError: ...］;就可以了，有点tricky，但是很实用。

PS：自己可以看一下的帖子：

* <https://devforums.apple.com/thread/25282>
* <https://devforums.apple.com/message/14845#14845>
* <https://devforums.apple.com/message/37677>
