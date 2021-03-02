---
layout:     post
title:      Android Jetpack之Navigation
subtitle:   
date:       2020-07-01
author:     dingqiang.l
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Android
---
## 开始

此篇章简单介绍Android Jetpack之Navigation组件的使用，概要介绍请参考[Android开发者文档](https://developer.android.google.cn/guide/navigation/)

## 配置环境

首先新建一个工程，然后创建Activity，这里AS提供了一些模板，我们可以选Navigation相应的模板，我这里为了熟悉编写流程，所以就建了个空的Activity。

要使用Navigation组件，Google方面有些限制：**AS必须要3.3及以上的版本**，然后在module下的build.gradle添加依赖，这里用的是Java的

```java
  def nav_version = "2.3.0"
  // Java language implementation
  implementation "androidx.navigation:navigation-fragment:$nav_version"
  implementation "androidx.navigation:navigation-ui:$nav_version"
```

这里简单做个登录和注册页面之间的跳转，分别新建一个LoginFragment、RegisterFragment和LoginActivity（作为登录注册Fragment的载体），activity_login简单如下

```java
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment
        android:id="@+id/my_nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        app:navGraph="@navigation/login_navigation"
        app:defaultNavHost="true"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

>   - **android:name** ：包含了NavHostFragment ，与导航图相关联
>   - **app:navGraph** ：NavHostFragment 与导航图相关联工作由它完成，在navigation中完成到目的视图导航
>   - **app:defaultNavHost**：是否被系统返回键拦截

登录注册界面如下，用户点注册按钮跳转到注册Fragment；输入账号密码，点登录跳转到主页；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701115939218.png#pic_center)
新建navigation导航文件，在res下创建navigation文件夹并且新建login_navigation.xml文件，内容如下![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701144701360.png)
到这里，在我们打开LoginActivity时，app:startDestination会指定到id位login的目的地，也就是LoginFragment，这里fragment标签下的id和name是必须的，tools:layout可选，供我们方便预览的。当然我们还可以设置action，可以设置进场退场动画啥的。

我这里的登录和注册操作都在LoginFragment和RegisterFragment中，LoginActivity没有任何逻辑

```java
public class LoginActivity extends AppCompatActivity {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
    }
}
```

点击注册按钮跳转注册页

```java
loginRegist.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View view) {
        NavOptions navOptions = new NavOptions.Builder()
                .setEnterAnim(R.anim.common_slide_in_right)
                .setExitAnim(R.anim.common_slide_out_left)
                .setPopEnterAnim(R.anim.common_slide_in_left)
                .setPopExitAnim(R.anim.common_slide_out_right)
                .build();
        NavController navController = Navigation.findNavController(view);
        navController.navigate(R.id.register, null, navOptions);
    }
});
```
通过导航控制器NavController指定目的地，例子中还使用NavOptions简单配置了进场出场动画，效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701152747822.gif#pic_center =180x320)

## Navigation之间参数传递

### Bundle方式传递

以往，我们可以通过Bundle实现Activity/Fragment之间进行参数传递

```java
 //传递
 Bundle bundle = new Bundle();
 bundle.putString("loginTag", "from LoginFragment");
 navController.navigate(R.id.register, bundle, navOptions);
 //接收
 Bundle bundle = getArguments();
 if (bundle != null) {
     String loginTag = bundle.getString("loginTag");
 }
```

### Safe Args方式传递
首先需要在我们project下的build.gradle中添加插件，并且在module下的build.gradle中引用

```java
//project下的build.gradle
def nav_version = "2.3.0"
classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
//module下的build.gradle
apply plugin: "androidx.navigation.safeargs"
```
然后再navigation.xml中添加argument，注意string类型需要单引号，否则编译报错

```java
<fragment
    android:id="@+id/login"
    android:name="com.smart.jetpack.ui.login.LoginFragment"
    tools:layout="@layout/fragment_login">
    <argument
        android:name="loginTag"
        android:defaultValue='"defaultValue"'
        app:argType="string" />
</fragment>
```
Make Project 之后会生成LoginFragmentArgs，我们可以通过该类完成数据传递。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701155850751.png#pic_center))

```java
//传递参数
NavController navController = Navigation.findNavController(view);
Bundle bundle = new LoginFragmentArgs.Builder().setLoginTag("from LoginFragment").build().toBundle();
bundle.putString("loginTag", "from LoginFragment");
navController.navigate(R.id.register, bundle, navOptions);

//接收参数
Bundle bundle = getArguments();
if (bundle != null) {
    String loginTag = LoginFragmentArgs.fromBundle(bundle).getLoginTag();
    usernameEditText.setText(loginTag);
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701161131975.gif#pic_center =180x320)
可以看到，这种方式也一点问题都没有。在这里可能会有人会疑惑，用Bundle不是更简单些吗，Safe Args 传递方式不仅编码复杂，还要安装它的插件，不会太麻烦吗？既然是Google推荐，那当然是有道理的这里不再啰嗦，直接贴上官方说明![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701161802607.png#pic_center)
这里只是简单介绍了跳转和数据传递的使用，更丰富的用法有兴趣的可以深入去了解
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200701162320701.png#pic_center)
[源码地址...](https://github.com/ldqmaybe/smart_parent)

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
