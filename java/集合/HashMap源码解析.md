**<span style="font-size: 35px;">ð HashMap æºç è§£æ</span>**

---

>[!TIP]
>
>HashMaoæ¯é¢è¯çéä¸­ä¹éï¼éè¦çéåæï¼å¾ä¹åææ¶é´ä¼æ´çæ´å®æ´çHashMapçç¬è®°ï¼è¿éåè½¬è½½pdaiå¤§ä½¬çæç« å ä½ã@vchicken

## Java7 HashMap

### æ¦è¿°

ä¹æä»¥æ*HashSet*å*HashMap*æ¾å¨ä¸èµ·è®²è§£ï¼æ¯å ä¸ºäºèå¨Javaéæçç¸åçå®ç°ï¼åèä»ä»æ¯å¯¹åèåäºä¸å±åè£ï¼ä¹å°±æ¯è¯´*HashSet*éé¢æä¸ä¸ª*HashMap*(ééå¨æ¨¡å¼)ãå æ­¤æ¬æå°éç¹åæ*HashMap*ã

*HashMap*å®ç°äº*Map*æ¥å£ï¼å³åè®¸æ¾å¥`key`ä¸º`null`çåç´ ï¼ä¹åè®¸æå¥`value`ä¸º`null`çåç´ ï¼é¤è¯¥ç±»æªå®ç°åæ­¥å¤ï¼å¶ä½è·`Hashtable`å¤§è´ç¸åï¼è·*TreeMap*ä¸åï¼è¯¥å®¹å¨ä¸ä¿è¯åç´ é¡ºåºï¼æ ¹æ®éè¦è¯¥å®¹å¨å¯è½ä¼å¯¹åç´ éæ°åå¸ï¼åç´ çé¡ºåºä¹ä¼è¢«éæ°ææ£ï¼å æ­¤ä¸åæ¶é´è¿­ä»£åä¸ä¸ª*HashMap*çé¡ºåºå¯è½ä¼ä¸åã æ ¹æ®å¯¹å²çªçå¤çæ¹å¼ä¸åï¼åå¸è¡¨æä¸¤ç§å®ç°æ¹å¼ï¼ä¸ç§å¼æ¾å°åæ¹å¼(Open addressing)ï¼å¦ä¸ç§æ¯å²çªé¾è¡¨æ¹å¼(Separate chaining with linked lists)ã**Java7ä¸­HashMapéç¨çæ¯å²çªé¾è¡¨æ¹å¼**ã

![HashMap_base](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-10/1cc12aa4-c95b-4464-8903-72c0686a1584_HashMap_base.png)

ä»ä¸å¾å®¹æçåºï¼å¦æéæ©åéçåå¸å½æ°ï¼`put()`å`get()`æ¹æ³å¯ä»¥å¨å¸¸æ°æ¶é´åå®æãä½å¨å¯¹*HashMap*è¿è¡è¿­ä»£æ¶ï¼éè¦éåæ´ä¸ªtableä»¥ååé¢è·çå²çªé¾è¡¨ãå æ­¤å¯¹äºè¿­ä»£æ¯è¾é¢ç¹çåºæ¯ï¼ä¸å®å°*HashMap*çåå§å¤§å°è®¾çè¿å¤§ã

æä¸¤ä¸ªåæ°å¯ä»¥å½±å*HashMap*çæ§è½: åå§å®¹é(inital capacity)åè´è½½ç³»æ°(load factor)ãåå§å®¹éæå®äºåå§`table`çå¤§å°ï¼è´è½½ç³»æ°ç¨æ¥æå®èªå¨æ©å®¹çä¸´çå¼ãå½`entry`çæ°éè¶è¿`capacity*load_factor`æ¶ï¼å®¹å¨å°èªå¨æ©å®¹å¹¶éæ°åå¸ãå¯¹äºæå¥åç´ è¾å¤çåºæ¯ï¼å°åå§å®¹éè®¾å¤§å¯ä»¥åå°éæ°åå¸çæ¬¡æ°ã

å°å¯¹è±¡æ¾å¥å°*HashMap*æ*HashSet*ä¸­æ¶ï¼æä¸¤ä¸ªæ¹æ³éè¦ç¹å«å³å¿: `hashCode()`å`equals()`ã`hashCode()`æ¹æ³**å³å®äºå¯¹è±¡ä¼è¢«æ¾å°åªä¸ª`bucket`é**ï¼å½å¤ä¸ªå¯¹è±¡çåå¸å¼å²çªæ¶ï¼`equals()`æ¹æ³**å³å®äºè¿äºå¯¹è±¡æ¯å¦æ¯âåä¸ä¸ªå¯¹è±¡â**ãæä»¥ï¼å¦æè¦å°èªå®ä¹çå¯¹è±¡æ¾å¥å°`HashMap`æ`HashSet`ä¸­ï¼éè¦@Overrideæ æ³¨`hashCode()`å`equals()`æ¹æ³ã

### get()

`get(Object key)`æ¹æ³æ ¹æ®æå®ç`key`å¼è¿åå¯¹åºç`value`ï¼è¯¥æ¹æ³è°ç¨äº`getEntry(Object key)`å¾å°ç¸åºç`entry`ï¼ç¶åè¿å`entry.getValue()`ãå æ­¤`getEntry()`æ¯ç®æ³çæ ¸å¿ã ç®æ³ææ³æ¯é¦åéè¿`hash()`å½æ°å¾å°å¯¹åº`bucket`çä¸æ ï¼ç¶åä¾æ¬¡éåå²çªé¾è¡¨ï¼éè¿`key.equals(k)`æ¹æ³æ¥å¤æ­æ¯å¦æ¯è¦æ¾çé£ä¸ª`entry`ã

![HashMap_getEntry](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-10/21a7b956-3670-4689-8675-c21784290f05_HashMap_getEntry.png)

ä¸å¾ä¸­`hash(k)&(table.length-1)`ç­ä»·äº`hash(k)%table.length`ï¼åå æ¯*HashMap*è¦æ±`table.length`å¿é¡»æ¯2çææ°ï¼å æ­¤`table.length-1`å°±æ¯äºè¿å¶ä½ä½å¨æ¯1ï¼è·`hash(k)`ç¸ä¸ä¼å°åå¸å¼çé«ä½å¨æ¹æï¼å©ä¸çå°±æ¯ä½æ°äºã

```java
//getEntry()æ¹æ³
final Entry<K,V> getEntry(Object key) {
	......
	int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[hash&(table.length-1)];//å¾å°å²çªé¾è¡¨
         e != null; e = e.next) {//ä¾æ¬¡éåå²çªé¾è¡¨ä¸­çæ¯ä¸ªentry
        Object k;
        //ä¾æ®equals()æ¹æ³å¤æ­æ¯å¦ç¸ç­
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

### put()

`put(K key, V value)`æ¹æ³æ¯å°æå®ç`key, value`å¯¹æ·»å å°`map`éãè¯¥æ¹æ³é¦åä¼å¯¹`map`åä¸æ¬¡æ¥æ¾ï¼çæ¯å¦åå«è¯¥åç»ï¼å¦æå·²ç»åå«åç´æ¥è¿åï¼æ¥æ¾è¿ç¨ç±»ä¼¼äº`getEntry()`æ¹æ³ï¼å¦ææ²¡ææ¾å°ï¼åä¼éè¿`addEntry(int hash, K key, V value, int bucketIndex)`æ¹æ³æå¥æ°ç`entry`ï¼æå¥æ¹å¼ä¸º**å¤´ææ³**ã

![HashMap_addEntry](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-10/f2aebcc7-1a6a-4fb1-8294-bcb6238deec1_HashMap_addEntry.png)

```java
//addEntry()
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);//èªå¨æ©å®¹ï¼å¹¶éæ°åå¸
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = hash & (table.length-1);//hash%table.length
    }
    //å¨å²çªé¾è¡¨å¤´é¨æå¥æ°çentry
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```

### remove()

`remove(Object key)`çä½ç¨æ¯å é¤`key`å¼å¯¹åºç`entry`ï¼è¯¥æ¹æ³çå·ä½é»è¾æ¯å¨`removeEntryForKey(Object key)`éå®ç°çã`removeEntryForKey()`æ¹æ³ä¼é¦åæ¾å°`key`å¼å¯¹åºç`entry`ï¼ç¶åå é¤è¯¥`entry`(ä¿®æ¹é¾è¡¨çç¸åºå¼ç¨)ãæ¥æ¾è¿ç¨è·`getEntry()`è¿ç¨ç±»ä¼¼ã

![HashMap_removeEntryForKey](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-10/0ebfd807-4e00-4c74-8229-8349cae3e639_HashMap_removeEntryForKey.png)

```java
//removeEntryForKey()
final Entry<K,V> removeEntryForKey(Object key) {
	......
	int hash = (key == null) ? 0 : hash(key);
    int i = indexFor(hash, table.length);//hash&(table.length-1)
    Entry<K,V> prev = table[i];//å¾å°å²çªé¾è¡¨
    Entry<K,V> e = prev;
    while (e != null) {//éåå²çªé¾è¡¨
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {//æ¾å°è¦å é¤çentry
            modCount++; size--;
            if (prev == e) table[i] = next;//å é¤çæ¯å²çªé¾è¡¨çç¬¬ä¸ä¸ªentry
            else prev.next = next;
            return e;
        }
        prev = e; e = next;
    }
    return e;
}
```

## Java8 HashMap

Java8 å¯¹ HashMap è¿è¡äºä¸äºä¿®æ¹ï¼æå¤§çä¸åå°±æ¯å©ç¨äºçº¢é»æ ï¼æä»¥å¶ç± **æ°ç»+é¾è¡¨+çº¢é»æ ** ç»æã

æ ¹æ® Java7 HashMap çä»ç»ï¼æä»¬ç¥éï¼æ¥æ¾çæ¶åï¼æ ¹æ® hash å¼æä»¬è½å¤å¿«éå®ä½å°æ°ç»çå·ä½ä¸æ ï¼ä½æ¯ä¹åçè¯ï¼éè¦é¡ºçé¾è¡¨ä¸ä¸ªä¸ªæ¯è¾ä¸å»æè½æ¾å°æä»¬éè¦çï¼æ¶é´å¤æåº¦åå³äºé¾è¡¨çé¿åº¦ï¼ä¸º O(n)ã

ä¸ºäºéä½è¿é¨åçå¼éï¼å¨ Java8 ä¸­ï¼å½é¾è¡¨ä¸­çåç´ è¾¾å°äº 8 ä¸ªæ¶ï¼ä¼å°é¾è¡¨è½¬æ¢ä¸ºçº¢é»æ ï¼å¨è¿äºä½ç½®è¿è¡æ¥æ¾çæ¶åå¯ä»¥éä½æ¶é´å¤æåº¦ä¸º O(logN)ã

æ¥ä¸å¼ å¾ç®åç¤ºæä¸ä¸å§ï¼

![img](https://vue-admin-imgages.oss-cn-hangzhou.aliyuncs.com/2022-09-10/afb57d51-091b-4d24-8b22-6a8a926d091a_java-collection-hashmap8.png)

æ³¨æï¼ä¸å¾æ¯ç¤ºæå¾ï¼ä¸»è¦æ¯æè¿°ç»æï¼ä¸ä¼è¾¾å°è¿ä¸ªç¶æçï¼å ä¸ºè¿ä¹å¤æ°æ®çæ¶åæ©å°±æ©å®¹äºã

ä¸é¢ï¼æä»¬è¿æ¯ç¨ä»£ç æ¥ä»ç»å§ï¼ä¸ªäººæè§ï¼Java8 çæºç å¯è¯»æ§è¦å·®ä¸äºï¼ä¸è¿ç²¾ç®ä¸äºã

Java7 ä¸­ä½¿ç¨ Entry æ¥ä»£è¡¨æ¯ä¸ª HashMap ä¸­çæ°æ®èç¹ï¼Java8 ä¸­ä½¿ç¨ Nodeï¼åºæ¬æ²¡æåºå«ï¼é½æ¯ keyï¼valueï¼hash å next è¿åä¸ªå±æ§ï¼ä¸è¿ï¼Node åªè½ç¨äºé¾è¡¨çæåµï¼çº¢é»æ çæåµéè¦ä½¿ç¨ TreeNodeã

æä»¬æ ¹æ®æ°ç»åç´ ä¸­ï¼ç¬¬ä¸ä¸ªèç¹æ°æ®ç±»åæ¯ Node è¿æ¯ TreeNode æ¥å¤æ­è¯¥ä½ç½®ä¸æ¯é¾è¡¨è¿æ¯çº¢é»æ çã

> [!TIP]
>
> å³äºçº¢é»æ çç¥è¯ï¼å¯ä»¥éè¯»[çº¢é»æ è¯¦è§£](/structures/æ°æ®ç»æåºç¡/çº¢é»æ )ä¸æ

### put è¿ç¨åæ

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// ç¬¬åä¸ªåæ° onlyIfAbsent å¦ææ¯ trueï¼é£ä¹åªæå¨ä¸å­å¨è¯¥ key æ¶æä¼è¿è¡ put æä½
// ç¬¬äºä¸ªåæ° evict æä»¬è¿éä¸å³å¿
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // ç¬¬ä¸æ¬¡ put å¼çæ¶åï¼ä¼è§¦åä¸é¢ç resize()ï¼ç±»ä¼¼ java7 çç¬¬ä¸æ¬¡ put ä¹è¦åå§åæ°ç»é¿åº¦
    // ç¬¬ä¸æ¬¡ resize ååç»­çæ©å®¹æäºä¸ä¸æ ·ï¼å ä¸ºè¿æ¬¡æ¯æ°ç»ä» null åå§åå°é»è®¤ç 16 æèªå®ä¹çåå§å®¹é
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // æ¾å°å·ä½çæ°ç»ä¸æ ï¼å¦ææ­¤ä½ç½®æ²¡æå¼ï¼é£ä¹ç´æ¥åå§åä¸ä¸ Node å¹¶æ¾ç½®å¨è¿ä¸ªä½ç½®å°±å¯ä»¥äº
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    else {// æ°ç»è¯¥ä½ç½®ææ°æ®
        Node<K,V> e; K k;
        // é¦åï¼å¤æ­è¯¥ä½ç½®çç¬¬ä¸ä¸ªæ°æ®åæä»¬è¦æå¥çæ°æ®ï¼key æ¯ä¸æ¯"ç¸ç­"ï¼å¦ææ¯ï¼ååºè¿ä¸ªèç¹
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // å¦æè¯¥èç¹æ¯ä»£è¡¨çº¢é»æ çèç¹ï¼è°ç¨çº¢é»æ çæå¼æ¹æ³ï¼æ¬æä¸å±å¼è¯´çº¢é»æ 
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // å°è¿éï¼è¯´ææ°ç»è¯¥ä½ç½®ä¸æ¯ä¸ä¸ªé¾è¡¨
            for (int binCount = 0; ; ++binCount) {
                // æå¥å°é¾è¡¨çæåé¢(Java7 æ¯æå¥å°é¾è¡¨çæåé¢)
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // TREEIFY_THRESHOLD ä¸º 8ï¼æä»¥ï¼å¦ææ°æå¥çå¼æ¯é¾è¡¨ä¸­çç¬¬ 8 ä¸ª
                    // ä¼è§¦åä¸é¢ç treeifyBinï¼ä¹å°±æ¯å°é¾è¡¨è½¬æ¢ä¸ºçº¢é»æ 
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // å¦æå¨è¯¥é¾è¡¨ä¸­æ¾å°äº"ç¸ç­"ç key(== æ equals)
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // æ­¤æ¶ breakï¼é£ä¹ e ä¸ºé¾è¡¨ä¸­[ä¸è¦æå¥çæ°å¼ç key "ç¸ç­"]ç node
                    break;
                p = e;
            }
        }
        // e!=null è¯´æå­å¨æ§å¼çkeyä¸è¦æå¥çkey"ç¸ç­"
        // å¯¹äºæä»¬åæçputæä½ï¼ä¸é¢è¿ä¸ª if å¶å®å°±æ¯è¿è¡ "å¼è¦ç"ï¼ç¶åè¿åæ§å¼
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // å¦æ HashMap ç±äºæ°æå¥è¿ä¸ªå¼å¯¼è´ size å·²ç»è¶è¿äºéå¼ï¼éè¦è¿è¡æ©å®¹
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

å Java7 ç¨å¾®æç¹ä¸ä¸æ ·çå°æ¹å°±æ¯ï¼Java7 æ¯åæ©å®¹åæå¥æ°å¼çï¼Java8 åæå¼åæ©å®¹ï¼ä¸è¿è¿ä¸ªä¸éè¦ã

### æ°ç»æ©å®¹

resize() æ¹æ³ç¨äºåå§åæ°ç»ææ°ç»æ©å®¹ï¼æ¯æ¬¡æ©å®¹åï¼å®¹éä¸ºåæ¥ç 2 åï¼å¹¶è¿è¡æ°æ®è¿ç§»ã

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) { // å¯¹åºæ°ç»æ©å®¹
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // å°æ°ç»å¤§å°æ©å¤§ä¸å
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // å°éå¼æ©å¤§ä¸å
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // å¯¹åºä½¿ç¨ new HashMap(int initialCapacity) åå§ååï¼ç¬¬ä¸æ¬¡ put çæ¶å
        newCap = oldThr;
    else {// å¯¹åºä½¿ç¨ new HashMap() åå§ååï¼ç¬¬ä¸æ¬¡ put çæ¶å
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;

    // ç¨æ°çæ°ç»å¤§å°åå§åæ°çæ°ç»
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab; // å¦ææ¯åå§åæ°ç»ï¼å°è¿éå°±ç»æäºï¼è¿å newTab å³å¯

    if (oldTab != null) {
        // å¼å§éååæ°ç»ï¼è¿è¡æ°æ®è¿ç§»ã
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // å¦æè¯¥æ°ç»ä½ç½®ä¸åªæåä¸ªåç´ ï¼é£å°±ç®åäºï¼ç®åè¿ç§»è¿ä¸ªåç´ å°±å¯ä»¥äº
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // å¦ææ¯çº¢é»æ ï¼å·ä½æä»¬å°±ä¸å±å¼äº
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    // è¿åæ¯å¤çé¾è¡¨çæåµï¼
                    // éè¦å°æ­¤é¾è¡¨ææä¸¤ä¸ªé¾è¡¨ï¼æ¾å°æ°çæ°ç»ä¸­ï¼å¹¶ä¸ä¿çåæ¥çååé¡ºåº
                    // loHeadãloTail å¯¹åºä¸æ¡é¾è¡¨ï¼hiHeadãhiTail å¯¹åºå¦ä¸æ¡é¾è¡¨ï¼ä»£ç è¿æ¯æ¯è¾ç®åç
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        // ç¬¬ä¸æ¡é¾è¡¨
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        // ç¬¬äºæ¡é¾è¡¨çæ°çä½ç½®æ¯ j + oldCapï¼è¿ä¸ªå¾å¥½çè§£
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### get è¿ç¨åæ

ç¸å¯¹äº put æ¥è¯´ï¼get ççå¤ªç®åäºã

- è®¡ç® key ç hash å¼ï¼æ ¹æ® hash å¼æ¾å°å¯¹åºæ°ç»ä¸æ : hash & (length-1)
- å¤æ­æ°ç»è¯¥ä½ç½®å¤çåç´ æ¯å¦åå¥½å°±æ¯æä»¬è¦æ¾çï¼å¦æä¸æ¯ï¼èµ°ç¬¬ä¸æ­¥
- å¤æ­è¯¥åç´ ç±»åæ¯å¦æ¯ TreeNodeï¼å¦ææ¯ï¼ç¨çº¢é»æ çæ¹æ³åæ°æ®ï¼å¦æä¸æ¯ï¼èµ°ç¬¬åæ­¥
- éåé¾è¡¨ï¼ç´å°æ¾å°ç¸ç­(==æequals)ç key

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // å¤æ­ç¬¬ä¸ä¸ªèç¹æ¯ä¸æ¯å°±æ¯éè¦ç
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // å¤æ­æ¯å¦æ¯çº¢é»æ 
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            // é¾è¡¨éå
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

