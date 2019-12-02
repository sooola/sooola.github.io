---
title: java四大引用类型
date: 2019-08-27 14:49:50
tags: java
---

每种编程语言都有自己操作内存中元素的方式，在C和C++中是通过指针，而在java中是通过引用，在Java中一切都是对象，但我们操作的实际是对象的一个引用，java将引用分为了四种类型，强引用、软引用、弱引用、虚引用。

### 强引用

java默认new 对象则为强引用，如

```java
StringBuffer buffer = new StringBuffer();
```

上面创建了一个StringBuffer对象，并将这个对象的强引用存到变量buffer中。如果一个对象通过一串强引用链接可到达，即使内存不足，也不会回收该对象。



### 软引用

用来描述一些非必须，但仍有用的对象。内存足够时，软引用对象不会被回收，只有在内存不足时，系统会回收软引用对象，通常用于实现缓存。

```java
Drawable drawable = new BitmapDrawable(bitmap);
SoftReference<Drawable> soft = new SoftReference<Drawable>(drawable);
if(soft!=null){
	view.setImageResource(soft.get())
}
        
```

当需要加载大图时，可以使用软引用，通过get()方法，获取图片对象，如果当前内存不足，则软引用会被回收，避免内存溢出发生。

### 弱引用

随时可能被垃圾回收器回收，无论内存是否足够，只要JVM开始进行垃圾回收，那些被弱引用关联的对象都会被回收。

```java
String str = new String("abc");    
WeakReference<String> abcWeakRef = new WeakReference<String>(str);
```

### 虚引用（PhantomReference）

虚引用是所有引用类最脆弱的一个，如果一个对象持有虚引用，那么这个对象随时可能被回收，甚至不能通过get方法来获得其指向的对象。虚引用唯一的作用是，当其指向的对象被回收后，自己被加入到引用队列，用做记录该引用指向的对象已被销毁。



### 总结

Java 4种引用的级别由高到低依次为：  

强引用  >  软引用  >  弱引用  >  虚引用

| 引用类型 | 被垃圾回收时间 | 用途     | 生存时间        |
| -------- | -------------- | -------- | --------------- |
| 强引用   | 从来不会       | 一般状态 | JVM停止运行终止 |
| 软引用   | 在内存不足时   | 缓存     | 内存不足时终止  |
| 弱引用   | 在垃圾回收时   | 缓存     | gc运行后终止    |