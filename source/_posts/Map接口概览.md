---
title: Map相关
date: 2017-10-25 10:00:58
tags: code
---

Map接口定义了一系列抽象方法

```java
int size();
boolean isEmpty();
boolean containsKey(Object key);
boolean containsValue(Object value);
V get(Object key);
V put(K key, V value);
V remove(Object key);
void putAll(Map<? extends K, ? extends V> m);
void clear();
Set<K> keySet();
Collection<V> values();
Set<Map.Entry<K, V>> entrySet();
int hashCode();
Collection<V> values();
boolean equals(Object o);
```

还有默认实现的抽象方法 

java8新增方法

compute(Object,BitFunction)

```java
1087
1088    default V compute(K key,
1089            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
1090        Objects.requireNonNull(remappingFunction);
1091        V oldValue = get(key);
1092
1093        V newValue = remappingFunction.apply(key, oldValue);
1094        if (newValue == null) {
1095            // delete mapping
1096            if (oldValue != null || containsKey(key)) {
1097                // something to remove
1098                remove(key);
1099                return null;
1100            } else {
1101                // nothing to do. Leave things as they were.
1102                return null;
1103            }
1104        } else {
1105            // add or replace old mapping
1106            put(key, newValue);
1107            return newValue;
1108        }
1109    }
```

入参是，key和BitFunction，BitFunction的实现，由外部定义。

如果newValues为空，即BitFunction的apply实现返回null值，则如果map有这个key就移除，并且也返回null值。

如果newValues不为null，则增加或者刷新map的key键值对。并返回apply的实现值。



再一个java8方法

computeIfAbsent(Object,Function)

```java
950 
951     default V computeIfAbsent(K key,
952             Function<? super K, ? extends V> mappingFunction) {
953         Objects.requireNonNull(mappingFunction);
954         V v;
955         if ((v = get(key)) == null) {
956             V newValue;
957             if ((newValue = mappingFunction.apply(key)) != null) {
958                 put(key, newValue);
959                 return newValue;
960             }
961         }
962 
963         return v;
964     }
```

如果本地map已经存在这个key键值对，直接返回value，如果不存在，看看Function的apply实现是不是为null，如果为null，返回null，不为null，压入map。

疑惑1：为什么compute computeIfAbsent的参数表一个是BitFunction一个是Function？

再一个java8新方法

computeIfPresent(Object,BitFunction)

```java
1011
1012    default V computeIfPresent(K key,
1013            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
1014        Objects.requireNonNull(remappingFunction);
1015        V oldValue;
1016        if ((oldValue = get(key)) != null) {
1017            V newValue = remappingFunction.apply(key, oldValue);
1018            if (newValue != null) {
1019                put(key, newValue);
1020                return newValue;
1021            } else {
1022                remove(key);
1023                return null;
1024            }
1025        } else {
1026            return null;
1027        }
1028    }

```

如果不存在key的键值对，直接返回null，不压入。如果存在，而apply实现不为空，就压入覆盖，如果apply实现为空，则移除已经有的key键值对。



以上三个方法比较：

compute 只要apply不为空，一定更新或者添加keyvalue，如果apply为空，还要移除map已经有的keyvalue

computeIfAbsent 如果map的keyvalue为空并且apply实现不为空，才会加入map，不然map返回原有值或null

​                                如果map不为空，不更新

computeIfPresent   如果map有keyvalue，并且apply非空实现才更新，空实现则移除。如果map没有keyvalue，不操作，返回null





java8新增foreach,默认实现

```java
617 
618     default void forEach(BiConsumer<? super K, ? super V> action) {
619         Objects.requireNonNull(action);
620         for (Map.Entry<K, V> entry : entrySet()) {
621             K k;
622             V v;
623             try {
624                 k = entry.getKey();
625                 v = entry.getValue();
626             } catch(IllegalStateException ise) {
627                 // this usually means the entry is no longer in the map.
628                 throw new ConcurrentModificationException(ise);
629             }
630             action.accept(k, v);
631         }
632     }
```



实际上用lambda可以写成这样

```java
map.forEach((k, v) ->
    System.out.println(k+"=" + v));
```

实际上就是实现了BitConsumer的accept的方法，然后被回调了。



java8新增getOrDefault

```
585 
586     default V More ...getOrDefault(Object key, V defaultValue) {
587         V v;
588         return (((v = get(key)) != null) || containsKey(key))
589             ? v
590             : defaultValue;
591     }
```

java 8新增merge方法

```java
1168
1169    default V merge(K key, V value,
1170            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
1171        Objects.requireNonNull(remappingFunction);
1172        Objects.requireNonNull(value);
1173        V oldValue = get(key);
1174        V newValue = (oldValue == null) ? value :
1175                   remappingFunction.apply(oldValue, value);
1176        if(newValue == null) {
1177            remove(key);
1178        } else {
1179            put(key, newValue);
1180        }
1181        return newValue;
1182    }
1183}
```

java 8新增putIfAbsent方法

```java
740 
741     default V More ...putIfAbsent(K key, V value) {
742         V v = get(key);
743         if (v == null) {
744             v = put(key, value);
745         }
746 
747         return v;
748     }
```

remove

```java
783 
784     default boolean remove(Object key, Object value) {
785         Object curValue = get(key);
786         if (!Objects.equals(curValue, value) ||
787             (curValue == null && !containsKey(key))) {
788             return false;
789         }
790         remove(key);
791         return true;
792     }
```

replace 没什么说的。。。。

```java
883 
884     default V replace(K key, V value) {
885         V curValue;
886         if (((curValue = get(key)) != null) || containsKey(key)) {
887             curValue = put(key, value);
888         }
889         return curValue;
890     }
835 
836     default boolean More ...replace(K key, V oldValue, V newValue) {
837         Object curValue = get(key);
838         if (!Objects.equals(curValue, oldValue) ||
839             (curValue == null && !containsKey(key))) {
840             return false;
841         }
842         put(key, newValue);
843         return true;
844     }
```



内部接口Entry<K,V>

新增java8几个方法

```java
469 
470         public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
471             return (Comparator<Map.Entry<K, V>> & Serializable)
472                 (c1, c2) -> c1.getKey().compareTo(c2.getKey());
473         }
504 
505         public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
506             Objects.requireNonNull(cmp);
507             return (Comparator<Map.Entry<K, V>> & Serializable)
508                 (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
509         }
486 
487         public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
488             return (Comparator<Map.Entry<K, V>> & Serializable)
489                 (c1, c2) -> c1.getValue().compareTo(c2.getValue());
490         }
523 
524         public static <K, V> Comparator<Map.Entry<K, V>>comparingByValue(Comparator<? super V> cmp) {
525             Objects.requireNonNull(cmp);
526             return (Comparator<Map.Entry<K, V>> & Serializable)
527                 (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
528         }
529     }

```

&符号，java8新语法，表示同时满足两个接口，就是返回的东西已经同时实现了Comparator Serializable接口

















