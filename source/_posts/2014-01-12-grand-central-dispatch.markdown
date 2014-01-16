---
layout: post
title: "GCD（Grand Central Dispatch）"
date: 2014-01-12 19:12:04 -0800
comments: true
categories: 
---

###什么是GCD
Grand Central Dispatch是异步执行任务的技术之一。开发者只需要定义想执行的任务追加到Dispatch Queue中，GCD就能生成必要的线程并计划执行任务。由于线程管理是作为系统的一部分来实现的，因此可统一管理，也可执行任务，这样就比以前的线程更有效率。

<!--more-->
###两种Dispatch Queue
1、Serial Dispatch Queue
串行执行

2、Concurrent Dispatch Queue
并行执行，但并行执行的处理数量取决于当前系统的状态

###如何获取Dispatch Queue
1. 通过dispatch_queue_create函数生成。
	可以生成任意多个queue，当生成多个serail queue的时候，多个serail queue将并行执行。虽然可以生成多个串行的queue来并行操作，但是系统对于一个serail queue就只生成一个线程，如果过多使用多线程，就会消耗大量内存，引起大量上下文切换，大幅度降低系统的响应性能。
	另外，生成的queue必须由程序员手动释放，使用dispatch_release函数。
	另外有下面一段代码需要注意：
	
		dispatch_queue_t myQueue = dispatch_queue_create(nil, DISPATCH_QUEUE_CONCURRENT);
		dispatch_async(queue, ^{NSLog(@"block on my queue");});
		dispatch_release(myQueue);
	
	在以上代码中，将block追加到queue，并立即release掉，这样做完全没有问题，在dispatch_async函数中追加block到queue，该block通过dispatch_retain函数持有queue，block结束时，会通过dispatch_release释放掉该block持有的queue。
	
2. 获取系统标准提供的Dispatch Queue

	Main Dispatch Queue(serail queue)和Global Dispatch Queue(Concurrent)
	
	另外Global Dispatch Queue有四个优先级。High Priority, Default Priority, Low Priority, Background Priority。
	对Main Dispatch Queue和Global Dispatch Queue执行release和retain没有任何变化
	
	
###相关操作
1. dispatch_set_target_queue函数

	dispatch_queue_create生成的queue都使用与默认优先级Global Disptch Queue相同执行优先级的线程，可以通过此函数变更优先级。
	
2. dispatch_after函数

	**它并不是在指定的时间后执行处理，而是在指定的事件后追加处理到Dispatch Queue。**

3. Dispatch Group

	可以实现追加到Dispatch Queue的多个处理全部结束后才追加某个操作。

4. dispatch_barrier_async函数

	它会等待追加到Concurrent Dispatch Queue上的并行执行的处理全部结束后，再将指定的处理追加到该queue中。
	
	
5. dispatch_sync
	
	和dispatch_async相反，此函数会等待追加的block执行操作直到其完成。
	
	
###Dispatch Source
	
关于这一部分，由于本人没真正使用过，所以简单的做个介绍。

GCD中除了Disptch Queue以外，还有个Dispatch Source，它是BSD系统内核惯有功能kqueue的包装。

kqueue是在XNU内核中发生各种事件时，在应用程序编程方执行处理的技术。其CPU负荷相当小，尽量不占用资源。



（尊重作者劳动成功，转载请注明出处）



	
	