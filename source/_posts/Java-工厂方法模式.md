---
title: Java 工厂方法模式
date: 2018-06-06 09:29:23
tags: Java
categories: Java
---


在工厂对象上调用创建方法，生成接口的某个实现的对象

通过这种方式，接口与实现分离

- 方法接口

```
/**
 * 方法接口
 */
public interface Service {
    void method1();
    void method2();
}

```

- 工厂方法接口

```
/**
 * 工厂方法接口
 */
public interface ServiceFactory {
    Service getService();
}

```

- 方法实现

```
/**
 * 实现类1
 */
public class Impl1 implements Service {
    public void method1(){System.out.println("Impl1 method1");}
    public void method2(){System.out.println("Impl1 method2");}
}

/**
 * 实现类2
 */
public class Impl2 implements Service{
    public void method1(){System.out.println("Impl2 method1");}
    public void method2(){System.out.println("Impl2 method2");}
}

```

- 工厂方法实现

```
public class ImplFactory1 implements ServiceFactory {
    public Service getService(){
        return new Impl1();
    }
}

public class ImplFactory2 implements  ServiceFactory{
    public Service getService(){
        return new Impl2();
    }
}
```

- 测试

```
public class Test {
    public static void main(String[] args){
        ServiceFactory sf1 = new ImplFactory1();
        Service s1 = sf1.getService();
        s1.method1();
        s1.method2();

        ServiceFactory sf2 = new ImplFactory2();
        Service s2 = sf2.getService();
        s2.method1();
        s2.method2();
    }
}
```