# JVM-内存管理

``` 老师:KING ```

### JVM是一种规范

groovy scala kotlin 都需要JVM规范

卧槽 [/img/a.jpg]

作业:

JDK = JRE + 基础类库 + 工具;

JRE = JVM + 核心类库

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

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_运行时数据区.png" alt="JVM1_运行时数据区" style="zoom: 50%;" />

除了运行时数据区(被虚拟化的内存),还有一块直接内存可以被jvm使用,

内存地址:0x0000012331231230FEDA ....(看不懂)

直接使用内存(C++可以)

注⚠️:8G机器,JVM启动,运行是数据区占用5G,剩余3G,java中,通过某个方式申请,使用、之后需要释放,但是不太方便,因为要直接操作内存,相对麻烦.



### JAVA方法运行和虚拟机栈

main方法-->调用--> A方法-->调用--> B方法-->调用--> C方法 

**每个线程都有虚拟机栈,**每运行一个方法,在**虚拟机栈**中都会压入一个**栈帧**,

此时如图

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_栈帧.png" style="zoom:50%;" />

#### 参数

[详细网址](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)

-Xss 可以调整栈的大小:默认1024KB,如图

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_指令Xss.png" style="zoom:50%;" />

一般情况下1M,如果A方法调用A方法本身,死递归,无限A栈帧压入,抛出栈溢出异常,虚拟机栈不可以动态调整,爆掉了,(循环几百次-上千次)

优化: 内存吃紧场景,系统只剩下200M内存,虚拟机栈来使用,系统多线程大概500个线程,每个虚拟机栈1M,所以需要改小虚拟机栈-Xss 256Kb ,这样可以优化.

随着硬件发展,运行内存上涨,这种问题比较少见.

#### 虚拟机栈的栈帧里面都有什么?栈帧与Java运行

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_栈帧2.png" alt="JVM1_栈帧2" style="zoom:50%;" />

**虚拟机栈** 包含栈帧,**栈帧**包含以下四个

1. **局部变量表**

2. **操作数栈**

3. **动态链接**

4. **完成出口**

   

   *程序计数器*

   

   <img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_栈帧3.png" alt="JVM1_栈帧3" style="zoom:50%;" />

线程1 运行, main方法-work方法(一个方法对应一个栈帧)

分析work方法的栈帧,

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_work方法.png" alt="JVM1_work方法" style="zoom:50%;" />

字节码---class

找到class文件,

javap 反汇编工具 

命令: javap -v Person.class

展示字节码

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_javap.png" alt="JVM1_javap" style="zoom:50%;" />

字节码地址 0 - 12 ; 对应程序计数器;

[指令地址](https://cloud.tencent.com/developer/article/1333540) 

---

`` int x = 1;``

指令:

0:iconst_1   操作数栈压入数值-1

1:istore_1   操作数栈顶的数值 压入 局部变量表1的位置

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_work栈帧演示1.png" alt="JVM1_work栈帧演示1" style="zoom:50%;" />

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

#### 静态方法和普通方法的区别

栈帧中的局部变量表第一个index-0 中 是否有this ,普通方法需要this来指向当前对象,而静态方法不需要this.

---

虚拟机栈 ---栈帧----4个:**1局部变量表2操作数栈3动态连接4完成出口**

本地方法栈(类似虚拟机栈)

程序计数器

---

### Java中直接操作不了线程

本地方法----操作系统,特殊的库(native)

##### hashCode() --- native修饰 就会调用本地方法 --- **本地方法栈**

本地方法栈,在hotspot合二为一,**在hotspot,不作区分**,



### 方法区 

ObjectAndClass定义 

静态变量(基本数据类型)

常量(基本数据类型)

成员变量(对象)

成员变量(基本数据类型)

方法内,

局部变量(基本数据类型)

局部变量(对象)

#### 类加载时

1. class类 类加载 放入 方法区
2. 静态变量、常量 放入方法区
3. 成员变量 new 对象 类加载时 不会执行 
4. 成员变量(基本数据类型) 也不会执行
5. 类加载会执行 静态代码块

#### main方法

局部变量(基本数据) 放入 虚拟机栈 栈帧

局部变量(对象) 对象在堆中分配, 局部变量的引用 放入 虚拟机栈的 局部变量表, 

此时 上边3的成员变量 与1局部变量(对象)一起在堆中分配  此对象 包含 成员变量

### 直接内存

ByteBuffer.allocateDirect(128 * 1024 * 1024); 直接分配128M内存

底层操作是用unsafe类,unsafe类已经跨越了虚拟机规范了

Java9本来要移除unsafe,结果很多缓存框架用了unsafe

unsafe可以操作对象.内存,地址.方法偏移量,找到class,唤醒线程,内存屏障等

直接内存申请会出现问题,绕过垃圾回收,手动申请,好处:速度快,缺点:忘记释放---内存泄露,无法控制地址,如果对底层了解,那么性能确实提升,你做java程序员,你就是小白,肯定玩不好啊😂

ByteBuffer一个线程不断检查内存问题,所有不会造成内存泄露.

### JVM整体内存结构

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_整体内存图.png" alt="JVM1_整体内存图" style="zoom:50%;" />

### 用代码深入理解到1010

VM参数设置

jps  // 进程状态列表

jinfo -flags 10004  //进程默认参数

#### 整体流程

1. JVM申请内存

2. 初始化运行时数据区

3. 类加载

   根据VM参数,申请方法区和堆内存等内存空间;类加载,把class和常量放入方法区

4. 运行方法,创建栈(虚拟机栈,本地方法栈),

5. 创建对象在堆中,引用在栈中;不断重复创建对象,执行方法,

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_深入理解1.png" alt="JVM1_深入理解1" style="zoom:50%;" />

### 堆空间分代划分

新生代,老年代(多次垃圾回收,但是变量有引用,没回收掉)

### GC概念(自动回收)

### JHSDB工具 :

可以查看内存,运行时数据区,可以看到虚拟的也可以看到真实的,

```bash
// mac地址:/Library/Java/JavaVirtualMachines/jdk1.8.0_261.jdk/Contents/Home/lib
java -cp ./sa-jdi.jar sun.jvm.hotspot.HSDB //命令在/jdk/lib下执行 
```

jps 进程ID列表

Attach 某process ID ,出现Java Threads :

Tools -> Heap Parameters ,堆空间细分

堆空间就是物理地址的虚拟化,0x0000000013400000 - 0x0000000015200000

##### 新生代Gen0: 

Eden区: xxx ---xxx

From区: xxx ---xxx

To区: xxx ---xxx

##### 老年代Gen1:xxx ---xxx

#### 对象地址

Tools -> Object Histogram -> search 

包名.class名字

#### 查看栈

线程中查找

#### 查看方法区

找Class Browser

等等

---

### 内存溢出

### 栈溢出

StackOverFlow

### 堆溢出

new 一个很大的数组 超出堆大小 ,就会堆溢出 OOM异常 

### 方法区溢出

OOM MetaSpace

### 直接内存

申请的内存比VM设置参数的大

---



#### 为什么为15次后,新生代  ---> 老年代?

对象经历1次垃圾回收,没被回收 年龄+ 1

age达到15后,晋级为老年代,

底层记录对象的年龄 ,4位2进制 最大 1111

#### 物理地址,是操作系统给!

#### 构造方法,也是栈帧!

#### 所有类都会一次性加载到方法区吗?

不会,按需!

#### 方法结束后,出站后,操作数栈的return怎么回到返回的方法?

操作数栈,属于执行引擎的一部分,JVM的操作数栈 类似于 CPU寄存器.

#### 为何如此设计?

跟计算机组成架构有关系,

硬件:CPU、高速缓存、内存

虚拟:执行引擎、栈、堆空间

#### unsafe申请的内存地址是操作系统给的内存地址?

是的

#### 操作系统的给的也tm是虚拟地址

#### void方法有完成出口吗?

都有!有返回值和美有返回值,在程序计数器中是否有数值,都有完成出口!

#### 关于新生代和老年代,有什么区别?

下一节课

#### 方法区放类,堆放对象的思考

使用Netty不规划,是因为使用了直接内存

#### 运行时常量池在方法区,有时候也在堆.

JVM虚拟机规范,没规定.

---

## JVM方向

**写JVM**,加强执行引擎+加强语法. +GC引擎

**语言研究**,java --> class ,学习编译原理,并且学习JVM规范,枯燥,造一门语言

---



<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/JVM/resource/JVM1_作业.png" alt="JVM1_作业" style="zoom:67%;" />

---

### 下期 : 对象分配+垃圾回收机制



