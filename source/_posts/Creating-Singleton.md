---
title: 设计模式之创建型模式：单例模式
date: 2019-03-14 10:22:30
tags:
 - Java
categories: 
 - 设计模式
---
## 1. 特点
在Java应用中，单例对象能保证在一个JVM中，该对象只有一个实例存在，且必须由单例类自己创建唯一实例，单例类必须给所有其他对象提供这一实例。这样的模式有几个好处：
1）某些类创建比较频繁，对于一些大型的对象，这是一笔很大的系统开销。

2）省去了new操作符，降低了系统内存的使用频率，减轻GC压力。

3）有些类如交易所的核心交易引擎，控制着交易流程，如果该类可以创建多个的话，系统完全乱了。（比如一个军队出现了多个司令员同时指挥，肯定会乱成一团），所以只有使用单例模式，才能保证核心交易服务器独立控制整个流程。
<!--more-->
## 实现方式
### 饿汉式单例（立即加载方式）

	// 饿汉式单例
	public class Singleton 
	{
    		// 私有构造
   	 	private Singleton() {};
   	 	private static Singleton single = new Singleton();
    		// 静态工厂方法
    		public static Singleton getInstance()
    		{
        		return single;
    		}
	}
饿汉式单例在类加载初始化时就创建好一个静态的对象供外部使用，除非系统重启，这个对象不会改变，所以本身就是线程安全的。Singleton通过将构造方法限定为private避免了类在外部被实例化，在同一个虚拟机范围内，Singleton的唯一实例只能通过getInstance()方法访问。（事实上，通过Java反射机制是能够实例化构造方法为private的类的，那基本上会使所有的Java单例实现失效。此问题在此处不做讨论，姑且闭着眼就认为反射机制不存在。）
### 懒汉式单例（延迟加载方式）

	// 饿汉式单例
	public class Singleton 
	{
    		// 私有构造
   	 	private Singleton() {};
   	 	private static Singleton single = null;
    		// 静态工厂方法
    		public static synchronized Singleton getInstance()
    		{
        		if (single==null)
        		{
        			single = new Singleton();
        		}
        		return single;
    		}
	}
这里的getInstance()方法用了synchronized修饰，单例在毫无线程安全保护下，放入多线程的环境下，肯定就会出现问题了。但是，synchronized关键字锁住的是这个对象，这样的用法，在性能上会有所下降，因为每次调用getInstance()，都要对对象上锁，事实上，只有在第一次创建对象的时候需要加锁，之后就不需要了，所以，这个地方需要改进。我们改成下面这个：

	public static Singleton getInstance()
    	{
        	if (single==null)
        	{
        		synchronized(Singleton.class)
        		{
        			if (single==null)
        			{
        				single = new Singleton();
        			}
        		}
        	}
        	return single;
    	}
只在第一次创建对象时上锁， 性能得到了保证。但还是存在线程安全问题。在Java指令操作中创建和赋值是分开的，即 single = new Singleton()其实是两个操作，jvm并不保证执行的顺序，有可能jvm在给Singleton实例分配好内存地址后就直接将地址赋值给single，再去初始化Singleton实例。以A、B两个线程为例：
故再进一步优化：

	private static class SingletonFactory
	{
		private static Singleton single = new Singleton();
	}
	public static Singleton getInstance()
	{
		return SingletonFactory.single;
	}
使用内部类来维护单例的实现，因为jvm内部机制保证一个类的加载过程是线程互斥的，内部类加载的同时Singleton单例就已经创建完成了。但这个方法也不是完美的，如果构造函数中抛出异常，单例将永远不能被创建	
## 最后
单例模式理解起来简单，但是具体实现起来还是有一定的难度，特别是要考虑线程安全问题，各家有个家的方法，各有优缺，就看实际情况来取舍了。

学习参考：
https://www.cnblogs.com/garryfu/p/7976546.html
https://blog.csdn.net/a745233700/article/details/83409999
  