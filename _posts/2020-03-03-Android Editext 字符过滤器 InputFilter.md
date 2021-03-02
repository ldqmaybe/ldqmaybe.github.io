---
layout:     post
title:      Android Editext 字符过滤器 InputFilter
subtitle:   
date:       2020-03-03
author:     dingqiang.l
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - Android
    - 工具
    - 开发技巧
---

> 今天测试小妹揣着机子过来又是一顿毒打：这个证件号输入框输入了一些非法字符，是输入不进去了，可为什么没有点提示啊，现在输入的emoji，界面一动不动的，快优化一下。好吧，疏忽了，面壁中。。。

之前呢，只是对Editext做了很简单的处理，像默认数字键盘 `inputType="number"` 和 `android:digits="xxx"`,digits是能控制输入类型了，但是没有提示啊，还得做一处理。

这时候，使用InputFilter就是一个比较好的解决方案了，那这个东东又是个什么玩意儿呢？我们继续往下看。
#### InputFilter API介绍
InputFilter 只有 filter一个方法，很简单。因为InputFilter是一个接口，所以我们使用的时候需要它的一个实现类。

        /**
         *
         * @param source 输入的源字符
         * @param start 输入的起始位置
         * @param end 输入的结束位置
         * @param dest 当前输入框中的内容
         * @param dstart 输入框中的内容被替换的起始位置
         * @param dend 输入框中的内容被替换的结束位置
         * @return
         */
        @Override
        public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {

#### 怎么使用
使用也很简单，`Editext`调用`setFilters`方法即可。剩下的就需要我们在filter方法中写具体的逻辑了。

	etInputFilter = findViewById(R.id.et);
    etInputFilter.setFilters(new InputFilter[]{new InputFilter() {
        @Override
        public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
            return null;
        }
    }});

#### 举几个例子
下面演示几个比较常用的例子。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200227165915278.jpg#pic_center)

##### 1、过滤表情
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200303160614707.png#pic_center)

       	etInputFilter = findViewById(R.id.et);
        etInputFilter.setFilters(new InputFilter[]{new InputFilter() {
            @Override
            public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
                if (source.length() <= 0) {
                    return null;
                }
                //过滤emoji表情包
                if (isMatcher(source.toString(), EMOJI)) {
                    ToastUtils.showShortToast("不允许输入表情");
                    return "";
                }
                return null;
            }
        }});
##### 2、过滤中文

        etInputFilter = findViewById(R.id.et);
        etInputFilter.setFilters(new InputFilter[]{new InputFilter() {
            @Override
            public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
                if (source.length() <= 0) {
                    return null;
                }
                //过滤中文
                if (!isMatcher(source.toString(), MATCHER_EN)) {
                    ToastUtils.showShortToast("不允许输入中文");
                    return "";
                }
                return null;
            }
        }});
##### 3、输入长度限制
        etInputFilter = findViewById(R.id.et);
        etInputFilter.setFilters(new InputFilter[]{new InputFilter() {
            @Override
            public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
                if (source.length() <= 0) {
                    return null;
                }
                //输入长度限制
                int maxLen = 6;//最大长度
                if (source.length() + dest.length() > maxLen) {
                    ToastUtils.showShortToast("输入长度超限");
                    return "";
                }
                return null;
            }
        }});

##### 4、不允许输入空格
       etInputFilter = findViewById(R.id.et);
        etInputFilter.setFilters(new InputFilter[]{new InputFilter() {
            @Override
            public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
                if (source.length() <= 0) {
                    return null;
                }
                //不允许空格
                if (" ".contentEquals(source)) {
                    ToastUtils.showShortToast("不允许输入空格");
                    return "";
                }
                return null;
            }
        }});

以上是几个比较简单也比较常见的例子，当让还有很多场景，比如Email、手机号、二代身份证等等，这就需要我们平时遇到的时候处理了。


**欢迎关注我的个人微信公众号，【优了个秀】和你每天进步一点点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191219165723960.jpg#pic_center =200x200)
