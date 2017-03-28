---
layout: post
title: java 基础总结
---

***

### 概述
+ java四种类型：接口、类、数组、基本类型，前三种是引用类型，基本类型不是对象。类由域、方法、成员类、成员接口组成。方法的签名包含名称和所有参数类型组成，不包含返回值类型。


### 重点详解
1.对象的创建和销毁
+ 利用静态工厂方法替代构造器，一般放在一个不可实例化的类中（如Collections）

```java
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
```

+ 利用Builder模式

```java
/**
NutritionFacts 为不可变，实现了可选参数构造
*/
public class NutritionFacts {
    private final int serviceSize;
    private final int services;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    private NutritionFacts(Builder builder){
        this.serviceSize = builder.serviceSize;
        this.services = builder.services;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    }

    public static class Builder{
        //required
        private final int serviceSize;
        private final int services;
        //optional
        private int calories = 0;
        private int fat = 0;
        private int sodium;
        private int carbohydrate;

        public Builder(int serviceSize, int services) {
            this.serviceSize = serviceSize;
            this.services = services;
        }

        public Builder calories(int calories){
            this.calories = calories;
            return this;
        }

        public Builder fat(int fat){
            this.fat = fat;
            return this;
        }

        public Builder sodium(int sodium){
            this.sodium = sodium;
            return this;
        }

        public Builder carbohydrate(int carbohydrate){
            this.carbohydrate = carbohydrate;
            return this;
        }

        public NutritionFacts build(){
            return new NutritionFacts(this);
        }
    }

    public static void main(String[] args) {
        NutritionFacts cocaCloa = new Builder(240, 8)
                .calories(100)
                .sodium(35)
                .carbohydrate(27)
                .build();
    }
}
```

+ 利用私有构造器或者枚举类型强化Singleton（只被实例化一次的类）属性
采用单元素的类据类型是实现Singleton的最佳实现<br>
[单例模式详解](http://cantellow.iteye.com/blog/838473)

+ 通过私有构造器强化不可实例化的能力Arrays和Collections的做法

+ 避免创建不必要的对象（重用对象）<br>
>适配器（视图）：将功能委托给一个后备对象，为后备对象提供一个可替代的接口（如Map的keySet（）方法）

+ 消除过期的对象引用<br>
>有可能引起内存泄漏，一旦类自己管理内存，就应该警惕内存泄漏<br>
>缓存是另外一个内存泄漏的常见来源（可以用WeakHashMap代表缓存，缓存应该定时清除一下无用的项，由一个后台线程负责）<br>
>监听器和其他回调（最佳实现时支持弱引用）

```java
//引起内存泄漏的例子
public class Stack {
    private Object[] elements;
    private int size;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    private void ensureCapacity(){
        if (elements.length == size){
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    public void push(Object obj){
        ensureCapacity();
        elements[size++] = obj;
    }

    public Object pop(){
        if (size == 0){
            throw new EmptyStackException();
        }
//        return elements[--size];会引起内存泄漏
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
}
```

+ 避免使用终结方法finalize（）

2.对于所有对象都通用的方法
+ 覆写equals时请遵守通用约定<br>
>自反性<br>
>传递性<br>
>对称性<br>
>一致性<br>

>使用==检查参数是否是这个对象的引用<br>
>使用instanceof检查参数是否是正确的类型，其中包含了参数为空的情况<br>
>将参数转换为正确的类型<br>
>检查参数的域是否与该对象的域相匹配，基本类型可以用==进行比较（float和double应该用Float.compare等进行比较）<br>

+ 覆写equals方法总要覆写hashCode方法<br>
>如果两个对象的equals相等，那么hashCode方法的返回结果必须相同<br>
>如果两个对象的euqals不相等，那么hashCode不一定不相等<br>
>倾向于为不同的对象产生不同的hashCode<br>
> 1)把某个非0常数放入result中<br>
> 2)对于对象中的每一个关键域f(指equals中涉及的每一个域)完成下述步骤<br>
>   a为该域计算int型的散列码<br>
>    I. boolean型，f ? 1 : 0<br>
>    II.byte、char、short、int，(int)f<br>
>    III.long，(int)(f^(f>>>32))<br>
>    IV.float，Float.floatToIntBits(f)<br>
>    V.double，(int)(Double.toLongBits(f)^(Double.toLongBits(f))>>>32)<br>
>    VI.对象引用，递归调用hashCode<br>
>    VII.数组，每个元素作为一个单独的域进行处理<br>
>   b.将a中的结果合并result += 31 * result + c（31 * i == i<<5 -i）<br>
>将冗余域排除在外，可以通过其他域计算得到<br>

+ 始终覆写toString方法
+ 谨慎覆写clone<br>
Cloneable 接口表明允许这个对象clone，如果一个类实现了cloneable接口，则object的clone方法会返回该对象的逐域拷贝。<br>
>x.clone() != x && x.clone().getClass == x.getClass()<br>
>首先调用super.clone()，然后修正任何需要修正的域，意味着要拷贝任何包含内部深层结构的可变对象。<br>
>替代方案<br>
> 拷贝构造器<br>

```java
public Yum(Yum yum);
```

> 静态工厂方法<br>

```java
public static Yum newinstance(Yum yum);
```

+ 考虑实现Comparable接口（只有一个compareTo方法）
>强烈建议compareTo与equals方法的逻辑相同<br>

3.类和接口
+ 使类和成员的可访问性最小化<br>
>private：<br>
>package-private：默认设置<br>
>protected：该类的子类可以访问这些成员，该成员的包内部的任何类也可以访问该成员<br>
>public: <br>
+ 在公有域中使用访问方法而非公有域
+ 使可变性最小化
不可变类的要求(String， 基本类型的包装类，BigInteger，BigDecimal):<br>
>不提供任何修改对象状态的方法<br>
>保证类不会被扩展，一般被final修饰<br>
>所有的域为final修饰<br>
>所有的域私有<br>
>确保对于任何可变组建的互斥访问<br>
不可变对象本质上是线程安全的，不要求进行同步，并且永远不需要进行保护性拷贝。

```java
//不可变对象负数操作
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public double realPart(){
        return re;
    }

    public double imaginaryPart(){
        return  im;
    }

    //注意返回的是新对象
    public Complex add(Complex c){
        return new Complex(re + c.re, im + c.im);
    }

    @Override
    public int hashCode() {
        int result = 17 + hashDouble(re);
        result += 31 * result + hashDouble(im);
        return result;
    }

    private int hashDouble(double val){
        long longBits = Double.doubleToLongBits(val);
        return (int)(longBits ^ (longBits >>> 32));
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this){
            return true;
        }
        if (!(obj instanceof Complex)){
            return false;
        }
        Complex dest = (Complex) obj;
        return Double.compare(this.re, dest.re) == 0 &&
                Double.compare(this.im, dest.im) == 0;
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

```java
//不可变对象的另外一种实现
public class ComplexAD {
    private final double re;
    private final double im;

    private ComplexAD(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im){
        return new Complex(re, im);
    }
}
```

+ 复合优于继承（包装类，且不适合用于回调框架）
+ 要么为继承而设计并提供文档说明，要么就禁止继承
+ 接口优于抽象类
+ 接口只用于定义类型
+ 类层次优于标签类
+ 用函数对象表示策略<br>
>只使用一次：通常使用匿名类来声明和实例化<br>
>重复使用：类实现为私有的静态成员类，并通过public static final域被到处<br>
+ 优先考虑静态成员类<br>
>内部类：<br>
> 非静态成员类<br>
> 匿名类<br>
> 局部类<br>
>静态成员类(独立于外部类)<br>
都是嵌套类。

4.泛型<br>
？代表无限制通配符类型
+ 消除非受检警告
+ 列表优先于数组
+ 优先使用泛型
+ 优先考虑泛型方法
+ 利用有限通配符来提升API的灵活性
+ 优先考虑类型安全的异构容器

5.枚举和注解<br>
枚举是由一组固定常量组成合法值的类型<br>
+ 利用枚举替代int常量

```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6);
    private final double mass;
    private final double radius;
    private final double surfaceGravity;
    private static final double G = 6.67300E-11;

    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        this.surfaceGravity = G * mass / (radius * radius);
    }

    public double mass(){
        return mass;
    }

    public double radius(){
        return radius;
    }

    public double surfaceGravity(){
        return surfaceGravity;
    }

    public double surfaceWeight(double mass){
        return mass * surfaceGravity;
    }
}
```

```java
public enum Operation {
    PLUS("+") {
        @Override
        double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        double apply(double x, double y) {
            return x / y;
        }
    };
    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }
    abstract double apply(double x, double y);

    @Override
    public String toString() {
        return symbol;
    }
}
```

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURESDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }
    double pay(double hoursWorked, double payRate){
        return payType.pay(hoursWorked, payRate);
    }

    private enum PayType{
        WEEKDAY{
            @Override
            double overtimePay(double hrs, double payRate) {
                return hrs <= HOUR_PER_SHIFT ? 0 : (hrs - HOUR_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND{
            @Override
            double overtimePay(double hrs, double payRate) {
                return hrs * payRate / 2;
            }
        };
        private static final int HOUR_PER_SHIFT = 8;
        abstract double overtimePay(double hrs, double payRate);
        double pay(double hoursWorked, double payRate){
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
```

+ 用实例域代替序数
+ 用EnumSet替代位域
+ 用EnumMap替代序数索引
+ 用接口模拟可伸缩的枚举
+ 注解优于命名模式
+ 坚持使用Override注解
+ 用标记接口定义类型

6.方法
+ 检查参数的有效性（引用不能为空，索引值必须为非负值）<br>
>抛出的异常一般为IllegalArgumentException、IndexOutOfBoundsException、NullPointerException<br>
>公有方法抛出异常，非公有方法进行断言（assert a != null）<br>
+ 必要时进行保护性拷贝<br>
>在检查参数之前进行，针对拷贝后的对象<br>
+ 谨慎设计方法签名<br>
>谨慎选择方法的名称<br>
>不要过于追求提供便利的方法<br>
>避免提供过长的参数列表（一般不能比四个更多）<br>
> 把方法分解成多个方法<br>
> 创建辅助静态成员类用于保存参数的分组<br>
> 从对象的构建到方法调用都采用Builder模式<br>
>对于参数的类型，应有限使用接口<br>
+ 慎用重载<br>
>对于重载方法的选择是静态的，对于覆写方法的选择是动态的<br>
>对于构造器，可以使用静态工厂方法<br>
+ 慎用可变参数<br>
+ 返回0长度的数组或者集合而不是null
+ 为导出所有的API元素编写注释文档

7.通用程序设计
+ 将局部变量的作用域最小化
+ for-each循环优于传统的for循环<br>
三种情况无法使用for-each<br>
>过滤操作<br>
>转换<br>
>平行迭代<br>
+ 了解和使用类库
+ 如果需要精确的答案，避免使用float和double
>使用BigDecimal、int或者long进行货币计算<br>
+ 基本类型优先于包装基本类型
>基本类型只有值<br>
>基本类型比包装类型节省空间和时间<br>
>基本类型没有null<br>
+ 如果其他类型更合适，则尽量避免使用字符串
+ 当心字符串链接的性能，使用StringBuilder替代
+ 通过接口引用对象
+ 接口优先于反射机制
>java.lang.reflect提供了通过程序来访问已加载的类的信息的能力<br>
> 丧失了编译时检查类型的好处<br>
> 执行反射访问所需的代码非常笨拙和冗余<br>
> 性能损失<br>
+ 谨慎使用本地方法<br>
Java Native Interface（JNI）允许调用本地方法（用本地C或者C++编写的特殊方法）<br>
本地方法由三个用途<br>
>提供“访问特定与平台的机制”的能力，比如访问注册表和文件锁<br>
>访问遗留代码库的能力<br>
>编写程序中注重性能提升的部分<br>
+ 谨慎使用性能优化
+ 遵守普遍接受的命名惯例

8.异常<br>
[异常详解](https://nanhuirong.github.io/Exception)
+ 只针对异常的情况才使用异常
+ 对可恢复的情况使用受检测异常，对编程错误使用运行时异常
+ 避免不必要使用受检测异常
+ 优先使用标准的异常
+ 抛出与抽象相对应的异常
+ 每个方法抛出的异常都要有文档
+ 在细节消息中包含能捕获的失败信息
+ 努力使失败保持原子性
+ 不要忽略异常

9.并发<br>
[java并发编程](https://nanhuirong.github.io/Concurrent)
+ 同步访问共享的可变数据
+ 避免过度同步
+ Executor和Task优先与线程
尽量不要再使用Thread，已经被分为工作单元和执行单元
>工作单元：Task Runnable和Callable<br>
>执行单元：Executor service<br>
+ 并发工具优先于wait和notify
+ 线程安全性文档化
+ 慎用延迟初始化<br>
>实例域采用双重检查模式<br>

```java
private volatile FieldType field;
FieldType getField(){
    FieldType result = field;
    if(result == null){
        synchronized(this){
            result = field;
            if(result == null){
            field = result = computeField();
            }
        }
    }
    return result;
}
```

>静态域采用lazy init holder class idiom<br>

```java
private static class FieldHolder{
    static final FieldType field = computeField();
}
static FieldType getField(){
    return FieldHolder.field;
}
```

>对于接受重复初始化的实例域，也可采用单重检查模式<br>

```java
private volatile FieldType field;
FieldType getField(){
    FieldType result = field;
    if(result == null){
        field = result = computeField();
    }
    return result;
}
```

+ 不要依赖于线程调度器<br>
线程优先级是java平台最不可移植的特征

+ 避免使用线程组ThreadGroup

10.序列化<br>
用于将对象编码成字节流，并从字节流编码中重新构建对象<br>
序列化：将对象编码成字节流<br>
反序列化：将字节流重构成对象<br>
+ 谨慎实现Serializable接口<br>
>UID，每一个可序列化的类有一个唯一的标识号与他关联<br>
+ 考虑使用自定义的序列化形式
transient 该实例域将会冲一个类的默认序列化形式中省略<br>
+ 保护性编写readObject方法
+ 对于实例控制，枚举类型优先于readResolve
+ 考虑使用序列化代理替代序列化实例


