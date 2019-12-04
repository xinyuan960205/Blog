# 枚举enum
## 一、背景
在java语言中还没有引入枚举类型之前，表示枚举类型的常用模式是声明一组具有int常量。之前我们通常利用`public final static` 方法定义的代码如下，分别用1 表示春天，2表示夏天，3表示秋天，4表示冬天。
```java
public class Season {
    public static final int SPRING = 1;
    public static final int SUMMER = 2;
    public static final int AUTUMN = 3;
    public static final int WINTER = 4;
}
```
这种方法称作int枚举模式。可这种模式有什么问题呢，我们都用了那么久了，应该没问题的。通常我们写出来的代码都会考虑它的安全性、易用性和可读性。 首先我们来考虑一下它的类型安全性。当然这种模式不是类型安全的。比如说我们设计一个函数，要求传入春夏秋冬的某个值。但是使用int类型，我们无法保证传入的值为合法。代码如下所示：
```java
private String getChineseSeason(int season){
        StringBuffer result = new StringBuffer();
        switch(season){
            case Season.SPRING :
                result.append("春天");
                break;
            case Season.SUMMER :
                result.append("夏天");
                break;
            case Season.AUTUMN :
                result.append("秋天");
                break;
            case Season.WINTER :
                result.append("冬天");
                break;
            default :
                result.append("地球没有的季节");
                break;
        }
        return result.toString();
    }

    public void doSomething(){
        System.out.println(this.getChineseSeason(Season.SPRING));//这是正常的场景

        System.out.println(this.getChineseSeason(5));//这个却是不正常的场景，这就导致了类型不安全问题
    }
```
程序`getChineseSeason(Season.SPRING)`是我们预期的使用方法。可`getChineseSeason(5)`显然就不是了，而且编译能通过，在运行时会出现什么情况，我们就不得而知了。这显然就不符合Java程序的类型安全。

接下来我们来考虑一下这种模式的可读性。使用枚举的大多数场合，我都需要方便得到枚举类型的字符串表达式。如果将int枚举常量打印出来，我们所见到的就是一组数字，这是没什么太大的用处。我们可能会想到使用String常量代替int常量。虽然它为这些常量提供了可打印的字符串，但是它会导致性能问题，因为它依赖于字符串的比较操作，所以这种模式也是我们不期望的。 从类型安全性和程序可读性两方面考虑，int和String枚举模式的缺点就显露出来了。幸运的是，从Java1.5发行版本开始，就提出了另一种可以替代的解决方案，可以避免int和String枚举模式的缺点，并提供了许多额外的好处。那就是枚举类型（enum type）。接下来的章节将介绍枚举类型的定义、特征、应用场景和优缺点。

## 二、定义
枚举类型（enum type）是指由一组固定的常量组成合法的类型。Java中由关键字enum来定义一个枚举类型。下面就是java枚举类型的定义。
```java
public enum Season {
    SPRING, SUMMER, AUTUMN, WINER;
}
```

## 三、特点
Java定义枚举类型的语句很简约。它有以下特点：

> 1) 使用关键字enum 
> 2) 类型名称，比如这里的Season 
> 3) 一串允许的值，比如上面定义的春夏秋冬四季 
> 4) 枚举可以单独定义在一个文件中，也可以嵌在其它Java类中

除了这样的基本要求外，用户还有一些其他选择
> 5) 枚举可以实现一个或多个接口（Interface） 
> 6) 可以定义新的变量 
> 7) 可以定义新的方法 
> 8) 可以定义根据具体枚举值而相异的类

## 四、应用场景
以在背景中提到的类型安全为例，用枚举类型重写那段代码。代码如下：
```java
public enum Season {
    SPRING(1), SUMMER(2), AUTUMN(3), WINTER(4);

    private int code;
    private Season(int code){
        this.code = code;
    }

    public int getCode(){
        return code;
    }
}
public class UseSeason {
    /**
     * 将英文的季节转换成中文季节
     * @param season
     * @return
     */
    public String getChineseSeason(Season season){
        StringBuffer result = new StringBuffer();
        switch(season){
            case SPRING :
                result.append("[中文：春天，枚举常量:" + season.name() + "，数据:" + season.getCode() + "]");
                break;
            case AUTUMN :
                result.append("[中文：秋天，枚举常量:" + season.name() + "，数据:" + season.getCode() + "]");
                break;
            case SUMMER : 
                result.append("[中文：夏天，枚举常量:" + season.name() + "，数据:" + season.getCode() + "]");
                break;
            case WINTER :
                result.append("[中文：冬天，枚举常量:" + season.name() + "，数据:" + season.getCode() + "]");
                break;
            default :
                result.append("地球没有的季节 " + season.name());
                break;
        }
        return result.toString();
    }

    public void doSomething(){
        for(Season s : Season.values()){
            System.out.println(getChineseSeason(s));//这是正常的场景
        }
        //System.out.println(getChineseSeason(5));
        //此处已经是编译不通过了，这就保证了类型安全
    }

    public static void main(String[] arg){
        UseSeason useSeason = new UseSeason();
        useSeason.doSomething();
    }
}
```

[中文：春天，枚举常量:SPRING，数据:1]<br> [中文：夏天，枚举常量:SUMMER，数据:2]<br> [中文：秋天，枚举常量:AUTUMN，数据:3]<br> [中文：冬天，枚举常量:WINTER，数据:4]

这里有一个问题，为什么我要将域添加到枚举类型中呢？目的是想将数据与它的常量关联起来。如1代表春天，2代表夏天。

那么什么时候应该使用枚举呢？每当需要一组固定的常量的时候，如一周的天数、一年四季等。或者是在我们编译前就知道其包含的所有值的集合。Java 1.5的枚举能满足绝大部分程序员的要求的，它的简明，易用的特点是很突出的。

## 五、用法
用法一：常量
```java
public enum Color {  
  RED, GREEN, BLANK, YELLOW  
}  
```
用法二：switch
```java
enum Signal {  
    GREEN, YELLOW, RED  
}  
public class TrafficLight {  
    Signal color = Signal.RED;  
    public void change() {  
        switch (color) {  
        case RED:  
            color = Signal.GREEN;  
            break;  
        case YELLOW:  
            color = Signal.RED;  
            break;  
        case GREEN:  
            color = Signal.YELLOW;  
            break;  
        }  
    }  
}  
```
用法三：向枚举中添加新方法
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    // 普通方法  
    public static String getName(int index) {  
        for (Color c : Color.values()) {  
            if (c.getIndex() == index) {  
                return c.name;  
            }  
        }  
        return null;  
    }  
    // get set 方法  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getIndex() {  
        return index;  
    }  
    public void setIndex(int index) {  
        this.index = index;  
    }  
}  
```
用法四：覆盖枚举的方法
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //覆盖方法  
    @Override  
    public String toString() {  
        return this.index+"_"+this.name;  
    }  
}  
```
用法五：实现接口
```java
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
//接口方法  
    @Override  
    public String getInfo() {  
        return this.name;  
    }  
    //接口方法  
    @Override  
    public void print() {  
        System.out.println(this.index+":"+this.name);  
    }  
}  
```
用法六：使用接口组织枚举
```java
public interface Food {  
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
}
```

## 六、枚举的实现
Java SE5提供了一种新的类型-Java的枚举类型，关键字enum可以将一组具名的值的有限集合创建为一种新的类型，而这些具名的值可以作为常规的程序组件使用，这是一种非常有用的功能。

要想看源码，首先得有一个类吧，那么枚举类型到底是什么类呢？是enum吗？答案很明显不是，enum就和class一样，只是一个关键字，他并不是一个类，那么枚举是由什么类维护的呢，我们简单的写一个枚举：
```java
public enum t {
    SPRING,SUMMER;
}
```
然后我们使用反编译，看看这段代码到底是怎么实现的，反编译后代码内容如下：
```java
public final class T extends Enum
{
    private T(String s, int i)
    {
        super(s, i);
    }
    public static T[] values()
    {
        T at[];
        int i;
        T at1[];
        System.arraycopy(at = ENUM$VALUES, 0, at1 = new T[i = at.length], 0, i);
        return at1;
    }

    public static T valueOf(String s)
    {
        return (T)Enum.valueOf(demo/T, s);
    }

    public static final T SPRING;
    public static final T SUMMER;
    private static final T ENUM$VALUES[];
    static
    {
        SPRING = new T("SPRING", 0);
        SUMMER = new T("SUMMER", 1);
        ENUM$VALUES = (new T[] {
            SPRING, SUMMER
        });
    }
}
```
通过反编译后代码我们可以看到，`public final class T extends Enum`，说明，该类是继承了Enum类的，同时final关键字告诉我们，这个类也是不能被继承的。

当我们使用enmu来定义一个枚举类型的时候，编译器会自动帮我们创建一个final类型的类继承Enum类，所以枚举类型不能被继承。