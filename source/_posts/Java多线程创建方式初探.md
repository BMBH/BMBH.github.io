---
title: Java多线程创建方式初探
date: 2018-05-23 11:31:40
tags: Java
categories: Java
---

#### Java多线程创建方式初探

##### 多线程概述

- 抢占式多任务	-直接中断而不需要事先和被中断程序协商

- 协作多任务	-被中断程序同意交出控制权之后才能执行中断

- 多线程和多进程区别？
	本质的区别在于每个进程有它自己的变量的完备集，线程则共享相同的数据

##### Thread

- Thread(Runnable target)

	构造有一个新的线程来调用指定的target的run()方法

- void start()

	启动这个线程，将引发调用run()方法

- void run()

	调用关联Runnable的run方法

- Thread 示例测试代码

```
public class ThreadTest extends Thread {

    private Thread mThread;

    private String mName;

    private final int mCount = 4;

    public ThreadTest(String name){
        this.mName = name;
        System.out.println("new ThreadTest"+name);
    }

    public void run(){
        System.out.println("run " + this.mName);
        try {
            for (int i = 0; i < mCount; i++) {
                System.out.println(this.mName + "Thread.sleep : " + i);
                Thread.sleep(50);
            }
        } catch (InterruptedException ie) {
            System.out.println("InterruptedException " + this.mName);
        }
    }

    public void start() {
        System.out.println("start " + this.mName);
        if (this.mThread == null) {
            this.mThread = new Thread(this);
            this.mThread.start();
        }
    }

    public static void main(String[] args) {
        ThreadTest thread1 = new ThreadTest("test1");
        thread1.start();
        ThreadTest thread2 = new ThreadTest("test2");
        thread2.start();
    }
}

```

##### Runnable

Runnable封装一个异步运行的任务

- Runnable示例测试代码

```

/**
 * Runnable继承类
 */
public class RunnableTest implements Runnable {

    private Thread mThread;

    private String mName;

    private final int mCount = 4;

    public RunnableTest(String name) {
        this.mName = name;
        System.out.println("new RunnableTest" + name);
    }

    public void run() {
        System.out.println("run " + this.mName);
        try {
            for (int i = 0; i < mCount; i++) {
                System.out.println(this.mName + "Thread.sleep : " + i);
                Thread.sleep(50);
            }
        } catch (InterruptedException ie) {
            System.out.println("InterruptedException " + this.mName);
        }
    }

    public void start() {
        System.out.println("start " + this.mName);
        if (this.mThread == null) {
            this.mThread = new Thread(this);
            this.mThread.start();
        }
    }

    public static void main(String[] args) {
        RunnableTest run1 = new RunnableTest("test1");
        run1.start();
        RunnableTest run2 = new RunnableTest("test2");
        run2.start();
    }

}


```

##### Callable和Future

Callable接口是一个参数化的类型，有一个方法call
Future保存异步计算的结果。当使用Future对象，启动有一个计算，把计算结果给某线程，Future对象在所有者结果计算好之后就可以得到它

- call()

	运行一个将产生结果的任务

- 代码示例

```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableTest implements Callable<Integer> {

    public Integer call() throws Exception {
        int i = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        return i;
    }

    public static void main(String[] args) {
        CallableTest test = new CallableTest();

        FutureTask<Integer> task = new FutureTask<Integer>(test);

        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " 的循环变量i的值" + i);
            if (i == 20) {
                new Thread(task, "有返回值的线程").start();
            }
        }

        try {
            System.out.println("子线程的返回值：" + task.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }

    }
}

```

##### 线程池

如果你的程序创建了大量生存期很短的线程，就应该使用线程池一个线程池包含大量准备运行的空闲线程。将一个Runnable对象给线程池，线程池中的一个线程就会调用run方法。

- newCachedThreadPool 构建，如果有空闲线程可用，立即让它执行任务，否则创建一个新线程
- newFixedThreadPool 创建一个大小固定的线程池。如果提交的任务数大于空闲线程数，那么得不到服务的任务将被置于队列中
- newSingleTreadExecutor是一个退化了大小为1的线程池

	
	
