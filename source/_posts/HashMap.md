---
title: HashMap相关
date: 2017-10-25 15:00:58
tags: code
---

HashMap继承AbstractMap，而AbstractMap实现了map接口

```java
110
111    public boolean containsValue(Object value) {
112        Iterator<Entry<K,V>> i = entrySet().iterator();
113        if (value==null) {
114            while (i.hasNext()) {
115                Entry<K,V> e = i.next();
116                if (e.getValue()==null)
117                    return true;
118            }
119        } else {
120            while (i.hasNext()) {
121                Entry<K,V> e = i.next();
122                if (value.equals(e.getValue()))
123                    return true;
124            }
125        }
126        return false;
127    }
```

key  和value都可以为null

```java
452    public boolean equals(Object o) {
453        if (o == this)
454            return true;
455
456        if (!(o instanceof Map))
457            return false;
458        Map<?,?> m = (Map<?,?>) o;
459        if (m.size() != size())
460            return false;
461
462        try {
463            Iterator<Entry<K,V>> i = entrySet().iterator();
464            while (i.hasNext()) {
465                Entry<K,V> e = i.next();
466                K key = e.getKey();
467                V value = e.getValue();
468                if (value == null) {
469                    if (!(m.get(key)==null && m.containsKey(key)))
470                        return false;
471                } else {
472                    if (!value.equals(m.get(key)))
473                        return false;
474                }
475            }
476        } catch (ClassCastException unused) {
477            return false;
478        } catch (NullPointerException unused) {
479            return false;
480        }
481
482        return true;
483    }
```

equals方法被重写，除了对比是否同一个对象，如果不是，还得对齐key，还有value的equals方法比较value

所以可以注重value类型的equal方法的重写

当都保持对齐，返回true

```java
502
503    public int hashCode() {
504        int h = 0;
505        Iterator<Entry<K,V>> i = entrySet().iterator();
506        while (i.hasNext())
507            h += i.next().hashCode();
508        return h;
509    }
```

哈希是子项哈希的和

```java
548
549    protected Object clone() throws CloneNotSupportedException {
550        AbstractMap<?,?> result = (AbstractMap<?,?>)super.clone();
551        result.keySet = null;
552        result.values = null;
553        return result;
554    }
```

克隆浅拷贝，不拷贝keyvalues

eq方法写的有意思

```java
561
562    private static boolean eq(Object o1, Object o2) {
563        return o1 == null ? o2 == null : o1.equals(o2);
564    }
```



put方法等待被重写

```java
207
208    public V put(K key, V value) {
209        throw new UnsupportedOperationException();
210    }
```

内部类SimpleEntry，SimpleImmutableEntry实现了map接口内部接口entry

# HashMap

先要知道啥叫哈希表，给定表M，存在函数f(key)，对任意给定的关键字值key，代入函数后若能得到包含该关键字的记录在表中的地址，则称表M为哈希(Hash）表，函数f(key)为哈希(Hash) 函数。

map 被存储在桶的哈希表里。（每个表单位就是一个桶），但是当桶已经满了时候，当前的桶会转化成树节点。数据结构类似Tree Map，在哈希表里面，尽可能只用正常的桶，每个桶的数据结构也可能被转化成树节点。桶上的节点会被遍历和使用，跟哈希表非树节点的保持平级。当数量过大，查找会更快（因为通过hash得到桶的index，时间复杂度只有O（1）,而如果当前桶的数据结构为链表节点（树节点）时，再通过链表查找O（n））

在定位桶的index用到map的key的hashcode函数，而如果桶为多节点链表时，则通过key的equal函数找到位置。

盗图-->

![HashMap](http://images2015.cnblogs.com/blog/1024555/201611/1024555-20161113235348670-746615111.png)



四个构造函数，从无参构造入口，只设置了加载因子，DEFAULT_LOAD_FACTOR=0.75



然后看put方法

```java
609 
610     public V put(K key, V value) {
611         return putVal(hash(key), key, value, false, true);
612     }
```

hash方法

```java
335 
336     static final int hash(Object key) {
337         int h;
338         return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
339     }

```



putVal方法

```java
623 
624     final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
625                    boolean evict) {
626         Node<K,V>[] tab; Node<K,V> p; int n, i;
627         if ((tab = table) == null || (n = tab.length) == 0)
628             n = (tab = resize()).length;
629         if ((p = tab[i = (n - 1) & hash]) == null)
630             tab[i] = newNode(hash, key, value, null);
631         else {
632             Node<K,V> e; K k;
633             if (p.hash == hash &&
634                 ((k = p.key) == key || (key != null && key.equals(k))))
635                 e = p;
636             else if (p instanceof TreeNode)
637                 e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
638             else {
639                 for (int binCount = 0; ; ++binCount) {
640                     if ((e = p.next) == null) {
641                         p.next = newNode(hash, key, value, null);
642                         if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
643                             treeifyBin(tab, hash);
644                         break;
645                     }
646                     if (e.hash == hash &&
647                         ((k = e.key) == key || (key != null && key.equals(k))))
648                         break;
649                     p = e;
650                 }
651             }
652             if (e != null) { // existing mapping for key
653                 V oldValue = e.value;
654                 if (!onlyIfAbsent || oldValue == null)
655                     e.value = value;
656                 afterNodeAccess(e);
657                 return oldValue;
658             }
659         }
660         ++modCount;
661         if (++size > threshold)
662             resize();
663         afterNodeInsertion(evict);
664         return null;
665     }
```

第一次进来会调用resize方法，让tab初始化和n作为tab长度初始化(627)

resize方法

```java
 final Node<K,V>[] resize() {
677         Node<K,V>[] oldTab = table;
678         int oldCap = (oldTab == null) ? 0 : oldTab.length;
679         int oldThr = threshold;
680         int newCap, newThr = 0;
681         if (oldCap > 0) {
682             if (oldCap >= MAXIMUM_CAPACITY) {
683                 threshold = Integer.MAX_VALUE;
684                 return oldTab;
685             }
686             else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
687                      oldCap >= DEFAULT_INITIAL_CAPACITY)
688                 newThr = oldThr << 1; // double threshold
689         }
690         else if (oldThr > 0) // initial capacity was placed in threshold
691             newCap = oldThr;
692         else {               // zero initial threshold signifies using defaults
693             newCap = DEFAULT_INITIAL_CAPACITY;
694             newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
695         }
696         if (newThr == 0) {
697             float ft = (float)newCap * loadFactor;
698             newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
699                       (int)ft : Integer.MAX_VALUE);
700         }
701         threshold = newThr;
702         @SuppressWarnings({"rawtypes","unchecked"})
703             Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
704         table = newTab;
705         if (oldTab != null) {
706             for (int j = 0; j < oldCap; ++j) {
707                 Node<K,V> e;
708                 if ((e = oldTab[j]) != null) {
709                     oldTab[j] = null;
710                     if (e.next == null)
711                         newTab[e.hash & (newCap - 1)] = e;
712                     else if (e instanceof TreeNode)
713                         ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
714                     else { // preserve order
715                         Node<K,V> loHead = null, loTail = null;
716                         Node<K,V> hiHead = null, hiTail = null;
717                         Node<K,V> next;
718                         do {
719                             next = e.next;
720                             if ((e.hash & oldCap) == 0) {
721                                 if (loTail == null)
722                                     loHead = e;
723                                 else
724                                     loTail.next = e;
725                                 loTail = e;
726                             }
727                             else {
728                                 if (hiTail == null)
729                                     hiHead = e;
730                                 else
731                                     hiTail.next = e;
732                                 hiTail = e;
733                             }
734                         } while ((e = next) != null);
735                         if (loTail != null) {
736                             loTail.next = null;
737                             newTab[j] = loHead;
738                         }
739                         if (hiTail != null) {
740                             hiTail.next = null;
741                             newTab[j + oldCap] = hiHead;
742                         }
743                     }
744                 }
745             }
746         }
747         return newTab;
748     }
```



Node内部类实现了Map.Entry，查看equal方法

```java
          public final boolean equals(Object o) {
306             if (o == this)
307                 return true;
308             if (o instanceof Map.Entry) {
309                 Map.Entry<?,?> e = (Map.Entry<?,?>)o;
310                 if (Objects.equals(key, e.getKey()) &&
311                     Objects.equals(value, e.getValue()))
312                     return true;
313             }
314             return false;
315         }
```

这个方法对链表寻址似乎没有影响

回头看resize方法，如果哈希表的长度已经超出了MAXIMUM_CAPACITY，threshold会被赋值为Integer的最大值

而这个threshold就是下次扩容的值，capacity * load factor 的结果值

686：如果旧的长度没有超出哈希表的默认最长长度，首先就是长度翻倍扩容

 而如果翻倍后，还是没有超过默认最长哈希长度，并且旧的长度大于等于初始长度，那么设置下次扩容，应该为两倍扩容

690如果旧的长度小于等于0，那么新的长度设置为扩容值。

693默认其他情况，



681-695总结：第一次进来，会初始化哈希表的长度为初始长度，下次扩容为capacity * load factor 的结果值

而如果表长度不够的时候，会进行翻倍，并且下次扩容的threshold也翻倍，直到超过最大值，如果超过最大值，下次扩容直接就是Integer最大值。



705开始进行一个扩容拷贝操作

treenode部分还未知，先略过

再返回上一层，查看putVal方法

629这行

```java
 if ((p = tab[i = (n - 1) & hash]) == null)
 tab[i] = newNode(hash, key, value, null);
```

新节点的位置通过这两句就确定下来了，确定的位置i，i的决定因素有hash整形数，和n，哈希表的长度减一，一个与运算，保证了位置一定在哈希表的范围内，并且位置和hash相关。如果在这个位置已经有了p!=null。说明hash一致。就不会new对象，如果没有，就执行了newNode方法

查看一下newNode方法

```java
 /*
1725     * The following package-protected methods are designed to be
1726     * overridden by LinkedHashMap, but not by any other subclass.
1727     * Nearly all other internal methods are also package-protected
1728     * but are declared final, so can be used by LinkedHashMap, view
1729     * classes, and HashSet.
1730     */
1732    // Create a regular (non-tree) node
1733    Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
1734        return new Node<>(hash, key, value, next);
1735    }
```

啥玩意，给LinkedHashMap用的？

如果哈希表，当前桶已经有值了，那么走632逻辑

633判断新来的节点是不是跟已经占桶的节点的哈希值保持一致，并且key一致，（key一致，要么key就是同一个对象，要么key不是null，equal方法的出来的结果一致）所以这时候，新的节点，就是认为老的节点。这边value的hash或者equal不拿来判断的。

636如果当前桶是treenode类型，就把新来的节点添加到treenode，首部。看putTreeVal方法

```java
1958
1959        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
1960                                       int h, K k, V v) {
1961            Class<?> kc = null;
1962            boolean searched = false;
1963            TreeNode<K,V> root = (parent != null) ? root() : this;
1964            for (TreeNode<K,V> p = root;;) {
1965                int dir, ph; K pk;
1966                if ((ph = p.hash) > h)
1967                    dir = -1;
1968                else if (ph < h)
1969                    dir = 1;
1970                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
1971                    return p;
1972                else if ((kc == null &&
1973                          (kc = comparableClassFor(k)) == null) ||
1974                         (dir = compareComparables(kc, k, pk)) == 0) {
1975                    if (!searched) {
1976                        TreeNode<K,V> q, ch;
1977                        searched = true;
1978                        if (((ch = p.left) != null &&
1979                             (q = ch.find(h, k, kc)) != null) ||
1980                            ((ch = p.right) != null &&
1981                             (q = ch.find(h, k, kc)) != null))
1982                            return q;
1983                    }
1984                    dir = tieBreakOrder(k, pk);
1985                }
1986
1987                TreeNode<K,V> xp = p;
1988                if ((p = (dir <= 0) ? p.left : p.right) == null) {
1989                    Node<K,V> xpn = xp.next;
1990                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
1991                    if (dir <= 0)
1992                        xp.left = x;
1993                    else
1994                        xp.right = x;
1995                    xp.next = x;
1996                    x.parent = x.prev = xp;
1997                    if (xpn != null)
1998                        ((TreeNode<K,V>)xpn).prev = x;
1999                    moveRootToFront(tab, balanceInsertion(root, x));
2000                    return null;
2001                }
2002            }
2003        }
```

如果当前节点不是根节点，就去找根节点，不然当前节点就是作为根节点

root方法，

```java
1803
1804        final TreeNode<K,V> root() {
1805            for (TreeNode<K,V> r = this, p;;) {
1806                if ((p = r.parent) == null)
1807                    return r;
1808                r = p;
1809            }
1810        }
```

生名了两个变量r,p。r被赋值为this，p为null值。沿着r，开始找到parent为null的时候，表明找到了根节点。

（真担心这里会死循环，但是并不会~）

返回putTreeVal方法

找到根节点之后，1966行，拿根节点的hash和入桶节点的hash进行比对，如果大于dir=-1，如果小于dir=1

如果等于，则判断key是不是一样（根节点的key是不是同一个对象或者equal方法返回是不是true），如果是就表明，已经在桶的节点就是要入的节点。返回当前节点;

如果key不一样，说明hash一样，但是key不一样，说明hash冲突了。

当hash冲突的时候，可以进入下一个if逻辑体满足其中一个情况：

情况一：kc==null并且comparableClassFor，方法也返回null。

查看comparableClassFor方法，只要key不是Compareable就返回null

```java
345     static Class<?> comparableClassFor(Object x) {
346         if (x instanceof Comparable) {
347             Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
348             if ((c = x.getClass()) == String.class) // bypass checks
349                 return c;
350             if ((ts = c.getGenericInterfaces()) != null) {
351                 for (int i = 0; i < ts.length; ++i) {
352                     if (((t = ts[i]) instanceof ParameterizedType) &&
353                         ((p = (ParameterizedType)t).getRawType() ==
354                          Comparable.class) &&
355                         (as = p.getActualTypeArguments()) != null &&
356                         as.length == 1 && as[0] == c) // type arg is c
357                         return c;
358                 }
359             }
360         }
361         return null;
362     }
```

对这个方法的介绍[参考](http://blog.csdn.net/leeqihe/article/details/76461442)

getGenericInterfaces()

方法返回的是该对象的运行时类型“直接实现”的接口，这意味着： 
返回的一定是接口。必然是该类型自己实现的接口，继承过来的不算。

```java
 static Class<?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {  // 判断是否实现了Comparable接口
            Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
            if ((c = x.getClass()) == String.class) 
                return c;   // 如果是String类型，直接返回String.class
            if ((ts = c.getGenericInterfaces()) != null) {  // 判断是否有直接实现的接口
                for (int i = 0; i < ts.length; ++i) {   // 遍历直接实现的接口
                    if (((t = ts[i]) instanceof ParameterizedType) &&   // 该接口实现了泛型
                        ((p = (ParameterizedType)t).getRawType() == // 获取接口不带参数部分的类型对象
                         Comparable.class) &&   //  该类型是Comparable
                        (as = p.getActualTypeArguments()) != null &&    // 获取泛型参数数组
                        as.length == 1 && as[0] == c)   // 只有一个泛型参数，且该实现类型是该类型本身
                        return c;   // 返回该类型
                }
            }
        }
        return null;
    }
```

返回putTreeVal

```java
367 
368     @SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
369     static int compareComparables(Class<?> kc, Object k, Object x) {
370         return (x == null || x.getClass() != kc ? 0 :
371                 ((Comparable)k).compareTo(x));
372     }
```













