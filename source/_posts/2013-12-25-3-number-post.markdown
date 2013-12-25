---
layout: post
title: "MAC下安装Octopress，Cocoapods关于ruby版本的问题"
date: 2013-12-25 14:01:54 -0800
comments: true
categories: 
---


不多说，直接上正题。
系统是10.8.4的，没升级，黑苹果不敢升级，每次升级会很麻烦！

因为开发iOS，所以需要使用cocoapods来管理三方, 另外需要搭建一个Octopress的blog，两者都需要用到ruby，问题来了：

Octopress要求ruby的版本不低于 1.9.3，而系统自带的ruby是1.8.7的，很明显不搭调，于是使用rvm install 1.9.3安装ruby。 然后照[Octopress官方安装教程][http://http://octopress.org/docs/setup/]配置。

OK，Octopress一切完工，这个时候我来安装cocoapods，它也用到了ruby，这个时候默认的ruby已经不是系统自带的ruby了。于是在使用 $sudo gem install cocoapods安装的时候一直报错 ：COreFoundation is needed to build the Xcodeproj C extension.

找了很多解决办法，比如重新安装xcode command line tools，使用xcode-selelct等等，都不起作用。好吧，最后还是去cocoapods看看，开发者强烈建议使用Mac自带的ruby来安装cocoapods。

好吧，最后我妥协了，将默认的ruby还原为系统自带的ruby，这个时候再来安装cocoapods，一切正常. 然后只是将octopress的工作目录使用rvm配置为使用1.9.3的ruby，也就是说使用了两个版本的ruby。

方法虽然看起来是土了一点，但是简单粗暴有效！



