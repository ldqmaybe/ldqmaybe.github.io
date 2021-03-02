---
layout:     post
title:      Android Framework
subtitle:   
date:       2017-04-08
author:     dingqiang.l
header-img: 
catalog: true
tags:
    - Android
---
# 写在前面：
 > 在学习Android Framework时特此将相关知识记录下，方便以后翻阅。

 自表层至底层可将android framework 分为：
>1. 应用层：就是我们使用java语言开发出来使用的app
2. 应用框架层：提供api（文本框、按钮、列表等），内容提供者contentProvider、通知管理、资源管理、活动管理（ActivityManager）等
3. 系统库：分为运行时库和核心库：显示管理库，SQLite库媒体库webkit浏览器引擎；dalvik虚拟机，与java虚拟机不太一样，dalvik虚拟机运行的是自己的字节码而不是和java虚拟机那样运行的是java字节码。
4. Linux核心库：提供网络、安全、内存等管理的支持，是运行在Linux2.6之上的

# 写在最后 ：
附上一张很重要的图

![AndroidFramework.png](https://img-blog.csdnimg.cn/img_convert/17dd49500c88306b989b3b40b522ca20.png)
