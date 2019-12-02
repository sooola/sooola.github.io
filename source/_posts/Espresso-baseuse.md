---
title: Android Espresso基本使用
date: 2016-09-07 14:29:19
tags: android
---

>上面一篇文章[Android Espresso UI自动测试的搭建](http://www.jianshu.com/p/d850e1453920) 讲了如何搭建环境，下面说一些语法和使用方法。

#找到View
使用[ViewMatcher.class](https://developer.android.com/reference/android/support/test/espresso/matcher/ViewMatchers.html) 里面的方法可以找到你想要的View,如你想找有Hello文字的View，你可以这样使用
 ```  
 onView(withText("Hello"));
 ```
相似的你也可以使用View的资源Id来找到该view
  ```  
onView(withId(R.id.hello));
  ```
当有多个约束条件时，可以使用[Matchers.class](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/Matchers.html)的allof()方法来组合，例子如下：
  ```  
onView(allOf(withText("Hello") ,withId(R.id.hello)));
  ```

#对View执行一些操作
对View操作的代码大概是这样： ```onView(...).perform();```  
在onView中找到这个View后，调用perform()方法进行操作，如点击该View：
```
onView(withId(R.id.hello)).perform(click());
```
也可以执行多个操作在一个perform中如
```
onView(withId(R.id.hello)).perform(click(),clearText());
```

#检查View
使用check()方法来检查View是否符合我们的期望 onView(...).check();
如检查一个View里面是否有文字Hello：
   ``` 
onView(withId(R.id.hello)).check(matches(withText("Hello")));```

更详细的用法可参考下图：![SKODS.png](http://upload-images.jianshu.io/upload_images/1816215-ea7b7c4d6720f0bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

google官方教程:https://developer.android.com/training/testing/ui-testing/espresso-testing.html
和youtube视频也可以参考https://www.youtube.com/watch?v=W8LJjfkTKik  ，需要梯子

列表用法看这里：[Android Espresso Recycle使用和一些坑](http://www.jianshu.com/p/9e4437615b40)
   ```