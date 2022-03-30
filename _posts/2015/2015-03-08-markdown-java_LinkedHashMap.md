---
title: "Java LinkedHaskMap"
layout: post
date: 2015-03-08 09:55
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- history note
- source code
category: blog
author: WuKongCoder
description: Java linkedHaskMap 
---


```java
/**
* 可以设置accessOrder的构造方法
*/
public LinkedHashMap(int initialCapacity, float loadFactor ,boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

```java
/**
*它是在HashMap构造方法中最后被调用的，在HashMap中无意义，专门给继承类*覆盖的。这里初始化了LinkedHashMap的header。
*
*/
void init() {
    header = new Entry<>(-1, null, null, null);
    header.before = header.after = header;
}
```

```java
/**
*是在resize方法中调用的，LinkedHashMap重写了这个方法，原HashMaph中的transfer遍历table中的每个元素Entry，然后对每个元素再进行遍历得到每一个Entry，重新hash到newTable中。
*而LinkedHashMap因为维护了一个header双向链表，只需循环header,提高了迭代效率。
*/
void transfer(HashMap.Entry[] newTable) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e = header.after; e != header; e = e.after) {
            int index = indexFor(e.hash, newCapacity);
            e.next = newTable[index];
            newTable[index] = e;
        }
    }
```

```java
/**
*为了header的双向链表与底层HashMap中的table共同维护，LinkedHashMap覆盖了void addEntry(int hash, K key, V value, int bucketIndex) 和void  createEntry(int hash, K key, V value, int bucketIndex)，同时覆盖了类Entry中的void recordAccess(HashMap m)实现header的排序
*
*
*/
void addEntry(int hash, K key, V value, int bucketIndex) {
    createEntry(hash, key, value, bucketIndex);
    // Remove eldest entry if instructed, else grow capacity if appropriate
    Entry<K,V> eldest = header.after;
    if (removeEldestEntry(eldest)) {
        removeEntryForKey(eldest.key);
    } else {
        if (size >= threshold)
            resize(2 * table.length);
    }
}
/**
*方法默认返回false,提供该方法是为了可以修改为判断是否需要移除最后那个entry,实现比如缓存中将最老的元素移除。
*/
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }

//同时维护了header
void createEntry(int hash, K key, V value, int bucketIndex) {
    HashMap.Entry<K,V> old = table[bucketIndex];
    Entry<K,V> e = new Entry<>(hash, key, value, old);
    table[bucketIndex] = e;
    e.addBefore(header);
    size++;
}
//插入到header前
private void addBefore(Entry<K,V> existingEntry) {
            after  = existingEntry;
            before = existingEntry.before;
            before.after = this;
            after.before = this;
        }
//当调用了get或set方法时，此方法被调用用来维护header中元素的顺序。
void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            if (lm.accessOrder) {
                lm.modCount++;
                remove();
                addBefore(lm.header);
            }
        }
```
