# <u>集合框架体系</u>

<p class="note note-primary">重要</p>

集合主要分为两大类：

- <img src="/Users/nanase/Library/Application Support/typora-user-images/截屏2021-09-10 10.00.25.png" alt="截屏2021-09-10 10.00.25" style="zoom:35%;" />

- <img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2010.02.13.png" alt="截屏2021-09-10 10.02.13" style="zoom:50%;" />



# Collection

**Collection的方法**

> remove：删除指定元素
>
> contains：查找元素是否存在
>
> size：获取元素个数
>
> isEmpty：判断是否为空
>
> clear：清空
>
> addAll：添加多个元素
>
> contiansAll：查找多个元素是否都存在
>
> removeAll：删除多个元素



**Collection的遍历**

- **Iterator**

> hasNext：是否还有下一条元素
>
> next：移动至下一个元素
>
> remove：删除当前元素



提示：在调用 `iterator.next()` 方法之前必须先调用 `iterator.hasNext()` ，若不调用，且下一条记录无效，`iterator.next()` 会抛出 `NoSuchElementException` 异常。



> 在调用 Iterator 的 `next` 方法之前，迭代器的索引位于第一个元素之前，不指向任何元素，当第一次调用迭代器的 `next` 方法后，迭代器的索引会向后移动一位，指向第一个元素并将该元素返回，当再次调用next方法时，迭代器的索引会指向第二个元素并将该元素返回，依此类推，直到 `hasNext` 方法返回 `false`，表示到达了集合的末尾，终止对元素的遍历。
>

```java
Itertor iterator = col.iterator();
while(iterator.hashNext()) {
  Object obj = iterator.next();
  System.out.println(obj);
}
```



- **增强for循环**

> for(元素类型 元素名  ：集合名或数组名) {
>
> ​		访问元素
>
> }



提示：底层依然是调用 `iterator` 。



```java
for(Object obj : col) {
  System.out.println(obj);
}
```



## List

> 集合中元素有序（即取出顺序和添加顺序一致）
>
> 集合中每个元素都有其对应的集合索引，即支持索引



**List的方法**

> add：添加元素
>
> addAll：添加多个元素
>
> get：指定 index 位置的元素
>
> indexOf：返回 obj 在集合中**首次**出现的位置
>
> lastIndexOf：返回 obj 在集合中**末次**出现的位置
>
> remove：删除 index 位置的元素，并返回该元素
>
> set：设置 index 位置的元素的值为 ele
>
> subList：返回从 fromIndex 至 toIndex 位置的子集合



**List的遍历**

- **Iterator**

```java
Itertor iterator = list.iterator();
while(iterator.hashNext()) {
  Object obj = iterator.next();
  System.out.println(obj);
}
```



- **增强for循环**

```java
for(Object obj : list) {
  System.out.println(obj);
}
```



- **普通for循环**

```java
for(int i = 0; i < list.size(); i++) {
  obj = list.get(i);
  System.out.println(obj);
}
```



### <u>ArrayList</u>

<p class="note note-primary">重要</p>

![截屏2021-09-10 16.34.43](https://gitee.com/tsuiraku/typora/raw/master/img/截屏2021-09-10 16.34.43.png)

> - ArrayList 中维护了一个 Object 类型的数组： ` transient Obejct[] elementData`；
>   - transient：表示该属性不会被序列化。
> - 当创建 ArrayList 对象时，如果使用的是无参数构造器，则初始 elementData 容量为0，第1次添加，则扩容为10，如需要再次扩容，则扩容为 elementData 当前容量的1.5倍；
> - 如果使用的是指定大小的构造器，则初始化 elementData 容量为指定大小，如果需要再次扩容，则直接扩容为 elementData 当前容量1.5倍。

<u>**源码分析**</u>

**无参数构造器**

1. 创建一个空的 `elementData` 数组

![截屏2021-09-10 16.11.19](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.11.19.png)

2. 执行 `list.add`
   1. 先确定是否需要进行扩容 `ensureCapacityInternal()`
   2. 然后再执行赋值

![截屏2021-09-10 16.12.19](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.12.19.png)

3.  `ensureCapacityInternal()` ：确定 `minCapacity` 的值，第一次扩容默认为10

![](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.13.47.png)

4. `ensureExplicitCapacity()`
   1. `modCount++`：记录集合修改的次数
   2. 如果 `elementData` 容量不够，执行 `grow()`

![截屏2021-09-10 16.17.18](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.17.18.png)

5. `grow`：真正执行扩容的方法
   1. 第一次 `newCapacity = 10`
   2. 第二次以及以后，按照1.5倍进行扩容
   3. 扩容使用的是 `Arrays.copyOf()`

![截屏2021-09-10 16.18.16](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.18.16.png)

**有参数构造器**

1. 区别在于创建 `elementData` 数组的不同

![截屏2021-09-10 16.23.33](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.23.33.png)

2. 剩下执行流程与无参数构造器一致，但扩容机制直接按照1.5倍进行扩容



### <u>Vector</u>

<p class="note note-primary">重要</p>

![截屏2021-09-10 16.35.38](/Users/nanase/Library/Application Support/typora-user-images/截屏2021-09-10 16.35.38.png)

> - 底层是对象数组：`protected Object[] elementData` ；
> - 线程同步，即线程安全的，Vector 类带有操作方法 **<u>synchronized</u>**；
> - 开发中，需要线程同步安全时，考虑使用 Vector。



**Vector与ArrayList区别**

|           | 底层结构 | 版本   | 线程同步（安全）效率 | 扩容倍数                                                |
| --------- | -------- | ------ | -------------------- | ------------------------------------------------------- |
| ArrayList | 可变数组 | jdk1.2 | 不安全，效率高       | 有参：1.5倍 无参：第一次默认10，第二次以及以后按照1.5倍 |
| Vector    | 可变数组 | jdk1.0 | 安全，效率低         | 有参：2倍 无参：第一次默认10，第二次以及以后按照2倍     |



**<u>源码分析</u>**

**无参数构造**

1. 无参数构造，初始化大小默认10

![截屏2021-09-10 16.53.15](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.53.15.png)

<u>PS</u>：有参构造会直接走这一步

![截屏2021-09-10 18.48.51](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2018.48.51.png)

2. 执行 `add` 方法

![截屏2021-09-10 16.54.58](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.54.58.png)

3. 是否需要扩容

![截屏2021-09-10 16.56.18](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.56.18.png)

4. 扩容方法 `grow`

![截屏2021-09-10 16.57.38](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-10%2016.57.38.png)



### LinkedList

> LinkedList 底层实现了双向链表和双端队列特点；
>
> 可以添加任何元素，元素可以重复，包括 null；
>
> 线程不安全，没有实现同步。



**ArrayList和LinkedList区别**

|            | 底层结构 | 增删效率 | 改查效率 |
| ---------- | -------- | -------- | -------- |
| ArrayList  | 可变数组 | 效率低   | 效率高   |
| LinkedList | 双向     | 效率高   | 效率低   |



**<u>源码分析</u>**

**无参构造器**

1. 执行无参构造器，此时`first = null && last = null`

![截屏2021-09-11 08.59.00](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2008.59.00.png)

2. 执行 `add` 方法

![img](data:image/png;base64,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%0A77ewtUiyKLbYZ3HfkmLpY5lpucayxbLHysxqnNVsqxqrO9Zkax9rkfU66zPW%0A72xsbVJsvrept3lma2DLtS2yrbG9Z0ezC7Kbbldld82eaO9jn2O/yf6yA+rg%0A6SByqHS45Ig6ejmKHTc5to0ijPIdJRlVNeqmE9WJ7VTgVOP00JnpHO1c7Fzv%0A/HK01ei00StHnxn92cXTJddlh8tdV13Xsa7Fro2ur90c3PhulW7X3Gnu4e7z%0A3BvcX3k4egg9Nnvc8mR4jvP83rPF85OXt5fMq9ary9vKO917o/dNHz2fOJ+l%0APmd9Cb4hvvN8m3zf+3n55fsd8PvT38k/x3+P/7MxtmOEY3aM6QiwCOAFbAto%0AD2QFpgduDWwPMg/iBVUFPQq2DBYE7wx+yrZnZ7P3sl+GuITIQg6FvOP4ceZw%0AmkOx0IjQ0tDWMN2wpLANYQ/CLcKzwmvCeyI8I2ZFNEcSIqMiV0be5Jpw+dxq%0Abs9Y77Fzxp6MokYlRG2IehTtEC2LbhyHjhs7bvW4ezHWMZKY+lgQy41dHXs/%0AzjZuetyv44nj48ZXjn8S7xo/O/5MAiNhasKehLeJIYnLE+8m2SUpklqS6cmT%0AkquT36WEpqxKaZ8wesKcCRdSjVLFqQ1ppLTktJ1pvRPDJq6d2DnJc1LJpBuT%0AbScXTj43xWhK7pQjU+lTeVMPphPSU9L3pH/kxfKqeL0Z3IyNGT18Dn8d/4Ug%0AWLBG0CUMEK4SPs0MyFyV+SwrIGt1VpcoSFQu6hZzxBvEr7Ijs7dkv8uJzdmV%0AM5CbkrsvTyMvPe+wRFeSIzk5zXRa4bQ2qaO0RNo+3W/62uk9sijZTjkinyxv%0AyNeDH/UXFXaK7xQPCwILKgv6ZiTPOFioUygpvDjTYeaSmU+Lwot+nIXP4s9q%0AmW0+e8Hsh3PYc7bNReZmzG2ZZzlv0bzO+RHzdy+gLMhZ8FuxS/Gq4r8Wpixs%0AXGSyaP6iju8ivqsp0SqRldz83v/7LYvxxeLFrUvcl6xf8rlUUHq+zKWsvOzj%0AUv7S8z+4/lDxw8CyzGWty72Wb15BXCFZcWNl0Mrdq3RWFa3qWD1udd0a1prS%0ANX+tnbr2XLlH+ZZ1lHWKde0V0RUN663Wr1j/cYNow/XKkMp9G403Ltn4bpNg%0A05XNwZtrt5hsKdvyYat4661tEdvqqmyqyrcTtxdsf7IjeceZH31+rN5ptLNs%0A56ddkl3tu+N3n6z2rq7eY7xneQ1ao6jp2jtp7+WfQn9qqHWq3baPua9sP9iv%0A2P/85/SfbxyIOtBy0Odg7S/Wv2w8xDhUWofUzazrqRfVtzekNrQdHnu4pdG/%0A8dCvzr/uajJvqjyif2T5UcrRRUcHjhUd622WNncfzzre0TK15e6JCSeunRx/%0AsvVU1Kmzp8NPnzjDPnPsbMDZpnN+5w6f9zlff8HrQt1Fz4uHfvP87VCrV2vd%0AJe9LDZd9Lze2jWk7eiXoyvGroVdPX+Neu3A95nrbjaQbt25Outl+S3Dr2e3c%0A26/uFNzpvzv/HuFe6X3t++UPjB9U/W7/+752r/YjD0MfXnyU8OhuB7/jxWP5%0A44+di57QnpQ/NXta/cztWVNXeNfl5xOfd76QvujvLvlD54+NL+1e/vJn8J8X%0Aeyb0dL6SvRp4vfSN4Ztdf3n81dIb1/vgbd7b/nelfYZ9u9/7vD/zIeXD0/4Z%0AH0kfKz7Zf2r8HPX53kDewICUJ+MNfgpgsKGZmQC83gUALRUABjy3USaqzoKD%0AgqjOr4ME/hOrzouD4gVALeyUn/GcZgD2w2YDG20+AMpP+MRggLq7Dze1yDPd%0A3VS+qPAkROgbGHhjAgCpEYBPsoGB/k0DA592wGRvA9A8XXUGVQoRnhm2Bijp%0AuoFgPvhGVOfTr9b4bQ+UGXiAb/t/AWWjju9+tKXDAAAAimVYSWZNTQAqAAAA%0ACAAEARoABQAAAAEAAAA+ARsABQAAAAEAAABGASgAAwAAAAEAAgAAh2kABAAA%0AAAEAAABOAAAAAAAAAJAAAAABAAAAkAAAAAEAA5KGAAcAAAASAAAAeKACAAQA%0AAAABAAACOqADAAQAAAABAAAAigAAAABBU0NJSQAAAFNjcmVlbnNob3SUQ5F/%0AAAAACXBIWXMAABYlAAAWJQFJUiTwAAAB1mlUWHRYTUw6Y29tLmFkb2JlLnht%0AcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4%0AbXB0az0iWE1QIENvcmUgNi4wLjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9%0AImh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMi%0APgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAg%0AICAgICB4bWxuczpleGlmPSJodHRwOi8vbnMuYWRvYmUuY29tL2V4aWYvMS4w%0ALyI+CiAgICAgICAgIDxleGlmOlBpeGVsWURpbWVuc2lvbj4xMzg8L2V4aWY6%0AUGl4ZWxZRGltZW5zaW9uPgogICAgICAgICA8ZXhpZjpQaXhlbFhEaW1lbnNp%0Ab24+NTcwPC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6%0AVXNlckNvbW1lbnQ+U2NyZWVuc2hvdDwvZXhpZjpVc2VyQ29tbWVudD4KICAg%0AICAgPC9yZGY6RGVzY3JpcHRpb24+CiAgIDwvcmRmOlJERj4KPC94OnhtcG1l%0AdGE+Cr6UaKsAAAAcaURPVAAAAAIAAAAAAAAARQAAACgAAABFAAAARQAAMnwi%0AMMZ6AAAySElEQVR4AexdBXwUVxr/Nm7EICQhRAkJQYO7uxRq0KNc2yu0pXY1%0AoO3VD9q7Kr22V9eruyKlLcULRYJLCAkRIMTd7b7vzb7N7KzNLrthQ977/bIz%0A8+bZ/N/A+89nTzNw8NAW0Kb4uBh2duzYMZ4ljgIBgYBAQCAgEBAICATaLQIa%0AQXTa7dyJgQsEBAICAYGAQEAgYAEBQXQsACRuCwQEAgIBgYBAQCDQfhEQRKf9%0Azp0YuUBAICAQEAgIBAQCFhAQRMcCQOK2QEAg0LER6D94JIyaOJ2BsO7bTyE7%0A42S7BsTTywduuH05uLi4wMG9O2Hn5g1t+jyDR06AAUNGgH9AMLh7eoKbmxts%0A+20tbN+4rk3HITrrOAgIouNkc333EDfwctNAdlkzfHa8SfXoZsa6wtAIF4Py%0AWaUt8L8jjQb5pjK6+WpgQZIbgAbgcH4TbMxqNlW0TfNtxaVNB4md2Wse2nrc%0A7bG/vtF+cO9V8aDBd3XzwQL48Lezqh7D2npLlz0OoeHdoby0BP7z5P2q+nD2%0AQretWAkhoeFQWlIELz/1YJsNN6FPMlx93VJGbuSdbvl1DWzZ8IM8S5wLBOyG%0AgCA6doPSPg29McMTvN0B0oqa4ck/GlQ3+rd+bjAxxtWgvLXt9A9xgWUjcACY%0Atmc3wdsH1ZMkg87tmGErLnYcgqqm7DUPqjrr4IUmJ3eGl+4YwFD4Ycc5ePiD%0AE6oQsabe2ClzYOKMeazdlD+3wZqvPjTaR+S40eDm4WH0njyzsb4ecrbukGdd%0A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%0A4whFMQQQQAABBAIkQNBxbAyCjiOUY7Fu2SLjj87ySm/eHZXNe+PzVjp+Q8cs%0AhkvHbFdqhQACbSdA0HG0J+g4QjkWO6l/llw1PscrvXRTROa/X+O4ZccuhkvH%0Abl9qhwACrS9A0HE0J+g4QjkW44TeOBQujbuwFgEEEEhXgKDjKEfQcYRyLMYJ%0AvXEoXBp3YS0CCCCQrgBBx1Guowed2WOyJU/HzWzdE5VF6yNy6qAsGd03Swb0%0AzJJIbVS+qIzKvzdG5ONd9cfSjMgPyZRjwtK3e0h6dwlJ5UEtuy8qr26JSMmX%0A9cteeGxY+vcIeeJ54ZCMPTRGZ1N5rZTq9ycub22tlXdLa+OrLjs+W8z4le0V%0AseOLf3DozRVjsyVLv3qDHt9/NkeSPxa/9Zs1WvenV9Y+07FDK7fXytSCsAzq%0AFZKeuSEpr4rK2h218rx6ZGJpiYvdv9/6tdTTT7vbYzSvBV8JyXlF4fiqymqR%0ABR9w2TIOwhsEEMi4AEHHkbSjB52Hzs2TLhok1n1Z64WVE/rXnYwskQlBN716%0A0P4qM74alrOHhTUQxMJL/AN9s686Kos1MD2j/+xy68k5MuSI2ABku66p10Xr%0AauSpj+u2feDrudJdQ8aGnbVy2xt6dkxaHrkgT0J6GKs/r5V5Kxp+7rd+dn8b%0Ad9VKd61fv0MBze42orls6f8isuC/LT9Jt8TFHk+69UvH02+722M0r5MGZMmP%0AxsXGZpnf92hovOaFur8ps44FAQQQyKQAQcdRs7MEnV37o9JLe2bMsutAVCr0%0ARNRVT/RHdgvJ59qbMmdp7KQ0Xf9XPuO4bC9caIePbNldK+UHRHrm6f/ae2VJ%0AWPNMtXbIPLSqWlZo+DDL97RXZkDP2Hfn6uc29OzQ3pxy3Vfi8qr2yryxra5H%0AxwaPdE7M5nttEHCtn92fPSbT41SmPVXGZqD2SpilUsPcjS8flN0tPE+3xMUe%0AX7r18+uZTrvbYzSvBJ1EDd4jgEBrCBB0HJU7S9AxHKY35rlPIvLshroelSn6%0AP/Fjj8zy7o4yl5DunJorR3QNSbUW+ceHNbJYezfsct7QsFw0MhaCNutlqZuX%0ANexh8TsWxQYPvydme0w2CJjfU9XPlLH7M+8/0Etov15eV4c5k3JkeJ9Yz9Q/%0AP6qRp9UqU4tfF7vfdOvnxzMT7U7QsS3GKwIItJYAQcdRujMFnec/qZG/f9T0%0A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)

3. 执行添加新结点方法

![截屏2021-09-11 09.05.30](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2009.05.30.png)



## Set

> 无序（添加和取出顺序不一致），没有索引；
>
> 不允许重复元素，最多包含一个 null；
>
> 取出顺序是固定的。



### <u>HashSet</u>

<p class="note note-primary">重要</p>

> HashSet 实际上是 HashMap；
>
> 可以存放 null 值，但是只能存放一个，底层实现；
>
> ```java
> public HashSet() {
>     map = new HashMap<>();
> }
> ```
>
> HashSet 不包吃元素是有序的，取决于 hash 后，再决定索引的结果；
>
> 不能有重复元素。



```java
set.add(new String("tsuiraku"))
set.add(new String("tsuiraku")) // false 无法重复添加
```



**数组链表模拟**

<img src="https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2010.08.29.png" alt="截屏2021-09-11 10.08.29" style="zoom:50%;" />



**<u>源码分析</u>**

> 底层是 HashMap；
>
> 添加元素时，先得到 **hash** 值，转化成索引值；
>
> 找到存储数据表 table，查看索引是否已经存入元素；
>
> 如果没有直接加入；
>
> 如果有，调用 equals 进行比较，如果相同，则放弃添加，如果不相同，则添加到最后；
>
> JDK8中，当一条链表元素个数大于 **TREEIFY_THRESHOLD**（默认8），且 table 大小大于 **MIN_TREEIFY_CAPACITY**（默认64），将会转化为<u>**红黑树**</u>。



1. 执行构造器

![截屏2021-09-11 10.42.31](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2010.42.31.png)

2. 执行 `add` 方法

![截屏2021-09-11 10.42.58](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2010.42.58.png)

3. 执行 `put` 方法

![截屏2021-09-11 10.43.17](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2010.43.17.png)

其中 `hash` 方法计算得到 key 对应的 hash 值

![截屏2021-09-11 10.43.57](https://gitee.com/tsuiraku/typora/raw/master/img/%E6%88%AA%E5%B1%8F2021-09-11%2010.43.57.png)

4. 执行 `putVal` 方法

```java
    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

