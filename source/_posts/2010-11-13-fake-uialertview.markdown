---
layout: post
title: 模仿UIAlertView，iOS模式对话框
categories:
- BYDialog
- iOS
- model dialog
- UIAlertView
- UIViewController
status: publish
type: post
published: true
comments: true
---
在程序UI设计上，很多情况我们会需要一种模式的对话框来进行操作。现有的iOS提供如下方式满足这种需求:

* UIAlertView
* UIViewController  : - (void)presentModalViewController:(UIViewController *)modalViewController animated:(BOOL)animated  

但是这两者明显都有不足，比如说UIAlertView官方只能显示title，message, buttons. 如果想要显示一个帐号密码输入框就不行（虽然网络上有一些程序员通过hack的方式实现了, <http://stackoverflow.com/questions/376104/uitextfield-in-uialertview-on-iphone-how-to-make-it-responsive>， 但是现在还是挺费劲的），所以想要随心所欲的在多样化模式对话框的UI界面标准的lib明显不能满足这个需求。于是我就实现了一个可以任意设置界面的模式对话框，叫 BYDialog.在给这个东西命名上，我想找一个好的前缀，但是一直没有更好的想法，所以就自恋地直接用自己的中文名的拼音开头作了前缀。

<!-- More -->

###BYDialog核心思想:

* 新建一个UIWindow，其windowLevel比UIWindowLevelStatusBar大一点，这样就可以让显示的内容完全覆盖status bar了。
* UIWindow背景透明，然后添加一个拥有灰灰背景图片的UIImageView，暂且加maskview。这样原来的内容就会有像UIAlertView一样的淡化效果，突出显示模式对话框的内容。
* 在UIWindow中添加BYDialog(是UIView的subclass)，BYDialog并显示bounce动画效果。

这样就能够像UIAlertView一现显示出来了:)

###如何自定义BYDialog用户界面：*（具体可以参考TestDialog的实现）*

* subclass BYDialog, 然后*必须*改写`- (void)loadContentView;`，并提供你自己想要显示的UI内容，注意这个内容要适当大小，不要超出屏幕大小。
* override `- (void)willPresentDialog;` `- (void)didPresentDialog;` `- (void)willDismissDialog;` `- (void)didDismissDialog;`并响应这种事件消息。
* 可以为自己的类提供各种delegate函数，根据以上捕捉到的事件消息或者其他地方，触发delegate函数调用。

所以整个UIAlertView 模仿过程完成了，效果如截图：

{% img /myimages/bydialog.png 400 400 %}

---
**Github: <https://github.com/vagase/BYDialog>**