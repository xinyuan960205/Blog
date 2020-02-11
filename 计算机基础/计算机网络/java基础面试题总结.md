## java基础

- 面向对象特性介绍、与C++区别
- 



## 数据类型

- 装箱拆箱
- java的数据类型
- String
  - 存储结构，8,9改变
  - String pool
  - 基本类型和String的相互转换
  - 字符串拼接
  - subString

- StringBuffer和StringBuilder

## 基本语法

- Java异常体系
- static关键字和final关键字使用情况，一个类不能被继承，除了final关键字之外，还有什么方法（从构造函数考虑）？
- 几个关键字都要掌握清楚，能说出来都是干嘛的
- 序列化和反序列化。反序列化失败的场景。
- 正则表达式会写吗？
- Java注解的原理



## 面向对象

- 抽象类和接口区别，以及各自的使用场景
- 继承与多态、重写和重载
- 多态实现原理
- 泛型以及泛型擦除。List<A>类型的list,可以加入无继承关系的B类型对象吗？如何加入？
- 反射原理以及使用场景
- Object类的方法



## java容器（常见集合的区别和实现原理，都背下来）

- ArrayList和LinkedList的区别和底层实现？如何实现线程安全？

- List遍历时如何删除元素？fail—fast是什么？fail—safe是什么？

- HashMap

  - 底层结构
    - 数组 + 链表 
  - 扩容情况
  - Put 的过程 
  - 哈希函数，为什么长度是2的倍数
    - 找索引时 key 的 hash 值与数组的长度值减 1 进行与运算，长度为 2 的倍数时能减少碰撞 
  - JDK 1.7 和 1.8 中 HashMap 的区别
    - 1.8 增加红黑树、头插变为尾插、扩容后元素位置要么在原位置，要么在原位置 + 扩容前旧容量 
  - 为什么线程不安全
    - 扩容时链表可能形成闭环 
    - 如何实现线程安全

  - 容量为什么始终都是2^N

- ConcurrentHashMap 

  - ConcurrentHashMap 怎么保证线程安全，底层实现 
  - ConcurrentHashMap 和 HashMap 区别 
  - JDK1.7与JDK1.8的区别

- 集合类的结构

  - Iterator、Collection（List、Set、Queue）、Map



## java IO

- IO/NIO 包，主要结合 IO 模型说（实际上 IO 包已经用 NIO 包重新实现过了）
- Java的io模型







