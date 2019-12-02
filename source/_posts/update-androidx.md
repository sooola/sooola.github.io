---
title: Android support项目升级到androidx
date: 2019-9-30 14:51:27
tags: android
---

要运行一个旧的项目，但是我的Android Studio 和gradle都已经升级了，没办法，只能把项目强行升级到androidx，在androidx中，以前所有的support兼容包都被去掉了，下面记录下可能出错的地方，和解决的方法。

#### 1.Android Studio自带可以把项目升级为Androidx

![1.png](https://upload-images.jianshu.io/upload_images/1816215-3addfa120b0d4c90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在这一步操作后，部分gradle和xml内容已经替换为androidx，但是项目还是会报错，部分java与xml文件没有替换需要我们逐一检查

#### 2\. 在gradle.properties文件添加
android.useAndroidX=true
android.enableJetifier=true

#### 3.修改java 类引用

| 修改前                                       | 修改后                                                |
| -------------------------------------------- | ----------------------------------------------------- |
| import android.app.FragmentManager;          | androidx.fragment.app.FragmentManager                 |
| android.support.v7.widget.Toolbar            | androidx.appcompat.widget.Toolbar                     |
| android.support.v7.app.AppCompatActivity     | androidx.appcompat.app.AppCompatActivity              |
| android.support.v4.widget.SwipeRefreshLayout | androidx.swiperefreshlayout.widget.SwipeRefreshLayout |
| android.support.v7.widget.RecyclerView       | androidx.recyclerview.widget.RecyclerView             |

当然上面还是有很多类没有列出来，上面没有列出来的类，可以在Android develop中查找，如下

![2.png](https://upload-images.jianshu.io/upload_images/1816215-e9a834713336008e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 4.布局xml控件修改

与3中相同，不再重复

以上就已经替换完成了，如有遗漏的地方欢迎留言