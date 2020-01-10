---
title: EventBus源码分析
date: 2019-12-24 14:25:29
tags: [android , EventBus]
---

EventBus是Android开发中常用的通知事件框架，使用EventBus非常简单只需3布即可，那么实际上EventBus是怎么实现订阅通知的？下面我们从源码开始分析

![eventbus_img.png](https://upload-images.jianshu.io/upload_images/1816215-340c82a817875d7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 二、源码分析

我们使用EventBus时，需要在生命周期中注册为订阅者 ，代码如下

```java
 @Override
 public void onStart() {
     super.onStart();
     EventBus.getDefault().register(this);
 }

 @Override
 public void onStop() {
     super.onStop();
     EventBus.getDefault().unregister(this);
 }
```

那么我们就从这里开始分析

#### 2.1 EventBus.getDefault()

```java
    public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
```

EventBus.getDefault()是使用单例模式，获取一个EventBus的实例

#### 2.2EventBus.getDefault().register(this)

```java
    public void register(Object subscriber) {
        //通过反射获取类名
        Class<?> subscriberClass = subscriber.getClass();
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
        synchronized (this) {
            for (SubscriberMethod subscriberMethod : subscriberMethods) {
                subscribe(subscriber, subscriberMethod);
            }
        }
    }
```

可以看到重要的方法为        

List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);

#### 2.3 findSubscriberMethods

根据他的名字，猜测是获取相关的订阅者，并返回列表集合

```java
    List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
        List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
        //从缓存中取出，有则直接返回
        if (subscriberMethods != null) {
            return subscriberMethods;
        }
//// ignoreGeneratedIndex在SubscriberMethodFinder()的构造函数初始化的，默认值是 false
        if (ignoreGeneratedIndex) {
            //通过反射获取
            subscriberMethods = findUsingReflection(subscriberClass);
        } else {
            //通过插件生成代码，使用SubscriberInfo获取
            subscriberMethods = findUsingInfo(subscriberClass);
        }
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass
                    + " and its super classes have no public methods with the @Subscribe annotation");
        } else {
            METHOD_CACHE.put(subscriberClass, subscriberMethods);
            return subscriberMethods;
        }
    }
```

获取订阅者的步骤为

1.检查缓存，若缓存存在，则直接返回

2.根据配置参数，使用反射或SubscriberInfo获取，默认情况下不使用反射，减少反射消耗，提高运行效率。

#### 2.4 findUsingReflection，prepareFindState

下面看看反射方法如何获取订阅者信息

```java
    private List<SubscriberMethod> findUsingReflection(Class<?> subscriberClass) {
        FindState findState = prepareFindState();	//findState 用来做订阅者的保存和验证
        findState.initForSubscriber(subscriberClass);	//初始化findState
        while (findState.clazz != null) {
            //通过反射获取订阅方法
            findUsingReflectionInSingleClass(findState);
            findState.moveToSuperclass();
        }
        return getMethodsAndRelease(findState);
    }
```

在查找订阅者之前，先做了FindState的准备工作，那先看看prepareFindState()

```java
    private FindState prepareFindState() {
        synchronized (FIND_STATE_POOL) {
            for (int i = 0; i < POOL_SIZE; i++) {
                FindState state = FIND_STATE_POOL[i];
                if (state != null) {
                    FIND_STATE_POOL[i] = null;
                    return state;
                }
            }
        }
        return new FindState();		
    }
```

prepareFindState()主要做了两步操作，1初始化FIND_STATE_POOL为null ，2返回一个FindState对象（FindState用来保存和校验订阅方法）

```java
        void initForSubscriber(Class<?> subscriberClass) {
            this.subscriberClass = clazz = subscriberClass;
            skipSuperClasses = false;
            subscriberInfo = null;
        }
```

赋值subscriberClass，clazz并设subscriberInfo为null

最后看看获取订阅方法的最主要的方法findUsingReflectionInSingleClass

```java
private void findUsingReflectionInSingleClass(FindState findState) {
        Method[] methods;
        try {
            methods = findState.clazz.getDeclaredMethods();  //获取订阅类中所有的非继承的方法
        } catch (Throwable th) {
            methods = findState.clazz.getMethods();
            findState.skipSuperClasses = true;
        }
        for (Method method : methods) { //遍历所有方法
            int modifiers = method.getModifiers();
            if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                //方法不是public,且不是abstract和static
                Class<?>[] parameterTypes = method.getParameterTypes();
                if (parameterTypes.length == 1) {
                    //方法的参数为1个
                    Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                    if (subscribeAnnotation != null) {
                        //方法的注解是@Subscribe
                        Class<?> eventType = parameterTypes[0];//获取这个Event对象
                        if (findState.checkAdd(method, eventType)) {
                            ThreadMode threadMode = subscribeAnnotation.threadMode();
                              //实例化SubscriberMethod对象并添加到findState
                            findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                    subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                        }
                    }
                } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                    String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                    throw new EventBusException("@Subscribe method " + methodName +
                            "must have exactly 1 parameter but has " + parameterTypes.length);
                }
            } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException(methodName +
                        " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
            }
        }
    }
```

到这里所有的订阅类的订阅方法已经被保存通过 List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass); 返回

#### 2.5 subscribe(subscriber, subscriberMethod);

最后回到刚开始的register方法，看看subscribe的实现

```java
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    	//获取订阅事件类型 （订阅方法的第一个参数）
        Class<?> eventType = subscriberMethod.eventType;
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    //根据订阅事件类型获取subscriptions（订阅者）
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        if (subscriptions == null) {
            //如果为空，则创建这个subscriptions，放到subscriptionsByEventType中
            subscriptions = new CopyOnWriteArrayList<>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            if (subscriptions.contains(newSubscription)) {
                throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                        + eventType);
            }
        }

        int size = subscriptions.size();
    //根据优先级进行排序
        for (int i = 0; i <= size; i++) {
            if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }
		//typesBySubscriber保存 订阅对象，和事件列表
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            //如果为空，则创建
            subscribedEvents = new ArrayList<>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        if (subscriberMethod.sticky) {	//是否粘性消息
            if (eventInheritance) {
                // Existing sticky events of all subclasses of eventType have to be considered.
                // Note: Iterating over all events may be inefficient with lots of sticky events,
                // thus data structure should be changed to allow a more efficient lookup
                // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
                Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
                for (Map.Entry<Class<?>, Object> entry : entries) {
                    Class<?> candidateEventType = entry.getKey();
                    if (eventType.isAssignableFrom(candidateEventType)) {
                        Object stickyEvent = entry.getValue();
                        //发送事件
                        checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                    }
                }
            } else {
                Object stickyEvent = stickyEvents.get(eventType);
                checkPostStickyEventToSubscription(newSubscription, stickyEvent);
            }
        }
    }

```

主要做的事情为 subscriptionsByEventType用于维护事件(eventType)与订阅者(subscriptions)关系，

 typesBySubscriber 用于维护 subscriber(订阅者)  与subscribedEvents（订阅事件）的关系，最后发送事件，发送事件与post类似，这个我们下面再看

用一张流程图总结一下注册的内容

![img](https://img2018.cnblogs.com/blog/1101915/201811/1101915-20181112090815126-528128420.png)

#### 2.6发布

发送事件给Eventbus

```java
    public void post(Object event) {
        PostingThreadState postingState = currentPostingThreadState.get();	
        //获取当前线程的eventQueue
        List<Object> eventQueue = postingState.eventQueue;
        //将要发送的event 添加到队列中
        eventQueue.add(event);

        if (!postingState.isPosting) {
            postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                while (!eventQueue.isEmpty()) {
                    //发送消息
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }

```

最终调用

```java
    private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
            case POSTING:
                invokeSubscriber(subscription, event);
                break;
            case MAIN:
                if (isMainThread) {
                    invokeSubscriber(subscription, event);
                } else {
                    mainThreadPoster.enqueue(subscription, event);
                }
                break;
            case BACKGROUND:
                if (isMainThread) {
                    backgroundPoster.enqueue(subscription, event);
                } else {
                    invokeSubscriber(subscription, event);
                }
                break;
            case ASYNC:
                asyncPoster.enqueue(subscription, event);
                break;
            default:
                throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }

    void invokeSubscriber(Subscription subscription, Object event) {
        try {
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
        } catch (InvocationTargetException e) {
            handleSubscriberException(subscription, event, e.getCause());
        } catch (IllegalAccessException e) {
            throw new IllegalStateException("Unexpected exception", e);
        }
    }



```

最终使用反射，调用订阅者的事件方法。

用一张流程图总结下发送流程![img](https://img2018.cnblogs.com/blog/1101915/201811/1101915-20181112090824029-247247233.png)

以上为EventBus的源码分析，当然还有很多地方没有分析到的，如果有不对的地方欢迎指出。



参考

https://www.cnblogs.com/fuyaozhishang/p/9944653.html

https://www.jianshu.com/p/6da03454f75a