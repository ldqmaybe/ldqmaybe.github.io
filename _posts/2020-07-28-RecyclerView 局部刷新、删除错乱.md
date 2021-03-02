---
layout:     post
title:      RecyclerView 局部刷新、删除错乱
subtitle:   
date:       2020-07-28
author:     dingqiang.l
header-img: img/post-bg-mma-0.png
catalog: true
tags:
    - Android
---
列表的局部刷新和删除错乱问题尽管网上已经有很多牛人分享过解决方法，这里还是根据自己的实践记录一下吧。

## 一、局部刷新
有这样的场景，我们需要修改item中的某一要素，如：详情介绍，那么通常有以下几种方式：

>  **1. Adapter.notifyDataSetChanged()** 
>  **2. Adapter.notifyItemChanged(position)**
>  **3. Adapter.notifyItemChanged(position, payload)**

下面分别介绍这几种使用的可行性
### 1、notifyDataSetChanged()
使用该方法会刷新当前列表可见Item，虽然能实现数据刷新，但是背离了我们的初衷（局部刷新），而且存在性能消耗，所以**不建议使用**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728113229502.png#pic_center =180x320)
```java
2020-07-28 11:25:27.192 I/RecyclerView: MineEntity{des='山西冰糖心苹果...', imgResId=2131558403}
2020-07-28 11:25:27.193 I/RecyclerView: MineEntity{des='刷新可见list。。。', imgResId=2131558404}
2020-07-28 11:25:27.195 I/RecyclerView: MineEntity{des='山西冰糖心苹果...', imgResId=2131558403}
2020-07-28 11:25:27.196 I/RecyclerView: MineEntity{des='正宗烟台脆甜苹果...', imgResId=2131558404}
```
可见，没触发一次notifyDataSetChanged()都会重新执行onBindViewHolder()绑定数据。
### 2、notifyItemChanged(position)
该方法能实现局部刷新，但是有个弊端，就是会刷新当前Item的全部数据，如果Item有图片，图片就会闪烁一下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728114624269.gif#pic_center =180x320)

```java
07-28 11:37:12.100 I/RecyclerView: MineEntity{des='山西冰糖心苹果...', imgResId=2131558403}
07-28 11:40:41.034 I/RecyclerView: MineEntity{des='图片闪烁。。。', imgResId=2131558403}
07-28 11:40:43.314 I/RecyclerView: MineEntity{des='图片闪烁。。。', imgResId=2131558403}
07-28 11:40:45.290 I/RecyclerView: MineEntity{des='图片闪烁。。。', imgResId=2131558403}
```
这下没有刷新可见Item了，只刷新了当前Item，但是存在图片闪烁问题，所以也**不建议使用该方法。**
### 3、notifyItemChanged(position, payload)
最终使用该方法，可以解决上面两种方法存在的问题，先看效果![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728115546382.gif#pic_center =180x320)

```java
07-28 12:01:33.478 I/RecyclerView: MineEntity{des='山西冰糖心苹果...', imgResId=2131558403}
07-28 12:01:33.481 I/RecyclerView: payloads : false
07-28 12:01:36.196 I/RecyclerView: payloads : false
07-28 12:01:37.918 I/RecyclerView: payloads : false
07-28 12:01:38.852 I/RecyclerView: payloads : false
```
可以看到，图片不闪烁了，而且也不会刷新可见Item。使用也较为简单，实现RecyclerView.Adapter中带payloads参数的onBindViewHolder方法即可
#### 3.1 onBindViewHolder带payloads参数方法实现
```java
@Override
public void onBindViewHolder(@NonNull NewViewHolder holder, int position, @NonNull List<Object> payloads) {
    Log.i("RecyclerView","payloads : "+payloads.isEmpty());
    if (payloads.isEmpty()) {
        this.onBindViewHolder(holder, position);
    } else {
        for (Object object : payloads) {
            int payload = (int) object;
            if (payload == PAYLOAD_CODE) {
                holder.tvDes.setText(entityList.get(position).getDes());
            }
        }
    }
}
```
这里只是简单实现描述信息的修改。

#### 3.2 notifyItemChanged(position, payload)方法调用

```java
 mineEntity.setDes("时间 "+ TimeUtils.getNowString(new SimpleDateFormat("HH:mm:ss SSS", Locale.getDefault())));
 initList.set(position, mineEntity);
 mAdapter.notifyItemChanged(position, NewsAdapter.PAYLOAD_CODE);//优化局部刷新
```
只需以上两步，就能实现平时常见的图片闪烁问题和性能问题了。

## 二、删除Item发生错乱
平时做Item删除时，肯定有很多伙伴遇到过，明明自己想删掉Item2，结果Item3却被删掉了，而且删着删着应用还崩掉了。下面直接给正确用法。

```java
//删除item
initList.remove(position);
mAdapter.notifyItemRemoved(position);
mAdapter.notifyItemRangeChanged(position, initList.size() - position);
```


**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
