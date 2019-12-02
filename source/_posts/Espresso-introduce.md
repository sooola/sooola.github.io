---
title: Android Espresso UI自动测试的搭建
date: 2016-09-07 14:27:51
tags: android
---

#Espresso是什么？
Espresso 是谷歌官方实现的一个UI测试框架，用来确保UI的正确，如每次上线前，你想简单的进入每个界面看看会不会crash，如果界面很多，每次都人工操作就太麻烦了，使用这个框架可以帮助你完成这个任务。
#开始搭建Espresso
项目的build.gradle修改如下:
1.packagingOptions {
 exclude 'META-INF/maven/com.google.guava/guava/pom.xml'
 exclude 'META-INF/maven/com.google.guava/guava/pom.properties'}    
解决重复加载的问题

2.defaultConfig内增加,testInstrumentationRunner 
defaultConfig {
testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"}  

3.在dependencies中加入相关的包  
dependencies {
    androidTestCompile('com.android.support.test:runner:0.5'){}
    androidTestCompile ('com.android.support.test:rules:0.5'){}
    androidTestCompile ('com.android.support.test.espresso:espresso-core:2.2.2'){}
    androidTestCompile ('com.android.support.test.uiautomator:uiautomator-v18:2.1.2'){}
    androidTestCompile 'org.mockito:mockito-core:1.9.5'
    androidTestCompile 'org.hamcrest:hamcrest-library:1.3'
    androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.0') {
  exclude group: 'com.android.support', module: 'appcompat'
 exclude group: 'com.android.support', module: 'support-v4'
 exclude module: 'recyclerview-v7'}
    androidTestCompile 'com.android.support.test.espresso:espresso-web:2.2.2'
}

####下面附上完整的build.gradle文件
```
android {
    compileSdkVersion 23
    buildToolsVersion "24.0.0"
    packagingOptions {
        exclude 'META-INF/maven/com.google.guava/guava/pom.xml'
        exclude 'META-INF/maven/com.google.guava/guava/pom.properties'
    }

    defaultConfig {
        applicationId "com.example.administrator.useespress"
        minSdkVersion 19
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'

    androidTestCompile('com.android.support.test:runner:0.5'){}
    // Set this dependency to use JUnit 4 rules
    androidTestCompile ('com.android.support.test:rules:0.5'){}
    // Set this dependency to build and run Espresso tests
    androidTestCompile ('com.android.support.test.espresso:espresso-core:2.2.2'){}
    // Set this dependency to build and run UI Automator tests
    androidTestCompile ('com.android.support.test.uiautomator:uiautomator-v18:2.1.2'){}
    //mock tests
    androidTestCompile 'org.mockito:mockito-core:1.9.5'
    androidTestCompile 'org.hamcrest:hamcrest-library:1.3'

    androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.0') {
        exclude group: 'com.android.support', module: 'appcompat'
        exclude group: 'com.android.support', module: 'support-v4'
        exclude module: 'recyclerview-v7'
    }

    androidTestCompile 'com.android.support.test.espresso:espresso-web:2.2.2'
}
```
#创建一个Espresso测试类
在app/androidTest/java 下创建一个类
```
public class MainActivityTest {
   private String mStringToBetyped;

   @Rule
   public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(
   MainActivity.class);
 
   @Before
   public void initString(){
   mStringToBetyped = "click";
   }

 

   @Test
   public void clickTest() throws InterruptedException {
   Thread.sleep(1000);
   onView(withText(mStringToBetyped)).perform(click());
   Thread.sleep(1000);
   }

}
```
1.首先创建一个@Rule ActivityTestRule用来指明被测试的Activity;
2.可以使用@Before注解来做测试前的准备工作。
3.测试方法写在@Test 注解下，方法名任意
onView(withText(mStringToBetyped)).perform(click());
上面会执行如下操作，找到显示click文字的view，进行点击操作。
#创建一个TestRunner
1.点击app，选中Edit Config

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-8dc3fcf17ad24568.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.点击+ ，选择Android Test

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-69e9ea13786d8c73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.需要配置的地方如下，
1.先修改名字 2. module选择app 3.测试的类选择刚刚我们创建的class 4.修改runner的类型 5.显示选择设配的dialog

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-5bf85e3f5769baf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Ok,这样就配置好了， run一下试试吧！

一些简单的用法文章在这里[Android Espresso基本使用](http://www.jianshu.com/p/a856ea119d11)