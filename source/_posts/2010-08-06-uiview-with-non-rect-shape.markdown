---
layout: post
title: UIView创建不规则视图
categories:
- iOS
- UIView
- 不规则
- regular
status: publish
type: post
published: true
comments: true
---
总所周知，UIView都是方形的，并不能产生一个不规则的形状的view（小弟才疏学浅，目前确实没有发现能够创建真实不规则视图的方法，如果有，不吝赐教，感激涕零！）。为什么我们有创建不规则view的需求？如果只是为了在View上显示不规则图形那大可不必，直接讲不规则图形添加到view上，然后讲view的backgroundColor设置为UIColor clearColor就可以；但是之所以有这样的需求，很大部分就是为了判断不规则的图形去响应触摸事件，判断图形是否被触摸选中了这样的要求，我们最直接的想法就是每个不规则图形都是一个view，那么图形是否选中就可以通过UIResponder的那一系列触摸有关的响应函数得知了，所以这个时候我们就需要不规则的view。但是显示的杯具是，iOS并没有提供这样的不规则view，如果要完成刚才的需求，就只能手动判断触摸的点是否在不规则图形里面了，这有的时候将是一件比较痛苦的事情。那么现在提供一种“创建不规则view”的解决方案：

不知道大家有没有注意到UIView有这样一个函数：`- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event` 这个函数就是用来判断指定的点是否在View内，我们方法就是在这个函数中，如果point在指定的不规则图形内返回YES，反之就返回NO。这样不规则图形的bounds就相当于代表了view自己的bounds。这样当你触摸view的时候，当且只有当触摸到指定图形内才会使得view被触摸到，才会调用到UIResponder一系列触摸响应事件。

<!-- More -->

代码如下：

**RoundView.h**
{% codeblock lang:objc %}
@interface MyView : UIView {
  UIBezierPath *_path;
}

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event;

@end
{% endcodeblock %}

**RoundView.m**
{% codeblock lang:objc %}
// Print current selector's name
#define PRINT_CURRENT_SEL NSLog(@"&lt;%@&gt; %@", NSStringFromClass([self class]),NSStringFromSelector(_cmd))

//
// PrivateMethods
// This category provide private apis for RoundView class
//
@interface RoundView(PrivateMethods)

// Initialize all view's state
- (void)_init;

// Highlight view's border, better visual effect for testing.
- (void) _highlightBorder;

@end
@implementation RoundView(PrivateMethods)

- (void)_init{
  [self _highlightBorder];
  self.backgroundColor = [UIColor clearColor];

  // Create path object as round rect
  _path = [[UIBezierPath bezierPathWithRoundedRect:CGRectInset(self.bounds,  20, 20) cornerRadius:60] retain];
}

- (void) _highlightBorder{
  CALayer *theLayer = [self layer];
  theLayer.borderColor = [UIColor blueColor].CGColor;
  theLayer.borderWidth = 2;
}

@end

#pragma mark -

@implementation RoundView

- (id)initWithCoder:(NSCoder *)aDecoder{
  if((self = [super initWithCoder:aDecoder])){
    [self _init];
  }
  return self;
}

- (id)initWithFrame:(CGRect)frame {
  if ((self = [super initWithFrame:frame])) {
    [self _init];
  }
  return self;
}

- (void)drawRect:(CGRect)rect {
  [[UIColor redColor] setFill];
  [_path fill];
}

- (void)dealloc {
  [_path release];

  [super dealloc];
}

- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{
  return [_path containsPoint:point];
}

#pragma mark -
#pragma mark UIResponder touches

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
  PRINT_CURRENT_SEL;
}
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event{
  PRINT_CURRENT_SEL;
}
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event{
  PRINT_CURRENT_SEL;
}
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event{
  PRINT_CURRENT_SEL;
}
@end
{% endcodeblock %}

这个demo中用到了3.2以后提供的UIBezierPath类，来创建一个图形的path，在pointInside:withEvent:中判断点是不是在图形的里面，如果是返回YES，反之NO。这样就用找个图形的path代表了view的bounds。那么即使你触摸在view内但是没有在找个图形上，touchesBegan就不会被调用，不会打印相应信息在console中。
所以这样这个view的边界就变成了这个不规则图形的边界了，就变相地创建出了一个不规则的view了！cheers！