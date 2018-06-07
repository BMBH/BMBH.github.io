---
title: Java 适配器模式
date: 2018-06-06 09:29:11
tags: Java
categories: Java
---


适配器模式用于消除接口不匹配造成的类兼容性问题

类模式的适配器采用继承的方式复用接口

对象模式的适配器采用组合的方式复用

##### 适配器模式-对象模式

新建适配器，接受原类对象的所有方法，然后生成新需要的接口方法

- 原类对象

```
/**
 * 原类
 */
public class Target {

    /**
     * 一种逻辑(算法)
     * @param str
     * @return
     */
    public String Arithmetic(String str) {
        return str;
    }
}
```

- 适配接口


```
/**
 * 适配接口
 */
public interface IAdapter {

    /**
     * 适配逻辑(算法)
     * @param str
     * @return
     */
    String Arithmetic_Another(String str);
}
```

- 适配器



```
public class Adapter implements IAdapter {
    Target tar;

    public Adapter(Target filter) {
        this.tar = filter;
    }

    /**
     * 原逻辑tar
     *
     * @param str
     * @return
     */
    public String Arithmetic(String str) {
        return "原逻辑" + this.tar.Arithmetic(str);
    }

    /**
     * 适配逻辑
     *
     * @param str
     * @return
     */
    public String Arithmetic_Another(String str) {
        return "适配逻辑";
    }
}
```

- 测试


```
    public static void main(String[] args){
        Target t = new Target();
        IAdapter ia = new Adapter(t);
        System.out.println(((Adapter) ia).Arithmetic("……"));
        System.out.println(ia.Arithmetic_Another(""));
    }
```

##### 适配器模式-类模式

通过创建类继承类和实现接口来实现适配

- 原类对象

```
/**
 * 原类
 */
public class Target {

    /**
     * 一种逻辑(算法)
     * @param str
     * @return
     */
    public String Arithmetic(String str) {
        return str;
    }
}
```

- 适配接口


```
/**
 * 适配接口
 */
public interface IAdapter {
    /**
     * 适配逻辑(算法)
     * @param str
     * @return
     */
    public String Arithmetic_Another(String str);

    public String Arithmetic(String str);
}
```

- 适配器



```
public class Adapter extends Target implements IAdapter {
    /**
     * 适配逻辑
     *
     * @param str
     * @return
     */
    public String Arithmetic_Another(String str) {
        return "适配逻辑";
    }
}
```

- 测试


```
    public static void main(String[] args){
        IAdapter ia = new Adapter();
        System.out.println(ia.Arithmetic("原逻辑……"));
        System.out.println(ia.Arithmetic_Another(""));
    }
```
