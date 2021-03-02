---
layout:     post
title:      TextUtils,你该认真了解一下了
subtitle:   
date:       2019-11-28
author:     dingqiang.l
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - Java
---
&ensp;&ensp;&ensp;&ensp;平时写项目，一般都会对文本进行一些简单的操作，比如判空、截取、替换等。看到这里，我们首先想到的一般是Java中的一些常用API，如isEmpty() 、substring(beginIndex)、replace(oldChar, newChar),这些方法固然是在我们使用到的时候首先想到的，但是今天我要介绍的是Android中的TextUtils，看看它到底有什么值得我们去使用的,直接开始吧。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128104114851.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)### concat(CharSequence... text) ###
>描述：多个字符串拼接，内部维护着SpannableStringBuilder和StringBuilder。

    TextUtils.concat("Andy","Bob","Charles")；// AndyBobCharles

### isEmpty(@Nullable CharSequence str) ###
>描述：字符串是或否为null或长度是否为0。

    字符串是或否为null或长度是否为0
    TextUtils.isEmpty("Andy")；// false


### equals(CharSequence a, CharSequence b) ###
>描述：判断字符串a、b是否相等。

    TextUtils.equals("a","b");//false


### expandTemplate(CharSequence template, CharSequence... values) ###
>描述：将模板字符串中的 \^1、^2...等替换成values，需要注意的是values个数不能超过9个，否则会报IllegalArgumentException。

    String template = "Replace instances of ^1, ^2";
    TextUtils.expandTemplate(template,"Andy","Bob");// Replace instances of Andy, Bob

### getChars(CharSequence s, int start, int end,char[] dest, int destoff) ###
>描述：将源字符串s的[start,end)区间中获取对应位置的字符，赋给目标字从destoff位置开始符数组dest。
>s:源字符串
>start：源字符串开始位置,大于等于start
>end：源字符串结束位置，小于end
>dest：目标字符数组
>destoff：目标字符数组起始位置

    String oldStr = "abcd";
    char[] newChar = new char[oldStr.length()];
    TextUtils.getChars(oldStr,0,3,newChar,0);//将abcd从0开始到2结束获取字符，赋给从0开始的newChar
    //[a,b,c,d, ]
    
### getOffsetAfter(CharSequence text, int offset) ###
>描述：获取text在offset位置之后一位的偏移，如果offset等于text的长度，则返回text的length

    TextUtils.getOffsetAfter("abc",1)；//2,获取text在offset位置之后一位的偏移
    TextUtils.getOffsetBefore(oldStr,1)//0,获取text在offset位置之前一位的偏移

    
### getTrimmedLength(CharSequence s) ###
>描述：获取s去掉头尾空格之后的长度，类似String的trim()

    TextUtils.getTrimmedLength("abc")；//3，没有空格
    TextUtils.getTrimmedLength(" abc ")；//3,头尾有空格
    
### htmlEncode(String s) ###
>描述：使用HTML编码的字符串

    String htmlEncode = "<html><body>hello world</body></html>";
    TextUtils.htmlEncode(htmlEncode)；//转换后的字符串

### indexOf(CharSequence s, CharSequence needle)  ###
>描述：字符串needle在源字符串s中第一次出现的位置,若匹配不到则返回-1,indexOf(CharSequence s, char ch)同理

    String indexOf = "hello world world";
    TextUtils.indexOf(indexOf,"w");//6
    TextUtils.indexOf(indexOf,"w",2);//6
    TextUtils.indexOf(indexOf,"w",8);//12
    TextUtils.indexOf(indexOf,"w",13);//-1
    TextUtils.indexOf(indexOf,"w",2,7);//6
### lastIndexOf(CharSequence s, char ch)  ###
>描述：字符ch在源字符串s中倒数第一次出现的位置,若匹配不到则返回-1,注意ch只能是char

    TextUtils.indexOf(indexOf,"w");//12
    TextUtils.indexOf(indexOf,"w",2);// -1
    TextUtils.indexOf(indexOf,"w",8);//6
    TextUtils.indexOf(indexOf,"w",13);//12
    TextUtils.indexOf(indexOf,"w",2,7);//6

### isDigitsOnly(CharSequence str)  ###
>描述：字符串str是否只包含数字

    TextUtils.isDigitsOnly("123456");//true
    TextUtils.isDigitsOnly("+123456");//false
    TextUtils.isDigitsOnly("-123456");//false
    TextUtils.isDigitsOnly("123456.78");//false
    TextUtils.isDigitsOnly("a123456");//false

### isGraphic(CharSequence str)   ###
>描述：str是否是可打印字符，可以简单理解为是否是特殊字符

    TextUtils.isGraphic("\n");//false
    TextUtils.isGraphic("hello world");//true

### join(@NonNull CharSequence delimiter, @NonNull Iterable tokens)   ###
>描述：返回包含由定界符连接的标记的字符串

    TextUtils.join("<->", new String[]{"Andy", "Bob", "Charles", "David"});//Andy<->Bob<->Charles<->David
    TextUtils.join("<->", Arrays.asList("Andy", "Bob", "Charles", "David"));//Andy<->Bob<->Charles<->David

### regionMatches(CharSequence one, int toffset,CharSequence two, int ooffset,int len)   ###
>描述：在one的第toffset个位置获取len个长度与two的第ooffset个位置获取len个长度进行匹配

    CharSequence one = "hello world";
    CharSequence two = "hi world";
    TextUtils.regionMatches(one, 2,two,0,1);//false,one中的l与two中的w匹配
    TextUtils.regionMatches(one, 0,two,0,1);//true,one中的h与two中的h匹配
    TextUtils.regionMatches(one, 5,two,2,5);//true,one中的world与two中的world匹配

### CharSequence replace(CharSequence template, String[] sources, CharSequence[] destinations)   ###
>描述：将模板中的sources替换成destinations，若sources在模板不存在，则输入模板里面的

    CharSequence temp = "Andy, Bob,Charles, David";
    String[] sources = new String[]{"Charles","1234"};
    String[] destinations = new String[]{"Hello","World"};
    TextUtils.replace(temp, sources,destinations);//Andy, Bob,Hello, World

### split(String text, String expression)  ###
>描述：将字符串text根据表达式或Pattern拆分成一个新的字符串数组

    TextUtils.split("Andy, Bob,Charles, David", ",");//{"Andy", "Bob", "Charles", "David"}
    TextUtils.split("Andy, Bob,Charles, David", Pattern.compile(","));//{"Andy", "Bob", "Charles", "David"}

### stringOrSpannedString(CharSequence source)  ###
>描述：将传入的source进行转换，若source是SpannedString，则直接返回source；若source是Spanned，则返回Spanned；否则返回source.toString()

    TextUtils.stringOrSpannedString("Andy,Bob,Charles,David")；// Andy, Bob,Charles, David

### substring(CharSequence source, int start, int end)  ###
>描述：字符串截取，类似String.substring()

    TextUtils.substring("abcdef",1,4);//bcd,start <= sublen < end

&ensp;&ensp;&ensp;&ensp;到此，TextUtils类中的常用方法就基本介绍完了。<br><br>
&ensp;&ensp;&ensp;&ensp;**欢迎关注我的个人微信公众号，优了个秀和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191203145342715.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xkcTEzMDI2ODc2OTU2,size_16,color_FFFFFF,t_70)

