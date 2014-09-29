---
layout: post
title: "逆向马铃薯的iPad视频客户端"
date: 2014-09-29 21:58:03 +0800
comments: true
categories: 
---

<!--more-->
&#160; &#160; &#160; &#160;最近突然喜欢看神盾局特工，每天既可以练习英语又可以放松下。我在iPad上使用马铃薯的客户端来看视频，可是发现它只能在线看，郁闷，必须干掉它。接下来就让我带你装逼带你飞，说下我是怎么破了它的（仅供学习目的）！


##### 准备工作:
 
 * 越狱设备一台（iPad mini1， iOS 7.1.2）
 * 电脑（不知道说什么了，当然电脑上你得装上各种工具，比如class-dump, theos, ldid, dpkg, charles等等）
 * 机智的你
 
 （马铃薯的客户端版本号我就不透露了，现在是2014年9月，最新的客户端）

###### 逆向开始：

&#160; &#160; &#160; &#160;首先，我需要用class-dump获取到它所有的头文件，可是在AppStore中下载的应用是加了壳的。你如果直接用class-dump去获取头文件的话，得到的结果就是“呵呵”。你可以去三方应用市场找到没有壳的ipa文件，也可以自己砸壳，我懒得去找ipa文件了，直接用dumpdecrypted砸开。接下来你才能用class-dump，不过注意一下这里的可执行文件是包含了多个架构的（armv7,armv7s），你需要指明你需要的架构才能获取到你想要的头文件信息。不管三七二十一，先在Xcode建立一个新的project，把获取到的头文件丢进去再说。

&#160; &#160; &#160; &#160;接下来，我使用了Cycript帮助我分析。我需要使用它勾上你想分析的应用，先使用ps命令找到app的进程号为1699，然后执行cycript -p 1699 就成功勾上了这个进程。

&#160; &#160; &#160; &#160;然后我进入视频播放界面，如下图，你会发现点击缓存之后弹出剧集下载界面上，按钮是灰色的中间还有一条杠，表示不能下载。于是，我需要弄清楚现在界面上是哪一个ViewController。
![1png](https://raw.githubusercontent.com/billwang1990/PostImageSource/master/1.png)

&#160; &#160; &#160; &#160;通过观察，我发现下载面板是通过一个动画从屏幕底部加载到界面上的，应该是一个封装了相关操作的一个subview，那么先把controller的subview打印出来看看再说。
![2png](https://raw.githubusercontent.com/billwang1990/PostImageSource/master/2.png)
&#160; &#160; &#160; &#160;一眼就看到数组最后一个TDDownloadPannelView，从字面上理解就应该知道这个就是展现在眼前的下载界面。到Xcode中查看它的头文件，我首先去找的是看它的方法中是否有有download的字眼，没有任何发现。说明还没有查找到根源。我继续查看它的成员变量。其中有两个subview引起了我的注意，一个SeriesTableView，一个SeriesGridView。根据当前界面的布局方式，我判断当前界面上是一个SeriesGridView，接下来去看它的头文件。
![3png](https://raw.githubusercontent.com/billwang1990/PostImageSource/master/3.png)

&#160; &#160; &#160; &#160;这里逐个看，我的目光停留在了willTapOnItemAtIndex 和didTapOnItemAtIndex这两个方法上。接下来我决定去看一下didTapOnItemAtIndex这个方法（原因是我通过操作发现下载事件是在我点击之后手指离开屏幕才会触发，所以我优先看这个方法）。
这个时候我使用Hopper来帮助我分析，有钱的土豪可以使用IDA（我连Hopper都用的试用版本，说多了都是泪）。
&#160; &#160; &#160; &#160;把APP丢进hopper后定位到这个方法去分析，我找到了这个地方。

![4png](https://raw.githubusercontent.com/billwang1990/PostImageSource/master/4.png)


&#160; &#160; &#160; &#160;我圈起来的这个部分从字面意思很明显就是在判断对象是不是假的，真的的话就开始下载。OK，那就先从这里入手，顺势在hopper中找到这个isDummy是TDVideo的一个属性，那么我赶快写了一个tweak把这个属性给设置为NO。只做好deb安装包在ipad上安装好了之后，我满心欢喜的准备见证奇迹。MD，点击虽然不会提示我不能下载，但是还是不能下载啊，亲~
&#160; &#160; &#160; &#160;那么我估计在那个downloadVideo里面还有判断的操作，继续在hopper里面分析。果然那个方法里面还进行了是否已经下载过的判断操作，没有被下载过会调用它的delegate 的seriesDownloadSelectedRecordList方法。我接下来去查看在TDDownloadPannelView 里的这个方法。这个方法挺长的，在这个方法里我找到了MBProgressHUD的字眼，看了app就是使用的这个三方来提示用户，那么我估计这个方法差不多就是我想要找的地方了。这一段代码里，有一个单词引起了我的注意，就是limit，因为我发现有几个if分支都是在对对象的这个属性进行判断，为3、4或者4都不会进行下载，然后进行错误提示或者别的相关提示。另外从字面上来看，这个应该也是表明是否限制的意思。TDBaseVideo里找到了这个属性，是一个int类型的属性。我想这个值应该直接从服务器获取来的。于是，打开Charles抓包看看，如下图，很清晰的看到了limit这个字段为1。

![5png](https://raw.githubusercontent.com/billwang1990/PostImageSource/master/5.png)

&#160; &#160; &#160; &#160;我赶紧找了一个能下载的视频进行抓包，发现这个字段为0。那么我在tweak中继续hook了这个属性，让它一直返回0。生成deb安装到机器上看效果，大工告成，想看的视频可以下载了。
&#160; &#160; &#160; &#160;最后，由于时间关系，一些太过于细节的东西我并没有过多介绍，如果有问题，直接联系我就好QQ:563691569。
 
&#160; &#160; &#160; &#160;挖掘机技术哪家强！
