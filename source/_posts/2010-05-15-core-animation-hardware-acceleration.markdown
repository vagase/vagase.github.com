---
layout: post
title: Core Animation硬件加速切身体验
categories: 
- iOS
- CALayer
- Core Animation
- UIImageView
- 播放器
- Media player
status: publish
type: post
published: true
comments: true
---
一直以来对Core Animation的理解都只是停留在这个技术是用来做动画的。通过CA为什么会得到流畅的动画效果，那是因为Core Animation是提供硬件加速的。然后，就没有然后了。

因为毕设的一部分是做一个iPhone的视频播放器，当然渲染解码出来的数据就是一个问题了。最先用很粗糙的UIImageView通过setImage的方式来呈现每一帧的数据。额的个天啊，虽然视频是播放出来了，但是明显有很卡的感觉！而且如果UIImageView越大，那么越卡，也就是渲染一张图片的时间越长。于是就想到了最坏的打算：用OPENGL来渲染。但是本人一向对非navtive sdk的开发比较反感，因为和系统打交道比较麻烦，所以这只是最坏的打算。于是有投奔CALayer，通过CAlayer的contents来渲染每一帧，结果效果大让人满意，渲染每一帧所花的时间基本上是同UIImageView所需时间的**60%**左右！播放那是非常流畅！

<!-- More -->

总结，Core Animation有硬件加速支持，所以通过CALayer的contents来渲染帧数据就会得到硬件加速带来的好处。所以如果您以后有同样要高效播放视频或者是多张图片（往往在做动画效果的时候需要）的时候，如果UIImageView不能满足你的需求，就用CALayer吧！
