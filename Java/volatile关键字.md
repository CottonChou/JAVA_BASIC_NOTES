## volatile 关键字解决了什么问题，它的实现原理是什么

 

> volatile是一种轻量级别的同步方式，用于保证变量在多线程之间的可见性，且禁止了与volatile变量修饰相关的代码的重排序。volatile不保证原子性。

应用场景有：DCL单例模式，多线程之间通信操作。

**实现原理**

volatile是通过内存屏障实现的可见性。

首先根据Java内存模型规定，每个线程都有自己的工作内存，线程之间的共享变量都存储在主内存中，本地内存只是保存了共享变量的一个拷贝副本，对变量的操作都是基于本地私有内存操作，这就会导致如果两个线程同时读取变量到私有内存空间进行操作，或者是不知道对方已经对变量进行了操作，这就导致了结果与预期不一致。Volatile的规定就是线程读取变量会将自己的私有内存置为无效，必须从主内存读取，线程对变量操作之后必须立刻刷到主内存。

JMM对volatile读操作的时候，在前后插入storeload屏障，写的时候前后插入strorestore屏障。java里面的内存屏障阻止屏障两侧的代码指令重排序，强制让工作内存的数据协会主内存，让工作内存的数据失效。内存屏障是一个CPU指令，不同的硬件实现内存屏障的手段不一样，为了屏蔽这些差异，统一由JVM来生成内存屏障的指令。

## 练习

**volatile的单例模式**

```java
public Volatile Test{
    //首先构造方法私有化
    private Volatile(){}
    //其次定义一个 volatile的实例变量
    public staic volatile Test instance;
    //定义一个获取的方法
    public static Test getInstance(){
        if(instance == null){
            synchronized(Test.class){
                if(instance == null){
                    instance = new Test();
                }
            }
        }
        return instance;
    }
}


/*
public Volatile Test{
    //首先构造方法私有化 确保无法通过调用构造方法破坏单例模式
    private Volatile(){}
    //其次定义一个 volatile的实例变量，确保多线程可见 且
    public staic volatile Test instance;
    //定义一个获取的方法
    public static Test getInstance(){
        if(instance == null){//因为这里加锁了，假设两个线程同时到达，他们都认为是null，此时一个线程初始化完成，另外一个再进去需要再判断一次
            synchronized(Test.class){
                if(instance == null){
                    instance = new Test();
                }
            }
        }
        return instance;
    }
}

为什么使用volatile 修饰了singleton 引用还用synchronized 锁？

volatile 只保证了共享变量 instance 的可见性，但是 instance = new Test(); 这个操作不是原子的，可以分为三步：

步骤1：在堆内存申请一块内存空间；

步骤2：初始化申请好的内存空间；

步骤3：将内存空间的地址赋值给 instance；

所以instance = new Test(); 是一个由三步操作组成的复合操作，多线程环境下A 线程执行了第一步、第二步之后发生线程切换，B 线程开始执行第一步、第二步、第三步（因为A 线程singleton 是还没有赋值的），所以为了保障这三个步骤不可中断，可以使用synchronized 在这段代码块上加锁。
*/
```

**volatile控制两个线程打印顺序**

打印FooBar

```java
public class FooBar {
    private int n;
    private volatile boolean printFooBoolean = true;

    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {

        for (int i = 0; i < n; i++) {
            while (true){
                if (printFooBoolean){
                    printFoo.run();
                    printFooBoolean = ! printFooBoolean;
                    break;
                }else {
                    Thread.yield();
                }
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {

        for (int i = 0; i < n; i++) {
            while (true){
                if (!printFooBoolean){
                    printBar.run();
                    printFooBoolean = ! printFooBoolean;
                    break;
                }else {
                    Thread.yield();
                }
            }
        }
    }
}
```

打印三个数字

```java
public class Foo {
    private volatile int flag = 1;
    public Foo() {

    }

    public void first(Runnable printFirst) throws InterruptedException {
        while (flag != 1){

        }
        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
        flag = 2;
    }

    public void second(Runnable printSecond) throws InterruptedException {
        while (flag !=2){}
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
        flag = 3;
    }

    public void third(Runnable printThird) throws InterruptedException {
        while (flag != 3){

        }
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
    }
}
```

