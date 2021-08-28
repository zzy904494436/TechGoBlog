# JVM3-垃圾回收器与性能优化

## 常用的垃圾回收器(JDK-1.8)

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM3-GC分类图(基于JDK1.8).png" alt="JVM3-GC分类图(基于JDK1.8)" style="zoom:50%;" />

### 早期 : Serial 和 SerialOld  - 单线程

早期,JVM堆空间很小, 垃圾回收要暂停所有程序,

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM3-GC早期.png" alt="JVM3-GC早期" style="zoom:50%;" />

```JVM
-XX:+UseSerialGC
```

单线程 ----> 多线程 --->  跨越

### Parallel Scavenge 和 Parallel Old (PS组合)- 多线程

吞吐量优先

#### 自适应: 堆空间动态调整

PS组合 程序启动 需要200M堆空间, GC后,堆空间调整变大-> 250M --> 扩容,伴随着GC .	Eden From To 比例不确定 不在是定死的 8:1:1



```JVM
// 垃圾回收器 暂停时间,时间其实无法得到保障,
// 如果每次都是满了才回收,肯定超过time;
// 如果设置的过短,那么频次会很高,所以体验也不好
// 所以使用这个参数也没有意义
-XX: MaxGCPauseMillis = time // ms 
//如果参数为19,允许最大的垃圾回收时间 占 总时间占比 1 / 19 + 1;  5%
// 默认99, 1%
-XX: GCTimeRatio = 0-100     // 
//启动自适应生成大小 默认开启
-XX:UseAdaptiveSizePolicy 

// 扩容条件:
// 默认 0 可用堆空间低于此值,空间将会被扩容. 如果扩容一定会GC.
-XX : MinHeapFreeRadio  

// 缩容条件: 
// 默认 100
-XX : MaxHeapFreeRadio 


```

吞吐量最大,PS组合

堆空间 最大最小设置一样,这样比较稳定

#### 吞吐优先 --> 响应优先

做游戏服务器,注重体验,服务器卡顿,所有人GC, stop the world

响应优先的垃圾回收器,STW的时间减少,

减少 stop the world 为重点 ,响应时间优先, 于是有了 CMS

## CMS垃圾回收器 (Concurrent Mark Sweep) ---多线程标记清除

CMS回收老年代,所以搭配了一个**ParNew**(多线程)延续了**Serial**(单线程)

**ParNew**针对CMS定制

---

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM3-CMS流程图.png" alt="JVM3-CMS流程图" style="zoom:50%;" />

### 初始标记

首先找到GCRoots,局部变量等GC Roots直接引用的对象, 此时,可达关系(间接引用)并没有标记完

### 并发标记

数量大,层数深,比较耗时,此时并发标记,不占用用户线程,不用暂停;

此时引用变化的问题,A,B,C可能垃圾状态转变为不是垃圾,或者非垃圾转变为垃圾,此时需要重新标记,但是重新标记时间很短,所以不影响性能.

#### 预清理阶段 (阿里P6-P7问题) -- 属于并发标记优化操作

在 **并发标记阶段**,不暂停工作线程,所以在 **并发标记阶段** 尽量做更多的工作,让 **重新标记阶段** 越短越好.尽量做重新标记阶段该做的事情

```JVM
CMSPrecleaningEnable // 默认是true
```

重新标记阶段的某些工作可以提前处理,所以加入预清理;

1. 跨代引用,有Eden引用了 老年代对象(原来是垃圾)

2. 引用发生变化:原本是需要重新标记的, 现在只分区: 1G 分区 100区 每区10M, 若变化了,则 标记 **Dirty** , 便于后面的**重新标记阶段** 不需要扫描全部,记录类似于 **Card Table**

#### 并发可中断的预清理 (阿里P6-P7问题) -- 属于并发标记优化操作

条件触发:**预清理阶段**是Eden与老年代**有引用关系**,但是如果From To区中与老年代**有引用关系**,那么证明 Eden区的有一定的对象,此时From区和To区才会有对象,此时会触发.

```JVM
//参数大概 2M ,当Eden区已经使用了内存2M的对象,才触发开启
CMSScheduleRemarkEdenSizeThreshold = 2 080 000
```

开启后,循环处理下面两件事: 

1. 处理From区和To区的对象 到 老年代可达 ,导致老年代的并发标记中的引用变化.
2. 老年代内部引用变化,处理Dirty分区,同**预清理的处理2** 

判断条件: 

```JVM
循环次数限制 :CMSMaxAbortablePrecleanLoops = 0 	// 0:没有次数限制 
循环时间限制:CMSMaxAbortablePrecleanTime = 5000 	// 时间限制 ms
Eden限制:CMSScheduleRemarkEdenPenetration = 50			// 比例 Eden区内存使用 达到 50%,退出循环
```

##### 之前两步都是为了节约下一步的时间!

### 重新标记

判断哪个地方有变化,因为在并发标记阶段可能存在引用变化,重新标记,以便于最后清理.

Card Table:

### 并发清理

用户线程不需要暂停

#### 重置线程

---

打印日志: -XX: +PrintGCDetails 

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM3-CMS日志.png" alt="JVM3-CMS日志" style="zoom:50%;" />

初始标记 -> 并发标记 -> 预处理 -> 可中断的并发预处理 -> 重新标记 -> 并发清理 -> 重置线程

---

### CMS中的问题

任何版本里面都不是默认的回收器,

1. **CPU敏感**,占用资源,CPU核心不超过4个,CMS会卡;
2. **浮动垃圾**,在**并发清理阶段**,产生的新垃圾,不会被本次清理,要下一次,需要预留一部分地址给浮动垃圾,要提前出发GC,不可能填满100%时再回收, 一般等到92%时,就开始清理,留给浮动垃圾;
3. 内存碎片, 因为标记清除算法,优势虽然是不暂停,但是一定会产生内存碎片.触发时,CMS 会退化为 Serial Old,使用标记整理,一旦触发退化,Serial Old 单线程时间会比较长.一旦产生,系统卡顿几分钟,很差.

JDK1.8,堆空间 8G以上 选择G1,堆空间小于6G,选择CMS. 

JDK1.9之后,默认G1.

---

## JVM调优

### 内存的分代划分

活跃数据(300M)

总大小: 3-4倍 活跃数据 = 1.2G 新生代 1.5倍 = 450M 老年代 2-3倍 750M

#### 扩容新生代能提高GC效率吗? 能,而且是最简单粗棒的方式.

复制算法:Eden区 ---> S0区

如果扩容,把新生代调大, 

---

MinorGC 间隔时间 500ms,对象A存活750ms,新生代容量R, A对象耗时 扫描 + 复制   =  T1 + T2 

假设扩容大小2R,容量变大, Minor GC 间隔时间 1000ms, 而对象A仍存活750ms, 所以来不及换代,就清理了,不需要复制了, 耗时并不会一定变大,A对象耗时 最多 2 * T1,T2不存在了,所以可以提高GC效率.

---

如果 老年代引用新生代, 会被认定为GC Roots,一旦有跨代引用,还要扫描全堆,那么,

### JVM如何避免Minor GC 是扫描全堆?

使用 **CardTable** 可以解决这个问题,

CardTable是一个数组[],每一张卡512B,

若老年代为512M,则卡表size [1 000 000 ],

当有跨代引用,用dirty标记,下一次扫描新生代和老年代内的dirty区域就好了

<img src="/Users/alex/Alex/md_space/TechGoBlog/source/img/JVM3-CardTable避免全堆扫瞄.png" alt="JVM3-CardTable避免全堆扫瞄" style="zoom:50%;" />

---

### 常量池(方法区)

#### Class常量池 ---- 静态常量池

javap -v Person.class  查看

#### 运行时常量池

类加载class文件后吗进入运行时数据区的方法区, 符号引用 变为 直接引用,的字面量 

#### 字符串常量池

没有规范,跟String有关系,话题引入String

---

### String类 JDK1.8

char数组 --- byte数组(JDK1.9)

final修饰 String类 和 char[] ,不可变.如果String可变,不安全问题很大,

那么String a = "a"  + "b"; // 此时经常变,那么内部是创建了新对象,

StringBuffer StringBuilder 之类的又是如何实现的呢? 以下要详细讲解一下原理.

1. String str = "abc"; // 字符串常量创建“abc” 把引用返回 ,池化技术,可复用.
2. String str = new String("abc"); // 字面量abc, 在常量池中创建常量"abc",调用new 创建String对象,并引用常量池中的字符串"abc",返回String对象的引用.多一步,所以不推荐.
3. 对象中的属性,对象在堆中, 对象的 属性 (也在堆中) 引用 指向字符串常量池.
4. String str2 = "a"  + "b" + "c"; // 编译器 "abc"
5. 循环,循环很大的话,用了StringBuilder
6. intern() - 复用常量池中的值 

```java
String a = new String("king").intern(); 
String b = new String("king").intern();
a == b; //返回常量池地址
String c = new String("king");
String d = new String("king");
c != d;	//返回对象地址
```



---

### JDK发展

Java 7 ---  nio ,多线程 fork-join等

**Java 8** --- **LTS版本** ---lamada表达式,并发安全工具类,等

Java 9 --- CMS垃圾回收器进入废弃倒计时,G1称为默认垃圾回收器

Java 10 --- JIT编译器(graal)

**Java 11** ---  **LTS版本** --- **ZGC**长期稳定版本,

Java 12 ---优化 ,新增垃圾回收器

Java 13 --- ZGC 提升空间

Java 14 --- 正式移除 CMS, ZGC开启跨平台 ,unsafe feibudiao

发展方向就是尽量减少 stop the world! 

---

#### 方法区会GC吗?

会,MetaSpace 回收,类太少

#### 持久代 == 永久代 ===> JDK1.8后元空间

#### 初始标记 只标记 GC Roots 直接关联的对象吗?  是的

#### 双亲委派??? 没有回答

