**<span style="font-size: 35px;">🫑 ArrayList 源码解析</span>**

---

#### 1.ArrayList的概述

*ArrayList*实现了*List*接口，是顺序容器，即元素存放的数据与放进去的顺序相同，允许放入`null`元素，底层通过**数组实现**。除该类未实现同步外，其余跟*Vector*大致相同。每个*ArrayList*都有一个`容量（capacity）`，表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。前面已经提过，**Java泛型只是编译器提供的语法糖，所以这里的数组是一个Object数组，以便能够容纳任何类型的对象。**

![ArrayList概述](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-03/e5608e2d-5521-4bed-b18b-2ae8e63596f9.png)


size(), isEmpty(), get(), set()方法均能在**常数时间**内完成，add()方法的时间开销跟插入位置有关，addAll()方法的时间开销跟添加元素的个数成正比。其余方法大都是**线性时间**。

注意，此实现不是同步的。如果多个线程同时访问一个ArrayList实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。

#### 2.ArrayList的实现

对于ArrayList而言，它实现List接口、底层使用数组保存所有元素。其操作基本上是对数组的操作。下面我们来分析ArrayList的源代码：

因为 ArrayList 是基于数组实现的，所以支持快速随机访问。RandomAccess 接口标识着该类支持快速随机访问。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

**底层使用数组实现**

```java
 /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```

 **构造方法**

```java
	/**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
	/** 
	* 构造一个具有指定初始容量的空列表。 
	* 
	* @param initialCapacity 列表的初始容量 
	* @throws IllegalArgumentException 如果指定的初始容量 
	* 为负数 
	*/
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     * 构造一个初始容量为 10 的空列表。
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
	/** 
	* 按照集合的迭代器返回的顺序，构造一个包含指定集合的元素的列表。 
	* 
	* @param c 将其元素放入此列表的集合 
	* @throws NullPointerException 如果指定的集合为空 
	*/
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

**自动扩容**

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，即 oldCapacity+oldCapacity/2。其中 oldCapacity >> 1 需要取整，所以新容量大约是旧容量的 1.5 倍左右。（oldCapacity 为偶数就是 1.5 倍，为奇数就是 1.5 倍-0.5）

扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。或者根据实际需求，通过调用ensureCapacity方法来手动增加ArrayList实例的容量。

```java
	/**
     * Increases the capacity of this <tt>ArrayList</tt> instance, if
     * necessary, to ensure that it can hold at least the number of elements
     * specified by the minimum capacity argument.
     *
     * @param   minCapacity   the desired minimum capacity
     */
	/** 
	* 如有必要，增加此 <tt>ArrayList</tt> 实例的容量，以确保它至少可以容纳最小容量参数指定的元素数量。 
	* 
	* @param minCapacity 所需的最小容量 
	*/
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table 如果不是默认元素表，则为任何大小
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            // 大于默认空表的默认值。它已经应该是默认大小。
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            //如果传入的值大于默认值，则开始扩容
            ensureExplicitCapacity(minCapacity);
        }
    }

    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code 溢出意识代码
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
	/** 
	* 要分配的数组的最大大小。 
	* 一些虚拟机在数组中保留一些标题字。 
	* 尝试分配更大的数组可能会导致 
	* OutOfMemoryError：请求的数组大小超过 VM 限制 
	*/
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
	/** 
	* 增加容量以确保它至少可以容纳 
	* 由最小容量参数指定的元素数量。 
	* 
	* @param minCapacity 所需的最小容量 
	*/
    private void grow(int minCapacity) {
        // overflow-conscious code
        // 记录扩容前的数组实际大小
        int oldCapacity = elementData.length;
        // 计算扩容后新数组的容量，即oldCapacity + oldCapacity/2 约等于1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            // 如果新数组的容量小于所需最小的容量，则将扩容后的数组容量设为所需的最小容量
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            // 最大数组容量MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8
            // 如果扩容后数组的容量大于最大数组容量，则判断所需的最小容量是否大于最大数组容量，
            // 是则扩容后容量为Integer.MAX_VALUE，否则为最大数组容量MAX_ARRAY_SIZE
            // 该判断目的是，防止内存溢出
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

![ArrayList自动扩容过程](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-03/e3384599-3b68-4bf2-84ca-74ff4cfd0f40_ArrayList自动扩容过程.png)

**add(), addAll()**

跟C++ 的*vector*不同，*ArrayList*没有`push_back()`方法，对应的方法是`add(E e)`，*ArrayList*也没有`insert()`方法，对应的方法是`add(int index, E e)`。这两个方法都是向容器中添加新元素，这可能会导致*capacity*不足，因此在添加元素之前，都需要进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过`grow()`方法完成的。

```java
	/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
	/** 
	* 将指定元素附加到此列表的末尾。 
	* 
	* @param e 要附加到此列表的元素 
	* @return <tt>true</tt>（由 {@link Collection#add} 指定）
	*/
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
	/** 
	* 在此列表中的指定位置插入指定元素。将当前位于该位置的元素（如果有）和 
	* 任何后续元素向右移动（将它们的索引加一）。 
	* 
	* @param index 要插入指定元素的索引 
	* @param element 要插入的元素 
	* @throws IndexOutOfBoundsException {@inheritDoc} 
	*/
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

	/**
     * A version of rangeCheck used by add and addAll.
     * add 和 addAll 使用的 rangeCheck 版本。
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

`add(int index, E e)`需要先对元素进行移动，然后完成插入操作，也就意味着该方法有着线性的时间复杂度。

`addAll()`方法能够一次添加多个元素，根据位置不同也有两个版本，一个是在末尾添加的`addAll(Collection<? extends E> c)`方法，一个是从指定位置开始插入的`addAll(int index, Collection<? extends E> c)`方法。跟`add()`方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 `addAll()`的时间复杂度不仅跟插入元素的多少有关，也跟插入的位置相关。

```java
	/**
     * Appends all of the elements in the specified collection to the end of
     * this list, in the order that they are returned by the
     * specified collection's Iterator.  The behavior of this operation is
     * undefined if the specified collection is modified while the operation
     * is in progress.  (This implies that the behavior of this call is
     * undefined if the specified collection is this list, and this
     * list is nonempty.)
     *
     * @param c collection containing elements to be added to this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws NullPointerException if the specified collection is null
     */
	/** 
	* 按照指定集合的迭代器返回的顺序，将指定集合中的所有元素附加到此列表的末尾。 
	* 如果在操作进行时修改了指定的集合，则此操作的行为是未定义的。 （这意味着如果指定的集合是此列表，则此调用的行为是未定义的，并且此 
	* 列表是非空的。） 
	* 
	* @param c 包含要添加到此列表的元素的集合 
	* @return <tt>true< /tt> 如果此列表因调用而更改 
	* @throws NullPointerException 如果指定的集合为空 
	*/
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * Inserts all of the elements in the specified collection into this
     * list, starting at the specified position.  Shifts the element
     * currently at that position (if any) and any subsequent elements to
     * the right (increases their indices).  The new elements will appear
     * in the list in the order that they are returned by the
     * specified collection's iterator.
     *
     * @param index index at which to insert the first element from the
     *              specified collection
     * @param c collection containing elements to be added to this list
     * @return <tt>true</tt> if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
	/** 
	* 将指定集合中的所有元素插入此列表 
	* 从指定位置开始。将当前位于该位置的元素 
	* （如果有）和任何后续元素 * 向右移动（增加它们的索引）。新元素将按照 * 指定集合的迭代器返回的顺序出现在列表中。 
	* 
	* @param index 插入指定集合中第一个元素的索引 
	* @param c 包含要添加到此列表的元素的集合 
	* @return <tt>true</tt> 如果此列表由于以下原因而更改调用 * @throws IndexOutOfBoundsException {@inheritDoc} 
	* @throws NullPointerException 如果指定的集合为空 
	*/
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        // 判断插入的位置是否在数组大小范围内
        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
```

**set()**

既然底层是一个数组*ArrayList*的`set()`方法也就变得非常简单，直接对数组的指定位置赋值即可。

```java
    /**
     * Replaces the element at the specified position in this list with
     * the specified element.
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
	/** 
	* 将此列表中指定位置的元素替换为 
	* 指定元素。 
	* 
	* @param index 要替换的元素的索引 
	* @param element 要存储在指定位置的元素 
	* @return 之前在指定位置的元素 
	* @throws IndexOutOfBoundsException {@inheritDoc} 
	*/
    public E set(int index, E element) {
        //下标越界检查
        rangeCheck(index);

        E oldValue = elementData(index);
        //赋值到指定位置，赋值的仅仅是引用
        elementData[index] = element;
        return oldValue;
    }
```

**get()**

`get()`方法同样很简单，唯一要注意的是由于底层数组是Object[]，得到元素后需要进行类型转换。

```java
 	@SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * Returns the element at the specified position in this list.
     *
     * @param  index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
	/** 
	* 返回此列表中指定位置的元素。 
	* 
	* @param index 要返回的元素的索引 
	* @return 此列表中指定位置的元素 
	* @throws IndexOutOfBoundsException {@inheritDoc} 
	*/
    public E get(int index) {
        rangeCheck(index);
	    //注意类型转换
        return elementData(index);
    }
```

**remove()**

`remove()`方法也有两个版本，一个是`remove(int index)`删除指定位置的元素，另一个是`remove(Object o)`删除第一个满足`o.equals(elementData[index])`的元素。删除操作是`add()`操作的逆过程，需要将删除点之后的元素向前移动一个位置。**需要注意的是为了让GC起作用，必须显式的为最后一个位置赋`null`值**。

```java
	/**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
	/** 
	* 移除此列表中指定位置的元素。 
	* 将任何后续元素向左移动（从它们的 * 索引中减去 1）。 
	* 
	* @param index 要删除的元素的索引 
	* @return 从列表中删除的元素 
	* @throws IndexOutOfBoundsException {@inheritDoc} */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    /**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If the list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns <tt>true</tt> if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return <tt>true</tt> if this list contained the specified element
     */
	/** 
	* 从此列表中删除第一个出现的指定元素，
	* 如果它存在。如果列表不包含该元素，则 
	* 不变。更正式地说，删除具有最低索引的元素 
	* <tt>i</tt> 使得 
	* <tt>(o==null ? get(i)==null : o.equals(get(i)))< /tt> 
	*（如果存在这样的元素）。如果此列表 
	* 包含指定的元素（或等效地，如果此列表 
	* 由于调用而更改），则返回 <tt>true</tt>。 
	* 
	* @param o 要从此列表中删除的元素，如果存在 
	* @return <tt>true</tt> 如果此列表包含指定的元素 
	*/
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

 	/*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
	/* 
	* 跳过边界检查并且不返回删除值的私有删除方法。 
	*/
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

关于Java GC这里需要特别说明一下，**有了垃圾收集器并不意味着一定不会有内存泄漏**。对象能否被GC的依据是是否还有引用指向它，上面代码中如果不手动赋`null`值，除非对应的位置被其他元素覆盖，否则原来的对象就一直不会被回收。

**trimToSize()**

ArrayList还给我们提供了**将底层数组的容量调整为当前列表保存的实际元素的大小**的功能。它可以通过trimToSize方法来实现。代码如下:

```java
    /**
     * Trims the capacity of this <tt>ArrayList</tt> instance to be the
     * list's current size.  An application can use this operation to minimize
     * the storage of an <tt>ArrayList</tt> instance.
     */
	/** 
	* 将此 <tt>ArrayList</tt> 实例的容量修剪为 
	* 列表的当前大小。应用程序可以使用此操作来最小化 
	* <tt>ArrayList</tt> 实例的存储。 
	*/
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```

**indexOf(), lastIndexOf()**

获取元素的第一次出现的index:

```java
	/**
     * Returns the index of the first occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the lowest index <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     */
	/** 
	* 返回此列表中第一次出现指定元素 
	* 的索引，如果此列表不包含该元素，则返回 -1。 
	* 更正式地说，返回最低索引 <tt>i</tt> 使得 
	* <tt>(o==null ? get(i)==null : o.equals(get(i)))</tt> , 
	* 或 -1 如果没有这样的索引。 
	*/
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

获取元素的最后一次出现的index:

```java
    /**
     * Returns the index of the last occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the highest index <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     */
	/** 
	* 返回此列表中指定元素 * 的最后一次出现的索引，如果此列表不包含该元素，则返回 -1。 
	* 更正式地说，返回最高索引 <tt>i</tt> 使得 
	* <tt>(o==null ? get(i)==null : o.equals(get(i)))</tt> , 
	* 或 -1 如果没有这样的索引。 
	*/
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

#### Fail-Fast机制

通过以上的源码，我们可以发现，无论是在数组扩容还是add()，remove()等操作时，我们看到代码中都对modCount做了修改，那么modCount是什么？它的作用说明是什么？

首先先来看源码中声明的modCount：

```java
/**
     * The number of times this list has been <i>structurally modified</i>.
     * Structural modifications are those that change the size of the
     * list, or otherwise perturb it in such a fashion that iterations in
     * progress may yield incorrect results.
     *
     * <p>This field is used by the iterator and list iterator implementation
     * returned by the {@code iterator} and {@code listIterator} methods.
     * If the value of this field changes unexpectedly, the iterator (or list
     * iterator) will throw a {@code ConcurrentModificationException} in
     * response to the {@code next}, {@code remove}, {@code previous},
     * {@code set} or {@code add} operations.  This provides
     * <i>fail-fast</i> behavior, rather than non-deterministic behavior in
     * the face of concurrent modification during iteration.
     *
     * <p><b>Use of this field by subclasses is optional.</b> If a subclass
     * wishes to provide fail-fast iterators (and list iterators), then it
     * merely has to increment this field in its {@code add(int, E)} and
     * {@code remove(int)} methods (and any other methods that it overrides
     * that result in structural modifications to the list).  A single call to
     * {@code add(int, E)} or {@code remove(int)} must add no more than
     * one to this field, or the iterators (and list iterators) will throw
     * bogus {@code ConcurrentModificationExceptions}.  If an implementation
     * does not wish to provide fail-fast iterators, this field may be
     * ignored.
     */
	/** 
	* 此列表被<i>结构修改</i>的次数。 
	* 结构修改是那些改变 
	* 列表大小的修改，或者以其他方式扰乱它，使得 
	* 进行中的迭代可能会产生不正确的结果。 
	* 
	* <p>此字段由迭代器和列表迭代器实现使用 
	* 由 {@code iterator} 和 {@code listIterator} 方法返回。 
	* 如果此字段的值发生意外更改，迭代器（或列表 * 迭代器）将在 
	* 响应 {@code next}、{@code remove}、{@code previous}、
	* 中抛出 {@code ConcurrentModificationException} {@code set} 或 {@code add} 操作。这提供了 
	* <i>fail-fast</i> 行为，而不是在迭代期间面对并发修改时的不确定行为。 
	* 
	* <p><b>子类对该字段的使用是可选的。</b> 如果子类 
	* 希望提供快速失败的迭代器（和列表迭代器），那么它 
	* 只需在其 { @code add(int, E)} 和 
	* {@code remove(int)} 方法（以及它覆盖的任何其他方法 
	* 导致对列表的结构修改）。对 
	* {@code add(int, E)} 或 {@code remove(int)} 的单个调用必须向该字段添加不超过 
	* 一个，否则迭代器（和列表迭代器）将抛出 
	* bogus {@code并发修改异常}。如果实现 
	* 不希望提供快速失败的迭代器，则可以 
	* 忽略此字段。 
	*/
    protected transient int modCount = 0;
```

我们知道集合采用了迭代器模式的设计模式，迭代器 Iterator 大家应该不陌生，面对 ArrayList、LinkedList、HashSet 等等，各种底层实现不同的容器，比如ArrayList底层维护的是一个数组，LinkedList是链表结构的，HashSet依赖的是哈希表，在每种容器都有自己特有的数据结构情况下，客户如果需要遍历这些集合中的元素，不需要去了解他们是怎么实现的（到底是链表还是数组等等），只要使用迭代器就可以遍历集合，实现解耦的效果。

这些容器都有对迭代器的实现。 拿 ArrayList 举例子，可以发现在ArrayList中有个 iterator() 方法

```java
public Iterator iterator() {
        return new Itr();
    }
```

方法返回 Itr 对象，而 Itr 是 ArrayList 的内部类，代码如下

```java
    private class Itr implements Iterator<E> {
        /**
         * Index of element to be returned by subsequent call to next.
         */
        int cursor = 0;

        /**
         * Index of element returned by most recent call to next or
         * previous.  Reset to -1 if this element is deleted by a call
         * to remove.
         */
        int lastRet = -1;

        /**
         * The modCount value that the iterator believes that the backing
         * List should have.  If this expectation is violated, the iterator
         * has detected concurrent modification.
         */
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size();
        }

        public E next() {
            checkForComodification();
            try {
                int i = cursor;
                E next = get(i);
                lastRet = i;
                cursor = i + 1;
                return next;
            } catch (IndexOutOfBoundsException e) {
                checkForComodification();
                throw new NoSuchElementException();
            }
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                AbstractList.this.remove(lastRet);
                if (lastRet < cursor)
                    cursor--;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException e) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

其中`modCount`代表该迭代器对象被修改的次数，每个迭代器对象被修改一次，modCount都会加1。注意remove 方法中有这个语句： expectedModCount = modCount;

Itr类里有一个成员变量expectedModCount，它的值为创建Itr对象的时候List的modCount值。用此变量来检验在迭代过程中List对象是否被修改了，如果被修改了则抛出java.util.ConcurrentModificationException异常。在每次调用Itr对象的next()方法的时候都会调用checkForComodification()方法进行一次检验，checkForComodification()方法中做的工作就是比较expectedModCount 和modCount的值是否相等，如果不相等，就认为还有其他对象正在对当前的List进行操作，那个就会抛出ConcurrentModificationException异常。

举个栗子：

```java
  public static void main(String[] args) {
        List list = new ArrayList();
        list.add("a");
        list.add("b");
        list.add("c");
        list.add("d");
        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            String str = (String) iterator.next();
            if (str.equals("b")) {
                list.remove(str);
            } else {
                System.out.println(str);
            }
        }
    }
// 执行结果
a
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at com.vchicken.test.main(test.java:20)
```

```java
 public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

注意：调用 remove 方法的是 strList 对象而不是 iterator，而在 list 的 remove 方法中没有 expectedModCount = modCount 语句！

也就是说，list.remove(str) 执行完后，modCount++，但是没有赋值给 expectedModCount，所以此时 expectedModCount不等于 modCount，所以下一个 next() 方法里的 checkForComodification() 就会抛出异常（再次提醒一下，modCount是在 ArrayList 中并作为全局变量，Itr 是 ArrayList 的内部类）。

如果把 list.remove(str) 改成 iterator.remove() 就正常运行了

上述既是我们经常听到的快速失败(Fail-Fast)机制 ,ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。为了解决这个问题建议使用线程安全的CopyOnWriteArrayList。关于CopyOnWriteArrayList详解可查看[JUC集合—ConcurrentHashMap等详解](http://vchicken.cn/note/#/java/并发框架?id=juc集合concurrenthashmap等详解)一文。



#### 参考文章

- [深入Java集合学习系列: ArrayList的实现原理](http://zhangshixi.iteye.com/blog/674856)
- [Java ArrayList源码剖析 结合源码对ArrayList进行讲解](http://www.cnblogs.com/CarpenterLee/p/5419880.html)
- [聊聊 ArrayList 中的modcount](https://www.it610.com/article/1290130213748940800.htm)
