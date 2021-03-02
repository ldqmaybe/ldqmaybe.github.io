---
layout:     post
title:      Java8 Lambda 表达式和方法引用
subtitle:   
date:       2020-01-14
author:     dingqiang.l
header-img: img/post-2020-01-14.jpg
catalog: true
tags:
    - Java
    - 工具
---
> 都说Java8已经发布好几个世纪了，现在才谈它的一些特性，是不是落伍了，老哥我是个慢热的男人，哈哈~~

在我们的项目中使用了JDK1.8了之后，回头再看看一些控件的点击事件，发现setOnClickListener的入参变灰了，鼠标移到参数上可以看到下图显示，大致的意思是**匿名内部类View.OnClickListener可以用Lambda替换**。

我们用Alt+Enter大法看看AS会给我们什么样的建议，发现变成了另外的一种格式，这就是Lambda？Lambda又是个啥？
![在这里插入图片描述](https://img-blog.csdnimg.cn/202001131629063.png)

> Lambda 表达式是 Java 8 发布的最重要新特性，它允许把函数作为一个方法的参数（函数作为参数传递进方法中），使用 Lambda
> 表达式可以使代码变的更加简洁紧凑。

好吧，啥意思咱也看不明白，先来几个常见的例子，看看它能做什么吧。

**Android View的点击事件setOnClickListener：**

```java
//Java8以前的写法
etSendMsg.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.i(TAG, "onClick: Java8以前的写法");
    }
});
//单个参数，带类型
etSendMsg.setOnClickListener((View v) -> {
	//接收一个参数View，无返回值，下同
    Log.i(TAG, "onClick: 单个参数，带类型");
});
//单个参数不用类型，类型推导
etSendMsg.setOnClickListener((v) -> {
    Log.i(TAG, "onClick: 单个参数不用类型，类型推导");
});
//单个参数不用圆括号
etSendMsg.setOnClickListener(v -> {
    Log.i(TAG, "onClick: 单个参数不用圆括号");
});
```

**List的排序：** 

```java
List<String> names = Arrays.asList("a", "b", "g", "c");//源数据
//旧写法
names.sort(new Comparator<String>() {
    @Override
    public int compare(String s1, String anotherString1) {
        return s1.compareTo(anotherString1);
    }
});
//Lambda表达式
names.sort((s, anotherString) -> s.compareTo(anotherString));
for (String name : names) {
    System.out.print(name);
}
//输出：abcg
```
**Thread线程启动：** 

```java
//旧写法
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Lambda表达式");
    }
}).start();
//Lambda表达式
new Thread(() -> System.out.println("Lambda表达式")).start();
```
从上面的例子中可以看到，写法比以前要简洁的多，下面介绍下Lambda的语法。

开始之前我们脑海里得先有个概念**函数式接口**

> **函数式接口：** 有且仅有一个抽象方法的接口，就叫函数式接口；这是Lambda表达式的前提，

#### Lambda 表达式
**语法**：

> `(parameters...) -> todo//执行语句` ，**parameters** 可为空、单个或多个
> or
>  `(parameters...) -> reruen value
> //执行语句`



语法虽简单，但是却大有学问，我们来一一分析：

 **无参无返回：** 即只有空括号（）并没有返回值，`() -> todo//执行语句`，示例：

```java
//旧写法
new Thread(new Runnable() {
    @Override
    public void run() {
    }
}).start();
//lambda表达式
new Thread(() -> System.out.println("Lambda表达式")).start();
```
**无参有返回：** `() -> return value;//执行语句`，如果执行语句只有一条，那么可以省略return关键字

```java
@Test
public void noParamHasReturn() {
    String result1 = addFunction(new Function() {
        @Override
        public String noParamHasReturn() {
            return "这是无参有返回(旧写法)";
        }
    });
    String result2 = addFunction(() -> "这是无参有返回(Lambda写法)");
    System.out.println(result1);//输出：这是无参有返回(旧写法)
    System.out.println(result2);//输出：这是无参有返回(Lambda写法)
}

private String addFunction(Function function) {
    return function.noParamHasReturn();
}

interface Function {
    String noParamHasReturn();
}
```

**有参无返回：**   `(parameter...) -> todo;//执行语句`，可传入单个或多个参数

```java
@Test
public void hasnoParamNoReturn() {
    addFunction("abc", new Function() {
        @Override
        public void hasnoParamNoReturn(String str) {
            System.out.println(str);//旧写法，原样输出:abc
        }
    });
    addFunction("abc", str -> System.out.println(str.toUpperCase()));//Lambda表达式写法，转换为大写:ABC
}

private void addFunction(String str, Function function) {
    function.hasnoParamNoReturn(str);
}

interface Function {
    void hasnoParamNoReturn(String str);
}
```
**有参有返回：**   `(parameter...) -> return value;//执行语句`，可传入单个或多个参数，并且返回value

```java
@Test
public void hasnoParamHasReturn() {
    String result1 = addFunction("abc", new Function() {
        @Override
        public String hasnoParamHasReturn(String str) {
            return str;//旧写法，原样返回，不做任何处理
        }
    });
    String result2 = addFunction("abc",str -> str.toUpperCase());//Lambda，转换为大写
    System.out.println(result1);//输出abc
    System.out.println(result2);//输出ABC
}

private String addFunction(String str, Function function) {
    return function.hasnoParamHasReturn(str);
}

interface Function {
    String hasnoParamHasReturn(String str);
}
```
**小结：** 可以看到Lambda表达式的优点除了简洁还是简洁，并且作为**函数式接口**编程应该是将来的一种趋势；但是还是一些不好的地方的，对于习惯了以前写法的老铁，一开始肯定会认为Lambda的可读性不如以前，而且也一定程度上增加了调试难度。不过老哥认为，既然是趋势，那么我们只有拥抱潮流的大腿了，这不也是我们这行的现状，**学到老活到老**。

#### 方法引用

还记得上面写排序的例子吗，AS检测出compareTo这里有更推荐的写法，意思是可以使用**方法引用**替换Lambda，有强迫症的我继续Alt+Enter，可以看到写法又有了变化
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200113174758308.png)


但是**方法引用**又是个啥玩意儿呢？

> **方法引用：** 在 Lambda 体中的功能，在函数式接口中有抽象方法提供了实现，可以使用方法引用（可以理解为方法引用就是 Lambda 表达式的另外一种表现形式）

方法引用通过 '::'关键字访问类的对象方法、静态方法和构造方法等。

首先我们先准备好一个接口IFunction和一个操作类ToDo

**IFunction接口：** 网上很多大佬都说要加上@FunctionalInterface注解，否则编译时会报`The target type of this expression must be a functional interface`,但是我这里没注解也不报错，不知道为啥，有踩过坑的还请指正。
```java
interface IFunction<T, F> {
    T convert(F from);
}
```
**ToDo类：**
```java
static class ToDo {
    //无参构造
    public ToDo() {}
    //有参构造
    public ToDo(String s) {System.out.println(s);}
    //静态方法
    public static String concatString(String s) {return s.concat(" 方法引用");}
    //对象方法
    public String subString(String s) {return s.substring(2);}
    public void empty() {}
}
```

##### 引用对象方法：

格式：对象引用::非静态方法名。

> 注意事项：
> 1、被引用方法的参数列表和函数式接口中的抽象方法的参数一致
> 2、接口的抽象方法没有返回值，引用的方法可以有返回值也可以没有
> 3、接口的抽象方法有返回值，引用的方法必须有相同类型的返回值

```java
ToDo toDo = new ToDo();
//访问对象方法：
//原始写法
IFunction<String,String> function1 = new IFunction<String, String>() {
    @Override
    public String convert(String s) {
        return toDo.subString(s);
    }
};
//Lambda
IFunction<String,String> function2 = s -> toDo.subString(s);
//方法引用
IFunction<String,String> function3 = toDo::subString;

String result1 = function1.convert("function1");
String result2 = function2.convert("function2");
String result3 = function3.convert("function3");
System.out.println(result1);//输出：nction1
System.out.println(result2);//输出：nction2
System.out.println(result3);//输出：nction3
```
##### 引用静态方法：
格式：类名::静态方法名。注意事项和引用对象方法一致

```java
//引用静态方法
//原始写法
IFunction<String,String> staticFunction = new IFunction<String, String>() {
    @Override
    public String convert(String from) {
        return ToDo.concatString(from);
    }
};
//Lambda
IFunction<String,String> staticFunction1 = s-> ToDo.concatString(s);
//引用静态方法
IFunction<String,String> staticFunction2 = ToDo::concatString;
String staticResult = staticFunction.convert("原始写法");
String staticResult1 = staticFunction1.convert("Lambda");
String staticResult2 = staticFunction2.convert("引用静态方法");
System.out.println(staticResult);//这里是:原始写法
System.out.println(staticResult1);//这里是:Lambda
System.out.println(staticResult2);//这里是:引用静态方法
```
##### 构造方法引用：
格式：类名::new

> 注意事项：被引用的类必须显示声明至少一个构造方法，并且与函数式接口中的抽象方法的参数列表一致

```java
//构造方法引用
//原始写法
IFunction<ToDo,String> origFunction = new IFunction<ToDo, String>() {
    @Override
    public ToDo convert(String from) {
        return new ToDo(from);
    }
};
//Lambda
IFunction<ToDo,String> lambdaFunction = s-> new ToDo(s);
//构造方法引用
IFunction<ToDo,String> constuctFunction = ToDo::new;
origFunction.convert("origFunction");//origFunction
lambdaFunction.convert("Lambda");//Lambda
constuctFunction.convert("这是构造方法引用");//这是构造方法引用
```

除了这几种方法引用外还有**数组构造方法引用、特定类型的方法引用和类中方法调用父类或本类方法引用**，上面的列表排序使用的就是特定类型的方法引用，这里不再一一赘述。

**总结：** Java8 的Lambda表达式和方法引用给人最直观的就是代码变的简洁多了，看习惯后，可读性还真的。。。。无论怎么样，时代在进步，优秀的东西会不断的出现在我们的面前，我们能做唯有学习、学习、再学习。

**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)



