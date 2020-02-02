

# AOP

## AOP概述

### 什么是AOP

AOP： 全称是 Aspect Oriented Programming 即： 面向切面编程。  

简单的说它就是把我们程序重复的代码抽取出来，在需要执行的时候，使用动态代理的技术，在不修改源码的
基础上，对我们的已有方法进行增强。  

首先，在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

- **所谓的核心业务**，比如登陆，增加数据，删除数据都叫核心业务
- **所谓的周边功能**，比如性能统计，日志，事务管理等等

周边功能在 Spring 的面向切面编程AOP思想里，即被定义为切面

在面向切面编程AOP的思想里面，核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起，这就叫AOP

### AOP 的作用及优势

- 作用：
  在程序运行期间，不修改源码对已有方法进行增强。
- 优势：
  减少重复代码
  提高开发效率
  维护方便  

### AOP 的实现方式

使用动态代理技术  

## AOP具体应用

本例是在方法里添加事务控制，通过动态代理的方式实现AOP

### 案例分析

下面是账户的业务层实现类。实现了账户金额的增删改查

```java
/**
* 账户的业务层实现类
* @Version 1.0
*/
public class AccountServiceImpl implements IAccountService {
    
    private IAccountDao accountDao;
    
    public void setAccountDao(IAccountDao accountDao) {
    	this.accountDao = accountDao;
    }
    
    @Override
    public void saveAccount(Account account) throws SQLException {
    	accountDao.save(account);
    }
    
    @Override
    public void updateAccount(Account account) throws SQLException{
    	accountDao.update(account);
    }
    
    @Override
    public void deleteAccount(Integer accountId) throws SQLException{
    	accountDao.delete(accountId);
    }
    
    @Override
    public Account findAccountById(Integer accountId) throws SQLException {
    	return accountDao.findById(accountId);
    }
    
    @Override
    public List<Account> findAllAccount() throws SQLException{
    	return accountDao.findAll();
    }
}
```

业务层实现转账功能，但是假如发生了异常，用上面的代码是不能实现事务回滚的。

业务层接口  

```java

/**
* 转账
* @param sourceName
* @param targetName
* @param money
*/
void transfer(String sourceName,String targetName,Float money);
```

业务层实现类：

```java
@Override
public void transfer(String sourceName, String targetName, Float money) {
    //根据名称查询两个账户信息
    Account source = accountDao.findByName(sourceName);
    Account target = accountDao.findByName(targetName);
    //转出账户减钱，转入账户加钱
    source.setMoney(source.getMoney()-money);
    target.setMoney(target.getMoney()+money);
    //更新两个账户
    accountDao.update(source);
    int i=1/0; //模拟转账异常
    accountDao.update(target);
}
```

当我们执行时，由于执行有**异常**，转账失败。但是因为我们是每次执行持久层方法都是独立事务，导致无法实
现事务控制（不符合事务的一致性）  

- 解决办法：
  让业务层来控制事务的提交和回滚。

```java
/**
* 账户的业务层实现类
* @Version 1.0
*/
public class AccountServiceImpl implements IAccountService {
    
    private IAccountDao accountDao = new AccountDaoImpl();
    
    @Override
    public void saveAccount(Account account) {
        try {
            TransactionManager.beginTransaction();
            accountDao.save(account);
            TransactionManager.commit();
        } catch (Exception e) {
            TransactionManager.rollback();
        	e.printStackTrace();
        }finally {
        	TransactionManager.release();
        }
    }
    
    @Override
    public void updateAccount(Account account) {
        try {
            TransactionManager.beginTransaction();
            accountDao.update(account);
            TransactionManager.commit();
        } catch (Exception e) {
            TransactionManager.rollback();
            e.printStackTrace();
        }finally {
            TransactionManager.release();
        }
    }
    
    @Override
    public void deleteAccount(Integer accountId) {
        try {
            TransactionManager.beginTransaction();
            accountDao.delete(accountId);
            TransactionManager.commit();
        } catch (Exception e) {
            TransactionManager.rollback();
            e.printStackTrace();
        }finally {
            TransactionManager.release();
        }
    }
    
    @Override
    public Account findAccountById(Integer accountId) {
        Account account = null;
        try {
            TransactionManager.beginTransaction();
            account = accountDao.findById(accountId);
            TransactionManager.commit();
        	return account;
        } catch (Exception e) {
            TransactionManager.rollback();
            e.printStackTrace();
        }finally {
        	TransactionManager.release();
        }
        return null;
    }
    
    @Override
    public List<Account> findAllAccount() {
        List<Account> accounts = null;
        try {
            TransactionManager.beginTransaction();
            accounts = accountDao.findAll();
            TransactionManager.commit();
        	return accounts;
        } catch (Exception e) {
            TransactionManager.rollback();
            e.printStackTrace();
        }finally {
        	TransactionManager.release();
        }
        return null;
    }
    
    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        try {
            TransactionManager.beginTransaction();
            Account source = accountDao.findByName(sourceName);
            Account target = accountDao.findByName(targetName);
            source.setMoney(source.getMoney()-money);
            target.setMoney(target.getMoney()+money);
            accountDao.update(source);
                int i=1/0;
            accountDao.update(target);
            TransactionManager.commit();
        } catch (Exception e) {
            TransactionManager.rollback();
            e.printStackTrace();
        }finally {
        	TransactionManager.release();
        }
    }
}
```

TransactionManager 类的代码：  

```java
/**
* 事务控制类
* @Version 1.0
*/
public class TransactionManager {
    
    //定义一个 DBAssit
    private static DBAssit dbAssit = new DBAssit(C3P0Utils.getDataSource(),true);
    
    //开启事务
    public static void beginTransaction() {
        try {
        	dbAssit.getCurrentConnection().setAutoCommit(false);
        } catch (SQLException e) {
        	e.printStackTrace();
        }
    }
    
    //提交事务
    public static void commit() {
        try {
        	dbAssit.getCurrentConnection().commit();
        } catch (SQLException e) {
        	e.printStackTrace();
        }
    }
    
    //回滚事务
    public static void rollback() {
        try {
        	dbAssit.getCurrentConnection().rollback();
        } catch (SQLException e) {
        	e.printStackTrace();
        }
    }
    
    //释放资源
    public static void release() {
        try {
        	dbAssit.releaseConnection();
        } catch (Exception e) {
        	e.printStackTrace();
        }
    }
}    
```

### 新的问题

​		上一小节的代码，通过对业务层改造，已经可以实现事务控制了，但是由于我们添加了事务控制，也产生了一个新的问题：

- 业务层方法变得臃肿了，里面充斥着很多重复代码。并且业务层方法和事务控制方法耦合了。
- 试想一下，如果我们此时提交，回滚，释放资源中任何一个方法名变更，都需要修改业务层的代码，况且这还
  只是一个业务层实现类，而实际的项目中这种业务层实现类可能有十几个甚至几十个。  

### 动态代理

#### 动态代理的特点

- 字节码随用随创建，随用随加载。
- 它与静态代理的区别也在于此。因为静态代理是字节码一上来就创建好，并完成加载。
- 装饰者模式就是静态代理的一种体现。  

#### 动态代理常用的有两种方式

- 基于接口的动态代理  

  提供者： JDK 官方的 Proxy 类。
  要求：被代理类最少实现一个接口。  

- 基于子类的动态代理
  提供者：第三方的 CGLib，如果报 asmxxxx 异常，需要导入 asm.jar。
  要求：被代理类不能用 final 修饰的类（最终类）。  
- 关于代理的选择
  在 spring 中，框架会根据目标类是否实现了接口来决定采用哪种动态代理的方式。  

#### 使用 JDK 官方的 Proxy 类创建代理对象  

此处我们使用的是一个演员的例子：
在很久以前，演员和剧组都是直接见面联系的。没有中间人环节。而随着时间的推移，产生了一个新兴职业：经纪人（中间人），这个时候剧组再想找演员就需要通过经纪人来找了。下面我们就用代码演示出来。

```java
/**
* 一个经纪公司的要求:
* 能做基本的表演和危险的表演
*/
public interface IActor {
    
    /**
    * 基本演出
    * @param money
    */
    public void basicAct(float money);
    
    /**
    * 危险演出
    * @param money
    */
    public void dangerAct(float money);
    }

}
```

```java
/**
* 一个演员
*/
//实现了接口，就表示具有接口中的方法实现。即：符合经纪公司的要求
public class Actor implements IActor{
    public void basicAct(float money){
        System.out.println("拿到钱，开始基本的表演： "+money);
    }
    public void dangerAct(float money){
        System.out.println("拿到钱，开始危险的表演： "+money);
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        //一个剧组找演员：
        final Actor actor = new Actor();//直接
        /**
        * 代理：
        * 间接。
        * 获取代理对象：
        * 要求：
        * 被代理类最少实现一个接口
        * 创建的方式
        * Proxy.newProxyInstance(三个参数)
        * 参数含义：
        * ClassLoader：和被代理对象使用相同的类加载器。
        * Interfaces：和被代理对象具有相同的行为。实现相同的接口。
        * InvocationHandler：如何代理。
        * 策略模式：使用场景是：
        * 数据有了，目的明确。
        * 如何达成目标，就是策略。
        *
        */
        IActor proxyActor = (IActor) Proxy.newProxyInstance(
            actor.getClass().getClassLoader(),
            actor.getClass().getInterfaces(),
            new InvocationHandler() {
                /**
                * 执行被代理对象的任何方法，都会经过该方法。
                * 此方法有拦截的功能。
                *
                * 参数：
                * proxy：代理对象的引用。不一定每次都用得到
                * method：当前执行的方法对象
                * args：执行方法所需的参数
                * 返回值：
                * 当前执行方法的返回值
                */
                @Override
                public Object invoke(Object proxy, Method method, Object[] args)
                throws Throwable {
                    String name = method.getName();
                    Float money = (Float) args[0];
                    Object rtValue = null;
                    //每个经纪公司对不同演出收费不一样，此处开始判断
                    if("basicAct".equals(name)){
                        //基本演出，没有 2000 不演
                        if(money > 2000){
                            //看上去剧组是给了 8000，实际到演员手里只有 4000
                            //这就是我们没有修改原来 basicAct 方法源码，对方法进行了增强
                            rtValue = method.invoke(actor, money/2);
                        }
                    }
                    if("dangerAct".equals(name)){
                        //危险演出,没有 5000 不演
                        if(money > 5000){
                            //看上去剧组是给了 50000，实际到演员手里只有 25000
                            //这就是我们没有修改原来 dangerAct 方法源码，对方法进行了增强
                            rtValue = method.invoke(actor, money/2);
                    	}
                	}
                	return rtValue;
            	}
        	});
        //没有经纪公司的时候，直接找演员。
        // actor.basicAct(1000f);
        // actor.dangerAct(5000f);
        //剧组无法直接联系演员，而是由经纪公司找的演员
        proxyActor.basicAct(8000f);
        proxyActor.dangerAct(50000f);
    }
}  
```

#### 使用 CGLib 的 Enhancer 类创建代理对象  

还是那个演员的例子，只不过不让他实现接口。  

```java
/**
* 一个演员
*/
public class Actor{ //没有实现任何接口
    public void basicAct(float money){
    	System.out.println("拿到钱，开始基本的表演： "+money);
    }
    public void dangerAct(float money){
    	System.out.println("拿到钱，开始危险的表演： "+money);
    }
}
```

```java
public class Client {
    /**
    * 基于子类的动态代理
    * 要求：
    * 被代理对象不能是最终类
    * 用到的类：
    * Enhancer
    * 用到的方法：
    * create(Class, Callback)
    * 方法的参数：
    * Class：被代理对象的字节码
    * Callback：如何代理
    * @param args
    */
    public static void main(String[] args) {
        final Actor actor = new Actor();
        Actor cglibActor = (Actor) Enhancer.create(actor.getClass(),
            new MethodInterceptor() {
                /**
                * 执行被代理对象的任何方法，都会经过该方法。在此方法内部就可以对被代理对象的任何
                方法进行增强。
                *
                * 参数：
                * 前三个和基于接口的动态代理是一样的。
                * MethodProxy：当前执行方法的代理对象。
                * 返回值：
                * 当前执行方法的返回值
                */
                @Override
                public Object intercept(Object proxy, Method method, Object[] args,
                MethodProxy methodProxy) throws Throwable {
                    String name = method.getName();
                    Float money = (Float) args[0];
                    Object rtValue = null;
                    if("basicAct".equals(name)){
                        //基本演出
                        if(money > 2000){
                        	rtValue = method.invoke(actor, money/2);
                        }
                    }
                    if("dangerAct".equals(name)){
                        //危险演出
                        if(money > 5000){
                        	rtValue = method.invoke(actor, money/2);
                    	}
                	}
                	return rtValue;
            	}
        	});
        cglibActor.basicAct(10000);
        cglibActor.dangerAct(100000);
    }
}    
```

### 解决案例中的问题

```java
/**
\* 用于创建客户业务层对象工厂（当然也可以创建其他业务层对象，只不过我们此处不做那么繁琐）
\* @Version 1.0
*/
public class BeanFactory {
    /**
    \* 创建账户业务层实现类的代理对象
    \* @return
    */
    public static IAccountService getAccountService() {
        //1.定义被代理对象
        final IAccountService accountService = new AccountServiceImpl();
        //2.创建代理对象
        IAccountService proxyAccountService = (IAccountService)
        Proxy.newProxyInstance(accountService.getClass().getClassLoader(),
        	accountService.getClass().getInterfaces(),
            new InvocationHandler() {
                /**
                \* 执行被代理对象的任何方法，都会经过该方法。
                \* 此处添加事务控制
                */
                @Override
                public Object invoke(Object proxy, Method method,  
                Object[] args) throws Throwable {
                    Object rtValue = null;
                    try {
                        //开启事务
                        TransactionManager.beginTransaction();
                        //执行业务层方法
                        rtValue = method.invoke(accountService, args);
                        //提交事务
                        TransactionManager.commit();
                    }catch(Exception e) {
                        //回滚事务
                        TransactionManager.rollback();
                        e.printStackTrace();
                    }finally {
                        //释放资源
                        TransactionManager.release();
                    }
                    return rtValue;
                }
            });
        return proxyAccountService;
    }
}  
```

当我们改造完成之后，业务层用于控制事务的重复代码就都可以删掉了。  

## AOP相关术语

- Joinpoint(连接点):
  所谓连接点是指那些被拦截到的点。在 spring 中,这些点指的是方法,因为 spring 只支持方法类型的连接点。
- Pointcut(切入点):
  所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。  

- Advice(通知/增强):
  所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。
  通知的类型： 前置通知,后置通知,异常通知,最终通知,环绕通知。
- Introduction(引介):
  引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field。
- Target(目标对象):
  代理的目标对象。
- Weaving(织入):
  是指把增强应用到目标对象来创建新的代理对象的过程。
  spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。
- Proxy（代理） :
  一个类被 AOP 织入增强后，就产生一个结果代理类。
- Aspect(切面):
  是切入点和通知（引介）的结合。  

## AOP实现

### XML

#### 声明 aop 配置  

`<aop:config>`
作用： 用于声明开始 aop 的配置  

```xml
<aop:config>
	<!-- 配置的代码都写在此处 -->
</aop:config>
```

#### 配置切面  

`<aop:aspect>`

- 作用：
  用于配置切面。
- 属性：
  id： 给切面提供一个唯一标识。
  ref： 引用配置好的通知类 bean 的 id。  

```xml
<aop:aspect id="txAdvice" ref="txManager">
	<!--配置通知的类型要写在此处-->
</aop:aspect>
```

#### 配置切入点表达式  

`<aop:pointcut>`

- 作用：
  用于配置切入点表达式。就是指定对哪些类的哪些方法进行增强。
- 属性：
  expression：用于定义切入点表达式。
  id： 用于给切入点表达式提供一个唯一标识  

```xml
<aop:pointcut expression="execution(
    public void com.itheima.service.impl.AccountServiceImpl.transfer(
    java.lang.String, java.lang.String, java.lang.Float)
)" id="pt1"/>
```

#### 配置对应的通知类型  

`<aop:before>`

- 作用：
  用于配置前置通知。 指定增强的方法在切入点方法之前执行
- 属性：
  method:用于指定通知类中的增强方法名称
  ponitcut-ref：用于指定切入点的表达式的引用
  poinitcut：用于指定切入点表达式
- 执行时间点：
  切入点方法执行之前执行  

```xml
<aop:before method="beginTransaction" pointcut-ref="pt1"/>
```

`<aop:after-returning>`

- 作用：
  用于配置后置通知
- 属性：
  method： 指定通知中方法的名称。
  pointct： 定义切入点表达式
  pointcut-ref： 指定切入点表达式的引用
- 执行时间点：
  切入点方法正常执行之后。它和异常通知只能有一个执行  

```xml
<aop:after-returning method="commit" pointcut-ref="pt1"/>
```

`<aop:after-throwing>`

- 作用：
  用于配置异常通知
- 属性：
  method： 指定通知中方法的名称。
  pointct： 定义切入点表达式
  pointcut-ref： 指定切入点表达式的引用
- 执行时间点：
  切入点方法执行产生异常后执行。它和后置通知只能执行一个  

```xml
<aop:after-throwing method="rollback" pointcut-ref="pt1"/>
```

`<aop:after>`

- 作用：
  用于配置最终通知
- 属性：
  method： 指定通知中方法的名称。
  pointct： 定义切入点表达式
  pointcut-ref： 指定切入点表达式的引用
- 执行时间点：
  无论切入点方法执行时是否有异常，它都会在其后面执行。  

```xml
<aop:after method="release" pointcut-ref="pt1"/>
```

#### 切入点表达式说明  

execution:匹配方法的执行(常用)
			execution(表达式)
表达式语法： `execution([修饰符] 返回值类型 包名.类名.方法名(参数))`
写法说明：

- 全匹配方式：
  `public void com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account)`

- 访问修饰符可以省略
  `void com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account)`

- 返回值可以使用*号，表示任意返回值

  \* com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account)

- 包名可以使用\*号，表示任意包，但是有几级包，需要写几个\*

  \* \*.\*.\*.\*.AccountServiceImpl.saveAccount(com.itheima.domain.Account)

- 使用..来表示当前包，及其子包
  \* com..AccountServiceImpl.saveAccount(com.itheima.domain.Account)

- 类名可以使用*号，表示任意类
  \* com..\*.saveAccount(com.itheima.domain.Account)

- 方法名可以使用*号，表示任意方法
  \* com..\*.\*( com.itheima.domain.Account)

- *参数列表可以使用\*，表示参数可以是任意数据类型，但是必须有参数
  \* com..\*.\*(\*)

- *参数列表可以使用..表示有无参数均可，有参数可以是任意类型
  \* com..\*.\*(..)

- 全通配方式：
  \* \*..\*.\*(..)

注：
通常情况下，我们都是对业务层的方法进行增强，所以切入点表达式都是切到业务层实现类。
execution(* com.itheima.service.impl.\*.\*(..))  

#### 环绕通知  

配置方式:  

```xml
<aop:config>
    <aop:pointcut expression="execution(* com.itheima.service.impl.*.*(..))"
    id="pt1"/>
    <aop:aspect id="txAdvice" ref="txManager">
        <!-- 配置环绕通知 -->
        <aop:around method="transactionAround" pointcut-ref="pt1"/>
    </aop:aspect>
</aop:config>
```

`<aop:around>`

- 作用：
  用于配置环绕通知
- 属性：
  method：指定通知中方法的名称。
  pointct：定义切入点表达式
  pointcut-ref：指定切入点表达式的引用
- 说明：
  它是 spring 框架为我们提供的一种可以在代码中手动控制增强代码什么时候执行的方式。
- 注意：
  通常情况下，环绕通知都是独立使用的  

```java
/**
* 环绕通知
* @param pjp
* spring 框架为我们提供了一个接口： ProceedingJoinPoint，它可以作为环绕通知的方法参数。
* 在环绕通知执行时， spring 框架会为我们提供该接口的实现类对象，我们直接使用就行。
* @return
*/
public Object transactionAround(ProceedingJoinPoint pjp) {
    //定义返回值
    Object rtValue = null;
    try {
        //获取方法执行所需的参数
        Object[] args = pjp.getArgs();
        //前置通知：开启事务
        beginTransaction();
        //执行方法
        rtValue = pjp.proceed(args);
        //后置通知：提交事务
        commit();
    }catch(Throwable e) {
        //异常通知：回滚事务
        rollback();
        e.printStackTrace();
    }finally {
        //最终通知：释放资源
        release();
    }
    return rtValue;
}
```



### 注解

#### 通知类使用注解配置  

```java
/**
* 事务控制类
* @Version 1.0
*/
@Component("txManager")
public class TransactionManager {
    //定义一个 DBAssit
    @Autowired
    private DBAssit dbAssit ;
}
```

#### 通知类上使用@Aspect 注解声明为切面  

- 作用：
  把当前类声明为切面类。  

```java
/**
* 事务控制类
* @Version 1.0
*/
@Component("txManager")
@Aspect//表明当前类是一个切面类
public class TransactionManager {
    //定义一个 DBAssit
    @Autowired
    private DBAssit dbAssit ;
}
```

#### 在增强的方法上使用注解配置通知  

`@Before`

- 作用：
  把当前方法看成是前置通知。
- 属性：
  value：用于指定切入点表达式，还可以指定切入点表达式的引用。 

```java
//开启事务
@Before("execution(* com.itheima.service.impl.*.*(..))")
public void beginTransaction() {
    try {
    	dbAssit.getCurrentConnection().setAutoCommit(false);
    } catch (SQLException e) {
    	e.printStackTrace();
    }
}
```

`@AfterReturning`

- 作用：
  把当前方法看成是后置通知。
- 属性：
  value：用于指定切入点表达式，还可以指定切入点表达式的引用  

```java
//提交事务
@AfterReturning("execution(* com.itheima.service.impl.*.*(..))")
public void commit() {
    try {
    	dbAssit.getCurrentConnection().commit();
    } catch (SQLException e) {
    	e.printStackTrace();
    }
}
```

`@AfterThrowing`

- 作用：
  把当前方法看成是异常通知。
- 属性：
  value：用于指定切入点表达式，还可以指定切入点表达式的引用  

```java
//回滚事务
@AfterThrowing("execution(* com.itheima.service.impl.*.*(..))")
public void rollback() {
    try {
    	dbAssit.getCurrentConnection().rollback();
    } catch (SQLException e) {
    	e.printStackTrace();
    }
}
```

`@After`

- 作用：
  把当前方法看成是最终通知。
- 属性：
  value：用于指定切入点表达式，还可以指定切入点表达式的引用  

```java
//释放资源
@After("execution(* com.itheima.service.impl.*.*(..))")
public void release() {
    try {
    	dbAssit.releaseConnection();
    } catch (Exception e) {
    	e.printStackTrace();
    }
}
```

#### 在 spring 配置文件中开启 spring 对注解 AOP 的支持  

```xml
<!-- 开启 spring 对注解 AOP 的支持 -->
<aop:aspectj-autoproxy/>
```

#### 环绕通知注解配置

`@Around`

- 作用：
  把当前方法看成是环绕通知。
- 属性：
  value：用于指定切入点表达式，还可以指定切入点表达式的引用。  

```java
/**
* 环绕通知
* @param pjp
* @return
*/
@Around("execution(* com.itheima.service.impl.*.*(..))")
public Object transactionAround(ProceedingJoinPoint pjp) {
    //定义返回值
    Object rtValue = null;
    try {
        //获取方法执行所需的参数
        Object[] args = pjp.getArgs();
        //前置通知：开启事务
        beginTransaction();
        //执行方法
        rtValue = pjp.proceed(args);
        //后置通知：提交事务
        commit();
    }catch(Throwable e) {
    	//异常通知：回滚事务
        rollback();
        e.printStackTrace();
    }finally {
        //最终通知：释放资源
        release();
    }
    return rtValue;
}
```

#### 切入点表达式注解

`@Pointcut`

- 作用：
  指定切入点表达式

- 属性：  

  value：指定表达式的内容  

```java
@Pointcut("execution(* com.itheima.service.impl.*.*(..))")
private void pt1() {}
```

引用方式：  

```java
/**
* 环绕通知
* @param pjp
* @return
*/
@Around("pt1()")//注意：千万别忘了写括号
public Object transactionAround(ProceedingJoinPoint pjp) {
    //定义返回值
    Object rtValue = null;
    try {
        //获取方法执行所需的参数
        Object[] args = pjp.getArgs();
        //前置通知：开启事务
        beginTransaction();
        //执行方法
        rtValue = pjp.proceed(args);
        //后置通知：提交事务
        commit();
    }catch(Throwable e) {
        //异常通知：回滚事务
        rollback();
        e.printStackTrace();
    }finally {
        //最终通知：释放资源
        release();
    }
    return rtValue;
}
```

#### 不使用 XML 的配置方式  

```java
@Configuration
@ComponentScan(basePackages="com.itheima")
@EnableAspectJAutoProxy
public class SpringConfiguration {
}
```

