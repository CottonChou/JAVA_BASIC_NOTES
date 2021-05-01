> ThreadLocal是线程本地变量，每个线程单独持有一份，所以不存在线程安全的问题，一般用源码推荐使用static修饰使用。一般用来线程内部传递变量或者是用来减少线程内多个函数或者组件之间传递变量的复杂度。

## 简述实现原理

在线程对象内部有一个`ThreadLocalMap`的静态类，线程持有一个`ThreadLocalMap`类型的`threadLocals`的对象。

当我们每次调用`ThreadLocal`的`setget`方法的时候，函数里面首先会获取当前线程，获取线程的`threadLocalMap`，以`ThreadLocal`为key，来取数据或者存数据。

Java在设计之初将`ThreadLocal`类型的key设计成弱引用类型，希望`在ThreadLocal`对象消失后，更快的进行垃圾回收 。但是实际上因为经常使用static的线程池的原因，使用不当会存在内存泄漏和脏数据的情况。

因为首先使用`Threadlocal`的场景就是需要在线程间共享变量，如果不声明为static的话，只是属于某个线程实例类，那么就是去了线程间共享的本质属性。如果申明为静态变量，静态变量对key持有了强引用，那么就不能触发弱引用机制回收。

而且现在应用都是配置了线程池，线程池会复用线程，那么连线程都不会回收了，线程的成员变量`ThreadLocalMap`自然也不会被回收。

如果希望触发GC回收Key，那么需要手动将static修饰的`ThreadLoal`设置为null。

同时希望value被回收或者是在线程池的情况下不产生脏数据，应当调用在合适的地方及时调用remove。



**涉及知识点**

GC Roots

强引用，软引用，弱引用，虚引用

**参考**

https://www.jianshu.com/p/94de80aee1bf?ivk_sa=1024320u

阿里巴巴编码规范