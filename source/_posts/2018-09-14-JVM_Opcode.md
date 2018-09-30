---
title: 深入理解JAVA虚拟机（四）  字节码指令
date: 2018/09/14 21:14
updated: 
tag: 
- JVM
- 字节码
categories:
- Java

---
Java虚拟的指令是由一个字节长度的、代表着特殊含义的数字（操作码）及其后跟随零个或多个操作数构成。
<!-- more -->

# 字节码指令特点：

* 数量有限：因一个字节长度的限制，最大值支持256条（种）指令
* 操作数不对齐：处理长度超过1字节的操作数时，需在字节种构造出具体数据结构（时间换空间）。

# 指令

ps：
1. 由于JVM为基于栈的指令架构，后续指的栈均指操作数栈。
2. 操作码中出现的T可被替换成指定类型
3. 表中出栈顺序为后序先出，入栈为前序先进
4. 所有指定了操作码操作数类型的指令，其操作数必须是所指定的类型
5. **操作码表存在缺误，仅供参考**，具体请查阅[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5) 。

## 加载和存储指令

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|Tload<br/>(i\l\f\d\a)|index|将局部变量表中索引为index的值压栈|-|value|--|可以与wide指令一起使用|
|Tload_&lt;t&gt;<br/>(i\l\f\d\a)|-|将局部变量表中索引为&lt;t&gt;的值压栈|-|value|--|可以与wide指令一起使用|
|Tstore<br/>(i\l\f\d\a)|index|将栈顶数据弹出，存到局部变量表index的位置||value|-|--|可以与wide指令一起使用|
|Tstore_&lt;t&gt;<br/>(i\l\f\d\a)|-|将栈顶数据弹出，存到局部变量表&lt;t&gt;的位置|value|-|--|可以与wide指令一起使用|
|bipush|byte|将byte值压栈|-|value|-|入栈前该数据会被有符号拓展为int型|
|sipush|byte1 byte2|将值为byte1<<8 &#124 byte1的数据压栈|-|value|-|入栈前该数据会被有符号拓展为int型|
|Tconst_&lt;t&gt;<br/>(i\l\f\d\a)|-|将某个固定常量压栈|-|&lt;t&gt;|-|具体的常量值与T及&lt;t&gt有关，可见附表|
|ldc|index|将常量池中索引为index的数据压栈|-|value|类解析、方法解析异常|必须是int、float或字符串常量的直接引用或类、方法的字符引用|
|ldc_w|byte1 byte2|将常量池中索引为index的数据压栈(宽索引)|-|value|类解析、方法解析异常|同上|
|ldc2_w|byte1 byte2|将常量池中索引为index的数据压栈(宽索引)|-|value|类解析、方法解析异常|必须是long、double|
|wide(1)|&lt;opcode&gt; indexbyte1 indexbyte2|将指定指令的操作数拓展为2字节|-|-|-|iload, fload, aload, lload, dload, istore, fstore, astore, lstore, dstore, ret之一|
|wide(2)|iinc indexbyte1 indexbyte2 constbyte1 constbyte2|将指定指令的操作数拓展为2字节|-|-|-|-|

* const常数指令：

|指令|对应关系，格式为(&lt;t&gt;,...):(const,...)|
|--|--|
|iconst_&lt;t&gt;|(m1,0,1,2,3,4,5) : (-1,0,1,2,3,4,5)|
|lconst_&lt;t&gt;|(0,1) : (0,1)|
|fconst_&lt;t&gt;|(0,1,2) : (0.0f,1.0f,2.0f)|
|dconst_&lt;t&gt;|(0,1) : (0.0,1.0)|
|aconst_&lt;t&gt;|(null) : (null)|

## 运算指令

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|Tadd<br/>（i\l\f\d）|-|将栈顶两个数弹栈并相加后压栈|value1 value2|result|--|会溢出，不会报错|
|Tsub<br/>（i\l\f\d）|-|将栈顶两个数弹栈并相减后压栈|value1 value2|result|--|结果为value1-value2，会溢出，不会报错|
|Tmul<br/>（i\l\f\d）|-|将栈顶两个数弹栈并相减乘压栈|value1 value2|result|--|会溢出，不会报错|
|Tdiv<br/>（i\l\f\d）|-|将栈顶两个数弹栈并相减后压栈|value1 value2|result|--|结果为value1/value2|
|Trem<br/>（i\l\f\d）|-|将栈顶两个数弹栈并求模后压栈|value1 value2|result|--|结果为value1%value2|
|Tneg<br/>（i\l\f\d）|-|将栈顶一个数弹栈并取反后压栈|value1|result|--|结果为value1%value2|
|Tshl，Tshr，Tushr<br/>（i\l）|-|将栈顶两个数弹栈，对value1进行value2位移动后压栈|value1 value2|result|--|shl：左，shr带符号右移（高位补符号），ushr无符号右移（高位补0），实际上，i位移数只取低5位|
|Tor，Tand，Txor<br/>（i\l）|-|将栈顶两个数弹栈，按位操作并压栈|value1 value2|result|--|按位与、或、非并压栈|
|iinc|index const|将局部变量表索引为index的参数加上const值并保存|-|result|-|-|
|Tcmp，Tcmpl，Tcmpg<br/>（l\f\d）|-|弹出栈顶两个数，比较v1和v2大小，大于=>1,等于=>0，小于=>-1|value1 value2|result|-|long型只有lcmp，当两个数有NaN时cmpg返回1，cmpl返回-1；比较结果是int型；|

* 除idiv、ldiv，irem、lrem指令遇到除数为0时才会报错，float和double会返回NaN
* 除比较外，任何关于NaN的运算都会返回NaN
* 浮点型的计算结果都会被舍入到能够表达的精度，优先采用最低有效位为0的舍入模式。
* 浮点型比较。擦欧派那个无符号比较的方式

## 类型转换

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|i2T<br/>（b\s\l\f\d）|-|从栈顶弹出一个数，强转为指定类型，后压栈|value|result|-|-|
|l2T<br/>（i\f\d）|-|从栈顶弹出一个数，强转为指定类型，后压栈|value|result|-|-|
|f2T<br/>（i\l\d）|-|从栈顶弹出一个数，强转为指定类型，后压栈|value|result|-|-|
|d2T<br/>（i\l\f）|-|从栈顶弹出一个数，强转为指定类型，后压栈|value|result|-|-|

* 虚拟中操作中，若本数据类型的操作码，则boolean、char被零位拓展成int，byte、short被带符号拓展成int
* 宽化类型转换一般时安全的：包括i2l,i2f,i2d;l2f,l2d;f2d
* 窄化类型转换则需显示转换；窄化转换只是简单舍弃了高位，因此会出现符号不同等问题。
* NaN转化为整数型数据时，结果为0
* 浮点型转化成整数型时，小数部分会直接被舍弃，因此，其值只会小于原来的数字
* 若浮点不为无穷大，且整数部分的范围在目标整数的范围内，则其值为浮点的整数部分值，否则，结果为目标整数类型能表示的最大值
* 基础的
* 类型转化不会报错

## 对象创建及访问

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|new|indexbyte1 indexbyte2|从常量池的indexByte1<<8 &#124;&#124; indexByte2位置去除类的符号引用，并创建实例，将实例应用压栈|-|instanceRef|InstantiationError、错误|-|
|newarray|typebyte|创建基本类型数组|count|arrRef|NegativeArraySizeException|4:boolean 5:char 6:float 7:double 8:byte 9:short 10:int 11:long|
|multianewarray|indexbyte indexbyte2 dimensions|创建类型为index指向的多维数组|count1, count2, ...|arrayref|IllegalAccessError，NegativeArraySizeException|dimensions（维度）|
|getField|indexbyte1 indexbyte2|从对象处获取字段值index指向常量池中字段的符号引用| objectref |value|IncompatibleClassChangeError， NullPointerException|getfield指令不能用于访问数组的长度字段|
|getStatic|indexbyte1 indexbyte2|获取类的静态属性值|-|value| IncompatibleClassChangeError， Error|-|
|putField|indexbyte1 indexbyte2|设置对象实例某字段的值| objectref, value |-|IncompatibleClassChangeError， IllegalAccessError，NullPointerException|-|
|putStatic|indexbyte1 indexbyte2|设置类静态字段的值|  value |-|IncompatibleClassChangeError， IllegalAccessError，Error|仅可用于在该字段的初始化时设置接口字段的值。|
|Taload<br/>（b\c\s\i\l\f\d\a）|-|加载数组中index的值|   arrayref, index|value|NullPointerException，ArrayIndexOutOfBoundsException|-|
|Tastore<br/>（b\c\s\i\l\f\d\a）|-|加载数组中index的值|   arrayref, index, value|-|NullPointerException，ArrayIndexOutOfBoundsException，ArrayStoreException|-|
|arraylength|-|获取数组长度|arrayref |length|NullPointerException|-|
|instanceof|indexbyte1 indexbyte2|判断对象是否为指定类型，index指运行时常量池中的索引|objectref |result|IllegalAccessError|遇到null推送结果|
|checkcast|indexbyte1 indexbyte2|检查对象是否为给定类型|objectref|objectref |-|遇到null报错，对操作数栈有影响|

## 操作数栈管理指令

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|pop|-|弹出栈顶的一个元素|value|-|-|栈顶元素必须是基础类型、引用类型或返回地址|
|pop2|-|弹出栈顶的一个或两个（双字节）元素|value、[value2]|-|-|栈顶元素必须是基础类型、引用类型或返回地址|
|dup，dup2，dup_x1，dup2_x1，dup_x2，dup2_x2|-|弹出栈顶的一个或两个元素，复制单份或双份，重新入栈|~~~|~~~|-|根据操作数的类型，出入栈情况不同|
|swap|-|交换栈顶元素的位置|value2，value1|value1，value2|-|不支持双字节数据交换位置|

## 控制转移

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|if<cond>（ifeq、ifne、iflt、ifge、ifgt、ifle、ifnonnull、ifnull）|branchbyte1 branchbyte2|比较栈顶的元素（int或reference）与0或null的关系|value|-|-|若比较成功，则跳转到从该指令地址开始，偏移量为branchbyte1 << 8 &#124； branchbyte2的地址执行，否则，顺序执行|
|if_acmp<cond>（if_acmpeq、if_acmpne）|branchbyte1 branchbyte2|比较栈顶的两个引用元素是否相同|value1，value2|-|-|若比较成功，则跳转到从该指令地址开始，偏移量为branchbyte1 << 8 &#124； branchbyte2的地址执行，否则，顺序执行|
|if_icmp<cond>（if_icmpeq、if_icmpne、if_icmplt、if_icmple、if_icmpgt、if_icmpge）|branchbyte1 branchbyte2|比较栈顶的两个int元素元素是否相同|value1，value2|-|-|若比较成功，则跳转到从该指令地址开始，偏移量为branchbyte1 << 8 &#124； branchbyte2的地址执行，否则，顺序执行|
|tableswitch|~~|通过索引和跳转访问跳转表|index|-|-|指令较复杂，请参阅官方文档|
|lookupswitch|~~|通过键匹配和跳转访问跳转表|key|-|-|指令较复杂，请参阅官方文档|
|goto|branchbyte1 branchbyte2|无条件转移|-|-|-|跳转到从该指令地址开始，偏移量为branchbyte1 << 8 &#124； branchbyte2的地址执行|
|goto_w|branchbyte1 branchbyte2 branchbyte3 branchbyte4|无条件转移（宽指令）|-|address|-|跳转到从该指令地址开始，偏移量为branchbyte1 << 24) &#124； (branchbyte2 << 16) &#124； (branchbyte3 << 8) &#124； branchbyte4.的地址执行|
|jsr|branchbyte1 branchbyte2|跳转子程序|-|-|-|紧跟在此jsr指令之后的指令的操作码的地址被作为returnAddress类型的值压入操作数堆栈|
|jsr_w|branchbyte1 branchbyte2 branchbyte3 branchbyte4|跳转子程序（宽指令）|-|address|-|紧跟在此jsr_w指令之后的指令的操作码的地址被推送到操作数堆栈，作为返回类型的值|
|ret|index|从子程序返回|-|-|-|索引是0到255之间的无符号字节，包括0和255|

## 方法调用和返回

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|invokevirtual|indexbyte1 indexbyte2|根据对象的实例类型进行分派|objectref, [arg1, [arg2 ...]]|-|Run-time Exceptions|-|
|invokeinterface|indexbyte1 indexbyte2 count 0|调用接口方法，找出合适的对象调用|objectref, [arg1, [arg2 ...]]|-|Run-time Exceptions|-|
|invokespecial|indexbyte1 indexbyte2|调用实例初始化方法、私有方法和父类方法|objectref, [arg1, [arg2 ...]]|-|Run-time Exceptions|-|
|invokestatic|indexbyte1 indexbyte2|调用类的静态方法|objectref, [arg1, [arg2 ...]]|-|Run-time Exceptions|-|
|invokedynamic|indexbyte1 indexbyte2 0 0|调用动态方法| [arg1, [arg2 ...]]|-|Run-time Exceptions|-|

## 方法调用和返回

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|athrow|--|显式抛错|objectref|objectref|Run-time Exceptions|-|

* 如果在当前方法中匹配此异常的处理程序，则athrow指令会丢弃操作数堆栈上的所有值，然后将抛出的对象推送到操作数堆栈。
* 如果当前方法中没有匹配处理程序，并且在方法调用链的更远处抛出异常，则清除处理异常的方法（如果有）的操作数堆栈，并将objectref推送到该空操作数堆栈。
* 从抛出异常的方法到处理异常的方法（但不包括处理异常的方法）中的所有中间帧都被丢弃。


## 同步指令

|操作码<br/>opCode|操作数<br/>operands|描述<br/>Operation| 出栈 | 入栈 |异常|备注|
|:--:|--|--|--|--|--|--|
|monitorenter|-|进入监听（同步）|objectref|-|Run-time Exceptions|-|
|monitorexit|-|退出监听（同步）|objectref|-|Run-time Exceptions|-|

* monitorenter指令可以与一个或多个monitorexit指令（§monitorexit）一起使用，以在Java编程语言（第3.14节）中实现同步语句。
* monitorenter和monitorexit指令不用于同步方法的实现，尽管它们可用于提供等效的锁定语义。
* 调用synchronized方法的监视条目，并在其返回时监视退出，由Java虚拟机的方法调用和返回指令隐式处理，就像使用monitorenter和monitorexit一样。
* 可以以超出本说明书范围的各种方式管理监视器与对象的关联。例如，可以与对象同时分配和释放监视器。或者，它可以在线程试图获得对对象的独占访问时动态分配，并且在稍后没有线程保留在对象的监视器中时释放。
* 除了进入和退出之外，Java编程语言的同步构造还需要支持监视器上的操作。

*** 

问题1：索引指的是字节的地址还是顺序？
答：指**栈帧中局部变量表**集合的索引。2018-09-15
问题2：iload 0x01 指的是加载局部变量的第一个元素还是指局部变量表的第一个int元素？若是后者怎么判断局部变量表中数据类型？
答：指**栈帧中局部变量表**集合的索引为1的数据。2018-09-15
问题3：CODE属性表中的局部变量表是否与类的字段表结构一致。
问题4：局部变量处的初始值怎么来？
问题5：double类型中，怎么区分小数部分和整数部分
问题6：instanceof规则？

