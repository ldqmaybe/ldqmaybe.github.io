---
layout:     post
title:      Android Intent 传输大图片
subtitle:   
date:       2020-06-02
author:     dingqiang.l
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Android
---
Android中的Intent作为四大组件间通讯的桥梁，支持传输基本数据类型、序列化对象等等
![\[图片\]](https://img-blog.csdnimg.cn/20200602143611604.png#pic_center )
但是要传大图片呢？能不能传呢？下面开始做个试验，先准备一张近500k大小的图片，存放在mipmap下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602143923626.png#pic_center)
直接通过Intent的putExtra方法传输，并且打印bitmap大小。

```java
Intent intent = new Intent(this, Main2Activity.class);
bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.abc);
Log.i("MainActivity", "bitmap大小: " + bitmap.getByteCount() / 1024 + " kb");
intent.putExtra("bitmap", bitmap);
startActivity(intent);
```

**运行结果**
控制台打印了bitmap的大小

```java
bitmap大小: 116109 kb
```
但是应用崩溃了，抛了`TransactionTooLargeException`异常，意思是交易数据过大？

> 原因是调用Binder IPC时，系统会临时分配一块缓冲内存，并且大小只有1M，所以使用Intent传递数据的时候（底层实际也是使用Binder IPC），数据超过限制时（1M 减去 其他事务使用的内存）就会抛出该异常。详见 Android 面试官的《跨进程传递大图，你能想到哪些方案呢？》

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602144837329.png)

## 解决方案
#### 1、传递路径方式
可以将图片保存在文件中，再将图片路径传给目标组件，目标组件用路径获取该图片。该方案可行，缺点是性能不高，所以不推荐。
#### 2、Socket 方式
数据会拷贝两次，同样也有性能缺陷，所以也不推荐（没用过）
#### 3、数据分组方式
将图片转成字节数组，分组通过Intent传递，目标组件获取所有的数据后再重新组合，太麻烦了，估计也没人这样做。
#### 4、Binder IPC 方式
前面也验证过直接用Intent传大图是不可行的，但是细心的我们发现，Bundle有个putBinder方法

```java
public void putBinder(@Nullable String key, @Nullable IBinder value) {
    unparcel();
    mMap.put(key, value);
}
```
可以看到，第二个参数接收的是一个IBinder对象，我们可以用这个IBinder接收Bitmap，然后通过putBinder方法传给目标组件。下面看看用法

## `Bundle#putBinder`使用
来个简单的例子，新建一BitmapBinder继承Binder

```java
public class BitmapBinder extends Binder {
    private Bitmap bitmap;

    BitmapBinder(Bitmap bitmap) {
        this.bitmap = bitmap;
    }

    Bitmap getBitmap() {
        return bitmap;
    }
}
```
通过putBinder传递Bitmap，并打印bitmap大小

```java
Intent intent = new Intent(this, Main2Activity.class);
bitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.abc);
Bundle bundle = new Bundle();
Log.i("MainActivity", "bitmap大小: " + bitmap.getByteCount() / 1024 + " kb");
bundle.putBinder("bitmap", new BitmapBinder(bitmap));
intent.putExtras(bundle);
```
在接收组件接收，并打印bitmap大小

```java
Bundle bundle = getIntent().getExtras();
BitmapBinder bitmapBinder = (BitmapBinder) bundle.getBinder("bitmap");
bitmap = bitmapBinder.getBitmap();
Log.i("MainActivity", "Mani2Activity bitmap 大小" + bitmap.getByteCount() / 1024 + " kb");
```
最终的结果是正常跳转，并且日志输出如下。可见使用该方式可以解决Activity之间传递大图的问题。

```java
bitmap大小: 116109 kb
Mani2Activity bitmap 大小116109 kb
```
**结论：** Intent直接传递是将数据拷贝到Binder的缓冲内存中（大小限制了1M）；putBinder则是利用共享内存，所以能满足大图传输，原理详见 Android 面试官的《跨进程传递大图，你能想到哪些方案呢？》

## 拓展
如果传递的是超长的String数组呢？这里猜测也会抛数据过大的异常。

这里将一个txt文件存放在assets目录，然后将内容读取，打印内容长度`内容大小478 kb`，然后直接使用Intent传递，可以看到同样抛了一样的异常，可见传递超长的基本数据类型也是不可行的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602164438855.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/20200602164454795.png)
使用putBinder方法，代码类似传递大图，结果是可以传递
```java
InputStream inputStream = getAssets().open("txt.txt");
int lenght = inputStream.available();
byte[] bytes = new byte[lenght];
int read = inputStream.read(bytes);
String txtContent = new String(bytes, StandardCharsets.UTF_8);
Log.i("MainActivity", "txt 内容大小" + txtContent.length() / 1024 + " kb");
bundle.putBinder("txt",new BitmapBinder(txtContent));
```
日志输出

```java
txt 内容大小478 kb
Mani2Activity txt 内容大小478 kb
```
其实结果也没什么可说的，原理和之前的大图类似。所以，如果平时开发过程中遇到要传递大图片或超长字符串等，可以考虑使用这种方法。

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)

