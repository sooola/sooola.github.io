---
title: Android Studio打造自己的Live Templates
date: 2019-12-02 11:39:28
tags: android
---

## **什么是Live Templates**

以我的理解，Live Templates是代码模板的快捷定义，如常见的，你定义了findViewById(R.id.)的模块代码的模板为fvd，那么以后我们按fv就可以快速的调用定义好的模板了！熟练的使用Live Templates将大大提到我们的编程效率！这个例子下面也有讲到，那么先看看怎么定义一个Live Templates

## 设置位置

setting->Editor->Live Templates

![img](https://upload-images.jianshu.io/upload_images/1816215-02eae4a13244be65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **创建一个Live Templates**

![img](https://upload-images.jianshu.io/upload_images/1816215-44c9299bcdcb4719.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

击点右边的 + 号出现如上界面

如我们想把findViewbyid创建一个Live Templates，可以先把代码cv到Template text里，然后用$...$ 替换等待输入的变量替换后如下

> ($cast$) findViewById(R.id.$res$);

记得要先点击下面的Define定义这个Templates使用的地方，一般全部勾选就可以了

然后就可以点击edit variables对等输入变量进行编辑了

![img](https://upload-images.jianshu.io/upload_images/1816215-f8fa26840cd43ad2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Name  你定义的所有$..$

Expression  为待输入变量

Default value  赋值默认值

Skip if defined是否跳过编辑

------

上面Expression使用**expectedType()**就可以根据我们前面定义的View自动转换，当前Expression还有很多函数可以使用具体可以查看如下官方说明：

https://www.jetbrains.com/help/idea/2016.1/live-template-variables.html

结果如下:

![img](https://upload-images.jianshu.io/upload_images/1816215-bc7f5f1857c05d9e.gif?imageMogr2/auto-orient/strip)

在定义一个常见的LOG

![img](https://upload-images.jianshu.io/upload_images/1816215-076e7d046787085e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

基本跟findviewById差不多，只是Expression改为className，OK结果如下

![img](https://upload-images.jianshu.io/upload_images/1816215-58c7538db2aeed1d.gif?imageMogr2/auto-orient/strip)

## **快速导入一个****Live Templates**

如果你跟我一样那么懒，自己慢慢加就太慢了，大神早已写好了一些常用的Live Templates，直接导入使用就可以了，地址如下：

https://github.com/keyboardsurfer/idea-live-templates

直接复制到你的 Android Studio config\templates目录下

我的在这里C:\Users\Administrator\.AndroidStudio2.0\config\templates

按下ctrl + J 就可以快速查看Live Templates了

![img](https://upload-images.jianshu.io/upload_images/1816215-bde55469c30fdb50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

------

参考文章:

http://blog.csdn.net/DesmondJ/article/details/47017205

https://www.bignerdranch.com/blog/android-studio-live-templates/

https://github.com/keyboardsurfer/idea-live-templates

https://www.jetbrains.com/help/idea/2016.1/live-template-variables.html?origin=old_help


