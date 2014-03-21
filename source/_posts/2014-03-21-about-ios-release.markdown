---
layout: post
title: "ios 中内存释放顺序"
date: 2014-03-21 22:59:53 +0800
comments: true
categories: 
---


今天的工作中遇到一个bug（非ARC），就是访问了一个被释放的对象。我找了好久查看是否被意外release之类的原因，都没有找到结果。后来我突然想到了一个点，是否是对象释放的顺序造成的。

我的程序中大概是这样的，viewcontroller的view中添加了一个tableview，然后tableview包含了一些subview。

我猜测是因为tableview持有那些subview的引用计数，那么如果我先释放tableview的话，tableview对subview的引用也就随之消失，那么就可能导致subview被提前释放，但是这个时候原本指向subview的那些指针并没有被置为nil，还是指向原来的地址，那么这个时候再进行release就会crash。

后来为了证明我得猜测，我修改了release的顺序，这个时候就没有再crash了。

另外[super dealloc]一定要写在最后，有点晚了，原因自己google吧，哈哈~~