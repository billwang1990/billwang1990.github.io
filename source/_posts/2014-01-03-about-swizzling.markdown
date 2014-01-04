---
layout: post
title: "关于swizzling"
date: 2014-01-03 20:07:36 -0800
comments: true
categories: 
---
<!--more-->

*（尊重作者劳动成果，转载请注明出处）*

今天在调试app的时候发现一个很奇怪的问题，我自定义的tableviewcell，使用了autolayout。在我的ipad（iOS7）上跑的时候一切正常。但是使用模拟器跑iOS6的系统的时候，抛出了一个异常：

	*** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Auto Layout still required after executing -layoutSubviews. UITableView's implementation of -layoutSubviews needs to call super.'
	
上网查资料，在stackoverflow上面有关于这个问题的讨论，貌似这个是iOS存在的一个bug，在iOS7已经解决了，在iOS6下使用autolayout的时候，可以通过swizzling（方法混写）来解决这个问题，本文的重点也是在swzzling上。

####什么是swizzling
简单来说，swizzling可以动态的改变方法的实现，达到修改类的行为的目的。能够实现这一点，归功于objective-c的动态语言特性。

在iOS中，所有的方法并不是在编译的时候确定下来的，而是通过send message，runtime通过方法签名去查找方法的实现，再调用实现函数。因此也就给了大家机会在运行时动态的改变方法的实现代码。

我们可以在分类中实现swizzling，这里有一个关于分类中实现swizzling时机的选择，很多人的是在分类的*+load*方法中实现的swizzling，而*《ios 6 pushing the limits》*的作者在其书中阐述了他的观点，他认为方法混写可能会引发出人意料的行为。在+load中实现意味着只要链接分类便会自动启用方法混写。这可能会导致一些很难调试的bug。
话虽如此，我还是在+load方法中实现的swizzling。
（注：+load方法是一个钩子，它会在第一次加载分类的时候执行，如果多个分类都实现了+load，那么都会被调用）

OK,上代码说事，下面就是使用代码混写的解决方式：

	+ (void)load
	{
    	static dispatch_once_t onceToken;
    	dispatch_once(&onceToken, ^{
        
        	SEL layoutSubviews = @selector(layoutSubviews);
        	SEL replaceLayoutSubviews = @selector(_autolayout_replacementLayoutSubviews);
        
     	   Method existingMethod = class_getInstanceMethod(self, @selector(layoutSubviews));
        	Method newMethod = class_getInstanceMethod(self, @selector(_autolayout_replacementLayoutSubviews));
        
        	BOOL methodAdded = class_addMethod([self class], layoutSubviews, method_getImplementation(newMethod), method_getTypeEncoding(newMethod));
        
        	if (methodAdded) {
            	class_replaceMethod([self class],
                                replaceLayoutSubviews,
                              method_getImplementation(existingMethod),
                                method_getTypeEncoding(existingMethod));
       		} else {
            	method_exchangeImplementations(existingMethod, newMethod);
        	}
        
    	});
    }
    
	- (void)_autolayout_replacementLayoutSubviews
	{
    	[super layoutSubviews];
    	[self _autolayout_replacementLayoutSubviews]; 
	    [super layoutSubviews];
	}

在+load方法中，先获取将要被代替的selector和代替它的selelctor，然后分别获取他们的Method，接着尝试把第二个方法的实现加在第一个方法的selector下，这么做的原因是防止第一个方法不存在。如果被成功加进去，那么也就是说第一个方法是空的，便把它替换了。如果bool值是NO，那么就需要将两者进行交换。

这里大家可能发现了在实现代码中有一句*[self _autolayout_replacementLayoutSubviews];*，这里并不会造成循环引用，因为在调用这个方法的时候已经通过swizzling将它交换了，这个时候你调用的是原来的实现函数。

大概总结了就这么多，如果又不对的地方，欢迎指正！
