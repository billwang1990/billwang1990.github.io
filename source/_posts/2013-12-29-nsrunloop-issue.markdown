---
layout: post
title: "关于NSRunloop的学习和理解"
date: 2013-12-29 10:49:38 -0800
comments: true
categories: 
---

####NSRunloop是iOS中比较重要的一个东西，有必要对它进行学习理解之后做一些记录：


<!-- more -->

**请尊重作者劳动成果，转载请注明出处！**

首先来看看苹果官方给出的解释：
*The NSRunLoop class declares the programmatic interface to objects that manage input sources. An NSRunLoop object processes input for sources such as mouse and keyboard events from the window system, NSPort objects, and NSConnection objects. An NSRunLoop object also processes NSTimer events.*

在程序中，每个*NSThread*对象，包括了*main thread*都会有一个自动创建的*NSRunloop*对象如果需要的话。如果你想要获取当前线程的runloop的话，只需要调用 *currentRunloop*.

每个runloop可以运行在不同的模式之下，不同的runloop mode处理其mode下包含的input sources.


查看苹果Docunment可以看到，它通过两个常量定义了两个run loop mode:
1.extern NSString* const NSDefaultRunLoopMode;
在这个模式下，将会处理除了NSConnection以外的input source.这是最常用的run loop mode。

2.extern NSString* const NSRunLoopCommonModes;
这是一个run loop mode 的合集，将input source加入之后意味着在common mode包含的所有模式下都可以处理，在Cocoa应用程序中，默认情况下Common Modes包含default modes,modal modes,event Tracking modes.

以上两个是由NSRunloop定义的，在文档中有句话，*Additional run loop modes are defined by NSConnection and NSApplication*，增加的三个mode就是下面这三个:

*NSConnectionReplyMode
这个mode表明NSConnection对象等待reply，通常不会用到。

*NSModalPanelRunLoopMode
需要等待处理的input source为modal panel时设置，比如NSSavePanel和NSOpenPanel。

*NSEventTrackingRunLoopMode
Cocoa使用该模式来处理用户界面相关的事件。

####何为Run loop 事件源
从字面翻译来看，run loop就是一个运行循环，的确它就是一个处理输入时间的运行循环，为什么需要这样处理，难道没有事件发生的时候让线程空转浪费资源？很明显在有事件发生的时候唤醒线程，没有事件发生的时候让其sleep更好。

下面我还是拿这张百看不厌的图来说事：

![alt text](https://developer.apple.com/library/mac/documentation/cocoa/conceptual/Multithreading/Art/runloop.jpg)

可以看到，runloop处理的source大体上分为两种，一种是input source 还有一种是time source.

1.Time Source.创建NSTimer添加到run loop中，这里需要注意的是，NSTimer默认是处于NSDefaultRunloopModex，这也就可以解释为什么如果你在你的控制器中添加了一个timer定时刷新你的界面，而你在拖动视图的时候timer不回fire，因为这个时候你的runloop 是NSEventTrackingRunloopMode,在这个mode下timer不回fire。

2.input sources



今天先写到这里，下次再来补充~~
