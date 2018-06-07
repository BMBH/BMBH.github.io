---
title: Java 策略模式(Strategy)
date: 2018-06-06 09:28:46
tags: Java
categories: Java
---


创建一个能够根据所传递的参数对象的不同而具有不同行为的方法

- 要执行的算法固定不变，封装到一个类(Context)中


- 策略就是传递进去的参数对象，它包含执行代码


- 策略接口


```
/**
 * 策略接口
 */
public interface IStrategy {
    String name();

    /**
     * 具体逻辑(算法)
     * @param str
     * @return
     */
    String Arithmetic(String str);
}
```

- 具体实现

```
public class Downcase implements IStrategy {
    public String name() {
        return getClass().getSimpleName();
    }

    public String Arithmetic(String str) {
        return str.toLowerCase();
    }
}

public class UpCase implements IStrategy {
    public String name() {
        return getClass().getSimpleName();
    }

    public String Arithmetic(String str) {
        return str.toUpperCase();
    }
}

public class Splitter implements IStrategy {
    public String name() {
        return getClass().getSimpleName();
    }

    public String Arithmetic(String str) {
        return Arrays.toString(str.split(" "));
    }
}
```

- 封装逻辑(算法)

```
/**
 * 策略模式通过组合的方式实现具体算法
 * 其要执行的算法不变，封装到一个类(Context)中
 */
public class Context {

    private IStrategy mStrategy;

    /**
     * 将抽象接口的实现传递给组合对象
     * @param strategy
     */
    public Context(IStrategy strategy){
        this.mStrategy = strategy;
    }

    /**
     * 封装逻辑(算法)
     * @param s
     */
    public void doAction(String s){
        System.out.println(mStrategy.name());
        System.out.println(this.mStrategy.Arithmetic(s));
    }
}
```

- 测试

```
    public static String s="Disagreement with beliefs is by definition incorrect";

    public static void main(String[] args){
        IStrategy is = new UpCase();
        Context c = new Context(is);
        c.doAction(s);

        IStrategy isd = new Downcase();
        Context c2 = new Context(isd);
        c2.doAction(s);

        IStrategy iss = new Splitter();
        Context c3 = new Context(iss);
        c3.doAction(s);

    }
```

