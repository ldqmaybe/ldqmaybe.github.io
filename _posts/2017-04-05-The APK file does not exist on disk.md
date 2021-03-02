---
layout:     post
title:      The APK file does not exist on disk
subtitle:   
date:       2017-04-05
author:     dingqiang.l
header-img: 
catalog: true
tags:
    - Android
    - 开发技巧
---
# 写在前面：
在我们使用android studio 时，一般我们想对编译时生成的apk文件进行自定义命名，并且使用了我们自己定制的规则，如：使用版本号和时间戳。 由于gradle在执行编译命令和安装命令时有时间差，且调用了两次你的名称规则，导致编译出来的apk名称和安装时获取到的apk名称不一致，所以它就报找不到指定的apk文件了。

##### 解决方案：

![The APK file does not exist on disk.png](https://img-blog.csdnimg.cn/img_convert/c76d68cd37c6df0cecfb1781aeebd36a.png)
同步一下 Gradle即可。

# 写在最后：
继续发现。。。
