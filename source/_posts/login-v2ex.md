---
title: Android模拟登录V2EX
date: 2016-11-16 11:41:33
tags: android
---



>最近在撸一个V2EX的客户端，官方API缺少一些功能如登录，发帖等，撸完官方API总觉得少了什么，本篇文章主要通过模拟登录实现一些官方没提供API的功能  

## 观察登录传输的数据  
在网页上登录帐号，通过chrome的调试模式可以看到 ，我们传了4个数据给服务器，分别是帐号，密码，once，和next，once是用来验证是否人为操作的标志，POST时必须带上这个字段，否则会认为是非人为操作而被禁止访问。


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-fbe6d5d338844af3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




知道了需要传什么参数后，退出登陆，回到登录之前的页面，https://www.v2ex.com/signin， 我们需要POST的数据都需要在这个页面里捉取下来

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-8b61db054c2c8378.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 代码实现
1.添加请求头  
在chrome登陆成功页面查看POST的请求头，在OkHttp中相应进行配置。  

```
        mClient = new OkHttpClient.Builder()
                .cookieJar(new CustomCookieJar(new CookieManager(mPersistentCookieStore, CookiePolicy.ACCEPT_ALL)))
                .readTimeout(12, TimeUnit.SECONDS)
                .addInterceptor(htmlInterceptor)
                .addInterceptor(loggingInterceptor)
                .build();

    private Interceptor htmlInterceptor = new Interceptor() {
        @Override
        public Response intercept(Chain chain) throws IOException {
            Request newRequest = chain.request().newBuilder()
                    .addHeader("Origin", "https://www.v2ex.com")
                    .addHeader("Content-Type", "application/x-www-form-urlencoded")
                    .build();
            return chain.proceed(newRequest);
        }
    };
```


2.定义登录接口api
分别定义get和Post 两种，get取得帐号，密码，once的值，用于POST提交
```
    @GET("signin")      //登录
    Observable<String> login();

    @FormUrlEncoded
    @Headers("Referer: https://www.v2ex.com/signin")
    @POST("signin")
    Observable<String> postLogin(@FieldMap HashMap<String, String> hashMap);
```
3获取数据，发送登陆请求
```
        Subscription subscription = LoginService.getInstance().auth().login()
                .map(new Func1<String, HashMap>() {
                    @Override
                    public HashMap call(String stringResponse) {
                        return JsoupUtil.parseUserNameAndPwd(stringResponse, username, password);
                    }
                }).flatMap(new Func1<HashMap, Observable<String>>() {
                    @Override
                    public Observable<String> call(HashMap requestMap) {
                        return LoginService.getInstance().auth().postLogin(requestMap);
                    }
                }).map(new Func1<String, LoginResult>() {
                    @Override
                    public LoginResult call(String response) {
                        String errorMsg = ApiErrorUtil.getErrorMsg(response);
                        if (errorMsg == null) {
                            return JsoupUtil.parseLoginResult(response);
                        } else {
                            LoginResult result = new LoginResult();
                            result.setMessage(errorMsg);
                            return result;
                        }
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(observer);

```
这里使用了Jsoup来解析html可以很便捷的获取需要的数据，没了解过的同学可以查看这里[Jsoup开发指南](http://www.open-open.com/jsoup/)

登录成功后返回的html里面就有用户的数据了

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-c66d3c3ca95e0b9f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Cookie
当你完成已上步骤还是无法获取个人信息，会自动跳转到登陆页面，因为http请求是无状态的，服务器并不知道你已经登录了，这里就需要配置Cookie，登录成功后，服务会返回一个Cookie(通行证)，客户端需保存这个Cookie，再次发送请求时带上这个Cookie，服务器就能识别该用户，并且返回用户相关的信息了。
在OKHttp3中配置Cookie持久化只需一句话

```
.cookieJar(new CustomCookieJar(new CookieManager(mPersistentCookieStore, CookiePolicy.ACCEPT_ALL)))
```
实现Cookie后就可以获取所有数据了，如用户关注节点等。

对cookie不了解的可以看这里 [Cookie/Session机制详解](http://blog.csdn.net/fangaoxin/article/details/6952954)
在OkHttp中如何实现Cookie的持久化可以看这里[[OkHttp3之Cookies管理及持久化](https://segmentfault.com/a/1190000004345545)](https://segmentfault.com/a/1190000004345545)

以上完整的代码在这里 https://github.com/sooola/V2EX-Android ，RxJava &Retrofit 的V2EX第三方客户端，项目在龟速更新中，有任何建议欢迎issues，开源不易，路过求start鼓励下(⊙ᗜ⊙)