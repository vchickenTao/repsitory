**<span style="font-size: 35px;">ð¥¬ Javaéåä½¿ç¨æ³¨æäºé¡¹æ»ç»</span>**

---

è¿ç¯æç« ææ ¹æ®ãé¿éå·´å·´ Java å¼åæåãæ»ç»äºå³äºéåä½¿ç¨å¸¸è§çæ³¨æäºé¡¹ä»¥åå¶å·ä½åçã

å¼ºçå»ºè®®å°ä¼ä¼´ä»¬å¤å¤éè¯»å éï¼é¿åèªå·±åä»£ç çæ¶ååºç°è¿äºä½çº§çé®é¢ã

## éåå¤ç©º

ãé¿éå·´å·´ Java å¼åæåãçæè¿°å¦ä¸ï¼

> **å¤æ­ææéååé¨çåç´ æ¯å¦ä¸ºç©ºï¼ä½¿ç¨ `isEmpty()` æ¹æ³ï¼èä¸æ¯ `size()==0` çæ¹å¼ã**

è¿æ¯å ä¸º `isEmpty()` æ¹æ³çå¯è¯»æ§æ´å¥½ï¼å¹¶ä¸æ¶é´å¤æåº¦ä¸º O(1)ã

ç»å¤§é¨åæä»¬ä½¿ç¨çéåç `size()` æ¹æ³çæ¶é´å¤æåº¦ä¹æ¯ O(1)ï¼ä¸è¿ï¼ä¹æå¾å¤å¤æåº¦ä¸æ¯ O(1) çï¼æ¯å¦ `java.util.concurrent` åä¸çæäºéåï¼`ConcurrentLinkedQueue` ã`ConcurrentHashMap`...ï¼ã

ä¸é¢æ¯ `ConcurrentHashMap` ç `size()` æ¹æ³å `isEmpty()` æ¹æ³çæºç ã

```java
public int size() {
    long n = sumCount();
    return ((n < 0L) ? 0 :
            (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
            (int)n);
}
final long sumCount() {
    CounterCell[] as = counterCells; CounterCell a;
    long sum = baseCount;
    if (as != null) {
        for (int i = 0; i < as.length; ++i) {
            if ((a = as[i]) != null)
                sum += a.value;
        }
    }
    return sum;
}
public boolean isEmpty() {
    return sumCount() <= 0L; // ignore transient negative values
}
```

## éåè½¬ Map

ãé¿éå·´å·´ Java å¼åæåãçæè¿°å¦ä¸ï¼

> **å¨ä½¿ç¨ `java.util.stream.Collectors` ç±»ç `toMap()` æ¹æ³è½¬ä¸º `Map` éåæ¶ï¼ä¸å®è¦æ³¨æå½ value ä¸º null æ¶ä¼æ NPE å¼å¸¸ã**

```java
class Person {
    private String name;
    private String phoneNumber;
     // getters and setters
}

List<Person> bookList = new ArrayList<>();
bookList.add(new Person("jack","18163138123"));
bookList.add(new Person("martin",null));
// ç©ºæéå¼å¸¸
bookList.stream().collect(Collectors.toMap(Person::getName, Person::getPhoneNumber));
```

ä¸é¢æä»¬æ¥è§£éä¸ä¸åå ã

é¦åï¼æä»¬æ¥ç `java.util.stream.Collectors` ç±»ç `toMap()` æ¹æ³ ï¼å¯ä»¥çå°å¶åé¨è°ç¨äº `Map` æ¥å£ç `merge()` æ¹æ³ã

```java
public static <T, K, U, M extends Map<K, U>>
Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                            Function<? super T, ? extends U> valueMapper,
                            BinaryOperator<U> mergeFunction,
                            Supplier<M> mapSupplier) {
    BiConsumer<M, T> accumulator
            = (map, element) -> map.merge(keyMapper.apply(element),
                                          valueMapper.apply(element), mergeFunction);
    return new CollectorImpl<>(mapSupplier, accumulator, mapMerger(mergeFunction), CH_ID);
}
```

`Map` æ¥å£ç `merge()` æ¹æ³å¦ä¸ï¼è¿ä¸ªæ¹æ³æ¯æ¥å£ä¸­çé»è®¤å®ç°ã

> å¦æä½ è¿ä¸äºè§£ Java 8 æ°ç¹æ§çè¯ï¼è¯·çè¿ç¯æç« ï¼[ãJava8 æ°ç¹æ§æ»ç»ã](https://mp.weixin.qq.com/s/ojyl7B6PiHaTWADqmUq2rw) ã

```java
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```

`merge()` æ¹æ³ä¼åè°ç¨ `Objects.requireNonNull()` æ¹æ³å¤æ­ value æ¯å¦ä¸ºç©ºã

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

## éåéå

ãé¿éå·´å·´ Java å¼åæåãçæè¿°å¦ä¸ï¼

> **ä¸è¦å¨ foreach å¾ªç¯éè¿è¡åç´ ç `remove/add` æä½ãremove åç´ è¯·ä½¿ç¨ `Iterator` æ¹å¼ï¼å¦æå¹¶åæä½ï¼éè¦å¯¹ `Iterator` å¯¹è±¡å éã**

éè¿åç¼è¯ä½ ä¼åç° foreach è¯­æ³åºå±å¶å®è¿æ¯ä¾èµ `Iterator` ãä¸è¿ï¼ `remove/add` æä½ç´æ¥è°ç¨çæ¯éåèªå·±çæ¹æ³ï¼èä¸æ¯ `Iterator` ç `remove/add`æ¹æ³

è¿å°±å¯¼è´ `Iterator` è«åå¶å¦å°åç°èªå·±æåç´ è¢« `remove/add` ï¼ç¶åï¼å®å°±ä¼æåºä¸ä¸ª `ConcurrentModificationException` æ¥æç¤ºç¨æ·åçäºå¹¶åä¿®æ¹å¼å¸¸ãè¿å°±æ¯åçº¿ç¨ç¶æä¸äº§çç **fail-fast æºå¶**ã

> **fail-fast æºå¶** ï¼å¤ä¸ªçº¿ç¨å¯¹ fail-fast éåè¿è¡ä¿®æ¹çæ¶åï¼å¯è½ä¼æåº`ConcurrentModificationException`ã å³ä½¿æ¯åçº¿ç¨ä¸ä¹æå¯è½ä¼åºç°è¿ç§æåµï¼ä¸é¢å·²ç»æå°è¿ã

Java8 å¼å§ï¼å¯ä»¥ä½¿ç¨ `Collection#removeIf()`æ¹æ³å é¤æ»¡è¶³ç¹å®æ¡ä»¶çåç´ ,å¦

```java
List<Integer> list = new ArrayList<>();
for (int i = 1; i <= 10; ++i) {
    list.add(i);
}
list.removeIf(filter -> filter % 2 == 0); /* å é¤listä¸­çææå¶æ° */
System.out.println(list); /* [1, 3, 5, 7, 9] */
```

é¤äºä¸é¢ä»ç»çç´æ¥ä½¿ç¨ `Iterator` è¿è¡éåæä½ä¹å¤ï¼ä½ è¿å¯ä»¥ï¼

- ä½¿ç¨æ®éç for å¾ªç¯
- ä½¿ç¨ fail-safe çéåç±»ã`java.util`åä¸é¢çææçéåç±»é½æ¯ fail-fast çï¼è`java.util.concurrent`åä¸é¢çææçç±»é½æ¯ fail-safe çã
- ......

## éåå»é

ãé¿éå·´å·´ Java å¼åæåãçæè¿°å¦ä¸ï¼

> **å¯ä»¥å©ç¨ `Set` åç´ å¯ä¸çç¹æ§ï¼å¯ä»¥å¿«éå¯¹ä¸ä¸ªéåè¿è¡å»éæä½ï¼é¿åä½¿ç¨ `List` ç `contains()` è¿è¡éåå»éæèå¤æ­åå«æä½ã**

è¿éæä»¬ä»¥ `HashSet` å `ArrayList` ä¸ºä¾è¯´æã

```java
// Set å»éä»£ç ç¤ºä¾
public static <T> Set<T> removeDuplicateBySet(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new HashSet<>();
    }
    return new HashSet<>(data);
}

// List å»éä»£ç ç¤ºä¾
public static <T> List<T> removeDuplicateByList(List<T> data) {

    if (CollectionUtils.isEmpty(data)) {
        return new ArrayList<>();

    }
    List<T> result = new ArrayList<>(data.size());
    for (T current : data) {
        if (!result.contains(current)) {
            result.add(current);
        }
    }
    return result;
}
```

ä¸¤èçæ ¸å¿å·®å«å¨äº `contains()` æ¹æ³çå®ç°ã

`HashSet` ç `contains()` æ¹æ³åºé¨ä¾èµç `HashMap` ç `containsKey()` æ¹æ³ï¼æ¶é´å¤æåº¦æ¥è¿äº Oï¼1ï¼ï¼æ²¡æåºç°åå¸å²çªçæ¶åä¸º Oï¼1ï¼ï¼ã

```java
private transient HashMap<E,Object> map;
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

æä»¬æ N ä¸ªåç´ æå¥è¿ Set ä¸­ï¼é£æ¶é´å¤æåº¦å°±æ¥è¿æ¯ O (n)ã

`ArrayList` ç `contains()` æ¹æ³æ¯éè¿éåææåç´ çæ¹æ³æ¥åçï¼æ¶é´å¤æåº¦æ¥è¿æ¯ O(n)ã

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

æä»¬ç `List` æ N ä¸ªåç´ ï¼é£æ¶é´å¤æåº¦å°±æ¥è¿æ¯ O (n^2)ã

## éåè½¬æ°ç»

ãé¿éå·´å·´ Java å¼åæåãçæè¿°å¦ä¸ï¼

> **ä½¿ç¨éåè½¬æ°ç»çæ¹æ³ï¼å¿é¡»ä½¿ç¨éåç `toArray(T[] array)`ï¼ä¼ å¥çæ¯ç±»åå®å¨ä¸è´ãé¿åº¦ä¸º 0 çç©ºæ°ç»ã**

`toArray(T[] array)` æ¹æ³çåæ°æ¯ä¸ä¸ªæ³åæ°ç»ï¼å¦æ `toArray` æ¹æ³ä¸­æ²¡æä¼ éä»»ä½åæ°çè¯è¿åçæ¯ `Object`ç±» åæ°ç»ã

```java
String [] s= new String[]{
    "dog", "lazy", "a", "over", "jumps", "fox", "brown", "quick", "A"
};
List<String> list = Arrays.asList(s);
Collections.reverse(list);
//æ²¡ææå®ç±»åçè¯ä¼æ¥é
s=list.toArray(new String[0]);
```

ç±äº JVM ä¼åï¼`new String[0]`ä½ä¸º`Collection.toArray()`æ¹æ³çåæ°ç°å¨ä½¿ç¨æ´å¥½ï¼`new String[0]`å°±æ¯èµ·ä¸ä¸ªæ¨¡æ¿çä½ç¨ï¼æå®äºè¿åæ°ç»çç±»åï¼0 æ¯ä¸ºäºèçç©ºé´ï¼å ä¸ºå®åªæ¯ä¸ºäºè¯´æè¿åçç±»åãè¯¦è§ï¼[https://shipilev.net/blog/2016/arrays-wisdom-ancients/open in new window](https://shipilev.net/blog/2016/arrays-wisdom-ancients/)

## æ°ç»è½¬éå

ãé¿éå·´å·´ Java å¼åæåãçæè¿°å¦ä¸ï¼

> **ä½¿ç¨å·¥å·ç±» `Arrays.asList()` ææ°ç»è½¬æ¢æéåæ¶ï¼ä¸è½ä½¿ç¨å¶ä¿®æ¹éåç¸å³çæ¹æ³ï¼ å®ç `add/remove/clear` æ¹æ³ä¼æåº `UnsupportedOperationException` å¼å¸¸ã**

æå¨ä¹åçä¸ä¸ªé¡¹ç®ä¸­å°±éå°ä¸ä¸ªç±»ä¼¼çåã

`Arrays.asList()`å¨å¹³æ¶å¼åä¸­è¿æ¯æ¯è¾å¸¸è§çï¼æä»¬å¯ä»¥ä½¿ç¨å®å°ä¸ä¸ªæ°ç»è½¬æ¢ä¸ºä¸ä¸ª `List` éåã

```java
String[] myArray = {"Apple", "Banana", "Orange"};
List<String> myList = Arrays.asList(myArray);
//ä¸é¢ä¸¤ä¸ªè¯­å¥ç­ä»·äºä¸é¢ä¸æ¡è¯­å¥
List<String> myList = Arrays.asList("Apple","Banana", "Orange");
```

JDK æºç å¯¹äºè¿ä¸ªæ¹æ³çè¯´æï¼

```java
/**
  *è¿åç±æå®æ°ç»æ¯æçåºå®å¤§å°çåè¡¨ãæ­¤æ¹æ³ä½ä¸ºåºäºæ°ç»ååºäºéåçAPIä¹é´çæ¡¥æ¢ï¼
  * ä¸ Collection.toArray()ç»åä½¿ç¨ãè¿åçListæ¯å¯åºååå¹¶å®ç°RandomAccessæ¥å£ã
  */
public static <T> List<T> asList(T... a) {
    return new ArrayList<>(a);
}
```

ä¸é¢æä»¬æ¥æ»ç»ä¸ä¸ä½¿ç¨æ³¨æäºé¡¹ã

**1ã`Arrays.asList()`æ¯æ³åæ¹æ³ï¼ä¼ éçæ°ç»å¿é¡»æ¯å¯¹è±¡æ°ç»ï¼èä¸æ¯åºæ¬ç±»åã**

```java
int[] myArray = {1, 2, 3};
List myList = Arrays.asList(myArray);
System.out.println(myList.size());//1
System.out.println(myList.get(0));//æ°ç»å°åå¼
System.out.println(myList.get(1));//æ¥éï¼ArrayIndexOutOfBoundsException
int[] array = (int[]) myList.get(0);
System.out.println(array[0]);//1
```

å½ä¼ å¥ä¸ä¸ªåçæ°æ®ç±»åæ°ç»æ¶ï¼`Arrays.asList()` ççæ­£å¾å°çåæ°å°±ä¸æ¯æ°ç»ä¸­çåç´ ï¼èæ¯æ°ç»å¯¹è±¡æ¬èº«ï¼æ­¤æ¶ `List` çå¯ä¸åç´ å°±æ¯è¿ä¸ªæ°ç»ï¼è¿ä¹å°±è§£éäºä¸é¢çä»£ç ã

æä»¬ä½¿ç¨åè£ç±»åæ°ç»å°±å¯ä»¥è§£å³è¿ä¸ªé®é¢ã

```java
Integer[] myArray = {1, 2, 3};
```

**2ãä½¿ç¨éåçä¿®æ¹æ¹æ³: `add()`ã`remove()`ã`clear()`ä¼æåºå¼å¸¸ã**

```java
List myList = Arrays.asList(1, 2, 3);
myList.add(4);//è¿è¡æ¶æ¥éï¼UnsupportedOperationException
myList.remove(1);//è¿è¡æ¶æ¥éï¼UnsupportedOperationException
myList.clear();//è¿è¡æ¶æ¥éï¼UnsupportedOperationException
```

`Arrays.asList()` æ¹æ³è¿åçå¹¶ä¸æ¯ `java.util.ArrayList` ï¼èæ¯ `java.util.Arrays` çä¸ä¸ªåé¨ç±»,è¿ä¸ªåé¨ç±»å¹¶æ²¡æå®ç°éåçä¿®æ¹æ¹æ³æèè¯´å¹¶æ²¡æéåè¿äºæ¹æ³ã

```java
List myList = Arrays.asList(1, 2, 3);
System.out.println(myList.getClass());//class java.util.Arrays$ArrayList
```

ä¸å¾æ¯ `java.util.Arrays$ArrayList` çç®ææºç ï¼æä»¬å¯ä»¥çå°è¿ä¸ªç±»éåçæ¹æ³æåªäºã

```java
  private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        ...

        @Override
        public E get(int index) {
          ...
        }

        @Override
        public E set(int index, E element) {
          ...
        }

        @Override
        public int indexOf(Object o) {
          ...
        }

        @Override
        public boolean contains(Object o) {
           ...
        }

        @Override
        public void forEach(Consumer<? super E> action) {
          ...
        }

        @Override
        public void replaceAll(UnaryOperator<E> operator) {
          ...
        }

        @Override
        public void sort(Comparator<? super E> c) {
          ...
        }
    }
```

æä»¬åçä¸ä¸`java.util.AbstractList`ç `add/remove/clear` æ¹æ³å°±ç¥éä¸ºä»ä¹ä¼æåº `UnsupportedOperationException` äºã

```java
public E remove(int index) {
    throw new UnsupportedOperationException();
}
public boolean add(E e) {
    add(size(), e);
    return true;
}
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}

public void clear() {
    removeRange(0, size());
}
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```

**é£æä»¬å¦ä½æ­£ç¡®çå°æ°ç»è½¬æ¢ä¸º `ArrayList` ?**

1ãæå¨å®ç°å·¥å·ç±»

```java
//JDK1.5+
static <T> List<T> arrayToList(final T[] array) {
  final List<T> l = new ArrayList<T>(array.length);

  for (final T s : array) {
    l.add(s);
  }
  return l;
}


Integer [] myArray = { 1, 2, 3 };
System.out.println(arrayToList(myArray).getClass());//class java.util.ArrayList
```

2ãæç®ä¾¿çæ¹æ³

```java
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```

3ãä½¿ç¨ Java8 ç `Stream`(æ¨è)

```java
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
//åºæ¬ç±»åä¹å¯ä»¥å®ç°è½¬æ¢ï¼ä¾èµboxedçè£ç®±æä½ï¼
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

4ãä½¿ç¨ Guava

å¯¹äºä¸å¯åéåï¼ä½ å¯ä»¥ä½¿ç¨[`ImmutableList`open in new window](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java)ç±»åå¶[`of()`open in new window](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L101)ä¸[`copyOf()`open in new window](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java#L225)å·¥åæ¹æ³ï¼ï¼åæ°ä¸è½ä¸ºç©ºï¼

```java
List<String> il = ImmutableList.of("string", "elements");  // from varargs
List<String> il = ImmutableList.copyOf(aStringArray);      // from array
```

å¯¹äºå¯åéåï¼ä½ å¯ä»¥ä½¿ç¨[`Lists`open in new window](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java)ç±»åå¶[`newArrayList()`open in new window](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/Lists.java#L87)å·¥åæ¹æ³ï¼

```java
List<String> l1 = Lists.newArrayList(anotherListOrCollection);    // from collection
List<String> l2 = Lists.newArrayList(aStringArray);               // from array
List<String> l3 = Lists.newArrayList("or", "string", "elements"); // from varargs
```

5ãä½¿ç¨ Apache Commons Collections

```java
List<String> list = new ArrayList<String>();
CollectionUtils.addAll(list, str);
```

6ã ä½¿ç¨ Java9 ç `List.of()`æ¹æ³

```java
Integer[] array = {1, 2, 3};
List<Integer> list = List.of(array);
```



## åèæç« 

- æ¬æè½¬è½½èª[JavaGuide](https://javaguide.cn/java/collection/java-collection-precautions-for-use.html)