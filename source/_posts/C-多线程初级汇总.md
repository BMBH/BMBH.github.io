---
title: 'C# 多线程初级汇总'
date: 2018-05-23 15:16:17
tags: C#
categories: .NET
---

#### 异步委托

	创建线程的一种简单方式是定义一个委托，并异步调用它

	委托是方法的类型安全的引用

	Delegate类还支持异步地调用方法。在后台，Delegate类会创建一个执行任务的线程

- 投票，并检查委托是否完成了任务

	- 所创建的Delegate类提供了BeginInvoke()方法，该方法中，可以传递用委托类型定义的输入参数。
	
	- BeginInvoke()方法总是有AsyncCallback和Object类型的两个额外参数
	
	- BeginInvoke()方法返回类型：IAsyncResult
	
	- 代码示例


```
        static void Main(string[] args)
        {
            // synchronous method call
            // TakesAWhile(1,3000);

            // asynchronous by using a delegate
            TakesAWhileDelegate dl = TakesAWhile;
            IAsyncResult ar = dl.BeginInvoke(1, 500, null, null);
            while (!ar.IsCompleted)
            {
                // doing something else in the main thread
                Console.Write(".");
                Thread.Sleep(50);
            }
            int result = dl.EndInvoke(ar);
            Console.WriteLine("result:{0}", result);
        }

        public delegate int TakesAWhileDelegate(int data, int ms);

        static int TakesAWhile(int data, int ms)
        {
            Console.WriteLine("TakesAWhile started");
            Thread.Sleep(ms);
            Console.WriteLine("TakesAWhile completed");
            return ++data;
        }
```


- 使用与IAsyncResult相关联的等待句柄

	- 使用AsyncWaitHandle属性可以访问等待句柄
	
	- 代码示例，在此将上述示例中的While循环更改一下即可，如下

```
            while (true)
            {
                Console.Write(".");
                if (ar.AsyncWaitHandle.WaitOne(50, false))
                {
                    Console.WriteLine("Can get the result now");
                    break;
                }
            }
```

- 异步回调

	- 在BeginInvoke()方法的第3个参数中，可以传递一个满足AsyncCallback委托的需求的方法
	
	- 代码示例

```
            TakesAWhileDelegate dl = TakesAWhile;
            dl.BeginInvoke(1, 500, TakesAWhileCompleted, dl);
            for (int i = 0; i < 100; i++)
            {
                Console.Write(".");
                Thread.Sleep(50);
            }
```

```
        static void TakesAWhileCompleted(IAsyncResult ar)
        {
            if (ar == null)
                throw new ArgumentNullException("ar");
            TakesAWhileDelegate dl = ar.AsyncState as TakesAWhileDelegate;
            Trace.Assert(dl != null, "Invalid object type");

            int result = dl.EndInvoke(ar);
            Console.WriteLine("result:{0}", result);
        }
```


#### Thread类

- Thread类可以创建和控制线程

- 默认情况，Thread类创建线程是前台线程。线程池中的线程总是后台线程。

- 只要有一个前台线程在运行，应用程序的进程就在运行。Thread类创建线程时，可以设置IsBackground属性，以确定该线程时前台线程还是后台线程

- 代码示例

```
            var t1 = new Thread(ThreadMain) { Name = "MyNewThread", IsBackground = false };
            t1.Start();
            Console.WriteLine("Main thread ending now.");

```

```
        static void ThreadMain()
        {
            Console.WriteLine("Thread {0} started", Thread.CurrentThread.Name);
            Thread.Sleep(3000);
            Console.WriteLine("Thread {0} completed", Thread.CurrentThread.Name);
        }
```

#### 线程池

- ThreadPool类托管，在需要时增减池中线程的线程数，直到最大的线程数

- ThreadPool.QueueUserWorkItem()方法，传递一个WaitCallback类型的委托

- 代码示例

```
            int nWorkerThreads;
            int nCompletionPortThreads;
            ThreadPool.GetMaxThreads(out nWorkerThreads, out nCompletionPortThreads);
            Console.WriteLine("Max worker threads:{0},I/O completion threads:{1}",
                nWorkerThreads, nCompletionPortThreads);
            for (int i = 0; i < 5; i++)
            {
                ThreadPool.QueueUserWorkItem(JobForAThread);
            }
```

```
        static void JobForAThread(object state)
        {
            for (int i = 0; i < 3; i++)
            {
                Console.WriteLine("loop{0},running inside pooled thread{1}",
                    i, Thread.CurrentThread.ManagedThreadId);
                Thread.Sleep(50);
            }
        }
```

#### 任务

- .NET 4包含新名称空间 System.Threading.Tasks，它包含的类抽象出了线程功能。在后台使用ThreadPool.

- 启动新任务
	- 实例化TaskFactory类，在其中把TaskMethod()方法传递给StartNew()方法
	- 使用Task类的构造函数，调用Task类的Start()方法
	- 示例代码

```
            //using task factory
            TaskFactory tf = new TaskFactory();
            Task t1 = tf.StartNew(TaskMethod);

            //using the task factory via a task
            Task t2 = Task.Factory.StartNew(TaskMethod);

            //using Task constructor
            Task t3 = new Task(TaskMethod);
            t3.Start();

            Task t4 = new Task(TaskMethod, TaskCreationOptions.PreferFairness);
            t4.Start();
```

```
        static void TaskMethod()
        {
            Console.WriteLine("running in a task");
            Console.WriteLine("Task id:{0}", Task.CurrentId);
        }
```