**<span style="font-size: 35px;">🥮 Java并发 JUC锁详解</span>**

---

## JUC锁: LockSupport详解

---

### LockSupport简介

LockSupport用来创建锁和其他同步类的基本线程阻塞原语。简而言之，当调用LockSupport.park时，表示当前线程将会等待，直至获得许可，当调用LockSupport.unpark时，必须把等待获得许可的线程作为参数进行传递，好让此线程继续运行。

### LockSupport源码分析

#### 类的属性

```java
public class LockSupport {
    // Hotspot implementation via intrinsics API
    private static final sun.misc.Unsafe UNSAFE;
    // 表示内存偏移地址
    private static final long parkBlockerOffset;
    // 表示内存偏移地址
    private static final long SEED;
    // 表示内存偏移地址
    private static final long PROBE;
    // 表示内存偏移地址
    private static final long SECONDARY;
    
    static {
        try {
            // 获取Unsafe实例
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            // 线程类类型
            Class<?> tk = Thread.class;
            // 获取Thread的parkBlocker字段的内存偏移地址
            parkBlockerOffset = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("parkBlocker"));
            // 获取Thread的threadLocalRandomSeed字段的内存偏移地址
            SEED = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSeed"));
            // 获取Thread的threadLocalRandomProbe字段的内存偏移地址
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
            // 获取Thread的threadLocalRandomSecondarySeed字段的内存偏移地址
            SECONDARY = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSecondarySeed"));
        } catch (Exception ex) { throw new Error(ex); }
    }
}  
```

说明: UNSAFE字段表示sun.misc.Unsafe类，查看其源码，点击在这里，一般程序中不允许直接调用，而long型的表示实例对象相应字段在内存中的偏移地址，可以通过该偏移地址获取或者设置该字段的值。

#### 类的构造函数

```java
// 私有构造函数，无法被实例化
private LockSupport() {}
```

说明: LockSupport只有一个私有构造函数，无法被实例化。

#### 核心函数分析

在分析LockSupport函数之前，先引入sun.misc.Unsafe类中的park和unpark函数，因为LockSupport的核心函数都是基于Unsafe类中定义的park和unpark函数，下面给出两个函数的定义:

```java
public native void park(boolean isAbsolute, long time);
public native void unpark(Thread thread);
```

说明: 对两个函数的说明如下:

- park函数，阻塞线程，并且该线程在下列情况发生之前都会被阻塞: ① 调用unpark函数，释放该线程的许可。② 该线程被中断。③ 设置的时间到了。并且，当time为绝对时间时，isAbsolute为true，否则，isAbsolute为false。当time为0时，表示无限等待，直到unpark发生。
- unpark函数，释放线程的许可，即激活调用park后阻塞的线程。这个函数不是安全的，调用这个函数时要确保线程依旧存活。

##### park函数

park函数有两个重载版本，方法摘要如下

```java
public static void park()；
public static void park(Object blocker)；
```

说明: 两个函数的区别在于park()函数没有没有blocker，即没有设置线程的parkBlocker字段。park(Object)型函数如下。

```java
public static void park(Object blocker) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    // 获取许可
    UNSAFE.park(false, 0L);
    // 重新可运行后再此设置Blocker
    setBlocker(t, null);
}
```

说明: 调用park函数时，首先获取当前线程，然后设置当前线程的parkBlocker字段，即调用setBlocker函数，之后调用Unsafe类的park函数，之后再调用setBlocker函数。那么问题来了，为什么要在此park函数中要调用两次setBlocker函数呢? 原因其实很简单，调用park函数时，当前线程首先设置好parkBlocker字段，然后再调用Unsafe的park函数，此后，当前线程就已经阻塞了，等待该线程的unpark函数被调用，所以后面的一个setBlocker函数无法运行，unpark函数被调用，该线程获得许可后，就可以继续运行了，也就运行第二个setBlocker，把该线程的parkBlocker字段设置为null，这样就完成了整个park函数的逻辑。如果没有第二个setBlocker，那么之后没有调用park(Object blocker)，而直接调用getBlocker函数，得到的还是前一个park(Object blocker)设置的blocker，显然是不符合逻辑的。总之，必须要保证在park(Object blocker)整个函数执行完后，该线程的parkBlocker字段又恢复为null。所以，park(Object)型函数里必须要调用setBlocker函数两次。setBlocker方法如下。

```java
private static void setBlocker(Thread t, Object arg) {
    // 设置线程t的parkBlocker字段的值为arg
    UNSAFE.putObject(t, parkBlockerOffset, arg);
}
```

说明: 此方法用于设置线程t的parkBlocker字段的值为arg。

另外一个无参重载版本，park()函数如下。

```java
public static void park() {
    // 获取许可，设置时间为无限长，直到可以获取许可
    UNSAFE.park(false, 0L);
}   
```

说明: 调用了park函数后，会禁用当前线程，除非许可可用。在以下三种情况之一发生之前，当前线程都将处于休眠状态，即下列情况发生时，当前线程会获取许可，可以继续运行。

- 其他某个线程将当前线程作为目标调用 unpark。
- 其他某个线程中断当前线程。
- 该调用不合逻辑地(即毫无理由地)返回。

##### parkNanos函数

此函数表示在许可可用前禁用当前线程，并最多等待指定的等待时间。具体函数如下。

```java
public static void parkNanos(Object blocker, long nanos) {
    if (nanos > 0) { // 时间大于0
        // 获取当前线程
        Thread t = Thread.currentThread();
        // 设置Blocker
        setBlocker(t, blocker);
        // 获取许可，并设置了时间
        UNSAFE.park(false, nanos);
        // 设置许可
        setBlocker(t, null);
    }
}
```

说明: 该函数也是调用了两次setBlocker函数，nanos参数表示相对时间，表示等待多长时间。

##### parkUntil函数

此函数表示在指定的时限前禁用当前线程，除非许可可用, 具体函数如下:

```java
public static void parkUntil(Object blocker, long deadline) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    UNSAFE.park(true, deadline);
    // 设置Blocker为null
    setBlocker(t, null);
}
```

说明: 该函数也调用了两次setBlocker函数，deadline参数表示绝对时间，表示指定的时间。

##### unpark函数

此函数表示如果给定线程的许可尚不可用，则使其可用。如果线程在 park 上受阻塞，则它将解除其阻塞状态。否则，保证下一次调用 park 不会受阻塞。如果给定线程尚未启动，则无法保证此操作有任何效果。具体函数如下:

```java
public static void unpark(Thread thread) {
    if (thread != null) // 线程为不空
        UNSAFE.unpark(thread); // 释放该线程许可
} 
```

说明: 释放许可，指定线程可以继续运行。

### LockSupport示例说明

#### 使用wait/notify实现线程同步

```java
class MyThread extends Thread {
    
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();            
        synchronized (myThread) {
            try {        
                myThread.start();
                // 主线程睡眠3s
                Thread.sleep(3000);
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

运行结果

```html
before wait
before notify
after notify
after wait
```

说明: 具体的流程图如下

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/a6424535-e9e7-43c7-807c-2ecef6bf630d_java-thread-x-locksupport-1.png)

使用wait/notify实现同步时，必须先调用wait，后调用notify，如果先调用notify，再调用wait，将起不了作用。具体代码如下

```java
class MyThread extends Thread {
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();        
        myThread.start();
        // 主线程睡眠3s
        Thread.sleep(3000);
        synchronized (myThread) {
            try {        
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

运行结果:

```html
before notify
after notify
before wait
```

说明: 由于先调用了notify，再调用的wait，此时主线程还是会一直阻塞。

#### 使用park/unpark实现线程同步

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        // 释放许可
        LockSupport.unpark((Thread) object);
        // 休眠500ms，保证先执行park中的setBlocker(t, null);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 再次获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));

        System.out.println("after unpark");
    }
}

public class test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

运行结果:

```html
before park
before unpark
Blocker info ParkAndUnparkDemo
after park
Blocker info null
after unpark
```

说明: 本程序先执行park，然后在执行unpark，进行同步，并且在unpark的前后都调用了getBlocker，可以看到两次的结果不一样，并且第二次调用的结果为null，这是因为在调用unpark之后，执行了Lock.park(Object blocker)函数中的setBlocker(t, null)函数，所以第二次调用getBlocker时为null。

上例是先调用park，然后调用unpark，现在修改程序，先调用unpark，然后调用park，看能不能正确同步。具体代码如下

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");        
        // 释放许可
        LockSupport.unpark((Thread) object);
        System.out.println("after unpark");
    }
}

public class ParkAndUnparkDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        try {
            // 主线程睡眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

运行结果:

```html
before unpark
after unpark
before park
after park
```

说明: 可以看到，在先调用unpark，再调用park时，仍能够正确实现同步，不会造成由wait/notify调用顺序不当所引起的阻塞。因此park/unpark相比wait/notify更加的灵活。

### 中断响应

看下面示例

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before interrupt");        
        try {
            // 休眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }    
        Thread thread = (Thread) object;
        // 中断线程
        thread.interrupt();
        System.out.println("after interrupt");
    }
}

public class InterruptDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

运行结果:

```html
before park
before interrupt
after interrupt
after park
```

说明: 可以看到，在主线程调用park阻塞后，在myThread线程中发出了中断信号，此时主线程会继续运行，也就是说明此时interrupt起到的作用与unpark一样。

### 更深入的理解

#### Thread.sleep()和Object.wait()的区别

首先，我们先来看看Thread.sleep()和Object.wait()的区别，这是一个烂大街的题目了，大家应该都能说上来两点。

- Thread.sleep()不会释放占有的锁，Object.wait()会释放占有的锁；
- Thread.sleep()必须传入时间，Object.wait()可传可不传，不传表示一直阻塞下去；
- Thread.sleep()到时间了会自动唤醒，然后继续执行；
- Object.wait()不带时间的，需要另一个线程使用Object.notify()唤醒；
- Object.wait()带时间的，假如没有被notify，到时间了会自动唤醒，这时又分好两种情况，一是立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁；

其实，他们俩最大的区别就是Thread.sleep()不会释放锁资源，Object.wait()会释放锁资源。

#### Object.wait()和Condition.await()的区别

Object.wait()和Condition.await()的原理是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的。

实际上，它在阻塞当前线程之前还干了两件事，一是把当前线程添加到条件队列中，二是“完全”释放锁，也就是让state状态变量变为0，然后才是调用LockSupport.park()阻塞当前线程。

#### Thread.sleep()和LockSupport.park()的区别

LockSupport.park()还有几个兄弟方法——parkNanos()、parkUtil()等，我们这里说的park()方法统称这一类方法。

- 从功能上来说，Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，且都不会释放当前线程占有的锁资源；
- Thread.sleep()没法从外部唤醒，只能自己醒过来；
- LockSupport.park()方法可以被另一个线程调用LockSupport.unpark()方法唤醒；
- Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出；
- LockSupport.park()方法不需要捕获中断异常；
- Thread.sleep()本身就是一个native方法；
- LockSupport.park()底层是调用的Unsafe的native方法；

#### Object.wait()和LockSupport.park()的区别

二者都会阻塞当前线程的运行，他们有什么区别呢? 经过上面的分析相信你一定很清楚了，真的吗? 往下看！

- Object.wait()方法需要在synchronized块中执行；
- LockSupport.park()可以在任意地方执行；
- Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；
- LockSupport.park()不需要捕获中断异常；
- Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容；
- LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，一定会继续执行后续内容；

park()/unpark()底层的原理是“二元信号量”，你可以把它相像成只有一个许可证的Semaphore，只不过这个信号量在重复执行unpark()的时候也不会再增加许可证，最多只有一个许可证。

#### 如果在wait()之前执行了notify()会怎样?

如果当前的线程不是此对象锁的所有者，却调用该对象的notify()或wait()方法时抛出IllegalMonitorStateException异常；

如果当前线程是此对象锁的所有者，wait()将一直阻塞，因为后续将没有其它notify()唤醒它。

#### 如果在park()之前执行了unpark()会怎样?

线程不会被阻塞，直接跳过park()，继续执行后续内容

#### LockSupport.park()会释放锁资源吗?

不会，它只负责阻塞当前线程，释放锁资源实际上是在Condition的await()方法中实现的。



### 课后问题

- 为什么LockSupport也是核心基础类? AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)
- 写出分别通过wait/notify和LockSupport的park/unpark实现同步?
- LockSupport.park()会释放锁资源吗? 那么Condition.await()呢?
- Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()的区别? 重点
- 如果在wait()之前执行了notify()会怎样?
- 如果在park()之前执行了unpark()会怎样?



### 参考文章

- 文章主要参考自leesf的https://www.cnblogs.com/leesf456/p/5347293.html，在此基础上做了增改。
- https://blog.csdn.net/tangtong1/article/details/102829724





## JUC锁: 锁核心类AQS详解

---

### AbstractQueuedSynchronizer简介

AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

#### AQS 核心思想

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

状态信息通过procted类型的getState，setState，compareAndSetState进行操作

```java
//返回同步状态的当前值
protected final int getState() {  
        return state;
}
 // 设置同步状态的值
protected final void setState(int newState) { 
        state = newState;
}
//原子地(CAS操作)将同步状态值设置为给定值update如果当前同步状态的值等于expect(期望值)
protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

#### AQS 对资源的共享方式

AQS定义两种资源共享方式

- Exclusive(独占)：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
  - 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
  - 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- Share(共享)：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护(如获取资源失败入队/唤醒出队等)，AQS已经在上层已经帮我们实现好了。

#### AQS底层使用了模板方法模式

> 同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样(模板方法模式很经典的一个应用)：

使用者继承AbstractQueuedSynchronizer并重写指定的方法。(这些重写方法很简单，无非是对于共享资源state的获取和释放) 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，模板方法模式请参看：[设计模式行为型 - 模板方法(Template Method)详解]()

AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

默认情况下，每个方法都抛出 UnsupportedOperationException。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0(即释放锁)为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的(state会累加)，这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

### AbstractQueuedSynchronizer数据结构

AbstractQueuedSynchronizer类底层的数据结构是使用`CLH(Craig,Landin,and Hagersten)队列`是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。其中Sync queue，即同步队列，是双向链表，包括head结点和tail结点，head结点主要用作后续的调度。而Condition queue不是必须的，其是一个单向链表，只有当使用Condition时，才会存在此单向链表。并且可能会有多个Condition queue。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/b29659e2-4da0-4a0b-8976-9912b1b796df_java-thread-x-juc-aqs-1.png)

### AbstractQueuedSynchronizer源码分析

#### 类的继承关系

AbstractQueuedSynchronizer继承自AbstractOwnableSynchronizer抽象类，并且实现了Serializable接口，可以进行序列化。

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable
  
        @pdai: 代码已经复制到剪贴板
    
```

1

其中AbstractOwnableSynchronizer抽象类的源码如下:

```java
public abstract class AbstractOwnableSynchronizer implements java.io.Serializable {
    
    // 版本序列号
    private static final long serialVersionUID = 3737899427754241961L;
    // 构造方法
    protected AbstractOwnableSynchronizer() { }
    // 独占模式下的线程
    private transient Thread exclusiveOwnerThread;
    
    // 设置独占线程 
    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
    
    // 获取独占线程 
    protected final Thread getExclusiveOwnerThread() {
        return exclusiveOwnerThread;
    }
}
```

AbstractOwnableSynchronizer抽象类中，可以设置独占资源线程和获取独占资源线程。分别为setExclusiveOwnerThread与getExclusiveOwnerThread方法，这两个方法会被子类调用。

> AbstractQueuedSynchronizer类有两个内部类，分别为Node类与ConditionObject类。下面分别做介绍。

#### 类的内部类 - Node类

```java
static final class Node {
    // 模式，分为共享与独占
    // 共享模式
    static final Node SHARED = new Node();
    // 独占模式
    static final Node EXCLUSIVE = null;        
    // 结点状态
    // CANCELLED，值为1，表示当前的线程被取消
    // SIGNAL，值为-1，表示当前节点的后继节点包含的线程需要运行，也就是unpark
    // CONDITION，值为-2，表示当前节点在等待condition，也就是在condition队列中
    // PROPAGATE，值为-3，表示当前场景下后续的acquireShared能够得以执行
    // 值为0，表示当前节点在sync队列中，等待着获取锁
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;        

    // 结点状态
    volatile int waitStatus;        
    // 前驱结点
    volatile Node prev;    
    // 后继结点
    volatile Node next;        
    // 结点所对应的线程
    volatile Thread thread;        
    // 下一个等待者
    Node nextWaiter;
    
    // 结点是否在共享模式下等待
    final boolean isShared() {
        return nextWaiter == SHARED;
    }
    
    // 获取前驱结点，若前驱结点为空，抛出异常
    final Node predecessor() throws NullPointerException {
        // 保存前驱结点
        Node p = prev; 
        if (p == null) // 前驱结点为空，抛出异常
            throw new NullPointerException();
        else // 前驱结点不为空，返回
            return p;
    }
    
    // 无参构造方法
    Node() {    // Used to establish initial head or SHARED marker
    }
    
    // 构造方法
        Node(Thread thread, Node mode) {    // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }
    
    // 构造方法
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

每个线程被阻塞的线程都会被封装成一个Node结点，放入队列。每个节点包含了一个Thread类型的引用，并且每个节点都存在一个状态，具体状态如下。

- `CANCELLED`，值为1，表示当前的线程被取消。
- `SIGNAL`，值为-1，表示当前节点的后继节点包含的线程需要运行，需要进行unpark操作。
- `CONDITION`，值为-2，表示当前节点在等待condition，也就是在condition queue中。
- `PROPAGATE`，值为-3，表示当前场景下后续的acquireShared能够得以执行。
- 值为0，表示当前节点在sync queue中，等待着获取锁。

#### 类的内部类 - ConditionObject类

这个类有点长，耐心看下:

```java
// 内部类
public class ConditionObject implements Condition, java.io.Serializable {
    // 版本号
    private static final long serialVersionUID = 1173984872572414699L;
    /** First node of condition queue. */
    // condition队列的头节点
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    // condition队列的尾结点
    private transient Node lastWaiter;

    /**
        * Creates a new {@code ConditionObject} instance.
        */
    // 构造方法
    public ConditionObject() { }

    // Internal methods

    /**
        * Adds a new waiter to wait queue.
        * @return its new wait node
        */
    // 添加新的waiter到wait队列
    private Node addConditionWaiter() {
        // 保存尾结点
        Node t = lastWaiter;
        // If lastWaiter is cancelled, clean out.
        if (t != null && t.waitStatus != Node.CONDITION) { // 尾结点不为空，并且尾结点的状态不为CONDITION
            // 清除状态为CONDITION的结点
            unlinkCancelledWaiters(); 
            // 将最后一个结点重新赋值给t
            t = lastWaiter;
        }
        // 新建一个结点
        Node node = new Node(Thread.currentThread(), Node.CONDITION);
        if (t == null) // 尾结点为空
            // 设置condition队列的头节点
            firstWaiter = node;
        else // 尾结点不为空
            // 设置为节点的nextWaiter域为node结点
            t.nextWaiter = node;
        // 更新condition队列的尾结点
        lastWaiter = node;
        return node;
    }

    /**
        * Removes and transfers nodes until hit non-cancelled one or
        * null. Split out from signal in part to encourage compilers
        * to inline the case of no waiters.
        * @param first (non-null) the first node on condition queue
        */
    private void doSignal(Node first) {
        // 循环
        do {
            if ( (firstWaiter = first.nextWaiter) == null) // 该节点的nextWaiter为空
                // 设置尾结点为空
                lastWaiter = null;
            // 设置first结点的nextWaiter域
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                    (first = firstWaiter) != null); // 将结点从condition队列转移到sync队列失败并且condition队列中的头节点不为空，一直循环
    }

    /**
        * Removes and transfers all nodes.
        * @param first (non-null) the first node on condition queue
        */
    private void doSignalAll(Node first) {
        // condition队列的头节点尾结点都设置为空
        lastWaiter = firstWaiter = null;
        // 循环
        do {
            // 获取first结点的nextWaiter域结点
            Node next = first.nextWaiter;
            // 设置first结点的nextWaiter域为空
            first.nextWaiter = null;
            // 将first结点从condition队列转移到sync队列
            transferForSignal(first);
            // 重新设置first
            first = next;
        } while (first != null);
    }

    /**
        * Unlinks cancelled waiter nodes from condition queue.
        * Called only while holding lock. This is called when
        * cancellation occurred during condition wait, and upon
        * insertion of a new waiter when lastWaiter is seen to have
        * been cancelled. This method is needed to avoid garbage
        * retention in the absence of signals. So even though it may
        * require a full traversal, it comes into play only when
        * timeouts or cancellations occur in the absence of
        * signals. It traverses all nodes rather than stopping at a
        * particular target to unlink all pointers to garbage nodes
        * without requiring many re-traversals during cancellation
        * storms.
        */
    // 从condition队列中清除状态为CANCEL的结点
    private void unlinkCancelledWaiters() {
        // 保存condition队列头节点
        Node t = firstWaiter;
        Node trail = null;
        while (t != null) { // t不为空
            // 下一个结点
            Node next = t.nextWaiter;
            if (t.waitStatus != Node.CONDITION) { // t结点的状态不为CONDTION状态
                // 设置t节点的nextWaiter域为空
                t.nextWaiter = null;
                if (trail == null) // trail为空
                    // 重新设置condition队列的头节点
                    firstWaiter = next;
                else // trail不为空
                    // 设置trail结点的nextWaiter域为next结点
                    trail.nextWaiter = next;
                if (next == null) // next结点为空
                    // 设置condition队列的尾结点
                    lastWaiter = trail;
            }
            else // t结点的状态为CONDTION状态
                // 设置trail结点
                trail = t;
            // 设置t结点
            t = next;
        }
    }

    // public methods

    /**
        * Moves the longest-waiting thread, if one exists, from the
        * wait queue for this condition to the wait queue for the
        * owning lock.
        *
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 唤醒一个等待线程。如果所有的线程都在等待此条件，则选择其中的一个唤醒。在从 await 返回之前，该线程必须重新获取锁。
    public final void signal() {
        if (!isHeldExclusively()) // 不被当前线程独占，抛出异常
            throw new IllegalMonitorStateException();
        // 保存condition队列头节点
        Node first = firstWaiter;
        if (first != null) // 头节点不为空
            // 唤醒一个等待线程
            doSignal(first);
    }

    /**
        * Moves all threads from the wait queue for this condition to
        * the wait queue for the owning lock.
        *
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 唤醒所有等待线程。如果所有的线程都在等待此条件，则唤醒所有线程。在从 await 返回之前，每个线程都必须重新获取锁。
    public final void signalAll() {
        if (!isHeldExclusively()) // 不被当前线程独占，抛出异常
            throw new IllegalMonitorStateException();
        // 保存condition队列头节点
        Node first = firstWaiter;
        if (first != null) // 头节点不为空
            // 唤醒所有等待线程
            doSignalAll(first);
    }

    /**
        * Implements uninterruptible condition wait.
        * <ol>
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * </ol>
        */
    // 等待，当前线程在接到信号之前一直处于等待状态，不响应中断
    public final void awaitUninterruptibly() {
        // 添加一个结点到等待队列
        Node node = addConditionWaiter();
        // 获取释放的状态
        int savedState = fullyRelease(node);
        boolean interrupted = false;
        while (!isOnSyncQueue(node)) { // 
            // 阻塞当前线程
            LockSupport.park(this);
            if (Thread.interrupted()) // 当前线程被中断
                // 设置interrupted状态
                interrupted = true; 
        }
        if (acquireQueued(node, savedState) || interrupted) // 
            selfInterrupt();
    }

    /*
        * For interruptible waits, we need to track whether to throw
        * InterruptedException, if interrupted while blocked on
        * condition, versus reinterrupt current thread, if
        * interrupted while blocked waiting to re-acquire.
        */

    /** Mode meaning to reinterrupt on exit from wait */
    private static final int REINTERRUPT =  1;
    /** Mode meaning to throw InterruptedException on exit from wait */
    private static final int THROW_IE    = -1;

    /**
        * Checks for interrupt, returning THROW_IE if interrupted
        * before signalled, REINTERRUPT if after signalled, or
        * 0 if not interrupted.
        */
    private int checkInterruptWhileWaiting(Node node) {
        return Thread.interrupted() ?
            (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
            0; 
    }

    /**
        * Throws InterruptedException, reinterrupts current thread, or
        * does nothing, depending on mode.
        */
    private void reportInterruptAfterWait(int interruptMode)
        throws InterruptedException {
        if (interruptMode == THROW_IE)
            throw new InterruptedException();
        else if (interruptMode == REINTERRUPT)
            selfInterrupt();
    }

    /**
        * Implements interruptible condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled or interrupted.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * </ol>
        */
    // // 等待，当前线程在接到信号或被中断之前一直处于等待状态
    public final void await() throws InterruptedException {
        if (Thread.interrupted()) // 当前线程被中断，抛出异常
            throw new InterruptedException();
        // 在wait队列上添加一个结点
        Node node = addConditionWaiter();
        // 
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            // 阻塞当前线程
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0) // 检查结点等待时的中断类型
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }

    /**
        * Implements timed condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled, interrupted, or timed out.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * </ol>
        */
    // 等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态 
    public final long awaitNanos(long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        final long deadline = System.nanoTime() + nanosTimeout;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (nanosTimeout <= 0L) {
                transferAfterCancelledWait(node);
                break;
            }
            if (nanosTimeout >= spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
            nanosTimeout = deadline - System.nanoTime();
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return deadline - System.nanoTime();
    }

    /**
        * Implements absolute timed condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled, interrupted, or timed out.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * <li> If timed out while blocked in step 4, return false, else true.
        * </ol>
        */
    // 等待，当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态
    public final boolean awaitUntil(Date deadline)
            throws InterruptedException {
        long abstime = deadline.getTime();
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        boolean timedout = false;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (System.currentTimeMillis() > abstime) {
                timedout = transferAfterCancelledWait(node);
                break;
            }
            LockSupport.parkUntil(this, abstime);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return !timedout;
    }

    /**
        * Implements timed condition wait.
        * <ol>
        * <li> If current thread is interrupted, throw InterruptedException.
        * <li> Save lock state returned by {@link #getState}.
        * <li> Invoke {@link #release} with saved state as argument,
        *      throwing IllegalMonitorStateException if it fails.
        * <li> Block until signalled, interrupted, or timed out.
        * <li> Reacquire by invoking specialized version of
        *      {@link #acquire} with saved state as argument.
        * <li> If interrupted while blocked in step 4, throw InterruptedException.
        * <li> If timed out while blocked in step 4, return false, else true.
        * </ol>
        */
    // 等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。此方法在行为上等效于: awaitNanos(unit.toNanos(time)) > 0
    public final boolean await(long time, TimeUnit unit)
            throws InterruptedException {
        long nanosTimeout = unit.toNanos(time);
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        final long deadline = System.nanoTime() + nanosTimeout;
        boolean timedout = false;
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            if (nanosTimeout <= 0L) {
                timedout = transferAfterCancelledWait(node);
                break;
            }
            if (nanosTimeout >= spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
            nanosTimeout = deadline - System.nanoTime();
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null)
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
        return !timedout;
    }

    //  support for instrumentation

    /**
        * Returns true if this condition was created by the given
        * synchronization object.
        *
        * @return {@code true} if owned
        */
    final boolean isOwnedBy(AbstractQueuedSynchronizer sync) {
        return sync == AbstractQueuedSynchronizer.this;
    }

    /**
        * Queries whether any threads are waiting on this condition.
        * Implements {@link AbstractQueuedSynchronizer#hasWaiters(ConditionObject)}.
        *
        * @return {@code true} if there are any waiting threads
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    //  查询是否有正在等待此条件的任何线程
    protected final boolean hasWaiters() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION)
                return true;
        }
        return false;
    }

    /**
        * Returns an estimate of the number of threads waiting on
        * this condition.
        * Implements {@link AbstractQueuedSynchronizer#getWaitQueueLength(ConditionObject)}.
        *
        * @return the estimated number of waiting threads
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 返回正在等待此条件的线程数估计值
    protected final int getWaitQueueLength() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        int n = 0;
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION)
                ++n;
        }
        return n;
    }

    /**
        * Returns a collection containing those threads that may be
        * waiting on this Condition.
        * Implements {@link AbstractQueuedSynchronizer#getWaitingThreads(ConditionObject)}.
        *
        * @return the collection of threads
        * @throws IllegalMonitorStateException if {@link #isHeldExclusively}
        *         returns {@code false}
        */
    // 返回包含那些可能正在等待此条件的线程集合
    protected final Collection<Thread> getWaitingThreads() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        ArrayList<Thread> list = new ArrayList<Thread>();
        for (Node w = firstWaiter; w != null; w = w.nextWaiter) {
            if (w.waitStatus == Node.CONDITION) {
                Thread t = w.thread;
                if (t != null)
                    list.add(t);
            }
        }
        return list;
    }
}
```

此类实现了Condition接口，Condition接口定义了条件操作规范，具体如下

```java
public interface Condition {

    // 等待，当前线程在接到信号或被中断之前一直处于等待状态
    void await() throws InterruptedException;
    
    // 等待，当前线程在接到信号之前一直处于等待状态，不响应中断
    void awaitUninterruptibly();
    
    //等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态 
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    
    // 等待，当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。此方法在行为上等效于: awaitNanos(unit.toNanos(time)) > 0
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    
    // 等待，当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态
    boolean awaitUntil(Date deadline) throws InterruptedException;
    
    // 唤醒一个等待线程。如果所有的线程都在等待此条件，则选择其中的一个唤醒。在从 await 返回之前，该线程必须重新获取锁。
    void signal();
    
    // 唤醒所有等待线程。如果所有的线程都在等待此条件，则唤醒所有线程。在从 await 返回之前，每个线程都必须重新获取锁。
    void signalAll();
}
```

Condition接口中定义了await、signal方法，用来等待条件、释放条件。之后会详细分析CondtionObject的源码。

#### 类的属性

属性中包含了头节点head，尾结点tail，状态state、自旋时间spinForTimeoutThreshold，还有AbstractQueuedSynchronizer抽象的属性在内存中的偏移地址，通过该偏移地址，可以获取和设置该属性的值，同时还包括一个静态初始化块，用于加载内存偏移地址。

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer
    implements java.io.Serializable {    
    // 版本号
    private static final long serialVersionUID = 7373984972572414691L;    
    // 头节点
    private transient volatile Node head;    
    // 尾结点
    private transient volatile Node tail;    
    // 状态
    private volatile int state;    
    // 自旋时间
    static final long spinForTimeoutThreshold = 1000L;
    
    // Unsafe类实例
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    // state内存偏移地址
    private static final long stateOffset;
    // head内存偏移地址
    private static final long headOffset;
    // state内存偏移地址
    private static final long tailOffset;
    // tail内存偏移地址
    private static final long waitStatusOffset;
    // next内存偏移地址
    private static final long nextOffset;
    // 静态初始化块
    static {
        try {
            stateOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("state"));
            headOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("head"));
            tailOffset = unsafe.objectFieldOffset
                (AbstractQueuedSynchronizer.class.getDeclaredField("tail"));
            waitStatusOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("waitStatus"));
            nextOffset = unsafe.objectFieldOffset
                (Node.class.getDeclaredField("next"));

        } catch (Exception ex) { throw new Error(ex); }
    }
}
```

#### 类的构造方法

此类构造方法为从抽象构造方法，供子类调用。

```java
protected AbstractQueuedSynchronizer() { } 
```

#### 类的核心方法 - acquire方法

该方法以独占模式获取(资源)，忽略中断，即线程在aquire过程中，中断此线程是无效的。源码如下:

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

由上述源码可以知道，当一个线程调用acquire时，调用方法流程如下

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/0dc509d7-3c21-4729-8496-a50ba58a16fe_java-thread-x-juc-aqs-2.png)

- 首先调用tryAcquire方法，调用此方法的线程会试图在独占模式下获取对象状态。此方法应该查询是否允许它在独占模式下获取对象状态，如果允许，则获取它。在AbstractQueuedSynchronizer源码中默认会抛出一个异常，即需要子类去重写此方法完成自己的逻辑。之后会进行分析。
- 若tryAcquire失败，则调用addWaiter方法，addWaiter方法完成的功能是将调用此方法的线程封装成为一个结点并放入Sync queue。
- 调用acquireQueued方法，此方法完成的功能是Sync queue中的结点不断尝试获取资源，若成功，则返回true，否则，返回false。
- 由于tryAcquire默认实现是抛出异常，所以此时，不进行分析，之后会结合一个例子进行分析。

首先分析addWaiter方法

```java
// 添加等待者
private Node addWaiter(Node mode) {
    // 新生成一个结点，默认为独占模式
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    // 保存尾结点
    Node pred = tail;
    if (pred != null) { // 尾结点不为空，即已经被初始化
        // 将node结点的prev域连接到尾结点
        node.prev = pred; 
        if (compareAndSetTail(pred, node)) { // 比较pred是否为尾结点，是则将尾结点设置为node 
            // 设置尾结点的next域为node
            pred.next = node;
            return node; // 返回新生成的结点
        }
    }
    enq(node); // 尾结点为空(即还没有被初始化过)，或者是compareAndSetTail操作失败，则入队列
    return node;
}
```

addWaiter方法使用快速添加的方式往sync queue尾部添加结点，如果sync queue队列还没有初始化，则会使用enq插入队列中，enq方法源码如下

```java
private Node enq(final Node node) {
    for (;;) { // 无限循环，确保结点能够成功入队列
        // 保存尾结点
        Node t = tail;
        if (t == null) { // 尾结点为空，即还没被初始化
            if (compareAndSetHead(new Node())) // 头节点为空，并设置头节点为新生成的结点
                tail = head; // 头节点与尾结点都指向同一个新生结点
        } else { // 尾结点不为空，即已经被初始化过
            // 将node结点的prev域连接到尾结点
            node.prev = t; 
            if (compareAndSetTail(t, node)) { // 比较结点t是否为尾结点，若是则将尾结点设置为node
                // 设置尾结点的next域为node
                t.next = node; 
                return t; // 返回尾结点
            }
        }
    }
}
```

enq方法会使用无限循环来确保节点的成功插入。

现在，分析acquireQueue方法。其源码如下

```java
// sync队列中的结点在独占且忽略中断的模式下获取(资源)
final boolean acquireQueued(final Node node, int arg) {
    // 标志
    boolean failed = true;
    try {
        // 中断标志
        boolean interrupted = false;
        for (;;) { // 无限循环
            // 获取node节点的前驱结点
            final Node p = node.predecessor(); 
            if (p == head && tryAcquire(arg)) { // 前驱为头节点并且成功获得锁
                setHead(node); // 设置头节点
                p.next = null; // help GC
                failed = false; // 设置标志
                return interrupted; 
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

首先获取当前节点的前驱节点，如果前驱节点是头节点并且能够获取(资源)，代表该当前节点能够占有锁，设置头节点为当前节点，返回。否则，调用shouldParkAfterFailedAcquire和parkAndCheckInterrupt方法，首先，我们看shouldParkAfterFailedAcquire方法，代码如下

```java
// 当获取(资源)失败后，检查并且更新结点状态
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 获取前驱结点的状态
    int ws = pred.waitStatus;
    if (ws == Node.SIGNAL) // 状态为SIGNAL，为-1
        /*
            * This node has already set status asking a release
            * to signal it, so it can safely park.
            */
        // 可以进行park操作
        return true; 
    if (ws > 0) { // 表示状态为CANCELLED，为1
        /*
            * Predecessor was cancelled. Skip over predecessors and
            * indicate retry.
            */
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0); // 找到pred结点前面最近的一个状态不为CANCELLED的结点
        // 赋值pred结点的next域
        pred.next = node; 
    } else { // 为PROPAGATE -3 或者是0 表示无状态,(为CONDITION -2时，表示此节点在condition queue中) 
        /*
            * waitStatus must be 0 or PROPAGATE.  Indicate that we
            * need a signal, but don't park yet.  Caller will need to
            * retry to make sure it cannot acquire before parking.
            */
        // 比较并设置前驱结点的状态为SIGNAL
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL); 
    }
    // 不能进行park操作
    return false;
}
```

只有当该节点的前驱结点的状态为SIGNAL时，才可以对该结点所封装的线程进行park操作。否则，将不能进行park操作。再看parkAndCheckInterrupt方法，源码如下

```java
// 进行park操作并且返回该线程是否被中断
private final boolean parkAndCheckInterrupt() {
    // 在许可可用之前禁用当前线程，并且设置了blocker
    LockSupport.park(this);
    return Thread.interrupted(); // 当前线程是否已被中断，并清除中断标记位
}
```

parkAndCheckInterrupt方法里的逻辑是首先执行park操作，即禁用当前线程，然后返回该线程是否已经被中断。再看final块中的cancelAcquire方法，其源码如下

```java
// 取消继续获取(资源)
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    // node为空，返回
    if (node == null)
        return;
    // 设置node结点的thread为空
    node.thread = null;

    // Skip cancelled predecessors
    // 保存node的前驱结点
    Node pred = node.prev;
    while (pred.waitStatus > 0) // 找到node前驱结点中第一个状态小于0的结点，即不为CANCELLED状态的结点
        node.prev = pred = pred.prev;

    // predNext is the apparent node to unsplice. CASes below will
    // fail if not, in which case, we lost race vs another cancel
    // or signal, so no further action is necessary.
    // 获取pred结点的下一个结点
    Node predNext = pred.next;

    // Can use unconditional write instead of CAS here.
    // After this atomic step, other Nodes can skip past us.
    // Before, we are free of interference from other threads.
    // 设置node结点的状态为CANCELLED
    node.waitStatus = Node.CANCELLED;

    // If we are the tail, remove ourselves.
    if (node == tail && compareAndSetTail(node, pred)) { // node结点为尾结点，则设置尾结点为pred结点
        // 比较并设置pred结点的next节点为null
        compareAndSetNext(pred, predNext, null); 
    } else { // node结点不为尾结点，或者比较设置不成功
        // If successor needs signal, try to set pred's next-link
        // so it will get one. Otherwise wake it up to propagate.
        int ws;
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
                (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) { // (pred结点不为头节点，并且pred结点的状态为SIGNAL)或者 
                                // pred结点状态小于等于0，并且比较并设置等待状态为SIGNAL成功，并且pred结点所封装的线程不为空
            // 保存结点的后继
            Node next = node.next;
            if (next != null && next.waitStatus <= 0) // 后继不为空并且后继的状态小于等于0
                compareAndSetNext(pred, predNext, next); // 比较并设置pred.next = next;
        } else {
            unparkSuccessor(node); // 释放node的前一个结点
        }

        node.next = node; // help GC
    }
}
```

该方法完成的功能就是取消当前线程对资源的获取，即设置该结点的状态为CANCELLED，接着我们再看unparkSuccessor方法，源码如下

```java
// 释放后继结点
private void unparkSuccessor(Node node) {
    /*
        * If status is negative (i.e., possibly needing signal) try
        * to clear in anticipation of signalling.  It is OK if this
        * fails or if status is changed by waiting thread.
        */
    // 获取node结点的等待状态
    int ws = node.waitStatus;
    if (ws < 0) // 状态值小于0，为SIGNAL -1 或 CONDITION -2 或 PROPAGATE -3
        // 比较并且设置结点等待状态，设置为0
        compareAndSetWaitStatus(node, ws, 0);

    /*
        * Thread to unpark is held in successor, which is normally
        * just the next node.  But if cancelled or apparently null,
        * traverse backwards from tail to find the actual
        * non-cancelled successor.
        */
    // 获取node节点的下一个结点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) { // 下一个结点为空或者下一个节点的等待状态大于0，即为CANCELLED
        // s赋值为空
        s = null; 
        // 从尾结点开始从后往前开始遍历
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0) // 找到等待状态小于等于0的结点，找到最前的状态小于等于0的结点
                // 保存结点
                s = t;
    }
    if (s != null) // 该结点不为为空，释放许可
        LockSupport.unpark(s.thread);
}
```

该方法的作用就是为了释放node节点的后继结点。

对于cancelAcquire与unparkSuccessor方法，如下示意图可以清晰的表示:

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/b66a5806-1ab2-47aa-89d7-5054684cbcdd_java-thread-x-juc-aqs-3.png)

其中node为参数，在执行完cancelAcquire方法后的效果就是unpark了s结点所包含的t4线程。

现在，再来看acquireQueued方法的整个的逻辑。逻辑如下:

- 判断结点的前驱是否为head并且是否成功获取(资源)。
- 若步骤1均满足，则设置结点为head，之后会判断是否finally模块，然后返回。
- 若步骤2不满足，则判断是否需要park当前线程，是否需要park当前线程的逻辑是判断结点的前驱结点的状态是否为SIGNAL，若是，则park当前结点，否则，不进行park操作。
- 若park了当前线程，之后某个线程对本线程unpark后，并且本线程也获得机会运行。那么，将会继续进行步骤①的判断。

#### 类的核心方法 - release方法

以独占模式释放对象，其源码如下:

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) { // 释放成功
        // 保存头节点
        Node h = head; 
        if (h != null && h.waitStatus != 0) // 头节点不为空并且头节点状态不为0
            unparkSuccessor(h); //释放头节点的后继结点
        return true;
    }
    return false;
}
```

其中，tryRelease的默认实现是抛出异常，需要具体的子类实现，如果tryRelease成功，那么如果头节点不为空并且头节点的状态不为0，则释放头节点的后继结点，unparkSuccessor方法已经分析过，不再累赘。

对于其他方法我们也可以分析，与前面分析的方法大同小异，所以，不再累赘。

### AbstractQueuedSynchronizer示例详解一

借助下面示例来分析AbstractQueuedSyncrhonizer内部的工作机制。示例源码如下

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class MyThread extends Thread {
    private Lock lock;
    public MyThread(String name, Lock lock) {
        super(name);
        this.lock = lock;
    }
    
    public void run () {
        lock.lock();
        try {
            System.out.println(Thread.currentThread() + " running");
        } finally {
            lock.unlock();
        }
    }
}
public class AbstractQueuedSynchonizerDemo {
    public static void main(String[] args) {
        Lock lock = new ReentrantLock();
        
        MyThread t1 = new MyThread("t1", lock);
        MyThread t2 = new MyThread("t2", lock);
        t1.start();
        t2.start();    
    }
}
```

运行结果(可能的一种):

```html
Thread[t1,5,main] running
Thread[t2,5,main] running
```

结果分析: 从示例可知，线程t1与t2共用了一把锁，即同一个lock。可能会存在如下一种时序。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/f186ef10-fdfe-480a-8c29-4c7e8cd98b98_java-thread-x-juc-aqs-4.png)

说明: 首先线程t1先执行lock.lock操作，然后t2执行lock.lock操作，然后t1执行lock.unlock操作，最后t2执行lock.unlock操作。基于这样的时序，分析AbstractQueuedSynchronizer内部的工作机制。

- t1线程调用lock.lock方法，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/4a7c71f7-4ec9-4960-85fe-bf727b44e4c9_java-thread-x-juc-aqs-5.png)

说明: 其中，前面的部分表示哪个类，后面是具体的类中的哪个方法，AQS表示AbstractQueuedSynchronizer类，AOS表示AbstractOwnableSynchronizer类。

- t2线程调用lock.lock方法，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/9bdbd848-2cac-496c-8272-8e0ce1d5b8e0_java-thread-x-juc-aqs-6.png)

说明: 经过一系列的方法调用，最后达到的状态是禁用t2线程，因为调用了LockSupport.park。

- t1线程调用lock.unlock，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/658d4790-e9da-4d27-8d31-1caa4f5f92a7_java-thread-x-juc-aqs-7.png)

说明: t1线程中调用lock.unlock后，经过一系列的调用，最终的状态是释放了许可，因为调用了LockSupport.unpark。这时，t2线程就可以继续运行了。此时，会继续恢复t2线程运行环境，继续执行LockSupport.park后面的语句，即进一步调用如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/854eb038-364b-46f8-8c75-f50ebceb461f_java-thread-x-juc-aqs-8.png)

说明: 在上一步调用了LockSupport.unpark后，t2线程恢复运行，则运行parkAndCheckInterrupt，之后，继续运行acquireQueued方法，最后达到的状态是头节点head与尾结点tail均指向了t2线程所在的结点，并且之前的头节点已经从sync队列中断开了。

- t2线程调用lock.unlock，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/184bb360-e504-406b-8777-e1b5d0b5d2b1_java-thread-x-juc-aqs-9.png)

说明: t2线程执行lock.unlock后，最终达到的状态还是与之前的状态一样。

### AbstractQueuedSynchronizer示例详解二

下面我们结合Condition实现生产者与消费者，来进一步分析AbstractQueuedSynchronizer的内部工作机制。

- Depot(仓库)类

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Depot {
    private int size;
    private int capacity;
    private Lock lock;
    private Condition fullCondition;
    private Condition emptyCondition;
    
    public Depot(int capacity) {
        this.capacity = capacity;    
        lock = new ReentrantLock();
        fullCondition = lock.newCondition();
        emptyCondition = lock.newCondition();
    }
    
    public void produce(int no) {
        lock.lock();
        int left = no;
        try {
            while (left > 0) {
                while (size >= capacity)  {
                    System.out.println(Thread.currentThread() + " before await");
                    fullCondition.await();
                    System.out.println(Thread.currentThread() + " after await");
                }
                int inc = (left + size) > capacity ? (capacity - size) : left;
                left -= inc;
                size += inc;
                System.out.println("produce = " + inc + ", size = " + size);
                emptyCondition.signal();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
    public void consume(int no) {
        lock.lock();
        int left = no;
        try {            
            while (left > 0) {
                while (size <= 0) {
                    System.out.println(Thread.currentThread() + " before await");
                    emptyCondition.await();
                    System.out.println(Thread.currentThread() + " after await");
                }
                int dec = (size - left) > 0 ? left : size;
                left -= dec;
                size -= dec;
                System.out.println("consume = " + dec + ", size = " + size);
                fullCondition.signal();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

- 测试类

```java
class Consumer {
    private Depot depot;
    public Consumer(Depot depot) {
        this.depot = depot;
    }
    
    public void consume(int no) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                depot.consume(no);
            }
        }, no + " consume thread").start();
    }
}

class Producer {
    private Depot depot;
    public Producer(Depot depot) {
        this.depot = depot;
    }
    
    public void produce(int no) {
        new Thread(new Runnable() {
            
            @Override
            public void run() {
                depot.produce(no);
            }
        }, no + " produce thread").start();
    }
}

public class ReentrantLockDemo {
    public static void main(String[] args) throws InterruptedException {
        Depot depot = new Depot(500);
        new Producer(depot).produce(500);
        new Producer(depot).produce(200);
        new Consumer(depot).consume(500);
        new Consumer(depot).consume(200);
    }
}
```

- 运行结果(可能的一种):

```java
produce = 500, size = 500
Thread[200 produce thread,5,main] before await
consume = 500, size = 0
Thread[200 consume thread,5,main] before await
Thread[200 produce thread,5,main] after await
produce = 200, size = 200
Thread[200 consume thread,5,main] after await
consume = 200, size = 0
```

说明: 根据结果，我们猜测一种可能的时序如下

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/29b53c42-a3fa-41c7-8ffe-01f7e21573a7_java-thread-x-juc-aqs-10.png)

说明: p1代表produce 500的那个线程，p2代表produce 200的那个线程，c1代表consume 500的那个线程，c2代表consume 200的那个线程。

- p1线程调用lock.lock，获得锁，继续运行，方法调用顺序在前面已经给出。
- p2线程调用lock.lock，由前面的分析可得到如下的最终状态。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/3346afa6-bd3c-4805-8214-9773f5df0e26_java-thread-x-juc-aqs-11.png)

说明: p2线程调用lock.lock后，会禁止p2线程的继续运行，因为执行了LockSupport.park操作。

- c1线程调用lock.lock，由前面的分析得到如下的最终状态。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/93864ba9-7a88-42e5-81d0-8f6bc37fed17_java-thread-x-juc-aqs-12.png)

说明: 最终c1线程会在sync queue队列的尾部，并且其结点的前驱结点(包含p2的结点)的waitStatus变为了SIGNAL。

- c2线程调用lock.lock，由前面的分析得到如下的最终状态。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/76409559-ab01-4f53-8c6b-0536e8fcf522_java-thread-x-juc-aqs-13.png)

说明: 最终c1线程会在sync queue队列的尾部，并且其结点的前驱结点(包含c1的结点)的waitStatus变为了SIGNAL。

- p1线程执行emptyCondition.signal，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/8f562a04-9991-417a-8222-b0465b54625a_java-thread-x-juc-aqs-14.png)

说明: AQS.CO表示AbstractQueuedSynchronizer.ConditionObject类。此时调用signal方法不会产生任何其他效果。

- p1线程执行lock.unlock，根据前面的分析可知，最终的状态如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/88c06cd8-b169-46b5-83d2-3f2ae2dda0cb_java-thread-x-juc-aqs-15.png)

说明: 此时，p2线程所在的结点为头节点，并且其他两个线程(c1、c2)依旧被禁止，所以，此时p2线程继续运行，执行用户逻辑。

- p2线程执行fullCondition.await，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/124a29c8-8e8a-48b6-808a-38c0378df27a_java-thread-x-juc-aqs-17-1.png)

说明: 最终到达的状态是新生成了一个结点，包含了p2线程，此结点在condition queue中；并且sync queue中p2线程被禁止了，因为在执行了LockSupport.park操作。从方法一些调用可知，在await操作中线程会释放锁资源，供其他线程获取。同时，head结点后继结点的包含的线程的许可被释放了，故其可以继续运行。由于此时，只有c1线程可以运行，故运行c1。

- 继续运行c1线程，c1线程由于之前被park了，所以此时恢复，继续之前的步骤，即还是执行前面提到的acquireQueued方法，之后，c1判断自己的前驱结点为head，并且可以获取锁资源，最终到达的状态如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/01f11bcb-f2e2-4304-89bc-7f2a632b1967_java-thread-x-juc-aqs-16.png)

说明: 其中，head设置为包含c1线程的结点，c1继续运行。

- c1线程执行fullCondtion.signal，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/5e653f63-006c-46ba-8071-ff8bc86bb404_java-thread-x-juc-aqs-17.png)

说明: signal方法达到的最终结果是将包含p2线程的结点从condition queue中转移到sync queue中，之后condition queue为null，之前的尾结点的状态变为SIGNAL。

- c1线程执行lock.unlock操作，根据之前的分析，经历的状态变化如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/82834112-8a24-406e-844d-963140041ff6_java-thread-x-juc-aqs-18.png)

说明: 最终c2线程会获取锁资源，继续运行用户逻辑。

- c2线程执行emptyCondition.await，由前面的第七步分析，可知最终的状态如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/382b5f16-f35d-4448-8b93-aab9024b68ae_java-thread-x-juc-aqs-19.png)

说明: await操作将会生成一个结点放入condition queue中与之前的一个condition queue是不相同的，并且unpark头节点后面的结点，即包含线程p2的结点。

- p2线程被unpark，故可以继续运行，经过CPU调度后，p2继续运行，之后p2线程在AQS:await方法中被park，继续AQS.CO:await方法的运行，其方法调用顺序如下，只给出了主要的方法调用。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/7415206d-82e6-45fb-852c-318a8281291a_java-thread-x-juc-aqs-20.png)

- p2继续运行，执行emptyCondition.signal，根据第九步分析可知，最终到达的状态如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/fbfc7844-ccb6-432a-8cfd-1d0a422cbcf9_java-thread-x-juc-aqs-21.png)

说明: 最终，将condition queue中的结点转移到sync queue中，并添加至尾部，condition queue会为空，并且将head的状态设置为SIGNAL。

- p2线程执行lock.unlock操作，根据前面的分析可知，最后的到达的状态如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/e37719bb-8eab-4f50-8117-dee290251867_java-thread-x-juc-aqs-22.png)

说明: unlock操作会释放c2线程的许可，并且将头节点设置为c2线程所在的结点。

- c2线程继续运行，执行fullCondition. signal，由于此时fullCondition的condition queue已经不存在任何结点了，故其不会产生作用。
- c2执行lock.unlock，由于c2是sync队列中最后一个结点，故其不会再调用unparkSuccessor了，直接返回true。即整个流程就完成了。

### AbstractQueuedSynchronizer总结

对于AbstractQueuedSynchronizer的分析，最核心的就是sync queue的分析。

- 每一个结点都是由前一个结点唤醒
- 当结点发现前驱结点是head并且尝试获取成功，则会轮到该线程运行。
- condition queue中的结点向sync queue中转移是通过signal操作完成的。
- 当结点的状态为SIGNAL时，表示后面的结点需要运行。



### 课后问题

- 什么是AQS? 为什么它是核心?
- AQS的核心思想是什么? 它是怎么实现的? 底层数据结构等
- AQS有哪些核心的方法?
- AQS定义什么样的资源获取方式? AQS定义了两种资源获取方式：`独占`(只有一个线程能访问执行，又根据是否按队列的顺序分为`公平锁`和`非公平锁`，如`ReentrantLock`) 和`共享`(多个线程可同时访问执行，如`Semaphore`、`CountDownLatch`、 `CyclicBarrier` )。`ReentrantReadWriteLock`可以看成是组合式，允许多个线程同时对某一资源进行读。
- AQS底层使用了什么样的设计模式? 模板
- AQS的应用示例?



### 参考文章

- 文章主要参考自leesf的https://www.cnblogs.com/leesf456/p/5350186.html，在此基础上做了增改。
- http://ifeve.com/introduce-abstractqueuedsynchronizer/
- http://blog.csdn.net/chen77716/article/details/6641477
- https://blog.csdn.net/mulinsen77/article/details/84583716



## JUC锁: ReentrantLock详解

---

### ReentrantLock源码分析

#### 类的继承关系

ReentrantLock实现了Lock接口，Lock接口中定义了lock与unlock相关操作，并且还存在newCondition方法，表示生成一个条件。

```java
public class ReentrantLock implements Lock, java.io.Serializable
```

#### 类的内部类

ReentrantLock总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/f1ee10af-89b7-4b19-8781-dcb956944233_java-thread-x-juc-reentrantlock-1.png)

说明: ReentrantLock类内部总共存在Sync、NonfairSync、FairSync三个类，NonfairSync与FairSync类继承自Sync类，Sync类继承自AbstractQueuedSynchronizer抽象类。下面逐个进行分析。

- Sync类

Sync类的源码如下:

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 序列号
    private static final long serialVersionUID = -5179523762034025860L;
    
    // 获取锁
    abstract void lock();
    
    // 非公平方式获取
    final boolean nonfairTryAcquire(int acquires) {
        // 当前线程
        final Thread current = Thread.currentThread();
        // 获取状态
        int c = getState();
        if (c == 0) { // 表示没有线程正在竞争该锁
            if (compareAndSetState(0, acquires)) { // 比较并设置状态成功，状态0表示锁没有被占用
                // 设置当前线程独占
                setExclusiveOwnerThread(current); 
                return true; // 成功
            }
        }
        else if (current == getExclusiveOwnerThread()) { // 当前线程拥有该锁
            int nextc = c + acquires; // 增加重入次数
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            // 设置状态
            setState(nextc); 
            // 成功
            return true; 
        }
        // 失败
        return false;
    }
    
    // 试图在共享模式下获取对象状态，此方法应该查询是否允许它在共享模式下获取对象状态，如果允许，则获取它
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread()) // 当前线程不为独占线程
            throw new IllegalMonitorStateException(); // 抛出异常
        // 释放标识
        boolean free = false; 
        if (c == 0) {
            free = true;
            // 已经释放，清空独占
            setExclusiveOwnerThread(null); 
        }
        // 设置标识
        setState(c); 
        return free; 
    }
    
    // 判断资源是否被当前线程占有
    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    // 新生一个条件
    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    // Methods relayed from outer class
    // 返回资源的占用线程
    final Thread getOwner() {        
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }
    // 返回状态
    final int getHoldCount() {            
        return isHeldExclusively() ? getState() : 0;
    }

    // 资源是否被占用
    final boolean isLocked() {        
        return getState() != 0;
    }

    /**
        * Reconstitutes the instance from a stream (that is, deserializes it).
        */
    // 自定义反序列化逻辑
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        setState(0); // reset to unlocked state
    }
}
```

Sync类存在如下方法和作用如下。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/91242cc8-73e7-4753-8288-552ac66afccd_java-thread-x-juc-reentrantlock-2.png)

- NonfairSync类

NonfairSync类继承了Sync类，表示采用非公平策略获取锁，其实现了Sync类中抽象的lock方法，源码如下:

```java
// 非公平锁
static final class NonfairSync extends Sync {
    // 版本号
    private static final long serialVersionUID = 7316153563782823691L;

    // 获得锁
    final void lock() {
        if (compareAndSetState(0, 1)) // 比较并设置状态成功，状态0表示锁没有被占用
            // 把当前线程设置独占了锁
            setExclusiveOwnerThread(Thread.currentThread());
        else // 锁已经被占用，或者set失败
            // 以独占模式获取对象，忽略中断
            acquire(1); 
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

说明: 从lock方法的源码可知，每一次都尝试获取锁，而并不会按照公平等待的原则进行等待，让等待时间最久的线程获得锁。

- FairSyn类

FairSync类也继承了Sync类，表示采用公平策略获取锁，其实现了Sync类中的抽象lock方法，源码如下:

```java
// 公平锁
static final class FairSync extends Sync {
    // 版本序列化
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        // 以独占模式获取对象，忽略中断
        acquire(1);
    }

    /**
        * Fair version of tryAcquire.  Don't grant access unless
        * recursive call or no waiters or is first.
        */
    // 尝试公平获取锁
    protected final boolean tryAcquire(int acquires) {
        // 获取当前线程
        final Thread current = Thread.currentThread();
        // 获取状态
        int c = getState();
        if (c == 0) { // 状态为0
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) { // 不存在已经等待更久的线程并且比较并且设置状态成功
                // 设置当前线程独占
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) { // 状态不为0，即资源已经被线程占据
            // 下一个状态
            int nextc = c + acquires;
            if (nextc < 0) // 超过了int的表示范围
                throw new Error("Maximum lock count exceeded");
            // 设置状态
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

说明: 跟踪lock方法的源码可知，当资源空闲时，它总是会先判断sync队列(AbstractQueuedSynchronizer中的数据结构)是否有等待时间更长的线程，如果存在，则将该线程加入到等待队列的尾部，实现了公平获取原则。其中，FairSync类的lock的方法调用如下，只给出了主要的方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/0a83038e-28c9-438a-8608-d42895e493d9_java-thread-x-juc-reentrantlock-3.png)

说明: 可以看出只要资源被其他线程占用，该线程就会添加到sync queue中的尾部，而不会先尝试获取资源。这也是和Nonfair最大的区别，Nonfair每一次都会尝试去获取资源，如果此时该资源恰好被释放，则会被当前线程获取，这就造成了不公平的现象，当获取不成功，再加入队列尾部。

#### 类的属性

ReentrantLock类的sync非常重要，对ReentrantLock类的操作大部分都直接转化为对Sync和AbstractQueuedSynchronizer类的操作。

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    // 序列号
    private static final long serialVersionUID = 7373984872572414699L;    
    // 同步队列
    private final Sync sync;
}  
```

#### 类的构造函数

- ReentrantLock()型构造函数

默认是采用的非公平策略获取锁

```java
public ReentrantLock() {
    // 默认非公平策略
    sync = new NonfairSync();
}
```

- ReentrantLock(boolean)型构造函数

可以传递参数确定采用公平策略或者是非公平策略，参数为true表示公平策略，否则，采用非公平策略:

```java
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

#### 核心函数分析

通过分析ReentrantLock的源码，可知对其操作都转化为对Sync对象的操作，由于Sync继承了AQS，所以基本上都可以转化为对AQS的操作。如将ReentrantLock的lock函数转化为对Sync的lock函数的调用，而具体会根据采用的策略(如公平策略或者非公平策略)的不同而调用到Sync的不同子类。

所以可知，在ReentrantLock的背后，是AQS对其服务提供了支持，由于之前我们分析AQS的核心源码，遂不再累赘。下面还是通过例子来更进一步分析源码。

### 示例分析

#### 公平锁

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class MyThread extends Thread {
    private Lock lock;
    public MyThread(String name, Lock lock) {
        super(name);
        this.lock = lock;
    }
    
    public void run () {
        lock.lock();
        try {
            System.out.println(Thread.currentThread() + " running");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } finally {
            lock.unlock();
        }
    }
}

public class AbstractQueuedSynchonizerDemo {
    public static void main(String[] args) throws InterruptedException {
        Lock lock = new ReentrantLock(true);
        
        MyThread t1 = new MyThread("t1", lock);        
        MyThread t2 = new MyThread("t2", lock);
        MyThread t3 = new MyThread("t3", lock);
        t1.start();
        t2.start();    
        t3.start();
    }
}
```

运行结果(某一次):

```html
Thread[t1,5,main] running
Thread[t2,5,main] running
Thread[t3,5,main] running
```

说明: 该示例使用的是公平策略，由结果可知，可能会存在如下一种时序。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/ba08c9d0-e5c0-42d3-800d-048b37055c89_java-thread-x-juc-reentrantlock-4.png)

说明: 首先，t1线程的lock操作 -> t2线程的lock操作 -> t3线程的lock操作 -> t1线程的unlock操作 -> t2线程的unlock操作 -> t3线程的unlock操作。根据这个时序图来进一步分析源码的工作流程。

- t1线程执行lock.lock，下图给出了方法调用中的主要方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/e967e78e-177b-4e55-85ce-3fe9f4a2699b_java-thread-x-juc-reentrantlock-5.png)

说明: 由调用流程可知，t1线程成功获取了资源，可以继续执行。

- t2线程执行lock.lock，下图给出了方法调用中的主要方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/3f2d5976-1bc4-40c2-8cef-d00bb37f404d_java-thread-x-juc-reentrantlock-6.png)

说明: 由上图可知，最后的结果是t2线程会被禁止，因为调用了LockSupport.park。

- t3线程执行lock.lock，下图给出了方法调用中的主要方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/c176d532-5020-4d27-86b2-62c97eb1394d_java-thread-x-juc-reentrantlock-7.png)

说明: 由上图可知，最后的结果是t3线程会被禁止，因为调用了LockSupport.park。

- t1线程调用了lock.unlock，下图给出了方法调用中的主要方法。

  ![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/f33fe876-c96b-4ad5-82eb-bf5da98ec093_java-thread-x-juc-reentrantlock-8.png)

说明: 如上图所示，最后，head的状态会变为0，t2线程会被unpark，即t2线程可以继续运行。此时t3线程还是被禁止。

- t2获得cpu资源，继续运行，由于t2之前被park了，现在需要恢复之前的状态，下图给出了方法调用中的主要方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/10355dcd-281c-41f3-82f3-0223b9d11242_java-thread-x-juc-reentrantlock-9.png)

说明: 在setHead函数中会将head设置为之前head的下一个结点，并且将pre域与thread域都设置为null，在acquireQueued返回之前，sync queue就只有两个结点了。

- t2执行lock.unlock，下图给出了方法调用中的主要方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/2e0cb837-8fb2-4564-8756-6c10a07b93bb_java-thread-x-juc-reentrantlock-10.png)

说明: 由上图可知，最终unpark t3线程，让t3线程可以继续运行。

- t3线程获取cpu资源，恢复之前的状态，继续运行。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/e050aacc-a45f-4fe3-8c7a-1bec2a7b75b2_java-thread-x-juc-reentrantlock-11.png)

说明: 最终达到的状态是sync queue中只剩下了一个结点，并且该节点除了状态为0外，其余均为null。

- t3执行lock.unlock，下图给出了方法调用中的主要方法。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/569318c7-e6ae-4c06-8cc8-475fdb67ffae_java-thread-x-juc-reentrantlock-12.png)

说明: 最后的状态和之前的状态是一样的，队列中有一个空节点，头节点为尾节点均指向它。

使用公平策略和Condition的情况可以参考上一篇关于AQS的源码示例分析部分，不再累赘。



### 课后问题

- 什么是可重入，什么是可重入锁? 它用来解决什么问题?
- ReentrantLock的核心是AQS，那么它怎么来实现的，继承吗? 说说其类内部结构关系。
- ReentrantLock是如何实现公平锁的?
- ReentrantLock是如何实现非公平锁的?
- ReentrantLock默认实现的是公平还是非公平锁?
- 使用ReentrantLock实现公平和非公平锁的示例?
- ReentrantLock和Synchronized的对比?



### 参考文章

- 文章主要参考自leesf的https://www.cnblogs.com/leesf456/p/5383609.html，在此基础上做了增改。



## JUC锁: ReentrantReadWriteLock详解

---

### ReentrantReadWriteLock数据结构

ReentrantReadWriteLock底层是基于ReentrantLock和AbstractQueuedSynchronizer来实现的，所以，ReentrantReadWriteLock的数据结构也依托于AQS的数据结构。

### ReentrantReadWriteLock源码分析

#### 类的继承关系

```java
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {}
```

说明: 可以看到，ReentrantReadWriteLock实现了ReadWriteLock接口，ReadWriteLock接口定义了获取读锁和写锁的规范，具体需要实现类去实现；同时其还实现了Serializable接口，表示可以进行序列化，在源代码中可以看到ReentrantReadWriteLock实现了自己的序列化逻辑。

#### 类的内部类

ReentrantReadWriteLock有五个内部类，五个内部类之间也是相互关联的。内部类的关系如下图所示。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/70aa3f84-a1e2-4937-86a6-84a92428cd08_java-thread-x-readwritelock-1.png)

说明: 如上图所示，Sync继承自AQS、NonfairSync继承自Sync类、FairSync继承自Sync类；ReadLock实现了Lock接口、WriteLock也实现了Lock接口。

#### 内部类 - Sync类

- 类的继承关系

```java
abstract static class Sync extends AbstractQueuedSynchronizer {}
```

说明: Sync抽象类继承自AQS抽象类，Sync类提供了对ReentrantReadWriteLock的支持。

- 类的内部类

Sync类内部存在两个内部类，分别为HoldCounter和ThreadLocalHoldCounter，其中HoldCounter主要与读锁配套使用，其中，HoldCounter源码如下。

```java
// 计数器
static final class HoldCounter {
    // 计数
    int count = 0;
    // Use id, not reference, to avoid garbage retention
    // 获取当前线程的TID属性的值
    final long tid = getThreadId(Thread.currentThread());
}
```

说明: HoldCounter主要有两个属性，count和tid，其中count表示某个读线程重入的次数，tid表示该线程的tid字段的值，该字段可以用来唯一标识一个线程。ThreadLocalHoldCounter的源码如下

```java
// 本地线程计数器
static final class ThreadLocalHoldCounter
    extends ThreadLocal<HoldCounter> {
    // 重写初始化方法，在没有进行set的情况下，获取的都是该HoldCounter值
    public HoldCounter initialValue() {
        return new HoldCounter();
    }
}
```

说明: ThreadLocalHoldCounter重写了ThreadLocal的initialValue方法，ThreadLocal类可以将线程与对象相关联。在没有进行set的情况下，get到的均是initialValue方法里面生成的那个HolderCounter对象。

- 类的属性

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 版本序列号
    private static final long serialVersionUID = 6317671515068378041L;        
    // 高16位为读锁，低16位为写锁
    static final int SHARED_SHIFT   = 16;
    // 读锁单位
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    // 读锁最大数量
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    // 写锁最大数量
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
    // 本地线程计数器
    private transient ThreadLocalHoldCounter readHolds;
    // 缓存的计数器
    private transient HoldCounter cachedHoldCounter;
    // 第一个读线程
    private transient Thread firstReader = null;
    // 第一个读线程的计数
    private transient int firstReaderHoldCount;
}
```

说明: 该属性中包括了读锁、写锁线程的最大量。本地线程计数器等。

- 类的构造函数

```java
// 构造函数
Sync() {
    // 本地线程计数器
    readHolds = new ThreadLocalHoldCounter();
    // 设置AQS的状态
    setState(getState()); // ensures visibility of readHolds
}
```

说明: 在Sync的构造函数中设置了本地线程计数器和AQS的状态state。

#### 内部类 - Sync核心函数分析

对ReentrantReadWriteLock对象的操作绝大多数都转发至Sync对象进行处理。下面对Sync类中的重点函数进行分析

- sharedCount函数

表示占有读锁的线程数量，源码如下

```java
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
```

说明: 直接将state右移16位，就可以得到读锁的线程数量，因为state的高16位表示读锁，对应的低十六位表示写锁数量。

- exclusiveCount函数

表示占有写锁的线程数量，源码如下

```java
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

说明: 直接将状态state和(2^16 - 1)做与运算，其等效于将state模上2^16。写锁数量由state的低十六位表示。

- tryRelease函数

```java
/*
* Note that tryRelease and tryAcquire can be called by
* Conditions. So it is possible that their arguments contain
* both read and write holds that are all released during a
* condition wait and re-established in tryAcquire.
*/

protected final boolean tryRelease(int releases) {
    // 判断是否伪独占线程
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 计算释放资源后的写锁的数量
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0; // 是否释放成功
    if (free)
        setExclusiveOwnerThread(null); // 设置独占线程为空
    setState(nextc); // 设置状态
    return free;
}
```

说明: 此函数用于释放写锁资源，首先会判断该线程是否为独占线程，若不为独占线程，则抛出异常，否则，计算释放资源后的写锁的数量，若为0，表示成功释放，资源不将被占用，否则，表示资源还被占用。其函数流程图如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/8d78560b-10a3-4234-8423-9aac459deb61_java-thread-x-readwritelock-2.png)

- tryAcquire函数

```java
protected final boolean tryAcquire(int acquires) {
    /*
        * Walkthrough:
        * 1. If read count nonzero or write count nonzero
        *    and owner is a different thread, fail.
        * 2. If count would saturate, fail. (This can only
        *    happen if count is already nonzero.)
        * 3. Otherwise, this thread is eligible for lock if
        *    it is either a reentrant acquire or
        *    queue policy allows it. If so, update state
        *    and set owner.
        */
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取状态
    int c = getState();
    // 写线程数量
    int w = exclusiveCount(c);
    if (c != 0) { // 状态不为0
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread()) // 写线程数量为0或者当前线程没有占有独占资源
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT) // 判断是否超过最高写线程数量
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        // 设置AQS状态
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires)) // 写线程是否应该被阻塞
        return false;
    // 设置独占线程
    setExclusiveOwnerThread(current);
    return true;
}
```

说明: 此函数用于获取写锁，首先会获取state，判断是否为0，若为0，表示此时没有读锁线程，再判断写线程是否应该被阻塞，而在非公平策略下总是不会被阻塞，在公平策略下会进行判断(判断同步队列中是否有等待时间更长的线程，若存在，则需要被阻塞，否则，无需阻塞)，之后在设置状态state，然后返回true。若state不为0，则表示此时存在读锁或写锁线程，若写锁线程数量为0或者当前线程为独占锁线程，则返回false，表示不成功，否则，判断写锁线程的重入次数是否大于了最大值，若是，则抛出异常，否则，设置状态state，返回true，表示成功。其函数流程图如下

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/d6f6c825-e32e-45d8-8719-c5109d1e51c3_java-thread-x-readwritelock-3.png)

- tryReleaseShared函数

```java
protected final boolean tryReleaseShared(int unused) {
    // 获取当前线程
    Thread current = Thread.currentThread();
    if (firstReader == current) { // 当前线程为第一个读线程
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1) // 读线程占用的资源数为1
            firstReader = null;
        else // 减少占用的资源
            firstReaderHoldCount--;
    } else { // 当前线程不为第一个读线程
        // 获取缓存的计数器
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
            // 获取当前线程对应的计数器
            rh = readHolds.get();
        // 获取计数
        int count = rh.count;
        if (count <= 1) { // 计数小于等于1
            // 移除
            readHolds.remove();
            if (count <= 0) // 计数小于等于0，抛出异常
                throw unmatchedUnlockException();
        }
        // 减少计数
        --rh.count;
    }
    for (;;) { // 无限循环
        // 获取状态
        int c = getState();
        // 获取状态
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc)) // 比较并进行设置
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

说明: 此函数表示读锁线程释放锁。首先判断当前线程是否为第一个读线程firstReader，若是，则判断第一个读线程占有的资源数firstReaderHoldCount是否为1，若是，则设置第一个读线程firstReader为空，否则，将第一个读线程占有的资源数firstReaderHoldCount减1；若当前线程不是第一个读线程，那么首先会获取缓存计数器(上一个读锁线程对应的计数器 )，若计数器为空或者tid不等于当前线程的tid值，则获取当前线程的计数器，如果计数器的计数count小于等于1，则移除当前线程对应的计数器，如果计数器的计数count小于等于0，则抛出异常，之后再减少计数即可。无论何种情况，都会进入无限循环，该循环可以确保成功设置状态state。其流程图如下

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/5dfe038c-f93f-41ab-88f1-1aaa9fdc9b17_java-thread-x-readwritelock-4.png)

- tryAcquireShared函数

```java
private IllegalMonitorStateException unmatchedUnlockException() {
    return new IllegalMonitorStateException(
        "attempt to unlock read lock, not locked by current thread");
}

// 共享模式下获取资源
protected final int tryAcquireShared(int unused) {
    /*
        * Walkthrough:
        * 1. If write lock held by another thread, fail.
        * 2. Otherwise, this thread is eligible for
        *    lock wrt state, so ask if it should block
        *    because of queue policy. If not, try
        *    to grant by CASing state and updating count.
        *    Note that step does not check for reentrant
        *    acquires, which is postponed to full version
        *    to avoid having to check hold count in
        *    the more typical non-reentrant case.
        * 3. If step 2 fails either because thread
        *    apparently not eligible or CAS fails or count
        *    saturated, chain to version with full retry loop.
        */
    // 获取当前线程
    Thread current = Thread.currentThread();
    // 获取状态
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current) // 写线程数不为0并且占有资源的不是当前线程
        return -1;
    // 读锁数量
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) { // 读线程是否应该被阻塞、并且小于最大值、并且比较设置成功
        if (r == 0) { // 读锁数量为0
            // 设置第一个读线程
            firstReader = current;
            // 读线程占用的资源数为1
            firstReaderHoldCount = 1;
        } else if (firstReader == current) { // 当前线程为第一个读线程
            // 占用资源数加1
            firstReaderHoldCount++;
        } else { // 读锁数量不为0并且不为当前线程
            // 获取计数器
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current)) // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
                // 获取当前线程对应的计数器
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0) // 计数为0
                // 设置
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

说明: 此函数表示读锁线程获取读锁。首先判断写锁是否为0并且当前线程不占有独占锁，直接返回；否则，判断读线程是否需要被阻塞并且读锁数量是否小于最大值并且比较设置状态成功，若当前没有读锁，则设置第一个读线程firstReader和firstReaderHoldCount；若当前线程线程为第一个读线程，则增加firstReaderHoldCount；否则，将设置当前线程对应的HoldCounter对象的值。流程图如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/05a0b63e-cc01-45d3-8dde-01628323c481_java-thread-x-readwritelock-5.png)

- fullTryAcquireShared函数

```java
final int fullTryAcquireShared(Thread current) {
    /*
        * This code is in part redundant with that in
        * tryAcquireShared but is simpler overall by not
        * complicating tryAcquireShared with interactions between
        * retries and lazily reading hold counts.
        */
    HoldCounter rh = null;
    for (;;) { // 无限循环
        // 获取状态
        int c = getState();
        if (exclusiveCount(c) != 0) { // 写线程数量不为0
            if (getExclusiveOwnerThread() != current) // 不为当前线程
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) { // 写线程数量为0并且读线程被阻塞
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) { // 当前线程为第一个读线程
                // assert firstReaderHoldCount > 0;
            } else { // 当前线程不为第一个读线程
                if (rh == null) { // 计数器不为空
                    // 
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) { // 计数器为空或者计数器的tid不为当前正在运行的线程的tid
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT) // 读锁数量为最大值，抛出异常
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) { // 比较并且设置成功
            if (sharedCount(c) == 0) { // 读线程数量为0
                // 设置第一个读线程
                firstReader = current;
                // 
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

说明: 在tryAcquireShared函数中，如果下列三个条件不满足(读线程是否应该被阻塞、小于最大值、比较设置成功)则会进行fullTryAcquireShared函数中，它用来保证相关操作可以成功。其逻辑与tryAcquireShared逻辑类似，不再累赘。

而其他内部类的操作基本上都是转化到了对Sync对象的操作，在此不再累赘。

#### 类的属性

```java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    // 版本序列号    
    private static final long serialVersionUID = -6992448646407690164L;    
    // 读锁
    private final ReentrantReadWriteLock.ReadLock readerLock;
    // 写锁
    private final ReentrantReadWriteLock.WriteLock writerLock;
    // 同步队列
    final Sync sync;
    
    private static final sun.misc.Unsafe UNSAFE;
    // 线程ID的偏移地址
    private static final long TID_OFFSET;
    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> tk = Thread.class;
            // 获取线程的tid字段的内存地址
            TID_OFFSET = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("tid"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

说明: 可以看到ReentrantReadWriteLock属性包括了一个ReentrantReadWriteLock.ReadLock对象，表示读锁；一个ReentrantReadWriteLock.WriteLock对象，表示写锁；一个Sync对象，表示同步队列。

#### 类的构造函数

- ReentrantReadWriteLock()型构造函数

```java
public ReentrantReadWriteLock() {
    this(false);
}   
```

说明: 此构造函数会调用另外一个有参构造函数。

- ReentrantReadWriteLock(boolean)型构造函数

```java
public ReentrantReadWriteLock(boolean fair) {
    // 公平策略或者是非公平策略
    sync = fair ? new FairSync() : new NonfairSync();
    // 读锁
    readerLock = new ReadLock(this);
    // 写锁
    writerLock = new WriteLock(this);
}
```

说明: 可以指定设置公平策略或者非公平策略，并且该构造函数中生成了读锁与写锁两个对象。

#### 核心函数分析

对ReentrantReadWriteLock的操作基本上都转化为了对Sync对象的操作，而Sync的函数已经分析过，不再累赘。

### ReentrantReadWriteLock示例

下面给出了一个使用ReentrantReadWriteLock的示例，源代码如下。

```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

class ReadThread extends Thread {
    private ReentrantReadWriteLock rrwLock;
    
    public ReadThread(String name, ReentrantReadWriteLock rrwLock) {
        super(name);
        this.rrwLock = rrwLock;
    }
    
    public void run() {
        System.out.println(Thread.currentThread().getName() + " trying to lock");
        try {
            rrwLock.readLock().lock();
            System.out.println(Thread.currentThread().getName() + " lock successfully");
            Thread.sleep(5000);        
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rrwLock.readLock().unlock();
            System.out.println(Thread.currentThread().getName() + " unlock successfully");
        }
    }
}

class WriteThread extends Thread {
    private ReentrantReadWriteLock rrwLock;
    
    public WriteThread(String name, ReentrantReadWriteLock rrwLock) {
        super(name);
        this.rrwLock = rrwLock;
    }
    
    public void run() {
        System.out.println(Thread.currentThread().getName() + " trying to lock");
        try {
            rrwLock.writeLock().lock();
            System.out.println(Thread.currentThread().getName() + " lock successfully");    
        } finally {
            rrwLock.writeLock().unlock();
            System.out.println(Thread.currentThread().getName() + " unlock successfully");
        }
    }
}

public class ReentrantReadWriteLockDemo {
    public static void main(String[] args) {
        ReentrantReadWriteLock rrwLock = new ReentrantReadWriteLock();
        ReadThread rt1 = new ReadThread("rt1", rrwLock);
        ReadThread rt2 = new ReadThread("rt2", rrwLock);
        WriteThread wt1 = new WriteThread("wt1", rrwLock);
        rt1.start();
        rt2.start();
        wt1.start();
    } 
}
```

运行结果(某一次):

```html
rt1 trying to lock
rt2 trying to lock
wt1 trying to lock
rt1 lock successfully
rt2 lock successfully
rt1 unlock successfully
rt2 unlock successfully
wt1 lock successfully
wt1 unlock successfully
```

说明: 程序中生成了一个ReentrantReadWriteLock对象，并且设置了两个读线程，一个写线程。根据结果，可能存在如下的时序图。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/39c45cd2-6c29-49eb-8c03-44cfb6e1c241_java-thread-x-readwritelock-6.png)

- rt1线程执行rrwLock.readLock().lock操作，主要的函数调用如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/988485b3-bd8e-47e7-88a6-5315daab5648_java-thread-x-readwritelock-7.png)

说明: 此时，AQS的状态state为2^16 次方，即表示此时读线程数量为1。

- rt2线程执行rrwLock.readLock().lock操作，主要的函数调用如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/610c9f04-8e1b-45b3-8c62-72ef177ed3b4_java-thread-x-readwritelock-8.png)

说明: 此时，AQS的状态state为2 * 2^16次方，即表示此时读线程数量为2。

- wt1线程执行rrwLock.writeLock().lock操作，主要的函数调用如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/93a5c2c0-cfac-4429-84eb-b45afa876cd9_java-thread-x-readwritelock-9.png)

说明: 此时，在同步队列Sync queue中存在两个结点，并且wt1线程会被禁止运行。

- rt1线程执行rrwLock.readLock().unlock操作，主要的函数调用如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/06dff244-803d-4721-8adc-e6dd18127ee6_java-thread-x-readwritelock-10.png)

说明: 此时，AQS的state为2^16次方，表示还有一个读线程。

- rt2线程执行rrwLock.readLock().unlock操作，主要的函数调用如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/4d0b26d3-dc7a-4ee2-8483-928bb92dceac_java-thread-x-readwritelock-11.png)

说明: 当rt2线程执行unlock操作后，AQS的state为0，并且wt1线程将会被unpark，其获得CPU资源就可以运行。

- wt1线程获得CPU资源，继续运行，需要恢复。由于之前acquireQueued函数中的parkAndCheckInterrupt函数中被禁止的，所以，恢复到parkAndCheckInterrupt函数中，主要的函数调用如下

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/c18a58ca-fed3-418e-8f31-b1c2e7f07742_java-thread-x-readwritelock-12.png)

说明: 最后，sync queue队列中只有一个结点，并且头节点尾节点均指向它，AQS的state值为1，表示此时有一个写线程。

- wt1执行rrwLock.writeLock().unlock操作，主要的函数调用如下。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/63a88387-b0d3-4709-8916-95535ed52f27_java-thread-x-readwritelock-13.png)

说明: 此时，AQS的state为0，表示没有任何读线程或者写线程了。并且Sync queue结构与上一个状态的结构相同，没有变化。

### 更深入理解

#### 什么是锁升降级?

锁降级指的是写锁降级成为读锁。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住(当前拥有的)写锁，再获取到读锁，随后释放(先前拥有的)写锁的过程。

接下来看一个锁降级的示例。因为数据不常变化，所以多个线程可以并发地进行数据处理，当数据变更后，如果当前线程感知到数据变化，则进行数据的准备工作，同时其他处理线程被阻塞，直到当前线程完成数据的准备工作，如代码如下所示：

```java
public void processData() {
    readLock.lock();
    if (!update) {
        // 必须先释放读锁
        readLock.unlock();
        // 锁降级从写锁获取到开始
        writeLock.lock();
        try {
            if (!update) {
                // 准备数据的流程(略)
                update = true;
            }
            readLock.lock();
        } finally {
            writeLock.unlock();
        }
        // 锁降级完成，写锁降级为读锁
    }
    try {
        // 使用数据的流程(略)
    } finally {
        readLock.unlock();
    }
}
```

上述示例中，当数据发生变更后，update变量(布尔类型且volatile修饰)被设置为false，此时所有访问processData()方法的线程都能够感知到变化，但只有一个线程能够获取到写锁，其他线程会被阻塞在读锁和写锁的lock()方法上。当前线程获取写锁完成数据准备之后，再获取读锁，随后释放写锁，完成锁降级。

锁降级中读锁的获取是否必要呢? 答案是必要的。主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程(记作线程T)获取了写锁并修改了数据，那么当前线程无法感知线程T的数据更新。如果当前线程获取读锁，即遵循锁降级的步骤，则线程T将会被阻塞，直到当前线程使用数据并释放读锁之后，线程T才能获取写锁进行数据更新。

RentrantReadWriteLock不支持锁升级(把持读锁、获取写锁，最后释放读锁的过程)。目的也是保证数据可见性，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。



### 课后问题

- 为了有了ReentrantLock还需要ReentrantReadWriteLock?
- ReentrantReadWriteLock底层实现原理?
- ReentrantReadWriteLock底层读写状态如何设计的? 高16位为读锁，低16位为写锁
- 读锁和写锁的最大数量是多少?
- 本地线程计数器ThreadLocalHoldCounter是用来做什么的?
- 缓存计数器HoldCounter是用来做什么的?
- 写锁的获取与释放是怎么实现的?
- 读锁的获取与释放是怎么实现的?
- RentrantReadWriteLock为什么不支持锁升级?
- 什么是锁的升降级? RentrantReadWriteLock为什么不支持锁升级?



### 参考文章

- 文章主要参考自leesf的https://www.cnblogs.com/leesf456/p/5419132.html，在此基础上做了增改。
- https://blog.csdn.net/jiankunking/article/details/83954263