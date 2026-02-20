---
title: ThreadLocal
date: 2025-08-18 11:25:33
tags: [技术碎片]
---
ThreadLocal 是一个 线程本地变量，每个线程都有自己的独立副本。

ThreadLocal 提供了一种线程级别的数据存储机制。每个线程都拥有自己独立的 ThreadLocal ，意味着每个线程都可以独立地、安全地操作这些变量，而不会影响其他线程。

<!-- more -->

## 应用场景和作用

**场景**：

- **用户身份信息存储**：在请求的拦截器/过滤器中鉴权校验用户身份，把**用户信息(如用户ID、权限)存入 ThreadLocal 中**，在请求的后续链路中如果需要获取用户信息的，直接在 ThreadLocal 中获取即可
- **线程安全**：ThreadLocal 可以用来存储一些**需要并发安全处理的成员变量**，比如SimpleDateFormat，由于 SimpleDateFormat 不是线程安全的，可以使用 ThreadLocal **为每个线程创建一个独立的SimpleDateFormat实例，从而避免线程安全问题**
- **日志上下文存储**：在常见的日志框架中，经常使用 **ThreadLocal 来存储与当前线程相关的日志上下文**。这允许开发者在打印日志消息时包含特定于线程的信息，如用户ID，这对于调试和监控是非常有用的（相当于为了打印每条日志时能看到用户ID或其他信息）
- **traceld 存储**：和上面存储日志上下文类似，在分布式链路追踪中，需要存储本次请求的 traceld，通常也都是基于 ThreadLocal 存储的
- **数据库 Session**：很多ORM框架，如Hibernate、Mybatis，都是使用 ThreadLocal 来存储和管理数据库会话的这样可以确保每个线程都有自己的会话实例，避免了在多线程环境中出现的线程安全问题

**两个作用**：

- 在线程中传递数据，在同一个线程执行过程中，ThreadLocal的数据一直在，所以可以在前面把数据放进去，后面需要时再取出来，可避免数据通过参数在多层方法中传递。
- 解决并发问题

## 实现原理

1. Thread 类对象中 维护了 **ThreadLocalMap** 成员变量
2. ThreadLocalMap 类对象中 维护了 **Entry 数组**，Entry数组中的每一个元素都是一个**Entry对象**

- 每个Entry对象中存储着 一个**ThreadLocal对象与其要存入的数据value值**

- 每个Entry对象在Entry数组中的位置是通过ThreadLocal对象的**threadLocalHashCode**计算出来的，以此来快速定位Entry对象在Entry数组中的位置

  3.所以，在Thread中，可以存储多个ThreadLocal对象
其中的核心方法包括set()、get()、remove（）

- ThreadLocal 的 set 方法实现：获取当前线程，获取当前线程的ThreadLocalMap，调用其set方法，存入的 **key** 是**this(即ThreadLocal对象本身)，value是要存入的数据**

```
public void set(T value) {
Thread t = Thread.currentThread() ;
ThreadLocalMap map = getMap (t) ; 
if (map != null) 
	map. set(this, value) ;
else  
	createMap(t, value) ;
```



- 
  ThreadLocal 的 get 方法实现：获取当前线程，获取当前线程的ThreadLocalMap，调用其get方法，通过this(即**ThreadLocal对象本身**) 获取到之前存入的数据

## 内存泄漏

内存泄漏：存在无法回收的对象

**假设我们单独开启一个线程**，并且将数据存储到ThreadLocal中，当 Thread 线程执行任务结束退出时，线程对象被销毁，那么Thread 线程与 ThreadLocalMap 实例对象之间的引用关系就不存在，在 GC 时 ThreadLocalMap 对象、ThreadLocal 对象 、之前存储的数据 都会被回收掉，所以其实不存在内存泄漏



**但是如果使用线程池的方式，核心线程是会反复使用的**，线程中对应的 ThreadLocalMap 会被线程 **强引用**

**所以 ThreadLocalMap 不能被 GC 自动回收，**而 ThreadLocalMap 中包含一个 Entry  数组

【Entry 数组中含有多个< Key 为 ThreadLocal，value为存储的数据 > 的Entry对象】

**虽然 Entry 对象中的 Key 是弱引用，能够被 GC 自动回收**

**但 value 是强引用，不能被 GC 自动回收**，所以，在线程池中使用ThreadLocal会存在内存泄露的风险



```
public class ThreadLocal<T>{
    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    }
}
```



```
强引用指的就是代码中普遍存在的赋值方式，比如A a = new A()这种。这种对象，永远不会被GC回收。并可以通过引用来访问。

软引用可以用**SoftReference**来描述，指的是那些**有用但是不是必须要**的对象（比如缓存）。系统在发生内存溢出前会对这类引用的对象进行回收。可以通过引用访问对象

弱引用可以用**WeakReference**来描述，他的强度比软引用更低一点，弱引用的对象**下一次GC的时候一定会被回收，**而不管内存是否足够。可以通过引用访问对象

虚引用也被称作幻影引用，是最弱的引用关系，可以用**PhantomReference**来描述，他必须和ReferenceQueue一起使用，**同样的当发生GC的时候，虚引用也会被回收**。可以用虚引用来**管理堆外内存**。
```



## **如何避免内存泄漏（解决方案）**

在使用完ThreadLocal对象中保存的数据后，一般在拦截器过滤器的后置处理中，在 finally{} 代码块中调用 ThreadLocal 的 remove() 方法.

​																			参考资料 `月如风`