---
title: 设计模式之创建型模式：工厂模式
date: 2019-03-13 14:12:12
tags: Java
categories: 设计模式
---
工厂模式是我们最常用的实例化对象模式了，是用工厂方法代替new操作的一种模式。著名的Jive论坛 ,就大量使用了工厂模式，工厂模式在Java程序系统可以说是随处可见。因为工厂模式就相当于创建实例对象的new，我们经常要根据类Class生成实例对象，如A a=new A() 工厂模式也是用来创建实例对象的，所以以后new时就要多个心眼，是否可以考虑使用工厂模式，虽然这样做，可能多做一些工作，但会给你系统带来更大的可扩展性和尽量少的修改量。
<!-- more-->
本文使用的结构图为UML，不知道的，{% post_link UML%}

## 1. 简单工厂模式
以买饮料为例
![](https://s2.ax1x.com/2019/03/13/AkdzvV.png)

我们先建立一个饮料的抽象类

	public abstract class drink()
	{
		public abstract void desc();
	}
然后就是具体的实物这里的coffe、tea、cola	
	
	public class tea() extend drinks
	{
		@Override
		public desc()
		{
			System.out.println("tea");
		}
	}
	public class coffee() extend drinks
	{
		@Override
		public desc()
		{
			System.out.println("coffee");
		}
	}
	public class cola() extend drinks
	{
		@Override
		public desc()
		{
			System.out.println("cola");
		}
	}
然后统一由饮料店售卖	
	
	public class shop()
	{
		public static drink sellddrinks(int type)
		{
			switch(type)
			{
				case 1 : return new tea();
				case 2 : return new coffee();
				case 3 : return new cola();
			}
		}
	}
你要哪种饮料，它就卖那种

	drink mydrink = shop.selldrinks(1);
	mydrink.desc();	
### 优点
1）简单工厂模式实现了对责任的分割，提供了专门的工厂类用于创建对象，实现对象的创建和对象的使用分离。
2）客户不需要知道创建对象的逻辑，甚至不需要知道具体类名，只要知道对应参数即可。
### 缺点
1）过于集中，所有的创建逻辑都在工厂类中，一旦不能正常工作，整个系统就瘫痪了，有道是鸡蛋不能都放一个篮子里。
2）对产品部分来说，它是符合开闭原则的，但是工厂部分不符合，系统扩展困难，一旦添加新产品就不得不修改工厂逻辑，在产品类型较多时，有可能造成工厂逻辑过于复杂，不利于系统的扩展和维护。

## 2. 多工厂方法
上面实现的工厂都是通过额外参数如type来确定不同产品。但传递的type出错，将不能得到正确的对象，容错率低。而多方法的工厂模式为不同产品，提供不同的生产方法
![](https://s2.ax1x.com/2019/03/13/Akt0L4.png)
具体代码我就不写了。
ps.也可以将方法设置为静态，不创建实例。

## 普通工厂
把简单工厂中具体的工厂类，划分为抽象工厂类和具体工厂类两层。简单来说，就是饮料店为统称，具体的是各大饮料的专卖店。
![](https://s2.ax1x.com/2019/03/13/Ak0oTg.png)

	public abstract class shop()
	{
		public abstract drink createDrink();
	}
	public class TeaShop() extend shop
	{
		public drink creat()
		{
			return new tea();
		}
	}
	public class CoffeeShop() extend shop
	{
		public drink creat()
		{
			return new coffee();
		}
	}
	public class ColaShop() extend shop
	{
		public drink creat()
		{
			return new cola();
		}
	}
接下来便是要什么饮料，找什么工厂
###普通工厂与简单工厂模式的区别：
可以看出，普通工厂模式特点：不仅仅做出来的产品要抽象， 工厂也应该需要抽象。
工厂方法使一个产品类的实例化延迟到其具体工厂子类.
工厂方法的好处就是更拥抱变化。当需求变化，只需要增删相应的类，不需要修改已有的类。
而简单工厂需要修改工厂类的create()方法，多方法工厂模式需要增加一个静态方法。

## 抽象工厂
以上介绍的工厂都是单产品系的。抽象工厂是多产品系。
不仅工厂抽象具体分离，产品也分离。在讲解抽象工厂模式之前，我们要讲两个概念：这回shop不单卖饮料了，还卖零食、文具等等。
1）产品等级结构：即同种产品，coffee、tea、cola等，都继承自一个抽象类
2）产品族：产品族是在抽象工厂模式中的。即都由一个工厂生产。
![](https://s2.ax1x.com/2019/03/13/AkrlOP.png)

	public abstract class drink()
	{
		public abstract void price();
	}
	public class tea() extend drink
	{
		public void price()
		{
			System.out.println("$.4");
		}
	}
	public class coffee() extend drink
	{
		public void price()
		{
			System.out.println("$.6");
		}
	}	
	public abstract class food()
	{
		public abstract void price();
	}
	public class chips() extend food
	{
		public void price()
		{
			System.out.println("$.11");
		}
	}
	public class hotdog() extend food
	{
		public void price()
		{
			System.out.println("$.14");
		}
	}
	public abstract class shop()
	{
		public abstract drink createDrink();
		public abstract food createFood();
	}
具体的快餐店

	public class KFC() extend shop
	{
		public food createDrink()
		{
			return new Tea();
		}
		public food createFood()
		{
			return new chips()
		}
	}
	public class PizzaHut() extend shop
	{
		public food createDrink()
		{
			return new coffee();
		}
		public food createFood()
		{
			return new hotdog()
		}
	}	

接下来的使用就看自己了
## 使用场景
一句话总结工厂模式：方便创建 同种产品类型的 复杂参数 对象
工厂模式重点就是适用于 构建同产品类型（同一个接口 基类）的不同对象时，这些对象new很复杂，需要很多的参数，而这些参数中大部分都是固定的，so，懒惰的程序员便用工厂模式封装之。 
（如果构建某个对象很复杂，需要很多参数，但这些参数大部分都是“不固定”的，应该使用Builder模式）
为了适应程序的扩展性，拥抱变化，便衍生出了 普通工厂、抽象工厂等模式。


学习参考
https://blog.csdn.net/a745233700/article/details/83592574
https://www.cnblogs.com/xuxinstyle/p/9128865.html
	





	