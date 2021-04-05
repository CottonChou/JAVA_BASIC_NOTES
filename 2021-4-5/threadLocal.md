## ThreadLocal 的实现原理

>ThreadLocal是除了加锁这种同步方式之外的一种保证一种规避多线程访问出现线程不安全的方法，当我们在创建一个变量后，如果每个线程对其进行访问的时候访问的都是线程自己的变量这样就不会存在线程不安全问题。

先简单使用一下。

```java
public static void main(String[] args) {
        ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
        threadLocal.set(1);
        System.out.println("取出来的值"+threadLocal.get());
    }
```

在线程的内部持有了两个对象

![image-20210405194756955](threadLocal.assets/image-20210405194756955.png)

当线程第一次ThreadLocal的set方法或者get方法的时候才会创建他们。ThreadLocal类型的本地变量是存放在具体的线程空间上。

通过ThreadLocal的set方法将value设置到线程的Threadlocals.

通过ThreadLocal的get方法从threadlocals中取出变量。

**当调用ThreadLocal的set方法**

```java
//threadlocal的set方法    
public void set(T value) {
    //拿到当前线程
        Thread t = Thread.currentThread();
    //拿到线程的threadlocals,也就是ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            //可以看到 key等于自己，val才是指定的值
            map.set(this, value);
        } else {
            //如果没有就创建并且赋值。
            createMap(t, value);
        }
    }
//getMap
 ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
//createMap
void createMap(Thread t, T firstValue) {
    //可以看到 key等于自己，val才是指定的值
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
//ThreadLocalMap的构造方法
 ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }
```

**get方法**

```java
//ThreadLocal的get方法   
public T get() {
    //拿到当前线程
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
    //如果map不为空，就返回值
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
    //如果为空，就返回初始值。
        return setInitialValue();
    }
```

**remove方法**

```java
//ThreadLocal.remove 
public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null) {
             m.remove(this);
         }
     }
//TheadLocalMap.remove
   private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i);
                    return;
                }
            }
        }
```

在线程类中，持有了一个ThreadLocalMap的实例，名字叫做ThreadLocals,对ThreadLocal的赋值最终会存放在ThreadLocalMap的Entry数组中，计算你声明的ThreadLocal的hashcode，存放在数组的具体位置，所以是可以申明多个ThreadLocal的。

```java
    public static void main(String[] args) {
//        Thread
        ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
        threadLocal.set(99);
        ThreadLocal<String> threadLocal1 = new ThreadLocal<>();
        threadLocal1.set("2");
        ThreadLocal<String> threadLocal3 = new ThreadLocal<>();
        threadLocal3.set("3");
//        2
        System.out.println("第二个取出来的只"+threadLocal1.get());
//        99
        System.out.println("第一个取出来的值"+threadLocal.get());
//        3
        System.out.println("第3个取出来的只"+threadLocal3.get());

    }
```

**内存泄漏**

如果我们申明的ThreadLocal不是静态变量，属于实例的某一个类，那么他就失去了线程间共享的本质属性，如果申明的ThreadLocal是静态变量，那么他的生命周期又不会随着线程结束而结束。所以这很容易产生内存泄露的问题，尽管Entry的key是弱引用，但是我们我们一般使用都是以静态变量来使用，触发弱引用回收机制不能回收value。所以养成使用后remove的好习惯。

**脏数据**

因为线程池的原因，线程复用必然产生脏数据的问题。如果每一次使用完后不去remove,而下一个线程不去set一下，就有可能get到以前的数据。

**Inherited穿透上下文**

其实就是子线程createMap的时候把父线程的所有属性全部拷贝一遍。