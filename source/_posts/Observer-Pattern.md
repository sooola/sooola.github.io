---
title: 观察者模式笔记
date: 2017-01-04 14:33:11
tags: [android, 设计模式]
---

# 什么是观察者模式
现实生活中的例子：  

1. 报社的业务是出版报纸  
2. 向某家报社订阅报纸，只要他们有新报纸出版，就会给你送来。只要你是他们的订户，就会一直收到报纸。  
3. 当你不想再看报纸时，取消订阅，他们就不会再送新报纸来。  
4. 只要报纸还在运营，就会一直有人向他们订阅报纸或取消报纸。  

# 定义观察者模式
**观察值模式**  定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

#观察者模式类图
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-5913b6364257b192.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 实例
**Subject.class 主题接口**

    public interface Subject{
    
    public void registerObserver (Observer o);    //需要一个观察者用与注册
    
    public void removeObserver(Observer o);
    
    public void notifyObservers();}

**WeatherData.class 主题实现类**

    public class WeatherData implements Subject {        //实现主题接口
    
    private ArrayList<Observer> observers;    //用来记录观察者
    private float temperature;
    private float humidity;
    private float pressure;
    
    public WeatherData() {
        observers = new ArrayList<>();
    }
    
    @Override
    public void registerObserver(Observer o) {
        observers.add(o);    //注册观察者，即添加到List后
    }
    
    @Override
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);    //移除观察者
        if (i >=0){
            observers.remove(i);
        }
    }
    
    @Override
    public void notifyObservers() {
        //遍历每一个观察者，调用观察则的update方法
        for (int i = 0 ; i < observers.size() ; i++){
            Observer observer = observers.get(i);
            observer.update(WeatherData.this);
        }
    }
    
    public void measurementsChanged(){
        notifyObservers();
    }
    
    public void setMeasurements(float temperature , float humidity , float pressure){
        //状态改变，调用measurementsChanged，后调用notifyObservers，通知观察则更新
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
    }

**观察者接口**

    public interface Observer {
    
    public void update(Subject subject);}

**观察者实现类**

    public class CurrentConditionsDisplay implements Observer ,DisplayElement {
    //实现Observer 可获得通知
    
    private float tempeature;
    private float humidity;
    private Subject weatherData;
    
    public CurrentConditionsDisplay(WeatherData weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }
    
    @Override
    public void display() {
        System.out.println("Current conditions: " + tempeature + "F degrees and " + humidity + 
                "% humidity");
    }
    
    @Override
    public void update(Subject subject) {
        if (subject instanceof WeatherData){
            WeatherData weatherData = (WeatherData) subject;
            this.tempeature = weatherData.getTemperature();
            this.humidity = weatherData.getHumidity();
            display();
        }
    }