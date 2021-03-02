---
layout:     post
title:      ButterKnife 8.0以上版本使用
subtitle:   
date:       2017-03-30
author:     dingqiang.l
header-img: 
catalog: true
tags:
    - Android
    - 开发技巧
---
# 写在前面 #
不会偷懒的程序员不是好程序员。平时我们的写布局的时候，无休止的写findViewById(),简直烦不胜烦，但是有了ButterKnife，从此就可以告别这繁琐的工作了。并且配合上Android ButterKnife Zelezny，简直连手写@BindView(R.id.title)TextView title和事件监听都不用了，这么好用又不影响性能的东西不去使用真的对不起自己了。

##### 为什么选择ButterKnife？ #####
>1. 强大的View绑定和Click事件处理功能，简化代码，提升开发效率
2. 方便的处理Adapter里的ViewHolder绑定问题
3. 运行时不会影响APP效率（编译时解析），使用配置方便
4. 代码清晰，可读性强

## 二、配置ButterKnife： ##
### 1、 在Project的build.gradle中添加如下配置：  ###

    buildscript {
      repositories {
        mavenCentral()
      }
      dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
      }
    }
    
###  2、 在Module的build.gradle添加如下配置： ###
    buildscript {
      repositories {
        mavenCentral()
      }
      dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
      }
    }
## 三、ButterKnife应用实践： ##
### 1、 在Activity中使用： ###
    public class ButterKnifeActivity extends Activity {
      @BindView(R.id.title)
      TextView title;
      @BindView(R.id.subtitle)
      TextView subtitle;
      @BindView(R.id.footer)
      TextView footer;
      private Unbinder unbinder;
      @Override
      public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.simple_activity);
	    unbinder = ButterKnife.bind(this);//绑定butterknife
	    // TODO Use fields... 
      }
      @Override
       public void onDestroyView() {
	    super.onDestroyView();
	    unbinder.unbind();
      }
    }
### 2、 在Fragment中使用： ###
    public class ButterKnifeFragment extends Fragment {
      @BindView(R.id.button1)
      Button button1;
      @BindView(R.id.button2)
      Button button2;
      private Unbinder unbinder;
    
      @Override
       public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
		    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
		    unbinder = ButterKnife.bind(this, view);//绑定butterknife
		    // TODO Use fields...
		    return view;
      }
    
      @Override
       public void onDestroyView() {
		    super.onDestroyView();
		    unbinder.unbind();
      }
    }
### 3、 在 ViewHolder中使用： ###
    public class ButterKnifeAdapter extends BaseAdapter {  
    @Override
    public View getView(int position, View view, ViewGroup parent) {
	    ViewHolder holder;
	    if (view != null) {
	      holder = (ViewHolder) view.getTag();
	    } else {
	      view = inflater.inflate(R.layout.whatever, parent, false);
	      holder = new ViewHolder(view);
	      view.setTag(holder);
	    }
	    holder.name.setText("John Doe");
	    //...
	    return view;
      }
    static class ViewHolder {
	    @BindView(R.id.title)
	    TextView name;
	    @BindView(R.id.job_title)
	    TextView jobTitle;
	    public ViewHolder(View view) {
	      ButterKnife.bind(this, view);
	    }
      }
    }
### 4、 单击事件注入使用： ###
#### 1)、单个控件： ####
    @OnClick(R.id.btn_butter_knife)
    public void onButterKnifeBtnClick(View view) {
      Log.e(TAG, "onButterKnifeBtnClick");
    }
#### 2)、多个控件： ####
    @OnClick({R.id.btn_butter_knife, R.id.btn_butter_knife1})
    public void onButterKnifeBtnClick(Button button) {
      Log.e(TAG, "onButterKnifeBtnClick");
    }

    
## PS： ##
如果只是想知道ButterKnife的用法，看完以上内容基本够用了，当然还有更多强大的功能，这里不再描述。如果想知道该框架是怎么实现的，后面会补上。。。
# 写在最后 #
尽管ButterKnife使用反射机制，对我们程序的性能不造成影响，但是我们在使用的时候，绑定了之后一定要记得的onDestroy中调用unbind()方法，避免内存泄露，这也是我们平时编码的时候必须要养成的一个习惯。
