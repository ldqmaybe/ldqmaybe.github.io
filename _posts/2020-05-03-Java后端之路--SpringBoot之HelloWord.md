---
layout:     post
title:      Java后端之路--SpringBoot之HelloWord
subtitle:   
date:       2020-05-03
author:     dingqiang.l
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - Java
---

使用SpringBoot构建HelloWorld项目，直接开始吧。
## 环境准备

 - 系统：macOS 
 - 开发工具：IntellIJ 	IDEA 
 - Java版本：open jdk1.8 
 - 项目管理工具：Maven

## 新建项目
打开idea，选择操作如图所示；如果我们已经打开了一个工作空间，File->New->Project即可，达到效果一致。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503183419601.png)

## 选择项目SDK版本
这里选择的是Java1.8，Service URL这里选择默认即可。点Next
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503183555875.png)

## 项目配置
这里使用的Maven管理，Java版本是1.8，并且简单配置了版本号、包名、项目描述等等，点Next
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503183910599.png)

## 选择项目依赖

正常新建一个Java Web项目的话，一般会选一些依赖，如数据库、SpringCloud、Test相关的等等，但是这里只是演示HelloWorld，所以就啥都不选了。直接点Next
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503183952663.png)
配置项目名称和路径，点Finish
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503184033322.png)

## 项目构建完成
最终项目结构就如下图所示了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503184158218.png)

## 编写HelloWord
如果我们要写一个HelloWord，通过浏览器访问，还需要添加一个web的依赖，当然SpringBoot也给我们提供有现成的。在pom.xml文件的dependencies下添加依赖即可

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
这样，写好一个简单的HelloWorldController后，run application就行了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503185419626.png?)
编译完成后如图所示，说明我们的项目已经跑起来了。值得注意的是，这里我们并没有配置过任何和Tomcat相关的东西，其实是SpringBoot内置了一个Tomcat服务器，我们直接使用即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503185521491.png)
最后打开我们的浏览器，键入[http://localhost:8080/](http://localhost:8080/) 就能看到之前在Controller编写的hello world了。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200503185617571.png)

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
