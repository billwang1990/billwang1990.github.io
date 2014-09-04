---
layout: post
title: "使用LLDB远程调试APP"
date: 2014-08-07 23:29:16 +0800
comments: true
categories: 
---

<!--more-->

####背景：

因为最近要开始学习一些iOS逆向的一些东西，调试别人的app自然是必不可少的工作，这个时候调试利器GDB和LLDB自然浮现在脑袋里。可是试验后发现用GDB调试并不好用，而且苹果推的也是LLDB，所以需要使用LLDB来进行调试工作。使用LLDB就不像使用GDB进行调试那么方便，使用GDB的话直接在Cydia里面安装好GDB之后，ssh到你的设备就可以开始工作了，使用LLDB远程调试你越狱设备上的APP稍微麻烦一点。

####准备工作：

* 一台越狱了的设备
* 一台装有Xcode的MAC

####调试步骤：

 1. 需要挂载Xcode的developer 磁盘镜像
 
 	<code>
 hdiutil attach /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneO S.platform/DeviceSupport/7.1\ \(11D167\)/DeveloperDiskImage.dmg
 	</code>
 
 2. 这个时候将usr/bin目录里的debugserver拷贝出来（我直接拖到的桌面） 
 
 3. 创建一个plist文件，我用xcode创建了的 entitlements.plist放在桌面，里面创建了下面4个key，都是YES。
 
 	* com.apple.springboard.debugapplications
 	* run-unsigned-code
 	* get-task-allow
 	* task_for_pid-allow
 
 4. 进行签名
 
 	`codesign -s - --entitlements entitlements.plist -f debugserver`

 5. 签名完成之后将debugserver复制到你的越狱设备上面去，你可以用scp，也可以用iTool等图形界面工具，随你喜好。
 
 6. ssh到你的设备去，然后执行下面的命令：
 
 	~~~~`./debugserver *.(端口号) -a "你要调试的app的name"`~~~~(写错了,下面这行才对)
 	
 	`./debugserver *:(端口号) -a "你要调试的app的name"`
 7. 在你的MAC上新开一个terminal，然后输入 *lldb* 命令，接着输入
 
 	`process connect connect://192.168.1.102` （注：记得换成你自己的IP）
 	

 8. 开始发挥LLDB的威力吧，啊哈哈哈哈哈。
 	
参考链接：

* [http://iphonedevwiki.net/index.php/Debugserver](http://iphonedevwiki.net/index.php/Debugserver)
