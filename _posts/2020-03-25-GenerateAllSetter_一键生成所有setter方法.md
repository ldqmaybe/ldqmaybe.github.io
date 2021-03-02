---
layout:     post
title:      GenerateAllSetter_一键生成所有setter方法
subtitle:   
date:       2020-03-25
author:     dingqiang.l
header-img: img/post-bg-YesOrNo.jpg
catalog: true
tags:
    - 工具
    - 开发技巧
---

写过Java的同学都知道，当JavaBean有大量的属性时，我们setXxxx()的时候就非常痛苦了，花费大量的时间去做一些苦力活，显示不是我们想要的。

使用GenerateAllSetter就能解决这个问题，它就能做到一键生成一个对象的所有的set方法。

**先看效果：**![在这里插入图片描述](https://img-blog.csdnimg.cn/20200325152001787.gif)
图中可以看到通过GenerateAllSetter插件一键生成了User类的setXxx方法，简化了我们平时大量手工码字的工作。

其实我们敲Alt+Enter键时，会看到有三个选项，视频中选的是填充默认值，另外两个看需求选择

**如何安装**
以Android Studio为例。选择File-》Settings-》Plugins，在Marketplace输入GenerateAllSetter搜索即可，安装完成后重启AS就大功告成了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200325153354689.png)
该插件同样适用于IntelliJ IDEA，JetBrains家族其他的工具应该也适用吧，这里有兴趣的同学可以去验证。

下载的时候我们可能会遇到下载失败或者下载慢的现在，这可能是由于国内处于半墙状态，类似GitHub，所以这里建议小伙伴们科学上网。

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)

