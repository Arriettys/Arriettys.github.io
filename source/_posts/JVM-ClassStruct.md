---
title: JVM-Class类文件结构
date: 2019-06-15 11:19:32
tags: Java
categories: JVM
---
最近在看JVM的Class类文件结构的章节，书上的例子因为图片较少常常需要往上回翻看表或者16进制表，觉得很是麻烦，所以自己动手做了下实验，并截图以供对比。参考自《_深入理解Java虚拟机_JVM高级特性与最佳实践 第2版》
<!--more-->
首先看看导图
![](https://s2.ax1x.com/2019/06/20/Vvx8je.png)
开始之前先介绍下工具，笔者用的系统是Ubuntu系统，编辑文本用的是vim，但用vim打开class文件后并不能直接看到16进制数
![](https://s2.ax1x.com/2019/06/15/VIdD00.png)
点击Esc键输入冒号:，然后输入%!xxd，即可显示16进制
![](https://s2.ax1x.com/2019/06/15/VIdI76.png)

你也可以使用Java自带的工具，在JDK的bin目录中有专门用于分析Class文件字节码的工具：javap。使用javap工具的-verbose参数可以输出.class文件字节码内容
![](https://s2.ax1x.com/2019/06/15/VIwDvd.png)

## 数据格式

+ Class文件以8位字节为基础单位的二进制流，当数据项占用8位以上空间，则会按Big-Endian方式分割成若干个8位字节进行存储。
+ Class文件有两种数据类型：无符号数和表。无符号数以u1、u2、u4来分别代表1个字节、2个字节4个字节等。表由多个无符号数或者其他表作为数据项构成复合数据类型，所有表以“_info”结尾。 

## 文件结构
为了解释方便，我已下面这段代码生成Class文件来进行比对
```
public class ViewClassStruct
{
	private static final int num = 1;
	public static void main(String[] args)
	{
		String str = "HelloWorld";
		System.out.println(str+num);
	}
	
	public int inc ()
	{
		return num+1;
	}
}
```
### 魔数与Class文件版本
Class文件的头4个字节成为魔数（Magic Number），唯一的作用是确定这个文件是否为一个被虚拟机接受的Class文件，值为0xCAFEBABE
![](https://s2.ax1x.com/2019/06/15/VI2OmD.png)
紧接着魔数的4个字节存储的是Class文件的版本号。前两个字节是次版本号，后两个字节是主版本号
![](https://s2.ax1x.com/2019/06/15/VIWVKK.png)
![](https://s2.ax1x.com/2019/06/15/VIWB2q.png)
ps.相关的Class文件版本号的16进制请自行网上查找

### 常量池
紧接着主次版本之后的就是常量池，它是Class文件结构中与其他项目关联最多的数据类型，也是占用Class文件空间最大的数据项目之一

####  常量池容量计数器（constant_pool_count）
u2类型，计数从1而不是0开始，第0项常量有特殊考虑，这里就不讲了。
![](https://s2.ax1x.com/2019/06/15/VIfX0U.png)

#### 常量池（constant_pool）
主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）
字面量接近Java语言的常量概念，如文本字符串、申明为final的常量值等

而符号引用则属于编译原理方面的概念，包括下面三类常量

+ 类和接口的权限定名（Fully Qualified Name）
+ 字段的名称和描述符（Descriptor）
+ 方法的名称和描述符

** Class文件不会保存各个方法、字段的最终内存布局信息，因为这些符号引用不经过运行期转换的话无法得到真正的内存入口地址，当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中 **

常量池中的每一项常量都是一个表，只JDK1.7开始总共有14种表，这些表共同的特点就是第一位是一个u2类型的标志位，代表当前这个常量属于哪种常量类型，这14种常量类型如下图所示
![](https://s2.ax1x.com/2019/06/15/VIq2CT.png)

这14种常量类结构型均有自己的结构，如图
![](https://s2.ax1x.com/2019/06/15/VILXF0.png)
我们测试的Class文件第一项常量就是CONSTANT_Class_info类型，其标志位是0x07，该类型代表类或者接口的符号引用，结构为
![](https://s2.ax1x.com/2019/06/15/VIOdmj.png)

其另一个项name_index是一个索引值，它指向常量池中的一个CONSTANT_Utf8_info类型常量，代表这个类（或者接口）的权限定名

图中name_index为0x0002，指向了常量池第二项常量，而第二项的标志位就是0x01，表结构为
![](https://s2.ax1x.com/2019/06/15/VIXqP0.png)

length值说明这个UTF8编码的字符串长度是多少字节，他后面紧跟着的长度为length字节的连续数据是一个使用UTF8缩略编码表示的字符串
图中length值为0x000f，也就是15字节长的字符串
![](https://s2.ax1x.com/2019/06/15/VIvepV.png)
通过比对ASCII表内容为ViewClassStruct
![](https://s2.ax1x.com/2019/06/15/VIvUXD.png)

** 由于Class文件中方法、字段等都需要引用CONSTANT_Utf8_info型常量来描述名称，所以CONSTANT_Utf8_info型常量的最大长度也就是Java中方法、字段名的最大长度，而这也就是length的最大值，即u2类型能表达的 最大值65535，所以Java程序中不能定义超过64KB英文字符的变量或方法名 **

后面的常量这里就不接下去讲了，大家可以比对或者用javap -verbose命令让计算机帮我们计算

### 访问标志
常量池结束后，紧接着的两个字节代表访问标志（access_flags）,这个表示用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被申明为final等，具体标志含义见图
![](https://s2.ax1x.com/2019/06/15/VoSuX8.png)
access_flags中一共有16个标志为可供使用，当前Java只用了其中8个，没有使用的标志位要求一律为0
![](https://s2.ax1x.com/2019/06/15/VoSDAJ.png)
上图中，我们的测试类只是一个普通类，所以ACC_PUBLIC、ACC_SUPER标志为真

### 类索引、父索引与接口索引集合
类索引（this_class）和父类索引（super_class）都是u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据集合，Class文件通过这三项确定这个类的继承关系，它们各自指向CONSTANT_Class_info类型的类描述符常量
类索引确定这个类的权限定名，父类索引确定父类的权限定名，接口索引中的接口顺序按implements语句后的接口顺序排列
![](https://s2.ax1x.com/2019/06/15/VopTZF.png)
我们的测试Class文件中可以看到类索引为0x0001，指向常量池中第一项常量
![](https://s2.ax1x.com/2019/06/15/Vo9Zsf.png)
正是这个类在常量池的位置，而其后的父类索引指向第三项
![](https://s2.ax1x.com/2019/06/15/Vo9YLT.png)
因为我们的测试类没有继承别的类，其父类就是Object类

### 字段表集合
用于描述接口或者类中声明的 变量，包括类级变量以及实例级变量，但不包括方法内部声明的局部变量，字段表格式
ps.字段表集合前有一个u2类型的容器计数器fields_count记录字段表集合长度
![](https://s2.ax1x.com/2019/06/15/VonNmd.png)
如图，当前测试类只有一个字段
![](https://s2.ax1x.com/2019/06/15/VoCc3n.png)
字段修饰符放在access_flags项目中，它与类中的access_flags项目类似，都是u2数据类型，可设置的标志位和含义见图
![](https://s2.ax1x.com/2019/06/15/VoCjHO.png)
ps。接口中字段必须有ACC_PUBLIC、ACC_STATIC、ACC_FINAL标志，因为接口中的只能有static final 类型的变量

跟随access_flags标志的是两项索引值：name_index和descriptor_index，他们都是对常量池的引用，分别代表字段的简单名称以及字段和方法的描述符

+ 简单名称
指没有类型和参数修饰的方法或者字段名称，如inc()方法和m字段的简单名称分别为“inc”和“m”
+ 描述符
描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值
根据描述符规则，基本数据类型以及代表无返回值的void类型用一个大写字符来表示，如图
![](https://s2.ax1x.com/2019/06/15/VoVqiT.png)
对于数组类型，每一维度将使用一个前置的“[”字符来描述，如一个定义为“java.lang.String[][]”类型的二维数组，将被记录为：“[[Ljava/lang/String;”,一个整形数组“int[]”将被记录为“[I”
来看看我们测试类中的字段
![](https://s2.ax1x.com/2019/06/15/VoNuvR.png)
如图access_flags为0x001a，说明ACC_PRIVATE、ACC_STATIC和ACC_FINAL三个标志为真

name_index值为0x0005，指向的常量为
![](https://s2.ax1x.com/2019/06/15/VoN7iF.png)
该字段简单名称为num

descriptor_index值为0x0006，指向的常量为
![](https://s2.ax1x.com/2019/06/15/VoNzdK.png)
该字段数据类型为Integer

attributes_count值为0x0001，即只有一个属性表
而紧接其后的attribute_info，其结构为（该表为规则结构，根据不同的属性，会在info出细分）
![](https://s2.ax1x.com/2019/06/15/VoUhSH.png)
让我们回到16进制表上
![](https://s2.ax1x.com/2019/06/15/VoUIOI.png)
atrribute_name_index值为0x0007,指向的常量为
![](https://s2.ax1x.com/2019/06/15/VoUOfg.png)
该属性是一个ConStantValue，所以后面会指向一个常量

attribute_length值为0x00000002，说明属性值为接下来2个u1，即0x0008，指向的常量为
![](https://s2.ax1x.com/2019/06/15/VoUOfg.png)

** 字段表中不会列出从父类或接口继承而来的字段，但可能列出原本Java代码中不存在的隐式字段，如内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段**

### 方法表集合
方法表的结构同字段表一样，以此包括了访问标志、名称索引、描述符索引、属性表集合几项，之前也有一个u2的计数器
![](https://s2.ax1x.com/2019/06/15/Vod5Zt.png)

access_flags
![](https://s2.ax1x.com/2019/06/15/Vodzd0.png)

看回16进制文件
![](https://s2.ax1x.com/2019/06/15/VowRYT.png)
如图methods_count为0x0003，有三个方法表
第一张表的access_flags为0x0001，即ACC_PUBLIC为真
name_index为0x0009，指向的常量为
![](https://s2.ax1x.com/2019/06/15/Vo0lNV.png)
该属性为Code，其结构细分为（此处因测试需要讲下Code，更多属性表将在下节讲解）
![](https://s2.ax1x.com/2019/06/15/Voszw9.png)
** 与字段表集合相对应的，如果父类方法在自类中没有被重写，方法表集合中就不会出现来自父类的方法信息。但同样的，有可能会出现有编译器自动添加的方法，最典型的便是类构造器&lt;clinit&gt;方法和实例构造器&lt;init&gt;方法 **

接下来再看descriptor_index为0x000a，指向常量为
![](https://s2.ax1x.com/2019/06/15/VoBn2D.png)
该方法参数列表为空，返回值为void
用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号“()”之内，如void inc()描述为“()V”，方法java.lang.String.toString()描述为“()Ljava/lang/Sting”

### 属性表
接着attributes_count计数为0x0001，有一张属性表，它可能是Code属性，也可能是LineNumberTable属性，这取决于表开头的attribute_name_index，它指向常量池中的名称
#### Code属性
![](https://s2.ax1x.com/2019/06/15/VoDYwR.png)
attribute_name_index为0x000b，指向常量
![](https://s2.ax1x.com/2019/06/15/VorE9K.png)
attribute_length为0x0000002f，属性值占用位数为47个u1

![](https://s2.ax1x.com/2019/06/15/VocMM8.png)
max_stack和max_locals值均为0x0001

+ max_stack代表操作数栈（Operand Stacks）深度的最大值
+ max_locals代表了局部变量表所需的存储空间，单位为Slot，Slot是虚拟机为局部变量分配内存所使用的最小单位。对于byte、char、float、int、short、boolean、returnAddress等长度不超过32位的数据类型，每个局部变量占用1个Slot，而double和long这两种64位的数据类型则需要两个Slot来存放。方法参数（包括实例方法中的隐藏参数“this”）、显式异常处理器的参数、方法体中定义的局部变量都需要使用局部变量表来存放

![](https://s2.ax1x.com/2019/06/15/VoRrdg.png)
code_length和code用来存储Java源程序编译后生成的字节码指令
code_length代表字节码长度，code是用于存储字节码指令的一系列字节流，也叫字节码指令，每个指令都是u1类型，当虚拟机读取到code中的字节码时，会找出对应的指令，并可以知道这条指令后面是否需要跟随参数
u1取值范围0x00~0xFF，一共可以表达256条指令，编码与指令的对应关系可以查询虚拟机字节码指令表

继续以测试类为例，我们现在分析的是&lt;init&gt;方法的Code属性，字节码区域所占空间（code_length）为0x0005，紧随其后的5个字节“2a b7 00 0c b1”对应的字节码指令：

+ 2a：aload_0，将第0个Slot中为reference类型的本地变量推送到操作数栈顶
+ b7：invokespecial，以栈顶的reference类型数据所指向的对象作为方法接收者，调用此对象的实例构造器方法、private方法或者它的父类方法。这个方法有一个u2类型的参数说明具体调用哪一个方法，它指向常量池中的一个CONSTANT_Methodref_info类型常量，即此方法的方法符号引用
+ 00 0c：这是invokespecial的参数，差常量池得0x000A对应的常量为实例构造函数器“&lt;init&gt;”方法的符号引用
![](https://s2.ax1x.com/2019/06/17/VbGNAf.png)
+ 读入B1，对应的指令为return

关于字节码指令，下次我会单独讲解

我们再回头看看javap计算的Code区域
![](https://s2.ax1x.com/2019/06/17/VbYKQH.png)
可以注意到Locals和Args_size的值为1，但方法inc()是没有参数的，并且方法体内也没有定义局部变量，那么为什么呢？不要忘了，在任何实例方法中，都可以是使用**this**关键字访问到此方法所属的对象，而Java编译器编译的时候把对this关键字的访问转变为对一个普通方法参数的访问，然后虚拟机调用实例方法时自动传入此参数，因此实例方法的局部变量表会预留一个Slot位存放对象实例的引用，方法参数从1开始计算，如果是静态方法，那Args_size值就是0

字节码指令之后是这个方法的显式异常处理表集合，这个不是必须的，我们测试类就没有异常表的生成
![](https://s2.ax1x.com/2019/06/17/Vbt5vQ.png)

#### Exceptions 属性
不要与上面的异常表产生混淆，Exception属性的作用是列举出方法中可能抛出的受查异常，也就是方法描述时在throws关键字后面列举的异常
![](https://s2.ax1x.com/2019/06/19/VX2WzF.png)
Exceptions属性中的number_of_exceptions项表示方法可能抛出number_of_exceptions种受查异常，每一种受查异常使用一个exception_index_table项表示，exception_index_table是一个指向常量池中CONSTANT_Class_info型常量得到索引，代表该受查异常得到类型

#### LineNumberTable 属性
用于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系
默认生成到Class文件中，可以在Javac中分别使用-g:none或-g:lines选项来取消或要求生成这项信息，不生成最大的影响就是当抛出异常时，堆栈将不会显示出错的行号，并且调试程序时无法按照源码行来设置断点
![](https://s2.ax1x.com/2019/06/19/VXWRu4.png)

属性表这里就介绍这些，其余表大家可以自己在网上查找