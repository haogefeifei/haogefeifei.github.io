---
layout: post
keywords: blog
description: blog
title: "Java设计模式 - 常用单例"
categories: [Java]
tags: [Java]
---
{% include codepiano/setup %}


单例是最常用到的一个设计模式，java的单例实现方式还是挺多的，总结下遇到的一些单例的实现

## 一. 最简单的单例

{% highlight java %}

    public class Singleton {

      private static Singleton instance = null;
  
      private Singleton() {
      }
  
      public static Singleton getInstance() {
          if (instance == null) {
              instance = new Singleton();
          }
          return instance;
      }
    }

{% endhighlight %}

这种有并发问题，为了避免并发问题需要在判断的时候加锁。于是有了下面的实现。

## 二. 加同步的单例

{% highlight java %}
    public class Singleton {

      private static Singleton instance = null;
  
      private Singleton() {
      }
  
      public synchronized static Singleton getInstance() {
          if (instance == null) {
              instance = new Singleton();
          }
          return instance;
      }
    }
{% endhighlight %}

这种方式在每次获取对象的时候都要先获取锁，如果多次获取对象的话，会造成一些性能开销。考虑到只有在第一次创建对象时才存在并发问题，于是便有了双重判断的实现方式。

## 三. 双重判断的同步的单例

{% highlight java %}
    public class Singleton {

      private static volatile Singleton instance = null;

      private Singleton() {
      }

      public static Singleton getInstance() {
          if (instance == null) {
              synchronized(Singleton.class){
                  if(instance == null)
                      instance = new Singleton();
              }

          }
          return instance;
      }
    }
{% endhighlight %}

java的同步有很多陷阱，考虑到JMM的指令重排问题，instance被声明为volatile，这能够避免一部分问题，但是仍然会有一些并发的陷阱，比如CPU内部流水线也会做些指令的重排，这可不是受JVM指令控制的。如果我们创建一个非常大的单例对象，有可能会出现这种小概率事件。于是，干脆将同步问题扼杀在摇篮里。

## 四. 典型饥饿模式的单例

{% highlight java %}
    public class Singleton {

      private static Singleton instance = new Singleton();

      private Singleton() {
      }

      public static Singleton getInstance() {
          return instance;
      }
    }
{% endhighlight %}

JVM在对类做初始化时，会做同步处理，以保证一个类只被初始化一次。一般，饥饿模式的单例已经能满足大多数的需求，如果你非常渴望延迟加载，而又不想自己去做同步的话。可以试试第5种方式。

## 五. 非同步延迟单例

{% highlight java %}
    public class Singleton {

      private static class SingletonHolder{
          static Singleton instance = new Singleton();
      }

      private Singleton() {
      }

      public static Singleton getInstance() {
          return SingletonHolder.instance;
      }
    }
{% endhighlight %}

这种方式利用JVM初始化的机制来达到延迟加载的目的，个人比较喜欢这种单例实现。不过具体实现的时候还是可能会有各种问题。比如构造类的时候可能出现异常，或者在初始化的时候出现死锁等问题。 最近看到有人用枚举来实现单例，也挺巧妙的。

## 六. 枚举单例

{% highlight java %}
    enum Singleton {
        INSTANCE;

        public static Singleton getInstance(){
            return INSTANCE;
        }
    }
{% endhighlight %}

觉得这种方式比较容易引起歧义，没太想好它的应用场景。不过保证单例是没啥问题。


























