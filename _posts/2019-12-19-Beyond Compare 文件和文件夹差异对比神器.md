---
layout:     post
title:      Beyond Compare 文件和文件夹差异对比神器
subtitle:   
date:       2019-12-19
author:     dingqiang.l
header-img: img/post-2019-12-19.jpg
catalog: true
tags:
    - 工具
---

平时工作中，或多或少都会遇到比较两个文件或两个文件夹之间的差异性，比如两份代码、两个文案甚至两张图片。如果内容很多，不可能一行一行去比较，这时我们可能需要一些工具来辅助，这里介绍神器Beyond Compare。

为什么说是神器？唯有强大！

文件比较？简单；文件夹呢？不再话下，甚至是表格、16进制、MP3、图片、注册表和版本都能轻松拿下。如此强大的伙伴，待我细细道来。

## 安装包下载 ##
[官方下载地址](http://www.scootersoftware.com/download.php),可以看到支持不同的操作系统，并支持多种语言。需要注意的是官方只提供30天的免费使用时间，意思是使用一天算一天，注意是实际的使用天数。下载后自行安装即可，建议不要装在C盘。
![](https://img-blog.csdnimg.cn/20191219165402417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)

## 如何使用 ##

安装好之后目录结构是这样的，其中BCompare.exe是我们的启动程序，双击打开即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165514562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
打开之后的主界面是这样的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165528690.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
### 文件比较 ###

将两个不同的文件拖拽进来即可，可以看到，显示面板直接将两个文件的内容展示，并以红色标记差异的内容，我们可以对这两个差异文件进行操作，点击黄色箭头会将当前文件的此处差异内容覆盖另一个文件，下方灰色小箭头也能实现一样的效果

右边有个预览面板，可以快速定位到差异内容的位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165545847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
### 文件夹比较 ###

同样，将两个文件夹拖拽进去，褐色表示多出，红色表示冲突（虽然文件名一样，但是内容有差异），灰色表示缺少。像平时对比两个版本代码，这是一个比较好的方法了吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165608519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
### 图片比较 ###

第一张图片是原图，第二张是经[压缩](https://tinypng.com/)后的。虽然肉眼看不出两张图片有什么不一样，但是通过工具还是能明显看出哪里的像素是有差异的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165637275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
其余	表格、MP3什么的对比这样就不一一展开了。下面开始介绍如何突破30天试用限制。

## 软件突围 ##

开始的时候已经说了，正版是只有30天的免费期的，那我们想一直使用怎么办，人民币玩家当然不用考虑这种问题啦。如果想一直免费，只能走捷径了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165653592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
突围的方法我搜罗了三种。

**方法一：** 修改安装目录下的  **BCUnRAR.dll** 文件，比如我安装在D:\SoftWare\Beyond Compare 4，这个文件（BCUnRAR.dll）重命名或者直接删除，则会新增30天试用期。该方法操作简单，缺点是不是永久的。

**方法二：** 一劳永逸，修改注册表：

> 1. 快捷键win+R或在搜索栏中输入 regedit，打开注册表
> 1. 删除项目：计算机\HKEY_CURRENT_USER\Software\Scooter Software\Beyond Compare 4\CacheId

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165713108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)
**方法三：** 修改C:\Users\Administrator\AppData\Roaming\BCompare\BCompare.ini这个配置文件中的时间戳。修改为当天即可

如果以上办法都失败了，还有一个终极的办法，那就是卸载重装，这样就又有30的免费使用期了（内心一万只草泥马在奔腾）。最后就介绍到这里了

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70#pic_center)











