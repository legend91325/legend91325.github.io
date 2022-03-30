---
title: "Java HaskMap"
layout: post
date: 2015-03-05 17:25
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Java
- history note
- source code
category: blog
author: WuKongCoder
description: Java HaskMap 
---

---
HashMap使用的数据结构，专业术语“链表散列”，代码实
```java
    /**
     * 定义了一个Entry的数组用来存储数据。
     */
     transient Entry[] table;
```
    
    
```java
    /**
     * Entry是内部定义的类
     */
     static class Entry<K,V> implements Map.Entry<K,V> {
     
     /**
      * 定义了两个常量key，就是HashMap的key
      * value，key对应的value
      * next，下一个Entry对象，故Entry的数据结构是一个单项的链表
      * 常量hash值，可以说他是bucket，存储的其实是table数组的下标值。
      */
        final K key;
        V value;
        Entry<K,V> next;
        final int hash;

        /**
         * 构造方法.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }
        /**
        设置value的同时会返回旧值
        */
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return (key==null   ? 0 : key.hashCode()) ^
                   (value==null ? 0 : value.hashCode());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        /**
        *    会在调用HashMap put方法时调用它，但是它本身是空方法，
        *目前了解的是LinkedHashMap会重写这个方法，
        *每次put时会将对象放到双向链表的尾，
        *因为LinkeHashMap要实现LRU算法，我会专门写LinkedHashMap,
        *要详解可以移步那里看看。
        */
        void recordAccess(HashMap<K,V> m) {
        }

        /**
         * This method is invoked whenever the entry is
         * removed from the table.
         */
        void recordRemoval(HashMap<K,V> m) {
        }
    }
```
- **transient** 实现了Serializable的类中，被标识了transient 的属性序列化时会被忽略。
```java
if (loadFactor <= 0 || Float.isNaN(loadFactor))
    throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
```
返回true意味着传入的loadFactor 为`NaN = 0.0f / 0.0f;`
```java
/**
hashMap存储数据的地方，长度永远是2的次幂（因为定位bucket时有对长度进行‘与’操作，其他作用后续说），默认初始16
*/
transient Entry[] table;
```
```java
/**
table表里允许存储的最大数据，其实可以计算出来（table.size()*loadFactor+1）但程序中多处使用，使用常量表示更方便。
*/
final float loadFactor;
```
```java
/**
负载因子，意味着数据可以存储相对于总容量的百分比，默认0.75， 当数据超过了loadFactor（table.size()*loadFactor+1），会调用resize()，增大table容量，重新hash定位所有数据，耗费性能，所以对于数据集可以提前可以预知数据大小的，可以初始化时设置此值还有初始容量initialCapacity
*/
final float loadFactor;
```
```java
/**
当hashMap结构发生改变（移除数据或者添加数据），会修改这个值，
modCount++，主要的作用在于hashMap是非线程安全的，在调用迭代器每次取数据时会判断modCount在这段期间内是否被修改，如果修改了说明数据发 * 生变化，会抛出ConcurrentModificationException，一种安全机制吧。
*/
    transient int modCount;
```

```java
    /**
    hashMap本身是散列表结构，它定位元素的逻辑是先取key的hashcode,再通过此hash函数
    再次hash,之后调用indexFor（）根据table表的长度取余，找到存储的偏移量，之后再比较value的值。
    此次hash的目的是因为key的hashcode无法保证高质量的将元素散列开来。
    NULL的key,将定位到0
    */
    static int hash(int h) {
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

```java
/**
get方法，其实没啥好说的，时间复杂度因为散列数据结构故O(1)
*/
  public V get(Object key) {
        if (key == null)
            return getForNullKey();
        int hash = hash(key.hashCode());
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
    }
```
```java
/**
这个也挺简单的，主要说下recordAccess(this),Entry中的方法，在hashMap中，是没有操作的，
放这个方法还有recordRemoval，是留给其他类使用的如LinkedHashMap ：用来实现LRU置换算法。
感兴趣的可以看下LinkedHashMap
addEntry(hash, key, value, i)是将value添加到key的链表上。
*/
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```
```java
/**
新建一个Entry，头插法。
主要在resize()方法
*/
 void addEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
    }
```
```java
/**
当hashMap中存储的size不在小于threshold（capacity * load factor）时，需要
重新调整table的大小，每次调整完都会将元素重新排序，当数据量大了这会是一个耗时操作，
并且我们使用的时候应该尽量让我们初始的hashMap容量接近要使用的容量/loadFactor，减少置换次数。
*/
 void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable);
        table = newTable;
        threshold = (int)(newCapacity * loadFactor);
    }

    /**
     * 将旧表中的数据重新hash，放入新表中。
     */
    void transfer(Entry[] newTable) {
        Entry[] src = table;
        int newCapacity = newTable.length;
        for (int j = 0; j < src.length; j++) {
            Entry<K,V> e = src[j];
            if (e != null) {
                src[j] = null;
                do {
                    Entry<K,V> next = e.next;
                    int i = indexFor(e.hash, newCapacity);
                    e.next = newTable[i];
                    newTable[i] = e;
                    e = next;
                } while (e != null);
            }
        }
    }
```

```java

 /**
     remove操作
     */
    public V remove(Object key) {
        Entry<K,V> e = removeEntryForKey(key);
        return (e == null ? null : e.value);
    }

    final Entry<K,V> removeEntryForKey(Object key) {
        int hash = (key == null) ? 0 : hash(key.hashCode());
        int i = indexFor(hash, table.length);
        Entry<K,V> prev = table[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                modCount++;
                size--;
                if (prev == e)
                    table[i] = next;
                else
                    prev.next = next;
                e.recordRemoval(this);
                return e;
            }
            prev = e;
            e = next;
        }

        return e;
    }
```

```java
/**
hashMap中是否含有相应的value，还有一个containsKey方法没说，
想提下这个是因为，判断包含value需要循环比对，时间复杂度O(n)。慎重使用。
*/
public boolean containsValue(Object value) {
        if (value == null)
            return containsNullValue();

        Entry[] tab = table;
        for (int i = 0; i < tab.length ; i++)
            for (Entry e = tab[i] ; e != null ; e = e.next)
                if (value.equals(e.value))
                    return true;
        return false;
    }
```

```java
/**
最后的是hashMap的迭代了，一起来说，HashIterator内部私有类，几个属性里说下expectedModCount
上面的hashMap添加删除操作，都会更新一个modCount，开头已经说了，是hashMap是非线程安全的，这个属性
是在迭代时用来判断迭代过程中，hashMap的值是否被更新过，expectedModCount就是在Iterateor构造时赋值
迭代过程中判断。
所以，当我们使用迭代器遍历hashmap时，如果想同时修改hashMap内容（只能删除）。使用Iterator提供的remove方法
*/
private abstract class HashIterator<E> implements Iterator<E> {
        Entry<K,V> next;        // next entry to return
        int expectedModCount;   // For fast-fail
        int index;              // current slot
        Entry<K,V> current;     // current entry

        HashIterator() {
            expectedModCount = modCount;
            if (size > 0) { // advance to first entry
                Entry[] t = table;
                //定位到第一个value中不为null的位置。
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
        }

        public final boolean hasNext() {
            return next != null;
        }

        final Entry<K,V> nextEntry() {
            //迭代过程中，元素被修改，抛出ConcurrentModificationException
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            Entry<K,V> e = next;
            if (e == null)
                throw new NoSuchElementException();

            if ((next = e.next) == null) {
                Entry[] t = table;
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
            current = e;
            return e;
        }

        public void remove() {
            if (current == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            Object k = current.key;
            current = null;
            HashMap.this.removeEntryForKey(k);
            expectedModCount = modCount;
        }

    }
/**
三个私有的迭代器，也就是说我们可以用这三种方式迭代HashMap。
*/
    private final class ValueIterator extends HashIterator<V> {
        public V next() {
            return nextEntry().value;
        }
    }

    private final class KeyIterator extends HashIterator<K> {
        public K next() {
            return nextEntry().getKey();
        }
    }

    private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {
        public Map.Entry<K,V> next() {
            return nextEntry();
        }
    }

    // Subclass overrides these to alter behavior of views' iterator() method
    Iterator<K> newKeyIterator()   {
        return new KeyIterator();
    }
    Iterator<V> newValueIterator()   {
        return new ValueIterator();
    }
    Iterator<Map.Entry<K,V>> newEntryIterator()   {
        return new EntryIterator();
    }


    /**
    好吧，三面的三种迭代初始化，只能提供包内部使用，真正暴露出来的方法是下面三个，但是换汤不换药，
    各位判官自行看看就好。可以根据自己的需求调用相应的Iterator
    这三种的初始赋值使用的分别是keySet，values和entrySet。
    */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        return (ks != null ? ks : (keySet = new KeySet()));
    }

    private final class KeySet extends AbstractSet<K> {
        public Iterator<K> iterator() {
            return newKeyIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsKey(o);
        }
        public boolean remove(Object o) {
            return HashMap.this.removeEntryForKey(o) != null;
        }
        public void clear() {
            HashMap.this.clear();
        }
    }

  
    public Collection<V> values() {
        Collection<V> vs = values;
        return (vs != null ? vs : (values = new Values()));
    }

    private final class Values extends AbstractCollection<V> {
        public Iterator<V> iterator() {
            return newValueIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsValue(o);
        }
        public void clear() {
            HashMap.this.clear();
        }
    }
    
    private transient Set<Map.Entry<K,V>> entrySet = null;
   
     /**
     Note:这里，这里，说实话我也没搞懂，为什么要这种写法entrySet()再调entrySet0()。
     网上找了找，也没啥令人信服的说法。如果哪位了解，请务必要告诉我一下。
     */
    public Set<Map.Entry<K,V>> entrySet() {
        return entrySet0();
    }

    private Set<Map.Entry<K,V>> entrySet0() {
        Set<Map.Entry<K,V>> es = entrySet;
        return es != null ? es : (entrySet = new EntrySet());
    }

    private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return newEntryIterator();
        }
        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<K,V> e = (Map.Entry<K,V>) o;
            Entry<K,V> candidate = getEntry(e.getKey());
            return candidate != null && candidate.equals(e);
        }
        public boolean remove(Object o) {
            return removeMapping(o) != null;
        }
        public int size() {
            return size;
        }
        public void clear() {
            HashMap.this.clear();
        }
    }
```
