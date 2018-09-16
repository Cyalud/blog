---
title: 深入理解JAVA虚拟机（一）  运行时数据区
date: 2018/9/11 20:15:25
updated: 
tag: 
- JVM
- 内存分配
categories:
- Java

---

JVM的书过了一遍，写点博客总结记录下，顺便再过第二遍。

<!-- more -->

# About JAVA

* 定义：面向对象 编程语言
* 特性：1、封装 继承 多态；2、平台无关性（Write Once， Run Anywhere）
* 引申：由软件及规范形成的技术体系

# About JVM

* 定义：用于计算设备的规范 虚拟的计算机 
* 用途：执行字节码文件
* 主流JVM：Sun HotSpot（Sun默认）；BEA JRockit（性能高）；IBM J9（IBM自用较多）  

<small>三个虚拟机的对比请见<https://blog.csdn.net/chenyi8888/article/details/4722410></small>

***
华丽的分割线
***

# JAVA运行时数据区

根据[JAVA虚拟机规范（JAVA SE7）](https://docs.oracle.com/javase/specs/index.html)的规定，JAVA运行时内存分以下区域：
 
{% asset_img RuntimeMemory.png JAVA运行时内存 %}


## 程序计数器

* 概念：当前线程执行字节码行号的指示器
* 用途：用以选取线程需执行的下一字节码（的位置？个人理解）
* 特性：线程私有（每个线程需由独立的程序计数器，线程执行才不会混乱）
* 其他：执行JAVA方法时，指向正在执行的JAVA字节码指令地址；执行Native方法时，为undefined；
* 不会报错

## JAVA虚拟机栈

* 每个方法创建时都会创建一个栈帧（Stack Frame），方法从调用到结束对应栈帧的出栈及入栈
* 栈帧包含：  
&nbsp;&nbsp;&nbsp;&nbsp;**局部变量表**：最小单位：slot（32位），通过索引来访问。  
&nbsp;&nbsp;&nbsp;&nbsp;主要存放1.基本数据类型（boolean、byte、char、short、int、float、long、double）；2.对象引用（对象起始地址、句柄、或其他对象相关位置）；3.returnAddress。  
&nbsp;&nbsp;&nbsp;&nbsp; **操作数栈**：因JVM没有寄存器，指令是从操作数栈中取得操作数（数据通过压入-弹出进行访问），存储的数据类型与局部变量表一致（但byte、char、short类型入栈会被转成int型）。
&nbsp;&nbsp;&nbsp;&nbsp;**动态链接**：指向运行时常量池中，该栈帧所属方法的引用（常量池中方法的符号引用），主要为了支持方法调用过程中的动态链接。
&nbsp;&nbsp;&nbsp;&nbsp;**方法出口**：记录上个方法的计数器信息，供方法正常退出时使用。
* 栈帧中局部变量表大小在编译后就已经确认
* 每个栈帧并非完全独立，存在小部分的重叠（操作数栈与局部量表）（个人理解，对应方法的传参，避免多余的复制操作及存储空间占用）
* 线程私有
* 会抛出StackOverflowException（栈深超过JVM限定深度），及OutOfMemoryException（动态拓展超过JVM内存）

## 本地方法栈

* 不属于虚拟机规范，用于执行Native方法
* 与虚拟机栈作用相似
* HotSpot中，不区分虚拟机栈与本地方法栈（为同一个内存模型区域）

## JAVA堆

* 存储对象实例、数组
* 线程共享（通过多个线程私有的分配缓冲区（TLAB），确保对象**创建**时内存区域不重叠）
* GC主要管理区域
* 基于分代收集算法，可细分为：新生代、老年代;进一步划分为Eden、From Survivor、To Survivor(三者大小比例默认8：1：1，可通过过参数修改）
* 可处于物理不连续区域，只要逻辑连续即可
* 会抛出OutOfMemoryException：Java heap space

注：对象实例并非一定需要存放在堆内存中

## 方法区

* 存储被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等。
* 在HotSpot中，也被称为永久代  
&nbsp;&nbsp;&nbsp;&nbsp;**JDK1.6及以前版本**，方法区也被称为永久代（原因：为管理方法区内存，将分代收集算法延申至方法区，省去专门编写垃圾回收）内存回收主要针对常量池及类的卸载  
&nbsp;&nbsp;&nbsp;&nbsp;**JDK1.7**，保留永久代，但字符串常量移动到了Java 堆（Heap）中开辟的一块区域中   
&nbsp;&nbsp;&nbsp;&nbsp;**JDK1.8——**， 元空间替代了了永久代，将方法区直接放在一个与堆不相连的本地内存区域（元空间） 
* 会抛出OutOfMemoryException：PermGen space

## 运行时常量池

* 存放类编译时期生成的各种1.字面常量（文本字符串、声明为final的常量值）；2.符号引用（类和接口的全限定名、字段名称和描述符、方法名称和描述符）
* 作用：1.节省内存空间（相同字符串常量被合并，减少空间占用）2.节省运行时间（避免频繁创建及销毁对象）
* HotSpot中，jdk1,6常量池放在方法区，jdk1.7常量池放在堆内存，jdk1.8放在元空间里面

## 直接内存

* 不属于JAVA运行时数据区，不再JAVA虚拟机规范中
* 会抛出OutOfMemoryException

## 元空间（JDK1.8，HotSpot）

* 永久代的替代方案
* 之存放类信息（Classes、Metadata），其他的数据，都被放到到Java堆上
* 会抛出OutOfMemoryException：Metaspace

## 本地内存（native Memory）

JVM管理的内存可以总体划分为两部分：Heap Memory和Native Memory。Heap Memory 如上所述，可通过参数设置大小。Native Memory没有相应的参数来控制大小，其大小依赖于操作系统进程的最大值（对于32位系统就是3~4G，各种系统的实现并不一样），以及生成的Java字节码大小、创建的线程数量、维持java对象的状态信息大小（用于GC）以及一些第三方的包，比如JDBC驱动使用的native内存。

对于IBM的JVM某些版本实现，类加载器和类信息都是保存在Native Memory中的。

{% asset_img DirectBuffer.png JAVA内存 %}

## 其他



# HotSpot对象创建

## 普通对象创建过程

当虚拟机遇到new 关键字时

1. 常量池由类的符号引用 && 判断类已加载、解析、初始化？是3：否2；
2. 加载类（加载-链接（验证-准备-解析）-初始化）
3. 分配堆内存：指针碰撞（Bump the Pointer）（内存规整时，使用带整理算法的CG收集器如：Seral、ParNew），或空闲列表（FreeList，使用带清除算法的GC收集器，如CMS等）；同时，为保证内存分配的线程安全，可采用TLAB（可不使用），使用TLAB只需在TLAB用完并分配新的TLAB时，才需要同步锁定。
4. 内存空间初始化（保证对象不赋值就可以直接使用）
5. 设置对象头（类的元数据信息、对象HASH、GC分代年龄等）
6. 执行对象的init方法

## 对象内存布局

对象在堆中的存储布局分为3块  
1. 对象头  
 * 运行时数据：数据长度根据虚拟机的位数的不同，分别为32bit及64bit存放HashCode、GC分代年龄、锁状态等   
 * 类型指针：指向该实例的类元数据  
2. 实例数据
 * 存放父类及本身定义的各种类型数据
 * 受分配策略（HotSpot中，long/double，int，short/Char，byte，boolean，oop分配到一起）及源码定义顺序影响（父类元素在子类元素之前）
 * 若CompactFields参数为true（默认true），子类较窄的变量可能会插到父类变量中
3. 对齐填充
 * 不必然存在，HotSpot中，对象的起始地址必须是8字节的整数倍，若对象实例数据不齐时，则需对其填充

## 对象的访问定位

* 句柄访问：在堆中开辟一块内存，作为句柄池，栈中存储句柄地址，句柄存放对象实例及对象类型数据地址信息。优势：稳定。
* 直接地址：栈中直接指向堆中对象实例地址，对象实例存放对象类型数据地址。优势：快速。

---

疑问1：boolean进栈是什么类型；  
疑问2：byte类型经过压入、弹出后怎么转成byte。  
~~疑问3：final static的属性（String，int等）放在哪个区域~~  
答：运行时常量池
疑问4：数组相关，数组放哪？怎么引用？怎么初始化？
疑问5：堆中存放的对象实例，放的是值还是引用符号？或者是什么？怎么知道哪个实际值是对应对象的哪个字段？
