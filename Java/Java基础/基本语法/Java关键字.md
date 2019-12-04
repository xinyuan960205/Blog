# Java关键字
## 一、this
- 普通方法中，this总是指向当前调用该方法的对象。此关键字是可选的，这意味着如果上面的示例在不使用此关键字的情况下表现相同。但是，使用此关键字可能会使代码更易读或易懂。
- 构造方法中，this总是指向正要初始化（创建）的对象。用于在构造方法中引用满足指定参数类型的构造器（其实也就是构造方法）。但是这里必须非常注意：只能引用一个构造方法且必须位于第一位！
- this不能用于static方法。


```java
public class Student {

    //静态数据
    String name;
    int id;

    //set方法
    public void setName(String name) {
        this.name = name;
    }
    
    //构造方法
    public Student(String name,int id){
        this();//通过this调用其他构造方法，必须位于第一句！
               //Constructor call must be the first statement in a constructor
        //this(name);
        /*此处不能用，因为其他任何方法都不能调用构造器，只有构造方法能调用他。
       但是必须注意：就算是构造方法调用构造器，也必须为于其第一行，构造方法也只能调
       用一个且仅一次构造器！*/
        this.name=name;
        this.id=id;
    }
    public Student(String name){
        this.name= name;
    }
    
    public Student(){
        System.out.println("构造一个对象");
    }
    //动态的行为
    public void study(){
        this.name="熊二";//this表示当前对象，在普通方法中，加与不加含义是一样的。
        System.out.println(name+"在学习");
    }
    
    public void sayHello(String sname){
        System.out.println(name + "向" +sname +"说，你好~");
    }
    
}
```

## 二、super
super关键字用于从子类访问父类的变量和方法。 例如：
```java
public class Super {
    protected int number;
     
    protected showNumber() {
        System.out.println("number = " + number);
    }
}
 
public class Sub extends Super {
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```
在上面的例子中，Sub 类访问父类成员变量 number 并调用其其父类 Super 的 `showNumber（）` 方法。

**使用 this 和 super 要注意的问题：**<br>
在构造器中使用 super（） 调用父类中的其他构造方法时，该语句必须处于构造器的首行，否则编译器会报错。另外，this 调用本类中的其他构造方法时，也要放在首行。
this、super不能用在static方法中。

**简单解释一下：**<br>
被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享。而 this 代表对本类对象的引用，指向本类对象；而 super 代表对父类对象的引用，指向父类对象；所以， this和super是属于对象范畴的东西，而静态方法是属于类范畴的东西。

## 三、final
Java关键字final有“这是无法改变的”或者“终态的”含义，它可以修饰非抽象类、非抽象类成员方法和变量。<br>
final关键字主要用在三个地方：变量、方法、类。

### 1. 数据

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

对于基本类型，final 使数值不变；<br>
对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。
```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```
### 2. 方法

声明方法不能被子类重写，但可以被重载！

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

使用final方法的原因有二： 
- 第一、把方法锁定，防止任何继承类修改它的意义和实现。 
- 第二、高效。编译器在遇到调用final方法时候会转入内嵌机制，大大提高执行效率。

### 3. 类

声明类不允许被继承。比如：String，Math。<br>
final类中的成员变量可以根据需要设为final，但是要注意final类中的所有成员方法都会被隐式地指定为final方法。

## 四、static
### 1. 修饰成员变量和成员方法(常用)
#### 静态成员变量
原来一个类里面的成员变量，每new一个对象，这个对象就有一份自己的成员变量，因为这些成员变量都不是静态成员变量。对于static成员变量来说，这个成员变量只有一份，而且这一份是这个类所有的对象共享。

◆在类中，用static声明的成员变量为静态变量，或者叫:类属性、类变量。

（注意：静态变量是从属于类，在对象里面是没有这个属性的；成员变量是从属于对象的，有了对象才有那个属性） 

它为该类的公用变量，属于类，被该类的所有实例共享，在类被载入时被显示初始化。
对于该类所有对象来说，static成员变量只有一份.被该类的所有对象共享！！
可以使用“对象.类属性”来调用。不过，一般都是用"类名.类属性”。<br>
static变量置于方法区中。在静态的方法里面不可以调用非静态的方法或变量；但是在非静态的方法里可以调用静态的方法或变量。

#### 静态成员方法
用static声明的方法为静态方法。

- 不需要对象，就可以调用（类名.方法名）
- 在调用该方法时，不会将对象的引用传递给它，所以在static方法中不可访问非static的成员。

### 2. 静态代码块
静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。 该类不管创建多少对象，静态代码块只执行一次.

静态代码块的格式是
```java
static {    
语句体;   
}
```
一个类中的静态代码块可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果静态代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95/%E9%9D%99%E6%80%81%E4%BB%A3%E7%A0%81%E5%9D%97.jpg)
静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问.

### 3. 修饰类(只能修饰内部类)
静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：

它的创建是不需要依赖外围类的创建。
它不能使用任何外围类的非static成员变量和方法。
Example（静态内部类实现单例模式）
```java
public class Singleton {
    
    声明为 private 避免调用默认构造方法创建对象
    private Singleton() {
    }
    
    声明为 private 表明静态内部该类只能在该 Singleton 类中被访问
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
当 Singleton 类加载时，静态内部类 SingletonHolder 没有被加载进内存。只有当调用 getUniqueInstance() 方法从而触发 `SingletonHolder.INSTANCE` 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。

这种方式不仅具有延迟初始化的好处，而且由 JVM 提供了对线程安全的支持。


### 4. 静态导包(用来导入类中的静态资源，1.5之后的新特性)
格式为：import static

这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法
```java
  Math. --- 将Math中的所有静态资源导入，这时候可以直接使用里面的静态方法，而不用通过类名进行调用
  如果只想导入单一某个静态方法，只需要将换成对应的方法名即可
 
import static java.lang.Math.;

  换成import static java.lang.Math.max;具有一样的效果
 
public class Demo {
  public static void main(String[] args) {
 
    int max = max(1,2);
    System.out.println(max);
  }
}
```

### 6. 补充内容
#### 静态方法与非静态方法
静态方法属于类本身，非静态方法属于从该类生成的每个对象。 如果您的方法执行的操作不依赖于其类的各个变量和方法，请将其设置为静态（这将使程序的占用空间更小）。 否则，它应该是非静态的。

Example
```java
class Foo {
    int i;
    public Foo(int i) { 
       this.i = i;
    }

    public static String method1() {
       return An example string that doesn't depend on i (an instance variable);
       
    }

    public int method2() {
       return this.i + 1;  Depends on i
    }

}
```
你可以像这样调用静态方法：`Foo.method1（）`。 如果您尝试使用这种方法调用` method2 `将失败。 但这样可行：`Foo bar = new Foo（1）;bar.method2（）;`

总结：

在外部调用静态方法时，可以使用”类名.方法名”的方式，也可以使用”对象名.方法名”的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。


静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），而不允许访问实例成员变量和实例方法；实例方法则无此限制

#### static{}静态代码块与{}非静态代码块(构造代码块)
相同点： 都是在JVM加载类时且在构造方法执行之前执行，在类中都可以定义多个，定义多个时按定义的顺序执行，一般在代码块中对一些static变量进行赋值。

不同点： 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。静态代码块只在第一次new执行一次，之后不再执行，而非静态代码块在每new一次就执行一次。 非静态代码块可在普通方法中定义(不过作用不大)；而静态代码块不行。

一般情况下,如果有些代码比如一些项目最常用的变量或对象必须在项目启动的时候就执行,需要使用静态代码块,这种代码是主动执行的。如果我们想要设计不需要创建对象就可以调用类中的方法，例如：Arrays类，Character类，String类等，就需要使用静态方法, 两者的区别是 静态代码块是自动执行的而静态方法是被调用的时候才执行的.

Example
```java
public class Test {
    public Test() {
        System.out.print(默认构造方法！--);
    }

     非静态代码块
    {
        System.out.print(非静态代码块！--);
    }
     静态代码块
    static {
        System.out.print(静态代码块！--);
    }

    public static void test() {
        System.out.print(静态方法中的内容! --);
        {
            System.out.print(静态方法中的代码块！--);
        }

    }
    public static void main(String[] args) {

        Test test = new Test();   
        Test.test();静态代码块！--静态方法中的内容! --静态方法中的代码块！--
    }
```
当执行` Test.test();` 时输出：
```
静态代码块！--静态方法中的内容! --静态方法中的代码块！--
```
当执行 `Test test = new Test(); `时输出：
```
静态代码块！--非静态代码块！--默认构造方法！--
```
非静态代码块与构造函数的区别是：

非静态代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，构造代码块中定义的是不同对象共性的初始化内容。

## 五、abstract
- 如果修饰类的话，此类必须被继承使用，abstrcat最大程度的将子类的共性抽出来。abstract虽然不能生成对象，但可以声明，作为编译时类型，final与abstract不能同时出现。
- 一个有抽象方法的类，一定是抽象类，但一个抽象类的不一定有抽象方法。
- 抽象类不可创建对象，但可以有构造函数（子类可以继承）。
- 子类继承抽象类，必须实现其中的抽象方法，除非子类也是抽象类。
- abstract不可与private、static、final、native、synchronized


## 六、instanceof
instanceof 是 Java 的一个二元操作符，类似于 ==，>，< 等操作符。

instanceof 是 Java 的保留关键字。它的作用是测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型。


```java
import java.util.ArrayList;
import java.util.Vector;
 
public class Main {
 
public static void main(String[] args) {
   Object testObject = new ArrayList();
      displayObjectClass(testObject);
   }
   public static void displayObjectClass(Object o) {
      if (o instanceof Vector)
      System.out.println("对象是 java.util.Vector 类的实例");
      else if (o instanceof ArrayList)
      System.out.println("对象是 java.util.ArrayList 类的实例");
      else
      System.out.println("对象是 " + o.getClass() + " 类的实例");
   }
}
```

以上代码运行输出结果为：
```java
对象是 java.util.ArrayList 类的实例
```

## 七、transient
### 1. 什么是序列化
在讨论transient之前，有必要先搞清楚Java中序列化的含义；

Java中对象的序列化指的是将对象转换成以字节序列的形式来表示，这些字节序列包含了对象的数据和信息，一个序列化后的对象可以被写到数据库或文件中，也可用于网络传输，一般当我们使用缓存cache（内存空间不够有可能会本地存储到硬盘）或远程调用rpc（网络传输）的时候，经常需要让我们的实体类实现Serializable接口，目的就是为了让其可序列化。

当然，序列化后的最终目的是为了反序列化，恢复成原先的Java对象，要不然序列化后干嘛呢，所以序列化后的字节序列都是可以恢复成Java对象的，这个过程就是反序列化。

### 2. transient关键字
Java中transient关键字的作用，简单地说，就是让某些被修饰的成员属性变量不被序列化，这一看好像很好理解，就是不被序列化，那么什么情况下，一个对象的某些字段不需要被序列化呢？如果有如下情况，可以考虑使用关键字transient修饰：

1. 类中的字段值可以根据其它字段推导出来，如一个长方形类有三个属性：长度、宽度、面积（示例而已，一般不会这样设计），那么在序列化的时候，面积这个属性就没必要被序列化了；

2. 其它，看具体业务需求吧，哪些字段不想被序列化；

PS: 记得之前看HashMap源码的时候，发现有个字段是用transient修饰的，我觉得还是有道理的，确实没必要对这个modCount字段进行序列化，因为没有意义，modCount主要用于判断HashMap是否被修改（像put、remove操作的时候，modCount都会自增），对于这种变量，一开始可以为任何值，0当然也是可以（new出来、反序列化出来、或者克隆clone出来的时候都是为0的），没必要持久化其值。

最后，为什么要不被序列化呢，主要是为了节省存储空间，其它的感觉没啥好处，可能还有坏处（有些字段可能需要重新计算，初始化什么的），总的来说，利大于弊。

## 八、native
初次遇见 native是在 java.lang.Object 源码中的一个hashCode方法：
```java
public native int hashCode();
```
为什么有个native呢？这是我所要学习的地方。所以下面想要总结下native。
### 1. 认识 native 即 JNI,Java Native Interface
凡是一种语言，都希望是纯。比如解决某一个方案都喜欢就单单这个语言来写即可。Java平台有个用户和本地C代码进行互操作的API，称为Java Native Interface (Java本地接口)。
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95/%E8%AE%A4%E8%AF%86native.png)

### 2. 用 Java 调用 C 的“Hello，JNI”
我们需要按照下班方便的步骤进行：

1、创建一个Java类，里面包含着一个 native 的方法和加载库的方法 loadLibrary。HelloNative.java 代码如下：
```java
public class HelloNative
{
    static
    {
        System.loadLibrary("HelloNative");
    }
     
    public static native void sayHello();
     
    @SuppressWarnings("static-access")
    public static void main(String[] args)
    {
        new HelloNative().sayHello();
    }
}
```
首先让大家注意的是native方法，那个加载库的到后面也起作用。native 关键字告诉编译器（其实是JVM）调用的是该方法在外部定义，这里指的是C。如果大家直接运行这个代码，  JVM会告之：“A Java Exception has occurred.”控制台输出如下：
```
Exception in thread "main" java.lang.UnsatisfiedLinkError: no HelloNative in java.library.path
    at java.lang.ClassLoader.loadLibrary(Unknown Source)
    at java.lang.Runtime.loadLibrary0(Unknown Source)
    at java.lang.System.loadLibrary(Unknown Source)
    at HelloNative.<clinit>(HelloNative.java:5)
```
这是程序使用它的时候，虚拟机说不知道如何找到sayHello。

2、运行javah，得到包含该方法的C声明头文件.h

将HelloNative.java ，简单地 javac javah，如图
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95/javah%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

就得到了下面的 HelloNative.h文件 ：

```c++
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class HelloNative */
 
#ifndef _Included_HelloNative
#define _Included_HelloNative
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     HelloNative
 * Method:    sayHello
 * Signature: ()V
 */
JNIEXPORT void JNICALL Java_HelloNative_sayHello
  (JNIEnv *, jclass);
 
#ifdef __cplusplus
}
#endif
#endif
```
jni.h 这个文件，在/%JAVA_HOME%include

3、根据头文件，写C实现本地方法。

这里我们简单地实现这个sayHello方法如下：

```c++
#include "HelloNative.h"
#include <stdio.h>
 
JNIEXPORT void JNICALL Java_HelloNative_sayHello
{
    printf("Hello，JNI");    
}
```
4、生成dll共享库，然后Java程序load库，调用即可。

在Windows上，MinGW GCC 运行如下

```
gcc -m64  -Wl,--add-stdcall-alias -I"C:\Program Files\Java\jdk1.7.0_71\include" -I"C:\Program Files\Java\jdk1.7.0_71\include\include\win32" -shared -o HelloNative.dll HelloNative.c
```
-m64表示生成dll库是64位的。然后运行 HelloNative：

```
java HelloNative
```

终于成功地可以看到控制台打印如下：

```
Hello，JNI
```

### 3. JNI 调用 C 流程图
![image](https://github.com/xinyuan960205/pic_resource/raw/master/java%E5%AD%A6%E4%B9%A0/%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95/JNI%E8%B0%83%E7%94%A8C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

### 4. 其他介绍
native是与C++联合开发的时候用的！java自己开发不用的！

---
使用native关键字说明这个方法是原生函数，也就是这个方法是用C/C++语言实现的，并且被编译成了DLL，由java去调用。
这些函数的实现体在DLL中，JDK的源代码中并不包含，你应该是看不到的。对于不同的平台它们也是不同的。这也是java的底层机制，实际上java就是在不同的平台上调用不同的native方法实现对操作系统的访问的。

---

1。native 是用做java 和其他语言（如c++）进行协作时用的
也就是native 后的函数的实现不是用java写的
2。既然都不是java，那就别管它的源代码了，呵呵

---

native的意思就是通知操作系统，
这个函数你必须给我实现，因为我要使用。
所以native关键字的函数都是操作系统实现的，
java只能调用。

---

java是跨平台的语言，既然是跨了平台，所付出的代价就是牺牲一些对底层的控制，而java要实现对底层的控制，就需要一些其他语言的帮助，这个就是native的作用了

Java不是完美的，Java的不足除了体现在运行速度上要比传统的C++慢许多之外，Java无法直接访问到操作系统底层（如系统硬件等)，为此Java使用native方法来扩展Java程序的功能。
　　可以将native方法比作Java程序同Ｃ程序的接口，其实现步骤：
　　１、在Java中声明native()方法，然后编译；
　　２、用javah产生一个.h文件；
　　３、写一个.cpp文件实现native导出方法，其中需要包含第二步产生的.h文件（注意其中又包含了JDK带的jni.h文件）；
　　４、将第三步的.cpp文件编译成动态链接库文件；
　　５、在Java中用System.loadLibrary()方法加载第四步产生的动态链接库文件，这个native()方法就可以在Java中被访问了。

---

JAVA本地方法适用的情况 
1.为了使用底层的主机平台的某个特性，而这个特性不能通过JAVA API访问

2.为了访问一个老的系统或者使用一个已有的库，而这个系统或这个库不是用JAVA编写的

3.为了加快程序的性能，而将一段时间敏感的代码作为本地方法实现。

## 九、synchronized



## 十、volatile