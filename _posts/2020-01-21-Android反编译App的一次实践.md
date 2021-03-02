---
layout:     post
title:      2020-01-21-Android反编译App的一次实践.md
subtitle:   
date:       2020-01-21
author:     dingqiang.l
header-img: img/post-2020-01-21.jpg
catalog: true
tags:
    - Android
    - 开发技巧
---


> 最近项目准备上线版本迭代，但是投产前要先过了安全检测，结果下来后懵逼了，检测出来的漏洞比上个版本的检测还多，其中有个中危的我比较在意：反编译结果显示未经过加固和有效混淆？于是抱着怀疑的态度去做一次加固前和加固后的反编译对比，结果。。。

本次反编译实践使用到的工具有：

> **apktool:** 用于从apk文件提取classes.dex文件的，v2.4.0版本。[官网下载](https://bitbucket.org/iBotPeaches/apktool/downloads/?tab=downloads)
> **dex2jar:** 将classes.dex文件转成classes-dex2jar.jar,v2.0版本。[官网下载](https://sourceforge.net/projects/dex2jar/)
> **jd-gui-windows-1.4.1** 图像化工具，查看classes-dex2jar.jar源码。[官网下载](http://java-decompiler.github.io/)

我这里是把这几个工具都整到一个目录下了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121165216476.png)
操作步骤可以简单分为3步：

> 1、利用apktool提取classes.dex 
> 2、利用dex2jar将dex转换成jar 
> 3、利用gui工具查看反编译代码

## apktool提取classes.dex
命令进入到apktool目录下，执行命令 `apktool d -s xxx.apk`,如下图，输入`apktool d -s sec_unsigned.apk`后会生成sec_unsigned文件夹，classes.dex就在其中，到此我们的dex文件就提取好了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121173107445.png)

**拓展**
上面用到的是apktool d -s ,有兴趣的可以试试其他的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121181413809.png)

## dex2jar将dex转成jar
上一步我们已经得到classes.dex文件，现在我们将dex拷贝到dex2jar-2.0文件夹下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121173738177.png)
然后命令进入dex2jar-2.0文件夹执行命令 `d2j-dex2jar classes.dex`即可生成我们最终需要的 **classes-dex2jar.jar**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121174010221.png)

## jd-gui查看源码
我们打开gui图形化工具，打开 **classes-dex2jar.jar**文件，可以看到源码结构和部分代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/202001211746307.png)
这里有遇到过两个坑，一是打开gui工具，如果我们遇到以下问题，可以敲命令到jd-**gui.exe** 目录下，输入 `java -jar jd-gui.exe` 即可打开
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121181624745.png)


到此，简单的反编译已经做完了。

## 加固前后对比

上面反编译的是加固前的apk，那么如果apk加固后了呢，还能不能按上面的方法反编译呢？搞起
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200121175604822.png)
可见，加固还是有点用的，同样的操作步骤，加固后是反编译不出来了。

> 市面上有很多加固平台可选，小伙伴们自己衡量，值得注意的是有些应用市场上，如果没加固过是禁止上的。

## 最后
以上就是本人的一次小小的实践，有哪里说的不对的，还请大神们指正。

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
