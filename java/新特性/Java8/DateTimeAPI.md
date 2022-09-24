# 🍷 Java8 新特性 - LocalDate/LocalDateTime

---

## 背景

> [!NOTE]
>
> Tiago Fernandez做过一次投票，选举最烂的JAVA API，排第一的EJB2.X，第二的就是日期API。

下面我们来看下，Java8以前的Date API有哪些主要槽点：

### 槽点一

!> 最开始的时候，Date既要承载日期信息，又要做日期之间的转换，还要做不同日期格式的显示，职责较繁杂

后来从JDK 1.1 开始，这三项职责分开了:

- 使用Calendar类实现日期和时间字段之间转换；
- 使用DateFormat类来格式化和分析日期字符串；
- Date只用来承载日期和时间信息。 

原有Date中的相应方法已废弃。不过，无论是Date，还是Calendar，都用着太不方便了，这是API没有设计好的地方。

### 槽点二

!> 坑爹的year和month

```java
Date date = new Date(2012,1,1);
System.out.println(date);
输出Thu Feb 01 00:00:00 CST 3912
```

观察输出结果，year是2012+1900，而month，月份参数我不是给了1吗? 怎么输出二月(Feb)了?

应该曾有人告诉你，如果你要设置日期，应该使用 java.util.Calendar，像这样…

```java
Calendar calendar = Calendar.getInstance();
calendar.set(2013, 8, 2);
```

这样写又不对了，calendar的month也是从0开始的，表达8月份应该用7这个数字，要么就干脆用枚举

```java
calendar.set(2013, Calendar.AUGUST, 2);
```

注意上面的代码，Calendar年份的传值不需要减去1900(当然月份的定义和Date还是一样)，这种不一致真是让人抓狂！

有些人可能知道，Calendar相关的API是IBM捐出去的，所以才导致不一致。

### 槽点三

!> java.util.Date与java.util.Calendar中的所有属性都是可变的

下面的代码，计算两个日期之间的天数….

```java
public static void main(String[] args) {
    Calendar birth = Calendar.getInstance();
    birth.set(1975, Calendar.MAY, 26);
    Calendar now = Calendar.getInstance();
    System.out.println(daysBetween(birth, now));
    System.out.println(daysBetween(birth, now)); // 显示 0? 
 }  

public static long daysBetween(Calendar begin, Calendar end) {
    long daysBetween = 0;
    while(begin.before(end)) {
        begin.add(Calendar.DAY_OF_MONTH, 1);
        daysBetween++;
    }
    return daysBetween;
}
```

daysBetween有点问题，如果连续计算两个Date实例的话，第二次会取得0，因为Calendar状态是可变的，考虑到重复计算的场合，最好复制一个新的Calendar

```java
public static long daysBetween(Calendar begin, Calendar end) {
    Calendar calendar = (Calendar) begin.clone(); // 复制
    long daysBetween = 0;
    while(calendar.before(end)) {
        calendar.add(Calendar.DAY_OF_MONTH, 1);
        daysBetween++;
    }
    return daysBetween;
}   
```

### 槽点四

!> SimpleDateTimeFormat是非线程安全的。



## Java8的LocalDate API

> [!NOTE]
>
> 因为上面这些原因，诞生了第三方库Joda-Time，可以替代 Java 的时间管理 API 。
>
> Java 8 中新的时间和日期管理 API 深受Joda-Time影响，并吸收了很多Joda-Time的精华，新的java.time包包含了所有关于日期、时间、时区、Instant(跟日期类似但是精确到纳秒)、duration(持续时间)和时钟操作的类。
>
> 新设计的 API 认真考虑了这些类的不变性，如果某个实例需要修改，则返回一个新的对象。

### 类概览

Java 8仍然延用了ISO的日历体系，并且与它的前辈们不同，java.time包中的类是不可变且线程安全的。新的时间及日期API位于java.time包中，下面是里面的一些关键的类:

- Instant——它代表的是时间戳
- LocalDate——不包含具体时间的日期，比如2014-01-14。它可以用来存储生日，周年纪念日，入职日期等。
- LocalTime——它代表的是不含日期的时间
- LocalDateTime——它包含了日期及时间，不过还是没有偏移信息或者说时区。
- ZonedDateTime——这是一个包含时区的完整的日期时间，偏移量是以UTC/格林威治时间为基准的。

新的库还增加了ZoneOffset及Zoned，可以为时区提供更好的支持。有了新的DateTimeFormatter之后日期的解析及格式化也变得焕然一新了。

### 方法概览

该包的API提供了大量相关的方法，这些方法一般有一致的方法前缀:

- of: 静态工厂方法。
- parse: 静态工厂方法，关注于解析。
- get: 获取某些东西的值。
- is: 检查某些东西的是否是true。
- with: 不可变的setter等价物。
- plus: 加一些量到某个对象。
- minus: 从某个对象减去一些量。
- to: 转换到另一个类型。
- at: 把这个对象与另一个对象组合起来，例如:  date.atTime(time)。



### 关键类的使用

接下来看看java.time包中的关键类和各自的使用例子:

#### 1、Clock类

Clock类使用时区来返回当前的纳秒时间和日期。Clock可以替代System.currentTimeMillis()和TimeZone.getDefault()，实例如下：



```java
final Clock clock = Clock.systemUTC(); 
System.out.println( clock.instant() ); 
System.out.println( clock.millis() ); 
```

输出结果是



```java
2021-02-24T12:24:54.678Z 
1614169494678 
```

#### 2、LocalDate、LocalTime 和 LocalDateTime类

LocalDate、LocalTime 和 LocalDateTime 类，都是用于处理日期时间的 API，在处理日期时间时可以不用强制性指定时区。

#### 2.1、LocalDate

LocalDate 仅仅包含ISO-8601日历系统中的日期部分，实例如下：



```java
//获取当前日期 
final LocalDate date = LocalDate.now(); 
//获取指定时钟的日期 
final LocalDate dateFromClock = LocalDate.now( clock ); 
 
System.out.println( date ); 
System.out.println( dateFromClock ); 
```

输出结果：



```java
2021-02-24 
2021-02-24 
```

#### 2.2、LocalTime

LocalTime 仅仅包含该日历系统中的时间部分，实例如下：



```java
//获取当前时间 
final LocalTime time = LocalTime.now(); 
//获取指定时钟的时间 
final LocalTime timeFromClock = LocalTime.now( clock ); 
 
System.out.println( time ); 
System.out.println( timeFromClock ); 
```

输出结果：



```java
20:36:16.315 
20:36:16.315 
```

#### 2.3、LocalDateTime

LocalDateTime 类包含了 LocalDate 和 LocalTime 的信息，但是不包含 ISO-8601 日历系统中的时区信息，实例如下：



```java
//获取当前日期时间 
final LocalDateTime datetime = LocalDateTime.now(); 
//获取指定时钟的日期时间 
final LocalDateTime datetimeFromClock = LocalDateTime.now( clock ); 
 
System.out.println( datetime ); 
System.out.println( datetimeFromClock ); 
```

输出结果：



```java
2021-02-24T20:38:13.633 
2021-02-24T20:38:13.633 
```

#### 3、ZonedDateTime类

如果你需要特定时区的信息，则可以使用 ZoneDateTime，它保存有 ISO-8601 日期系统的日期和时间，而且有时区信息，实例如下：



```java
// 获取当前时间日期 
final ZonedDateTime zonedDatetime = ZonedDateTime.now(); 
//获取指定时钟的日期时间 
final ZonedDateTime zonedDatetimeFromClock = ZonedDateTime.now( clock ); 
//获取纽约时区的当前时间日期 
final ZonedDateTime zonedDatetimeFromZone = ZonedDateTime.now( ZoneId.of("America/New_York") ); 
 
System.out.println( zonedDatetime ); 
System.out.println( zonedDatetimeFromClock ); 
System.out.println( zonedDatetimeFromZone ); 
```

输出结果：



```java
2021-02-24T20:42:27.238+08:00[Asia/Shanghai] 
2021-02-24T12:42:27.238Z 
2021-02-24T07:42:27.241-05:00[America/New_York] 
```

#### 4、Duration类

Duration类，它持有的时间精确到秒和纳秒。利用它我们可以很容易得计算两个日期之间的不同，实例如下：



```java
final LocalDateTime from = LocalDateTime.of( 2020, Month.APRIL, 16, 0, 0, 0 ); 
final LocalDateTime to = LocalDateTime.of( 2021, Month.APRIL, 16, 23, 59, 59 ); 
//获取时间差 
final Duration duration = Duration.between( from, to ); 
System.out.println( "Duration in days: " + duration.toDays() ); 
System.out.println( "Duration in hours: " + duration.toHours() ); 
```

输出结果：



```java
Duration in days: 365 
Duration in hours: 8783 
```



## 参考资料

- [Java8 新特性全面介绍，强烈建议收藏](https://www.51cto.com/article/647804.html)
- [Java 全栈知识体系](https://www.pdai.tech/md/java/java8/java8-localdatetime.html)

