---
layout: post
title: iOS IO 重定向（NSLog to UITextView）
categories:
- iOS
- dup2
- io 重定向
- NSFileHandle
- NSLog
- NSPipe
- uitextview
- Xcode
- 管道
- 重定向
status: publish
type: post
published: true
comments: true
---
###情形描述:

在调试程序的时候，通过NSLog打印log，很方便的就可以在Xcode里面看到。但是程序一旦“离开XCode运行”， 比如将App交付给了公司的测试团队，怎样能够很随意看到NSLog打印的信息呢？通常在离开xcode之后，NSLog的信息会保存在Systemlog里面（[这里有NSLog详细描述](http://octopress.dev/ios-osx-log-sys-in-deep-part1/)），你可以通过一定办法取出这个log。甚至可以写一套日志系统，然后将这些信息保存到日志中，然后导出或者上传自己的服务器。但是这些太麻烦了，简直是弱爆鸟。我们的目的是：在App里面能够直接像xCode console窗口那样显示NSLog的信息，准确的说是标准输出的信息。

<!-- More -->

###关键技术：IO重定向

通过IO重定向，我们可以直接“截取” stdout,stderr等标准输出的信息（NSLog->stderr），然后再在自己的View上显示出来。

1. 通过NSPipe创建一个管道（[这里有详细讲NSPipe的文章](http://www.cocoadev.com/index.pl?NSPipe)），pipe有读端和写端.
2. 通过dup2（[这里有详细将dup2的文章<](http://developer.apple.com/library/IOs/#documentation/System/Conceptual/ManPages_iPhoneOS/man2/dup2.2.html)）讲标准输入重定向到pipe的写端。
3. 通过NSFileHandle监听pipe的读端，然后讲读出的信息显示在uitextview上。

####相关代码：

{% codeblock lang:objc %}
@implement TestAppDelegate

- (void)redirectNotificationHandle:(NSNotification *)nf{
  NSData *data = [[nf userInfo] objectForKey:NSFileHandleNotificationDataItem];
  NSString *str = [[[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding] autorelease];

  self.logTextView.text = [NSString stringWithFormat:@"%@\n%@",self.logTextView.text, str];
  NSRange range;
  range.location = [self.logTextView.text length] - 1;
  range.length = 0;
  [self.logTextView scrollRangeToVisible:range];

  [[nf object] readInBackgroundAndNotify];
}

- (void)redirectSTD:(int )fd{
  NSPipe * pipe = [NSPipe pipe] ;
  NSFileHandle *pipeReadHandle = [pipe fileHandleForReading] ;
  dup2([[pipe fileHandleForWriting] fileDescriptor], fd) ;

  [[NSNotificationCenter defaultCenter] addObserver:self
                                           selector:@selector(redirectNotificationHandle:)
                                               name:NSFileHandleReadCompletionNotification
                                             object:pipeReadHandle] ;
  [pipeReadHandle readInBackgroundAndNotify];
}
- (BOOL)application:(UIApplication *)application 
didFinishLaunchingWithOptions:(NSDictionary *)launchOption{

  [self redirectSTD:STDOUT_FILENO];
  [self redirectSTD:STDERR_FILENO];

//YOUR CODE HERE...
}

//YOUR CODE HERE...

@end

{% endcodeblock %}

一旦通过printf或者NSLog写数据，因为重定向过，这些数据都会写到Pipe的写端。同时pipe会将这些数据从写端直接传送到读端。读端通过NSFileHandle的“监控”函数取出这些数据，并最终显示在uitextview上。

####截图：
{% img /myimages/io-redirect.png 300 400 %}
