**<span style="font-size: 35px;">🍯 Java专项练习</span>**

---

<!-- tabs:start -->

### **面向对象和基础**

#### Java的8种基本数据类型

基本数据类型在**栈**中直接分配内存
字节长度：
boolean布尔型 1或4（单独使用时是占4个字节（boolean被JVM编译成int），组成数组时每个布尔型数据占1个字节（boolean数组被JVM编译成byte数组））
byte字节类型 1
char字符型 2
short短整型 2
int整数类型 4
float单精度浮点型 4
long长整型 8
double双精度浮点型 8

**如下代码，执行test()函数后，屏幕打印结果为**

```java
public class Test2
{
    public void add(Byte b)
    {
        b = b++;
    }
    public void test()
    {
        Byte a = 127;
        Byte b = 127;
        add(++a);
        System.out.print(a + " ");
        add(b);
        System.out.print(b + "");
    }
}
```

public void add(Byte b){ b=b++; } 这里涉及java的**自动装包/自动拆包**(AutoBoxing/UnBoxing) Byte的首字母为大写，是类，看似是引用传递，但是在add函数内实现++操作，会自动拆包成byte值传递类型，所以add函数还是不能实现自增功能。也就是说add函数只是个摆设，没有任何作用。 Byte类型值大小为-128~127之间。 add(++a);这里++a会越界，a的值变为-128 add(b); 前面说了，add不起任何作用，b还是127

#### java中将ISO8859-1字符串转成GB2312编码，语句为？

```java
注意这里"ISO8859-1"是一个普通字符串，不要被迷惑了
String.getBytes("ISO8859-1"）表示获取这个字符串的byte数组，
然后new String(String.getBytes("ISO8859-1"），GB2312)是上面的字符数组按照GB2312编码成新的字符串
```

#### 要导入java/awt/event下面的所有类，叙述正确的是？

假如,你要导入一个类 A;   java.awt 和 java.awt.event 都有类A, 你到底该引用哪一个;
所有只能导入当前层; 而且不能导入子包下的类, 避免同名类无法识别;因此，答案:只能是import java.awt.event.*

#### 以下哪些方法是Object类中的方法

```java
registerNatives()   //私有方法
getClass()    //返回此 Object 的运行类。
hashCode()    //用于获取对象的哈希值。
equals(Object obj)     //用于确认两个对象是否“相同”。
clone()    //创建并返回此对象的一个副本。
toString()   //返回该对象的字符串表示。   
notify()    //唤醒在此对象监视器上等待的单个线程。   
notifyAll()     //唤醒在此对象监视器上等待的所有线程。   
wait(long timeout)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或        者超过指定的时间量前，导致当前线程等待。   
wait(long timeout, int nanos)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。
wait()    //用于让当前线程失去操作权限，当前线程进入等待序列
finalize()    //当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。
```

#### 以下关于Object类的说法正确的是

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-27/8224d269-c5b4-453b-84d0-0ec5c4ed88d8.png)
A. Java中的所有类都直接或间接继承自Object，无论是否明确的指明，也无论其是否是抽象类。
B. Java中的接口（interface）并没有继承自Object
      一个类的子类必然是另一个类，如果interface继承自Object，那么interface也必然是一个类，但事实并非如此
C. 这是两个不同的方法，比较的规则也不一样
D. toString方法是Object中定义的方法，不需要重写也可以使用

#### 关于String值传递的理解

```java
public class Example{
    String str=new String("hello");
    char[]ch={'a','b'};
    public static void main(String args[]){
        Example ex=new Example();
        ex.change(ex.str,ex.ch);
        System.out.print(ex.str+" and ");
        System.out.print(ex.ch);
    }
    public void change(String str,char ch[]){
        str="test ok";
        ch[0]='c';
    }
}
```

![String值传递](https://uploadfiles.nowcoder.com/images/20160907/6316247_1473261594090_EF10301D326448E047AAD72D03582AB9)
String很奇特，虽然是引用数据类型，但是采用的却是值传递！基本数据类型采用的都是值传递,数组和对象都是引用传递(数组可以按照对象来算),值传递不会改变本身,只是传递拷贝,而引用传递却会改变本身。str属于值传递,不会改变。char[] 属于引用传递，所以改变本身值。

#### Java标识符命名规则

1. 由26个大小写的英文字母"A-Z","a-z";数字"0-9"，下划线"_" 和 
   美元符号"$"四部分组成；
2. 标识符以字母或下划线"_"或"$"开头;
   注：尽管"$"是一个合法的Java字符，但尽量不要在代码中使用这个字符，
    它一般用在Java编译器或其他工具生成的名字中；
3. 标识符不能是关键字，如：public，protected,....
   以及两个保留的 关键字const和goto；
4. 标识符区分大小写。

#### 成员和内部类的访问权限

内部类理解成类的成员，成员有4种访问权限吧，内部类也是！分别为private、protected、public以及默认的访问权限。

#### 对象的hashcode

- equals()相等的两个对象hashCode()一定相等。
- hashCode()相等的两个对象equal()不一定相等。

#### 关于abstract的理解

![关于abstract的理解](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-27/7d64d65a-a818-42d7-8694-d50dfae92de7.png)
选 ACD
A，抽象类和接口都不可以实例化。
B，final类不能被继承。
C，abstract不能和final共用修饰类。
D，抽象类中可以没有抽象方法，有抽象方法的类一定是抽象类。
注意：abstract是用来修饰类和方法的：
       1. 修饰方法：abstract不能和private、final、static共用。
       2. 修饰外部类：abstract不能和final、static共用。（外部类的访问修饰符只能是默认和public）
       3. 修饰内部类：abstract不能和final共用。（内部类四种访问修饰符都可以修饰）

#### 局部变量能否和成员变量重名？

局部变量可以和成员变量重名，不加“this”修饰时，优先使用最近的变量。

#### 关于静态成员变量的引用，下列代码执行准确的是？

![静态成员变量的引用](https://uploadfiles.nowcoder.com/images/20191125/514787832_1574658927046_431F17616A9EB69355860768C75387CB)

#### 创建子类的对象时，会先执行父类的构造函数

```java
class Base {
    Base() {
    System.out.print("Base"); 
    }
}
public class Alpha extends Base {
    public static void main( String[] args ) {
        new Alpha();
        //调用父类无参的构造方法
        new Base();
    } 
}
//运行结果：BaseBase
```

子类构造器的默认第一行就是super()，默认调用直接父类的无参构造。这也就是一旦一个子类的直接父类没有无参的构造的情况下，必须在自己构造器的第一行显式的指明调用父类或者自己的哪一个构造器。

#### 关于序列化，有以下一个对象：

```java
public class DataObject implements Serializable{
    private static int i=0;
    private String word=" ";
    public void setWord(String word){
        this.word=word;
    }
    public void setI(int i){
        Data0bject.i=i;
     }
}

创建一个如下方式的DataObject:
DataObject object=new Data0bject ( );
object.setWord("123");
object.setI(2); 
```

将此对象序列化为文件，并在另外一个JVM中读取文件，进行反序列化，请问此时读出的Data0bject对象中的word和i的值分别为：

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/0086117c-6645-4ad9-8228-f48f983631f8_对象序列化1.png)

**解答：**序列化保存的是对象的状态，静态变量属于类的状态，因此，序列化并不保存静态变量。所以i是没有改变的,所以选D

#### 根据下面这个程序的内容，判断哪些描述是正确的

```java
public class Test {
    public static void main(String args[]) {
        String s = "tommy";
        Object o = s;
        sayHello(o); //语句1
        sayHello(s); //语句2
    }
    public static void sayHello(String to) {
        System.out.println(String.format("Hello, %s", to));
    }
    public static void sayHello(Object to) {
        System.out.println(String.format("Welcome, %s", to));
    }
}


A这段程序有编译错误
B语句1输出为:Hello, tommy
C语句2输出为:Hello, tommy
D语句1输出为:Welcome, tommy
E语句2输出为:Welcome, tommy
F根据选用的Java编译器不同，这段程序的输出可能不同
```

**解答：**

```html
重载：静态分派，按照静态类型(外观类型)查找。 重写，动态分派，按照实际类型(运行时类型)查找方法。
方法调用的静态分派，静态分派依据参数声明类型，在编译期就已经确定调用的方法,选参数类型最符合作为原则，故选CD
```

#### Java逻辑运算（|，||，&，&&）

```html
一、&与&&的异同点。

相同点：二者都表示与操作，当且仅当运算符两边的操作数都为true时，其结果才为true，否则为false。

不同点：在使用&进行运算时，不论左边为true或者false，右边的表达式都会进行运算。如果使用&&进行运算时，当左边为false时，右边的表达式不会进行运算，因此&&被称作短路与。

二、|与||的异同点。

相同点：二者都表示或操作，当运算符两边的操作数任何一边的值为true时，其结果为true，当两边的值都为false时，其结果才为false。

不同点：同与操作类似，||表示短路或，当运算符左边的值为true时，右边的表达式不会进行运算。
```

#### 在Java语言中，下列关于字符集编码（Character set encoding）和国际化（i18n）的问题，哪些是正确的?

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/28294461-113a-4e7a-86fe-1335cbce3936_字符集编码（Character set encoding）和国际化（i18n）的问题.png)

**解答：**

```html
A 显然是错误的，Java一律采用Unicode编码方式，每个字符无论中文还是英文字符都占用2个字节。
B 也是不正确的，不同的编码之间是可以转换的，通常流程如下：
将字符串S以其自身编码方式分解为字节数组，再将字节数组以你想要输出的编码方式重新编码为字符串。
例：String newUTF8Str = new String(oldGBKStr.getBytes("GBK"), "UTF8");
C 是正确的。Java虚拟机中通常使用UTF-16的方式保存一个字符
D 也是正确的。ResourceBundle能够依据Local的不同，选择性的读取与Local对应后缀的properties文件，以达到国际化的目的。
```





### **IO框架**

#### IO流类的层级关系

![IO类的层级关系](http://uploadfiles.nowcoder.com/images/20150328/138512_1427527478646_1.png)



### **并发框架**

#### 创建线程对象三种方式：

1. 继承Thread类，重载run方法；
2. 实现Runnable接口，实现run方法；
3. 实现Callable接口



#### 关于ThreadLocal类

![ThreadLocal](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-08-27/34f637ec-8e23-41a2-8793-4cdd0221526b.png)
1、ThreadLocal的类声明：public class ThreadLocal<T>
可以看出ThreadLocal并没有继承自Thread，也没有实现Runnable接口。所以AB都不对。
2、ThreadLocal类为每一个线程都维护了自己独有的**变量拷贝**。每个线程都拥有了自己独立的一个变量。所以ThreadLocal重要**作用并不在于多线程间的数据共享，而是数据的独立**，C选项错。
由于每个线程在访问该变量时，**读取和修改的，都是自己独有的那一份变量拷贝**，不会被其他线程访问，变量被彻底封闭在每个访问的线程中。所以E对。
3、ThreadLocal中定义了一个哈希表用于为每个线程都提供一个变量的副本：

```java
 static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal k, Object v) {
                super(k);
                value = v;
            }
        }

        /**
         * The table, resized as necessary.
         * table.length MUST always be a power of two.
         */
        private Entry[] table;
}
```

所以D对。

#### JDK提供的用于并发编程的同步器有哪些？

- Java 并发库的`Semaphore`可以很轻松完成信号量控制，Semaphore可以控制某个资源可被同时访问的个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。
- `CyclicBarrier`主要的方法就是一个：await()。await() 方法没被调用一次，计数便会减少1，并阻塞住当前线程。当计数减至0时，阻塞解除，所有在此 CyclicBarrier 上面阻塞的线程开始运行。
- `(CountDown)门闩(Latch)`。倒计数不用说，门闩的意思顾名思义就是阻止前进。在这里就是指 CountDownLatch.await() 方法在倒计数为0之前会阻塞当前线程。

> https://blog.csdn.net/qq_18975791/article/details/80728045

#### volatile和synchronized的区别

- volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
- volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、代码块和类级别的。
- volatile仅能实现变量的修改**可见性**，**不能保证原子性**；而synchronized则可以**保证变量的修改可见性和原子性**。
- volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
- volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化。
- volatile能保证数据的可见性，但不能完全保证数据的原子性（不能保证复合操作的原子性），synchronized即保证了数据的可见性，也保证了原子性。



### **集合框架**



#### 在Java中，HashMap中是用哪些方法来解决哈希冲突的？

解决哈希冲突的方法一般有：`开放定址法`、`链地址法(拉链法)`、`再哈希法`、`建立公共溢出区`等方法。

**2.1 开放定址法**
从发生冲突的那个单元起，按照一定的次序，从哈希表中找到一个空闲的单元。然后把发生冲突的元素存入到该单元的一种方法。开放定址法需要的表长度要大于等于所需要存放的元素。
在开放定址法中解决冲突的方法有：线行探查法、平方探查法、双散列函数探查法。
开放定址法的缺点在于删除元素的时候不能真的删除，否则会引起查找错误，只能做一个特殊标记。只到有下个元素插入才能真正删除该元素。

- **2.1.1 线行探查法**
  线行探查法是开放定址法中最简单的冲突处理方法，它从发生冲突的单元起，依次判断下一个单元是否为空，当达到最后一个单元时，再从表首依次判断。直到碰到空闲的单元或者探查完全部单元为止。
  可以参考csdn上flash对该方法的演示：
  http://student.zjzk.cn/course_ware/data_structure/web/flash/cz/kfdzh.swf
- **2.1.2 平方探查法**
  平方探查法即是发生冲突时，用发生冲突的单元d[i], 加上 1²、 2²等。即d[i] + 1²，d[i] + 2², d[i] + 3²...直到找到空闲单元。
  在实际操作中，平方探查法不能探查到全部剩余的单元。不过在实际应用中，能探查到一半单元也就可以了。若探查到一半单元仍找不到一个空闲单元，表明此散列表太满，应该重新建立。
- **2.1.3 双散列函数探查法**
  这种方法使用两个散列函数hl和h2。其中hl和前面的h一样，以关键字为自变量，产生一个0至m—l之间的数作为散列地址；h2也以关键字为自变量，产生一个l至m—1之间的、并和m互素的数(即m不能被该数整除)作为探查序列的地址增量(即步长)，探查序列的步长值是固定值l；对于平方探查法，探查序列的步长值是探查次数i的两倍减l；对于双散列函数探查法，其探查序列的步长值是同一关键字的另一散列函数的值。

**2.2 链地址法(拉链法)**

链接地址法的思路是将哈希值相同的元素构成一个同义词的单链表，并将单链表的头指针存放在哈希表的第i个单元中，查找、插入和删除主要在同义词链表中进行。链表法适用于经常进行插入和删除的情况。
如下一组数字,(32、40、36、53、16、46、71、27、42、24、49、64)哈希表长度为13，哈希函数为H(key)=key%13,则链表法结果如下：
注：在java中，链接地址法也是HashMap解决哈希冲突的方法之一，jdk1.7完全采用单链表来存储同义词，jdk1.8则采用了一种混合模式，对于链表长度大于8的，会转换为红黑树存储。

**2.3 再哈希法**

就是同时构造多个不同的哈希函数：
Hi = RHi(key) i= 1,2,3 ... k;
当H1 = RH1(key) 发生冲突时，再用H2 = RH2(key) 进行计算，直到冲突不再产生，这种方法不易产生聚集，但是增加了计算时间。

**2.4 建立公共溢出区**

将哈希表分为公共表和溢出表，当溢出发生时，将所有溢出数据统一放到溢出区。

#### 关于HashMap和Hashtable正确的说法有：

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/d5841d0f-dcfb-4b59-8ed9-66798a043cca_hashtable和hashmap的区别.png)

- HashMap和Hashtable都是典型的Map实现，选项A正确。
- Hashtable在实现Map接口时保证了线程安全性，而HashMap则是非线程安全的。所以，Hashtable的性能不如HashMap，因为为了保证线程安全它牺牲了一些性能。因此选项B错误
- Hashtable不允许存入null，无论是以null作为key或value，都会引发异常。而HashMap是允许存入null的，无论是以null作为key或value，都是可以的。选项C正确，D错误。

#### 下面的对象创建方法中哪些会调用构造方法

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-02/9f0c7b2d-0be5-4ed4-8a32-8885c9e872f8_调用构造方法.png)

**解答：**这道题的考察其实很简单，就是在考察类加载时的初始化过程发生在什么情况下。一般而言，初始化为类变量赋予正确的初始值。初始化会在某个类被首次主动使用的时候触发，主动使用分为6类： 

```html
  1.new 一个类的对象； 
  2.访问类或者接口的静态变量，或者进行赋值； 
  3.调用类的静态方法； 
  4.反射； 
  5.初始化某个子类，其父类也会初始化； 
  6.在jvm启动时被标明为启动类的类。
```

选项B：“序列化”是一种把对象的状态转化成字节流的机制，“反序列”是其相反的过程，把序列化成的字节流用来在内存中**重新创建一个实际的Java对象**。这个机制被用来“持久化”对象。通过对象序列化，可以方便的实现对象的持久化储存以及在网络上的传输。

选项D：为什么需要克隆对象？直接new一个对象不行吗？**因为克隆的对象可能包含一些已经修改过的属性，而new出来的对象的属性都还是初始化时候的值，所以当需要一个新的对象来保存当前对象的“状态”就靠clone方法了**。那么我把这个对象的临时属性一个一个的赋值给我新new的对象不也行嘛？可以是可以，但是一来麻烦不说，二来，大家通过上面的源码都发现了clone是一个native方法，是在底层实现的，运行效率特别高特别快。常见的Object a=new Object();Object b;b=a;这种形式的代码复制的是**引用**，即对象在内存中的地址，a和b对象仍然指向了同一个对象。而**通过clone方法赋值的对象跟原来的对象时同时独立存在的**。



### **Java虚拟机（JVM）**

#### 运用下列哪个命令能够获取JVM的内存映像

```html
1、jps：查看本机java进程信息。
2、jstack：打印线程的栈信息，制作线程dump文件。
3、jmap：打印内存映射，制作堆dump文件
4、jstat：性能监控工具
5、jhat：内存分析工具
6、jconsole：简易的可视化控制台
7、jvisualvm：功能强大的控制台
```

<!-- tabs:end -->