---
layout: post
title:  "Java系列ConcurrentHashMap源码分析"
keywords: "java,ConcurrentHashMap"
description: "首先了解下concurrentHashMap的原理：该类是通过存放一个Segments(段数组),然后在每个Segments[i]中放入一个类似hashMap的结构"
category: java 
tags: [java]
---

###ConcurrentHashMap原理分析
 集合是编程中最常用的数据结构。而谈到并发，几乎总是离不开集合这类高级数据结构的支持。比如两个线程需要同时访问一个中间临界区（Queue），比如常会用缓存作为外部文件的副本（HashMap）。这篇文章主要分析jdk1.5的3种并发集合类型（concurrent，copyonright，queue）中的ConcurrentHashMap，让我们从原理上细致的了解它们，能够让我们在深度项目开发中获益非浅。
 
我们使用得最多的数据结构之一就是HashMap和Hashtable。大家都知道，HashMap中未进行同步考虑，而Hashtable则使用了synchronized，带来的直接影响就是可选择，我们可以在单线程时使用HashMap提高效率，而多线程时用Hashtable来保证安全。
    
当我们享受着jdk带来的便利时同样承受它带来的不幸恶果。通过分析Hashtable就知道，synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，安全的背后是巨大的浪费，慧眼独具的Doug Lee立马拿出了解决方案----ConcurrentHashMap。
    
ConcurrentHashMap和Hashtable主要区别就是围绕着锁的粒度以及如何锁。如图

![图1](/static/images/currenthashmap01.jpg)
   左边便是Hashtable的实现方式---锁整个hash表；而右边则是ConcurrentHashMap的实现方式---锁桶（或段）。ConcurrentHashMap将hash表分为16个桶（默认值），诸如get,put,remove等常用操作只锁当前需要用到的桶。试想，原来只能一个线程进入，现在却能同时16个写线程进入（写线程才需要锁定，而读线程几乎不受限制，之后会提到），并发性的提升是显而易见的。
   
![图2](/static/images/currenthashmap02.jpg)
更令人惊讶的是ConcurrentHashMap的读取并发，因为在读取的大多数时候都没有用到锁定，所以读取操作几乎是完全的并发操作，而写操作锁定的粒度又非常细，比起之前又更加快速（这一点在桶更多时表现得更明显些）。
ConcurrentHashMap中主要实体类就是三个：ConcurrentHashMap（整个Hash表）,Segment（桶），HashEntry（节点），对应上面的图可以看出之间的关系。

#### ConcurrentHashMap：该类有4个构造方法分别，也是中间相互调用，看下最主要的构造函数：

```
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();

        if (concurrencyLevel > MAX_SEGMENTS)
            concurrencyLevel = MAX_SEGMENTS;
   
        //第一个参数是段里面的数组长度，第二个参数是效率因子（前面有讲到）,第三个参数是段的长度 上面可以不用看。这里和hashMap的原理一样获取2的倍数来进行数组长度
        int sshift = 0;
        int ssize = 1;
        while (ssize < concurrencyLevel) {
            ++sshift;
            ssize <<= 1;//向左移位
      }
        segmentShift = 32 - sshift;
        segmentMask = ssize - 1;
        this.segments = Segment.newArray(ssize);//在这里创建了一个长度为ssize的数组，可见如果使用默认new concurrentHashMap的话段的长度将会永远固定，默认16个
        
        //，也就是说最多只有16个线程进行并发，所以这个参数对并发是非常关键的
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        int c = initialCapacity / ssize;
        if (c * ssize < initialCapacity)
            ++c;
        int cap = 1;//该参数是对段里面的数组进行长度设置
        while (cap < c)
            cap <<= 1;

        for (int i = 0; i < this.segments.length; ++i)
            this.segments[i] = new Segment<K,V>(cap, loadFactor);//创建每个段
    }
```
和前面一样我们还是看最主要的方法：
    put方法：该方法是在setment(段)里面放入键值（根据hashcode确定segment段）
    
```
public V put(K key, V value) {
    if (value == null)
          throw new NullPointerException();
          int hash = hash(key.hashCode());
          return segmentFor(hash).put(key, hash, value, false);
  }
```
调用下面segment段中的put方法

```
      V put(K key, int hash, V value, boolean onlyIfAbsent) {//主要的区别在于对修改段的时候进行的同步的处理
            lock();//因为segment是继承ReentrantLock类，该类可以实现同步锁功能和synxxx什么的有一点点区别（这里我不多讲，）
            try {
                int c = count;
                if (c++ > threshold) // 这个是却断段里面是否需要对数组进行扩容
                    rehash();
                HashEntry<K,V>[] tab = table;
                int index = hash & (tab.length - 1);
                HashEntry<K,V> first = tab[index];//缺点段里面数组的位置
                HashEntry<K,V> e = first;
                while (e != null && (e.hash != hash || !key.equals(e.key)))
                    e = e.next;//获取key-value的HashEntry

                V oldValue;
                if (e != null) {
                    oldValue = e.value;
                    if (!onlyIfAbsent)
                        e.value = value;//值替换
                }
                else {
                    oldValue = null;
                    ++modCount;
                    tab[index] = new HashEntry<K,V>(key, hash, first, value);//值添加和hashMap一样
                    count = c;
                }
                return oldValue;
            } finally {
                unlock();//解锁
            }
        }
```
remove方法和put一样，看一下get方法：

```
V get(Object key, int hash) {
            if (count != 0) { // read-volatile
                HashEntry<K,V> e = getFirst(hash);
                while (e != null) {
                    if (e.hash == hash && key.equals(e.key)) {
                        V v = e.value;
                        if (v != null)
                            return v;
                        return readValueUnderLock(e); //这个方法是当获得的value值为null的时候需要对其进行锁，这个好像是编译的时候初始化map会有问题。
                    }
                    e = e.next;
                }
            }
            return null;
        }
```
  该方法和hashmap基本没什么区别，所以对于concurrenthashmap而言它的读是异步了但是数据却保持了一致性这个就是volatile的好处，没有同步这样大大提高了效率
  
下面还要介绍一下size方法，这个方法我觉得使用的很巧妙

```
    public int size() {
        final Segment<K,V>[] segments = this.segments;
        long sum = 0;//计算concurrenthashmap中的个数
        long check = 0;
        int[] mc = new int[segments.length];//统计所有段的修改次数
        for (int k = 0; k < RETRIES_BEFORE_LOCK; ++k) {//进行2次非锁统计，如果还不行就把每个concurrenthashmap给锁了进行统计
            check = 0;
            sum = 0;
            int mcsum = 0;
            for (int i = 0; i < segments.length; ++i) {
                sum += segments[i].count;
                mcsum += mc[i] = segments[i].modCount;//判定在读取过程中有没有其他线程对它进行修改段内容操作
            }
            if (mcsum != 0) {
                for (int i = 0; i < segments.length; ++i) {
                    check += segments[i].count;
                    if (mc[i] != segments[i].modCount) {//这里很巧妙，如果修改段操作刚好是一增一减，那还是保持不变所以不用重新统计。
                        check = -1; // force retry
                        break;
                    }
                }
            }
            if (check == sum)//如果验证结果和统计结果一致就输出总和
                break;
        }
        if (check != sum) { // 如果2次统计还是不一致，则将每个段都锁了，再统计再进行解锁
            sum = 0;
            for (int i = 0; i < segments.length; ++i)
                segments[i].lock();
            for (int i = 0; i < segments.length; ++i)
                sum += segments[i].count;
            for (int i = 0; i < segments.length; ++i)
                segments[i].unlock();
        }
        if (sum > Integer.MAX_VALUE)
            return Integer.MAX_VALUE;
        else
            return (int)sum;
    }
```

#### segment(桶)
该类其实你可以看成是一个hashMap，它和hashMap的结构非常相似，里面也是一个数组。唯一的却别在于volatile int count，就是在统计每个segment(段)的存储数量的时候进行了同步。构造函数这些和hashMap基本一致
#### HashEntry
该类和HashMap中的Entry类似，唯一的却别在于构造函数，具体功能参考前章的hashmap和hashtable

```
        final K key;
        final int hash;
        volatile V value;//该volatile的作用是保持主内存和线程中的值同步，每个线程获取这个值的时候都会拷贝这份，这样就可以解决读的同步问题
        final HashEntry<K,V> next;

        HashEntry(K key, int hash, HashEntry<K,V> next, V value) {
            this.key = key;
            this.hash = hash;
            this.next = next;
            this.value = value;
        }

```
    
    