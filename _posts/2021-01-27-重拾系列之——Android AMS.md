---
layout:     post
title:      重拾系列之——Android AMS
subtitle:   翻一翻快遗忘了的AMS
date:       2021-01-27
author:     dingqiang.l
header-img: img/post-bg-coffee.jpeg
catalog: true
tags:
    - Android
---

## 概述
AMS是系统的引导服务，管理着应用进程的启动、切换和调度，管理四大组件的启动和生命周期。

## 初始化
AMS的初始化是在SystemServer中完成的，关于SystemServer，在上一篇[重拾系列之——Android系统启动流程](https://blog.csdn.net/ldq13026876956/article/details/112800308)中有做过简单的介绍。

在SystemServer中

```java
private void run() {
            createSystemContext();
            // Create the system service manager.
            mSystemServiceManager = new SystemServiceManager(mSystemContext);
            mSystemServiceManager.setStartInfo(mRuntimeRestart,
                    mRuntimeStartElapsedTime, mRuntimeStartUptime);
            LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
   
            traceBeginAndSlog("StartServices");
            startBootstrapServices();
            startCoreServices();
            startOtherServices();
    }
```



先在createSystemContext()中创建了两个context，分别是mSystemContext和systemUiContext,其中mSystemContext会在初始化SystemServiceManager时传入。

ActivityThread.systemMain()中绑定SystemServer进程，注意这里的attach传的是true；如果启动的是应用进程，attach传的是false（在ActivityThread.main()中可看到），方法中attachApplication。


```java
private void createSystemContext() {
    ActivityThread activityThread = ActivityThread.systemMain();
    mSystemContext = activityThread.getSystemContext();
    mSystemContext.setTheme(DEFAULT_SYSTEM_THEME);

    final Context systemUiContext = activityThread.getSystemUiContext();
    systemUiContext.setTheme(DEFAULT_SYSTEM_THEME);
}

 #ActivityThread.systemMain()
 public static ActivityThread systemMain() {
     //......
     ActivityThread thread = new ActivityThread();
     thread.attach(true, 0);
     return thread;
 }
```

AMS就在启动引导服务 startBootstrapServices()中启动。

```java
    private void startBootstrapServices() {
        // Activity manager runs the show.
        mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        mActivityManagerService.setInstaller(installer);
	}
```
startService()最终在SystemServiceManager中调用了Lifecycle的onStart()方法，Lifecycle是AMS的内部类，在onStart()方法调用了AMS的start()方法。

```java
public static final class Lifecycle extends SystemService {
    private final ActivityManagerService mService;

    public Lifecycle(Context context) {
        super(context);
        mService = new ActivityManagerService(context);
    }

    @Override
    public void onStart() {
        mService.start();
    }

    @Override
    public void onBootPhase(int phase) {
        mService.mBootPhase = phase;
        if (phase == PHASE_SYSTEM_SERVICES_READY) {
            mService.mBatteryStatsService.systemServicesReady();
            mService.mServices.systemServicesReady();
        }
    }

    @Override
    public void onCleanupUser(int userId) {
        mService.mBatteryStatsService.onCleanupUser(userId);
    }

    public ActivityManagerService getService() {
        return mService;
    }
}
```
这里看了一圈都没看到哪里有初始化Lifecycle的地方，乍一看，原来是在SystemServiceManager的startService(Class<T> serviceClass)中通过反射调用Lifecycle的构造方法。

```java
public <T extends SystemService> T startService(Class<T> serviceClass) {
    try {
        final String name = serviceClass.getName();
        // Create the service.
        final T service;
        Constructor<T> constructor = serviceClass.getConstructor(Context.class);
        service = constructor.newInstance(mContext);
        startService(service);
        return service;
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
    }
}
```

回到上一步，在new ActivityManagerService(context)中，主要做了创建处理AMS消息的Handler，创建四大组件相关管理对象，还有内存、权限、电池等监控。代码太多这里就不贴出来了。

整体来看

```java
//1、启动服务
mActivityManagerService = mSystemServiceManager.startService(ActivityManagerService.Lifecycle.class).getService();
//2、SystemServiceManager传入AMS
mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
//3、Installer传入AMS，应用安装相关
mActivityManagerService.setInstaller(installer);
//4、初始化电源相关服务
mActivityManagerService.initPowerManagement();
 //5、设置系统进程
mActivityManagerService.setSystemProcess();
//6、
mActivityManagerService.setUsageStatsManager(LocalServices.getService(UsageStatsManagerInternal.class));
//7、初始化系统Provider相关
mActivityManagerService.installSystemProviders();
//8、传入WindowManager
mActivityManagerService.setWindowManager(wm);
//9、
mActivityManagerService.enterSafeMode();
//10、
mActivityManagerService.showSafeModeOverlay();
//11、在这里启动Launcher
mActivityManagerService.systemReady(...)
//12、
mActivityManagerService.startObservingNativeCrashes()
```

## 最后
附上思维导图一张
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127112606886.png)

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)


