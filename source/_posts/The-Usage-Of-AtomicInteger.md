---
title: AtomicInteger的使用与场景
date: 2021-01-11 21:50:58
cover: images/java.jpeg
photos: 
  -  images/java.jpeg
categories: 
  - java基础 
tags: 
  - java
---

本文主要简单介绍java中AtomicInteger方法的使用与场景
<!--more -->

# **问题引入**

引入一个场景：开启两个线程同时对一个静态变量进行累加，代码如下：

```java
public class AtomicIntegerDemo implements Runnable {
    private static int k = 0;

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            k++;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerDemo autoIncrease = new AtomicIntegerDemo();

        Thread thread1 = new Thread(autoIncrease);
        Thread thread2 = new Thread(autoIncrease);

        thread1.start();
        thread2.start();

        Thread.sleep(1000);
        System.out.println(AtomicIntegerDemo.k);
    }
}
```

每个线程对k值累加10000次，我们希望得到的结果是20000。让我们运行一下看看结果：

```java
11251
```

与我们预期的结果20000产生差异的原因主要就是：**多线程并发问题**

# **问题解读**

并发的三大特性：**原子性**、**有序性**、**可见性**（这三大特性在后面的文章再详细讲）。

我们来分析下这个问题。现在有两个线程同时对一个内存中的数字进行累加。k++在内存中执行的流程可以分为三步：

**1. 从主内存中拿值**

**2. 在工作内存里面进行加一操作**

**3. 再把计算后的结果赋值给主内存中**

虽然这三步每步都是原子性操作，但合起来就不是了，因此就有可能会出现线程安全问题。

上述代码的具体流程是：一开始，这两个线程**某个时候**同时从主内存里面拿到k的值，比如k=0，然后分别复制一份到各自线程的工作内存里面（对变量的所有操作均可以理解为在工作内存里面执行），然后执行k++，得到k=1，然后再把k=1赋值给主内存，这样两个线程执行同样的操作，一共累加了两次，主内存的值却还是只有1，所以导致最后的结果始终小于预期值。

那应该怎么保证线程之前互不干扰，得到正确的结果呢？



# **AtomicInteger介绍**

AtomicInteger是jdk源码中concurrent包下的一个类，主要是用于保证整形进行计算时的原子操作，解决并发问题，此外，类似的还有AtomicBoolean、AtomicLong等。

这次主要介绍jdk1.8中的AtomicInteger类。直接看源码：

```java
/**
 * An {@code int} value that may be updated atomically.  See the
 * {@link java.util.concurrent.atomic} package specification for
 * description of the properties of atomic variables. An
 * {@code AtomicInteger} is used in applications such as atomically
 * incremented counters, and cannot be used as a replacement for an
 * {@link java.lang.Integer}. However, this class does extend
 * {@code Number} to allow uniform access by tools and utilities that
 * deal with numerically-based classes.
 *
 * @since 1.5
 * @author Doug Lea
*/
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
```

有几个需要关注的点：

1. 注释中提到：cannot be used as a replacement for an {@link java.lang.Integer}，意思是不能作为Integer类的替代，我在知乎提了个问，我觉得有个大佬说得挺好([知乎链接](https://www.zhihu.com/question/439395937/answer/1684900893))，简而言之就是：**Integer类中的value是final修饰，不可修改的，和int一样，当做常量使用；而AtomicInteger中的可以修改。两个使用场景不同**。当然，性能以及易读性也是原因之一。
2. 可以看到该类继承了Number，和Integer一样，代表着有到基本数据类型的转换。
3. Unsafe unsafe = Unsafe.getUnsafe(); 这段代码是使用了unsafe包下的类，该类不允许在指定包路径下直接调用(反射可以绕过)，主要是涉及到java对内存的操作。

继续往下看：

```java
// value变量在对象内存中的偏移量
private static final long valueOffset;

static {
    try {
        // 通过反射以及unsafe类的方法，获取value字段在对象内存中的偏移量
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
// volatile保证了并发中的可见性和有序性
private volatile int value;
```

接下来看一个常用的方法incrementAndGet()，该方法的作用主要是获取自增后的整数值，类似于++i

```java
/**
* Atomically increments by one the current value.
*
* @return the updated value
*/
public final int incrementAndGet() {
   return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

接着就是保证原子性的核心方法：getAndAddInt(Object o, long offset, int delta)，该方法需要传入三个参数，前两个参数分别传入对象本身和value在对象中的偏移量，最后一个是需要整数的变化量。

```java
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
    v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    return v;
}
```

这个方法中，有一个do...while循环，里面涉及到两个方法getIntVolatile(o, offset)和compareAndSwapInt(o, offset, v, v + delta)

```java
/** Volatile version of {@link #getInt(Object, long)}  */
public native int     getIntVolatile(Object o, long offset);

/**
     * Atomically update Java variable to <tt>x</tt> if it is currently
     * holding <tt>expected</tt>.
     * @return <tt>true</tt> if successful
     */
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
```

均存在native修饰，说明这个方法并不是由java自己实现的。经查阅，

**getIntVolatile(o, offset)** ：通过传入对象以及偏移量，获取内存对应的值

**compareAndSwapInt(o, offset, v, v + delta)** ：

读取传入对象o在内存中偏移量为offset位置的值与期望值expected作比较。

相等就把x值赋值给offset位置的值。方法返回true。

不相等，就取消赋值，方法返回false。

这也是CAS的思想，及比较并交换。用于保证并发时的无锁并发的安全性。

理解了这两个方法后，getAndAddInt方法的作用就很明确了：**先获取对象中内存的值，然后通过CAS方法判断，是否值有发生过变动，若比较方法返回返回false，则说明值已被其它线程的使用了，于是继续循环，直到比较方法返回true，那么返回累加后的值，从而保证了线程间互不干扰**。

# **实际使用**

改造后的代码如下：

```java
public class AtomicIntegerRefactorDemo implements Runnable {
    private static final AtomicInteger k = new AtomicInteger(0);

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            k.incrementAndGet();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerRefactorDemo autoIncrease = new AtomicIntegerRefactorDemo();

        Thread thread1 = new Thread(autoIncrease);
        Thread thread2 = new Thread(autoIncrease);

        thread1.start();
        thread2.start();

        Thread.sleep(1000);
        System.out.println(AtomicIntegerRefactorDemo.k.get());
    }
}
```

再次运行程序，查看结果：

```java
20000
```

这下，终于实现了线程间互不干扰，达到了我们预期的效果。

demo地址： https://github.com/Phukety/study-demo/tree/master/src/main/java/com/phukety/demo/concurrent/atomicinteger