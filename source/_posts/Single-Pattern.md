---
title: 单例模式学习笔记
date: 2016-06-26 14:38:41
tags: [android , 设计模式]
---

## 单例模式的作用

 有些对象的实例只需要一个，如缓存，数据库等，如果有多个实例就会产生许多问题，**单例模式确保了类只有一个实例，并给了我们一个全局的访问点**。  
相对与全局变量，利用单例模式可以延迟创建对象，

### 一个简单的单例模式    

    public class Singleton {
        private static Singleton uniqueInstance;
        
        private Singleton(){};
        
        public static Singleton getInstance(){
            if (null == uniqueInstance){
                uniqueInstance = new Singleton();
            }
            return uniqueInstance;
        }
    }
单例模式创建步骤如下：  
1. 利用一个**静态**变量来记录Singleton类的唯一实例。  
2. 把构造器声明为私有的，只要在Singleton类内部才可以调用。
3. 用getInstance()方法实例化对象，并返回这个实例。

### 单例模式多线程的问题

如果我们开启多个线程，调用getInstance()方法，可能多有多个线程同时进入这个方法，这样就创建了不同的实例。

- getInstance()改为同步方法  
public static synchronized Singleton getInstance(){}  
通过增加 synchronized到getInstance()方法中，迫使每个线程进入到这个方法之前，需要等待别的线程离开该方法，即不会有两个线程可以同时进入该方法。  
缺点：只有第一次调用创建实例时才需要同步，之后每次这个方法，同步都是一种累赘，一个同步方法可能造成程序执行效率下降100倍。  
##   改善多线程 ##
1.如果调用getInstance()对应用程序不是很关键，则加上synchronized。  
2.如果getInstance()比较频繁，使用"急切"创建实例，而不使用延迟做法。  


    private  static Singleton uniqueInstance = new Singleton();
    
    private Singleton(){};
    
    public static synchronized Singleton getInstance(){
        return uniqueInstance;
    }
使用这个方法时，JVM在加载这个类时马上创建此类的单例实例，JVM保证任何线程访问uniqueInstance静态变量前，一定会创建此实例。  
3.用"双重检查加锁"  
首先检查实例是否已经创建了，如果没有创建，才进行同步，这样只有第一次才会同步。

    private volatile static Singleton uniqueInstance;
    //保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的
    
    private Singleton(){};
    
    public static synchronized Singleton getInstance(){
        if (null == uniqueInstance){
            //如果不存在进入同步
            synchronized (Singleton.class) {
            //再次检查，如果不存在才创建
                if ( null == uniqueInstance){
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }