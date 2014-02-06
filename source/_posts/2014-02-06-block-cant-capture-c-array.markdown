---
layout: post
title: "#iOS# Block为什么不能捕获C语言数组的值"
date: 2014-02-06 10:43:55 +0800
comments: true
categories: 
---


众所周知，在iOS的block中，我们可以截获自动变量，但是为什么如下截获C语言数组的代码却不行：
<!--more-->

	const char text[]  = “关注我得博客billwang1990.github.io”；

	void (^block)(void) = ^{
    	 printf(“%c\n”, text[2]);
	}

要弄清楚这个问题，就必须明白block是怎样实现的。

简单来说，所谓的“截获自动变量”意味着在执行Block语法时，Block语法表达式所使用的自动变量的值被被保存到Block结构体实例(即Block自身)中。

之所以C数组不能截获，就类似下面的代码：

	void func(char a[10])
	{
       char b[10] = a ;
       printf(“%d\n”, b[0]);
	}

	int main()
	{
    	 char a[10] = {2};
	     func(a);
	}

这段代码是不能通过编译的，这也解释了为什么Block不能截获C数组。


