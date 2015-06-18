---
layout: post
title:  "Java系列HashTable源码分析"
keywords: "java,HashTable"
description: "Hashtable比较早，是线程安全的哈希映射表。内部采用Entry[]数组，每个Entry均可作为链表的头，用来解决冲突（碰撞）"
category: java 
tags: [java]
---
### HashTable存储结构
HashTable和HashMap采用相同的存储机制，二者的实现基本一致，不同的是：

* HashMap是非线程安全的，HashTable是线程安全的，内部的方法基本都是synchronized。
* HashTable不允许有null值的存在。
* 在HashTable中调用put方法时，如果key为null，直接抛出NullPointerException。其它细微的差别还有，比如初始化Entry数组的大小等等，但基本思想和HashMap一样。

### HashTable特点
* 线程安全。
* Key、Value均不能为null。
* 包含了一个Entry[]数组，而Entry又是一个链表，用来处理冲突。
* 每个Key对应了Entry数组中固定的位置（记为index），称为槽位（Slot）。槽位计算公式为： (key.hashCode() & 0x7FFFFFFF) % Entry[].length() 。
* 当Entry[]的实际元素数量（Count）超过了分配容量（Capacity）的75%时，新建一个Entry[]是原先的2倍，并重新Hash（rehash）。
* rehash的核心思路是，将旧Entry[]数组的元素重新计算槽位，散列到新Entry[]中。
#### HashTable源码简要分析
Entry类

```
class Entry<K,V> // Entry<K,V>是槽中的元素，可做链表，解决散列冲突。
{
     int hash; // 即key.hashCode()
     K key;
     V value;
     Entry<K,V> next; // 用来实现链表结构。同一链表中的key的hash是相同的。
     protected Entry(int hash, K key, V value, Entry<K,V> next) {
          this.hash=hash;this.key=key;this.value=value;this.next=next;
     }
}
```
HashTable类

```
public class Hashtable<K,V>
{
     Entry[] table; // 槽数组，也称桶数组。
     int count; // table中实际存放的Entry数量。
     int threshold; // 当table数量超过该阈值后，进行reash。（该值为 capacity * loadFactor）
     float loadFactor; // 加载因子，默认是0.75f。
 
     public Hashtable(int initialCapacity/*默认是11*/, float loadFactor) {
          if(initialCapacity==0) initialCapacity=1;
          this.locadFactor = locadFactor;
          table = new Entry[initialCapacity];
          threshold = (int)(initialCapacity * locadFactor);
     }
 
     // put()： 若key存在，返回旧value；若key不存在，返回null。
     public synchronized V put(K key,V value) {
          // 检查key是否已经存在，若存在则覆盖已经存在value，并返回被覆盖的value。
          Entry tab[] = table;
          int hash = key.hashCode();
          int index = (hash & 0x7FFFFFFF) % tab.length; // 存储槽位索引。
          for(Entry<K,V> e = tab[index]; e!=null; e=e.next ) { // 在冲突链表中寻找
               if( (e.hash == hash ) && e.key.equals(key) ) {
                         V old = e.value;
                         e.value = value; // 新value覆盖旧value
                         return old;
                    }
          }
 
          // 是否需要rehash
          if(count >= threshold){
               rehash();
               tab = table; // rehash完毕后，修正tab指针指向新的Entry[]
               index =  (hash & 0x7FFFFFFF) % tab.length; // 重新计算Slot的index
          }
 
          // 存储到槽位，如果有冲突，新来的元素被放到了链表前面。
          Entry<K,V> e = tab[index]; // 旧有Entry
          tab[index] = new Entry<>(hash,key,value,e/* 旧有Entry成为了新增Entry的next */);
          count ++;
          return null;
     }
 
     // rehash(): 再次hash。当Entry[]的实际存储数量占分配容量的约75%时，扩容并且重新计算各个对象的槽位
     static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 ;
     protected void rehash() {
          int oldCapacity = table.length;
          Entry[] oldMap = table;
          int newCapacity = (oldCapacity << 1) + 1; // 2倍+1
          Entry[] newMap = new Entry[newCapacity];
          threshold = (int)(newCapacity * loadFactor);
          table = newMap;
 
          for( int i=oldCapacity; i-- >0;){ //  i的取值范围为 [oldCapacity-1,0]
               for (Entry<K,V> old = oldMap[i]; old!=null;){ // 遍历旧Entry[]
                    Entry<K,V> e = old;
                    int index = (e.hash & 0x7FFFFFFF) % newCapacity;     // 重新计算各个元素在新Entry[]中的槽位index。
                    e.next = newMap[index]; // 已经存在槽位中的Entry放到当前元素的next中
                    newMap[index]=e;     // 放到槽位中
                    old = old.next;
               }
          }
 
     }
}
```
### 参考资料
[JDK文档](http://docs.oracle.com/javase/7/docs/api/java/util/Hashtable.html)