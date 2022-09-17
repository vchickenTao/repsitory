### 1.知识体系

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-09/b6fbe2df-cb03-4a87-bded-c7493193e0c2.png)
    （图片来源https://www.pdai.tech/md/resource/tools.html， 下同，感谢pdai大佬）
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-09/81e31c54-ca9d-484d-9b75-b0ad38c54886.png)

### 2.JVM-类加载
 **1.类字节码详解**
  > 类的源代码通过编译器编译为字节码，再由类加载的子系统加载到JVM中。

- **语言编译为字节码在JVM中运行**
  计算机是不能直接运行java代码的，必须要先运行java虚拟机，再由java虚拟机运行编译后的java代码。这个编译后的java代码，就是`字节码`。
  **1.为什么jvm不能直接运行java代码呢？**
    答：因为在cpu层面看来计算机中所有的操作都是一个个指令的运行汇集而成的，java是高级语言，只有人类才能理解其逻辑，计算机是无法识别的，所以java代码必须要先编译成字节码文件，jvm才能正确识别代码转换后的指令并将其运行。这也是Java是支持`跨平台`的原因所在，只要能安装JVM就可以运行Java代码,实现一次编写，到处运行的目的，同时衍生出了许多基于JVM的编程语言，如Groovy, Scala, Koltin等等。

- **Java字节码文件**
class文件本质上是一个以`8位字节`为基础单位的`二进制流`，各个数据项目严格按照顺序紧凑的排列在class文件中。jvm根据其特定的规则解析该二进制数据，从而得到相关信息。 Class文件采用一种伪结构来存储数据，它有两种类型：无符号数和表。

- **Class文件的结构属性**
  ![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-09/5d919e4f-c36a-4ef7-9a90-e1b06f9739f4.png)

**2.字节码的增强技术**
> 待补充

**3.Java 类加载机制**
 - **<u>类的生命周期</u>**
类加载的过程包括了加载，验证，准备，解析，初始化五个阶段。其中前四个发生的顺序是确定的。而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定(也称为动态绑定或晚期绑定)。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-09/39458a37-780e-4c27-bae3-45e126784e1b.png)
- **加载: 查找并加载类的二进制数据**
  `加载`是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情:
   - 通过一个类的全限定名来获取其定义的二进制字节流。 
   - 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。 
   - 在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-09/9108e69f-7e8c-419f-8040-abf95436e902.png)
相对于类加载的其他阶段而言，加载阶段(准确地说，是加载阶段获取类的二进制字节流的动作)是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。 
加载阶段完成后，虚拟机外部的 二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个java.lang.Class类的对象，这样便可以通过该对象访问方法区中的这些数据。 
类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误(LinkageError错误)如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

- **连接**
  - **验证: 确保被加载的类的正确性** 
  验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作: 
    - `文件格式验证`: 验证字节流是否符合Class文件格式的规范；例如: 是否以0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。 
    - `元数据验证`: 对字节码描述的信息进行语义分析(注意: 对比javac编译阶段的语义分析)，以保证其描述的信息符合Java语言规范的要求；例如: 这个类是否有父类，除了java.lang.Object之外。 
    - `字节码验证`: 通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。 
    - `符号引用验证`: 确保解析动作能正确执行。
  - **准备: 为类的静态变量分配内存，并将其初始化为默认值**
准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意: 
    - 这时候进行内存分配的仅包括类变量(static)，而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。 
    - 这里所设置的初始值通常情况下是数据类型默认的零值(如0、0L、null、false等)，而不是被在Java代码中被显式地赋予的值。 假设一个类变量的定义为: `public static int value = 3`；那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何Java方法，而把value赋值为3的put static指令是在程序编译后，存放于类构造器<clinit>()方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行。
    - 对基本数据类型来说，对于类变量(static)和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值，而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过。 
    - 对于同时被static和final修饰的常量，必须在声明的时候就为其显式地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。 
    - 对于引用数据类型reference来说，如数组引用、对象引用等，如果没有对其进行显式地赋值而直接使用，系统都会为其赋予默认的零值，即null。 
    - 如果在数组初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认的零值。 
    - 如果类字段的字段属性表中存在ConstantValue属性，即同时被final和static修饰，那么在准备阶段变量value就会被初始化为ConstValue属性所指定的值。假设上面的类变量value被定义为: `public static final int value = 3`；编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value赋值为3。我们可以理解为static final常量在编译期就将其结果放入了调用它的类的常量池中
  - **解析: 把类中的符号引用转换为直接引用**
解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。
符号引用就是一组符号来描述目标，可以是任何字面量。 
直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。(`待理解`)
- **初始化**
初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式:
   - 声明类变量是指定初始值
   - 使用静态代码块为类变量指定初始值

**JVM初始化步骤**
- 假如这个类还没有被`加载`和`连接`，则程序先加载并连接该类;
- 假如该类的`直接父类`还没有被初始化，则先初始化其直接父类;
- 假如类中有初始化语句，则系统依次执行这些初始化语句。

**类初始化时机：**
只有当对类的主动使用的时候才会导致类的初始化，类的主动使用包括以下六种:
- 创建类的实例，也就是new的方式；
- 访问某个类或接口的`静态变量`，或者对该静态变量赋值；
- 调用类的`静态方法`； 
- `反射`(如Class.forName("com.pdai.jvm.Test"))； 
- 初始化某个类的`子类`，则其父类也会被初始化； 
- Java虚拟机启动时被标明为`启动类`的类(Java Test)，直接使用java.exe命令来运行某个主类。

**使用**
`类`访问`方法区`内的数据结构的接口， `对象`是`Heap区`的数据。

**卸载**
Java虚拟机将结束生命周期的几种情况:
- 执行了System.exit()方法
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致Java虚拟机进程终止

**<u>类加载器， JVM类加载机制</u>**

**类加载器的层次**
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-10/901154e6-5631-4c49-8c64-f82dc93ad320.png)
> 注意: 这里父类加载器并不是通过继承关系来实现的，而是采用组合实现的。

**站在Java开发人员的角度来看，类加载器可以大致划分为以下三类:**
- `启动类加载器`: Bootstrap ClassLoader,负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库(如rt.jar，所有的java.*开头的类均被Bootstrap ClassLoader加载)。`启动类加载器`是无法被Java程序直接引用的。
- `拓展类加载器`：Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载JDK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库(如javax.*开头的类)，开发者可以直接使用扩展类加载器。
- `应用程序类加载器`：Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径(ClassPath)所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器。因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的java class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点: 
- 在执行非置信代码之前，自动验证数字签名。 
- 动态地创建符合用户特定需要的定制化构建类。 
- 从特定的场所取得java class，例如数据库中和网络中。

**<u>寻找类加载器</u>**
```java
package com.vchicken.jvm;

public class ClassLoaderTest {

    public static void main(String[] args) {
        ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
        System.out.println(contextClassLoader);
        System.out.println(contextClassLoader.getParent());
        System.out.println(contextClassLoader.getParent().getParent());
    }
}

#输出结果：
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@5caf905d
null
```
从上面的结果可以看出，并没有获取到ExtClassLoader的父Loader，原因是BootstrapLoader(启动类加载器)是用`C语言`实现的，找不到一个确定的返回父Loader的方式，于是就返回null。

<u>**类的加载**</u>
类加载有三种方式:

1、命令行启动应用时候由JVM初始化加载

2、通过Class.forName()方法动态加载

3、通过ClassLoader.loadClass()方法动态加载

```java
public class loaderTest {

    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader classLoader = HelloWorld.class.getClassLoader();
        System.out.println(classLoader);
        //ClassLoader.loadClass，不会执行初始化块
//        classLoader.loadClass("Test2");

        //使用Class.forName()来加载类，默认会执行初始化块，注意这里要使用类的全路径名，否则会报ClassNotFoundExcepion，因为通过反射实例化类时传递的类名称必须是全路径名称
//        Class.forName("com.vchicken.jvm.Test2");
        //使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块
//        Class.forName("com.vchicken.jvm.Test2", false, classLoader);
    }
}

package com.vchicken.jvm;

public class Test2{

    static {
        System.out.println("执行静态代码块");
    }
}
```

> Class.forName()和ClassLoader.loadClass()区别? 
- Class.forName(): 将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块； 
- ClassLoader.loadClass(): 只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。 
- Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 

<u>**JVM类加载机制**</u>
- `全盘负责`，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显式使用另外一个类加载器来载入 
- `父类委托`，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类 
- `缓存机制`，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效 
- `双亲委派机制`, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

**双亲委派机制过程？** 
1. 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。 
2. 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。 
3. 如果BootStrapClassLoader加载失败(例如在$JAVA_HOME/jre/lib里未查找到该class)，会使用ExtClassLoader来尝试加载； 
4. 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

**双亲委派代码实现**
```java
public Class<?> loadClass(String name)throws ClassNotFoundException {
            return loadClass(name, false);
    }
    protected synchronized Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException {
            // 首先判断该类型是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
                //如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                try {
                    if (parent != null) {
                         //如果存在父类加载器，就委派给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {
                    //如果不存在父类加载器，就检查是否是由启动类加载器加载的类，通过调用本地方法native Class findBootstrapClass(String name)
                        c = findBootstrapClass0(name);
                    }
                } catch (ClassNotFoundException e) {
                 // 如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }

```

**双亲委派优势**

- 系统类防止内存中出现多份同样的字节码
- 保证Java程序安全稳定运行

**沙箱安全机制？**
**1.概述**：
Java安全模型的核心就是Java沙箱(Sandbox)，什么是沙箱？沙箱是一个限制程序运行的环境。

沙箱机制就是将Java代码限定在虚拟机(JVM)特定的运行范围中，并且严格限制代码对本地系统资源访问。通过这样的措施来保证对代码的有限隔离，防止对本地系统造成破坏。

沙箱主要限制系统资源访问，那系统资源包括什么？CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。
`所有的Java程序运行都可以指定沙箱，可以定制安全策略。`

**2.演变过程**：
- 在Java中将执行程序分成本地代码和远程代码两种，本地代码默认视为可信任的，而远程代码则被看作是不受信的。对于授信的本地代码，可以访问一切本地资源。而对于非授信的远程代码在早期的Java实现中，安全依赖于沙箱（Sandbox）机制。如下图所示JDK1.0安全模型：
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/826cc49a-c089-49b4-915f-c1da793f663a.png)
- JDK1.0中如此严格的安全机制也给程序的功能扩展带来障碍，比如当用户希望远程代码访问本地系统的文件时候，就无法实现。
因此在后续的Java1.1版本中，针对安全机制做了改进，增加了安全策略。允许用户指定代码对本地资源的访问权限。如下图所示JDK1.1安全模型：
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/10ae4ef9-468e-497e-8871-ed5b228ce8d7.png)
- 在Java1.2版本中，再次改进了安全机制，增加了代码签名。不论本地代码或是远程代码，都会按照用户的安全策略设定，由类加载器加载到虚拟机中权限不同的运行空间，来实现差异化的代码执行权限控制。如下图所示JDK1.2安全模型： 
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/5804d61b-df90-459d-9ab3-5e2d937154e3.png)
- 当前最新的安全机制实现，则引入了域（Domain）的概念。
虚拟机会把所有代码加载到不同的系统域和应用域。系统域部分专门负责与关键资源进行交互，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问。虚拟机中不同的受保护域（Protected Domain），对应不一样的权限（Permission）。存在于不同域中的类文件就具有了当前域的全部权限，如下图所示，最新的安全模型（jdk1.6）：
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/79d31220-5236-4733-84f6-9224f7c20d74.png)

**3.组成沙箱的基本组件**：
`字节码校验器`（bytecode verifier）：确保Java类文件遵循Java语言规范。这样可以帮助Java程序实现内存保护。但并不是所有的类文件都会经过字节码校验，比如核心类。
`类装载器`（class loader）：其中类装载器在3个方面对Java沙箱起作用
  - 它防止恶意代码去干涉善意的代码；
  - 它守护了被信任的类库边界；
  - 它将代码归入保护域，确定了代码可以进行哪些操作。

    虚拟机为不同的类加载器载入的类提供不同的命名空间，命名空间由一系列唯一的名称组成，每一个被装载的类将有一个名字，这个命名空间是由Java虚拟机为每一个类装载器维护的，它们互相之间甚至不可见。

  类装载器采用的机制是双亲委派模式。

1.从最内层JVM自带类加载器开始加载，外层恶意同名类得不到加载从而无法使用；
2.由于严格通过包来区分了访问域，外层恶意的类通过内置代码也无法获得权限访问到内层类，破坏代码就自然无法生效。
- `存取控制器`（access controller）：存取控制器可以控制核心API对操作系统的存取权限，而这个控制的策略设定，可以由用户指定。
- `安全管理器`（security manager）：是核心API和操作系统之间的主要接口。实现权限控制，比存取控制器优先级高。
- `安全软件包`（security package）：java.security下的类和扩展包下的类，允许用户为自己的应用增加新的安全特性，包括：
- 安全提供者
- 消息摘要
- 数字签名
- 加密
- 鉴别

<u>**自定义类加载器**</u>
```java
public class MyClassLoader extends ClassLoader {

    private String root;

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] bytes = loadClassData(name);
        if (bytes == null){
            throw new ClassNotFoundException();
        }else{
            return defineClass(name,bytes,0,bytes.length);
        }
    }

    private byte[] loadClassData(String className) {
        String fileName = root + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }


    public static void main(String[] args)  {

        MyClassLoader classLoader = new MyClassLoader();
        classLoader.setRoot("D:\\idea\\javademo");

        Class<?> testClass = null;
        try {
            testClass = classLoader.loadClass("com.vchicken.jvm.Test2");
            Object object = testClass.newInstance();
            System.out.println(object.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}

//执行结果：
执行静态代码块
sun.misc.Launcher$AppClassLoader@18b4aac2
```
**这里有几点需要注意 :** 
1、这里传递的文件名需要是类的`全限定性名称`，即com.vchicken.jvm.Test2格式的，因为 defineClass 方法是按这种格式进行处理的。 
2、最好不要重写loadClass方法，因为这样容易破坏双亲委托模式。 
3、这类Test 类本身可以被 AppClassLoader 类加载，因此我们不能把com/vchicken/jvm/Test2.class 放在类路径下。否则，由于双亲委托机制的存在，会直接导致该类由 AppClassLoader 加载，而不会通过我们自定义类加载器来加载。

### 3.补充：native关键字
**1、native概念**
`native关键字`修饰的Java方法是一个原生态方法，方法对应的实现Java作用范围达不到，而是在用其他编程语言（如C和C++）文件中实现。Java语言本身不能直接对操作系统底层进行访问和操作，但可以通过JNI接口调用其他编程语言来实现对操作系统底层的访问。
`native` 用来修饰方法，用 native 声明的方法表示告知 JVM 调用，该方法在外部定义，我们可以用任何语言去实现它。 简单地讲，一个native Method就是一个 Java 调用非 Java 代码的接口。

**2.什么是JNI？**
JNI是Java本机接口（Java Native Interface），是一个本机编程接口，它是Java软件开发工具箱（Java Software Development Kit，SDK）的一部分。JNI允许Java代码调用以其他编程语言编写的代码和代码库。JNI中的Invocation API可以用来将Java虚拟机（JVM）嵌入到本机应用程序中，从而允许开发人员从本机代码内部调用Java代码。

**3.native 语法：**
①、修饰方法的位置必须在返回类型之前，和其余的方法控制符前后关系不受限制。
②、不能用 abstract 修饰，也没有方法体，也没有左右大括号。
③、返回值可以是任意类型
我们在日常编程中看到native修饰的方法，只需要知道这个方法的作用是什么，至于别的就不用管了，操作系统会给我们实现。

### 4.PC寄存器
`程序计数寄存器`(Program Counter Register)
JVM中的PC寄存器是对物理PC寄存器的一种抽象模拟。

**1.作用：**
PC寄存器用来存储指向下一条指令的地址，也即将要执行的指令代码。
由执行引擎读取下一条指令。
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/f99f32e0-a994-410a-afd9-49d5ec9ec6d9.png)

**2.特性**
- 它是一块很小的内存空间，几乎可以忽略不记。也是运行速度最快的存储区域
- 在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，就是一个指针，指向方法区中的方法字节码，生命周期与线程的生命周期保持一致。
- 任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。
- 程序计数器会存储当前线程正在执行的Java方法的JVM指令地址，若在执行native方法，则是未指定值(undefined)

### 5.方法区
方法区（Method Area）

**1.概念**
`方法区`，是一个被线程共享的内存区域 。 其中主要存储加载的类字节码 、class/method/field 等元数据、static final 常量、static 变量、即时编译器编译后的代码等数据。另外，方法区包含了一个特殊的域“`运行时常量池`”。

> 静态变量,变量，类信息（构造方法,接口定义),运行时的常量池存放在方法区中，但实例变量存在堆内存中,和方法区无关。

**2.栈、堆、方法区的交互关系**
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/e2a5656e-8840-4833-82ec-beeb8b66431f.png)
- Person：存放在元空间，也可以说方法区
- person：存放在Java栈的局部变量表中
- new Person()：存放在Java堆中

**3.方法区的理解**
《Java虚拟机规范》中明确说明：“尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩。”但对于HotSpotJVM而言，方法区还有一个别名叫做Non-Heap（非堆），目的就是要和堆分开。
所以，方法区看作是一块独立于Java堆的内存空间。![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/197a0757-ae59-4a10-8f18-7defaa5df561.png)
方法区主要存放的是 Class，而堆中主要存放的是 实例化的对象

- 方法区（Method Area）与Java堆一样，是各个线程共享的内存区域。
- 方法区在JVM启动的时候被创建，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的。
- 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。
- 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：java.lang.OutofMemoryError：PermGen space 或者java.lang.OutOfMemoryError:Metaspace
  - 加载大量的第三方的jar包
  - Tomcat部署的工程过多（30~50个）
  - 大量动态的生成反射类
- 关闭JVM就会释放这个区域的内存。

**4.方法区的内部结构**
![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-11/4127b974-e835-45ea-a54c-222678fb51bc.png)