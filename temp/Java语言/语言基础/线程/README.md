# 线程介绍

## 进程

- 进程是运行的程序。进程是程序的一次执行过程。操作系统会为进程分配内存空间；

- 动态过程：有自身的产生、存在和存亡的过程。

## 线程

- 线程是由进程创建的，是进程的一个实体；
- 一个进程可以拥有多个线程。

### 单线程

同一时刻，只允许执行一个线程。

### 多线程

同一时刻，可以允许执行多个线程。

### 并发

<p class="note note-primary">重要</p>

同一个时刻，多个任务交替执行。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2008.54.41.png" alt="截屏2021-09-16 08.54.41" style="zoom:50%;" />

### 并行

<p class="note note-primary">重要</p>

同一个时刻，多个任务同时执行。多核 CPU 可以实现并行。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2008.54.55.png" alt="截屏2021-09-16 08.54.55" style="zoom:50%;" />





# <u>线程使用</u>

<p class="note note-primary">重要</p>

> 创建线程的两种方法
>
> - 基础 Thread 类，重写 run 方法；
> - 实现 Runnable 接口，重写 run 方法。



**Diagram关系**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2009.08.04.png" alt="截屏2021-09-16 09.08.04" style="zoom:50%;" />



**案例1： 继承Thread**

1. 编程一个线程，该线程每隔1秒在控制台输出文字 "Hello"。

   ```java
   public class thread {
       public static void main(String[] args) {
           // 创建 Cat 对象当作线程使用
           Cat cat = new Cat();
           // 启动线程
           cat.start();
   
       }
   }
   
   
   /**
    *  @Author: tsuiraku
    *  @Description:
    *  1. 当一个类继承了 Thread 类， 该类就可以当做线程使用
    *  2. 重写 run 方法，写上自己的业务代码
    *  3. run Thread 类 实现了 Runnable 接口的 run 方法
    */
   class Cat extends Thread{
       @Override
       public void run() { // 重写 run 方法，写自己的逻辑业务
           // 调用 start 方法，会执行一次 run 方法
           while(true){
               System.out.println("Hello");
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       }
   }
   ```

2. 对上题进行改进，当输出100次 "Hello" 后，停止输出。

   ```java
   public class thread {
       public static void main(String[] args) {
           // 创建 Cat 对象当作线程使用
           Cat cat = new Cat();
           // 启动线程
           cat.start();
   
       }
   }
   
   
   /**
    *  @Author: tsuiraku
    *  @Description:
    *  1. 当一个类继承了 Thread 类， 该类就可以当做线程使用
    *  2. 重写 run 方法，写上自己的业务代码
    *  3. run Thread 类 实现了 Runnable 接口的 run 方法
    */
   class Cat extends Thread{
       private int time = 0;
       @Override
       public void run() { // 重写 run 方法，写自己的逻辑业务
           // 调用 start 方法，会执行一次 run 方法
           while(true){
               System.out.println("Hello");
               time++;
               try {
                   Thread.sleep(1000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
               if(time == 100) {
                   break;  
               }
           }
       }
   }
   ```

3. 画出程序的执行图。

- 当主线程（main 线程）启动一个子线程 Thread-0，主线程不会阻塞，会继续执行，此时主线程和子线程交替进行；
- 主线程结束后，若子线程还在运行，整个进程并不会结束。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2009.37.42.png" alt="截屏2021-09-16 09.37.42" style="zoom:50%;" />



**案例2：实现Runnable**

> Java是单继承的；
>
> 通过实现 Runnable接口来创建线程。

编程一个线程，该线程每隔1秒在控制台输出文字 "Hello"。当输出10次后，自动退出。

```java
public class thread  {
    public static void main(String[] args) {
        Dog dog = new Dog();
        // 无法直接调用 start 方法
        // 创建 Thread 对象，将 dog 对象放入 Thread
        Thread thread = new Thread(dog);
        thread.start();;

    }
}

class Dog implements Runnable{
    int count = 0;
    @Override
    public void run() {
        while (true){
            System.out.println("Hello" + ++count + Thread.currentThread().getName());
            try{
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if(count == 10) {
                break;
            }
        }
    }
}
```



**案例3：多线程**

创建两个线程，一个线程每隔1秒输出 "Hello World"，10次后退出；另一个线程每隔1秒输出 "hi"，5次后退出。

```java
package com.tsuiraku.thread;

public class Thread03 {
    public static void main(String[] args) {
        Hello hello = new Hello();
        Hi hi = new Hi();
        Thread thread1 = new Thread(hello);
        Thread thread2 = new Thread(hi);
        thread1.start();
        thread2.start();
    }
}


class Hello implements Runnable {
    int count = 0;
    @Override
    public void run() {
        while (true) {
            System.out.println("Hello" + " " + (++count) + " " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (count == 10) {
                break;
            }
        }
    }
}

class Hi implements Runnable {
    int count = 0;
    @Override
    public void run() {
        while (true) {
            System.out.println("Hi" + " " + (++count) + " " + Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (count == 5) {
                break;
            }
        }
    }
}

```



**案例4：多线程模拟售票系统**

```java
package com.tsuiraku.thread.demo;

/**
 *  @Author: tsuiraku
 *  @Description:
 *
 */
public class SellTicket {
    public static void main(String[] args) {
//        SellTicket01 s1 = new SellTicket01();
//        SellTicket01 s2 = new SellTicket01();
//        SellTicket01 s3 = new SellTicket01();
//        s1.start();
//        s2.start();
//        s3.start();

        SellTicket02 s = new SellTicket02();

        Thread t1 = new Thread(s);
        Thread t2 = new Thread(s);
        Thread t3 = new Thread(s);

        t1.start();
        t2.start();
        t3.start();

    }
}

// 使用 Thread 方式

class SellTicket01 extends Thread {
    private static int ticketNum = 100;

    @Override
    public void run() {
        while (true) {
            if(ticketNum <= 0) {
                System.out.println("售票结束...");
                break;
            }
            // 休眠50ms
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("窗口" + Thread.currentThread().getName() + "售出了一张票"
            + ";剩余票数 = " + (--ticketNum));
        }
    }
}

// 使用 Runnable 方式

class SellTicket02 implements Runnable {
    private int ticketNum = 100;

    @Override
    public void run() {
        while (true) {
            if(ticketNum <= 0) {
                System.out.println("售票结束...");
                break;
            }
            // 休眠50ms
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("窗口" + Thread.currentThread().getName() + "售出了一张票"
                    + ";剩余票数 = " + (--ticketNum));
        }
    }
}
```

问题：出现 ticketNum 值为负数的情况。因为三个线程同时访问修改 tiketNum。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2010.51.12.png" alt="截屏2021-09-16 10.51.12" style="zoom:50%;" />

如何解决？[Synchronized](#Synchronized)



<u>**源码**</u>

`thread.start()`

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2009.48.18.png" alt="截屏2021-09-16 09.48.18" style="zoom:50%;" />

**<u>start0</u>**：`start0` 是一个 native 方法，由 JVM 调用。`start()` 方法调用 `start0()` 方法后，该线程并不一定马上执行，只是讲线程变成了可运行状态。具体执行时间，由 CPU 统一调度。



**思考**

- `thread.start()` 和 `thread.run()`；

  如果直接执行 `thread.run()` 方法，那么实际上是主线程在执行，并不会真正启动一个线程，并且程序会阻塞。而使用 `thread.start()` 方法，主线程会开启一个子线程，程序并不会阻塞，而是主线程和子线程交替进行。



### 线程终止

> 当线程完成任务后，自动退出；
>
> 通过使用变量来控制 run 方法退出的方式停止线程，即通知方式。

**案例**

启动一个线程 t，要求在 main 线程中停止线程 t。

```java
public class Thread {
    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        t.start();

        // 希望 main 线程控制 t 线程的终止，即修改 loop
        for(int i = 0; i < 10; i++) {
            Thread.sleep(1000);
        }
        
        t.setLoop(false);
    }
}

class T extends Thread {
    boolean loop = true;
    @Override
    public void run() {
        while (loop) {
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("T运行中...");;
        }
    }

    public void setLoop(boolean loop) {
        this.loop = loop;
    }
}
```

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2011.07.12.png" alt="截屏2021-09-16 11.07.12" style="zoom:50%;" />



# 线程方法

> setName；// 设置线程名称
>
> getName；// 获取线程名称
>
> start；// 执行线程，JVM调用底层 start0
>
> run；// 调用线程对象 run 方法
>
> setPriority；// 设置线程优先级
>
> getPriority；// 获取线程优先级
>
> sleep；// 线程休眠
>
> interrupt；// 线程中断，但并没有真正结束线程。一般用于中断正在休眠的线程。
>
> yield；// 线程的礼让。让出 CPU，让其他线程执行，但礼让的时间不确定，所以也不一定会成功。
>
> join。//线程的插队。插队的线程一旦插队成功，则肯定先执行完插入的线程的所有任务。



### 线程中断

实际上并没有让线程停止运行。一般用于唤醒正在休眠的线程。



### 线程插队

- yield

线程主动礼让 CPU。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2013.48.42.png" alt="截屏2021-09-16 13.48.42" style="zoom:50%;" />

- join

主动放弃 CPU，让其他线程插队。

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2015.36.56.png" alt="截屏2021-09-16 15.36.56" style="zoom:50%;" />

```java
public class ThreadMethodExercise {
  public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(new T());
    // 创建子线程 
    for (int i = 1; i <= 10; i++) {
      System.out.println("hi " + i);
      if(i == 5) { // 说明主线程输出了 5 次 hi
        t.start(); // 启动子线程 输出 hello...
        t.join(); // 立即将 t 子线程，插入到 main 线程，让 t 先执行 
      }
        Thread.sleep(1000); //输出一次 hi, 让 main 线程也休眠 1s 
    }
  } 
}
  
class T implements Runnable { 
  private int count = 0;
	@Override
  public void run() {
    while (true) {
      System.out.println("hello " + (++count)); 
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace(); }
      if (count == 10) { 
        break;
       } 
    }
	} 
}
```



<u>**源码**</u>

优先级

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-16%2011.18.02.png" alt="截屏2021-09-16 11.18.02" style="zoom:50%;" />



### 用户线程

也称为工作线程。当线程的任务执行完或者通知方式结束。



### 守护线程

一般是为工作线程服务的，当所有的用户线程结束后，守护线程自动结束。常见的守护线程：垃圾回收机制。

如何将一个线程设置为守护线程。

```java
public class Thread {
    public static void main(String[] args) throws InterruptedException {
        T t = new T();
        t.setDaemon(true); // 设置守护线程
        t.start();

        // 希望当 main 线程结束后，希望子线程 t 自动终止
      	// 将子线程设置为守护线程
        for(int i = 0; i < 10; i++) {
            Thread.sleep(1000);
        }
    }
}

class T extends Thread {
    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("T 运行中...");;
        }
    }
}
```





# 线程生命周期

<p class="note note-primary">重要</p>

### <u>七状态转化</u>

> **新建状态（New）**
>
> 当用 new 操作符创建一个线程后， 此时线程处在新建状态。 当一个线程处于新建状态时，线程中的任务代码还没开始运行。
>
> **就绪状态（Runnable）**
>
> 也被称为“可执行状态”。一个新创建的线程并不自动开始运行，要执行线程，必须调用线程的 `start()` 方法。当调用了线程对象的 `start()` 方法即启动了线程，此时线程就处于就绪状态。
>
> 处于就绪状态的线程并不一定立即运行 `run()` 方法，线程还必须同其他就绪线程竞争 CPU，只有获得 CPU 使用权才可以运行线程。比如在单核心 CPU 的计算机系统中，不可能同时运行多个线程，一个时刻只能有一个线程处于运行状态。对与多个处于就绪状态的线程是由 Java 运行时系统的线程调度程序（thread scheduler）来调度执行。
>
> 除了调用 `start()` 方法后让线程变成就绪状态，一个线程阻塞状态结束后也可以变成就绪状态，或者从运行状态变化到就绪状态。
>
> **运行状态（Running）**
>
> 线程获取到 CPU 使用权进行执行。需要注意的是，线程只能从就绪状态进入到运行状态。真正开始执行 `run()` 方法的内容。
>
> **阻塞状态（Blocked）**
>
> 线程在获取锁失败时（因为锁被其它线程抢占），它会被加入锁的同步阻塞队列，然后线程进入阻塞状态（Blocked）。处于阻塞状态（Blocked）的线程放弃 CPU 使用权，暂时停止运行。待其它线程释放锁之后，阻塞状态（Blocked）的线程将在次参与锁的竞争，如果竞争锁成功，线程将进入就绪状态（Runnable）。
>
> ```
> # 进入阻塞状态
> 等待进入同步代码块的锁
> 
> # 离开
> 活的锁
> ```
>
> **等待状态（WAITING）**
>
> 或者叫条件等待状态，当线程的运行条件不满足时，通过锁的条件等待机制（调用锁对象的 `wait()` 或显示锁条件对象的 `await()` 方法）让线程进入等待状态（WAITING）。处于等待状态的线程将不会被 CPU 执行，除非线程的运行条件得到满足后，其可被其他线程唤醒，进入阻塞状态（Blocked）。调用不带超时的 Thread.join()方法也会进入等待状态。
>
> ```
> # 进入等待状态
> o.wait(time)
> t.join()
> LockSupport.park()
> 
> # 离开
> o.notify()
> o.notifyAll()
> LockSupport.unpark()
> ```
>
> **限时等待状态（TIMED_WAITING）**
>
> 限时等待是等待状态的一种特例，线程在等待时我们将设定等待超时时间，如超过了我们设定的等待时间，等待线程将自动唤醒进入阻塞状态（Blocked）或就绪状态（Runnable)）。在调用 `Thread.sleep()` 方法，带有超时设定的 `Object.wait()` 方法，带有超时设定的 `Thread.join()` 方法等，线程会进入限时等待状态（TIMED_WAITING）。
>
> ```
> # 进入限时等待状态
> Thread.sleep(time)
> o.wait(time)
> t.join(time)
> LockSupport.parkNanos()
> LockSupport.parkUntil()
> 
> # 离开
> 时间结束
> ```
>
> **死亡状态（TERMINATED）**
>
> 线程执行完了或者因异常退出了 `run() 方法，该线程结束生命周期。



![img](https://gitee.com/tsuiraku/typora/raw/master/img/1539204-20190625104746568-573625770.png)



# <u>Synchronized</u>

<p class="note note-primary">重要</p>

> 线程同步机制，同一时刻，最多有一个线程访问某个内存地址，直到该线程完成操作，其他线程才能对这个内存地址进行操作，以保证数据的完整性。



> **<u>Synchronized</u>**
>
> 1. 同步代码块
>
>    ```java
>    synchronized(对象) { // 得到对象的锁，才能操作同步代码
>      // 需要被同步的代码
>    }
>    ```
>
> 2. 放在方法上
>
>    ```java
>    public synchronized void m (String name) {
>      // 需要被同步的代码
>    }
>    ```



解决：售票问题的同步问题

- 代码块加锁

  ```java
  /**
   *  @Author: tsuiraku
   *
   */
  public class SellTicket {
      public static void main(String[] args) {
  
          SellTicket03 s = new SellTicket03();
  
          Thread t1 = new Thread(s);
          Thread t2 = new Thread(s);
          Thread t3 = new Thread(s);
  
          t1.start();
          t2.start();
          t3.start();
  
      }
  }
  
  class SellTicket03 implements Runnable {
      private int ticketNum = 100;
    	private boolean loop = true;
  
      @Override
      public void run() { 
        while(loop) {
          sell();
  			}
      }
    
      // 代码块上加锁
    	// 锁加在 this 对象
      public void sell() { // 同步方法，同一时刻，只能有一个线程来执行 sell 方法
        synchronized(this){
          if(ticketNum <= 0) {
            System.out.println("售票结束...");
            loop = false;
            return;
          }
          // 休眠50ms
          try {
            Thread.sleep(50);
          } catch (InterruptedException e) {
            e.printStackTrace();
          }
          System.out.println("窗口" + Thread.currentThread().getName() + "售出了一张票"
                             + ";剩余票数 = " + (--ticketNum));
        	}
        }
  }
  ```

- 方法上加锁

  ```java
  /**
   *  @Author: tsuiraku
   *
   */
  public class SellTicket {
      public static void main(String[] args) {
  
          SellTicket03 s = new SellTicket03();
  
          Thread t1 = new Thread(s);
          Thread t2 = new Thread(s);
          Thread t3 = new Thread(s);
  
          t1.start();
          t2.start();
          t3.start();
  
      }
  }
  
  class SellTicket03 implements Runnable {
      private int ticketNum = 100;
    	private boolean loop = true;
  
      @Override
      public void run() { 
        while(loop) {
          sell();
  			}
      }
    
    	// 方法上加锁
    	// public synchronized void sell() 同步方法
    	// 锁加在 this 对象
      public synchronized void sell() { // 同步方法，同一时刻，只能有一个线程来执行 sell 方法
        if(ticketNum <= 0) {
          System.out.println("售票结束...");
          loop = false;
          return;
        }
        // 休眠50ms
        try {
          Thread.sleep(50);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println("窗口" + Thread.currentThread().getName() + "售出了一张票"
                           + ";剩余票数 = " + (--ticketNum));
  	  }
  }
  ```

  

# <u>互斥锁</u>

<p class="note note-primary">重要</p>

> 作用是保证共享数据的完整性；
>
> 每个对象都对应于一个可称为 “互斥锁” 的标记，这个标记用于保证在同一时刻，只能有一个线程访问该对象；
>
> 关键字 synchronized 与对象互斥锁联系，当某个对象用 synchronized 修饰时，表面该对象任一时刻都只能由一个线程访问；
>
> 同步的局限性：导致程序的执行效率要降低；
>
> 同步方法（非静态）的锁可以是 this，也可以是其他对象（要求是同一个对象）；
>
> 同步方法（静态的）的锁为当前类本身，当前类.class。



# 死锁

模拟死锁的程序

```java
public class DeadLock_ {
  public static void main(String[] args) {
		//模拟死锁现象
    DeadLockDemo A = new DeadLockDemo(true); A.setName("A 线程");
    DeadLockDemo B = new DeadLockDemo(false); B.setName("B 线程");
    A.start();
    B.start();
  } 
}

class DeadLockDemo extends Thread {
  static Object o1 = new Object(); // 保证多线程，共享一个对象,这里使用 static 
  static Object o2 = new Object();
  boolean flag;
  public DeadLockDemo(boolean flag) { // 构造器 
    this.flag = flag;
}
  @Override
  public void run() {
    // 下面业务逻辑的分析
    
    // 如果 flag 为 T, 线程 A 就会先持有 o1 对象锁, 然后尝试去获取 o2 对象锁 
    // 如果线程 A 得不到 o2 对象锁，就会 Blocked
    
    // 如果 flag 为 F, 线程 B 就会先持有 o2 对象锁, 然后尝试去获取 o1 对象锁 
    // 如果线程 B 得不到 o1 对象锁，就会 Blocked
    if (flag) {
      synchronized (o1) {//对象互斥锁, 下面就是同步代
        System.out.println(Thread.currentThread().getName() + " 进入 1"); 
        synchronized (o2) {
          // 这里获得 li 对象的监视权
          System.out.println(Thread.currentThread().getName() + " 进入 2"); }
      } 
    } else {
      synchronized (o2) { 
        System.out.println(Thread.currentThread().getName() + " 进入 3"); 
        synchronized (o1) { // 这里获得 li 对象的监视权
          System.out.println(Thread.currentThread().getName() + " 进入 4"); 
        }
      } 
    }
  } 
}
```



### 释放锁

> 释放锁的操作
>
> 当前线程的同步方法、同步代码块执行结束；
>
> 当前线程在同步方法、同步代码块中遇到 break、return；
>
> 当前线程在同步方法、同步代码块中出现了未处理的 Error 或者 Exception；
>
> 当前线程在同步方法、同步代码块中执行了 `wait()` 方法，使得线程暂停，并释放锁。
>
> 
>
> 不会释放锁的操作
>
> 线程的同步方法、同步代码块执行 `Thread.sleep()`、`Thread.yield()`；
>
> 线程执行同步代码块，其他线程调用该线程的 `suspend()` 方法将该线程挂起。



### 产生原因

由于系统中存在一些不可剥夺资源，而当两个或两个以上进程占有自身资源，并请求对方资源时，会导致每个进程都无法向前推进，这就是死锁。 

> 1. 竞争资源 例如：系统中只有一台打印机，可供进程 A 使用，假定 A 已占用了打印机，若 B 继续要求打印机打印将被阻塞。 
>
>    系统中的资源可以分为两类：
>
>    - 可剥夺资源：是指某进程在获得这类资源后，该资源可以再被其他进程或系统剥夺，CPU 和主存均属于可剥夺性资源； 
>    - 不可剥夺资源，当系统把这类资源分配给某进程后，再不能强行收回，只能在进程用完后自行释放，如磁带机、打印机等。 
>
> 2. 进程推进顺序不当 
>
>    例如：进程 A 和 进程 B 互相等待对方的数据。



### 必要条件

> 1. 互斥条件：进程要求对所分配的资源进行排它性控制，即在一段时间内某资源仅为一进程所占用；
> 2. 请求和保持条件：当进程因请求资源而阻塞时，对已获得的资源保持不放；
> 3. 不剥夺条件：进程已获得的资源在未使用完之前，不能剥夺，只能在使用完时由自己释放；
> 4. 环路等待条件：在发生死锁时，必然存在一个进程–资源的环形链。





# 感谢

- [韩顺平-Java线程](https://www.bilibili.com/video/BV1zB4y1A7rb?p=17)

