# JSON基础

zero老师,普通话一般,备课不行

序列化与反序列化

## 语法

Object {}  

key - value ,key一般为 "" : value

Array [] 

value,value 

#### Value 可以为 Object 、 int 、String等.

#### key不可为null value可以.

格式良好的json 可以被 gson jackson fastjson等解析库解析

## 解析方式

### Android Studio自带org.json解析



### Gson解析以及原理

#### 使用注解 

##### @SerialzedName("name") 

##### key 名字

##### @Expose(serialize = false , deserialize = true)

#####  是否参与序列号 和 反序列号?

##### @Since(1.0) @Until(1.1) 

##### 版本控制 当前版本满足,参与序列号.  GsonBuilder().setVersion().create();

##### @JsonAdapter(.class) 

##### 如果不定义此注解,那么需要 反射 解析实体类,如果有此注解,注解后面的类class作为解析器

#### 使用GsonBuilder

GsonBuilder 可以设置格式等其他属性.

默认使用**反射TypeAdapter**,耗性能

采取自定义TypeAdapter, 自定义write 自定义read

JsonWriter Bean.class -> Json 

JsonReader Json -> Bean.class

错误:  illegalStateException Json格式问题,   

##### nullSafe();

##### 找后台修复,为了稳定性, 要客户端健壮性修复, 两个方案:

1. 自定义Adapter;
2. 实现 JsonDesrializer 接口;

### Gson原理

1. 按照符号分割,词法分析,构词规则.
2. 源码JsonToken,枚举类,split成不同scope.

1. 一次性载入内存,  

2. 基于事件驱动,压栈,   **类似xml的 SAX.PULL解析.**

"{" - 对象,  做各种操作 , 等到"}" ,pop 继续 

#### JsonElement 

抽象类,判断Element的类型,

JsonNull 不支持转换成其他数据类型

JsonObject = String - JsonElement 

JsonPrimitive = Object 基本类型

#### 解析流程

<img src="/Users/alex/Alex/everyday/TechGoBlog/Java/数据/resource/JSON-解析流程.png" alt="JSON-解析流程" style="zoom:50%;" />

#### Json里的适配器模式 

TypeAdapter abstract

write() and read() 方法

任意的类型都有自己的typeAdapter , 基本数据类型

如果业务Bean,第一种自定义TypeAdapter ,不太可能,因为业务Bean很多,不可能挨个写.

第二种就是其他 使用**ReflectiveTypeAdapter** , TypeToken 获取gson T的类型 ,创建对应的TypeAdapter

#### new Gson();创建Gson类 

#### 门面模式、代理模式、工厂代理、建造者模式(GsonBuilder)

##### excluder 排除器

Expose,Since、Until相关,排除不参与序列化的对象

##### fieldNamingStrategy 命名策略

@SerializeName 相关

##### ConstructorConstructor 构造器

自定义TypeAdapter不需要,但是ReflectTypeAdapter的反射-需要用构造器

等等...

构造中下一步是 创建TypeAdapter工厂列表,**添加所有TypeAdapterFactory**, 优先excluder 、 用户自定义TypeAdapter、默认所有数据类型TypeAdapter,最后是 **ReflectiveTypeAdapter**(兜底).

GsonBuilder().create()方法 创建出Gson对象,

#### 创建完成,开始解析

##### toJson(Object , Type , JsonWriter) 

##### JsonWriter 中 write()方法 value

##### JsonRead 中 doPeek()  peek()  getAdapter() 找到对应的TypeAdapter

### FastJson解析以及原理

