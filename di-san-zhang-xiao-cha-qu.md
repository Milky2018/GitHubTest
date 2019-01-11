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

问题终于解决了。这个实现看起来有些过于考虑底层问题，所以，在不是很在乎性能的情况下，我还是建议大家用 lazy 方式，不过，如果你的程序有多个类加载器就要小心了，它们有可能创建各自的 Singleton 实例。

## 3.3 Singleton vs. 全局变量 vs. static类

以及继承 Singleton 的问题。这些内容暂时保留，因为我自己都还没有理解清楚。。。

## 3.4 Singletonitis

Singletonitis 是 Joshua Kerievsky 在《重构与模式》一书中发明的词汇，意思是“沉迷于 Singleton 模式“。Kerievsky 认为绝大多数时候，Singleton 的出现是不必要的，此时使用内联 Singleton 重构是一种摆脱不必要的 Singleton 的有效方法。

Kerievsky 认为：当把一个对象资源以引用的方式传给需要它的对象，比让需要它的对象全局访问这一资源更简单的时候，或者 Singleton 被用来获得无关紧要的内存或性能改进时，以及系统中深层次代码需要访问一个并不存在相同层次中的资源的时候，Singleton 就是不必要的。

究其原因，我认为，Kerievsky 在担心一个 Singleton 会被很容易地访问，而它的状态又是被共享的，这或许是一种灾难。这些事情留给未来的我们去权衡，我们处于入门阶段，尚不需要懂得如何拒绝它。

## 3.5 回到fastjson

为什么我在第二章说 fastjson 中的各种 Codec 类不是严格的 Singleton 模式，原因在于它们的构造器都是公开的，不严格限制实例数量。以 FloatCodec 为例：

{% code-tabs %}
{% code-tabs-item title="FloatCodec.java" %}
```java
public class FloatCodec implements ObjectSerializer, ObjectDeserializer {
    public static FloatCodec instance = new FloatCodec();

    public FloatCodec(){

    }
    ...
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

那么它算不算 Singleton 模式呢？纠结这个问题没什么价值，不过我还是认为它算是 Singleton 模式。首先它的应用场景符合 Singleton 的动机，一个解码器不需要多个实例；其次，整个 fastjson 类库的源代码中找不到第二处对于 FloatCodec 类的实例化，可以猜测公开其构造器是为了测试或者用户扩展。

幸运的是，在这个例子中我们也不需要担心 Kerievsky 考虑的问题，因为一个 FloatCodec 的状态在它被创建时就已经被确定了，类的定义中没有和字段相关的 set 方法。

