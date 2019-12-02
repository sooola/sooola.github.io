---
title: ArrayList源码分析
date: 2019-08-23 14:47:49
tags: [android ,源码]
---

### 什么是ArrayList

ArrayList 是 java 集合框架中比较常用的数据结构，底层基于数组实现容量大小动态变化。允许 null 的存在。同时还实现了 RandomAccess、Cloneable、Serializable 接口，所以ArrayList 是支持快速访问、复制、序列化的。

我们知道在java中，数组定义了大小，就不能改变，那么ArrayList是怎么实现动态扩容，扩容的规则是怎样的，我们带着这些问题去查看源码。

首先看一段代码

```java
List<String> list = new ArrayList<String>();
list.add("apple");
list.add("snow pear");
list.add("orange");
list.remove(0);
```

上面代码中创建了一个集合List，并通过调用add方法，添加数据，最后通过remove方法，溢出了集合第一个的数据。

那么我们从代码一步步往下看

```java
 private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0];

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

可以看到当new ArrayList()时，其实时创建了一个大小为0的对象数组。一般来说，这个数组大小为0，是不能添加元素的，下面继续看看list.add()方法

```java
    public boolean add(E var1) {
        ensureCapacityInternal(this.size + 1);	//确保对象数组elementData有足够的容量，可以将新加入的元素e加进去
        elementData[this.size++] = var1;
        return true;
    }
```

可以看到首先调用了ensureCapacityInternal，后是把值添加到了elementData尾部，那么根据ensureCapacityInternal（确保内部容量）大概猜测，这个方式就是**核心的扩展数组容量**的方法。

那么我们看一下他的实现

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    //如果需要的最小容量 大于 当前数组提供的容量
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);	//调用扩容方法
}
private void grow(int minCapacity) {
	//记录旧的容量
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩为1.5倍还不满足需求，直接扩为需求值
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //如果新的容量超出SIZE最大值
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //通过Arrays.copyOf方法，把原来的数组复制到新的容器
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
上面注释已经解释了跨容的方式  

总结一下扩容的步骤
1. 判断所需容量是否大于当前提供的容量
2. 不满足需求，则扩容为1.5倍，如果还不满足，则扩容为需求值
3. 通过Arrays.copyOf把旧的对象数组复制到新对象数组

那么现在就可以回答我们之前的问题了。

参考资料
[ArrayList工作原理及实现](https://yikun.github.io/2015/04/04/Java-ArrayList%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)
[走进 JDK 之 ArrayList（一）](https://juejin.im/post/5cc070a2f265da037a3cee37)