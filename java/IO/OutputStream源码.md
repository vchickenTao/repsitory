**<span style="font-size: 35px;">🍌 Java IO OutputStream源码</span>**

---

> OutputStream是输出字节流，具体的实现类层次结构如下：

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-06/76abdc28-6236-4853-8945-3e8ff748284b_io-outputstream-1.png)

## OutputStream 抽象类

OutputStream 类重要方法设计如下：

```java
// 写入一个字节，可以看到这里的参数是一个 int 类型，对应上面的读方法，int 类型的 32 位，只有低 8 位才写入，高 24 位将舍弃。
public abstract void write(int b)

// 将数组中的所有字节写入，实际调用的是write(byte b[], int off, int len)方法。
public void write(byte b[])

// 将 byte 数组从 off 位置开始，len 长度的字节写入
public void write(byte b[], int off, int len)

// 强制刷新，将缓冲中的数据写入; 默认是空实现，供子类覆盖
public void flush()

// 关闭输出流，流被关闭后就不能再输出数据了; 默认是空实现，供子类覆盖
public void close()
```

## 源码实现

> 梳理部分OutputStream及其实现类的源码分析。

### OutputStream

OutputStream抽象类源码如下：

```java
public abstract class OutputStream implements Closeable, Flushable {
    
    // JDK11中增加了一个nullOutputStream，即空模式实现，以便可以直接调用而不用判空（可以看如下的补充说明）
    public static OutputStream nullOutputStream() {
        return new OutputStream() {
            private volatile boolean closed;

            private void ensureOpen() throws IOException {
                if (closed) {
                    throw new IOException("Stream closed");
                }
            }

            @Override
            public void write(int b) throws IOException {
                ensureOpen();
            }

            @Override
            public void write(byte b[], int off, int len) throws IOException {
                Objects.checkFromIndexSize(off, len, b.length);
                ensureOpen();
            }

            @Override
            public void close() {
                closed = true;
            }
        };
    }

    // 写入一个字节，可以看到这里的参数是一个 int 类型，对应上面的读方法，int 类型的 32 位，只有低 8 位才写入，高 24 位将舍弃。
    public abstract void write(int b) throws IOException;

    // 将数组中的所有字节写入，实际调用的是write(byte b[], int off, int len)方法
    public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
    }

    // 将 byte 数组从 off 位置开始，len 长度的字节写入
    public void write(byte b[], int off, int len) throws IOException {
        // 检查边界合理性
        Objects.checkFromIndexSize(off, len, b.length);
        // len == 0 的情况已经在如下的for循环中隐式处理了
        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);
        }
    }

    // 强制刷新，将缓冲中的数据写入; 默认是空实现，供子类覆盖
    public void flush() throws IOException {
    }

    // 关闭输出流，流被关闭后就不能再输出数据了; 默认是空实现，供子类覆盖
    public void close() throws IOException {
    }

}
```

补充下JDK11为什么会增加nullOutputStream方法的设计？即空对象模式

- **空对象模式**

举个例子：

```java
public class MyParser implements Parser {
  private static Action NO_ACTION = new Action() {
    public void doSomething() { /* do nothing */ }
  };

  public Action findAction(String userInput) {
    // ...
    if ( /* we can't find any actions */ ) {
      return NO_ACTION;
    }
  }
}
```

然后便**可以始终可以这么调用，而不用再判断空了**

```java
ParserFactory.getParser().findAction(someInput).doSomething();
```

### FilterOutputStream

FilterOutputStream 源码如下

```java
public class FilterOutputStream extends OutputStream {
    
    // 被装饰的实际outputStream
    protected OutputStream out;

    // 当前stream是否已经被close
    private volatile boolean closed;

    // close stream时加锁，防止其它线程同时close
    private final Object closeLock = new Object();

    // 初始化构造函数，传入被装饰的实际outputStream
    public FilterOutputStream(OutputStream out) {
        this.out = out;
    }

    // 写入数据，本质调用被装饰outputStream的方法
    @Override
    public void write(int b) throws IOException {
        out.write(b);
    }

    // 将数组中的所有字节写入
    @Override
    public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
    }

    // 一个个写入
    @Override
    public void write(byte b[], int off, int len) throws IOException {
        if ((off | len | (b.length - (len + off)) | (off + len)) < 0)
            throw new IndexOutOfBoundsException();

        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);
        }
    }

     // 强制刷新，将缓冲中的数据写入; 本质调用被装饰outputStream的方法
    @Override
    public void flush() throws IOException {
        out.flush();
    }

    // 关闭Stream
    @Override
    public void close() throws IOException {
        // 如果已经close, 直接退出
        if (closed) {
            return;
        }
        // 加锁处理，如果已经有线程正在closing则退出；
        synchronized (closeLock) {
            if (closed) {
                return;
            }
            closed = true;
        }

        // close前调用flush
        Throwable flushException = null;
        try {
            flush();
        } catch (Throwable e) {
            flushException = e;
            throw e;
        } finally {
            if (flushException == null) {
                out.close();
            } else {
                try {
                    out.close();
                } catch (Throwable closeException) {
                   // evaluate possible precedence of flushException over closeException
                   if ((flushException instanceof ThreadDeath) &&
                       !(closeException instanceof ThreadDeath)) {
                       flushException.addSuppressed(closeException);
                       throw (ThreadDeath) flushException;
                   }

                    if (flushException != closeException) {
                        closeException.addSuppressed(flushException);
                    }

                    throw closeException;
                }
            }
        }
    }
}
```

对比下JDK8中，close方法是没有加锁处理的。这种情况下你可以看JDK8源码中，直接利用java7的try with resources方式，优雅的调用flush方法后对out进行关闭。

```java
public void close() throws IOException {
    try (OutputStream ostream = out) {
        flush();
    }
}
```

### ByteArrayOutputStream

ByteArrayOutputStream 源码如下

```java
public class ByteArrayOutputStream extends OutputStream {

    // 实际的byte数组
    protected byte buf[];

    // 数组中实际有效的byte的个数
    protected int count;

    // 初始化默认构造，初始化byte数组大小为32
    public ByteArrayOutputStream() {
        this(32);
    }

    // 初始化byte的大小
    public ByteArrayOutputStream(int size) {
        if (size < 0) {
            throw new IllegalArgumentException("Negative initial size: "
                                               + size);
        }
        buf = new byte[size];
    }

    // 扩容，确保它至少可以容纳由最小容量参数指定的元素数
    private void ensureCapacity(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - buf.length > 0)
            grow(minCapacity);
    }

    // 分配的最大数组大小。
    // 由于一些VM在数组中保留一些头字，所以尝试分配较大的阵列可能会导致OutOfMemoryError（请求的阵列大小超过VM限制）
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    // 扩容的实质方法，确保它至少可以容纳由最小容量参数指定的元素数
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = buf.length;
        int newCapacity = oldCapacity << 1;
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        buf = Arrays.copyOf(buf, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    // 写入，写入前确保byte数据长度
    public synchronized void write(int b) {
        ensureCapacity(count + 1);
        buf[count] = (byte) b;
        count += 1;
    }

    
    public synchronized void write(byte b[], int off, int len) {
        Objects.checkFromIndexSize(off, len, b.length);
        ensureCapacity(count + len);
        System.arraycopy(b, off, buf, count, len);
        count += len;
    }

    public void writeBytes(byte b[]) {
        write(b, 0, b.length);
    }

    public synchronized void writeTo(OutputStream out) throws IOException {
        out.write(buf, 0, count);
    }

    // 重置，显然将实际有效的byte数量置为0
    public synchronized void reset() {
        count = 0;
    }

    
    public synchronized byte[] toByteArray() {
        return Arrays.copyOf(buf, count);
    }

    // 长度，即count
    public synchronized int size() {
        return count;
    }

    // 转成string
    public synchronized String toString() {
        return new String(buf, 0, count);
    }

    // 转成string，指定的字符集
    public synchronized String toString(String charsetName)
        throws UnsupportedEncodingException
    {
        return new String(buf, 0, count, charsetName);
    }

    public synchronized String toString(Charset charset) {
        return new String(buf, 0, count, charset);
    }

    // 弃用
    @Deprecated
    public synchronized String toString(int hibyte) {
        return new String(buf, hibyte, 0, count);
    }

    // 对byte 数组而言，close没啥实质意义，所以空实现
    public void close() throws IOException {
    }

}
```

### BufferedOutputStream

BufferedOutputStream 源码如下

```java
public class BufferedOutputStream extends FilterOutputStream {
    
    // Buffered outputStream底层也是byte数组
    protected byte buf[];

    // 大小，buf[0]到buf[count-1]是实际存储的bytes
    protected int count;

    // 构造函数，被装饰的outputStream，以及默认buf大小是8192
    public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
    }

    public BufferedOutputStream(OutputStream out, int size) {
        super(out);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }

    /** Flush the internal buffer */
    // 内部的flush方法，将buffer中的有效bytes(count是有效的bytes大小)通过被装饰的outputStream写入
    private void flushBuffer() throws IOException {
        if (count > 0) {
            out.write(buf, 0, count);
            count = 0;
        }
    }

    // 写入byte
    @Override
    public synchronized void write(int b) throws IOException {
        // 当buffer满了以后，flush buffer
        if (count >= buf.length) {
            flushBuffer();
        }
        buf[count++] = (byte)b;
    }

    // 将 byte 数组从 off 位置开始，len 长度的字节写入
    @Override
    public synchronized void write(byte b[], int off, int len) throws IOException {
        if (len >= buf.length) {
            // 如果请求长度已经超过输出缓冲区的大小，直接刷新输出缓冲区，然后直接写入数据。
            flushBuffer();
            out.write(b, off, len);
            return;
        }
        if (len > buf.length - count) {
            flushBuffer();
        }
        System.arraycopy(b, off, buf, count, len);
        count += len;
    }

    // flush方法，需要先将buffer中写入，最后在调用被装饰outputStream的flush方法
    @Override
    public synchronized void flush() throws IOException {
        flushBuffer();
        out.flush();
    }
}
```