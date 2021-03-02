---
layout:     post
title:      Android Studio Debug 断点调试，各种骚操作都在这里了
subtitle:   
date:       2019-12-04
author:     dingqiang.l
header-img: img/post-2019-12-04.jpg
catalog: true
tags:
    - Android
    - 开发技巧
---

Android Studio中的断点调试功能很好用，可谓是强大，用的好，不仅可以一定程度上提高开发进度，还能提高逼格。首先从最开始的来吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181137859.jpg)

## 启动Debug ##

启动Debug有两种方法，一是Debug启动APP；二是Attach Debugger。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181237885.png)
方法一和Run App操作类似，点击之后会项目会运行在我们的设备上，然后就可以开始后面的调试了；

方法二的前提是我们的项目必须已经运行过，点击之后就可以选择我们要调试的进程了，后续的调试两种方法操作是一样的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181254894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
## 设置断点 ##
设置断点比较简单，在调试处鼠标单击左键行号即可。当程序运行到该行将停下来，同时我们可以在Debug调试面板上可以看到该断点所处的类、方法和变量等信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181319170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
## 简单调试 ##

###  1、Step Over (F8)  ###
点击该按钮或快捷键F8，会直接跳到下一行，尽管该行有方法，也会运行完方法后执行到下一行。**有个小技巧：鼠标左键点击行号，会直接执行到该行**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181341671.png)
###  2、Step Into (F7)  ###
点击该按钮或快捷键F7，如果该行有方法，则进入到方法中，否则执行下一行；其实主要作用是进入方法中
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181351768.png)
###  3、Force Step Into (Alt+Shift+F7)  ###
Force Step Into和Step Into作用是一样的，区别是Step Into只能进入我们自己定义的方法,遇到JDK的方法如图的String.length(),不会进入；而Force Step Into可以进入自定义和JDK的方法中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181359660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
###  4、Step Out (Shift+F8)  ###

跳出方法:如果我们调试进入了String.length()，可以通过该操作跳出length()执行下一步
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181423466.png)
###  5、Run to Cursor (Alt+F9)  ###

执行到下一个断点，比如下图中，程序执行到55行时我们点一下按钮，则直接执行到57行，再点一下会执行到61行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181438411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
以上是平时的一些基础用法... ...

## 更高级的玩法 ##

###  1、条件断点 ###

非常有用的一个操作，设想我们的断点打在一个循环中，而我们只想验证某个值，常规做法是Step Over（可能想哭）和日志输出，但是有了条件断点就方便很多了。直接在红点上右键，然后在条件框中输入我们的条件，如i==50,只有i等于50的时候，程序才会暂停执行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181503426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
###  2、方法断点 ###
除了红点形状不一样外，感觉用法没什么不同（哈哈，看来还没达到一定的境界）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181521262.png)
###  3、异常断点 ###
有时候应用出现了异常，如最常见的NullPointerException，问题的定位通常是抓日志打断点再分析，但是学会了异常断电后，我们可以粗暴的运用该技巧，系统直接定位到抛该异常的位置并暂定执行。写个例子，强行抛出NullPointerException。

> 操作步骤：点击位置1处弹出Breakpoints面板-》点击位置2处+号-》选择3.Java Exception Breakpoints-》在弹框中键入异常类型如NullPointerException即可

好处是我们不用去打断点，系统会自动定位到抛该异常的位置，并且程序暂停执行，非常好
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181542559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
###  4、Evaluate Expression  (Alt+F8) ###

计算表达式窗口：可以动态查看和修改某个变量的值。比如例子中的isAdd初始值是false，通过该操作，不仅可以查看变量的执行到该步骤的值，而且还能修改值，如图，将isAdd该成true后，if条件不成立，可以实现我们临时修改某一状态的目的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181603911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
###  5、View Breakpoints  (Ctrl+Shift+F8) ###
可以看到程序中所有打了断点的位置，这里我们可以快速取消某个断点，也可以选择执行线程中的或全部的断点，甚至可以设置条件，满足条件才暂停执行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181623755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
###  6、Watches  ###
如果想动态观察某个值的变化，而Variables面板的变量又太多，这是我们可以使用Watches；点击 + 按钮，在输入框输入或下拉选择历史记录，就可以观察到该变量的值了。

当然也有快捷操作，直接在代码中选中该变量，右键选择Add to Watches，同样能将该变量添加到Watches面板。如果想在Watches面板中移除该变量，右键Remove Watches即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181653615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
如果不喜欢上图展示的界面，点击 像眼镜的按钮，就可以看到另一种展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191204181715718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
常用的基本就这些了，如果有哪里不恰当的，请大家指出

**欢迎关注我的个人微信公众号，优了个秀和你每天进步一点点**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120418174542.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)
