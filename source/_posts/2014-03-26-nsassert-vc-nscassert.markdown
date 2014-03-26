---
layout: post
title: "小心NSAssert"
date: 2014-03-26 22:28:30 +0800
comments: true
categories: 
---

今天在看Facebook的[KVOController](https://github.com/facebook/KVOController)源码的时候，里面除了使用 NSAssert以外 还使用了 NSCAssert这个更少人用的。 就想起了之前遇到的一个由它引发的问题，顺便记录回忆下。

在苹果的SDK中可以看到这两个都是定义的宏

NSAssert 的定义如下：

	#define NSAssert(condition, desc, ...)	\
    do {				\
	__PRAGMA_PUSH_NO_EXTRA_ARG_WARNINGS \
	if (!(condition)) {		\
	    [[NSAssertionHandler currentHandler] handleFailureInMethod:_cmd \
		object:self file:[NSString stringWithUTF8String:__FILE__] \
	    	lineNumber:__LINE__ description:(desc), ##__VA_ARGS__]; \
	}				\
        __PRAGMA_POP_NO_EXTRA_ARG_WARNINGS \
    } while(0)
	#endif
	
	
NSCassert的定义如下：

	#define NSCAssert(condition, desc, ...) \
    do {				\
	__PRAGMA_PUSH_NO_EXTRA_ARG_WARNINGS \
	if (!(condition)) {		\
	    [[NSAssertionHandler currentHandler] handleFailureInFunction:[NSString stringWithUTF8String:__PRETTY_FUNCTION__] \
		file:[NSString stringWithUTF8String:__FILE__] \
	    	lineNumber:__LINE__ description:(desc), ##__VA_ARGS__]; \
	}				\
        __PRAGMA_POP_NO_EXTRA_ARG_WARNINGS \
    } while(0)
	#endif
	
之所以要说小心使用NSAssert是因为大家可以看到它的定义中出现了一个*self*, 那么有可能在你的block中你会发现你明明没有self的strong引用，但是仍然出现了循环引用就看看你是否使用了NSAssert这样的小东西，有可能稍微疏忽用了它，这个宏被展开之后就持有了self，那么有可能就会出现让你百思不得其解的问题。
	
	
