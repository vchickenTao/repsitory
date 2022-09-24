# ☕ Java8 新特性 - Stream API

---

## 概念

Java 8 API添加了一个新的java.util.stream工具包，被称为`流 Stream`，可以让你以一种声明的方式处理数据，这是目前为止最大的一次对 Java 库的完善。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

Stream API 可以极大提高 Java 程序员的生产力，让程序员写出高效率、干净、简洁的代码。

**这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。**

元素流在管道中经过中间操作(intermediate operation)的处理，最后由最终操作(terminal operation)得到前面处理的结果。

```
+--------------------+       +------+   +------+   +---+   +-------+ 
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect| 
+--------------------+       +------+   +------+   +---+   +-------+ 
```

以上的流程转换为 Java 代码，实例如下：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
// 获取集合中大于2、并且经过排序、平方去重的有序集合 
List<Integer> squaresList = numbers 
        .stream() 
        .filter(x -> x > 2) 
        .sorted((x,y) -> x.compareTo(y)) 
        .map( i -> i*i).distinct().collect(Collectors.toList()); 
```



## 常见方法和使用

在 Java 8 中，集合接口有两个方法来生成流：

- `stream()`：为集合创建**串行流**
- `parallelStream()`：为集合创建**并行流**

当然，流的来源可以是`集合`，`数组`，`I/O channel`， `产生器generator `等!

### stream分类

- **顺序流stream**

```java
List <Person> people = list.getStream.collect(Collectors.toList());
```

- **并行流parallelStream**

```java
List <Person> people = list.getStream.parallel().collect(Collectors.toList());
```

?> 顾名思义，当使用顺序方式去遍历时，每个item读完后再读下一个item。而使用并行去遍历时，数组会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。

###  parallelStream原理:

```java
List originalList = someData;
split1 = originalList(0, mid);//将数据分小部分
split2 = originalList(mid,end);
new Runnable(split1.process());//小部分执行操作
new Runnable(split2.process());
List revisedList = split1 + split2;//将结果合并
```

大家对hadoop有稍微了解就知道，里面的 MapReduce 本身就是用于并行处理大数据集的软件框架，其 处理大数据的核心思想就是大而化小，分配到不同机器去运行map，最终通过reduce将所有机器的结果结合起来得到一个最终结果，与MapReduce不同，Stream则是利用多核技术可将大数据通过多核并行处理，而MapReduce则可以分布式的。

### stream与parallelStream性能测试对比

如果是多核机器，理论上并行流则会比顺序流快上一倍，下面是测试代码

```java
long t0 = System.nanoTime();

//初始化一个范围100万整数流,求能被2整除的数字，toArray()是终点方法

int a[]=IntStream.range(0, 1_000_000).filter(p -> p % 2==0).toArray();

long t1 = System.nanoTime();

//和上面功能一样，这里是用并行流来计算

int b[]=IntStream.range(0, 1_000_000).parallel().filter(p -> p % 2==0).toArray();

long t2 = System.nanoTime();

//我本机的结果是serial: 0.06s, parallel 0.02s，证明并行流确实比顺序流快

System.out.printf("serial: %.2fs, parallel %.2fs%n", (t1 - t0) * 1e-9, (t2 - t1) * 1e-9);
  
```



### 常见方法

- stream(), parallelStream() 
- filter() 
- findAny() findFirst() 
- sort 
- forEach
- map(), reduce() 
- flatMap() - 将多个Stream连接成一个Stream 
- collect(Collectors.toList()) 
- distinct, limit 
- count 
- min, max, summaryStatistics 

**所有API：**

![stream](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-24/b3fdf225-481e-44cc-81e8-9b5483df6a44_stream所有API.png)

#### 1、filter & Predicate

> filter方法用于通过设置的条件过滤出元素。以下代码片段使用filter方法过滤出空字符串。

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList()); 
```

多个Predicate组合filter

```java
// 可以用and()、or()和xor()逻辑函数来合并Predicate，
// 例如要找到所有以J开始，长度为四个字母的名字，你可以合并两个Predicate并传入
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
Predicate<String> fourLetterLong = (n) -> n.length() == 4;
names.stream()
    .filter(startsWithJ.and(fourLetterLong))
    .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n));

```

#### 2、limit

> limit方法用于获取指定数量的流。以下代码片段使用limit方法打印出 10 条数据：

```java
Random random = new Random(); 
random.ints().limit(10).forEach(System.out::println); 
```

#### 3、sorted

> sorted方法用于对流进行排序。以下代码片段使用sorted方法对集合中的数字进行排序：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
numbers.stream().sorted().forEach(System.out::println); 
```

#### 4、map&reduce

> map方法用于映射每个元素到对应的结果，以下代码片段使用map输出了元素对应的平方数：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
// 获取对应的平方数 
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList()); 
```

> reduce() 函数可以将所有值合并成一个

```java
int total = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9).reduce((sum, n) -> sum + n).get();
System.out.println(total); // 45
```

为了方便理解，上面式子等价于

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9);
int total = 0;
for (n : stream) {
   total = (total, n) -> total + n;
}
System.out.println(total);
```

#### 5、forEach

> forEach方法用于迭代流中的每个数据。以下代码片段使用forEach输出集合中的数字：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
numbers.stream().forEach(System.out::println); 
```

#### 6、Collectors

> Collectors类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors可用于返回列表或字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList()); 
 
System.out.println("筛选列表: " + filtered); 
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", ")); 
System.out.println("合并字符串: " + mergedString); 
```

#### 7、统计

> 一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
  
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics(); 
  
System.out.println("列表中最大的数 : " + stats.getMax()); 
System.out.println("列表中最小的数 : " + stats.getMin()); 
System.out.println("所有数之和 : " + stats.getSum()); 
System.out.println("平均数 : " + stats.getAverage()); 
```

#### 8、flatMap

> 将多个Stream连接成一个Stream

```java
List<Integer> result= Stream.of(Arrays.asList(1,3),Arrays.asList(5,6)).flatMap(a->a.stream()).collect(Collectors.toList());
// 结果
[1, 3, 5, 6]
```

#### 9、distinct

> 去重

```java
List<LikeDO> likeDOs=new ArrayList<LikeDO>();
List<Long> likeTidList = likeDOs.stream().map(LikeDO::getTid)
                .distinct().collect(Collectors.toList());
```

#### 10、count

> 计总数

```java
int countOfAdult=persons.stream()
                       .filter(p -> p.getAge() > 18)
                       .map(person -> new Adult(person))
                       .count();
```

#### 11、match

> 匹配

```java
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true

boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false

boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true

```

#### 13、peek

> 可以使用peek方法，peek方法可只包含一个空的方法体，只要能设置断点即可，但有些IDE不允许空，可以如下文示例，简单写一个打印逻辑。

```java
List<Person> lists = new ArrayList<Person>();
lists.add(new Person(1L, "p1"));
lists.add(new Person(2L, "p2"));
lists.add(new Person(3L, "p3"));
lists.add(new Person(4L, "p4"));
System.out.println(lists);

List<Person> list2 = lists.stream()
				 .filter(f -> f.getName().startsWith("p"))
                .peek(t -> {
                    System.out.println(t.getName());
                })
                .collect(Collectors.toList());
System.out.println(list2);
  
```

#### 14、并行(parallel)程序

> parallelStream是流并行处理程序的代替方法。以下实例我们使用 parallelStream来输出空字符串的数量：

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
// 获取空字符串的数量 
long count = strings.parallelStream().filter(string -> string.isEmpty()).count(); 
```

 

## 参考资料

- [Java8 新特性全面介绍，强烈建议收藏](https://www.51cto.com/article/647804.html)
- [Java 全栈知识体系](https://www.pdai.tech/md/java/java8/java8-stream.html)