**<span style="font-size: 35px;">ð¥ LinkedList æºç è§£æ</span>**

---

## æ¦è¿°

`LinkedList`åæ¶å®ç°äº**Listæ¥å£**å**Dequeæ¥å£**ï¼ä¹å°±æ¯è¯´å®æ¢å¯ä»¥çä½ä¸ä¸ª**é¡ºåºå®¹å¨**ï¼åå¯ä»¥çä½ä¸ä¸ª`éå(Queue)`ï¼åæ¶åå¯ä»¥çä½ä¸ä¸ª`æ (Stack)`ãè¿æ ·çæ¥ï¼*LinkedList*ç®ç´å°±æ¯ä¸ªå¨è½å åãå½ä½ éè¦ä½¿ç¨æ æèéåæ¶ï¼å¯ä»¥èèä½¿ç¨*LinkedList*ï¼ä¸æ¹é¢æ¯å ä¸ºJavaå®æ¹å·²ç»å£°æä¸å»ºè®®ä½¿ç¨*Stack*ç±»ï¼æ´éæ¾çæ¯ï¼Javaéæ ¹æ¬æ²¡æä¸ä¸ªå«å*Queue*çç±»(å®æ¯ä¸ªæ¥å£åå­)ãå³äºæ æéåï¼ç°å¨çé¦éæ¯*ArrayDeque*ï¼å®æçæ¯*LinkedList*(å½ä½æ æéåä½¿ç¨æ¶)æçæ´å¥½çæ§è½ã

![LinkedList_base](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-03/eb7e2301-324a-45d7-8eb0-d2b65b905349_LinkedList_base.png)

*LinkedList*çå®ç°æ¹å¼å³å®äºææè·ä¸æ ç¸å³çæä½é½æ¯çº¿æ§æ¶é´ï¼èå¨é¦æ®µæèæ«å°¾å é¤åç´ åªéè¦å¸¸æ°æ¶é´ãä¸ºè¿½æ±æç*LinkedList*æ²¡æå®ç°åæ­¥(synchronized)ï¼å¦æéè¦å¤ä¸ªçº¿ç¨å¹¶åè®¿é®ï¼å¯ä»¥åéç¨`Collections.synchronizedList()`æ¹æ³å¯¹å¶è¿è¡åè£ã

## LinkedListå®ç°

### åºå±æ°æ®ç»æ

*LinkedList*åºå±**éè¿ååé¾è¡¨å®ç°**ï¼æ¬èå°çéè®²è§£æå¥åå é¤åç´ æ¶ååé¾è¡¨çç»´æ¤è¿ç¨ï¼ä¹å³æ¯ä¹é´è§£è·*List*æ¥å£ç¸å³çå½æ°ï¼èå°*Queue*å*Stack*ä»¥å*Deque*ç¸å³çç¥è¯æ¾å¨ä¸ä¸èè®²ãååé¾è¡¨çæ¯ä¸ªèç¹ç¨åé¨ç±»*Node*è¡¨ç¤ºã*LinkedList*éè¿`first`å`last`å¼ç¨åå«æåé¾è¡¨çç¬¬ä¸ä¸ªåæåä¸ä¸ªåç´ ãæ³¨æè¿éæ²¡ææè°çååï¼å½é¾è¡¨ä¸ºç©ºçæ¶å`first`å`last`é½æå`null`ã

```java
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last; 
```

å¶ä¸­Nodeæ¯ç§æçåé¨ç±»:

```java
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

### æé å½æ°

```java
    /**
     * Constructs an empty list.
     */
    public LinkedList() {
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param  c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

### getFirst(), getLast()

è·åç¬¬ä¸ä¸ªåç´ ï¼ åè·åæåä¸ä¸ªåç´ :

```java
    /**
     * Returns the first element in this list.
     *
     * @return the first element in this list
     * @throws NoSuchElementException if this list is empty
     */
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }

    /**
     * Returns the last element in this list.
     *
     * @return the last element in this list
     * @throws NoSuchElementException if this list is empty
     */
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
```

### removeFirst(), removeLast(), remove(e), remove(index)

`remove()`æ¹æ³ä¹æä¸¤ä¸ªçæ¬ï¼ä¸ä¸ªæ¯å é¤è·æå®åç´ ç¸ç­çç¬¬ä¸ä¸ªåç´ `remove(Object o)`ï¼å¦ä¸ä¸ªæ¯å é¤æå®ä¸æ å¤çåç´ `remove(int index)`ã

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-03/d09a2a1c-bf9b-402a-952a-b09d257898a5.png)

å é¤åç´  - æçæ¯å é¤ç¬¬ä¸æ¬¡åºç°çè¿ä¸ªåç´ , å¦ææ²¡æè¿ä¸ªåç´ ï¼åè¿åfalseï¼å¤æ­çä¾æ®æ¯equalsæ¹æ³ï¼ å¦æequalsï¼åç´æ¥unlinkè¿ä¸ªnodeï¼ç±äºLinkedListå¯å­æ¾nullåç´ ï¼æä¹å¯ä»¥å é¤ç¬¬ä¸æ¬¡åºç°nullçåç´ ï¼

```java
    /**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If this list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns {@code true} if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return {@code true} if this list contained the specified element
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
    
    /**
     * Unlinks non-null node x.
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {// ç¬¬ä¸ä¸ªåç´ 
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {// æåä¸ä¸ªåç´ 
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null; // GC
        size--;
        modCount++;
        return element;
    }
```

`remove(int index)`ä½¿ç¨çæ¯ä¸æ è®¡æ°ï¼ åªéè¦å¤æ­è¯¥indexæ¯å¦æåç´ å³å¯ï¼å¦ææåç´æ¥unlinkè¿ä¸ªnodeã

```java
    /**
     * Removes the element at the specified position in this list.  Shifts any
     * subsequent elements to the left (subtracts one from their indices).
     * Returns the element that was removed from the list.
     *
     * @param index the index of the element to be removed
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
```

å é¤headåç´ :

```java
    /**
     * Removes and returns the first element from this list.
     *
     * @return the first element from this list
     * @throws NoSuchElementException if this list is empty
     */
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }


    /**
     * Unlinks non-null first node f.
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

å é¤laståç´ :

```java
	/**
     * Removes and returns the last element from this list.
     *
     * @return the last element from this list
     * @throws NoSuchElementException if this list is empty
     */
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
    
    /**
     * Unlinks non-null last node l.
     */
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```

### add()

*add()\*æ¹æ³æä¸¤ä¸ªçæ¬ï¼ä¸ä¸ªæ¯`add(E e)`ï¼è¯¥æ¹æ³å¨\*LinkedList*çæ«å°¾æå¥åç´ ï¼å ä¸ºæ`last`æåé¾è¡¨æ«å°¾ï¼å¨æ«å°¾æå¥åç´ çè±è´¹æ¯å¸¸æ°æ¶é´ãåªéè¦ç®åä¿®æ¹å ä¸ªç¸å³å¼ç¨å³å¯ï¼å¦ä¸ä¸ªæ¯`add(int index, E element)`ï¼è¯¥æ¹æ³æ¯å¨æå®ä¸è¡¨å¤æå¥åç´ ï¼éè¦åéè¿çº¿æ§æ¥æ¾æ¾å°å·ä½ä½ç½®ï¼ç¶åä¿®æ¹ç¸å³å¼ç¨å®ææå¥æä½ã

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * <p>This method is equivalent to {@link #addLast}.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

![](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-03/fbb38429-c9e2-4eae-925d-d76ec2d46c5c.png)

`add(int index, E element)`, å½index==sizeæ¶ï¼ç­åäºadd(E e); å¦æä¸æ¯ï¼ååä¸¤æ­¥: 1.åæ ¹æ®indexæ¾å°è¦æå¥çä½ç½®,å³node(index)æ¹æ³ï¼2.ä¿®æ¹å¼ç¨ï¼å®ææå¥æä½ã

```java
    /**
     * Inserts the specified element at the specified position in this list.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

ä¸é¢ä»£ç ä¸­ç`node(int index)`å½æ°æä¸ç¹å°å°çtrickï¼å ä¸ºé¾è¡¨ååçï¼å¯ä»¥ä»å¼å§å¾åæ¾ï¼ä¹å¯ä»¥ä»ç»å°¾å¾åæ¾ï¼å·ä½æé£ä¸ªæ¹åæ¾åå³äºæ¡ä»¶`index < (size >> 1)`ï¼ä¹å³æ¯indexæ¯é è¿åç«¯è¿æ¯åç«¯ãä»è¿éä¹å¯ä»¥çåºï¼linkedListéè¿indexæ£ç´¢åç´ çæçæ²¡æarrayListé«ã

```java
    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

### addAll()

addAll(index, c) å®ç°æ¹å¼å¹¶ä¸æ¯ç´æ¥è°ç¨add(index,e)æ¥å®ç°ï¼ä¸»è¦æ¯å ä¸ºæççé®é¢ï¼å¦ä¸ä¸ªæ¯fail-fastä¸­modCountåªä¼å¢å 1æ¬¡ï¼

```java
    /**
     * Appends all of the elements in the specified collection to the end of
     * this list, in the order that they are returned by the specified
     * collection's iterator.  The behavior of this operation is undefined if
     * the specified collection is modified while the operation is in
     * progress.  (Note that this will occur if the specified collection is
     * this list, and it's nonempty.)
     *
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }

    /**
     * Inserts all of the elements in the specified collection into this
     * list, starting at the specified position.  Shifts the element
     * currently at that position (if any) and any subsequent elements to
     * the right (increases their indices).  The new elements will appear
     * in the list in the order that they are returned by the
     * specified collection's iterator.
     *
     * @param index index at which to insert the first element
     *              from the specified collection
     * @param c collection containing elements to be added to this list
     * @return {@code true} if this list changed as a result of the call
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @throws NullPointerException if the specified collection is null
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

### clear()

ä¸ºäºè®©GCæ´å¿«å¯ä»¥åæ¶æ¾ç½®çåç´ ï¼éè¦å°nodeä¹é´çå¼ç¨å³ç³»èµç©ºã

```java
    /**
     * Removes all of the elements from this list.
     * The list will be empty after this call returns.
     */
    public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```

### Positional Access æ¹æ³

éè¿indexè·ååç´ 

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

å°æä¸ªä½ç½®çåç´ éæ°èµå¼:

```java
    /**
     * Replaces the element at the specified position in this list with the
     * specified element.
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```

å°åç´ æå¥å°æå®indexä½ç½®:

```java
    /**
     * Inserts the specified element at the specified position in this list.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

å é¤æå®ä½ç½®çåç´ :

```java
    /**
     * Removes the element at the specified position in this list.  Shifts any
     * subsequent elements to the left (subtracts one from their indices).
     * Returns the element that was removed from the list.
     *
     * @param index the index of the element to be removed
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
```

å¶å®ä½ç½®çæ¹æ³:

```java
    /**
     * Tells if the argument is the index of an existing element.
     */
    private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }

    /**
     * Tells if the argument is the index of a valid position for an
     * iterator or an add operation.
     */
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }

    /**
     * Constructs an IndexOutOfBoundsException detail message.
     * Of the many possible refactorings of the error handling code,
     * this "outlining" performs best with both server and client VMs.
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

### æ¥æ¾æä½

æ¥æ¾æä½çæ¬è´¨æ¯æ¥æ¾åç´ çä¸æ :

æ¥æ¾ç¬¬ä¸æ¬¡åºç°çindex, å¦ææ¾ä¸å°è¿å-1ï¼

```java
    /**
     * Returns the index of the first occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the lowest index {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     *
     * @param o element to search for
     * @return the index of the first occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     */
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

æ¥æ¾æåä¸æ¬¡åºç°çindex, å¦ææ¾ä¸å°è¿å-1ï¼

```java
    /**
     * Returns the index of the last occurrence of the specified element
     * in this list, or -1 if this list does not contain the element.
     * More formally, returns the highest index {@code i} such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
     * or -1 if there is no such index.
     *
     * @param o element to search for
     * @return the index of the last occurrence of the specified element in
     *         this list, or -1 if this list does not contain the element
     */
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
```

### Queue æ¹æ³

```java
    /**
     * Retrieves, but does not remove, the head (first element) of this list.
     *
     * @return the head of this list, or {@code null} if this list is empty
     * @since 1.5
     */
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    /**
     * Retrieves, but does not remove, the head (first element) of this list.
     *
     * @return the head of this list
     * @throws NoSuchElementException if this list is empty
     * @since 1.5
     */
    public E element() {
        return getFirst();
    }

    /**
     * Retrieves and removes the head (first element) of this list.
     *
     * @return the head of this list, or {@code null} if this list is empty
     * @since 1.5
     */
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    /**
     * Retrieves and removes the head (first element) of this list.
     *
     * @return the head of this list
     * @throws NoSuchElementException if this list is empty
     * @since 1.5
     */
    public E remove() {
        return removeFirst();
    }

    /**
     * Adds the specified element as the tail (last element) of this list.
     *
     * @param e the element to add
     * @return {@code true} (as specified by {@link Queue#offer})
     * @since 1.5
     */
    public boolean offer(E e) {
        return add(e);
    }
```

### Deque æ¹æ³

```java
    /**
     * Inserts the specified element at the front of this list.
     *
     * @param e the element to insert
     * @return {@code true} (as specified by {@link Deque#offerFirst})
     * @since 1.6
     */
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }

    /**
     * Inserts the specified element at the end of this list.
     *
     * @param e the element to insert
     * @return {@code true} (as specified by {@link Deque#offerLast})
     * @since 1.6
     */
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }

    /**
     * Retrieves, but does not remove, the first element of this list,
     * or returns {@code null} if this list is empty.
     *
     * @return the first element of this list, or {@code null}
     *         if this list is empty
     * @since 1.6
     */
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
     }

    /**
     * Retrieves, but does not remove, the last element of this list,
     * or returns {@code null} if this list is empty.
     *
     * @return the last element of this list, or {@code null}
     *         if this list is empty
     * @since 1.6
     */
    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }

    /**
     * Retrieves and removes the first element of this list,
     * or returns {@code null} if this list is empty.
     *
     * @return the first element of this list, or {@code null} if
     *     this list is empty
     * @since 1.6
     */
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }

    /**
     * Retrieves and removes the last element of this list,
     * or returns {@code null} if this list is empty.
     *
     * @return the last element of this list, or {@code null} if
     *     this list is empty
     * @since 1.6
     */
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }

    /**
     * Pushes an element onto the stack represented by this list.  In other
     * words, inserts the element at the front of this list.
     *
     * <p>This method is equivalent to {@link #addFirst}.
     *
     * @param e the element to push
     * @since 1.6
     */
    public void push(E e) {
        addFirst(e);
    }

    /**
     * Pops an element from the stack represented by this list.  In other
     * words, removes and returns the first element of this list.
     *
     * <p>This method is equivalent to {@link #removeFirst()}.
     *
     * @return the element at the front of this list (which is the top
     *         of the stack represented by this list)
     * @throws NoSuchElementException if this list is empty
     * @since 1.6
     */
    public E pop() {
        return removeFirst();
    }

    /**
     * Removes the first occurrence of the specified element in this
     * list (when traversing the list from head to tail).  If the list
     * does not contain the element, it is unchanged.
     *
     * @param o element to be removed from this list, if present
     * @return {@code true} if the list contained the specified element
     * @since 1.6
     */
    public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }

    /**
     * Removes the last occurrence of the specified element in this
     * list (when traversing the list from head to tail).  If the list
     * does not contain the element, it is unchanged.
     *
     * @param o element to be removed from this list, if present
     * @return {@code true} if the list contained the specified element
     * @since 1.6
     */
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```



## åèæç« 

- [Java LinkedListæºç åæ ç»åæºç å¯¹LinkedListè¿è¡è®²è§£](http://www.cnblogs.com/CarpenterLee/p/5457150.html)