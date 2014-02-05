---
layout: post
title: "[译]NSNotificationCenter with Block considered harmful"
date: 2014-02-05 10:00:00 +0800
comments: true
categories: 
---


本文是这篇[原文](http://sealedabstract.com/code/nsnotificationcenter-with-blocks-considered-harmful/)的翻译，初次翻译，水平有限，望指正，同时也请尊重本人的劳动成果，转载请注明出处。

(注：有些地方翻译的感觉不好，贴上了原文。)

在过去，我们想要注册接受一个notification通常使用如下的方法：

	-[NSNotificationCenter  addObserver:selector:name:object:]

换句话说，就是target-action模式。当收到notification的时候，就会调用相应的selector。

<!--more-->

从iOS4开始，block被加入到iOS中。将任何东西添加到block是一件很酷的事情。

基于block的添加到NSNotificationCenter得代码如下：

	-[NSNotificationCenter addObserverForName:object:queue:usingBlock:]

这是一个糟糕的想法。对于这，我认为这可能是iOS API设计中最大的一个错误。我为这个api 调试过不下10次，浪费了我至少4个星期的时间。

####So how bad could it be?

在写博文的过程中，我写了一些TDD代码，让我们先来看看：

	- (void)testExample
	{
    	for(int i =0; i < 5; i++) {
        	YourAttempt *attempt1 = [[YourAttempt alloc] init];
        	[[NSNotificationCenter defaultCenter] postNotificationName:notificationName object:nil];
       		XCTAssertEqual(counter, i+1, @"Unexpected value for counter.");
        	XCTAssertEqual(1, attempt1.localCounter, @"Unexpected value for localCounter.");
    	}
	}
	
这是一个很简单的测试：

*	我们创建了一个Attempt对象。
*	发送一个notification。
*	检查notification增加的全局变量counter。
*	检查notification增加的对象内部变量 localCounter。 

到现在你可能会说：“这看起来太简单了”。如果你是这样的，那就跳过博客，从GitHub上pull down[代码仓库](https://github.com/drewcrawford/NSNoficationCenter_With_Blocks)，在YourAttempt.m中输入你的解决方案，然后按下 Command+U。不用担心，直到你真的确信它是对的，我会等你的结果。如果你一开始就失败的话那会更加有趣。

继续阅读？你可真懒。让我来完成接下来的工作。

####If at first you don’t succeed

我们的第一个Attempt非常简单：

	@interface Attempt1() {
	}
	@end
	@implementation Attempt1
	-(id)init {
	    if (self = [super init]) {
        	[[NSNotificationCenter defaultCenter] addObserverForName:notificationName object:nil queue:nil usingBlock:^(NSNotification *note) {
    	        int oldCounterValue = counter;
	            counter++;
            	self.localCounter++;
            	NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee violated.");
        	}];
    	}
    	return self;
	}	
	@end
	
下面是结果：

	"3" is not equal to "2" - Unexpected value for counter.
	"6" is not equal to "3" - Unexpected value for counter.
	"10" is not equal to "4" - Unexpected value for counter.
	"15" is not equal to "5" - Unexpected value for counter.

你能指出这里是怎么回事么，我们希望看到counter是1，2，3，4，5，但是确得到了1，3，6，10，15，为什么是这些数字？

这些数列被称为[triangular numbers](http://en.wikipedia.org/wiki/Triangular_number).
我们第一次发送notification的时候，它运行一次。counter是1。在第二次它运行了两次，所以counter是3。在第三次代码运行了三次所以counter为6。

现在你可能会说：“为什么要运行那么多次，我不会傻到用全局变量”。额，如果你使用camera，microphone或者在你app delegate中得任何东西，你实际上都在使用一个全局变量。稍微忘掉这一下——想象如果我们随机从您的代码库选了一个函数跑它两次而不是一次会发生什么。我们可能插入两个对象到你的数据库中，或者删除一个对象两次，我们可能pop一个已经失效的viewcontroller，我们可能重复你得网上支付过程，这又有谁知道呢？

事实上，这是一件非常危险的事情因为它可能导致任何情况的发生。这个bug可能会导致任何bug。这太糟糕了。这也是我花了很多时间在这个bug的原因，让我来给你看一些真实的bug报告：

“Whenever I try to take a picture, the lens doesn’t open.”

“If I go to Screen A, leave it, and come back, the button on Screen B does something really strange.”

“After I pick a photo from my photo library, the app works fine. For about 20 seconds. Then it crashes. But I can only reproduce this once per testing session. I have to wait until tomorrow to catch it again.”


这些听起来像是notification的bug么？不像吧，这也是为什么它如此可怕的原因。

那么：让我们不要接受notification两次。很明显，我们忘记了unregister notification。让我们动手吧。


####A very selfish attempt


	@interface Attempt2() {
    	id cleanupObj;
	}
	@end
	@implementation Attempt2
	-(id)init {
    	if (self = [super init]) {
        	cleanupObj = [[NSNotificationCenter defaultCenter] 	addObserverForName:notificationName object:nil queue:nil 	usingBlock:^(NSNotification *note) {
    	        int oldCounterValue = counter;
        	    counter++;
            	self.localCounter++;
	            NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee 	violated.");
    	    }];
    	}
    	return self;
	}
	- (void)dealloc {
    	[[NSNotificationCenter defaultCenter] removeObserver:cleanupObj];
	}
	@end
	
接着运行，结果是：

	"3" is not equal to "2" - Unexpected value for counter.
	"6" is not equal to "3" - Unexpected value for counter.
	"10" is not equal to "4" - Unexpected value for counter.
	"15" is not equal to "5" - Unexpected value for counter.

为什么又得到了相同的结果，这里发生了什么？

OK，和上次的结果一样，尽管我们在dealloc中将Attempt从notification中移除了，但是仍然收到了通知。为什么。是我们语法有错误还是别的什么原因？

不，语法没有错误，错的是dealloc从来没有被调用，为什么没有呢？

当你申明一个block的时候，编译器将会对block的行为进行检查。这是因为比如你写了类似的代码 `id x = @(42)`；然后申明了一个block用到了x，block会延长x的生命周期。那么x需要在blcok执行的时候一直存在。

在这里罪魁祸首的就是block中有如下的表达式：

	self.localCounter++;

和下面的代码是等价的：

	[self setLocalCounter:[self localCounter]+1];
	
这里有了对self的两次引用。所以只要申明了block，就获得了对self的引用，因为block需要self来运行。然后又因为NSNotificationCenter有block的引用计数，block又引用self，所以self不会被dealloced。

嘿，你想知道还有什么可怕的么？这段代码非常干净，不是从编译器中窥看的，不是从Clang Static Analyzer中窥看到得。事实上，在本博文中你看到的所有代码都很干净。（原文：This code builds cleanly. Not a peep from the compiler; not a peep from Clang Static Analyzer. In fact, every buggy code listing you see in this post gets a clean bill of health from both.）事实上，LLVM将会给你警告，你可能会看到：

Capturing self strongly in this block is likely to lead to a retain cycle

Clang目前不够强大来找到类似的bug。

那么解决方案很简单：仅仅需要在block中移除self的引用计数。


####Practicing selflessness

	@interface Attempt3() {
    	id cleanupObj;
	}
	@end
	@implementation Attempt3
	-(id)init {
    	if (self = [super init]) {
        	cleanupObj = [[NSNotificationCenter defaultCenter] addObserverForName:notificationName object:nil queue:nil usingBlock:^(NSNotification *note) {
            	int oldCounterValue = counter;
	            counter++;
    	        _localCounter++;
        	    NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee violated.");
	        }];
    	}
	    return self;
	}
	
	- (void)dealloc {
    	[[NSNotificationCenter defaultCenter] removeObserver:cleanupObj];
	}
	@end	
	
	
然后：

	"3" is not equal to "2" - Unexpected value for counter.
	"6" is not equal to "3" - Unexpected value for counter.
	"10" is not equal to "4" - Unexpected value for counter.
	"15" is not equal to "5" - Unexpected value for counter.

还是一样？

对，是一样的。这实际上和之前是同样的问题，它不过被隐藏起来了。如你所见，我们持有了_localCounter实例变量的同时保留了self。

文档中这样说的：

*When a block is copied, it creates strong references to object variables used within the block. If you use a block within the implementation of a method [and] you access an instance variable by reference, a strong reference is made to self*

文档接着还建议说

*To override this behavior for a particular object variable, you can mark it with the __block storage type modifier.*

那么这很简单，我们仅仅需要用__block来修饰_localCounter。

####It’s a __block party

	@interface Attempt4() {
    	id cleanupObj;
	    __block int _localCounter;
	}
	@end
	@implementation Attempt4
	-(id)init {
    	if (self = [super init]) {
        	cleanupObj = [[NSNotificationCenter defaultCenter] addObserverForName:notificationName object:nil queue:nil usingBlock:^(NSNotification *note) {
            	int oldCounterValue = counter;
	            counter++;
    	        _localCounter++;
        	    NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee violated.");
	        }];
    	}
	    return self;
	}
	- (void)dealloc {
    	[[NSNotificationCenter defaultCenter] removeObserver:cleanupObj];
	}
	@end

这有多么糟糕：


	"3" is not equal to "2" - Unexpected value for counter.
	"6" is not equal to "3" - Unexpected value for counter.
	"10" is not equal to "4" - Unexpected value for counter.
	"15" is not equal to "5" - Unexpected value for counter.

嗯，OK，你在误导我，Apple。你告诉我说这样做可以解决问题，但是结果呢？

文档来来回回想要谈论的是对象变量，相反的，而不是别的类型，请看文档（原文：What gives is that this documentation flits back and forth between whether or not it’s talking about an object variable, as opposed to, I guess, the other kind. See）：

`it creates strong references to object variables used within the block… If you access an instance variable by reference, a strong reference is made to self;… To override this behavior for a particular object variable, you can mark it with the __block storage type modifier.`

换句话说，我们的解决措施一开始就是讨论的对象变量。然而我们仅仅使用的是一个integer。

OK，那么将我们的代码换成使用对象变量，那么解决方法就应该起作用？

####When the documentation fails

	@interface Attempt5() {
    	id cleanupObj;
	    __block NSNumber *counterObj;
	}
	@end
	@implementation Attempt5
	-(id)init {
    	if (self = [super init]) {
        	cleanupObj = [[NSNotificationCenter defaultCenter] addObserverForName:notificationName object:nil queue:nil usingBlock:^(NSNotification *note) {
            	int oldCounterValue = counter;
	            counter++;
    	        counterObj = @(counterObj.intValue + 1);
        	    NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee violated.");
	        }];
    	}
	    return self;
	}
	- (void)setLocalCounter:(int)localCounter {
    	counterObj = @(localCounter);
	}
	- (int)localCounter {
    	return counterObj.intValue;
	}
	- (void)dealloc {
    	[[NSNotificationCenter defaultCenter] removeObserver:cleanupObj];
	}
	@end

按下Command+U，得到结果：

	"3" is not equal to "2" - Unexpected value for counter.
	"6" is not equal to "3" - Unexpected value for counter.
	"10" is not equal to "4" - Unexpected value for counter.
	"15" is not equal to "5" - Unexpected value for counter.

Seriously?这是怎样通过QA的？有人测试过么？写文档的是些什么人？

Well，那些人没有阅读编译说明书。菜鸟。因为，在WWDC视频，文档，示例代码，甚至是写代码。我打赌没有比阅读clang.org上得技术手册更好的了。

因为在一个27页的[Clang](http://clang.llvm.org/docs/index.html)文档中，甚至是在[目录](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#id50)中，非常清楚的指出在7.5章节( 原文：Because a 27-page document that doesn’t even rate a mention in the Clang documentation table of contents very clearly states buried in the middle of Section 7.5):

`The inference rules apply equally to __block variables, which is a shift in semantics from non-ARC, where __block variables did not implicitly retain during capture.`

让你继续搞懂这句话的意思。

No? So essentially this is compilerese for “We changed it.”

回到ARC以前，使用*__block*关键字将会阻止一个block去ratain一个变量。然而在ARC的世界中，我们有一系列的关于内存的关键字：*__strong*, *__weak*, *__autoreleasing*, *__unsafe_unretained*… 当介绍这些的时候，他们将*__block*从这些关键字中分离出去，所以你可以像这样写`__unsafe_unretained __block id foo`如果你喜欢的话。和其他类型的变量一样，默认的，*__block*变量隐式的内存关键字是*__strong*。

这就是为什么它没有起作用的原因。现在，你可能会说，把*__counterObj* 用*__weak*来修饰。当然，它不再拥有强引用。我们有了一个指向counter的弱引用，block将会使用它，会给它设置一个新的值。

接下来继续演示：

####Your invariants may vary

	
	@interface Attempt6() {
    	id cleanupObj;
	}
	@end
	@implementation Attempt6
	-(id)init {
    	if (self = [super init]) {
        	__weak Attempt6 *mySelf = self;
	        cleanupObj = [[NSNotificationCenter defaultCenter] 	addObserverForName:notificationName object:nil queue:nil 	usingBlock:^(NSNotification *note) {
    	        int oldCounterValue = counter;
        	    counter++;
            	NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee 	violated.");
    	        mySelf.localCounter++;
        	}];
	    }
    	return self;
	}
	- (void)dealloc {
    	[[NSNotificationCenter defaultCenter] removeObserver:cleanupObj];
	}
	@end


有很多人不喜欢这个解决方案：它依赖于使用一个公共的接口来访问自己的成员，例如，所以任何的notification 的实行你都需要使用一个public的API，这意味着你将要暴露自己。希望没有人用它。
但不管怎样，它可以工作，对吗？

	"3" is not equal to "2" - Unexpected value for counter.
	"6" is not equal to "3" - Unexpected value for counter.
	"10" is not equal to "4" - Unexpected value for counter.
	"15" is not equal to "5" - Unexpected value for counter.

SERIOUSLY. MUST. KILL. COMPILER.

OK，哪里出错了，我给你一个提示：如果在使用release模式来测试，它不会出错，它只会在debug模式下出错。

放弃么?

这里是答案：

	NSAssert(counter==oldCounterValue+1, @"Atomicity guarantee violated.");
	
看着，NSAssert是一个宏，扩展之后如下：


	do {
    	__PRAGMA_PUSH_NO_EXTRA_ARG_WARNINGS
	    if (!(condition)) {
    	    [[NSAssertionHandler currentHandler] handleFailureInMethod:_cmd
        	object:self file:[NSString stringWithUTF8String:__FILE__]
            	lineNumber:__LINE__ description:(desc), ##__VA_ARGS__];
	    }               \
    	    __PRAGMA_POP_NO_EXTRA_ARG_WARNINGS
	} while(0)


看见没有，多大一个self，引用循环自然测试失败。


####The Final Solution

这里是最终答案，使用更少被人知道的NSCAssert函数，它不支持Objective-C。

这个宏仅仅支持C函数。


Here’s the final answer, using the lesser-known NSCAssert function. Which, by the way, is not supposed to be used in Objective-C:

`This macro should be used only within C functions.`

下面的代码：

	@interface Attempt7() {
    	id cleanupObj;
	}
	@end
	@implementation Attempt7
	-(id)init {
    	if (self = [super init]) {
        	__weak Attempt7 *mySelf = self;
	        cleanupObj = [[NSNotificationCenter defaultCenter] 	addObserverForName:notificationName object:nil queue:nil 	usingBlock:^(NSNotification *note) {
    	        int oldCounterValue = counter;
        	    counter++;
            	NSCAssert(counter==oldCounterValue+1, @"Atomicity guarantee 	violated.");
    	        mySelf.localCounter++;
        	}];
	    }
    	return self;
	}	
	- (void)dealloc {
    	[[NSNotificationCenter defaultCenter] removeObserver:cleanupObj];
	}
	@end
	
	
	
更多内容请看[原文](http://sealedabstract.com/code/nsnotificationcenter-with-blocks-considered-harmful/)。
