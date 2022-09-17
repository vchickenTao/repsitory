**<span style="font-size: 35px;">🥠 Java并发 JUC线程池详解</span>**

---

## JUC线程池: FutureTask详解

---

### FutureTask简介

FutureTask 为 Future 提供了基础实现，如获取任务执行结果(get)和取消任务(cancel)等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用runAndReset执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由CAS来保证。

### FutureTask类关系

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/47c56480-de31-4473-8114-10a6404057af_java-thread-x-juc-futuretask-1 (1).png)

可以看到,FutureTask实现了RunnableFuture接口，则RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果。

### FutureTask源码解析

#### Callable接口

Callable是个泛型接口，泛型V就是要call()方法返回的类型。对比Runnable接口，Runnable不会返回数据也不能抛出异常。

```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

#### Future接口

Future接口代表异步计算的结果，通过Future接口提供的方法可以查看异步计算是否执行完成，或者等待执行结果并获取执行结果，同时还可以取消执行。Future接口的定义如下:

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

- `cancel()`:cancel()方法用来取消异步任务的执行。如果异步任务已经完成或者已经被取消，或者由于某些原因不能取消，则会返回false。如果任务还没有被执行，则会返回true并且异步任务不会被执行。如果任务已经开始执行了但是还没有执行完成，若mayInterruptIfRunning为true，则会立即中断执行任务的线程并返回true，若mayInterruptIfRunning为false，则会返回true且不会中断任务执行线程。
- `isCanceled()`:判断任务是否被取消，如果任务在结束(正常执行结束或者执行异常结束)前被取消则返回true，否则返回false。
- `isDone()`:判断任务是否已经完成，如果完成则返回true，否则返回false。需要注意的是：任务执行过程中发生异常、任务被取消也属于任务已完成，也会返回true。
- `get()`:获取任务执行结果，如果任务还没完成则会阻塞等待直到任务执行完成。如果任务被取消则会抛出CancellationException异常，如果任务执行过程发生异常则会抛出ExecutionException异常，如果阻塞等待过程中被中断则会抛出InterruptedException异常。
- `get(long timeout,Timeunit unit)`:带超时时间的get()版本，如果阻塞等待过程中超时则会抛出TimeoutException异常。

#### 核心属性

```java
//内部持有的callable任务，运行完毕后置空
private Callable<V> callable;

//从get()中返回的结果或抛出的异常
private Object outcome; // non-volatile, protected by state reads/writes

//运行callable的线程
private volatile Thread runner;

//使用Treiber栈保存等待线程
private volatile WaitNode waiters;

//任务状态
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;  
```

其中需要注意的是state是volatile类型的，也就是说只要有任何一个线程修改了这个变量，那么其他所有的线程都会知道最新的值。7种状态具体表示：

- `NEW`:表示是个新的任务或者还没被执行完的任务。这是初始状态。
- `COMPLETING`:任务已经执行完成或者执行任务的时候发生异常，但是任务执行结果或者异常原因还没有保存到outcome字段(outcome字段用来保存任务执行结果，如果发生异常，则用来保存异常原因)的时候，状态会从NEW变更到COMPLETING。但是这个状态会时间会比较短，属于中间状态。
- `NORMAL`:任务已经执行完成并且任务执行结果已经保存到outcome字段，状态会从COMPLETING转换到NORMAL。这是一个最终态。
- `EXCEPTIONAL`:任务执行发生异常并且异常原因已经保存到outcome字段中后，状态会从COMPLETING转换到EXCEPTIONAL。这是一个最终态。
- `CANCELLED`:任务还没开始执行或者已经开始执行但是还没有执行完成的时候，用户调用了cancel(false)方法取消任务且不中断任务执行线程，这个时候状态会从NEW转化为CANCELLED状态。这是一个最终态。
- `INTERRUPTING`: 任务还没开始执行或者已经执行但是还没有执行完成的时候，用户调用了cancel(true)方法取消任务并且要中断任务执行线程但是还没有中断任务执行线程之前，状态会从NEW转化为INTERRUPTING。这是一个中间状态。
- `INTERRUPTED`:调用interrupt()中断任务执行线程之后状态会从INTERRUPTING转换到INTERRUPTED。这是一个最终态。 有一点需要注意的是，所有值大于COMPLETING的状态都表示任务已经执行完成(任务正常执行完成，任务执行异常或者任务被取消)。

各个状态之间的可能转换关系如下图所示:

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/ebf3d56d-41d4-434f-866e-36bc57bb708a_java-thread-x-juc-futuretask-2.png)

#### 构造函数

- FutureTask(Callable<V> callable)

```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

这个构造函数会把传入的Callable变量保存在this.callable字段中，该字段定义为`private Callable<V> callable`;用来保存底层的调用，在被执行完成以后会指向null,接着会初始化state字段为NEW。

- FutureTask(Runnable runnable, V result)

```java
public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}
```

这个构造函数会把传入的Runnable封装成一个Callable对象保存在callable字段中，同时如果任务执行成功的话就会返回传入的result。这种情况下如果不需要返回值的话可以传入一个null。

顺带看下Executors.callable()这个方法，这个方法的功能是把Runnable转换成Callable，代码如下:

```java
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
       throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}
```

可以看到这里采用的是适配器模式，调用`RunnableAdapter<T>(task, result)`方法来适配，实现如下:

```java
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```

这个适配器很简单，就是简单的实现了Callable接口，在call()实现中调用Runnable.run()方法，然后把传入的result作为任务的结果返回。

在new了一个FutureTask对象之后，接下来就是在另一个线程中执行这个Task,无论是通过直接new一个Thread还是通过线程池，执行的都是run()方法，接下来就看看run()方法的实现。

#### 核心方法 - run()

```java
public void run() {
    //新建任务，CAS替换runner为当前线程
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);//设置执行结果
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);//处理中断逻辑
    }
} 
```

**说明：**

- 运行任务，如果任务状态为NEW状态，则利用CAS修改为当前线程。执行完毕调用set(result)方法设置执行结果。set(result)源码如下：

```java
protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();//执行完毕，唤醒等待线程
    }
} 
```

- 首先利用cas修改state状态为COMPLETING，设置返回结果，然后使用 lazySet(UNSAFE.putOrderedInt)的方式设置state状态为NORMAL。结果设置完毕后，调用finishCompletion()方法唤醒等待线程，源码如下：

```java
private void finishCompletion() {
    // assert state > COMPLETING;
    for (WaitNode q; (q = waiters) != null;) {
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {//移除等待线程
            for (;;) {//自旋遍历等待线程
                Thread t = q.thread;
                if (t != null) {
                    q.thread = null;
                    LockSupport.unpark(t);//唤醒等待线程
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }
    //任务完成后调用函数，自定义扩展
    done();

    callable = null;        // to reduce footprint
}
```

- 回到run方法，如果在 run 期间被中断，此时需要调用handlePossibleCancellationInterrupt方法来处理中断逻辑，确保任何中断(例如cancel(true))只停留在当前run或runAndReset的任务中，源码如下：

```java
private void handlePossibleCancellationInterrupt(int s) {
    //在中断者中断线程之前可能会延迟，所以我们只需要让出CPU时间片自旋等待
    if (s == INTERRUPTING)
        while (state == INTERRUPTING)
            Thread.yield(); // wait out pending interrupt
}
```

#### 核心方法 - get()

```java
//获取执行结果
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
}
```

说明：FutureTask 通过get()方法获取任务执行结果。如果任务处于未完成的状态(`state <= COMPLETING`)，就调用awaitDone方法(后面单独讲解)等待任务完成。任务完成后，通过report方法获取执行结果或抛出执行期间的异常。report源码如下：

```java
//返回执行结果或抛出异常
private V report(int s) throws ExecutionException {
    Object x = outcome;
    if (s == NORMAL)
        return (V)x;
    if (s >= CANCELLED)
        throw new CancellationException();
    throw new ExecutionException((Throwable)x);
}
```

#### 核心方法 - awaitDone(boolean timed, long nanos)

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {//自旋
        if (Thread.interrupted()) {//获取并清除中断状态
            removeWaiter(q);//移除等待WaitNode
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;//置空等待节点的线程
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            //CAS修改waiter
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);//超时，移除等待节点
                return state;
            }
            LockSupport.parkNanos(this, nanos);//阻塞当前线程
        }
        else
            LockSupport.park(this);//阻塞当前线程
    }
}
```

说明：awaitDone用于等待任务完成，或任务因为中断或超时而终止。返回任务的完成状态。函数执行逻辑如下：

如果线程被中断，首先清除中断状态，调用removeWaiter移除等待节点，然后抛出InterruptedException。removeWaiter源码如下：

```java
private void removeWaiter(WaitNode node) {
    if (node != null) {
        node.thread = null;//首先置空线程
        retry:
        for (;;) {          // restart on removeWaiter race
            //依次遍历查找
            for (WaitNode pred = null, q = waiters, s; q != null; q = s) {
                s = q.next;
                if (q.thread != null)
                    pred = q;
                else if (pred != null) {
                    pred.next = s;
                    if (pred.thread == null) // check for race
                        continue retry;
                }
                else if (!UNSAFE.compareAndSwapObject(this, waitersOffset,q, s)) //cas替换
                    continue retry;
            }
            break;
        }
    }
} 
```

- 如果当前状态为结束状态(state>COMPLETING),则根据需要置空等待节点的线程，并返回 Future 状态；
- 如果当前状态为正在完成(COMPLETING)，说明此时 Future 还不能做出超时动作，为任务让出CPU执行时间片；
- 如果state为NEW，先新建一个WaitNode，然后CAS修改当前waiters；
- 如果等待超时，则调用removeWaiter移除等待节点，返回任务状态；如果设置了超时时间但是尚未超时，则park阻塞当前线程；
- 其他情况直接阻塞当前线程。

#### 核心方法 - cancel(boolean mayInterruptIfRunning)

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    //如果当前Future状态为NEW，根据参数修改Future状态为INTERRUPTING或CANCELLED
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
        if (mayInterruptIfRunning) {//可以在运行时中断
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();//移除并唤醒所有等待线程
    }
    return true;
}
```

说明：尝试取消任务。如果任务已经完成或已经被取消，此操作会失败。

- 如果当前Future状态为NEW，根据参数修改Future状态为INTERRUPTING或CANCELLED。
- 如果当前状态不为NEW，则根据参数mayInterruptIfRunning决定是否在任务运行中也可以中断。中断操作完成后，调用finishCompletion移除并唤醒所有等待线程。

### FutureTask示例

**常用使用方式：**

- 第一种方式: Future + ExecutorService
- 第二种方式: FutureTask + ExecutorService
- 第三种方式: FutureTask + Thread

#### Future使用示例

```java
public class FutureDemo {
      public static void main(String[] args) {
          ExecutorService executorService = Executors.newCachedThreadPool();
          Future future = executorService.submit(new Callable<Object>() {
              @Override
              public Object call() throws Exception {
                  Long start = System.currentTimeMillis();
                  while (true) {
                      Long current = System.currentTimeMillis();
                     if ((current - start) > 1000) {
                         return 1;
                     }
                 }
             }
         });
  
         try {
             Integer result = (Integer)future.get();
             System.out.println(result);
         }catch (Exception e){
             e.printStackTrace();
         }
     }
}
```

#### FutureTask+Thread例子

```java
import java.util.concurrent.*;
 
public class CallDemo {
 
    public static void main(String[] args) throws ExecutionException, InterruptedException {
 
        /**
         * 第一种方式:Future + ExecutorService
         * Task task = new Task();
         * ExecutorService service = Executors.newCachedThreadPool();
         * Future<Integer> future = service.submit(task1);
         * service.shutdown();
         */
 
 
        /**
         * 第二种方式: FutureTask + ExecutorService
         * ExecutorService executor = Executors.newCachedThreadPool();
         * Task task = new Task();
         * FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
         * executor.submit(futureTask);
         * executor.shutdown();
         */
 
        /**
         * 第三种方式:FutureTask + Thread
         */
 
        // 2. 新建FutureTask,需要一个实现了Callable接口的类的实例作为构造函数参数
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Task());
        // 3. 新建Thread对象并启动
        Thread thread = new Thread(futureTask);
        thread.setName("Task thread");
        thread.start();
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
 
        // 4. 调用isDone()判断任务是否结束
        if(!futureTask.isDone()) {
            System.out.println("Task is not done");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        int result = 0;
        try {
            // 5. 调用get()方法获取任务结果,如果任务没有执行完成则阻塞等待
            result = futureTask.get();
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        System.out.println("result is " + result);
 
    }
 
    // 1. 继承Callable接口,实现call()方法,泛型参数为要返回的类型
    static class Task  implements Callable<Integer> {
 
        @Override
        public Integer call() throws Exception {
            System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
            int result = 0;
            for(int i = 0; i < 100;++i) {
                result += i;
            }
 
            Thread.sleep(3000);
            return result;
        }
    }
}
```



### 课后问题

- FutureTask用来解决什么问题的? 为什么会出现?
- FutureTask类结构关系怎么样的?
- FutureTask的线程安全是由什么保证的?
- FutureTask结果返回机制?
- FutureTask内部运行状态的转变?
- FutureTask通常会怎么用? 举例说明。



### 参考文章

- 本文主要参考了https://www.cnblogs.com/linghu-java/p/8991824.html以及https://www.jianshu.com/p/d61d7ffa6abc，在此基础上增改
- https://blog.csdn.net/xingzhong128/article/details/80553789



##  JUC线程池: ThreadPoolExecutor详解

---

### 为什么要有线程池

线程池能够对线程进行统一分配，调优和监控:

- 降低资源消耗(线程无限制地创建，然后使用完毕后销毁)
- 提高响应速度(无须创建线程)
- 提高线程的可管理性

### ThreadPoolExecutor例子

Java是如何实现和管理线程池的?

从JDK 5开始，把工作单元与执行机制分离开来，工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

- WorkerThread

```java
public class WorkerThread implements Runnable {
     
    private String command;
     
    public WorkerThread(String s){
        this.command=s;
    }
 
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
        processCommand();
        System.out.println(Thread.currentThread().getName()+" End.");
    }
 
    private void processCommand() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    @Override
    public String toString(){
        return this.command;
    }
}
```

- SimpleThreadPool

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class SimpleThreadPool {
 
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            Runnable worker = new WorkerThread("" + i);
            executor.execute(worker);
          }
        executor.shutdown(); // This will make the executor accept no new threads and finish all existing threads in the queue
        while (!executor.isTerminated()) { // Wait until all threads are finish,and also you can use "executor.awaitTermination();" to wait
        }
        System.out.println("Finished all threads");
    }

}
```

程序中我们创建了固定大小为五个工作线程的线程池。然后分配给线程池十个工作，因为线程池大小为五，它将启动五个工作线程先处理五个工作，其他的工作则处于等待状态，一旦有工作完成，空闲下来工作线程就会捡取等待队列里的其他工作进行执行。

这里是以上程序的输出。

```html
pool-1-thread-2 Start. Command = 1
pool-1-thread-4 Start. Command = 3
pool-1-thread-1 Start. Command = 0
pool-1-thread-3 Start. Command = 2
pool-1-thread-5 Start. Command = 4
pool-1-thread-4 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
pool-1-thread-3 End.
pool-1-thread-3 Start. Command = 8
pool-1-thread-2 End.
pool-1-thread-2 Start. Command = 9
pool-1-thread-1 Start. Command = 7
pool-1-thread-5 Start. Command = 6
pool-1-thread-4 Start. Command = 5
pool-1-thread-2 End.
pool-1-thread-4 End.
pool-1-thread-3 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
Finished all threads
```

输出表明线程池中至始至终只有五个名为 "pool-1-thread-1" 到 "pool-1-thread-5" 的五个线程，这五个线程不随着工作的完成而消亡，会一直存在，并负责执行分配给线程池的任务，直到线程池消亡。

Executors 类提供了使用了 ThreadPoolExecutor 的简单的 ExecutorService 实现，但是 ThreadPoolExecutor 提供的功能远不止于此。我们可以在创建 ThreadPoolExecutor 实例时指定活动线程的数量，我们也可以限制线程池的大小并且创建我们自己的 RejectedExecutionHandler 实现来处理不能适应工作队列的工作。

这里是我们自定义的 RejectedExecutionHandler 接口的实现。

- RejectedExecutionHandlerImpl.java

```java
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;
 
public class RejectedExecutionHandlerImpl implements RejectedExecutionHandler {
 
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println(r.toString() + " is rejected");
    }
}
```

ThreadPoolExecutor 提供了一些方法，我们可以使用这些方法来查询 executor 的当前状态，线程池大小，活动线程数量以及任务数量。因此我是用来一个监控线程在特定的时间间隔内打印 executor 信息。

- MyMonitorThread.java

```java
import java.util.concurrent.ThreadPoolExecutor;
 
public class MyMonitorThread implements Runnable
{
    private ThreadPoolExecutor executor;
     
    private int seconds;
     
    private boolean run=true;
 
    public MyMonitorThread(ThreadPoolExecutor executor, int delay)
    {
        this.executor = executor;
        this.seconds=delay;
    }
     
    public void shutdown(){
        this.run=false;
    }
 
    @Override
    public void run()
    {
        while(run){
                System.out.println(
                    String.format("[monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, isShutdown: %s, isTerminated: %s",
                        this.executor.getPoolSize(),
                        this.executor.getCorePoolSize(),
                        this.executor.getActiveCount(),
                        this.executor.getCompletedTaskCount(),
                        this.executor.getTaskCount(),
                        this.executor.isShutdown(),
                        this.executor.isTerminated()));
                try {
                    Thread.sleep(seconds*1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        }
             
    }
}
```

这里是使用 ThreadPoolExecutor 的线程池实现例子。

- WorkerPool.java

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
public class WorkerPool {
 
    public static void main(String args[]) throws InterruptedException{
        //RejectedExecutionHandler implementation
        RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();
        //Get the ThreadFactory implementation to use
        ThreadFactory threadFactory = Executors.defaultThreadFactory();
        //creating the ThreadPoolExecutor
        ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);
        //start the monitoring thread
        MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
        Thread monitorThread = new Thread(monitor);
        monitorThread.start();
        //submit work to the thread pool
        for(int i=0; i<10; i++){
            executorPool.execute(new WorkerThread("cmd"+i));
        }
         
        Thread.sleep(30000);
        //shut down the pool
        executorPool.shutdown();
        //shut down the monitor thread
        Thread.sleep(5000);
        monitor.shutdown();
         
    }
}
```

注意在初始化 ThreadPoolExecutor 时，我们保持初始池大小为 2，最大池大小为 4 而工作队列大小为 2。因此如果已经有四个正在执行的任务而此时分配来更多任务的话，工作队列将仅仅保留他们(新任务)中的两个，其他的将会被 RejectedExecutionHandlerImpl 处理。

上面程序的输出可以证实以上观点。

```html
pool-1-thread-1 Start. Command = cmd0
pool-1-thread-4 Start. Command = cmd5
cmd6 is rejected
pool-1-thread-3 Start. Command = cmd4
pool-1-thread-2 Start. Command = cmd1
cmd7 is rejected
cmd8 is rejected
cmd9 is rejected
[monitor] [0/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-4 End.
pool-1-thread-1 End.
pool-1-thread-2 End.
pool-1-thread-3 End.
pool-1-thread-1 Start. Command = cmd3
pool-1-thread-4 Start. Command = cmd2
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-1 End.
pool-1-thread-4 End.
[monitor] [4/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
```

注意 executor 的活动任务、完成任务以及所有完成任务，这些数量上的变化。我们可以调用 shutdown() 方法来结束所有提交的任务并终止线程池。

### ThreadPoolExecutor使用详解

其实java线程池的实现原理很简单，说白了就是一个线程集合workerSet和一个阻塞队列workQueue。当用户向线程池提交一个任务(也就是线程)时，线程池会先将任务放入workQueue中。workerSet中的线程会不断的从workQueue中获取线程然后执行。当workQueue中没有任务的时候，worker就会阻塞，直到队列中有任务了就取出来继续执行。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/fe4d11ed-e8ce-4426-80a2-3b8c96bfdd58_java-thread-x-executors-1.png)

#### Execute原理

当一个任务提交至线程池之后:

1. 线程池首先当前运行的线程数量是否少于corePoolSize。如果是，则创建一个新的工作线程来执行任务。如果都在执行任务，则进入2.
2. 判断BlockingQueue是否已经满了，倘若还没有满，则将线程放入BlockingQueue。否则进入3.
3. 如果创建一个新的工作线程将使当前运行的线程数量超过maximumPoolSize，则交给RejectedExecutionHandler来处理任务。

当ThreadPoolExecutor创建新线程时，通过CAS来更新线程池的状态ctl.

#### 参数

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler)
```

- `corePoolSize` 线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize, 即使有其他空闲线程能够执行新来的任务, 也会继续创建线程；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。
- `workQueue` 用来保存等待被执行的任务的阻塞队列. 在JDK中提供了如下阻塞队列:  具体可以参考[JUC 集合: BlockQueue详解]()
  - `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按FIFO排序任务；
  - `LinkedBlockingQueue`: 基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQueue；
  - `SynchronousQueue`: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue；
  - `PriorityBlockingQueue`: 具有优先级的无界阻塞队列；

`LinkedBlockingQueue`比`ArrayBlockingQueue`在插入删除节点性能方面更优，但是二者在`put()`, `take()`任务的时均需要加锁，`SynchronousQueue`使用无锁算法，根据节点的状态判断执行，而不需要用到锁，其核心是`Transfer.transfer()`.

- `maximumPoolSize` 线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue.
- `keepAliveTime` 线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用, 超过这个时间的空闲线程将被终止；
- `unit` keepAliveTime的单位
- `threadFactory` 创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为DefaultThreadFactory
- `handler` 线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略:
  - `AbortPolicy`: 直接抛出异常，默认策略；
  - `CallerRunsPolicy`: 用调用者所在的线程来执行任务；
  - `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；
  - `DiscardPolicy`: 直接丢弃任务；

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

#### 三种类型

##### newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
}
```

线程池的线程数量达corePoolSize后，即使线程池没有可执行任务时，也不会释放线程。

FixedThreadPool的工作队列为无界队列LinkedBlockingQueue(队列容量为Integer.MAX_VALUE), 这会导致以下问题:

- 线程池里的线程数量不超过corePoolSize,这导致了maximumPoolSize和keepAliveTime将会是个无用参数
- 由于使用了无界队列, 所以FixedThreadPool永远不会拒绝, 即饱和策略失效

##### newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行.

由于使用了无界队列, 所以SingleThreadPool永远不会拒绝, 即饱和策略失效

##### newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}
```

线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列； 和newFixedThreadPool创建的线程池不同，newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销； 执行过程与前两种稍微不同:

- 主线程调用SynchronousQueue的offer()方法放入task, 倘若此时线程池中有空闲的线程尝试读取 SynchronousQueue的task, 即调用了SynchronousQueue的poll(), 那么主线程将该task交给空闲线程. 否则执行(2)
- 当线程池为空或者没有空闲的线程, 则创建新的线程执行任务.
- 执行完任务的线程倘若在60s内仍空闲, 则会被终止. 因此长时间空闲的CachedThreadPool不会持有任何线程资源.

#### 关闭线程池

遍历线程池中的所有线程，然后逐个调用线程的interrupt方法来中断线程.

##### 关闭方式 - shutdown

将线程池里的线程状态设置成SHUTDOWN状态, 然后中断所有没有正在执行任务的线程.

##### 关闭方式 - shutdownNow

将线程池里的线程状态设置成STOP状态, 然后停止所有正在执行或暂停任务的线程. 只要调用这两个关闭方法中的任意一个, isShutDown() 返回true. 当所有任务都成功关闭了, isTerminated()返回true.

### ThreadPoolExecutor源码详解

#### 几个关键属性

```java
//这个属性是用来存放 当前运行的worker数量以及线程池状态的
//int是32位的，这里把int的高3位拿来充当线程池状态的标志位,后29位拿来充当当前运行worker的数量
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
//存放任务的阻塞队列
private final BlockingQueue<Runnable> workQueue;
//worker的集合,用set来存放
private final HashSet<Worker> workers = new HashSet<Worker>();
//历史达到的worker数最大值
private int largestPoolSize;
//当队列满了并且worker的数量达到maxSize的时候,执行具体的拒绝策略
private volatile RejectedExecutionHandler handler;
//超出coreSize的worker的生存时间
private volatile long keepAliveTime;
//常驻worker的数量
private volatile int corePoolSize;
//最大worker的数量,一般当workQueue满了才会用到这个参数
private volatile int maximumPoolSize;
```

#### 内部状态

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
private static int runStateOf(int c)     { return c & ~CAPACITY; }
private static int workerCountOf(int c)  { return c & CAPACITY; }
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

其中AtomicInteger变量ctl的功能非常强大: 利用低29位表示线程池中线程数，通过高3位表示线程池的运行状态:

- RUNNING: -1 << COUNT_BITS，即高3位为111，该状态的线程池会接收新任务，并处理阻塞队列中的任务；
- SHUTDOWN:  0 << COUNT_BITS，即高3位为000，该状态的线程池不会接收新任务，但会处理阻塞队列中的任务；
- STOP :  1 << COUNT_BITS，即高3位为001，该状态的线程不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在运行的任务；
- TIDYING :  2 << COUNT_BITS，即高3位为010, 所有的任务都已经终止；
- TERMINATED:  3 << COUNT_BITS，即高3位为011, terminated()方法已经执行完成

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/4aa8bcc6-d593-40c6-834b-8d48257c4da3_java-thread-x-executors-2.png)

#### 任务的执行

> execute –> addWorker –>runworker (getTask)

线程池的工作线程通过Woker类实现，在ReentrantLock锁的保证下，把Woker实例插入到HashSet后，并启动Woker中的线程。 从Woker类的构造方法实现可以发现: 线程工厂在创建线程thread时，将Woker实例本身this作为参数传入，当执行start方法启动线程thread时，本质是执行了Worker的runWorker方法。 firstTask执行完成之后，通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用cpu资源；

##### execute()方法

ThreadPoolExecutor.execute(task)实现了Executor.execute(task)

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {  
    //workerCountOf获取线程池的当前线程数；小于corePoolSize，执行addWorker创建新线程执行command任务
       if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // double check: c, recheck
    // 线程池处于RUNNING状态，把提交的任务成功放入阻塞队列中
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // recheck and if necessary 回滚到入队操作前，即倘若线程池shutdown状态，就remove(command)
        //如果线程池没有RUNNING，成功从阻塞队列中删除任务，执行reject方法处理任务
        if (! isRunning(recheck) && remove(command))
            reject(command);
        //线程池处于running状态，但是没有线程，则创建线程
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 往线程池中创建新的线程失败，则reject任务
    else if (!addWorker(command, false))
        reject(command);
}
```

- 为什么需要double check线程池的状态?

在多线程环境下，线程池的状态时刻在变化，而ctl.get()是非原子操作，很有可能刚获取了线程池状态后线程池状态就改变了。判断是否将command加入workque是线程池之前的状态。倘若没有double check，万一线程池处于非running状态(在多线程环境下很有可能发生)，那么command永远不会执行。

##### addWorker方法

从方法execute的实现可以看出: addWorker主要负责创建新的线程并执行任务 线程池创建新线程执行任务时，需要 获取全局锁:

```java
private final ReentrantLock mainLock = new ReentrantLock();
```

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    // CAS更新线程池数量
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            // 线程池重入锁
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();  // 线程启动，执行任务(Worker.thread(firstTask).start());
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

##### Worker类的runworker方法

```java
 private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
     Worker(Runnable firstTask) {
         setState(-1); // inhibit interrupts until runWorker
         this.firstTask = firstTask;
         this.thread = getThreadFactory().newThread(this); // 创建线程
     }
     /** Delegates main run loop to outer runWorker  */
     public void run() {
         runWorker(this);
     }
     // ...
 }
```

- 继承了AQS类，可以方便的实现工作线程的中止操作；
- 实现了Runnable接口，可以将自身作为一个任务在工作线程中执行；
- 当前提交的任务firstTask作为参数传入Worker的构造方法；

一些属性还有构造方法:

```java
//运行的线程,前面addWorker方法中就是直接通过启动这个线程来启动这个worker
final Thread thread;
//当一个worker刚创建的时候,就先尝试执行这个任务
Runnable firstTask;
//记录完成任务的数量
volatile long completedTasks;

Worker(Runnable firstTask) {
    setState(-1); // inhibit interrupts until runWorker
    this.firstTask = firstTask;
    //创建一个Thread,将自己设置给他,后面这个thread启动的时候,也就是执行worker的run方法
    this.thread = getThreadFactory().newThread(this);
}
```

runWorker方法是线程池的核心:

- 线程启动之后，通过unlock方法释放锁，设置AQS的state为0，表示运行可中断；
- Worker执行firstTask或从workQueue中获取任务:
  - 进行加锁操作，保证thread不被其他线程中断(除非线程池被中断)
  - 检查线程池状态，倘若线程池处于中断状态，当前线程将中断。
  - 执行beforeExecute
  - 执行任务的run方法
  - 执行afterExecute方法
  - 解锁操作

> 通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用cpu资源；

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 先执行firstTask，再从workerQueue中取task(getTask())

        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                    (Thread.interrupted() &&
                    runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

##### getTask方法

下面来看一下getTask()方法，这里面涉及到keepAliveTime的使用，从这个方法我们可以看出线程池是怎么让超过corePoolSize的那部分worker销毁的。

```java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

注意这里一段代码是keepAliveTime起作用的关键:

```java
boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
```

allowCoreThreadTimeOut为false，线程即使空闲也不会被销毁；倘若为ture，在keepAliveTime内仍空闲则会被销毁。

如果线程允许空闲等待而不被销毁timed == false，workQueue.take任务: 如果阻塞队列为空，当前线程会被挂起等待；当队列中有任务加入时，线程被唤醒，take方法返回任务，并执行；

如果线程不允许无休止空闲timed == true, workQueue.poll任务: 如果在keepAliveTime时间内，阻塞队列还是没有任务，则返回null；

#### 任务的提交

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/e98a476e-521f-48a6-8013-792fba0664ad_java-thread-x-executors-3.png)

1. submit任务，等待线程池execute
2. 执行FutureTask类的get方法时，会把主线程封装成WaitNode节点并保存在waiters链表中， 并阻塞等待运行结果；
3. FutureTask任务执行完成后，通过UNSAFE设置waiters相应的waitNode为null，并通过LockSupport类unpark方法唤醒主线程；

```java
public class Test{
    public static void main(String[] args) {

        ExecutorService es = Executors.newCachedThreadPool();
        Future<String> future = es.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "future result";
            }
        });
        try {
            String result = future.get();
            System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

在实际业务场景中，Future和Callable基本是成对出现的，Callable负责产生结果，Future负责获取结果。

1. Callable接口类似于Runnable，只是Runnable没有返回值。
2. Callable任务除了返回正常结果之外，如果发生异常，该异常也会被返回，即Future可以拿到异步执行任务各种结果；
3. Future.get方法会导致主线程阻塞，直到Callable任务执行完成；

##### submit方法

AbstractExecutorService.submit()实现了ExecutorService.submit() 可以获取执行完的返回值, 而ThreadPoolExecutor 是AbstractExecutorService.submit()的子类，所以submit方法也是ThreadPoolExecutor`的方法。

```java
// submit()在ExecutorService中的定义
<T> Future<T> submit(Callable<T> task);

<T> Future<T> submit(Runnable task, T result);

Future<?> submit(Runnable task);
```

```java
// submit方法在AbstractExecutorService中的实现
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    // 通过submit方法提交的Callable任务会被封装成了一个FutureTask对象。
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
```

通过submit方法提交的Callable任务会被封装成了一个FutureTask对象。通过Executor.execute方法提交FutureTask到线程池中等待被执行，最终执行的是FutureTask的run方法；

##### FutureTask对象

`public class FutureTask<V> implements RunnableFuture<V>` 可以将FutureTask提交至线程池中等待被执行(通过FutureTask的run方法来执行)

- 内部状态

```java
/* The run state of this task, initially NEW. 
    * ...
    * Possible state transitions:
    * NEW -> COMPLETING -> NORMAL
    * NEW -> COMPLETING -> EXCEPTIONAL
    * NEW -> CANCELLED
    * NEW -> INTERRUPTING -> INTERRUPTED
    */
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

内部状态的修改通过sun.misc.Unsafe修改

- get方法

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    return report(s);
} 
```

内部通过awaitDone方法对主线程进行阻塞，具体实现如下:

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }

        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        else if (q == null)
            q = new WaitNode();
        else if (!queued)
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,q.next = waiters, q);
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        else
            LockSupport.park(this);
    }
}
```

1. 如果主线程被中断，则抛出中断异常；
2. 判断FutureTask当前的state，如果大于COMPLETING，说明任务已经执行完成，则直接返回；
3. 如果当前state等于COMPLETING，说明任务已经执行完，这时主线程只需通过yield方法让出cpu资源，等待state变成NORMAL；
4. 通过WaitNode类封装当前线程，并通过UNSAFE添加到waiters链表；
5. 最终通过LockSupport的park或parkNanos挂起线程；

run方法

```java
public void run() {
    if (state != NEW || !UNSAFE.compareAndSwapObject(this, runnerOffset, null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

FutureTask.run方法是在线程池中被执行的，而非主线程

1. 通过执行Callable任务的call方法；
2. 如果call执行成功，则通过set方法保存结果；
3. 如果call执行有异常，则通过setException保存异常；

#### 任务的关闭

shutdown方法会将线程池的状态设置为SHUTDOWN,线程池进入这个状态后,就拒绝再接受任务,然后会将剩余的任务全部执行完

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //检查是否可以关闭线程
        checkShutdownAccess();
        //设置线程池状态
        advanceRunState(SHUTDOWN);
        //尝试中断worker
        interruptIdleWorkers();
            //预留方法,留给子类实现
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}

private void interruptIdleWorkers() {
    interruptIdleWorkers(false);
}

private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //遍历所有的worker
        for (Worker w : workers) {
            Thread t = w.thread;
            //先尝试调用w.tryLock(),如果获取到锁,就说明worker是空闲的,就可以直接中断它
            //注意的是,worker自己本身实现了AQS同步框架,然后实现的类似锁的功能
            //它实现的锁是不可重入的,所以如果worker在执行任务的时候,会先进行加锁,这里tryLock()就会返回false
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
} 
```

shutdownNow做的比较绝，它先将线程池状态设置为STOP，然后拒绝所有提交的任务。最后中断左右正在运行中的worker,然后清空任务队列。

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        //检测权限
        advanceRunState(STOP);
        //中断所有的worker
        interruptWorkers();
        //清空任务队列
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}

private void interruptWorkers() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //遍历所有worker，然后调用中断方法
        for (Worker w : workers)
            w.interruptIfStarted();
    } finally {
        mainLock.unlock();
    }
}
```

### 更深入理解

#### 为什么线程池不允许使用Executors去创建? 推荐方式是什么?

线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端：

- newFixedThreadPool和newSingleThreadExecutor:  主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
- newCachedThreadPool和newScheduledThreadPool:  主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

##### 推荐方式 1

首先引入：commons-lang3包

```java
ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
```

##### 推荐方式 2

首先引入：com.google.guava包

```java
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder().setNameFormat("demo-pool-%d").build();

//Common Thread Pool
ExecutorService pool = new ThreadPoolExecutor(5, 200, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

// excute
pool.execute(()-> System.out.println(Thread.currentThread().getName()));

 //gracefully shutdown
pool.shutdown();
```

##### 推荐方式 3

spring配置线程池方式：自定义线程工厂bean需要实现ThreadFactory，可参考该接口的其它默认实现类，使用方式直接注入bean调用execute(Runnable task)方法即可

```xml
    <bean id="userThreadPool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        <property name="corePoolSize" value="10" />
        <property name="maxPoolSize" value="100" />
        <property name="queueCapacity" value="2000" />

    <property name="threadFactory" value= threadFactory />
        <property name="rejectedExecutionHandler">
            <ref local="rejectedExecutionHandler" />
        </property>
    </bean>
    
    //in code
    userThreadPool.execute(thread);
```

#### 配置线程池需要考虑因素

从任务的优先级，任务的执行时间长短，任务的性质(CPU密集/ IO密集)，任务的依赖关系这四个角度来分析。并且近可能地使用有界的工作队列。

性质不同的任务可用使用不同规模的线程池分开处理:

- CPU密集型: 尽可能少的线程，Ncpu+1
- IO密集型: 尽可能多的线程, Ncpu*2，比如数据库连接池
- 混合型: CPU密集型的任务与IO密集型任务的执行时间差别较小，拆分为两个线程池；否则没有必要拆分。

#### 监控线程池的状态

可以使用ThreadPoolExecutor以下方法:

- `getTaskCount()` Returns the approximate total number of tasks that have ever been scheduled for execution.
- `getCompletedTaskCount()` Returns the approximate total number of tasks that have completed execution. 返回结果少于getTaskCount()。
- `getLargestPoolSize()` Returns the largest number of threads that have ever simultaneously been in the pool. 返回结果小于等于maximumPoolSize
- `getPoolSize()` Returns the current number of threads in the pool.
- `getActiveCount()` Returns the approximate number of threads that are actively executing tasks.



### 课后问题

- 为什么要有线程池?
- Java是实现和管理线程池有哪些方式?  请简单举例如何使用。
- 为什么很多公司不允许使用Executors去创建线程池? 那么推荐怎么使用呢?
- ThreadPoolExecutor有哪些核心的配置参数? 请简要说明
- ThreadPoolExecutor可以创建哪是哪三种线程池呢?
- 当队列满了并且worker的数量达到maxSize的时候，会怎么样?
- 说说ThreadPoolExecutor有哪些RejectedExecutionHandler策略? 默认是什么策略?
- 简要说下线程池的任务执行机制? execute –> addWorker –>runworker (getTask)
- 线程池中任务是如何提交的?
- 线程池中任务是如何关闭的?
- 在配置线程池的时候需要考虑哪些配置因素?
- 如何监控线程池的状态?



### 参考文章

- 《Java并发编程艺术》
- https://www.jianshu.com/p/87bff5cc8d8c
- https://blog.csdn.net/programmer_at/article/details/79799267
- https://blog.csdn.net/u013332124/article/details/79587436
- https://www.journaldev.com/1069/threadpoolexecutor-java-thread-pool-example-executorservice



## JUC线程池: ScheduledThreadPoolExecutor详解

---

### ScheduledThreadPoolExecutor简介

ScheduledThreadPoolExecutor继承自 ThreadPoolExecutor，为任务提供延迟或周期执行，属于线程池的一种。和 ThreadPoolExecutor 相比，它还具有以下几种特性:

- 使用专门的任务类型—ScheduledFutureTask 来执行周期任务，也可以接收不需要时间调度的任务(这些任务通过 ExecutorService 来执行)。
- 使用专门的存储队列—DelayedWorkQueue 来存储任务，DelayedWorkQueue 是无界延迟队列DelayQueue 的一种。相比ThreadPoolExecutor也简化了执行机制(delayedExecute方法，后面单独分析)。
- 支持可选的run-after-shutdown参数，在池被关闭(shutdown)之后支持可选的逻辑来决定是否继续运行周期或延迟任务。并且当任务(重新)提交操作与 shutdown 操作重叠时，复查逻辑也不相同。

### ScheduledThreadPoolExecutor数据结构

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/4e7ebf65-962e-4bb5-879e-8e15f3433288_java-thread-x-stpe-1.png)

ScheduledThreadPoolExecutor继承自 `ThreadPoolExecutor`:

- 详情请参考:  [JUC线程池: ThreadPoolExecutor详解]()

ScheduledThreadPoolExecutor 内部构造了两个内部类 `ScheduledFutureTask` 和 `DelayedWorkQueue`:

- `ScheduledFutureTask`: 继承了FutureTask，说明是一个异步运算任务；最上层分别实现了Runnable、Future、Delayed接口，说明它是一个可以延迟执行的异步运算任务。
- `DelayedWorkQueue`: 这是 ScheduledThreadPoolExecutor 为存储周期或延迟任务专门定义的一个延迟队列，继承了 AbstractQueue，为了契合 ThreadPoolExecutor 也实现了 BlockingQueue 接口。它内部只允许存储 RunnableScheduledFuture 类型的任务。与 DelayQueue 的不同之处就是它只允许存放 RunnableScheduledFuture 对象，并且自己实现了二叉堆(DelayQueue 是利用了 PriorityQueue 的二叉堆结构)。

### ScheduledThreadPoolExecutor源码解析

> 以下源码的解析是基于你已经理解了FutureTask。

#### 内部类ScheduledFutureTask

##### 属性

```java
//为相同延时任务提供的顺序编号
private final long sequenceNumber;

//任务可以执行的时间，纳秒级
private long time;

//重复任务的执行周期时间，纳秒级。
private final long period;

//重新入队的任务
RunnableScheduledFuture<V> outerTask = this;

//延迟队列的索引，以支持更快的取消操作
int heapIndex;
```

- `sequenceNumber`: 当两个任务有相同的延迟时间时，按照 FIFO 的顺序入队。sequenceNumber 就是为相同延时任务提供的顺序编号。
- `time`: 任务可以执行时的时间，纳秒级，通过triggerTime方法计算得出。
- `period`: 任务的执行周期时间，纳秒级。正数表示固定速率执行(为scheduleAtFixedRate提供服务)，负数表示固定延迟执行(为scheduleWithFixedDelay提供服务)，0表示不重复任务。
- `outerTask`: 重新入队的任务，通过reExecutePeriodic方法入队重新排序。

##### 核心方法run()

```java
public void run() {
    boolean periodic = isPeriodic();//是否为周期任务
    if (!canRunInCurrentRunState(periodic))//当前状态是否可以执行
        cancel(false);
    else if (!periodic)
        //不是周期任务，直接执行
        ScheduledFutureTask.super.run();
    else if (ScheduledFutureTask.super.runAndReset()) {
        setNextRunTime();//设置下一次运行时间
        reExecutePeriodic(outerTask);//重排序一个周期任务
    }
} 
```

说明: ScheduledFutureTask 的run方法重写了 FutureTask 的版本，以便执行周期任务时重置/重排序任务。任务的执行通过父类 FutureTask 的run实现。内部有两个针对周期任务的方法:

- setNextRunTime(): 用来设置下一次运行的时间，源码如下:

```java
//设置下一次执行任务的时间
private void setNextRunTime() {
    long p = period;
    if (p > 0)  //固定速率执行，scheduleAtFixedRate
        time += p;
    else
        time = triggerTime(-p);  //固定延迟执行，scheduleWithFixedDelay
}
//计算固定延迟任务的执行时间
long triggerTime(long delay) {
    return now() +
        ((delay < (Long.MAX_VALUE >> 1)) ? delay : overflowFree(delay));
}
```

- reExecutePeriodic(): 周期任务重新入队等待下一次执行，源码如下:

```java
//重排序一个周期任务
void reExecutePeriodic(RunnableScheduledFuture<?> task) {
    if (canRunInCurrentRunState(true)) {//池关闭后可继续执行
        super.getQueue().add(task);//任务入列
        //重新检查run-after-shutdown参数，如果不能继续运行就移除队列任务，并取消任务的执行
        if (!canRunInCurrentRunState(true) && remove(task))
            task.cancel(false);
        else
            ensurePrestart();//启动一个新的线程等待任务
    }
}
```

reExecutePeriodic与delayedExecute的执行策略一致，只不过reExecutePeriodic不会执行拒绝策略而是直接丢掉任务。

##### cancel方法

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    boolean cancelled = super.cancel(mayInterruptIfRunning);
    if (cancelled && removeOnCancel && heapIndex >= 0)
        remove(this);
    return cancelled;
}
```

ScheduledFutureTask.cancel本质上由其父类 FutureTask.cancel 实现。取消任务成功后会根据removeOnCancel参数决定是否从队列中移除此任务。

#### 核心属性

```java
//关闭后继续执行已经存在的周期任务 
private volatile boolean continueExistingPeriodicTasksAfterShutdown;

//关闭后继续执行已经存在的延时任务 
private volatile boolean executeExistingDelayedTasksAfterShutdown = true;

//取消任务后移除 
private volatile boolean removeOnCancel = false;

//为相同延时的任务提供的顺序编号，保证任务之间的FIFO顺序
private static final AtomicLong sequencer = new AtomicLong();
```

- `continueExistingPeriodicTasksAfterShutdown`和`executeExistingDelayedTasksAfterShutdown`是 ScheduledThreadPoolExecutor 定义的 `run-after-shutdown` 参数，用来控制池关闭之后的任务执行逻辑。
- `removeOnCancel`用来控制任务取消后是否从队列中移除。当一个已经提交的周期或延迟任务在运行之前被取消，那么它之后将不会运行。默认配置下，这种已经取消的任务在届期之前不会被移除。 通过这种机制，可以方便检查和监控线程池状态，但也可能导致已经取消的任务无限滞留。为了避免这种情况的发生，我们可以通过`setRemoveOnCancelPolicy`方法设置移除策略，把参数`removeOnCancel`设为true可以在任务取消后立即从队列中移除。
- `sequencer`是为相同延时的任务提供的顺序编号，保证任务之间的 FIFO 顺序。与 ScheduledFutureTask 内部的sequenceNumber参数作用一致。

#### 构造函数

首先看下构造函数，ScheduledThreadPoolExecutor 内部有四个构造函数，这里我们只看这个最大构造灵活度的:

```java
public ScheduledThreadPoolExecutor(int corePoolSize,
                                   ThreadFactory threadFactory,
                                   RejectedExecutionHandler handler) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue(), threadFactory, handler);
}
```

构造函数都是通过super调用了ThreadPoolExecutor的构造，并且使用特定等待队列DelayedWorkQueue。

#### 核心方法:Schedule

```java
public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                       long delay,
                                       TimeUnit unit) {
    if (callable == null || unit == null)
        throw new NullPointerException();
    RunnableScheduledFuture<V> t = decorateTask(callable,
        new ScheduledFutureTask<V>(callable, triggerTime(delay, unit)));//构造ScheduledFutureTask任务
    delayedExecute(t);//任务执行主方法
    return t;
}
```

说明: schedule主要用于执行一次性(延迟)任务。函数执行逻辑分两步:

- `封装 Callable/Runnable`: 首先通过triggerTime计算任务的延迟执行时间，然后通过 ScheduledFutureTask 的构造函数把 Runnable/Callable 任务构造为ScheduledThreadPoolExecutor可以执行的任务类型，最后调用decorateTask方法执行用户自定义的逻辑；decorateTask是一个用户可自定义扩展的方法，默认实现下直接返回封装的RunnableScheduledFuture任务，源码如下:

```java
protected <V> RunnableScheduledFuture<V> decorateTask(
    Runnable runnable, RunnableScheduledFuture<V> task) {
    return task;
}
```

- `执行任务`: 通过delayedExecute实现。下面我们来详细分析。

```java
private void delayedExecute(RunnableScheduledFuture<?> task) {
    if (isShutdown())
        reject(task);//池已关闭，执行拒绝策略
    else {
        super.getQueue().add(task);//任务入队
        if (isShutdown() &&
            !canRunInCurrentRunState(task.isPeriodic()) &&//判断run-after-shutdown参数
            remove(task))//移除任务
            task.cancel(false);
        else
            ensurePrestart();//启动一个新的线程等待任务
    }
}  
```

说明: delayedExecute是执行任务的主方法，方法执行逻辑如下:

- 如果池已关闭(ctl >= SHUTDOWN)，执行任务拒绝策略；
- 池正在运行，首先把任务入队排序；然后重新检查池的关闭状态，执行如下逻辑:

`A`: 如果池正在运行，或者 run-after-shutdown 参数值为true，则调用父类方法ensurePrestart启动一个新的线程等待执行任务。ensurePrestart源码如下:

```java
void ensurePrestart() {
    int wc = workerCountOf(ctl.get());
    if (wc < corePoolSize)
        addWorker(null, true);
    else if (wc == 0)
        addWorker(null, false);
}  
```

ensurePrestart是父类 ThreadPoolExecutor 的方法，用于启动一个新的工作线程等待执行任务，即使corePoolSize为0也会安排一个新线程。

`B`: 如果池已经关闭，并且 run-after-shutdown 参数值为false，则执行父类(ThreadPoolExecutor)方法remove移除队列中的指定任务，成功移除后调用ScheduledFutureTask.cancel取消任务

#### 核心方法:scheduleAtFixedRate 和 scheduleWithFixedDelay

```java
/**
 * 创建一个周期执行的任务，第一次执行延期时间为initialDelay，
 * 之后每隔period执行一次，不等待第一次执行完成就开始计时
 */
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                              long initialDelay,
                                              long period,
                                              TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    if (period <= 0)
        throw new IllegalArgumentException();
    //构建RunnableScheduledFuture任务类型
    ScheduledFutureTask<Void> sft =
        new ScheduledFutureTask<Void>(command,
                                      null,
                                      triggerTime(initialDelay, unit),//计算任务的延迟时间
                                      unit.toNanos(period));//计算任务的执行周期
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);//执行用户自定义逻辑
    sft.outerTask = t;//赋值给outerTask，准备重新入队等待下一次执行
    delayedExecute(t);//执行任务
    return t;
}

/**
 * 创建一个周期执行的任务，第一次执行延期时间为initialDelay，
 * 在第一次执行完之后延迟delay后开始下一次执行
 */
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                 long initialDelay,
                                                 long delay,
                                                 TimeUnit unit) {
    if (command == null || unit == null)
        throw new NullPointerException();
    if (delay <= 0)
        throw new IllegalArgumentException();
    //构建RunnableScheduledFuture任务类型
    ScheduledFutureTask<Void> sft =
        new ScheduledFutureTask<Void>(command,
                                      null,
                                      triggerTime(initialDelay, unit),//计算任务的延迟时间
                                      unit.toNanos(-delay));//计算任务的执行周期
    RunnableScheduledFuture<Void> t = decorateTask(command, sft);//执行用户自定义逻辑
    sft.outerTask = t;//赋值给outerTask，准备重新入队等待下一次执行
    delayedExecute(t);//执行任务
    return t;
}
```

说明: scheduleAtFixedRate和scheduleWithFixedDelay方法的逻辑与schedule类似。

**注意scheduleAtFixedRate和scheduleWithFixedDelay的区别**: 乍一看两个方法一模一样，其实，在unit.toNanos这一行代码中还是有区别的。没错，scheduleAtFixedRate传的是正值，而scheduleWithFixedDelay传的则是负值，这个值就是 ScheduledFutureTask 的period属性。

#### 核心方法:shutdown()

```java
public void shutdown() {
    super.shutdown();
}
//取消并清除由于关闭策略不应该运行的所有任务
@Override void onShutdown() {
    BlockingQueue<Runnable> q = super.getQueue();
    //获取run-after-shutdown参数
    boolean keepDelayed =
        getExecuteExistingDelayedTasksAfterShutdownPolicy();
    boolean keepPeriodic =
        getContinueExistingPeriodicTasksAfterShutdownPolicy();
    if (!keepDelayed && !keepPeriodic) {//池关闭后不保留任务
        //依次取消任务
        for (Object e : q.toArray())
            if (e instanceof RunnableScheduledFuture<?>)
                ((RunnableScheduledFuture<?>) e).cancel(false);
        q.clear();//清除等待队列
    }
    else {//池关闭后保留任务
        // Traverse snapshot to avoid iterator exceptions
        //遍历快照以避免迭代器异常
        for (Object e : q.toArray()) {
            if (e instanceof RunnableScheduledFuture) {
                RunnableScheduledFuture<?> t =
                    (RunnableScheduledFuture<?>)e;
                if ((t.isPeriodic() ? !keepPeriodic : !keepDelayed) ||
                    t.isCancelled()) { // also remove if already cancelled
                    //如果任务已经取消，移除队列中的任务
                    if (q.remove(t))
                        t.cancel(false);
                }
            }
        }
    }
    tryTerminate(); //终止线程池
}
```

说明: 池关闭方法调用了父类ThreadPoolExecutor的shutdown，具体分析见 ThreadPoolExecutor 篇。这里主要介绍以下在shutdown方法中调用的关闭钩子onShutdown方法，它的主要作用是在关闭线程池后取消并清除由于关闭策略不应该运行的所有任务，这里主要是根据 run-after-shutdown 参数(continueExistingPeriodicTasksAfterShutdown和executeExistingDelayedTasksAfterShutdown)来决定线程池关闭后是否关闭已经存在的任务。

### 再深入理解

- **为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor？**

例如: 由于 ScheduledThreadPoolExecutor 是一个固定核心线程数大小的线程池，并且使用了一个无界队列，所以调整maximumPoolSize对其没有任何影响(所以 ScheduledThreadPoolExecutor 没有提供可以调整最大线程数的构造函数，默认最大线程数固定为Integer.MAX_VALUE)。此外，设置corePoolSize为0或者设置核心线程空闲后清除(allowCoreThreadTimeOut)同样也不是一个好的策略，因为一旦周期任务到达某一次运行周期时，可能导致线程池内没有线程去处理这些任务。

- Executors 提供了哪几种方法来构造 ScheduledThreadPoolExecutor？
  - newScheduledThreadPool: 可指定核心线程数的线程池。
  - newSingleThreadScheduledExecutor: 只有一个工作线程的线程池。如果内部工作线程由于执行周期任务异常而被终止，则会新建一个线程替代它的位置。

注意: newScheduledThreadPool(1, threadFactory) 不等价于newSingleThreadScheduledExecutor。newSingleThreadScheduledExecutor创建的线程池保证内部只有一个线程执行任务，并且线程数不可扩展；而通过newScheduledThreadPool(1, threadFactory)创建的线程池可以通过setCorePoolSize方法来修改核心线程数。



### 课后问题

- ScheduledThreadPoolExecutor要解决什么样的问题?
- ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?
- ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?
- ScheduledThreadPoolExecutor有哪两个关闭策略? 区别是什么?
- ScheduledThreadPoolExecutor中scheduleAtFixedRate 和 scheduleWithFixedDelay区别是什么?
- 为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?
- Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?



### 参考文章

- 文章主要参考自泰迪的bagwell的https://www.jianshu.com/p/8c97953f2751，在此基础上做了增改。



## JUC线程池: Fork/Join框架详解

---

### Fork/Join框架简介

Fork/Join框架是Java并发工具包中的一种可以将一个大任务拆分为很多小任务来异步执行的工具，自JDK1.7引入。

#### 三个模块及关系

Fork/Join框架主要包含三个模块:

- 任务对象: `ForkJoinTask` (包括`RecursiveTask`、`RecursiveAction` 和 `CountedCompleter`)
- 执行Fork/Join任务的线程: `ForkJoinWorkerThread`
- 线程池: `ForkJoinPool`

这三者的关系是: ForkJoinPool可以通过池中的ForkJoinWorkerThread来处理ForkJoinTask任务。

```java
// from 《A Java Fork/Join Framework》Dong Lea
Result solve(Problem problem) {
	if (problem is small)
 		directly solve problem
 	else {
 		split problem into independent parts
 		fork new subtasks to solve each part
 		join all subtasks
 		compose result from subresults
	}
}
```

ForkJoinPool 只接收 ForkJoinTask 任务(在实际使用中，也可以接收 Runnable/Callable 任务，但在真正运行时，也会把这些任务封装成 ForkJoinTask 类型的任务)，RecursiveTask 是 ForkJoinTask 的子类，是一个可以递归执行的 ForkJoinTask，RecursiveAction 是一个无返回值的 RecursiveTask，CountedCompleter 在任务完成执行后会触发执行一个自定义的钩子函数。

在实际运用中，我们一般都会继承 `RecursiveTask` 、`RecursiveAction` 或 `CountedCompleter` 来实现我们的业务需求，而不会直接继承 ForkJoinTask 类。

#### 核心思想: 分治算法(Divide-and-Conquer)

分治算法(Divide-and-Conquer)把任务递归的拆分为各个子任务，这样可以更好的利用系统资源，尽可能的使用所有可用的计算能力来提升应用性能。首先看一下 Fork/Join 框架的任务运行机制:

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/0fbbe038-e320-4c6f-8a7e-58bc5008d189_java-thread-x-forkjoin-2.png)

- 这里也可以一并看下: [算法思想 - 分治算法]()

#### 核心思想: work-stealing(工作窃取)算法

work-stealing(工作窃取)算法: 线程池内的所有工作线程都尝试找到并执行已经提交的任务，或者是被其他活动任务创建的子任务(如果不存在就阻塞等待)。这种特性使得 ForkJoinPool 在运行多个可以产生子任务的任务，或者是提交的许多小任务时效率更高。尤其是构建异步模型的 ForkJoinPool 时，对不需要合并(join)的事件类型任务也非常适用。

在 ForkJoinPool 中，线程池中每个工作线程(ForkJoinWorkerThread)都对应一个任务队列(WorkQueue)，工作线程优先处理来自自身队列的任务(LIFO或FIFO顺序，参数 mode 决定)，然后以FIFO的顺序随机窃取其他队列中的任务。

具体思路如下:

- 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。
- 队列支持三个功能push、pop、poll
- push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用。
- 划分的子任务调用fork时，都会被push到自己的队列中。
- 默认情况下，工作线程从自己的双端队列获出任务并执行。
- 当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/b57a9338-a7b5-4de1-8710-97943ac3a9ca_java-thread-x-forkjoin-3.png)

#### Fork/Join 框架的执行流程

上图可以看出ForkJoinPool 中的任务执行分两种:

- 直接通过 FJP 提交的外部任务(external/submissions task)，存放在 workQueues 的偶数槽位；
- 通过内部 fork 分割的子任务(Worker task)，存放在 workQueues 的奇数槽位。

那Fork/Join 框架的执行流程是什么样的?

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/5ffab91c-6645-48a2-8735-c27a94855678_java-thread-x-forkjoin-5.png)

> 后续的源码解析将围绕上图进行。

### Fork/Join类关系

#### ForkJoinPool继承关系

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/76c03b60-454c-41a9-8c65-192d2bb0eea8_java-thread-x-forkjoin-1.png)

内部类介绍:

- ForkJoinWorkerThreadFactory: 内部线程工厂接口，用于创建工作线程ForkJoinWorkerThread
- DefaultForkJoinWorkerThreadFactory: ForkJoinWorkerThreadFactory 的默认实现类
- InnocuousForkJoinWorkerThreadFactory: 实现了 ForkJoinWorkerThreadFactory，无许可线程工厂，当系统变量中有系统安全管理相关属性时，默认使用这个工厂创建工作线程。
- EmptyTask: 内部占位类，用于替换队列中 join 的任务。
- ManagedBlocker: 为 ForkJoinPool 中的任务提供扩展管理并行数的接口，一般用在可能会阻塞的任务(如在 Phaser 中用于等待 phase 到下一个generation)。
- WorkQueue: ForkJoinPool 的核心数据结构，本质上是work-stealing 模式的双端任务队列，内部存放 ForkJoinTask 对象任务，使用 @Contented 注解修饰防止伪共享。
  - 工作线程在运行中产生新的任务(通常是因为调用了 fork())时，此时可以把 WorkQueue 的数据结构视为一个栈，新的任务会放入栈顶(top 位)；工作线程在处理自己工作队列的任务时，按照 LIFO 的顺序。
  - 工作线程在处理自己的工作队列同时，会尝试窃取一个任务(可能是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的队列任务)，此时可以把 WorkQueue 的数据结构视为一个 FIFO 的队列，窃取的任务位于其他线程的工作队列的队首(base位)。
- 伪共享状态: 缓存系统中是以缓存行(cache line)为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

#### ForkJoinTask继承关系

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-11/88e87298-df2f-44c8-8999-1c1035662bba_java-thread-x-forkjoin-4.png)

ForkJoinTask 实现了 Future 接口，说明它也是一个可取消的异步运算任务，实际上ForkJoinTask 是 Future 的轻量级实现，主要用在纯粹是计算的函数式任务或者操作完全独立的对象计算任务。fork 是主运行方法，用于异步执行；而 join 方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果。 其内部类都比较简单，ExceptionNode 是用于存储任务执行期间的异常信息的单向链表；其余四个类是为 Runnable/Callable 任务提供的适配器类，用于把 Runnable/Callable 转化为 ForkJoinTask 类型的任务(因为 ForkJoinPool 只可以运行 ForkJoinTask 类型的任务)。

### Fork/Join框架源码解析

> 分析思路: 在对类层次结构有了解以后，我们先看下内部核心参数，然后分析上述流程图。会分4个部分:

- 首先介绍任务的提交流程 - 外部任务(external/submissions task)提交
- 然后介绍任务的提交流程 - 子任务(Worker task)提交
- 再分析任务的执行过程(ForkJoinWorkerThread.run()到ForkJoinTask.doExec()这一部分)；
- 最后介绍任务的结果获取(ForkJoinTask.join()和ForkJoinTask.invoke())

#### ForkJoinPool

##### 核心参数

在后面的源码解析中，我们会看到大量的位运算，这些位运算都是通过我们接下来介绍的一些常量参数来计算的。

例如，如果要更新活跃线程数，使用公式(UC_MASK & (c + AC_UNIT)) | (SP_MASK & c)；c 代表当前 ctl，UC_MASK 和 SP_MASK 分别是高位和低位掩码，AC_UNIT 为活跃线程的增量数，使用(UC_MASK & (c + AC_UNIT))就可以计算出高32位，然后再加上低32位(SP_MASK & c)，就拼接成了一个新的ctl。

这些运算的可读性很差，看起来有些复杂。在后面源码解析中有位运算的地方我都会加上注释，大家只需要了解它们的作用即可。

ForkJoinPool 与 内部类 WorkQueue 共享的一些常量:

```java
// Constants shared across ForkJoinPool and WorkQueue

// 限定参数
static final int SMASK = 0xffff;        //  低位掩码，也是最大索引位
static final int MAX_CAP = 0x7fff;        //  工作线程最大容量
static final int EVENMASK = 0xfffe;        //  偶数低位掩码
static final int SQMASK = 0x007e;        //  workQueues 数组最多64个槽位

// ctl 子域和 WorkQueue.scanState 的掩码和标志位
static final int SCANNING = 1;             // 标记是否正在运行任务
static final int INACTIVE = 1 << 31;       // 失活状态  负数
static final int SS_SEQ = 1 << 16;       // 版本戳，防止ABA问题

// ForkJoinPool.config 和 WorkQueue.config 的配置信息标记
static final int MODE_MASK = 0xffff << 16;  // 模式掩码
static final int LIFO_QUEUE = 0; //LIFO队列
static final int FIFO_QUEUE = 1 << 16;//FIFO队列
static final int SHARED_QUEUE = 1 << 31;       // 共享模式队列，负数
```

ForkJoinPool 中的相关常量和实例字段:

```java
//  低位和高位掩码
private static final long SP_MASK = 0xffffffffL;
private static final long UC_MASK = ~SP_MASK;

// 活跃线程数
private static final int AC_SHIFT = 48;
private static final long AC_UNIT = 0x0001L << AC_SHIFT; //活跃线程数增量
private static final long AC_MASK = 0xffffL << AC_SHIFT; //活跃线程数掩码

// 工作线程数
private static final int TC_SHIFT = 32;
private static final long TC_UNIT = 0x0001L << TC_SHIFT; //工作线程数增量
private static final long TC_MASK = 0xffffL << TC_SHIFT; //掩码
private static final long ADD_WORKER = 0x0001L << (TC_SHIFT + 15);  // 创建工作线程标志

// 池状态
private static final int RSLOCK = 1;
private static final int RSIGNAL = 1 << 1;
private static final int STARTED = 1 << 2;
private static final int STOP = 1 << 29;
private static final int TERMINATED = 1 << 30;
private static final int SHUTDOWN = 1 << 31;

// 实例字段
volatile long ctl;                   // 主控制参数
volatile int runState;               // 运行状态锁
final int config;                    // 并行度|模式
int indexSeed;                       // 用于生成工作线程索引
volatile WorkQueue[] workQueues;     // 主对象注册信息，workQueue
final ForkJoinWorkerThreadFactory factory;// 线程工厂
final UncaughtExceptionHandler ueh;  // 每个工作线程的异常信息
final String workerNamePrefix;       // 用于创建工作线程的名称
volatile AtomicLong stealCounter;    // 偷取任务总数，也可作为同步监视器

/** 静态初始化字段 */
//线程工厂
public static final ForkJoinWorkerThreadFactory defaultForkJoinWorkerThreadFactory;
//启动或杀死线程的方法调用者的权限
private static final RuntimePermission modifyThreadPermission;
// 公共静态pool
static final ForkJoinPool common;
//并行度，对应内部common池
static final int commonParallelism;
//备用线程数，在tryCompensate中使用
private static int commonMaxSpares;
//创建workerNamePrefix(工作线程名称前缀)时的序号
private static int poolNumberSequence;
//线程阻塞等待新的任务的超时值(以纳秒为单位)，默认2秒
private static final long IDLE_TIMEOUT = 2000L * 1000L * 1000L; // 2sec
//空闲超时时间，防止timer未命中
private static final long TIMEOUT_SLOP = 20L * 1000L * 1000L;  // 20ms
//默认备用线程数
private static final int DEFAULT_COMMON_MAX_SPARES = 256;
//阻塞前自旋的次数，用在在awaitRunStateLock和awaitWork中
private static final int SPINS  = 0;
//indexSeed的增量
private static final int SEED_INCREMENT = 0x9e3779b9; 
```

说明:  ForkJoinPool 的内部状态都是通过一个64位的 long 型 变量ctl来存储，它由四个16位的子域组成:

- AC: 正在运行工作线程数减去目标并行度，高16位
- TC: 总工作线程数减去目标并行度，中高16位
- SS: 栈顶等待线程的版本计数和状态，中低16位
- ID:  栈顶 WorkQueue 在池中的索引(poolIndex)，低16位

在后面的源码解析中，某些地方也提取了ctl的低32位(sp=(int)ctl)来检查工作线程状态，例如，当sp不为0时说明当前还有空闲工作线程。

##### ForkJoinPool.WorkQueue 中的相关属性:

```java
//初始队列容量，2的幂
static final int INITIAL_QUEUE_CAPACITY = 1 << 13;
//最大队列容量
static final int MAXIMUM_QUEUE_CAPACITY = 1 << 26; // 64M

// 实例字段
volatile int scanState;    // Woker状态, <0: inactive; odd:scanning
int stackPred;             // 记录前一个栈顶的ctl
int nsteals;               // 偷取任务数
int hint;                  // 记录偷取者索引，初始为随机索引
int config;                // 池索引和模式
volatile int qlock;        // 1: locked, < 0: terminate; else 0
volatile int base;         //下一个poll操作的索引(栈底/队列头)
int top;                   //  下一个push操作的索引(栈顶/队列尾)
ForkJoinTask<?>[] array;   // 任务数组
final ForkJoinPool pool;   // the containing pool (may be null)
final ForkJoinWorkerThread owner; // 当前工作队列的工作线程，共享模式下为null
volatile Thread parker;    // 调用park阻塞期间为owner，其他情况为null
volatile ForkJoinTask<?> currentJoin;  // 记录被join过来的任务
volatile ForkJoinTask<?> currentSteal; // 记录从其他工作队列偷取过来的任务
  
        
    
```

#### ForkJoinTask

##### 核心参数

```java
/** 任务运行状态 */
volatile int status; // 任务运行状态
static final int DONE_MASK   = 0xf0000000;  // 任务完成状态标志位
static final int NORMAL      = 0xf0000000;  // must be negative
static final int CANCELLED   = 0xc0000000;  // must be < NORMAL
static final int EXCEPTIONAL = 0x80000000;  // must be < CANCELLED
static final int SIGNAL      = 0x00010000;  // must be >= 1 << 16 等待信号
static final int SMASK       = 0x0000ffff;  //  低位掩码
```

### Fork/Join框架源码解析

#### 构造函数

```java
public ForkJoinPool(int parallelism,
                    ForkJoinWorkerThreadFactory factory,
                    UncaughtExceptionHandler handler,
                    boolean asyncMode) {
    this(checkParallelism(parallelism),
            checkFactory(factory),
            handler,
            asyncMode ? FIFO_QUEUE : LIFO_QUEUE,
            "ForkJoinPool-" + nextPoolId() + "-worker-");
    checkPermission();
}
```

说明:  在 ForkJoinPool 中我们可以自定义四个参数:

- parallelism: 并行度，默认为CPU数，最小为1
- factory: 工作线程工厂；
- handler: 处理工作线程运行任务时的异常情况类，默认为null；
- asyncMode: 是否为异步模式，默认为 false。如果为true，表示子任务的执行遵循 FIFO 顺序并且任务不能被合并(join)，这种模式适用于工作线程只运行事件类型的异步任务。

在多数场景使用时，如果没有太强的业务需求，我们一般直接使用 ForkJoinPool 中的common池，在JDK1.8之后提供了ForkJoinPool.commonPool()方法可以直接使用common池，来看一下它的构造:

```java
private static ForkJoinPool makeCommonPool() {
    int parallelism = -1;
    ForkJoinWorkerThreadFactory factory = null;
    UncaughtExceptionHandler handler = null;
    try {  // ignore exceptions in accessing/parsing
        String pp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.parallelism");//并行度
        String fp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.threadFactory");//线程工厂
        String hp = System.getProperty
                ("java.util.concurrent.ForkJoinPool.common.exceptionHandler");//异常处理类
        if (pp != null)
            parallelism = Integer.parseInt(pp);
        if (fp != null)
            factory = ((ForkJoinWorkerThreadFactory) ClassLoader.
                    getSystemClassLoader().loadClass(fp).newInstance());
        if (hp != null)
            handler = ((UncaughtExceptionHandler) ClassLoader.
                    getSystemClassLoader().loadClass(hp).newInstance());
    } catch (Exception ignore) {
    }
    if (factory == null) {
        if (System.getSecurityManager() == null)
            factory = defaultForkJoinWorkerThreadFactory;
        else // use security-managed default
            factory = new InnocuousForkJoinWorkerThreadFactory();
    }
    if (parallelism < 0 && // default 1 less than #cores
            (parallelism = Runtime.getRuntime().availableProcessors() - 1) <= 0)
        parallelism = 1;//默认并行度为1
    if (parallelism > MAX_CAP)
        parallelism = MAX_CAP;
    return new ForkJoinPool(parallelism, factory, handler, LIFO_QUEUE,
            "ForkJoinPool.commonPool-worker-");
}
```

使用common pool的优点就是我们可以通过指定系统参数的方式定义“并行度、线程工厂和异常处理类”；并且它使用的是同步模式，也就是说可以支持任务合并(join)。

#### 执行流程 - 外部任务(external/submissions task)提交

向 ForkJoinPool 提交任务有三种方式:

- invoke()会等待任务计算完毕并返回计算结果；
- execute()是直接向池提交一个任务来异步执行，无返回结果；
- submit()也是异步执行，但是会返回提交的任务，在适当的时候可通过task.get()获取执行结果。

这三种提交方式都都是调用externalPush()方法来完成，所以接下来我们将从externalPush()方法开始逐步分析外部任务的执行过程。

##### externalPush(ForkJoinTask<?> task)

```java
//添加给定任务到submission队列中
final void externalPush(ForkJoinTask<?> task) {
    WorkQueue[] ws;
    WorkQueue q;
    int m;
    int r = ThreadLocalRandom.getProbe();//探针值，用于计算WorkQueue槽位索引
    int rs = runState;
    if ((ws = workQueues) != null && (m = (ws.length - 1)) >= 0 &&
            (q = ws[m & r & SQMASK]) != null && r != 0 && rs > 0 && //获取随机偶数槽位的workQueue
            U.compareAndSwapInt(q, QLOCK, 0, 1)) {//锁定workQueue
        ForkJoinTask<?>[] a;
        int am, n, s;
        if ((a = q.array) != null &&
                (am = a.length - 1) > (n = (s = q.top) - q.base)) {
            int j = ((am & s) << ASHIFT) + ABASE;//计算任务索引位置
            U.putOrderedObject(a, j, task);//任务入列
            U.putOrderedInt(q, QTOP, s + 1);//更新push slot
            U.putIntVolatile(q, QLOCK, 0);//解除锁定
            if (n <= 1)
                signalWork(ws, q);//任务数小于1时尝试创建或激活一个工作线程
            return;
        }
        U.compareAndSwapInt(q, QLOCK, 1, 0);//解除锁定
    }
    externalSubmit(task);//初始化workQueues及相关属性
}
```

首先说明一下externalPush和externalSubmit两个方法的联系: 它们的作用都是把任务放到队列中等待执行。不同的是，externalSubmit可以说是完整版的externalPush，在任务首次提交时，需要初始化workQueues及其他相关属性，这个初始化操作就是externalSubmit来完成的；而后再向池中提交的任务都是通过简化版的externalSubmit-externalPush来完成。

externalPush的执行流程很简单: 首先找到一个随机偶数槽位的 workQueue，然后把任务放入这个 workQueue 的任务数组中，并更新top位。如果队列的剩余任务数小于1，则尝试创建或激活一个工作线程来运行任务(防止在externalSubmit初始化时发生异常导致工作线程创建失败)。

##### externalSubmit(ForkJoinTask<?> task)

```java
//任务提交
private void externalSubmit(ForkJoinTask<?> task) {
    //初始化调用线程的探针值，用于计算WorkQueue索引
    int r;                                    // initialize caller's probe
    if ((r = ThreadLocalRandom.getProbe()) == 0) {
        ThreadLocalRandom.localInit();
        r = ThreadLocalRandom.getProbe();
    }
    for (; ; ) {
        WorkQueue[] ws;
        WorkQueue q;
        int rs, m, k;
        boolean move = false;
        if ((rs = runState) < 0) {// 池已关闭
            tryTerminate(false, false);     // help terminate
            throw new RejectedExecutionException();
        }
        //初始化workQueues
        else if ((rs & STARTED) == 0 ||     // initialize
                ((ws = workQueues) == null || (m = ws.length - 1) < 0)) {
            int ns = 0;
            rs = lockRunState();//锁定runState
            try {
                //初始化
                if ((rs & STARTED) == 0) {
                    //初始化stealCounter
                    U.compareAndSwapObject(this, STEALCOUNTER, null,
                            new AtomicLong());
                    //创建workQueues，容量为2的幂次方
                    // create workQueues array with size a power of two
                    int p = config & SMASK; // ensure at least 2 slots
                    int n = (p > 1) ? p - 1 : 1;
                    n |= n >>> 1;
                    n |= n >>> 2;
                    n |= n >>> 4;
                    n |= n >>> 8;
                    n |= n >>> 16;
                    n = (n + 1) << 1;
                    workQueues = new WorkQueue[n];
                    ns = STARTED;
                }
            } finally {
                unlockRunState(rs, (rs & ~RSLOCK) | ns);//解锁并更新runState
            }
        } else if ((q = ws[k = r & m & SQMASK]) != null) {//获取随机偶数槽位的workQueue
            if (q.qlock == 0 && U.compareAndSwapInt(q, QLOCK, 0, 1)) {//锁定 workQueue
                ForkJoinTask<?>[] a = q.array;//当前workQueue的全部任务
                int s = q.top;
                boolean submitted = false; // initial submission or resizing
                try {                      // locked version of push
                    if ((a != null && a.length > s + 1 - q.base) ||
                            (a = q.growArray()) != null) {//扩容
                        int j = (((a.length - 1) & s) << ASHIFT) + ABASE;
                        U.putOrderedObject(a, j, task);//放入给定任务
                        U.putOrderedInt(q, QTOP, s + 1);//修改push slot
                        submitted = true;
                    }
                } finally {
                    U.compareAndSwapInt(q, QLOCK, 1, 0);//解除锁定
                }
                if (submitted) {//任务提交成功，创建或激活工作线程
                    signalWork(ws, q);//创建或激活一个工作线程来运行任务
                    return;
                }
            }
            move = true;                   // move on failure 操作失败，重新获取探针值
        } else if (((rs = runState) & RSLOCK) == 0) { // create new queue
            q = new WorkQueue(this, null);
            q.hint = r;
            q.config = k | SHARED_QUEUE;
            q.scanState = INACTIVE;
            rs = lockRunState();           // publish index
            if (rs > 0 && (ws = workQueues) != null &&
                    k < ws.length && ws[k] == null)
                ws[k] = q;                 // 更新索引k位值的workQueue
            //else terminated
            unlockRunState(rs, rs & ~RSLOCK);
        } else
            move = true;                   // move if busy
        if (move)
            r = ThreadLocalRandom.advanceProbe(r);//重新获取线程探针值
    }
}
```

说明:  externalSubmit是externalPush的完整版本，主要用于第一次提交任务时初始化workQueues及相关属性，并且提交给定任务到队列中。具体执行步骤如下:

- 如果池为终止状态(runState<0)，调用tryTerminate来终止线程池，并抛出任务拒绝异常；
- 如果尚未初始化，就为 FJP 执行初始化操作: 初始化stealCounter、创建workerQueues，然后继续自旋；
- 初始化完成后，执行在externalPush中相同的操作: 获取 workQueue，放入指定任务。任务提交成功后调用signalWork方法创建或激活线程；
- 如果在步骤3中获取到的 workQueue 为null，会在这一步中创建一个 workQueue，创建成功继续自旋执行第三步操作；
- 如果非上述情况，或者有线程争用资源导致获取锁失败，就重新获取线程探针值继续自旋。

##### signalWork(WorkQueue[] ws, WorkQueue q)

```java
final void signalWork(WorkQueue[] ws, WorkQueue q) {
    long c;
    int sp, i;
    WorkQueue v;
    Thread p;
    while ((c = ctl) < 0L) {                       // too few active
        if ((sp = (int) c) == 0) {                  // no idle workers
            if ((c & ADD_WORKER) != 0L)            // too few workers
                tryAddWorker(c);//工作线程太少，添加新的工作线程
            break;
        }
        if (ws == null)                            // unstarted/terminated
            break;
        if (ws.length <= (i = sp & SMASK))         // terminated
            break;
        if ((v = ws[i]) == null)                   // terminating
            break;
        //计算ctl，加上版本戳SS_SEQ避免ABA问题
        int vs = (sp + SS_SEQ) & ~INACTIVE;        // next scanState
        int d = sp - v.scanState;                  // screen CAS
        //计算活跃线程数(高32位)并更新为下一个栈顶的scanState(低32位)
        long nc = (UC_MASK & (c + AC_UNIT)) | (SP_MASK & v.stackPred);
        if (d == 0 && U.compareAndSwapLong(this, CTL, c, nc)) {
            v.scanState = vs;                      // activate v
            if ((p = v.parker) != null)
                U.unpark(p);//唤醒阻塞线程
            break;
        }
        if (q != null && q.base == q.top)          // no more work
            break;
    }
}
  
        
    
```

说明: 新建或唤醒一个工作线程，在externalPush、externalSubmit、workQueue.push、scan中调用。如果还有空闲线程，则尝试唤醒索引到的 WorkQueue 的parker线程；如果工作线程过少((ctl & ADD_WORKER) != 0L)，则调用tryAddWorker添加一个新的工作线程。

##### tryAddWorker(long c)

```java
private void tryAddWorker(long c) {
    boolean add = false;
    do {
        long nc = ((AC_MASK & (c + AC_UNIT)) |
                   (TC_MASK & (c + TC_UNIT)));
        if (ctl == c) {
            int rs, stop;                 // check if terminating
            if ((stop = (rs = lockRunState()) & STOP) == 0)
                add = U.compareAndSwapLong(this, CTL, c, nc);
            unlockRunState(rs, rs & ~RSLOCK);//释放锁
            if (stop != 0)
                break;
            if (add) {
                createWorker();//创建工作线程
                break;
            }
        }
    } while (((c = ctl) & ADD_WORKER) != 0L && (int)c == 0);
}
  
        
    
```

说明: 尝试添加一个新的工作线程，首先更新ctl中的工作线程数，然后调用createWorker()创建工作线程。

##### createWorker()

```java
private boolean createWorker() {
    ForkJoinWorkerThreadFactory fac = factory;
    Throwable ex = null;
    ForkJoinWorkerThread wt = null;
    try {
        if (fac != null && (wt = fac.newThread(this)) != null) {
            wt.start();
            return true;
        }
    } catch (Throwable rex) {
        ex = rex;
    }
    deregisterWorker(wt, ex);//线程创建失败处理
    return false;
}
  
        
    
```

说明: createWorker首先通过线程工厂创一个新的ForkJoinWorkerThread，然后启动这个工作线程(wt.start())。如果期间发生异常，调用deregisterWorker处理线程创建失败的逻辑(deregisterWorker在后面再详细说明)。

ForkJoinWorkerThread 的构造函数如下:

```java
protected ForkJoinWorkerThread(ForkJoinPool pool) {
    // Use a placeholder until a useful name can be set in registerWorker
    super("aForkJoinWorkerThread");
    this.pool = pool;
    this.workQueue = pool.registerWorker(this);
}
  
        
    
```

可以看到 ForkJoinWorkerThread 在构造时首先调用父类 Thread 的方法，然后为工作线程注册pool和workQueue，而workQueue的注册任务由ForkJoinPool.registerWorker来完成。

##### registerWorker()

```java
final WorkQueue registerWorker(ForkJoinWorkerThread wt) {
    UncaughtExceptionHandler handler;
    //设置为守护线程
    wt.setDaemon(true);                           // configure thread
    if ((handler = ueh) != null)
        wt.setUncaughtExceptionHandler(handler);
    WorkQueue w = new WorkQueue(this, wt);//构造新的WorkQueue
    int i = 0;                                    // assign a pool index
    int mode = config & MODE_MASK;
    int rs = lockRunState();
    try {
        WorkQueue[] ws;
        int n;                    // skip if no array
        if ((ws = workQueues) != null && (n = ws.length) > 0) {
            //生成新建WorkQueue的索引
            int s = indexSeed += SEED_INCREMENT;  // unlikely to collide
            int m = n - 1;
            i = ((s << 1) | 1) & m;               // Worker任务放在奇数索引位 odd-numbered indices
            if (ws[i] != null) {                  // collision 已存在，重新计算索引位
                int probes = 0;                   // step by approx half n
                int step = (n <= 4) ? 2 : ((n >>> 1) & EVENMASK) + 2;
                //查找可用的索引位
                while (ws[i = (i + step) & m] != null) {
                    if (++probes >= n) {//所有索引位都被占用，对workQueues进行扩容
                        workQueues = ws = Arrays.copyOf(ws, n <<= 1);//workQueues 扩容
                        m = n - 1;
                        probes = 0;
                    }
                }
            }
            w.hint = s;                           // use as random seed
            w.config = i | mode;
            w.scanState = i;                      // publication fence
            ws[i] = w;
        }
    } finally {
        unlockRunState(rs, rs & ~RSLOCK);
    }
    wt.setName(workerNamePrefix.concat(Integer.toString(i >>> 1)));
    return w;
}
  
        
    
```

说明: registerWorker是 ForkJoinWorkerThread 构造器的回调函数，用于创建和记录工作线程的 WorkQueue。比较简单，就不多赘述了。注意在此为工作线程创建的 WorkQueue 是放在奇数索引的(代码行: i = ((s << 1) | 1) & m;)

##### 小结

OK，外部任务的提交流程就先讲到这里。在createWorker()中启动工作线程后(wt.start())，当为线程分配到CPU执行时间片之后会运行 ForkJoinWorkerThread 的run方法开启线程来执行任务。工作线程执行任务的流程我们在讲完内部任务提交之后会统一讲解。

#### 执行流程: 子任务(Worker task)提交

子任务的提交相对比较简单，由任务的fork()方法完成。通过上面的流程图可以看到任务被分割(fork)之后调用了ForkJoinPool.WorkQueue.push()方法直接把任务放到队列中等待被执行。

##### ForkJoinTask.fork()

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        ForkJoinPool.common.externalPush(this);
    return this;
}
  
        
    
```

说明: 如果当前线程是 Worker 线程，说明当前任务是fork分割的子任务，通过ForkJoinPool.workQueue.push()方法直接把任务放到自己的等待队列中；否则调用ForkJoinPool.externalPush()提交到一个随机的等待队列中(外部任务)。

##### ForkJoinPool.WorkQueue.push()

```java
final void push(ForkJoinTask<?> task) {
    ForkJoinTask<?>[] a;
    ForkJoinPool p;
    int b = base, s = top, n;
    if ((a = array) != null) {    // ignore if queue removed
        int m = a.length - 1;     // fenced write for task visibility
        U.putOrderedObject(a, ((m & s) << ASHIFT) + ABASE, task);
        U.putOrderedInt(this, QTOP, s + 1);
        if ((n = s - b) <= 1) {//首次提交，创建或唤醒一个工作线程
            if ((p = pool) != null)
                p.signalWork(p.workQueues, this);
        } else if (n >= m)
            growArray();
    }
}
  
        
    
```

说明: 首先把任务放入等待队列并更新top位；如果当前 WorkQueue 为新建的等待队列(top-base<=1)，则调用signalWork方法为当前 WorkQueue 新建或唤醒一个工作线程；如果 WorkQueue 中的任务数组容量过小，则调用growArray()方法对其进行两倍扩容，growArray()方法源码如下:

```java
final ForkJoinTask<?>[] growArray() {
    ForkJoinTask<?>[] oldA = array;//获取内部任务列表
    int size = oldA != null ? oldA.length << 1 : INITIAL_QUEUE_CAPACITY;
    if (size > MAXIMUM_QUEUE_CAPACITY)
        throw new RejectedExecutionException("Queue capacity exceeded");
    int oldMask, t, b;
    //新建一个两倍容量的任务数组
    ForkJoinTask<?>[] a = array = new ForkJoinTask<?>[size];
    if (oldA != null && (oldMask = oldA.length - 1) >= 0 &&
            (t = top) - (b = base) > 0) {
        int mask = size - 1;
        //从老数组中拿出数据，放到新的数组中
        do { // emulate poll from old array, push to new array
            ForkJoinTask<?> x;
            int oldj = ((b & oldMask) << ASHIFT) + ABASE;
            int j = ((b & mask) << ASHIFT) + ABASE;
            x = (ForkJoinTask<?>) U.getObjectVolatile(oldA, oldj);
            if (x != null &&
                    U.compareAndSwapObject(oldA, oldj, x, null))
                U.putObjectVolatile(a, j, x);
        } while (++b != t);
    }
    return a;
}
  
        
    
```

##### 小结

到此，两种任务的提交流程都已经解析完毕，下一节我们来一起看看任务提交之后是如何被运行的。

#### 执行流程: 任务执行

回到我们开始时的流程图，在ForkJoinPool .createWorker()方法中创建工作线程后，会启动工作线程，系统为工作线程分配到CPU执行时间片之后会执行 ForkJoinWorkerThread 的run()方法正式开始执行任务。

##### ForkJoinWorkerThread.run()

```java
public void run() {
    if (workQueue.array == null) { // only run once
        Throwable exception = null;
        try {
            onStart();//钩子方法，可自定义扩展
            pool.runWorker(workQueue);
        } catch (Throwable ex) {
            exception = ex;
        } finally {
            try {
                onTermination(exception);//钩子方法，可自定义扩展
            } catch (Throwable ex) {
                if (exception == null)
                    exception = ex;
            } finally {
                pool.deregisterWorker(this, exception);//处理异常
            }
        }
    }
}
  
        
    
```

说明: 方法很简单，在工作线程运行前后会调用自定义钩子函数(onStart和onTermination)，任务的运行则是调用了ForkJoinPool.runWorker()。如果全部任务执行完毕或者期间遭遇异常，则通过ForkJoinPool.deregisterWorker关闭工作线程并处理异常信息(deregisterWorker方法我们后面会详细讲解)。

##### ForkJoinPool.runWorker(WorkQueue w)

```java
final void runWorker(WorkQueue w) {
    w.growArray();                   // allocate queue
    int seed = w.hint;               // initially holds randomization hint
    int r = (seed == 0) ? 1 : seed;  // avoid 0 for xorShift
    for (ForkJoinTask<?> t; ; ) {
        if ((t = scan(w, r)) != null)//扫描任务执行
            w.runTask(t);
        else if (!awaitWork(w, r))
            break;
        r ^= r << 13;
        r ^= r >>> 17;
        r ^= r << 5; // xorshift
    }
}  
```

说明: runWorker是 ForkJoinWorkerThread 的主运行方法，用来依次执行当前工作线程中的任务。函数流程很简单: 调用scan方法依次获取任务，然后调用WorkQueue .runTask运行任务；如果未扫描到任务，则调用awaitWork等待，直到工作线程/线程池终止或等待超时。

##### ForkJoinPool.scan(WorkQueue w, int r)

```java
private ForkJoinTask<?> scan(WorkQueue w, int r) {
    WorkQueue[] ws;
    int m;
    if ((ws = workQueues) != null && (m = ws.length - 1) > 0 && w != null) {
        int ss = w.scanState;                     // initially non-negative
        //初始扫描起点，自旋扫描
        for (int origin = r & m, k = origin, oldSum = 0, checkSum = 0; ; ) {
            WorkQueue q;
            ForkJoinTask<?>[] a;
            ForkJoinTask<?> t;
            int b, n;
            long c;
            if ((q = ws[k]) != null) {//获取workQueue
                if ((n = (b = q.base) - q.top) < 0 &&
                        (a = q.array) != null) {      // non-empty
                    //计算偏移量
                    long i = (((a.length - 1) & b) << ASHIFT) + ABASE;
                    if ((t = ((ForkJoinTask<?>)
                            U.getObjectVolatile(a, i))) != null && //取base位置任务
                            q.base == b) {//stable
                        if (ss >= 0) {  //scanning
                            if (U.compareAndSwapObject(a, i, t, null)) {//
                                q.base = b + 1;//更新base位
                                if (n < -1)       // signal others
                                    signalWork(ws, q);//创建或唤醒工作线程来运行任务
                                return t;
                            }
                        } else if (oldSum == 0 &&   // try to activate 尝试激活工作线程
                                w.scanState < 0)
                            tryRelease(c = ctl, ws[m & (int) c], AC_UNIT);//唤醒栈顶工作线程
                    }
                    //base位置任务为空或base位置偏移，随机移位重新扫描
                    if (ss < 0)                   // refresh
                        ss = w.scanState;
                    r ^= r << 1;
                    r ^= r >>> 3;
                    r ^= r << 10;
                    origin = k = r & m;           // move and rescan
                    oldSum = checkSum = 0;
                    continue;
                }
                checkSum += b;//队列任务为空，记录base位
            }
            //更新索引k 继续向后查找
            if ((k = (k + 1) & m) == origin) {    // continue until stable
                //运行到这里说明已经扫描了全部的 workQueues，但并未扫描到任务

                if ((ss >= 0 || (ss == (ss = w.scanState))) &&
                        oldSum == (oldSum = checkSum)) {
                    if (ss < 0 || w.qlock < 0)    // already inactive
                        break;// 已经被灭活或终止,跳出循环

                    //对当前WorkQueue进行灭活操作
                    int ns = ss | INACTIVE;       // try to inactivate
                    long nc = ((SP_MASK & ns) |
                            (UC_MASK & ((c = ctl) - AC_UNIT)));//计算ctl为INACTIVE状态并减少活跃线程数
                    w.stackPred = (int) c;         // hold prev stack top
                    U.putInt(w, QSCANSTATE, ns);//修改scanState为inactive状态
                    if (U.compareAndSwapLong(this, CTL, c, nc))//更新scanState为灭活状态
                        ss = ns;
                    else
                        w.scanState = ss;         // back out
                }
                checkSum = 0;//重置checkSum，继续循环
            }
        }
    }
    return null;
} 
```

说明: 扫描并尝试偷取一个任务。使用w.hint进行随机索引 WorkQueue，也就是说并不一定会执行当前 WorkQueue 中的任务，而是偷取别的Worker的任务来执行。

函数的大概执行流程如下:

- 取随机位置的一个 WorkQueue；

- 获取base位的 ForkJoinTask，成功取到后更新base位并返回任务；如果取到的 WorkQueue 中任务数大于1，则调用signalWork创建或唤醒其他工作线程；

- 如果当前工作线程处于不活跃状态(INACTIVE)，则调用tryRelease尝试唤醒栈顶工作线程来执行。

  tryRelease源码如下:

  ```java
  private boolean tryRelease(long c, WorkQueue v, long inc) {
      int sp = (int) c, vs = (sp + SS_SEQ) & ~INACTIVE;
      Thread p;
      //ctl低32位等于scanState，说明可以唤醒parker线程
      if (v != null && v.scanState == sp) {          // v is at top of stack
          //计算活跃线程数(高32位)并更新为下一个栈顶的scanState(低32位)
          long nc = (UC_MASK & (c + inc)) | (SP_MASK & v.stackPred);
          if (U.compareAndSwapLong(this, CTL, c, nc)) {
              v.scanState = vs;
              if ((p = v.parker) != null)
                  U.unpark(p);//唤醒线程
              return true;
          }
      }
      return false;
  } 
  ```

- 如果base位任务为空或发生偏移，则对索引位进行随机移位，然后重新扫描；

- 如果扫描整个workQueues之后没有获取到任务，则设置当前工作线程为INACTIVE状态；然后重置checkSum，再次扫描一圈之后如果还没有任务则跳出循环返回null。

##### ForkJoinPool.awaitWork(WorkQueue w, int r)

```java
private boolean awaitWork(WorkQueue w, int r) {
    if (w == null || w.qlock < 0)                 // w is terminating
        return false;
    for (int pred = w.stackPred, spins = SPINS, ss; ; ) {
        if ((ss = w.scanState) >= 0)//正在扫描，跳出循环
            break;
        else if (spins > 0) {
            r ^= r << 6;
            r ^= r >>> 21;
            r ^= r << 7;
            if (r >= 0 && --spins == 0) {         // randomize spins
                WorkQueue v;
                WorkQueue[] ws;
                int s, j;
                AtomicLong sc;
                if (pred != 0 && (ws = workQueues) != null &&
                        (j = pred & SMASK) < ws.length &&
                        (v = ws[j]) != null &&        // see if pred parking
                        (v.parker == null || v.scanState >= 0))
                    spins = SPINS;                // continue spinning
            }
        } else if (w.qlock < 0)                     // 当前workQueue已经终止，返回false recheck after spins
            return false;
        else if (!Thread.interrupted()) {//判断线程是否被中断，并清除中断状态
            long c, prevctl, parkTime, deadline;
            int ac = (int) ((c = ctl) >> AC_SHIFT) + (config & SMASK);//活跃线程数
            if ((ac <= 0 && tryTerminate(false, false)) || //无active线程，尝试终止
                    (runState & STOP) != 0)           // pool terminating
                return false;
            if (ac <= 0 && ss == (int) c) {        // is last waiter
                //计算活跃线程数(高32位)并更新为下一个栈顶的scanState(低32位)
                prevctl = (UC_MASK & (c + AC_UNIT)) | (SP_MASK & pred);
                int t = (short) (c >>> TC_SHIFT);  // shrink excess spares
                if (t > 2 && U.compareAndSwapLong(this, CTL, c, prevctl))//总线程过量
                    return false;                 // else use timed wait
                //计算空闲超时时间
                parkTime = IDLE_TIMEOUT * ((t >= 0) ? 1 : 1 - t);
                deadline = System.nanoTime() + parkTime - TIMEOUT_SLOP;
            } else
                prevctl = parkTime = deadline = 0L;
            Thread wt = Thread.currentThread();
            U.putObject(wt, PARKBLOCKER, this);   // emulate LockSupport
            w.parker = wt;//设置parker，准备阻塞
            if (w.scanState < 0 && ctl == c)      // recheck before park
                U.park(false, parkTime);//阻塞指定的时间

            U.putOrderedObject(w, QPARKER, null);
            U.putObject(wt, PARKBLOCKER, null);
            if (w.scanState >= 0)//正在扫描，说明等到任务，跳出循环
                break;
            if (parkTime != 0L && ctl == c &&
                    deadline - System.nanoTime() <= 0L &&
                    U.compareAndSwapLong(this, CTL, c, prevctl))//未等到任务，更新ctl，返回false
                return false;                     // shrink pool
        }
    }
    return true;
}
  
        
    
```

说明: 回到runWorker方法，如果scan方法未扫描到任务，会调用awaitWork等待获取任务。函数的具体执行流程大家看源码，这里简单说一下:

- 在等待获取任务期间，如果工作线程或线程池已经终止则直接返回false。如果当前无 active 线程，尝试终止线程池并返回false，如果终止失败并且当前是最后一个等待的 Worker，就阻塞指定的时间(IDLE_TIMEOUT)；等到届期或被唤醒后如果发现自己是scanning(scanState >= 0)状态，说明已经等到任务，跳出等待返回true继续 scan，否则的更新ctl并返回false。

##### WorkQueue.runTask()

```java
final void runTask(ForkJoinTask<?> task) {
    if (task != null) {
        scanState &= ~SCANNING; // mark as busy
        (currentSteal = task).doExec();//更新currentSteal并执行任务
        U.putOrderedObject(this, QCURRENTSTEAL, null); // release for GC
        execLocalTasks();//依次执行本地任务
        ForkJoinWorkerThread thread = owner;
        if (++nsteals < 0)      // collect on overflow
            transferStealCount(pool);//增加偷取任务数
        scanState |= SCANNING;
        if (thread != null)
            thread.afterTopLevelExec();//执行钩子函数
    }
} 
```

说明: 在scan方法扫描到任务之后，调用WorkQueue.runTask()来执行获取到的任务，大概流程如下:

- 标记scanState为正在执行状态；

- 更新currentSteal为当前获取到的任务并执行它，任务的执行调用了ForkJoinTask.doExec()方法，源码如下:

  ```java
  //ForkJoinTask.doExec()
  final int doExec() {
      int s; boolean completed;
      if ((s = status) >= 0) {
          try {
              completed = exec();//执行我们定义的任务
          } catch (Throwable rex) {
              return setExceptionalCompletion(rex);
          }
          if (completed)
              s = setCompletion(NORMAL);
      }
      return s;
  }
    
          
      
  ```

- 调用execLocalTasks依次执行当前WorkerQueue中的任务，源码如下:

  ```java
  //执行并移除所有本地任务
  final void execLocalTasks() {
      int b = base, m, s;
      ForkJoinTask<?>[] a = array;
      if (b - (s = top - 1) <= 0 && a != null &&
              (m = a.length - 1) >= 0) {
          if ((config & FIFO_QUEUE) == 0) {//FIFO模式
              for (ForkJoinTask<?> t; ; ) {
                  if ((t = (ForkJoinTask<?>) U.getAndSetObject
                          (a, ((m & s) << ASHIFT) + ABASE, null)) == null)//FIFO执行，取top任务
                      break;
                  U.putOrderedInt(this, QTOP, s);
                  t.doExec();//执行
                  if (base - (s = top - 1) > 0)
                      break;
              }
          } else
              pollAndExecAll();//LIFO模式执行，取base任务
      }
  }
    
          
      
  ```

- 更新偷取任务数；

- 还原scanState并执行钩子函数。

##### ForkJoinPool.deregisterWorker(ForkJoinWorkerThread wt, Throwable ex)

```java
final void deregisterWorker(ForkJoinWorkerThread wt, Throwable ex) {
    WorkQueue w = null;
    //1.移除workQueue
    if (wt != null && (w = wt.workQueue) != null) {//获取ForkJoinWorkerThread的等待队列
        WorkQueue[] ws;                           // remove index from array
        int idx = w.config & SMASK;//计算workQueue索引
        int rs = lockRunState();//获取runState锁和当前池运行状态
        if ((ws = workQueues) != null && ws.length > idx && ws[idx] == w)
            ws[idx] = null;//移除workQueue
        unlockRunState(rs, rs & ~RSLOCK);//解除runState锁
    }
    //2.减少CTL数
    long c;                                       // decrement counts
    do {} while (!U.compareAndSwapLong
                 (this, CTL, c = ctl, ((AC_MASK & (c - AC_UNIT)) |
                                       (TC_MASK & (c - TC_UNIT)) |
                                       (SP_MASK & c))));
    //3.处理被移除workQueue内部相关参数
    if (w != null) {
        w.qlock = -1;                             // ensure set
        w.transferStealCount(this);
        w.cancelAll();                            // cancel remaining tasks
    }
    //4.如果线程未终止，替换被移除的workQueue并唤醒内部线程
    for (;;) {                                    // possibly replace
        WorkQueue[] ws; int m, sp;
        //尝试终止线程池
        if (tryTerminate(false, false) || w == null || w.array == null ||
            (runState & STOP) != 0 || (ws = workQueues) == null ||
            (m = ws.length - 1) < 0)              // already terminating
            break;
        //唤醒被替换的线程，依赖于下一步
        if ((sp = (int)(c = ctl)) != 0) {         // wake up replacement
            if (tryRelease(c, ws[sp & m], AC_UNIT))
                break;
        }
        //创建工作线程替换
        else if (ex != null && (c & ADD_WORKER) != 0L) {
            tryAddWorker(c);                      // create replacement
            break;
        }
        else                                      // don't need replacement
            break;
    }
    //5.处理异常
    if (ex == null)                               // help clean on way out
        ForkJoinTask.helpExpungeStaleExceptions();
    else                                          // rethrow
        ForkJoinTask.rethrow(ex);
}
  
        
    
```

说明: deregisterWorker方法用于工作线程运行完毕之后终止线程或处理工作线程异常，主要就是清除已关闭的工作线程或回滚创建线程之前的操作，并把传入的异常抛给 ForkJoinTask 来处理。具体步骤见源码注释。

##### 小结

本节我们对任务的执行流程进行了说明，后面我们将继续介绍任务的结果获取(join/invoke)。

#### 获取任务结果 - ForkJoinTask.join() / ForkJoinTask.invoke()

- join() :

```java
//合并任务结果
public final V join() {
    int s;
    if ((s = doJoin() & DONE_MASK) != NORMAL)
        reportException(s);
    return getRawResult();
}

//join, get, quietlyJoin的主实现方法
private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
    return (s = status) < 0 ? s :
        ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
        tryUnpush(this) && (s = doExec()) < 0 ? s :
        wt.pool.awaitJoin(w, this, 0L) :
        externalAwaitDone();
} 
```

- invoke() :

```java
//执行任务，并等待任务完成并返回结果
public final V invoke() {
    int s;
    if ((s = doInvoke() & DONE_MASK) != NORMAL)
        reportException(s);
    return getRawResult();
}

//invoke, quietlyInvoke的主实现方法
private int doInvoke() {
    int s; Thread t; ForkJoinWorkerThread wt;
    return (s = doExec()) < 0 ? s :
        ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        (wt = (ForkJoinWorkerThread)t).pool.
        awaitJoin(wt.workQueue, this, 0L) :
        externalAwaitDone();
}
```

说明:  join()方法一把是在任务fork()之后调用，用来获取(或者叫“合并”)任务的执行结果。

ForkJoinTask的join()和invoke()方法都可以用来获取任务的执行结果(另外还有get方法也是调用了doJoin来获取任务结果，但是会响应运行时异常)，它们对外部提交任务的执行方式一致，都是通过externalAwaitDone方法等待执行结果。不同的是invoke()方法会直接执行当前任务；而join()方法则是在当前任务在队列 top 位时(通过tryUnpush方法判断)才能执行，如果当前任务不在 top 位或者任务执行失败调用ForkJoinPool.awaitJoin方法帮助执行或阻塞当前 join 任务。(所以在官方文档中建议了我们对ForkJoinTask任务的调用顺序，一对 fork-join操作一般按照如下顺序调用: a.fork(); b.fork(); b.join(); a.join();。因为任务 b 是后面进入队列，也就是说它是在栈顶的(top 位)，在它fork()之后直接调用join()就可以直接执行而不会调用ForkJoinPool.awaitJoin方法去等待。)

在这些方法中，join()相对比较全面，所以之后的讲解我们将从join()开始逐步向下分析，首先看一下join()的执行流程:

![img](https://pdai.tech/_images/thread/java-thread-x-forkjoin-6.png)

后面的源码分析中，我们首先讲解比较简单的外部 join 任务(externalAwaitDone)，然后再讲解内部 join 任务(从ForkJoinPool.awaitJoin()开始)。

##### ForkJoinTask.externalAwaitDone()

```java
private int externalAwaitDone() {
    //执行任务
    int s = ((this instanceof CountedCompleter) ? // try helping
             ForkJoinPool.common.externalHelpComplete(  // CountedCompleter任务
                 (CountedCompleter<?>)this, 0) :
             ForkJoinPool.common.tryExternalUnpush(this) ? doExec() : 0);  // ForkJoinTask任务
    if (s >= 0 && (s = status) >= 0) {//执行失败，进入等待
        boolean interrupted = false;
        do {
            if (U.compareAndSwapInt(this, STATUS, s, s | SIGNAL)) {  //更新state
                synchronized (this) {
                    if (status >= 0) {//SIGNAL 等待信号
                        try {
                            wait(0L);
                        } catch (InterruptedException ie) {
                            interrupted = true;
                        }
                    }
                    else
                        notifyAll();
                }
            }
        } while ((s = status) >= 0);
        if (interrupted)
            Thread.currentThread().interrupt();
    }
    return s;
}
```

说明:  如果当前join为外部调用，则调用此方法执行任务，如果任务执行失败就进入等待。方法本身是很简单的，需要注意的是对不同的任务类型分两种情况:

- 如果我们的任务为 CountedCompleter 类型的任务，则调用externalHelpComplete方法来执行任务。

- 其他类型的 ForkJoinTask 任务调用tryExternalUnpush来执行，源码如下:

  ```java
  //为外部提交者提供 tryUnpush 功能(给定任务在top位时弹出任务)
  final boolean tryExternalUnpush(ForkJoinTask<?> task) {
      WorkQueue[] ws;
      WorkQueue w;
      ForkJoinTask<?>[] a;
      int m, s;
      int r = ThreadLocalRandom.getProbe();
      if ((ws = workQueues) != null && (m = ws.length - 1) >= 0 &&
              (w = ws[m & r & SQMASK]) != null &&
              (a = w.array) != null && (s = w.top) != w.base) {
          long j = (((a.length - 1) & (s - 1)) << ASHIFT) + ABASE;  //取top位任务
          if (U.compareAndSwapInt(w, QLOCK, 0, 1)) {  //加锁
              if (w.top == s && w.array == a &&
                      U.getObject(a, j) == task &&
                      U.compareAndSwapObject(a, j, task, null)) {  //符合条件，弹出
                  U.putOrderedInt(w, QTOP, s - 1);  //更新top
                  U.putOrderedInt(w, QLOCK, 0); //解锁，返回true
                  return true;
              }
              U.compareAndSwapInt(w, QLOCK, 1, 0);  //当前任务不在top位，解锁返回false
          }
      }
      return false;
  }
    
          
      
  ```

  tryExternalUnpush的作用就是判断当前任务是否在top位，如果是则弹出任务，然后在externalAwaitDone中调用doExec()执行任务。

##### ForkJoinPool.awaitJoin()

```java
final int awaitJoin(WorkQueue w, ForkJoinTask<?> task, long deadline) {
    int s = 0;
    if (task != null && w != null) {
        ForkJoinTask<?> prevJoin = w.currentJoin;  //获取给定Worker的join任务
        U.putOrderedObject(w, QCURRENTJOIN, task);  //把currentJoin替换为给定任务
        //判断是否为CountedCompleter类型的任务
        CountedCompleter<?> cc = (task instanceof CountedCompleter) ?
                (CountedCompleter<?>) task : null;
        for (; ; ) {
            if ((s = task.status) < 0)  //已经完成|取消|异常 跳出循环
                break;

            if (cc != null)//CountedCompleter任务由helpComplete来完成join
                helpComplete(w, cc, 0);
            else if (w.base == w.top || w.tryRemoveAndExec(task))  //尝试执行
                helpStealer(w, task);  //队列为空或执行失败，任务可能被偷，帮助偷取者执行该任务

            if ((s = task.status) < 0) //已经完成|取消|异常，跳出循环
                break;
            //计算任务等待时间
            long ms, ns;
            if (deadline == 0L)
                ms = 0L;
            else if ((ns = deadline - System.nanoTime()) <= 0L)
                break;
            else if ((ms = TimeUnit.NANOSECONDS.toMillis(ns)) <= 0L)
                ms = 1L;

            if (tryCompensate(w)) {//执行补偿操作
                task.internalWait(ms);//补偿执行成功，任务等待指定时间
                U.getAndAddLong(this, CTL, AC_UNIT);//更新活跃线程数
            }
        }
        U.putOrderedObject(w, QCURRENTJOIN, prevJoin);//循环结束，替换为原来的join任务
    }
    return s;
}     
```

说明:  如果当前 join 任务不在Worker等待队列的top位，或者任务执行失败，调用此方法来帮助执行或阻塞当前 join 的任务。函数执行流程如下:

- 由于每次调用awaitJoin都会优先执行当前join的任务，所以首先会更新currentJoin为当前join任务；

- 进入自旋:

  - 首先检查任务是否已经完成(通过task.status < 0判断)，如果给定任务执行完毕|取消|异常 则跳出循环返回执行状态s；
  - 如果是 CountedCompleter 任务类型，调用helpComplete方法来完成join操作(后面笔者会开新篇来专门讲解CountedCompleter，本篇暂时不做详细解析)；
  - 非 CountedCompleter 任务类型调用WorkQueue.tryRemoveAndExec尝试执行任务；
  - 如果给定 WorkQueue 的等待队列为空或任务执行失败，说明任务可能被偷，调用helpStealer帮助偷取者执行任务(也就是说，偷取者帮我执行任务，我去帮偷取者执行它的任务)；
  - 再次判断任务是否执行完毕(task.status < 0)，如果任务执行失败，计算一个等待时间准备进行补偿操作；
  - 调用tryCompensate方法为给定 WorkQueue 尝试执行补偿操作。在执行补偿期间，如果发现 资源争用|池处于unstable状态|当前Worker已终止，则调用ForkJoinTask.internalWait()方法等待指定的时间，任务唤醒之后继续自旋，ForkJoinTask.internalWait()源码如下:

  ```java
  final void internalWait(long timeout) {
      int s;
      if ((s = status) >= 0 && // force completer to issue notify
          U.compareAndSwapInt(this, STATUS, s, s | SIGNAL)) {//更新任务状态为SIGNAL(等待唤醒)
          synchronized (this) {
              if (status >= 0)
                  try { wait(timeout); } catch (InterruptedException ie) { }
              else
                  notifyAll();
          }
      }
  }  
  ```

在awaitJoin中，我们总共调用了三个比较复杂的方法: tryRemoveAndExec、helpStealer和tryCompensate，下面我们依次讲解。

##### WorkQueue.tryRemoveAndExec(ForkJoinTask<?> task)

```java
final boolean tryRemoveAndExec(ForkJoinTask<?> task) {
    ForkJoinTask<?>[] a;
    int m, s, b, n;
    if ((a = array) != null && (m = a.length - 1) >= 0 &&
            task != null) {
        while ((n = (s = top) - (b = base)) > 0) {
            //从top往下自旋查找
            for (ForkJoinTask<?> t; ; ) {      // traverse from s to b
                long j = ((--s & m) << ASHIFT) + ABASE;//计算任务索引
                if ((t = (ForkJoinTask<?>) U.getObject(a, j)) == null) //获取索引到的任务
                    return s + 1 == top;     // shorter than expected
                else if (t == task) { //给定任务为索引任务
                    boolean removed = false;
                    if (s + 1 == top) {      // pop
                        if (U.compareAndSwapObject(a, j, task, null)) { //弹出任务
                            U.putOrderedInt(this, QTOP, s); //更新top
                            removed = true;
                        }
                    } else if (base == b)      // replace with proxy
                        removed = U.compareAndSwapObject(
                                a, j, task, new EmptyTask()); //join任务已经被移除，替换为一个占位任务
                    if (removed)
                        task.doExec(); //执行
                    break;
                } else if (t.status < 0 && s + 1 == top) { //给定任务不是top任务
                    if (U.compareAndSwapObject(a, j, t, null)) //弹出任务
                        U.putOrderedInt(this, QTOP, s);//更新top
                    break;                  // was cancelled
                }
                if (--n == 0) //遍历结束
                    return false;
            }
            if (task.status < 0) //任务执行完毕
                return false;
        }
    }
    return true;
}
  
        
    
```

说明:  从top位开始自旋向下找到给定任务，如果找到把它从当前 Worker 的任务队列中移除并执行它。注意返回的参数: 如果任务队列为空或者任务未执行完毕返回true；任务执行完毕返回false。

##### ForkJoinPool.helpStealer(WorkQueue w, ForkJoinTask<?> task)

```java
private void helpStealer(WorkQueue w, ForkJoinTask<?> task) {
    WorkQueue[] ws = workQueues;
    int oldSum = 0, checkSum, m;
    if (ws != null && (m = ws.length - 1) >= 0 && w != null &&
            task != null) {
        do {                                       // restart point
            checkSum = 0;                          // for stability check
            ForkJoinTask<?> subtask;
            WorkQueue j = w, v;                    // v is subtask stealer
            descent:
            for (subtask = task; subtask.status >= 0; ) {
                //1. 找到给定WorkQueue的偷取者v
                for (int h = j.hint | 1, k = 0, i; ; k += 2) {//跳两个索引，因为Worker在奇数索引位
                    if (k > m)                     // can't find stealer
                        break descent;
                    if ((v = ws[i = (h + k) & m]) != null) {
                        if (v.currentSteal == subtask) {//定位到偷取者
                            j.hint = i;//更新stealer索引
                            break;
                        }
                        checkSum += v.base;
                    }
                }
                //2. 帮助偷取者v执行任务
                for (; ; ) {                         // help v or descend
                    ForkJoinTask<?>[] a;            //偷取者内部的任务
                    int b;
                    checkSum += (b = v.base);
                    ForkJoinTask<?> next = v.currentJoin;//获取偷取者的join任务
                    if (subtask.status < 0 || j.currentJoin != subtask ||
                            v.currentSteal != subtask) // stale
                        break descent; // stale，跳出descent循环重来
                    if (b - v.top >= 0 || (a = v.array) == null) {
                        if ((subtask = next) == null)   //偷取者的join任务为null，跳出descent循环
                            break descent;
                        j = v;
                        break; //偷取者内部任务为空，可能任务也被偷走了；跳出本次循环，查找偷取者的偷取者
                    }
                    int i = (((a.length - 1) & b) << ASHIFT) + ABASE;//获取base偏移地址
                    ForkJoinTask<?> t = ((ForkJoinTask<?>)
                            U.getObjectVolatile(a, i));//获取偷取者的base任务
                    if (v.base == b) {
                        if (t == null)             // stale
                            break descent; // stale，跳出descent循环重来
                        if (U.compareAndSwapObject(a, i, t, null)) {//弹出任务
                            v.base = b + 1;         //更新偷取者的base位
                            ForkJoinTask<?> ps = w.currentSteal;//获取调用者偷来的任务
                            int top = w.top;
                            //首先更新给定workQueue的currentSteal为偷取者的base任务，然后执行该任务
                            //然后通过检查top来判断给定workQueue是否有自己的任务，如果有，
                            // 则依次弹出任务(LIFO)->更新currentSteal->执行该任务(注意这里是自己偷自己的任务执行)
                            do {
                                U.putOrderedObject(w, QCURRENTSTEAL, t);
                                t.doExec();        // clear local tasks too
                            } while (task.status >= 0 &&
                                    w.top != top && //内部有自己的任务，依次弹出执行
                                    (t = w.pop()) != null);
                            U.putOrderedObject(w, QCURRENTSTEAL, ps);//还原给定workQueue的currentSteal
                            if (w.base != w.top)//给定workQueue有自己的任务了，帮助结束，返回
                                return;            // can't further help
                        }
                    }
                }
            }
        } while (task.status >= 0 && oldSum != (oldSum = checkSum));
    }
}
```

说明:  如果队列为空或任务执行失败，说明任务可能被偷，调用此方法来帮助偷取者执行任务。基本思想是: 偷取者帮助我执行任务，我去帮助偷取者执行它的任务。 函数执行流程如下:

循环定位偷取者，由于Worker是在奇数索引位，所以每次会跳两个索引位。定位到偷取者之后，更新调用者 WorkQueue 的hint为偷取者的索引，方便下次定位； 定位到偷取者后，开始帮助偷取者执行任务。从偷取者的base索引开始，每次偷取一个任务执行。在帮助偷取者执行任务后，如果调用者发现本身已经有任务(w.top != top)，则依次弹出自己的任务(LIFO顺序)并执行(也就是说自己偷自己的任务执行)。

##### ForkJoinPool.tryCompensate(WorkQueue w)

```java
//执行补偿操作: 尝试缩减活动线程量，可能释放或创建一个补偿线程来准备阻塞
private boolean tryCompensate(WorkQueue w) {
    boolean canBlock;
    WorkQueue[] ws;
    long c;
    int m, pc, sp;
    if (w == null || w.qlock < 0 ||           // caller terminating
            (ws = workQueues) == null || (m = ws.length - 1) <= 0 ||
            (pc = config & SMASK) == 0)           // parallelism disabled
        canBlock = false; //调用者已终止
    else if ((sp = (int) (c = ctl)) != 0)      // release idle worker
        canBlock = tryRelease(c, ws[sp & m], 0L);//唤醒等待的工作线程
    else {//没有空闲线程
        int ac = (int) (c >> AC_SHIFT) + pc; //活跃线程数
        int tc = (short) (c >> TC_SHIFT) + pc;//总线程数
        int nbusy = 0;                        // validate saturation
        for (int i = 0; i <= m; ++i) {        // two passes of odd indices
            WorkQueue v;
            if ((v = ws[((i << 1) | 1) & m]) != null) {//取奇数索引位
                if ((v.scanState & SCANNING) != 0)//没有正在运行任务，跳出
                    break;
                ++nbusy;//正在运行任务，添加标记
            }
        }
        if (nbusy != (tc << 1) || ctl != c)
            canBlock = false;                 // unstable or stale
        else if (tc >= pc && ac > 1 && w.isEmpty()) {//总线程数大于并行度 && 活动线程数大于1 && 调用者任务队列为空，不需要补偿
            long nc = ((AC_MASK & (c - AC_UNIT)) |
                    (~AC_MASK & c));       // uncompensated
            canBlock = U.compareAndSwapLong(this, CTL, c, nc);//更新活跃线程数
        } else if (tc >= MAX_CAP ||
                (this == common && tc >= pc + commonMaxSpares))//超出最大线程数
            throw new RejectedExecutionException(
                    "Thread limit exceeded replacing blocked worker");
        else {                                // similar to tryAddWorker
            boolean add = false;
            int rs;      // CAS within lock
            long nc = ((AC_MASK & c) |
                    (TC_MASK & (c + TC_UNIT)));//计算总线程数
            if (((rs = lockRunState()) & STOP) == 0)
                add = U.compareAndSwapLong(this, CTL, c, nc);//更新总线程数
            unlockRunState(rs, rs & ~RSLOCK);
            //运行到这里说明活跃工作线程数不足，需要创建一个新的工作线程来补偿
            canBlock = add && createWorker(); // throws on exception
        }
    }
    return canBlock;
}
  
        
    
```

说明:  具体的执行看源码及注释，这里我们简单总结一下需要和不需要补偿的几种情况:

**需要补偿** :

- 调用者队列不为空，并且有空闲工作线程，这种情况会唤醒空闲线程(调用tryRelease方法)
- 池尚未停止，活跃线程数不足，这时会新建一个工作线程(调用createWorker方法)

**不需要补偿** :

- 调用者已终止或池处于不稳定状态
- 总线程数大于并行度 && 活动线程数大于1 && 调用者任务队列为空

### Fork/Join的陷阱与注意事项

使用Fork/Join框架时，需要注意一些陷阱, 在下面 `斐波那契数列`例子中你将看到示例:

#### 避免不必要的fork()

划分成两个子任务后，不要同时调用两个子任务的fork()方法。

表面上看上去两个子任务都fork()，然后join()两次似乎更自然。但事实证明，直接调用compute()效率更高。因为直接调用子任务的compute()方法实际上就是在当前的工作线程进行了计算(线程重用)，这比“将子任务提交到工作队列，线程又从工作队列中拿任务”快得多。

> 当一个大任务被划分成两个以上的子任务时，尽可能使用前面说到的三个衍生的invokeAll方法，因为使用它们能避免不必要的fork()。

#### 注意fork()、compute()、join()的顺序

为了两个任务并行，三个方法的调用顺序需要万分注意。

```java
right.fork(); // 计算右边的任务
long leftAns = left.compute(); // 计算左边的任务(同时右边任务也在计算)
long rightAns = right.join(); // 等待右边的结果
return leftAns + rightAns;  
```

如果我们写成:

```java
left.fork(); // 计算完左边的任务
long leftAns = left.join(); // 等待左边的计算结果
long rightAns = right.compute(); // 再计算右边的任务
return leftAns + rightAns;
```

或者

```java
long rightAns = right.compute(); // 计算完右边的任务
left.fork(); // 再计算左边的任务
long leftAns = left.join(); // 等待左边的计算结果
return leftAns + rightAns;
```

这两种实际上都没有并行。

#### 选择合适的子任务粒度

选择划分子任务的粒度(顺序执行的阈值)很重要，因为使用Fork/Join框架并不一定比顺序执行任务的效率高: 如果任务太大，则无法提高并行的吞吐量；如果任务太小，子任务的调度开销可能会大于并行计算的性能提升，我们还要考虑创建子任务、fork()子任务、线程调度以及合并子任务处理结果的耗时以及相应的内存消耗。

官方文档给出的粗略经验是: 任务应该执行`100~10000`个基本的计算步骤。决定子任务的粒度的最好办法是实践，通过实际测试结果来确定这个阈值才是“上上策”。

> 和其他Java代码一样，Fork/Join框架测试时需要“预热”或者说执行几遍才会被JIT(Just-in-time)编译器优化，所以测试性能之前跑几遍程序很重要。

#### 避免重量级任务划分与结果合并

Fork/Join的很多使用场景都用到数组或者List等数据结构，子任务在某个分区中运行，最典型的例子如并行排序和并行查找。拆分子任务以及合并处理结果的时候，应该尽量避免System.arraycopy这样耗时耗空间的操作，从而最小化任务的处理开销。

### 再深入理解

#### 有哪些JDK源码中使用了Fork/Join思想?

我们常用的数组工具类 Arrays 在JDK 8之后新增的并行排序方法(parallelSort)就运用了 ForkJoinPool 的特性，还有 ConcurrentHashMap 在JDK 8之后添加的函数式方法(如forEach等)也有运用。

#### 使用Executors工具类创建ForkJoinPool

Java8在Executors工具类中新增了两个工厂方法:

```java
// parallelism定义并行级别
public static ExecutorService newWorkStealingPool(int parallelism);
// 默认并行级别为JVM可用的处理器个数
// Runtime.getRuntime().availableProcessors()
public static ExecutorService newWorkStealingPool();
```

#### 关于Fork/Join异常处理

Java的受检异常机制一直饱受诟病，所以在ForkJoinTask的invoke()、join()方法及其衍生方法中都没有像get()方法那样抛出个ExecutionException的受检异常。

所以你可以在ForkJoinTask中看到内部把受检异常转换成了运行时异常。

```java
static void rethrow(Throwable ex) {
    if (ex != null)
        ForkJoinTask.<RuntimeException>uncheckedThrow(ex);
}

@SuppressWarnings("unchecked")
static <T extends Throwable> void uncheckedThrow(Throwable t) throws T {
    throw (T)t; // rely on vacuous cast
}  
```

关于Java你不知道的10件事中已经指出，JVM实际并不关心这个异常是受检异常还是运行时异常，受检异常这东西完全是给Java编译器用的: 用于警告程序员这里有个异常没有处理。

但不可否认的是invoke、join()仍可能会抛出运行时异常，所以ForkJoinTask还提供了两个不提取结果和异常的方法quietlyInvoke()、quietlyJoin()，这两个方法允许你在所有任务完成后对结果和异常进行处理。

使用quitelyInvoke()和quietlyJoin()时可以配合isCompletedAbnormally()和isCompletedNormally()方法使用。

### 一些Fork/Join例子

#### 采用Fork/Join来异步计算1+2+3+…+10000的结果

```java
public class Test {
	static final class SumTask extends RecursiveTask<Integer> {
		private static final long serialVersionUID = 1L;
		
		final int start; //开始计算的数
		final int end; //最后计算的数
		
		SumTask(int start, int end) {
			this.start = start;
			this.end = end;
		}

		@Override
		protected Integer compute() {
			//如果计算量小于1000，那么分配一个线程执行if中的代码块，并返回执行结果
			if(end - start < 1000) {
				System.out.println(Thread.currentThread().getName() + " 开始执行: " + start + "-" + end);
				int sum = 0;
				for(int i = start; i <= end; i++)
					sum += i;
				return sum;
			}
			//如果计算量大于1000，那么拆分为两个任务
			SumTask task1 = new SumTask(start, (start + end) / 2);
			SumTask task2 = new SumTask((start + end) / 2 + 1, end);
			//执行任务
			task1.fork();
			task2.fork();
			//获取任务执行的结果
			return task1.join() + task2.join();
		}
	}
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ForkJoinPool pool = new ForkJoinPool();
		ForkJoinTask<Integer> task = new SumTask(1, 10000);
		pool.submit(task);
		System.out.println(task.get());
	}
}
```

- 执行结果

```java
ForkJoinPool-1-worker-1 开始执行: 1-625
ForkJoinPool-1-worker-7 开始执行: 6251-6875
ForkJoinPool-1-worker-6 开始执行: 5626-6250
ForkJoinPool-1-worker-10 开始执行: 3751-4375
ForkJoinPool-1-worker-13 开始执行: 2501-3125
ForkJoinPool-1-worker-8 开始执行: 626-1250
ForkJoinPool-1-worker-11 开始执行: 5001-5625
ForkJoinPool-1-worker-3 开始执行: 7501-8125
ForkJoinPool-1-worker-14 开始执行: 1251-1875
ForkJoinPool-1-worker-4 开始执行: 9376-10000
ForkJoinPool-1-worker-8 开始执行: 8126-8750
ForkJoinPool-1-worker-0 开始执行: 1876-2500
ForkJoinPool-1-worker-12 开始执行: 4376-5000
ForkJoinPool-1-worker-5 开始执行: 8751-9375
ForkJoinPool-1-worker-7 开始执行: 6876-7500
ForkJoinPool-1-worker-1 开始执行: 3126-3750
50005000
```

#### 实现斐波那契数列

> 斐波那契数列: 1、1、2、3、5、8、13、21、34、…… 公式 : F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)(n>=3，n∈N*)

```java
public static void main(String[] args) {
    ForkJoinPool forkJoinPool = new ForkJoinPool(4); // 最大并发数4
    Fibonacci fibonacci = new Fibonacci(20);
    long startTime = System.currentTimeMillis();
    Integer result = forkJoinPool.invoke(fibonacci);
    long endTime = System.currentTimeMillis();
    System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
}
//以下为官方API文档示例
static  class Fibonacci extends RecursiveTask<Integer> {
    final int n;
    Fibonacci(int n) {
        this.n = n;
    }
    @Override
    protected Integer compute() {
        if (n <= 1) {
            return n;
        }
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork(); 
        Fibonacci f2 = new Fibonacci(n - 2);
        return f2.compute() + f1.join(); 
    }
}
```

当然你也可以两个任务都fork，要注意的是两个任务都fork的情况，必须按照f1.fork()，f2.fork()， f2.join()，f1.join()这样的顺序，不然有性能问题，详见上面注意事项中的说明。

官方API文档是这样写到的，所以平日用invokeAll就好了。invokeAll会把传入的任务的第一个交给当前线程来执行，其他的任务都fork加入工作队列，这样等于利用当前线程也执行任务了。

```java
{
    // ...
    Fibonacci f1 = new Fibonacci(n - 1);
    Fibonacci f2 = new Fibonacci(n - 2);
    invokeAll(f1,f2);
    return f2.join() + f1.join();
}

public static void invokeAll(ForkJoinTask<?>... tasks) {
    Throwable ex = null;
    int last = tasks.length - 1;
    for (int i = last; i >= 0; --i) {
        ForkJoinTask<?> t = tasks[i];
        if (t == null) {
            if (ex == null)
                ex = new NullPointerException();
        }
        else if (i != 0)   //除了第一个都fork
            t.fork();
        else if (t.doInvoke() < NORMAL && ex == null)  //留一个自己执行
            ex = t.getException();
    }
    for (int i = 1; i <= last; ++i) {
        ForkJoinTask<?> t = tasks[i];
        if (t != null) {
            if (ex != null)
                t.cancel(false);
            else if (t.doJoin() < NORMAL)
                ex = t.getException();
        }
    }
    if (ex != null)
        rethrow(ex);
} 
```



### 课后问题

- Fork/Join主要用来解决什么样的问题?
- Fork/Join框架是在哪个JDK版本中引入的?
- Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?
- ForkJoinPool类继承关系?
- ForkJoinTask抽象类继承关系? 在实际运用中，我们一般都会继承 RecursiveTask 、RecursiveAction 或 CountedCompleter 来实现我们的业务需求，而不会直接继承 ForkJoinTask 类。
- 整个Fork/Join 框架的执行流程/运行机制是怎么样的?
- 具体阐述Fork/Join的分治思想和work-stealing 实现方式?
- 有哪些JDK源码中使用了Fork/Join思想?
- 如何使用Executors工具类创建ForkJoinPool?
- 写一个例子: 用ForkJoin方式实现1+2+3+...+100000?
- Fork/Join在使用时有哪些注意事项? 结合JDK中的斐波那契数列实例具体说明。



### 参考文章

- 首先推荐阅读ForkJoinPool的作者Doug Lea的一篇文章《A Java Fork/Join Framework》[英文原文地址  (opens new window)](http://gee.cs.oswego.edu/dl/papers/fj.pdf)
- 本文主要参考自泰迪的bagwell的https://www.jianshu.com/p/32a15ef2f1bf和https://www.jianshu.com/p/6a14d0b54b8d，在此基础上参考了如下文章
- https://blog.csdn.net/u010841296/article/details/83963637
- https://blog.csdn.net/Holmofy/article/details/82714665
- https://blog.csdn.net/abc123lzf/article/details/82873181
- https://blog.csdn.net/yinwenjie/article/details/71524140
- https://blog.csdn.net/cowbin2012/article/details/89791757