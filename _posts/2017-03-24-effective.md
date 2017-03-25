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