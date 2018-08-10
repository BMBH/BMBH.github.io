---
title: Java 代理模式
date: 2018-06-08 11:37:37
tags: Java
categories: Java
---

#### 代理模式

代理模式 实现逻辑和实现的解耦

代理模式 为了提供额外的的操作，插入用来代替实际对象的对象。这些操作通常涉及与实际对象通信，代理充当中间人的角色

- 接口

```
/**
 * 接口
 */
public interface Interface {
    void doSomething();
    void somethingElse(String arg);
}
```

- 实际对象

```
/**
 * 实际对象
 */
public class RealObject implements Interface {
    public void doSomething() {
        System.out.println("doSomething");
    }

    public void somethingElse(String arg) {
        System.out.println("somethingElse" + arg);
    }
}

```

- 代理对象

```
/**
 * 代理对象
 */
public class Proxy implements Interface {
    private Interface proxied;

    public Proxy(Interface proxied) {
        this.proxied = proxied;
    }

    public void doSomething() {
        System.out.println("Proxy doSomething");
        proxied.doSomething();
    }

    public void somethingElse(String arg) {
        System.out.println("Proxy somethingElse" + arg);
        proxied.somethingElse(arg);
    }
}
```

- 测试

```
    /**
     * 测试代理，比较原对象与代理对象
     *
     * @param args
     */
    public static void main(String[] args) {
        Interface iface = new RealObject();
        iface.doSomething();
        iface.somethingElse("bonobo");

        Interface iface2 = new Proxy(iface);
        iface2.doSomething();
        iface2.somethingElse("bonobo");
    }
```

#### 动态代理

Java动态代理可以动态创建代理并动态处理对所代理的方法的调用

在动态里上所做的所有调用都会被重定向到单一的调用处理器上，它的工作是揭示调用的类型并确定对应的对策

- 动态代理

```
public class DynamicProxyHandler implements InvocationHandler {
    private Object proxied;

    public DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("**** proxy:" + proxy.getClass() + ".method: " + method + ".args: " + args);
        if (args != null) {
            for (Object arg : args) {
                System.out.println("    " + args);
            }
        }
        return method.invoke(proxied, args);
    }
}
```

- 测试

```
    public static void main(String[] args){
        RealObject real = new RealObject();
        real.doSomething();
        real.somethingElse("bonobo");

        Interface proxy = (Interface) Proxy.newProxyInstance(
                Interface.class.getClassLoader(),
                new Class[]{Interface.class},
                new DynamicProxyHandler(real));
        proxy.doSomething();
        proxy.somethingElse("bonobo");

    }
```