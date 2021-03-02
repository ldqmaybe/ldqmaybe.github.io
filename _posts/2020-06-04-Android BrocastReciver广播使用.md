---
layout:     post
title:      Android BrocastReciver广播使用
subtitle:   
date:       2020-06-04
author:     dingqiang.l
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Android
---
BrocastReciver(广播)是Android四大组件之一，在Android体系中占据比较重要的地位。比如监听网络的变化、应用安装卸载、电量变化等等
广播按类型可分为普通广播、有序广播。按注册方式分为静态注册和动态注册

> 1、普通广播：sendBroadcast()方法发送，顺序不确定，无法拦截；但是效率较高
> 2、有序广播：sendOrderedBroadcast()方法发送，顺序确定（-1000~1000），数值越大优先级就越高，可以被拦截。
> 3、静态注册：在AndroidManifest.xml中注册，并在intent-filter中添加响应的action
> 4、动态注册：在Context(即Service或Activity)组件中注册，通过registerReceiver()方法注册，注意在不使用时取消注册unregisterReceiver();

## 普通广播：

下面以网络变化广播为例

#### 静态注册：
自定义一个广播类CustomBrocastReciver继承BrocastReciver，在清单文件中注册，并申请权限

```java
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    
<receiver android:name=".CustomBrocastReciver">
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
    </intent-filter>
</receiver>
```
简单实现如下：

```java
public class CustomBrocastReciver extends BroadcastReceiver {
    private static final String TAG = "CustomBrocastReciver";

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        Log.i(TAG, "onReceive: action = " + action);
        if (ConnectivityManager.CONNECTIVITY_ACTION.equals(action)) {
            ConnectivityManager connectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
            if (networkInfo != null && networkInfo.isAvailable()) {
                Log.i(TAG, "onReceive: 已连网");
                Toast.makeText(context, "已连网！ " + networkInfo.getSubtypeName(), Toast.LENGTH_SHORT).show();
            } else {
                Log.i(TAG, "onReceive: 已断网");
                Toast.makeText(context, "已断网", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```
手动改变设备网络状态，查看日志如下：
```java
I/CustomBrocastReciver: onReceive: action = android.net.conn.CONNECTIVITY_CHANGE
I/CustomBrocastReciver: onReceive: 已连网
I/CustomBrocastReciver: onReceive: action = android.net.conn.CONNECTIVITY_CHANGE
I/CustomBrocastReciver: onReceive: 已断网
```

> **注**：7.0以上系统用静态注册方法已经监听不到网络状态变化。

#### 动态注册：
动态注册则不需要在清单文件中添加intent-filter#action，本例是在Activity中注册，如下

```java
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION);
reciver = new CustomBrocastReciver();
registerReceiver(reciver, intentFilter);
//取消注册
unregisterReceiver(reciver);
```
操作设备上网络开关，日子输出结果和静态注册一致

## 有序广播：
新建3个广播并在AndroidManifest.xml中注册，这里演示动态注册，所以不需要在清单文件中添加相关的intent-filter信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604153235792.png)

```java
<receiver android:name=".Order1BrocastReciver"/>
<receiver android:name=".Order2BrocastReciver"/>
<receiver android:name=".Order3BrocastReciver"/>
```

在代码中注册这3个广播，设置不同的优先级，并同时监听同一个广播`OrderBrocastReciverAction`

```java
//注册3个有序播，设置不同的优先级，监听同一个广播信息
String orderBrocastReciverActioin = "OrderBrocastReciverAction";
IntentFilter intentFilter1 = new IntentFilter(orderBrocastReciverActioin);
intentFilter1.setPriority(3);
IntentFilter intentFilter2 = new IntentFilter(orderBrocastReciverActioin);
intentFilter2.setPriority(2);
IntentFilter intentFilter3 = new IntentFilter(orderBrocastReciverActioin);
intentFilter3.setPriority(4);
order1BrocastReciver = new Order1BrocastReciver();
registerReceiver(order1BrocastReciver, intentFilter1);
order2BrocastReciver = new Order2BrocastReciver();
registerReceiver(order2BrocastReciver, intentFilter2);
order3BrocastReciver = new Order3BrocastReciver();
registerReceiver(order3BrocastReciver, intentFilter3);
```
注册好之后就可以发送广播了

```java
Intent orderIntent = new Intent();
orderIntent.setAction(orderBrocastReciverActioin);
//发送有序广播
sendOrderedBroadcast(orderIntent, null);
```
查看日志发现这三个广播接收者接收到的广播顺序和设置的优先级是一致的

```java
I/BroadcastReceiver: onReceive: Order3BrocastReciver
I/BroadcastReceiver: onReceive: Order1BrocastReciver
I/BroadcastReceiver: onReceive: Order2BrocastReciver
```
验证是否能拦截，Order1BrocastReciver中添加 `abortBroadcast()`方法进行拦截

```java
public class Order1BrocastReciver extends BroadcastReceiver {
    private static final String TAG = "BroadcastReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.i(TAG, "onReceive: Order1BrocastReciver");
        abortBroadcast();
    }
}
```
重新运行查看效果发现，Order2BrocastReciver并没有接收到广播。原因是Order1BrocastReciver的优先级比Order2BrocastReciver的高，Order1BrocastReciver接收到广播后进行了拦截，导致了该广播并没有往下传，所以Order2BrocastReciver接收不到。

```java
I/BroadcastReceiver: onReceive: Order3BrocastReciver
I/BroadcastReceiver: onReceive: Order1BrocastReciver
```
最后：使用广播时千万要记得，有注册就已经要有反注册，否则迎来的就是内存泄露；如果时Android7.0以上的系统，部分广播可能无法接收到，如网络状态变化广播，[详见这里。](https://developer.android.com/guide/components/broadcast-exceptions)

## 拓展
上面说到网络状态变化广播在Android7.0以上无法通过静态注册接收，如果要监控网络变化，一个方法是通过动态注册方法；一个是通过Google推荐的registerDefaultNetworkCallback。

CONNECTIVITY_ACTION已经被@Deprecated标识，并解释应该使用更牛B的方式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604155308517.png)
下面简单介绍registerDefaultNetworkCallback方式，需要注意的是该方法要SDK24以上

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
    connectivityManager.registerDefaultNetworkCallback(new ConnectivityManager.NetworkCallback() {

        @Override
        public void onAvailable(Network network) {
            super.onAvailable(network);
            Log.i(TAG, "onAvailable: 网络已连接：" + network);
        }

        @Override
        public void onLost(Network network) {
            super.onLost(network);
            Log.i(TAG, "onLost:网络已经断开， " + network);
        }

        @Override
        public void onCapabilitiesChanged(Network network, NetworkCapabilities networkCapabilities) {
            super.onCapabilitiesChanged(network, networkCapabilities);
            if (networkCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)){
                //已经连接成功
                if (networkCapabilities.hasTransport(NetworkCapabilities.TRANSPORT_WIFI)) {
                    Log.i(TAG, "onCapabilitiesChanged: 此网络使用Wi-Fi传输" );
                }else if (networkCapabilities.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR)) {
                    Log.i(TAG, "onCapabilitiesChanged: 此网络使用蜂窝传输");
                }else {
                    Log.i(TAG, "onCapabilitiesChanged: 其他网络类型");
                }
            }
        }
    });
```
改变设备网络，日志输出如下，这里使用的华为mate20pro测试

```java
onAvailable: 网络已连接：318
onCapabilitiesChanged: 此网络使用蜂窝传输
onLost:网络已经断开， 318
onAvailable: 网络已连接：319
onCapabilitiesChanged: 此网络使用Wi-Fi传输
```
NetworkCallback有很多回调方法，常用的为上面几个，其他的这里不再一一罗列，这里解释一点，上面的蜂窝传输指的就是我们常说的移动数据。


**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)

