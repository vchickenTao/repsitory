**<span style="font-size: 35px;">🍑 Java IO - NIO 详解</span>**

---

> [!NOTE]   
>
> *导读*
>
> Standard IO是对字节流的读写，在进行IO之前，首先创建一个流对象，流对象进行读写操作都是按字节 ，一个字节一个字节的来读或写。而NIO把IO抽象成**块**，类似磁盘的读写，每次IO操作的单位都是一个块，块被读入内存之后就是一个byte[]，**NIO一次可以读或写多个字节**。

## 流与块

I/O 与 NIO 最重要的区别是`数据打包`和`传输`的方式，**I/O 以流的方式处理数据**，而 **NIO 以块的方式处理数据**。

面向流的 I/O 一次处理一个`字节数据`: **一个输入流产生一个字节数据，一个输出流消费一个字节数据**。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。不利的一面是，面向流的 I/O 通常相当慢。

面向块的 I/O 一次处理一个`数据块`，按块处理数据比按流处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

I/O 包和 NIO 已经很好地集成了，java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。



## 什么是NIO

- Java NIO全称`java non-blocking IO`， 是指JDK提供的新API。从JDK1.4开始，Java提供了一系列改进的输入/输出的新特性，被统称为NIO(即New IO)，是同步非阻塞的

- NIO有三大核心部分:
  - Channel(通道)
  - Buffer(缓冲区)
  - Selector(选择器)

- NIO是面向缓冲区，或者面向块编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。



## 通道

### 背景

应用程序进行读写操作调用函数时，**底层调用的操作系统提供给用户的读写API**，调用这些API时会生成对应的指令，CPU则会执行这些指令。在计算机刚出现的那段时间，**所有读写请求的指令都有CPU去执行**，过多的读写请求会导致CPU无法去执行其他命令，从而CPU的利用率降低。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/62079109-9279-47e9-89c4-5db69116bdfe_java-nio-channel-01.png)

后来，**DMA**(Direct Memory Access，直接存储器访问)出现了。当IO请求传到计算机底层时，**DMA会向CPU请求，让DMA去处理这些IO操作**，从而可以让CPU去执行其他指令。DMA处理IO操作时，会请求获取总线的使用权。**当IO请求过多时，会导致大量总线用于处理IO请求，从而降低效率** 。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/178c4bbe-e8a9-4d3b-8322-1de674c4a8ae_java-nio-channel-02.png)

于是便有了`Channel(通道)`，Channel相当于一个**专门用于IO操作的独立处理器**，它具有独立处理IO请求的能力，当有IO请求时，它会自行处理这些IO请求 。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/0f8b977f-3741-4508-8acd-e2da8538196a_java-nio-channel-03.png)



### 概念

> `通道 Channel`是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。Channel由java.nio.channels 包定义的。Channel 表示**IO 源与目标打开的连接**。Channel 类似于传统的“流”。只不过**Channel 本身不能直接访问数据，Channel 只能与Buffer 进行交互** 。

通道与流的不同之处在于，流只能在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/a057b90e-bc2e-4456-8b08-b52cfe53ca64_java-nio-channel-04.png)

通道包括以下类型:

- 本地文件IO
  - FileChannel: 从文件中读写数据；

- 网络IO

  - DatagramChannel: 通过 UDP 读写网络中数据；

  - SocketChannel: 通过 TCP 读写网络中数据；

  - ServerSocketChannel: 可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

### 获得通道的方法

#### 对象调用getChannel() 方法

获取通道的一种方式是**对支持通道的对象调用getChannel() 方法**。支持通道的类如下：

- FileInputStream
- FileOutputStream
- RandomAccessFile
- DatagramSocket
- Socket
- ServerSocket

例子：

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.net.DatagramSocket;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.channels.DatagramChannel;
import java.nio.channels.FileChannel;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.nio.file.Paths;

public class demo2 {
    public static void main(String[] args) throws IOException {
        // 本地通道
        FileInputStream fileInputStream = new FileInputStream("zwt");
        FileChannel channel1 = fileInputStream.getChannel();

        FileOutputStream fileOutputStream = new FileOutputStream("zwt");
        FileChannel channel2 = fileOutputStream.getChannel();

        // 网络通道
        Socket socket = new Socket();
        SocketChannel channel3 = socket.getChannel();

        ServerSocket serverSocket = new ServerSocket();
        ServerSocketChannel channel4 = serverSocket.getChannel();

        DatagramSocket datagramSocket = new DatagramSocket();
        DatagramChannel channel5 = datagramSocket.getChannel();

        // 最后要关闭通道

        FileChannel open = FileChannel.open(Paths.get("zwt"));

        SocketChannel open1 = SocketChannel.open();

    }
}
```

#### getChannel()+非直接缓冲区

- getChannel()获得通道
- allocate()获得**非直接缓冲区**

通过非直接缓冲区读写数据，需要通过通道来传输缓冲区里的数据

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class demo4 {
    public static void main(String[] args) {
        FileInputStream is = null;
        FileOutputStream os = null;
        // 获得通道
        FileChannel inChannel = null;
        FileChannel outChannel = null;

        // 利用 try-catch-finally 保证关闭
        try {
            is = new FileInputStream("");
            os = new FileOutputStream("");

            // 获得通道
            inChannel = is.getChannel();
            outChannel = os.getChannel();

            // 获得缓冲区，用于在通道中传输数据
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

            // 循环将字节数据放入到buffer中，然后写入磁盘中
            while (inChannel.read(byteBuffer) != -1) {
                // 切换模式
                byteBuffer.flip();
                outChannel.write(byteBuffer);
                byteBuffer.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (inChannel != null) {
                try {
                    inChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (outChannel != null) {
                try {
                    outChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (os != null) {
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### open()+直接缓冲区

- 通过open获得通道
- 通过FileChannel.map()获取直接缓冲区

使用直接缓冲区时，无需通过通道来传输数据，直接将数据放在缓冲区内即可

```java
import java.io.IOException;
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;

public class demo5 {
   public static void main(String[] args) throws IOException {
       // 通过open()方法来获得通道
       FileChannel inChannel = FileChannel.open(Paths.get(""), StandardOpenOption.READ);

       // outChannel需要为 READ WRITE CREATE模式
       // READ WRITE是因为后面获取直接缓冲区时模式为READ_WRITE模式
       // CREATE是因为要创建新的文件
       FileChannel outChannel = FileChannel.open(Paths.get(""), StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE);

       // 获得直接缓冲区
       MappedByteBuffer inMapBuf = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
       MappedByteBuffer outMapBuf = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());

       // 字节数组
       byte[] bytes = new byte[inMapBuf.limit()];

       // 因为是直接缓冲区，可以直接将数据放入到内存映射文件，无需通过通道传输
       inMapBuf.get(bytes);
       outMapBuf.put(bytes);

       // 关闭缓冲区，这里没有用try-catch-finally
       inChannel.close();
       outChannel.close();
   }
}
```

### 通道间直接传输

```java
public static void channelToChannel() throws IOException {
  long start = System.currentTimeMillis();
  // 通过open()方法来获得通道
  FileChannel inChannel = FileChannel.open(Paths.get(""), StandardOpenOption.READ);

  // outChannel需要为 READ WRITE CREATE模式
  // READ WRITE是因为后面获取直接缓冲区时模式为READ_WRITE模式
  // CREATE是因为要创建新的文件
  FileChannel outChannel = FileChannel.open(Paths.get(""), StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE);

  // 通道间直接传输
  inChannel.transferTo(0, inChannel.size(), outChannel);
  // 对应的还有transferFrom
  // outChannel.transferFrom(inChannel, 0, inChannel.size());

  inChannel.close();
  outChannel.close();
}
```





## 缓冲区

发送给一个通道的所有数据都必须首先放到缓冲区中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下类型:

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer



### 获取缓冲区

通过**allocate方法**可以获取一个对应缓冲区的对象，它是缓冲区类的一个静态方法

例：

```java
// 获取一个容量大小为1024字节的字节缓冲区
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
```



### 核心属性

缓冲区的父类Buffer中有几个核心属性，如下

```java
// Invariants: mark <= position <= limit <= capacity
private int mark = -1;
private int position = 0;
private int limit;
private int capacity;
```

- capacity：缓冲区的容量。通过构造函数赋予，一旦设置，无法更改
- limit：缓冲区的界限。位于limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量
- position：下一个读写位置的索引（类似PC）。缓冲区的位置不能为负，并且不能大于limit
- mark：记录当前position的值。position被改变后，可以通过调用reset() 方法恢复到mark的位置。

以上四个属性必须满足以下要求

> mark <= position <= limit <= capacity



### 缓冲区状态变量

状态变量的改变过程举例:

① 新建一个大小为 8 个字节的缓冲区，此时 position 为 0，而 limit = capacity = 8。capacity 变量不会改变，下面的讨论会忽略它。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-07/43570acb-c6ab-4c85-8f05-df69b06c867a_nio基础详解1.png)

② 从输入通道中读取 5 个字节数据写入缓冲区中，此时 position 移动设置为 5，limit 保持不变。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-07/0e0c51a9-0a71-47f3-8117-bce9a95c5c76_nio基础详解2.png)

③ 在将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，这个方法将 limit 设置为当前 position，并将 position 设置为 0。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-07/3e36fd03-273d-4f70-833f-8c0ca1133897_nio基础详解3.png)

④ 从缓冲区中取 4 个字节到输出缓冲中，此时 position 设为 4。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-07/7b4c0d9c-ec7b-47dc-86ee-c93f215dc299_nio基础详解4.png)

⑤ 最后需要调用 clear() 方法来清空缓冲区，此时 position 和 limit 都被设置为最初位置。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-07/50a20bcc-cb56-4c4f-89c2-822dd723ea29_nio基础详解5.png)



### 核心方法

#### put()方法

- put()方法可以将一个数据放入到缓冲区中。
- 进行该操作后，postition的值会+1，指向下一个可以放入的位置。capacity = limit ，为缓冲区容量的值。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/f4ba5e79-7055-4f49-89b6-7f1713552a57_java-nio-buffer-01.png)

#### flip()方法

- flip()方法会**切换对缓冲区的操作模式**，由写->读 / 读->写
- 进行该操作后
  - 如果是写模式->读模式，position = 0 ， limit 指向最后一个元素的下一个位置，capacity不变
  - 如果是读->写，则恢复为put()方法中的值

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/a0f6bc56-1050-474b-8d54-2022acfde521_java-nio-buffer-02.png)

#### get()方法

- get()方法会读取缓冲区中的一个值
- 进行该操作后，position会+1，如果超过了limit则会抛出异常

#### rewind()方法

- 该方法**只能在读模式下使用**
- rewind()方法后，会恢复position、limit和capacity的值，变为进行get()前的值

#### clean()方法

- clean()方法会将缓冲区中的各个属性恢复为最初的状态，position = 0, capacity = limit
- **此时缓冲区的数据依然存在**，处于“被遗忘”状态，下次进行写操作时会覆盖这些数据

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/454f3293-ce66-4a31-8c8f-901be9211253_java-nio-buffer-03.png)

#### mark()和reset()方法

- mark()方法会将postion的值保存到mark属性中
- reset()方法会将position的值改为mark中保存的值

**使用展示**

```java
import java.nio.ByteBuffer;

public class demo1 {
    public static void main(String[] args) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        System.out.println("放入前参数");
        System.out.println("position " + byteBuffer.position());
        System.out.println("limit " + byteBuffer.limit());
        System.out.println("capacity " + byteBuffer.capacity());
        System.out.println();

        System.out.println("------put()------");
        System.out.println("放入3个数据");
        byte bt = 1;
        byteBuffer.put(bt);
        byteBuffer.put(bt);
        byteBuffer.put(bt);

        System.out.println("放入后参数");
        System.out.println("position " + byteBuffer.position());
        System.out.println("limit " + byteBuffer.limit());
        System.out.println("capacity " + byteBuffer.capacity());
        System.out.println();

        System.out.println("------flip()-get()------");
        System.out.println("读取一个数据");
        // 切换模式
        byteBuffer.flip();
        byteBuffer.get();

        System.out.println("读取后参数");
        System.out.println("position " + byteBuffer.position());
        System.out.println("limit " + byteBuffer.limit());
        System.out.println("capacity " + byteBuffer.capacity());
        System.out.println();

        System.out.println("------rewind()------");
        byteBuffer.rewind();
        System.out.println("恢复后参数");
        System.out.println("position " + byteBuffer.position());
        System.out.println("limit " + byteBuffer.limit());
        System.out.println("capacity " + byteBuffer.capacity());
        System.out.println();

        System.out.println("------clear()------");
        // 清空缓冲区，这里只是恢复了各个属性的值，但是缓冲区里的数据依然存在
        // 但是下次写入的时候会覆盖缓冲区中之前的数据
        byteBuffer.clear();
        System.out.println("清空后参数");
        System.out.println("position " + byteBuffer.position());
        System.out.println("limit " + byteBuffer.limit());
        System.out.println("capacity " + byteBuffer.capacity());
        System.out.println();
        System.out.println("清空后获得数据");
        System.out.println(byteBuffer.get());

    }
}
```

**执行结果**

```java
放入前参数
position 0
limit 1024
capacity 1024

------put()------
放入3个数据
放入后参数
position 3
limit 1024
capacity 1024

------flip()-get()------
读取一个数据
读取后参数
position 1
limit 3
capacity 1024

------rewind()------
恢复后参数
position 0
limit 3
capacity 1024

------clear()------
清空后参数
position 0
limit 1024
capacity 1024

清空后获得数据
1

Process finished with exit code 0
```



### 非直接缓冲区和直接缓冲区

#### 非直接缓冲区

通过**allocate()方法获取的缓冲区都是非直接缓冲区。这些缓冲区是建立在JVM堆内存**之中的。

```java
public static ByteBuffer allocate(int capacity) {
   if (capacity < 0)
   throw new IllegalArgumentException();

   // 在堆内存中开辟空间
   return new HeapByteBuffer(capacity, capacity);
}

HeapByteBuffer(int cap, int lim) {        // package-private
   // new byte[cap] 创建数组，在堆内存中开辟空间
   super(-1, 0, lim, cap, new byte[cap], 0);
   /*
   hb = new byte[cap];
   offset = 0;
   */
}
```


通过非直接缓冲区，想要将数据写入到物理磁盘中，或者是从物理磁盘读取数据。**都需要经过JVM和操作系统**，数据在两个地址空间中传输时，会copy一份保存在对方的空间中。所以非直接缓冲区的读取效率较低.。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/d923d8d8-cc0a-4e24-8c98-46ae3974ecc3_java-nio-buffer-04.png)

#### 直接缓冲区

**只有ByteBuffer可以获得直接缓冲区**，通过allocateDirect()获取的缓冲区为直接缓冲区，这些缓冲区是建立在**物理内存**之中的。

```java
public static ByteBuffer allocateDirect(int capacity) {
   return new DirectByteBuffer(capacity);
}

DirectByteBuffer(int cap) {                   // package-private
   ...
   // 申请物理内存
   boolean pa = VM.isDirectMemoryPageAligned();
   ...
}
```

直接缓冲区通过在操作系统和JVM之间**创建物理内存映射文件**加快缓冲区数据读/写入物理磁盘的速度。放到物理内存映射文件中的数据就不归应用程序控制了，操作系统会自动将物理内存映射文件中的数据写入到物理内存中。

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/bc95081c-4bec-4aae-80a9-33012710422c_java-nio-buffer-05.png)

#### 直接缓冲区VS非直接缓冲区

```java
// getChannel() + 非直接缓冲区耗时
708
// open() + 直接缓冲区耗时
115
// channel transferTo channel耗时
47

// 直接缓冲区的读写速度虽然很快，但是会占用很多很多内存空间。如果文件过大，会使得计算机运行速度变慢
```



### 分散和聚集

#### 分散读取

分散读取（Scattering Reads）是指**从Channel 中读取的数据“分散”到多个Buffer 中**。

注意：按照缓冲区的顺序，从Channel 中读取的数据依次将 Buffer 填满。

#### 聚集写入

聚集写入（Gathering Writes）是指**将多个Buffer 中的数据“聚集”到Channel**。

按照缓冲区的顺序，写入position 和limit 之间的数据到Channel。

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class demo6 {
    public static void main(String[] args) throws IOException {
        FileInputStream is = new FileInputStream("");
        FileOutputStream os = new FileOutputStream("");

        FileChannel inChannel = is.getChannel();
        FileChannel outChannel = os.getChannel();

        // 获得多个缓冲区，并且放入到缓冲区数组中
        ByteBuffer byteBuffer1 = ByteBuffer.allocate(50);
        ByteBuffer byteBuffer2 = ByteBuffer.allocate(1024);
        ByteBuffer[] byteBuffers = {byteBuffer1, byteBuffer2};

        // 分散读取
        inChannel.read(byteBuffers);

        byteBuffer1.flip();
        byteBuffer2.flip();

        // 聚集写入
        outChannel.write(byteBuffers);
    }
}
```



### 文件 NIO 实例

以下展示了使用 NIO 快速复制文件的实例:

```java
public static void fastCopy(String src, String dist) throws IOException {

    /* 获得源文件的输入字节流 */
    FileInputStream fin = new FileInputStream(src);

    /* 获取输入字节流的文件通道 */
    FileChannel fcin = fin.getChannel();

    /* 获取目标文件的输出字节流 */
    FileOutputStream fout = new FileOutputStream(dist);

    /* 获取输出字节流的通道 */
    FileChannel fcout = fout.getChannel();

    /* 为缓冲区分配 1024 个字节 */
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

    while (true) {

        /* 从输入通道中读取数据到缓冲区中 */
        int r = fcin.read(buffer);

        /* read() 返回 -1 表示 EOF */
        if (r == -1) {
            break;
        }

        /* 切换读写 */
        buffer.flip();

        /* 把缓冲区的内容写入输出文件中 */
        fcout.write(buffer);
        
        /* 清空缓冲区 */
        buffer.clear();
    }
}
```



## 选择器

NIO 常常被叫做非阻塞 IO，主要是因为 NIO 在网络通信中的非阻塞特性被广泛使用。

NIO 实现了 IO 多路复用中的 Reactor 模型，一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去监听多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

通过配置监听的通道 Channel 为非阻塞，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它 Channel，找到 IO 事件已经到达的 Channel 执行。

因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件具有更好的性能。

应该注意的是，只有套接字 Channel 才能配置为非阻塞，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

![image](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-07/3a2644d1-488e-4203-8216-5a1501024389_nio基础详解6.png)

#### 1. 创建选择器

```java
Selector selector = Selector.open();
```

#### 2. 将通道注册到选择器上

```java
ServerSocketChannel ssChannel = ServerSocketChannel.open();
ssChannel.configureBlocking(false);
ssChannel.register(selector, SelectionKey.OP_ACCEPT);
```

通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它事件，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。

在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类:

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE

它们在 SelectionKey 的定义如下:

```java
public static final int OP_READ = 1 << 0;
public static final int OP_WRITE = 1 << 2;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_ACCEPT = 1 << 4;
```

可以看出每个事件可以被当成一个位域，从而组成事件集整数。例如:

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

#### 3. 监听事件

```java
int num = selector.select();
```

使用 select() 来监听到达的事件，它会一直阻塞直到有至少一个事件到达。

#### 4. 获取到达的事件

```java
Set<SelectionKey> keys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = keys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if (key.isAcceptable()) {
        // ...
    } else if (key.isReadable()) {
        // ...
    }
    keyIterator.remove();
}
```

#### 5. 事件循环

因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。

```java
while (true) {
    int num = selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = keys.iterator();
    while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if (key.isAcceptable()) {
            // ...
        } else if (key.isReadable()) {
            // ...
        }
        keyIterator.remove();
    }
} 
```

## 套接字 NIO 实例

```java
public class NIOServer {

    public static void main(String[] args) throws IOException {

        Selector selector = Selector.open();

        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        ssChannel.configureBlocking(false);
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);

        ServerSocket serverSocket = ssChannel.socket();
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
        serverSocket.bind(address);

        while (true) {

            selector.select();
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = keys.iterator();

            while (keyIterator.hasNext()) {

                SelectionKey key = keyIterator.next();

                if (key.isAcceptable()) {

                    ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();

                    // 服务器会为每个新连接创建一个 SocketChannel
                    SocketChannel sChannel = ssChannel1.accept();
                    sChannel.configureBlocking(false);

                    // 这个新连接主要用于从客户端读取数据
                    sChannel.register(selector, SelectionKey.OP_READ);

                } else if (key.isReadable()) {

                    SocketChannel sChannel = (SocketChannel) key.channel();
                    System.out.println(readDataFromSocketChannel(sChannel));
                    sChannel.close();
                }

                keyIterator.remove();
            }
        }
    }

    private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {

        ByteBuffer buffer = ByteBuffer.allocate(1024);
        StringBuilder data = new StringBuilder();

        while (true) {

            buffer.clear();
            int n = sChannel.read(buffer);
            if (n == -1) {
                break;
            }
            buffer.flip();
            int limit = buffer.limit();
            char[] dst = new char[limit];
            for (int i = 0; i < limit; i++) {
                dst[i] = (char) buffer.get(i);
            }
            data.append(dst);
            buffer.clear();
        }
        return data.toString();
    }
}
```

```java
public class NIOClient {

    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8888);
        OutputStream out = socket.getOutputStream();
        String s = "hello world";
        out.write(s.getBytes());
        out.close();
    }
}
```



## 内存映射文件

内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。

向内存映射文件写入可能是危险的，只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

下面代码行将文件的前 1024 个字节映射到内存中，map() 方法返回一个 MappedByteBuffer，它是 ByteBuffer 的子类。因此，可以像使用其他任何 ByteBuffer 一样使用新映射的缓冲区，操作系统会在需要时负责执行映射。

```java
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
```



## BIO与NIO对比

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-10-09/53672474-5e84-43db-8c35-a15947b41783_java-nio-bio-vs-nio.png)

NIO 与BIO 的区别主要有以下几点:

- NIO 是非阻塞的，BIO是阻塞的；
- NIO 面向块，I/O 面向流，块I/O 的效率比流I/O高很多；
- BIO基于`字节流`和`字符流`进行操作，而NIO 基于`Channel(通道)`和`Buffer(缓冲区)`进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择器)用于监听多个通道的事件(比如:连接请求，数据到达等)，因此使用**单个线程就可以监听多个客户端通道**。



## 参考文章

- [Java 全栈知识体系](https://www.pdai.tech/md/java/io/java-io-nio.html)

- [华为二面！！！面试官直接问我Java中到底什么是NIO？这不是直接送分题？？？](https://blog.csdn.net/wj1314250/article/details/120653293?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166532001916800184117685%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166532001916800184117685&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-4-120653293-null-null.142^v52^control,201^v3^control&utm_term=nio&spm=1018.2226.3001.4187)

  