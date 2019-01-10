---
description: Lei Zhengyu - 2018年1月10日
---

# 第三章：插曲：Singleton 模式的讨论

在第二章末尾，我们谈到了 fastjson 在诸多 Serializer 中用到的 Singleton 模式，本章从这些例子出发探讨面向对象设计模式。

## 3.1 Singleton模式

GOF\(Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides\)在《设计模式：可复用面向对象软件的基础》中对 Singleton 模式的定义是：

> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

对 Singleton 的理解是很简单的，我们在 SerializeConfig.java 中已经看到了。实现起来也很容易，可以在需要它的时候创建，即 lazy 初始化：

```java
if (instance == null)
    instance = new Singleton();
return instance;
```

也可以在加载类的时候创建，即 hungry 初始化：

```java
static Singleton instance = new Singleton();
```

实现方式也不止这两种，一般来说，hungry 初始化已经很好了。要使用它也很简单，直接将单例定义成 public 或者设置一个 static 的 get\(\) 方法就可以了。当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时，且这个唯一实例是通过子类化可扩展的，且客户无需更改代码就能使用扩展的实例时，就可以使用 Singleton 模式。

Singleton 被用到的情景很多，如线程池、缓存、对话框、注册表，这些类都不应当制造多个实例。Singleton 的优点有不少，但针对不同的程序设计语言表现不太一致，可以说，只要你意识到某个类只应有一个实例，或者只需要一个实例而程序的性能到了瓶颈，你就可以使用 Singleton. 很多有特定功能的类都满足这个要求，如设计模式中的 Abstract Factory, Builder 和 Prototype 模式就可以用 Singleton 模式实现。

## 3.2 线程安全性

对于 Singleton, 讨论得最多的是它的多线程问题。使用 hungry 初始化方式可以避免多线程问题，这个方式可能会造成资源浪费，毕竟加载类时单例就会被创建，而单例不一定会被使用（个人认为这是一种很好的方式，大部分 Singleton 都是这样使用的，包括 fastjson 中的各种 Codec 和 Serializer）。

而在 lazy 方式中，容易想象，如果两个线程同时进入下面的方法（示例来源于《Head First 设计模式》）：

```java
public class Singleton {
    private static Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

问题就发生了，线程可能在 if 的条件判断结束后切走，导致两个线程分别得到两个实例。要解决这个问题也很容易，在 getInstance\(\) 方法前加上 synchronized 关键字就可以了。只是这样又会导致多余的同步，因为只有在第一次进入这个方法时才需要 synchronized. 

在 Java 5 后的版本中，可以用 double-checked locking 解决这个问题。只是实现起来要更加复杂：

```java
public class Singleton {
    private volatile static Singleton uniqueInstance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

