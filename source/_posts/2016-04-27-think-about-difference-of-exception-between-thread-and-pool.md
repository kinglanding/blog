---
title: 关于异常在线程与线程池不同表现的一个思考
date: 2016-04-27 14:09:21
comments: true
tags: 
- java线程池
- ThreadPool  
- 线程
---

如果一个线程在执行的过程中，遇到了某种异常，中断异常啊、空指针异常啊、算数异常，并且程序本身没有捕获异常，会发送什么？你可能会说，程序直接挂掉呗~然后日志里有异常日志。

但实际上，要看这个线程是在什么地方。如果是在线程池外面，这个程序确实就直接挂掉了；  
<!-- more --> 

```java
    public static void main(String[] args) {
        testThreadFeatures();
    }

    public static void testThreadFeatures() {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {

                char[] ast = null;
                String str = new String(ast);

                System.out.println(Thread.currentThread().getName() + " ： can you see me :>....");

            }
        }, "thread1");

        thread.run();
    }
 ```
 
结果，毫无意外的显示为：
 
```txt
Exception in thread "main" java.lang.NullPointerException
 	at java.lang.String.<init>(String.java:168)
 	at thread.ThreadRunAndHang$1.run(ThreadRunAndHang.java:51)
 	at java.lang.Thread.run(Thread.java:744)
 	at thread.ThreadRunAndHang.testThreadFeatures(ThreadRunAndHang.java:58)
 	at thread.ThreadRunAndHang.main(ThreadRunAndHang.java:42)
 	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
 	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
 	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
 	at java.lang.reflect.Method.invoke(Method.java:606)
 	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)
```
 
如果是在线程池中呢？线程池怎么捕捉线程出现的异常？例如 一个线程因为某个RuntimeException导致线程挂了。如何处理呢？
 
```java
 public static void testThreadPoolFeatures() {
        ExecutorService service = new ThreadPoolExecutor(10, 10, 100, TimeUnit.MILLISECONDS, 
        new LinkedBlockingDeque<Runnable>(), 
        new DefaultThreadFactory());

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + 
                    " ： hello i am running....");
                try {
                    TimeUnit.SECONDS.sleep(30);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + 
                    " : now i begin to make something different....");

                char[] ast = null;
                String str = new String(ast);

                System.out.println(Thread.currentThread().getName() + 
                    " ： can you see me :>....");

            }
        }, "thread1"); // 这里给线程命名其实是不管用的哈！因为有ThreadFactory，那里面对线程重新命名了=。=
        
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + 
                    " ： hello i am running....");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + 
                    " ： now i begin to make something different....");

                System.out.println(Thread.currentThread().getName() + 
                    " ： can you see me :>....");
                while (true) {
                    ;
                }

            }
        }, "thread2");
        service.submit(thread);
        service.submit(thread2);
        System.out.println("running....");
    }
```

结果显示,但是程序本身并没有终止运行。
```bash
 running....
test-1-thread-2 ： hello i am running....
test-1-thread-1 ： hello i am running....
test-1-thread-2 ： now i begin to make something different....
test-1-thread-2 ： can you see me :>....
test-1-thread-1 : now i begin to make something different....
```

感兴趣的话可以查看下程序栈，会发现线程test-1-thread-1是处于WAITING状态的！尝试从queue队列中取新的任务。所以是WAITING状态

```bash
"test-1-thread-2" prio=6 tid=0x000000000c42b800 nid=0x2e9c waiting on condition [0x000000000d10f000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007d6088290> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
	at java.util.concurrent.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:489)
	at java.util.concurrent.LinkedBlockingDeque.take(LinkedBlockingDeque.java:678)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:744)

   Locked ownable synchronizers:
	- None

"test-1-thread-1" prio=6 tid=0x000000000c42b000 nid=0x3a0 waiting on condition [0x000000000cf2f000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000007d6088290> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
	at java.util.concurrent.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:489)
	at java.util.concurrent.LinkedBlockingDeque.take(LinkedBlockingDeque.java:678)
	at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:744)

   Locked ownable synchronizers:
	- None
```

看接口Runnable的定义，及其简洁
```java
public
interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

但是定义成这样，就意味着我们没有办法在run()方法的实现中抛出异常，啊哈哈，没有办法抛出Checked异常啦。要处理checked exception，简单的一个try/catch块就可以了。但是万一程序里不小心抛出一个RuntimeException，又该咋办捏？

还是看Thread源码，已经规定了这么一个玩意儿，`UncaughtExceptionHandler`这个接口的实现可以用来处理那些线程未捕获的异常。

```java
public interface UncaughtExceptionHandler {
        /**
         * Method invoked when the given thread terminates due to the
         * given uncaught exception.
         * <p>Any exception thrown by this method will be ignored by the
         * Java Virtual Machine.
         * @param t the thread
         * @param e the exception
         */
        void uncaughtException(Thread t, Throwable e);
    }

    // null unless explicitly set
    private volatile UncaughtExceptionHandler uncaughtExceptionHandler;

    // null unless explicitly set
    private static volatile UncaughtExceptionHandler defaultUncaughtExceptionHandler;
```
但是默认这类handler是没有设置的（`null unless explicitly set`）！这好办！自个儿写个UncaughtExceptionHandler接口的实现。

```java
static final Thread.UncaughtExceptionHandler exceptionHandler = new Thread.UncaughtExceptionHandler() {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println(t.getName() + "有麻烦了，详情是：" + e.toString());
        e.printStackTrace();
        System.out.println("线程的状态是：" + t.getState());
    }
};
thread.setDefaultUncaughtExceptionHandler(exceptionHandler);
```

这么一来，运行结果就是这样啦。

```java
main有麻烦了，详情是：java.lang.NullPointerException
[Ljava.lang.StackTraceElement;@14c4b664
java.lang.NullPointerException
	at java.lang.String.<init>(String.java:168)
	at thread.ThreadRunAndHang$1.run(ThreadRunAndHang.java:51)
	at java.lang.Thread.run(Thread.java:744)
	at thread.ThreadRunAndHang.testThreadFeatures(ThreadRunAndHang.java:66)
	at thread.ThreadRunAndHang.main(ThreadRunAndHang.java:42)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)
```

这部分的输出正好就是UncaughtExceptionHandler接口的实现。对于unchecked exception，可以采用类似事件注册的机制做一定程度的处理。程序也不会崩溃啦~~

#### 阶段性总结
Java thread里面关于异常的部分比较奇特。接口的定义意味着不能直接在一个线程中抛出异常。一般在线程里碰到checked exception，标准的做法是采用try/catch块来处理。而对于unchecked exception，比较合理的方式是注册一个实现 UncaughtExceptionHandler 接口的对象实例来处理。

在回到线程池的问题中...

为啥线程池没有打印处理这些UncaughtException呢？(低声:因为你没有set线程池中线程的 UncaughtExceptionHandler 啊...)=。=好吧！在DefaultThreadFactory的newThread方法中加上线程的UncaughtExceptionHandler嘛~。运行，一脸黑线，TMD的什么鬼，还是木有！！！

翻看了下 ThreadPoolExecutor 的代码，找到了这个关键点！！！看 runWorker 这个方法的注释哈！原文描述的第4点和第5点很重要~ 不贴代码了，可以看下`jdk1.7.0_51#ThreadPoolExecutor#runWorker#1081~1165`

> 假设 `beforeExecute` 运行没问题， 就开始执行`task`了，注意是调用 `task.ran` 当做普通方法调用的！！然后收集任何抛出的异常，扔给 `afterExecute （默认是空实现哈，所以异常没有被处理）`因为 `Runnable.run` 方法不能抛出 Throwables ，所以就打包扔了出去，其实是扔给线程的 UncaughtExceptionHandler 了。保险起见，任何抛出的异常都会导致线程挂掉。执行完 task.run 方法之后，就开始调用 `afterExecute` 了。 这个方法也可能会抛出异常，此时同样会导致线程挂掉。

这个时候又看了下 `afterExecute` 的注释，重载或者重写该方法就好了，咱们直接这么用就好啦~ ExecutorService 对象的创建方法变一变，覆盖原来的方法.接下来就是见证奇迹的时刻~~

```java
        ExecutorService service = new ThreadPoolExecutor(10, 10, 100, TimeUnit.MILLISECONDS,
                new LinkedBlockingDeque<Runnable>(),
                new DefaultThreadFactory())
                // 覆盖ThreadPoolExecutor中的afterExecute方法！
        {
            protected void afterExecute(Runnable r, Throwable t) {
                super.afterExecute(r, t);
                System.out.println(Thread.currentThread().getName() + "有麻烦了，详情是：" + t.toString());
                t.printStackTrace();
            }
        };
```
运行一下,见证下奇迹,你妹的,千呼万唤始出来啊

```text
test-1-thread-2 ： hello i am running....
test-1-thread-1 ： hello i am running....
test-1-thread-2 ： now i begin to make something different....
test-1-thread-2 ： can you see me :>....
java.lang.NullPointerException
	at thread.ThreadRunAndHang$3.afterExecute(ThreadRunAndHang.java:82)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1153)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
	at java.lang.Thread.run(Thread.java:744)
test-1-thread-1 : now i begin to make something different....
Thread-2
test-1-thread-1有麻烦了，详情是：java.lang.NullPointerException
线程的状态是：RUNNABLE
```

#### 阶段性总结
线程池如果想要处理unchecked exception :
1. 要重新实现线程工厂的newThread方法，需要为线上加上 UncaughtExceptionHandler。
2. 实现  UncaughtExceptionHandler 接口，定义个性化的需求。
3. 覆盖ThreadPoolExecutor.afterExecute 方法。
以上，就能捕获线程池中线程的异常了。

> Reference

1. http://www.blogjava.net/xylz/archive/2013/08/05/402405.html
2. http://stackoverflow.com/questions/1838923/why-is-uncaughtexceptionhandler-not-called-by-executorservice

