# IOC

## 一、程序的耦合和解耦

### 什么是程序的耦合

​		**耦合性**(Coupling)，也叫耦合度，是对模块间关联程度的度量。耦合的强弱取决于模块间接口的复杂性、调
用模块的方式以及通过界面传送数据的多少。模块间的耦合度是指模块之间的依赖关系，包括控制关系、调用关
系、数据传递关系。模块间联系越多，其耦合性越强，同时表明其独立性越差( 降低耦合性，可以提高其独立
性)。 耦合性存在于各个领域，而非软件设计中独有的，但是我们只讨论软件工程中的耦合。
​		在软件工程中， 耦合指的就是就是对象之间的依赖性。对象之间的耦合越高，维护成本越高。因此对象的设计应使类和构件之间的耦合最小。 软件设计中通常用耦合度和内聚度作为衡量模块独立程度的标准。 ==划分模块的一个准则就是高内聚低耦合。==
**它有如下分类**：

1. 内容耦合。当一个模块直接修改或操作另一个模块的数据时，或一个模块不通过正常入口而转入另一个模块时，这样的耦合被称为内容耦合。内容耦合是最高程度的耦合，应该避免使用之。
2. 公共耦合。两个或两个以上的模块共同引用一个全局数据项，这种耦合被称为公共耦合。在具有大量公共耦合的结构中，确定究竟是哪个模块给全局变量赋了一个特定的值是十分困难的。
3. 外部耦合 。一组模块都访问同一全局简单变量而不是同一全局数据结构，而且不是通过参数表传递该全局变量的信息，则称之为外部耦合。
4. 控制耦合 。一个模块通过接口向另一个模块传递一个控制信号，接受信号的模块根据信号值而进行适当的动作，这种耦合被称为控制耦合。
5. 标记耦合 。若一个模块 A 通过接口向两个模块 B 和 C 传递一个公共参数，那么称模块 B 和 C 之间存在一个标记耦合。
6. 数据耦合。模块之间通过参数来传递数据，那么被称为数据耦合。数据耦合是最低的一种耦合形式，系统中一般都存在这种类型的耦合，因为为了完成一些有意义的功能，往往需要将某些模块的输出数据作为另一些模块的输入数据。
7. 非直接耦合 。两个模块之间没有直接关系，它们之间的联系完全是通过主模块的控制和调用来实现的。

**总结**：

​		耦合是影响软件复杂程度和设计质量的一个重要因素，在设计上我们应采用以下原则：如果模块间必须
存在耦合，就尽量使用数据耦合，少用控制耦合，限制公共耦合的范围，尽量避免使用内容耦合。

**内聚与耦合**

​		内聚标志一个模块内各个元素彼此结合的紧密程度，它是信息隐蔽和局部化概念的自然扩展。 内聚是从功能角度来度量模块内的联系，一个好的内聚模块应当恰好做一件事。它描述的是模块内的功能联系。耦合是软件结构中各模块之间相互连接的一种度量，耦合强弱取决于模块间接口的复杂程度、进入或访问一个模块的点以及通过接口的数据。 程序讲究的是低耦合，高内聚。就是同一个模块内的各个元素之间要高度紧密，但是各个模块之间的相互依存度却要不那么紧密。
​		内聚和耦合是密切相关的，同其他模块存在高耦合的模块意味着低内聚，而高内聚的模块意味着该模块同其他模块之间是低耦合。在进行软件设计时，应力争做到高内聚，低耦合。  

==我们在开发中，有些依赖关系是必须的，有些依赖关系可以通过优化代码来解除的。==
请看下面的示例代码：

```java
/**
\* 账户的业务层实现类
\* @Version 1.0
*/
public class AccountServiceImpl implements IAccountService {
	private IAccountDao accountDao = new AccountDaoImpl();
}
```

上面的代码表示：
业务层调用持久层，并且此时业务层在依赖持久层的接口和实现类。如果此时没有持久层实现类，编译将不能通过。 这种编译期依赖关系，应该在我们开发中杜绝。 我们需要优化代码解决。  

**再比如：**
早期我们的 JDBC 操作，注册驱动时，我们为什么不使用 DriverManager 的 register 方法，而是采用 Class.forName 的方式？  

```java
public class JdbcDemo1 {
    /**
    * @Version 1.0
    * @param args
    * @throws Exception
    */
	public static void main(String[] args) throws Exception {
    //1.注册驱动
    //DriverManager.registerDriver(new com.mysql.jdbc.Driver());
    Class.forName("com.mysql.jdbc.Driver");
        //2.获取连接
        //3.获取预处理 sql 语句对象
        //4.获取结果集
        //5.遍历结果集
    }
}
```

原因就是：
我们的类依赖了数据库的具体驱动类（MySQL） ，如果这时候更换了数据库品牌（比如 Oracle） ，需要修改源码来重新数据库驱动。这显然不是我们想要的。  

### 解决程序耦合的思路

当是我们讲解 jdbc 时，是通过反射来注册驱动的，代码如下：

```java
Class.forName("com.mysql.jdbc.Driver");//此处只是一个字符串  
```

​		此时的好处是，我们的类中不再依赖具体的驱动类，此时就算删除 mysql 的驱动 jar 包，依然可以编译（运
行就不要想了，没有驱动不可能运行成功的） 。
​		同时，也产生了一个新的问题， mysql 驱动的全限定类名字符串是在 java 类中写死的，一旦要改还是要修改
源码。
​		解决这个问题也很简单，使用配置文件配置。  

### 工厂模式解耦

​		在实际开发中我们可以把三层的对象都使用配置文件配置起来，当启动服务器应用加载的时候， 让一个类中的方法通过读取配置文件，把这些对象创建出来并存起来。在接下来的使用的时候，直接拿过来用就好了。那么，这个读取配置文件， 创建和获取三层对象的类就是工厂。

## 二、控制反转-Inversion Of Control

### 解耦的思路有 2 个问题

1. 存哪去？
   		分析：由于我们是很多对象，肯定要找个集合来存。这时候有 Map 和 List 供选择。到底选 Map 还是 List 就看我们有没有查找需求。有查找需求，选 Map。所以我们的答案就是，在应用加载时，创建一个 Map，用于存放三层对象。我们把这个 map 称之为容器。

2. 还是没解释什么是工厂？

   工厂就是负责给我们从容器中获取指定对象的类。这时候我们获取对象的方式发生了改变。

   原来：

   ​		我们在获取对象时，都是采用 new 的方式。 是**主动的**。  

   现在：
   		我们获取对象时，同时跟工厂要，有工厂为我们查找或者创建对象。 是**被动的**。  

这种被动接收的方式获取对象的思想就是控制反转，它是 spring 框架的核心之一。  

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/DI.jpg)

**明确 ioc 的作用：**削减计算机程序的耦合(解除我们代码中的依赖关系)。  

### IOC概念

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。** IoC 在其他语言中也有应用，并非 Spirng 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

<img src="https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/IOC%E5%9B%BE%E4%BE%8B%E8%AF%B4%E6%98%8E.PNG" style="zoom: 67%;" />

## 三、Spring容器

### 概念

在Spring IOC容器读取Bean配置创建Bean实例之前，必须对它进行实例化，只有在容器实例化后，才可以从IOC容器里获取Bean实例并使用。

```java 
//1.使用 ApplicationContext 接口，就是在获取 spring 容器
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
```



### 容器实现

#### spring 中工厂的类结构图  

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/Spring%E4%B8%AD%E5%B7%A5%E5%8E%82%E7%B1%BB%E7%9A%84%E7%BB%93%E6%9E%84%E5%9B%BE.PNG)



![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/Spring%E4%B8%AD%E5%B7%A5%E5%8E%82%E7%B1%BB%E7%9A%84%E7%BB%93%E6%9E%84%E5%9B%BE1.PNG)

#### BeanFactory 和 ApplicationContext 

- BeanFactory 才是 Spring 容器中的顶层接口。

  BeanFactory是**Spring的心脏**，以Factory结尾，表示它是一个工厂类接口，它是负责生产和管理bean的一个工厂。在Spring中，BeanFactory是IOC容器的核心接口依赖。**所有的Bean都是由BeanFactory（也就是IOC容器）来进行管理的**。它的职责包括：**实例化、配置和管理Bean**及**建立这些对象间的依赖**。

- ApplicationContext 是它的子接口。

  如果说BeanFactory是Spring的心脏，那么ApplicationContext就是**完整的躯体**了，ApplicationContext由BeanFactory派生而来，是IOC容器另一个重要的接口，它继承了BeanFactory的基本功能，提供了更多面向实际应用的功能。在BeanFactory中，很多功能需要以**编程的方式**实现，而**在ApplicationContext中则可以通过配置实现**。

- BeanFactory 和 ApplicationContext 的区别：

  - 创建对象的时间点不一样。

  - ApplicationContext：只要一读取配置文件，默认情况下就会创建对象。

  - BeanFactory：什么使用什么时候创建对象。  

#### ApplicationContext 接口的实现类

1. ClassPathXmlApplicationContext：
   它是从类的根路径下加载配置文件 推荐使用这种
2. FileSystemXmlApplicationContext：
   它是从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。
3. AnnotationConfigApplicationContext:
   当我们使用注解配置容器对象时，需要使用此类来创建 spring 容器。它用来读取注解。  

## 四、DI(Dependency Injection)依赖注入

### 概念

依赖注入： Dependency Injection。 它是 spring 框架核心 ioc 的具体实现。

我们的程序在编写时， 通过控制反转， 把对象的创建交给了 spring，但是代码中不可能出现没有依赖的情况。ioc 解耦只是降低他们的依赖关系，但不会消除。 例如：我们的业务层仍会调用持久层的方法。

那这种业务层和持久层的依赖关系， 在使用 spring 之后， 就让 spring 来维护了。

简单的说，就是坐等框架把持久层对象传入业务层，而不用我们自己去获取。  

### 构造器注入

顾名思义，就是使用类中的构造函数，给成员变量赋值。注意，赋值的操作不是我们自己做的，而是通过配置
的方式，让 spring 框架来为我们注入。

具体代码如下：  

service代码

```java
public class AccountServiceImpl implements IAccountService {
    private String name;
    private Integer age;
    private Date birthday;
    
    public AccountServiceImpl(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }
    
    @Override
    public void saveAccount() {
    	System.out.println(name+","+age+","+birthday);
    }
}
```

使用构造函数的方式，给 service 中的属性传值

要求：

​	类中需要提供一个对应参数列表的构造函数。

涉及的标签`<constructor-arg>`：

- **index**:指定参数在构造函数参数列表的索引位置

- **type**:指定参数在构造函数中的数据类型  

- **name**:指定参数在构造函数中的名称 用这个找给谁赋值

  上面三个都是找给谁赋值，下面两个指的是赋什么值的

- **value**:它能赋的值是基本数据类型和 String 类型

- **ref**:它能赋的值是其他 bean 类型，也就是说，必须得是在配置文件中配置过的 bean  

bean中

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
    <constructor-arg name="name" value="张三"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
<bean id="now" class="java.util.Date"></bean>
```

### set方法注入

#### 通常方法

就是在类中提供需要注入成员的 set 方法。  

```java
public class AccountServiceImpl implements IAccountService {
    private String name;
    private Integer age;
    private Date birthday;
    
    public void setName(String name) {
    	this.name = name;
    }
    
    public void setAge(Integer age) {
    	this.age = age;
    }
    
    public void setBirthday(Date birthday) {
    	this.birthday = birthday;
    }
    
    @Override
    public void saveAccount() {
    	System.out.println(name+","+age+","+birthday);
    }
}
```

通过配置文件给 bean 中的属性传值：使用 set 方法的方式
涉及的标签`<property>`

属性：

- name：找的是类中 set 方法后面的部分
- ref：给属性赋值是其他 bean 类型的
- value：给属性赋值是基本数据类型和 string 类型的

bean.xml

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
    <property name="name" value="test"></property>
    <property name="age" value="21"></property>
    <property name="birthday" ref="now"></property>
</bean>
<bean id="now" class="java.util.Date"></bean>
```

实际开发中，此种方式用的较多。

#### 使用 p 名称空间注入数据（本质还是调用 set 方法）  

此种方式是通过在 xml 中导入 p 名称空间，使用 p:propertyName 来注入数据，它的本质仍然是调用类中的
set 方法实现注入功能。  

service代码和set注入相同。

bean.xml

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:p="http://www.springframework.org/schema/p"
       	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation=" http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
    <bean id="accountService"
        class="com.itheima.service.impl.AccountServiceImpl4"
        p:name="test" p:age="21" p:birthday-ref="now"/>
</beans>
```

#### 注入集合属性  

 就是给类中的集合成员传值，它用的也是 set方法注入的方式，只不过变量的数据类型都是集合。
我们这里介绍注入数组， List,Set,Map,Properties。  

```java
public class AccountServiceImpl implements IAccountService {
    private String[] myStrs;
    private List<String> myList;
    private Set<String> mySet;
    private Map<String,String> myMap;
    private Properties myProps;
    
    public void setMyStrs(String[] myStrs) {
    	this.myStrs = myStrs;
    }
    
    public void setMyList(List<String> myList) {
   	 	this.myList = myList;
    }
    
    public void setMySet(Set<String> mySet) {
    	this.mySet = mySet;
    }
    
    public void setMyMap(Map<String, String> myMap) {
    	this.myMap = myMap;
    }
    
    public void setMyProps(Properties myProps) {
    	this.myProps = myProps;
    }
    
    @Override
    public void saveAccount() {
        System.out.println(Arrays.toString(myStrs));
        System.out.println(myList);
        System.out.println(mySet);
        System.out.println(myMap);
        System.out.println(myProps);
    }
}
```

bean.xml

```xml
<!-- 注入集合数据
    List 结构的：
    array,list,set
    Map 结构的
    map,entry,props,prop
-->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
    <!-- 在注入集合数据时，只要结构相同，标签可以互换 -->
    <!-- 给数组注入数据 -->
    <property name="myStrs">
        <set>
            <value>AAA</value>
            <value>BBB</value>
            <value>CCC</value>
        </set>
    </property>
    <!-- 注入 list 集合数据 -->
    <property name="myList">
        <array>
            <value>AAA</value>
            <value>BBB</value>
            <value>CCC</value>
        </array>
    </property>
    <!-- 注入 set 集合数据 -->
    <property name="mySet">
        <list>
            <value>AAA</value>
            <value>BBB</value>
            <value>CCC</value>
        </list>
    </property>
    <!-- 注入 Map 数据 -->
    <property name="myMap">
        <props>
            <prop key="testA">aaa</prop>
            <prop key="testB">bbb</prop>
        </props>
    </property>
    <!-- 注入 properties 数据 -->
    <property name="myProps">
        <map>
            <entry key="testA" value="aaa"></entry>
            <entry key="testB">
            <value>bbb</value>
            </entry>
        </map>
    </property>
</bean>
```



## 五、bean

### bean标签

1. 作用：
   - 用于配置对象让 spring 来创建的。
   - 默认情况下它调用的是类中的无参构造函数。如果没有无参构造函数则不能创建成功。

2. 属性：

- id： 给对象在容器中提供一个唯一标识。用于获取对象。
- class： 指定类的全限定类名。用于反射创建对象。默认情况下调用无参构造函数。
- scope： 指定对象的作用域。
  \* singleton :默认值，单例的.
  \* prototype :多例的.
  \* request :WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到 request 域中.
  \* session :WEB 项目中,Spring 创建一个 Bean 的对象,将对象存入到 session 域中.
  \* global session :WEB 项目中,应用在 Portlet 环境.如果没有 Portlet 环境那么
- globalSession 相当于 session.
- init-method： 指定类中的初始化方法名称。
- destroy-method： 指定类中销毁方法名称。

### bean的线程安全

**Spring对每个bean提供了一个scope属性来表示该bean的作用域**。它是bean的生命周期。例如，一个scope为singleton的bean，在第一次被注入时，会创建为一个单例对象，该对象会一直被复用到应用结束。

- singleton：默认的scope，每个scope为singleton的bean都会被定义为一个单例对象，该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）。

- prototype：bean被定义为在每次注入时都会创建一个新的对象。

- request：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象。

- session：bean被定义为在一个session的生命周期内创建一个单例对象。

- application：bean被定义为在ServletContext的生命周期中复用一个单例对象。

- websocket：bean被定义为在websocket的生命周期中复用一个单例对象。

我们交由Spring管理的大多数对象其实都是一些无状态的对象，这种不会因为多线程而导致状态被破坏的对象很适合Spring的默认scope，**每个单例的无状态对象都是线程安全的（也可以说只要是无状态的对象，不管单例多例都是线程安全的，不过单例毕竟节省了不断创建对象与GC的开销）。**

> 有状态就是有数据存储功能
> 无状态就是不会保存数据

**无状态的对象即是自身没有状态的对象，自然也就不会因为多个线程的交替调度而破坏自身状态导致线程安全问题**。无状态对象包括我们经常使用的DO、DTO、VO这些只作为数据的实体模型的贫血对象，还有Service、DAO和Controller，这些对象并没有自己的状态，它们只是用来执行某些操作的。例如，每个DAO提供的函数都只是对数据库的CRUD，而且每个数据库Connection都作为函数的局部变量（局部变量是在用户栈中的，而且用户栈本身就是线程私有的内存区域，所以不存在线程安全问题），用完即关（或交还给连接池）。

有人可能会认为，我使用request作用域不就可以避免每个请求之间的安全问题了吗？这是完全错误的，因为Controller默认是单例的，一个HTTP请求是会被多个线程执行的，这就又回到了线程的安全问题。当然，你也可以把Controller的scope改成prototype，实际上Struts2就是这么做的，但有一点要注意，Spring MVC对请求的拦截粒度是基于每个方法的，而Struts2是基于每个类的，所以把Controller设为多例将会频繁的创建与回收对象，严重影响到了性能。

通过阅读上文其实已经说的很清楚了，**Spring根本就没有对bean的多线程安全问题做出任何保证与措施**。对于每个bean的线程安全问题，根本原因是每个bean自身的设计。**不要在bean中声明任何有状态的实例变量或类变量，如果必须如此，那么就使用ThreadLocal把变量变为线程私有的，如果bean的实例变量或类变量需要在多个线程之间共享，那么就只能使用synchronized、lock、CAS等这些实现线程同步的方法了。**

具体可以看这个https://www.cnblogs.com/myseries/p/11729800.html

结论需要注意：

　　1、在@Controller/@Service等容器中，默认情况下，scope值是单例-singleton的，也是线程不安全的。
　　2、尽量不要在@Controller/@Service等容器中定义静态变量，不论是单例(singleton)还是多实例(prototype)他都是线程不安全的。
　　3、默认注入的Bean对象，在不设置scope的时候他也是线程不安全的。
　　4、一定要定义变量的话，用ThreadLocal来封装，这个是线程安全的

### bean生命周期

1. 单例对象： scope="singleton"

一个应用只有一个对象的实例。它的作用范围就是整个引用。

- 生命周期：

对象出生：当应用加载，创建容器时，对象就被创建了。

对象活着：只要容器在，对象一直活着。

对象死亡：当应用卸载，销毁容器时，对象就被销毁了。

2. 多例对象： scope="prototype"

每次访问对象时，都会重新创建对象实例。

- 生命周期：

对象出生：当使用对象时，创建新的对象实例。

对象活着：只要对象在使用中，就一直活着。

对象死亡：当对象长时间不用时，被 java 的垃圾回收器回收了。  

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/springbean%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.PNG)

1.Spring对Bean进行实例化（相当于程序中的new Xx()）

2.Spring将值和Bean的引用注入进Bean对应的属性中

3.如果Bean实现了BeanNameAware接口，Spring将Bean的ID传递给setBeanName()方法
**（实现BeanNameAware清主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有用到Bean的ID的**）

4.如果Bean实现了BeanFactoryAware接口，Spring将调用setBeanDactory(BeanFactory bf)方法并把BeanFactory容器实例作为参数传入。
**（实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等）**

5.如果Bean实现了ApplicationContextAwaer接口，Spring容器将调用setApplicationContext(ApplicationContext ctx)方法，把y应用上下文作为参数传入.
**(作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory )**

6.如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessBeforeInitialization（预初始化）方法
**（作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能）**

7.如果Bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。

8.如果Bean实现了BeanPostProcess接口，Spring将调用它们的postProcessAfterInitialization（后初始化）方法
**（作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同 )**

9.经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁

10.如果Bean实现了DispostbleBean接口，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。

### 装配

#### 基于xml（手动装配）

##### 使用默认无参构造函数  

它会根据默认无参构造函数来创建类对象。如果 bean 中没有默认无参构造函数，将会创建失败。  

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"/>
```



##### 通过工厂方法

1. 静态工厂

工厂：

```java
public class StaticFactory {
    public static IAccountService createAccountService(){
    	return new AccountServiceImpl();
    }
}
```

配置：

```xml
<!-- 此种方式是:
        使用 StaticFactory 类中的静态方法 createAccountService 创建对象，并存入 spring 容器
        id 属性：指定 bean 的 id，用于从容器中获取
        class 属性：指定静态工厂的全限定类名
        factory-method 属性：指定生产对象的静态方法
-->
<bean id="accountService"
    class="com.itheima.factory.StaticFactory"
    factory-method="createAccountService"></bean>
```

2. 实例工厂

工厂：

```java
/**
* 模拟一个实例工厂，创建业务层实现类
* 此工厂创建对象，必须现有工厂实例对象，再调用方法
*/
public class InstanceFactory {
    public IAccountService createAccountService(){
    	return new AccountServiceImpl();
    }
}
```

配置：

```xml
<!-- 此种方式是：
    先把工厂的创建交给 spring 来管理。
    然后在使用工厂的 bean 来调用里面的方法
    factory-bean 属性：用于指定实例工厂 bean 的 id。
    factory-method 属性：用于指定实例工厂中创建对象的方法。
-->
<bean id="instancFactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService"
        factory-bean="instancFactory"
        factory-method="createAccountService"></bean>
```





#### 基于注解（自动装配）

##### 自动扫描

xml配置文件

```xml
<context:component-scan base-package="想要扫描的包"/>  
```

注解

```java
@ComponentScan("想要扫描的包")
```

##### 装配bean

1. @component

把普通pojo实例化到spring容器中，相当于配置文件中的`<bean id="" class=""/>`

2. @value

装配bean属性

代码示例：

```java
@component
public class user{
	@value("74110")
	private int id;
	@value("张三")
	private String userName;
	get set...
}
```

## 六、IOC注解配置

### 6.1 用于创建对象的

相当于： <bean id="" class="">

#### @component

- 作用：
  把资源让 spring 来管理。相当于在 xml 中配置一个 bean。
- 属性：
  value：指定 bean 的 id。如果不指定 value 属性，默认 bean 的 id 是当前类的类名。首字母小写。  



#### @Controller @Service @Repository

他们三个注解都是针对一个的衍生注解，他们的作用及属性都是一模一样的。
他们只不过是提供了更加明确的语义化。
@Controller： 一般用于表现层的注解。
@Service： 一般用于业务层的注解。
@Repository： 一般用于持久层的注解。
细节：如果注解中有且只有一个属性要赋值时，且名称是 value， value 在赋值是可以不写。  



### 6.2 用于注入数据

相当于： 

<property name="" ref="">
<property name="" value="">  

#### @Autowired

- 作用：
  自动按照类型注入。当使用注解注入属性时， set 方法可以省略。它只能注入其他 bean 类型。当有多个类型匹配时，使用要注入的对象变量名称作为 bean 的 id，在 spring 容器查找，找到了也可以注入成功。找不到就报错。

#### @Resource

- 作用：
  直接按照 Bean 的 id 注入。它也只能注入其他 bean 类型。
- 属性：
  name：指定 bean 的 id。  

#### @Value

- 作用：
  注入基本数据类型和 String 类型数据的
- 属性：
  value：用于指定值  

### 6.3 用于改变作用范围

相当于： <bean id="" class="" scope="">  

#### @Scope

- 作用：
  指定 bean 的作用范围。
- 属性：
  value：指定范围的值。
              取值： singleton prototype request session globalsession  



### 6.4 和生命周期相关的

相当于： <bean id="" class="" init-method="" destroy-method="" />  

#### @PostConstruct

- 作用：
  用于指定初始化方法。  

#### @PreDestroy

- 作用：
  用于指定销毁方法。  

### 6.5 纯注解配置

我们的目标就是使用纯注解方式，取消xml的配置方式。

我们发现，之所以我们现在离不开 xml 配置文件，是因为我们有一句很关键的配置：
<!-- 告知spring框架在，读取配置文件，创建容器时，扫描注解，依据注解创建对象，并存入容器中 -->
`<context:component-scan base-package="com.itheima"></context:component-scan>`
如果他要也能用注解配置，那么我们就离脱离 xml 文件又进了一步。  

另外，数据源和 JdbcTemplate 的配置也需要靠注解来实现。  

```xml
<!-- 配置 dbAssit -->
<bean id="dbAssit" class="com.itheima.dbassit.DBAssit">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!-- 配置数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql:///spring_day02"></property>
    <property name="user" value="root"></property>
    <property name="password" value="1234"></property>
</bean>
```

#### @Configuration

- 作用：
  用于指定当前类是一个 spring 配置类， 当创建容器时会从该类上加载注解。 获取容器时需要使用
  AnnotationApplicationContext(有@Configuration 注解的类.class)。
- 属性：
  value:用于指定配置类的字节码  

示例代码：

```java
/**
* spring 的配置类，相当于 bean.xml 文件
* @author 黑马程序员
* @Company http://www.ithiema.com
* @Version 1.0
*/
@Configuration
public class SpringConfiguration {
}
```

- 注意：
  我们已经把配置文件用类来代替了， 但是如何配置创建容器时要扫描的包呢？
  请看下一个注解。  

#### @ComponentScan

- 作用：
  用于指定 spring 在初始化容器时要扫描的包。 作用和在 spring 的 xml 配置文件中的：
  `<context:component-scan base-package="com.itheima"/>`是一样的。
- 属性：
  basePackages：用于指定要扫描的包。和该注解中的 value 属性作用一样。
- 示例代码：  

```java
/**
* spring 的配置类，相当于 bean.xml 文件
* @author 黑马程序员
* @Company http://www.ithiema.com
* @Version 1.0
*/
@Configuration
@ComponentScan("com.itheima")
public class SpringConfiguration {
}
```

- 注意：
  我们已经配置好了要扫描的包，但是数据源和 JdbcTemplate 对象如何从配置文件中移除呢？
  请看下一个注解。  

#### @Bean

- 作用：
  该注解只能写在方法上，表明使用此方法创建一个对象，并且放入 spring 容器。
- 属性：
  name：给当前@Bean 注解方法创建的对象指定一个名称(即 bean 的 id）。  

示例代码：

```java
/**
* 连接数据库的配置类
* @author 黑马程序员
* @Company http://www.ithiema.com
* @Version 1.0
*/
public class JdbcConfig {
    /**
    * 创建一个数据源，并存入 spring 容器中
    * @return
    */
    @Bean(name="dataSource")
    public DataSource createDataSource() {
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setUser("root");
            ds.setPassword("1234");
            ds.setDriverClass("com.mysql.jdbc.Driver");
            ds.setJdbcUrl("jdbc:mysql:///spring_day02");
            return ds;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    /**
    * 创建一个 DBAssit，并且也存入 spring 容器中
    * @param dataSource
    * @return
    */
    @Bean(name="dbAssit")
    public DBAssit createDBAssit(DataSource dataSource) {
        return new DBAssit(dataSource);
    }
}
```

- 注意:
  我们已经把数据源和 DBAssit 从配置文件中移除了，此时可以删除 bean.xml 了。
  但是由于没有了配置文件，创建数据源的配置又都写死在类中了。如何把它们配置出来呢？
  请看下一个注解。  

#### @PropertySource

- 作用：

  用于加载.properties 文件中的配置。例如我们配置数据源时，可以把连接数据库的信息写到
  properties 配置文件中，就可以使用此注解指定 properties 配置文件的位置。

- 属性：
  value[]：用于指定 properties 文件位置。如果是在类路径下，需要写上 classpath:  

- 示例代码：

```java
配置：
/**
* 连接数据库的配置类
* @author 黑马程序员
* @Company http://www.ithiema.com
* @Version 1.0
*/
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;
    
    /**
    * 创建一个数据源，并存入 spring 容器中
    * @return
    */
    @Bean(name="dataSource")
    public DataSource createDataSource() {
        try {
            ComboPooledDataSource ds = new ComboPooledDataSource();
            ds.setDriverClass(driver);
            ds.setJdbcUrl(url);
            ds.setUser(username);
            ds.setPassword(password);
            return ds;
        } catch (Exception e) {
        	throw new RuntimeException(e);
        }
    }
}
```

jdbc.properties 文件：  

```xml
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/day44_ee247_spring
jdbc.username=root
jdbc.password=1234
```

- 注意：
  此时我们已经有了两个配置类，但是他们还没有关系。如何建立他们的关系呢？
  请看下一个注解。  

#### @Import

- 作用：
  用于导入其他配置类，在引入其他配置类时，可以不用再写@Configuration 注解。 当然，写上也没问题。
- 属性：
  value[]：用于指定其他配置类的字节码。  

- 示例代码 :

```java
@Configuration
@ComponentScan(basePackages = "com.itheima.spring")
@Import({ JdbcConfig.class})
public class SpringConfiguration {
}

@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig{
}
```

- 注意：
  我们已经把要配置的都配置好了，但是新的问题产生了，由于没有配置文件了，如何获取容器呢？
  请看下一小节。  

#### 通过注解获取容器

`ApplicationContext ac =new AnnotationConfigApplicationContext(SpringConfiguration.class);  `







