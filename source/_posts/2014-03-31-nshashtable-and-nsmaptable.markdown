---
layout: post
title: "NSHashTable &amp; NSMapTable"
date: 2014-03-31 13:40:38 +0800
comments: true
categories: 
---

在ios开发中大家用到更多的集合类可能是像NSSet或者NSDictionary，NSArray这样的。这里要介绍的是更少人使用的两个类，一个是NSMapTable，另一个是NSHashTable。
<!--more-->

####NSHashTable

NSHashTable看上去就像NSSet的替代品，对比NSSet/NSMutableSet，NSHashTable有以下的特性：

* NSSet/NSMutableSet 拥有所包含对象的strong refrence。
* NSHashTable是mutable可变的，没有不可变的类。
* NSHashTable可以拥有成员的weak refrence；
* NSHashTable可以在输入的时候copy成员。
* NSHashTable可以包含任意的指针，可以使用指针去比较或者hash校验。

NSHashTable在初始化的时候可以传入一个参数option，有以下几种选项

* NSHashTableStrongMemory: Equal to NSPointerFunctionsStrongMemory. This is the default behavior, equivalent to NSSet member storage.
* NSHashTableWeakMemory: Equal to NSPointerFunctionsWeakMemory. Uses weak read and write barriers. Using NSPointerFunctionsWeakMemory, object references will turn to NULL on last release.
* NSHashTableZeroingWeakMemory: This option has been deprecated. Instead use the NSHashTableWeakMemory option.
* NSHashTableCopyIn: Use the memory acquire function to allocate and copy items on input (see NSPointerFunction -acquireFunction). Equal to NSPointerFunctionsCopyIn.
* NSHashTableObjectPointerPersonality: Use shifted pointer for the hash value and direct comparison to determine equality; use the description method for a description. Equal to NSPointerFunctionsObjectPointerPersonality.

####NSMapTable
NSMapTable和NSDictionary有点类似，和dictionary相比，它有以下的特性：

* NSDictionary和NSMutableDictionary会copy keys(这也是导致他们构造的时候性能相对低一点的原因)，还会持有object的strong引用。
* NSMapTable也都是mutable的，没有不可变类型。
* NSMapTable也可以在input的时候copy它的值。
* NSMapTable能包含任意指针，用指针来进行比较或者hash校验。

类似的NSMapTable也有一个option参数来初始化，有以下几种选项

* NSMapTableStrongMemory: Specifies a strong reference from the map table to its contents.
* NSMapTableWeakMemory: Uses weak read and write barriers appropriate for ARC or GC. Using NSPointerFunctionsWeakMemory, object references will turn to NULL on last release. Equal to NSMapTableZeroingWeakMemory.
* NSHashTableZeroingWeakMemory: This option has been superseded by the NSMapTableWeakMemory option.
* NSMapTableCopyIn: Use the memory acquire function to allocate and copy items on input (see acquireFunction (see NSPointerFunction -acquireFunction). Equal to NSPointerFunctionsCopyIn.
* NSMapTableObjectPointerPersonality: Use shifted pointer hash and direct equality, object description. Equal to NSPointerFunctionsObjectPointerPersonality.



这里有一些关于集合类的参考链接：

* [http://nshipster.com/nshashtable-and-nsmaptable/](http://nshipster.com/nshashtable-and-nsmaptable/)
* [http://www.cocoawithlove.com/2008/08/nsarray-or-nsset-nsdictionary-or.html](http://www.cocoawithlove.com/2008/08/nsarray-or-nsset-nsdictionary-or.html)
* [http://www.cocoawithlove.com/2008/07/nsmaptable-more-than-nsdictionary-for.html](http://www.cocoawithlove.com/2008/07/nsmaptable-more-than-nsdictionary-for.html)