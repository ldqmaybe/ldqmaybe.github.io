---
layout:     post
title:      Android 手撸一个简易路由Router
subtitle:   
date:       2020-07-31
author:     dingqiang.l
header-img: img/post-bg-hacker.jpg
catalog: true
tags:
    - Android
---

[上一篇文章](https://blog.csdn.net/ldq13026876956/article/details/107683017) 介绍了搭建一个组件化项目，但是有一个痛点是我们必须要解决的，那就是组件之间的通信，用惯了单体结构项目的同学，大多数是使用显示Intent进行Activity的，但是这种做法在组件化项目中是行不通的，毕竟各组件是相互独立的，并不能持有另一个组件的引用。

那只能用别的方法了，路由表是目前比较流行的方案，这里推荐两个比较多人喜欢的路由框架，[Arouter](https://github.com/alibaba/ARouter) 和 [ActivityRouter](https://github.com/mzule/ActivityRouter) ，具体用法和解释网上很多牛人都已经介绍得很清楚了。这里是打造一个自己的专属路由。

## 开始工作
这里使用的是APT技术帮助我们在编译时自动生成我们需要用到相关的类，首先新建三个module ，arouter、nnotation和annotation_compiler，需要注意的是nnotation和annotation_compile在New Module的时候要选的是Java or Kotlin Library 而不是其他。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020073115214073.png#pic_center)


## 编写router
router模块很简单，只有Arouter和IRouter接口 。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731151955460.png#pic_center =400x250)

Arouter是一个单例，内部维护着一个Map用于管理跳转需要的path路径和目标Activity，这里只做简单的处理。IRouter则是提供对外实现的接口，下面会用到。
```java
public void jumpActivity(String key, Bundle bundle) {
    Class<? extends Activity> activityClazz = map.get(key);
    if (null != activityClazz) {
        Intent intent = new Intent(context, activityClazz);
        if (null != bundle) {
            intent.putExtras(bundle);
        }
        context.startActivity(intent);
    }
}
```

## 编写annotation
annotation模块非常简单，这里只创建了一个Router注解，然后其他业务组件（如Login）依赖它

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface Route {
    String value();
}
```
## 编写annotation_compiler
重头戏是这个模块，这里使用几个依赖
```java
    //as3.4以上
    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'
    compileOnly 'com.google.auto.service:auto-service:1.0-rc3'
    //as3.4以下
//    implementation 'com.google.auto.service:auto-service:1.0-rc3'
    implementation 'com.squareup:javapoet:1.13.0'
```
新建个编译处理器AnnotationCompiler继承AbstractProcessor并实现下面4个方法，标记@AutoService(Process.class)注解
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731154802547.png)

>  - getSupportedAnnotationTypes()方式表示我们要处理那个注解，这里是Route这个注解
>  - getSupportedSourceVersion() 声明支持java 版本
>  - init(ProcessingEnvironment processingEnv) 初始化，得到下面需要用到的Filer
>  - process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) 主要代码生成逻辑在这里进行


利用APT技术生成一个工具类，生成的类实现上面的IRouter，并覆写putActivity方法，并且方法内执行Arouter.getInstance().addActivity(key,activityClass)，具体生成如下：

```java
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        Set<? extends Element> elementsAnnotatedWith = roundEnv.getElementsAnnotatedWith(Route.class);

        Map<String, String> map = new HashMap<>();

        for (Element element : elementsAnnotatedWith) {
            TypeElement typeElement = (TypeElement) element;
            String key = typeElement.getAnnotation(Route.class).value();
            String activityName = typeElement.getQualifiedName().toString();
            map.put(key, activityName + ".class");
        }
        if (map.size() > 0) {
            creatClass(map);
        }
        return false;
    }
    
    private void creatClass(Map<String, String> map) {
        try {
            MethodSpec.Builder methodBuilder = MethodSpec.methodBuilder("putActivity")
                    .addModifiers(Modifier.PUBLIC)
                    .returns(void.class);

            for (String key : map.keySet()) {
                String activityName = map.get(key);
                methodBuilder.addStatement("com.smart.router.Arouter.getInstance().addActivity(\"" + key + "\"," + activityName + ")");
            }
            MethodSpec methodSpec = methodBuilder.build();

            ClassName iArouter = ClassName.get("com.smart.router", "IRouter");
            TypeSpec typeSpec = TypeSpec.classBuilder("ActivityUtil" + System.currentTimeMillis())
                    .addModifiers(Modifier.PUBLIC)
                    .addSuperinterface(iArouter)
                    .addMethod(methodSpec)
                    .build();
            JavaFile javaFile = JavaFile.builder("com.smart.utils", typeSpec).build();
            javaFile.writeTo(filer);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

到这里理论上应该能生成ActivityUtil了的，但是不知道是不是AS版本问题，我这里要在main目录下添加这个文件javax.annotation.processing.Processor并加上如下内容才行

```java
com.smart.anotation_compiler.AnnotationCompiler
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020073116032740.png#pic_center)

## 最后工作
回到router模块下的Arouter中还有点事情要搞，在初始化Arouter的时候，通过反射调用IRouter的putActivity方法，将被@Router注解的类添加到路由表中


```java
public void init(Context context) {
    this.context = context;
    List<String> classNames = getClassName("com.smart.utils");
    try {
        for (String className : classNames) {
            Class<?> utilName = Class.forName(className);
            if (IRouter.class.isAssignableFrom(utilName)) {
                IRouter iRouter = (IRouter) utilName.newInstance();
                iRouter.putActivity();
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}

private List<String> getClassName(String packageName) {
    List<String> classList = new ArrayList<>();
    try {
        DexFile dexFile = new DexFile(context.getPackageCodePath());
        Enumeration<String> entries = dexFile.entries();
        while (entries.hasMoreElements()) {
            String className = entries.nextElement();
            if (className.contains(packageName)) {
                classList.add(className);
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return classList;
}
```
然后在Activity上加上@Route(path)即可，如

```java
@Route("main/main")
public class MainActivity extends BasicActivity {}
@Route("login/login")
public class LoginActivity extends BasicActivity {}
@Route("member/member")
public class MemberActivity extends BasicActivity {}
//跳转到MainActivity
Arouter.getInstance().jumpActivity("main/main");
//跳转到LoginActivity 
Arouter.getInstance().jumpActivity("login/login");
//跳转到MemberActivity 
Arouter.getInstance().jumpActivity("member/member");
```
最后看一下效果：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731163438475.gif#pic_center =180x320)

一点问题都没有，但是真正项目中这样使用显然还不行。起码思路有了，慢慢完善就好了。




