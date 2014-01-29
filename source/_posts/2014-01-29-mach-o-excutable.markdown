---
layout: post
title: "Mach-O Excutables"
date: 2014-01-29 14:40:15 +0800
comments: true
categories: 
---

当我们使用xcode开发程序的时候，源文件（*.h *.m）会被编译成可执行文件。那么这一切到底是怎样发生的呢？下面来简单说明。

<!--more-->

####xcrun

xrun是我们使用最多的一个命令行工具，这个小工具是用来运行别的工具的，比如：

	% clang -v

在命令行中，我们可以使用如下命令来代替上面的：

	% xcrun clang -v
	
xcrun将会指定clang然后传递后面的参数给它来运行。

####不用IDE来运行hello world

回到命令行中，我们创建一个文件夹并生成一个C源文件。

	% mkdir ~/Desktop/objcio-command-line
	% cd !$
	% touch helloworld.c
	
接下来打开生成的C文件

	% open -e helloworld.c

编辑它：

	#include <stdio.h>
	int main(int argc, char *argv[])
	{
    	printf("Hello World!\n");
    	return 0;
	}

保存后编译运行：

	% xcrun clang helloworld.c
	% ./a.out
	
这个时候命令行就会输出你熟悉的Hello World了，整个过程都在命令行完成。

在刚才的过程中，我们将helloworld.c编译成一个Mach-O的可目标文件被称作a.out。


####Hello World and the Compiler

我们使用的编译器是clang。

简单来说，编译器会将源文件生成可执行文件，这个过程由一系列步骤组成。


####预处理

* `Tokenization`
* `Macro expansion`
* `#include expansion`


####语义分析

* `Translates preprocessor tokens into a parse tree`
* `Applies semantic analysis to the parse tree`
* `Outputs an Abstract Syntax Tree (AST)`

####生成代码并优化

* `Translates an AST into low-level intermediate code (LLVM IR)`
* `Responsible for optimizing the generated code`
* `target-specific code generation`
* `Outputs assembly`

####汇编

* `Translates assembly code into a target object file`


####链接

* `Merges multiple object files into an executable (or a dynamic library)`


简单的记录了一下几个步骤，更多详细的内容请查看[这篇文章](http://www.objc.io/issue-6/mach-o-executables.html)。

