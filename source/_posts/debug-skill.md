---
title: Android Studio Debug调试技巧
date: 2019-07-15 14:46:17
tags: [android,debug]
---

在我们日常开发中，debug是我们必不可少的一种能力，不仅可以帮助我们快速判断程序的错误，且在看源码理解思路的时候也有很大的作用，下面总结Android开发中常用的debug技巧。

### 1.单步运行(快捷键Shift + F7)

单步运行是最基本的调试方式，在添加断点之后逐步运行，直到程序结束。

![1563118716048.png](https://upload-images.jianshu.io/upload_images/1816215-4f0ec4746ed58fb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如在list第2行处添加断点，程序将继续向下逐步执行，到System.out，程序结束。

### 2.Step Into(快捷键F7)

当运行到getAllCity方法，想进入方法内部调试时，可以用Step Into，则可进入到方法内部调试。

![1563119142818.png](https://upload-images.jianshu.io/upload_images/1816215-9396b344f2127cb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.Step Out(快捷键Shift + F8)

当在方法内部，如在getAllCity方法里面，不想调试循环，可用Step out跳出方法，继续走主方法步骤。

### 4.跳到下断点（Resume Progrom 快捷键 F9）

当你的程序有多个断点时，Resume Progrom可以帮助你快速切换断点，当已经没有断点时，程序结束

![1563119709529.png](https://upload-images.jianshu.io/upload_images/1816215-74fbe9a243a7b586.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5.条件断点（快捷键 Ctrl + Shift + F8）

条件断点常用在List循环中，当满足一定条件才进入断点调试

如我想在city的循环中，在list = Tokyo 才进入循环调试，可以在断点处右键（或快捷键 Ctrl + Shift + F8），输入条件，条件为java表达式

![GIF.gif](https://upload-images.jianshu.io/upload_images/1816215-0fe4c8c39b07a75f.gif?imageMogr2/auto-orient/strip)


可以看到，即使List里面有London,Denver ，还是在list 为 Tokyo时，才进入断点

### 6.表达式求值（快捷键Alt + F8）

在调试的过程中，在断点处，如果你想查看某个值的结果是否跟你的预期想同，可以用表达式求值去查看

![1563121898627.png](https://upload-images.jianshu.io/upload_images/1816215-5eb27eefa5b71b55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如，断点在cityList第三行处，你希望查看cityList的值，这时可右键Evaluate Expression 即可查看cityList的值

还可以调用表达式的方法，如cityList.size()，动态查看此时内容是否与预期相同

![1563122166543.png](https://upload-images.jianshu.io/upload_images/1816215-a82b9d980fc2d4c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 7 跳转到指定行（快捷键 Alt + F9）

可快速跳转到指定行数，中间代码默认执行

![1563122371084.png](https://upload-images.jianshu.io/upload_images/1816215-20bebc630e7a903e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如想快速跳转到getAllCity方法行，则需要光标落在该行上，下方按钮，或快捷键Alt +F9 则可快速跳转到此行

### 8 设置值（快捷键F2）

在调试过程中，还可以动态设置值，来测试程序
![GIF2.gif](https://upload-images.jianshu.io/upload_images/1816215-3e45fdbb0e0bcde4.gif?imageMogr2/auto-orient/strip)



如，想测试list为空的情况，程序执行是否正确，下面cityList处，按下F2，设置值为null，可见，cityList虽然上面已经被赋值，但是还是动态改变为null。

参考资料：
[https://www.imooc.com/video/16228](https://www.imooc.com/video/16228)