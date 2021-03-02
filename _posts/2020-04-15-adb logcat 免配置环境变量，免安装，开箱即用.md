---
layout:     post
title:      Java后端之路--SpringBoot之HelloWord
subtitle:   
date:       2020-05-03
author:     dingqiang.l
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - Android
    - 工具
    - 开发技巧
---

以前有写过一篇adb常用命令的使用和环境变量配置，[详见此文章](https://www.jianshu.com/p/4facec2cfc19)。基于此，现在分享一个非常好用的抓取日志的小工具。**注意：该工具免配置环境变量，免安装，开箱即用**

#### 一、准备工作

首先我们先准备adb相关文件，保存在adb文件夹下，如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200415124415844.png)
然后在adb文件夹同级目录新建一文本编辑这么一段代码。将文件保存为bat格式文件。
```java
@echo off
title 输出日志到文件
echo 【提示】
echo 1. 建议将cmd窗口设置为120x32或更大
echo 2. 使用Ctrl+C来终止日志输出
pause
set var=%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%
echo 日志输出已启动，请开始操作设备
echo 日志文件 log_%var%.txt
adb\adb logcat -v threadtime > log_%var%.txt
pause
```

#### 二、开始使用
直接双击打开刚刚的.bat文件即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200415131254280.gif)
#### 三、工具获取
关注我的微信公众号，回复"**adb**"，即可获取下载连接。

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)

