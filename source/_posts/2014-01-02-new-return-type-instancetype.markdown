---
layout: post
title: "iOS7 新的值类型 instancetype"
date: 2014-01-02 17:01:56 -0800
comments: true
categories: 
---

随着新的xcode和iOS7发布，iOS新增了一个类型instancetype. 但是它只能被用作返回值的类型，提示编译器方法的返回值的类型和调用此方法的类一致。<!--more-->
`注意：这并不是iOS7或者xcode5的特性，而是Clang中加入的`

####为什么需要instancetype

考虑下面一段代码￼￼￼￼

	NSDictionary *d = [NSArray arrayWithObjects:@(1), @(2), nil]; NSLog(@"%i", d.count);
	
很明显，这里面有个错误，但是编译器并不会提示你，不信你可以试试。这段代码甚至可以运行，因为NSDictionary和NSArray中都有count这个属性。

能够运行的原因是因为runtime的魔力，Obj-c的动态特性。runtime找到了count，因此编译器认为它是正确的。但是，如果你调用的是别的方法，那些在NSDictionary中并不存在的方法，比如objectAtIndex:，它会立刻指出问题的所在。

那么为什么编译器没有指出 +[NSArray arrayWithObjects::]返回的类型并不是NSDictionary类型的呢？请看代码：

	+ (id)arrayWithObjects:(id)firstObj, ...;
	
看到了么，返回值是id，意味着可以是任何Objective-C的对象。所以，编译器不会告诉你说“你错了！”。

那么为什么返回值要设置为id。因为这样你就可以写出你自己的子类而不回发生任何的问题。比如你创建一个继承自NSArray的子类
	
	@interface MyArray : NSArray
	@end

现在你可以这样使用你的MYArray

	MyArray *array = [MyArray arrayWithObjects:@(1), @(2), nil];

如果你的类方法的返回值不是id类型的话，你需要对子类进行类型转换。于是引入了新的关键字 *instancetype*。

如果你查看iOS7的SDK你会发现关于这个方法的申明已经变成了下面的样子：

	+ (instancetype)arrayWithObjects:(id)firstObj, ...;

不同之处就是返回值类型变得不同了，新的返回类型告诉编译器说返回值将是与调用这个方法的类相同。那么，当*NSArray*调用 *arrayWithObejects:*时，编译器就知道了它的返回值是*NSArray*类型的。如果是由*MYArray*调用的，那么返回值的类型就是*MYArray*。

那么我们最初的代码，也就是给 *NSDictionary* **d*赋值的那段代码，在使用xcode5编译的时候，它就会警告你说你错了，是不是感觉很棒？

