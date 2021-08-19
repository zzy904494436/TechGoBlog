# JVM-内存管理

``` 老师:KING ```

### JVM是一种规范

groovy scala kotlin 都需要JVM规范

卧槽 [/img/a.jpg]

作业:

JDK = JRE + x;

JRE = JVM + 类库

类加载 

JVM配置参数、调优参数 参照 oracle官网

字节码解释器  JIT执行

JVM是由C++写的

#### 解释执行

C++解释器:

if(JAVA_new){

C++代码完成java中的new关键字

}

特点:经过一次JVM翻译,速度相对慢,绝大多数是此种方式执行,JVM后端1w多一点

#### JIT执行(hotspot)

java代码被翻译成 **汇编码 ** (codecache),机器码(10101010),

特点:速度很快,

因为以下两个特点,Java生态圈很牛逼,但是新语言会觉得java很low,其实不然,

### JVM跨平台性

windows、Mac、arm等的JDK不同,

### JVM语言无关性

class规范,

不只有Java、Kotlin、scala等语言,king老师的新语言,任何语言只要符合JVM规范,就可以在JVM上运行

### JVM实现

按照JVM规范文档,就可以实现虚拟机,



1. Hotspot(sun公司)、收购了Jrockit(BEA公司) ,Oracle合并了团队
2. J9(IBM)
3. LiquidVM(BEA公司)
4. Zing(Azul公司,C4算法,算法无版权,被Hotspot借鉴成ZGC)收费
5. TaobaoVM(淘宝,阿里内部)
6. 毕晟(华为,针对openJDK的定制版,对arm架构优化,不支持windows)



### 运行时数据区(重点)

**永久代**,**元空间**是**HotSpot**里的称呼而已; 在JVM规范里都统一称为**方法区**,hotspot只是利用永久代和元空间实现了方法区而已

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_运行时数据区.png" alt="JVM1_运行时数据区" style="zoom: 50%;" />

除了运行时数据区(被虚拟化的内存),还有一块直接内存可以被jvm使用,

内存地址:0x0000012331231230FEDA ....(看不懂)

直接使用内存(C++可以)

注⚠️:8G机器,JVM启动,运行是数据区占用5G,剩余3G,java中,通过某个方式申请,使用、之后需要释放,但是不太方便,因为要直接操作内存,相对麻烦.



### JAVA方法运行和虚拟机栈

main方法-->调用--> A方法-->调用--> B方法-->调用--> C方法 

**每个线程都有虚拟机栈,**每运行一个方法,在**虚拟机栈**中都会压入一个**栈帧**,

此时如图

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_栈帧.png" style="zoom:50%;" />

#### 参数

[详细网址](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)

-Xss 可以调整栈的大小:默认1024KB,如图

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_指令Xss.png" style="zoom:50%;" />

一般情况下1M,如果A方法调用A方法本身,死递归,无限A栈帧压入,抛出栈溢出异常,虚拟机栈不可以动态调整,爆掉了,(循环几百次-上千次)

优化: 内存吃紧场景,系统只剩下200M内存,虚拟机栈来使用,系统多线程大概500个线程,每个虚拟机栈1M,所以需要改小虚拟机栈-Xss 256Kb ,这样可以优化.

随着硬件发展,运行内存上涨,这种问题比较少见.

#### 虚拟机栈的栈帧里面都有什么?栈帧与Java运行

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_栈帧2.png" alt="JVM1_栈帧2" style="zoom:50%;" />

*虚拟机栈*

1. 局部变量表

2. 操作数栈

3. 动态链接

4. 完成出口

   

   *程序计数器*

   

   <img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_栈帧3.png" alt="JVM1_栈帧3" style="zoom:50%;" />

线程1 运行, main方法-work方法(一个方法对应一个栈帧)

分析work方法的栈帧,

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_work方法.png" alt="JVM1_work方法" style="zoom:50%;" />

字节码---class

找到class文件,

javap 反汇编工具 

命令: javap -v Person.class

展示字节码

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_javap.png" alt="JVM1_javap" style="zoom:50%;" />

字节码地址 0 - 12 ; 对应程序计数器;

[指令地址](https://cloud.tencent.com/developer/article/1333540) 

---

`` int x = 1;``

指令:

0:iconst_1   操作数栈压入数值-1

1:istore_1   操作数栈顶的数值 压入 局部变量表1的位置

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM1_work栈帧演示1.png" alt="JVM1_work栈帧演示1" style="zoom:50%;" />

```int y = 2;```

指令:(iconst 1-5 数值大就要bipush)

2:iconst_2   操作数栈压入数值-2

3:istore_2   操作数栈顶的数值 存入 局部变量表2的位置

4:iload_1   把局部变量表下标为1地址的数值**加载**到操作数栈

5:iload_2   把局部变量表下标为2地址的数值**加载**到操作数栈

6:iadd 		操作数栈两个数值加法,**步骤:先出栈;相加;再入栈;**

7:bipush           10    数字-10 压入操作数栈 

9:imul			操作数栈两个数值 **乘法**,**步骤:先出栈;相加;再入栈;**

10:istore_3 	操作数栈顶的数值 **存入** 局部变量表3的位置

11:iload_3	  拿出局部变量表3的位置**加载**到操作数栈

12:ireturn	return带回操作数栈中的数值.

---

#### 疑问解答:

#### 少了个8?

字节码的偏移量,有的指令很大,例如bipush,占用了7、8两个,程序计数器记录的是偏移量.

#### 完成出口是什么

首先,字节码的行号(偏移量)是**本方法从0开始**

在main方法中 调用person.work(),字节码的行号假设为3 , 当work返回时,要返回到上个方法栈帧,那么不能从头运行main,

那么在work()方法中的完成出口为3(**main()方法的的调用栈帧中的偏移量**),接着3执行

#### 程序计数器 可能会重复

会重复,但是没关系,虚拟机栈,只会执行最顶的栈帧,所以程序计数器只记录当前运行的也就是最顶的栈帧,所以不会有问题.

####  动态链接

动态链接跟**多态**有关,过于复杂
