**<span style="font-size: 35px;">🍊 Java IO - 分类</span>**

---

## IO分类 — 按传输方式分类

从数据传输或运输方式的角度来看，我们可以将IO分为：

- 字节流
- 字符流

> [!TIP] 
>
> `字节`是给计算机看的，而`字符`才是给人看的

**整体结构**

![IO整体结构](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-05/cfa74381-1893-4224-821f-feb6ea88a47d_IO流.png)



### 字节流

整体结构图如下所示：

![Java IO-字节流](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-05/0b8b4015-745b-4de2-85b9-aa4f73259a71_IO-字节流.png.png)

#### 字节流各个类的详细说明

- **InputStream**

1. `InputStream`：`InputStream`是所有字节输入流的抽象基类，前面说过抽象类不能被实例化，实际上是作为模板而存在的，为所有实现类定义了处理输入流的方法。
2. `FileInputSream`：文件输入流，一个非常重要的字节输入流，用于对文件进行读取操作。
3. `PipedInputStream`：管道字节输入流，能实现多线程间的管道通信。
4. `ByteArrayInputStream`：字节数组输入流，从字节数组(byte[])中进行以字节为单位的读取，也就是将资源文件都以字节的形式存入到该类中的字节数组中去。
5. `FilterInputStream`：装饰者类，具体的装饰者继承该类，这些类都是处理类，作用是对节点类进行封装，实现一些特殊功能。
6. `DataInputStream`：数据输入流，它是用来装饰其它输入流，作用是“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”。
7. `BufferedInputStream`：缓冲流，对节点流进行装饰，内部会有一个缓存区，用来存放字节，每次都是将缓存区存满然后发送，而不是一个字节或两个字节这样发送，效率更高。
8. `ObjectInputStream`：对象输入流，用来提供对**基本数据或对象**的持久存储。通俗点说，也就是能直接传输对象，通常应用在反序列化中。它也是一种处理流，构造器的入参是一个`InputStream`的实例对象。

- **OutputStream**

`OutputStream`类继承关系与`InputStream`类似，需要注意的是`PrintStream`

### 字符流

整体结构图如下所示：

![IO-字符流](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-05/e7577b9a-a144-4606-8589-5510c28de395_IO-字符流.png)

#### 字符流各个类的详细说明

- **Reader**

1. `InputStreamReader`：从字节流到字符流的桥梁（`InputStreamReader`构造器入参是`FileInputStream`的实例对象），它读取字节并使用指定的字符集将其解码为字符。它使用的字符集可以通过名称指定，也可以显式给定，或者可以接受平台的默认字符集。
2. `BufferedReader`：从字符输入流中读取文本，设置一个缓冲区来提高效率。`BufferedReader`是对`InputStreamReader`的封装，前者构造器的入参就是后者的一个实例对象。
3. `FileReader`：用于读取字符文件的便利类，`new FileReader(File file)`等同于`new InputStreamReader(new FileInputStream(file, true),"UTF-8")`，但`FileReader`不能指定字符编码和默认字节缓冲区大小。
4. `PipedReader` ：管道字符输入流。实现多线程间的管道通信。
5. `CharArrayReader`：从`Char`数组中读取数据的介质流。
6. `StringReader` ：从`String`中读取数据的介质流。

- **Writer**

`Writer`与`Reader`结构类似，方向相反，不再赘述。唯一有区别的是，`Writer`的子类`PrintWriter`。



### 字节流和字符流的区别

- 字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码中文汉字是 3 个字节，GBK编码中文汉字是 2 个字节。)
- 字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。



### 字节转字符Input/OutputStreamReader/Writer

**编码**就是把`字符`转换为`字节`，而**解码**是把`字节`重新组合成`字符`。

如果编码和解码过程使用不同的编码方式那么就出现了乱码。

- GBK 编码中，中文字符占 2 个字节，英文字符占 1 个字节；
- UTF-8 编码中，中文字符占 3 个字节，英文字符占 1 个字节；
- UTF-16be 编码中，中文字符和英文字符都占 2 个字节。

UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。

Java 使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码是为了让一个中文或者一个英文都能使用一个 char 来存储。

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-05/dff9935a-15ef-49a1-8ba8-c5468b020b1e_java-io-1.png)



## IO分类  — 按数据操作分类

从数据来源或者说是操作对象角度看，IO 类可以分为:

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-05/50adf9aa-c6cc-451a-8849-10e348ea8138_IO-按数据操作分类.png)

### 文件(file)

FileInputStream、FileOutputStream、FileReader、FileWriter

### 数组([])

- 字节数组(byte[]): ByteArrayInputStream、ByteArrayOutputStream
- 字符数组(char[]): CharArrayReader、CharArrayWriter

### 管道操作

PipedInputStream、PipedOutputStream、PipedReader、PipedWriter

### 基本数据类型

DataInputStream、DataOutputStream

### 缓冲操作

BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter

### 打印

PrintStream、PrintWriter

### 对象序列化反序列化

ObjectInputStream、ObjectOutputStream

### 转换

InputStreamReader、OutputStreamWriter



## 参考文章

- [Java 全栈知识体系](https://www.pdai.tech/md/java/io/java-io-basic-category.html)
- [吃透Java IO：字节流、字符流、缓冲流—最详细总结！](https://zhuanlan.zhihu.com/p/339834622)