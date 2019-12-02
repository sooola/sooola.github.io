---
title: 适配器模式笔记
date: 2016-07-03 14:36:27
tags: [android , 设计模式]
---

#适配器模式
定义：将一个类的接口，转换成客户端期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。  
Example:如我们有一个鸭子对象，现在假设缺少鸭子，想要用一些火鸡来冒充,通过适配器就可以把火鸡封装成像鸭子。  
鸭子接口

    public interface Duck {
        public void quack();        //叫
        public void fly();            //会飞
    }

鸭子对象

    public class MallardDuck implements Duck {
    
        @Override
        public void quack() {
            System.out.println("Quack");
        }
    
        @Override
        public void fly() {
            System.out.println("I'm flying");
        }
    }
火鸡接口

    public interface Turkey {
    
        public void gobble();        //不同的叫法
        public void fly();            //会飞
    }
火鸡对象

    public class WildTurkey implements Turkey {
    
        @Override
        public void gobble() {
            System.out.println("gobble");
        }
    
        @Override
        public void fly() {
            System.out.println("I'm flying a short distance");
        }
    }
火鸡适配器

    public class TurkeyAdapter implements Duck {
        Turkey turkey;
        
        public TurkeyAdapter(Turkey turkey) {
            this.turkey = turkey;
        }
        
        @Override
        public void quack() {
            turkey.gobble();        //假装鸭子叫
        }
    
        @Override
        public void fly() {
            //因为飞的距离短，连续调用5次
            for(int i = 0 ;i < 5 ; i++){
                turkey.fly();
            }
        }
    }
要用火鸡适配鸭子过程如下：  

- 适配器要实现想转换成的类型接口(鸭子接口)。
- 要取得适配对象的引用。
- 实现接口中所有的方法。

最后测试一下:  
    public class DuckTestDrive {
    
        public static void main(String[] args) {
            MallardDuck duck = new MallardDuck();
            
            WildTurkey turkey = new WildTurkey();
            Duck turkeyAdapter = new TurkeyAdapter(turkey);
            System.out.println("The Turkey says...");
            
            turkey.gobble();
            turkey.fly();
            
            System.out.println("\n The Duck say...");
            testDuck(duck);
            System.out.println("\nThe TurkeyAdapter say...");
            testDuck(turkeyAdapter);
        }
        
        static void testDuck(Duck duck){
            duck.quack();
            duck.fly();
        }
    }
测试结果

    The Turkey says...
    gobble
    I'm flying a short distance
    
     The Duck say...
    Quack
    I'm flying
    
    The TurkeyAdapter say...
    gobble
    I'm flying a short distance
    I'm flying a short distance
    I'm flying a short distance
    I'm flying a short distance
    I'm flying a short distance

可以知道客户使用适配模式的过程如下：  

- 客户通过目标接口调用适配器的方法对适配器发出请求
- 适配器使用被适配者接口把请求转换成适配者的一个或多个调用接口。
- 客户接收到调用的结果，但不知道这是适配器起转换作用。  

##总结
- 当需要使用一个现有的类而其接口并不符合你的需求时，使用适配器
- 适配器改变接口以符合客户的期望
- 适配器将一个对象包装起来以改变其接口