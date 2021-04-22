---
title: HashMap和HashSet源码解析
date: 2021-04-17 11:57:41
tags: ["Java基础", "集合"]
---

### HashMap

HashMap可以说是平时使用最多的一个集合类型了，面试中也经常会被问到。下面就从底层源码来看看它的具体实现。

<!-- more -->

先看下HashMap中有哪些默认的参数

```java
//默认的初始容量大小 初始为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

//容器最大的大小
static final int MAXIMUM_CAPACITY = 1 << 30;

//默认的负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//链表转为红黑树的临界值
static final int TREEIFY_THRESHOLD = 8;

//红黑树退化成链表的临界值
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```



```java
/**
 * 数组+链表形式存储的数据
 */
transient Node<K,V>[] table;

/**
 * hashmap的节点集合，可用于迭代等操作
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * map的大小
 */
transient int size;

/**
 * The number of times this HashMap has been structurally modified
 * Structural modifications are those that change the number of mappings in
 * the HashMap or otherwise modify its internal structure (e.g.,
 * rehash).  This field is used to make iterators on Collection-views of
 * the HashMap fail-fast.  (See ConcurrentModificationException).
 */
transient int modCount;

/**
 * 发生扩容时的临界值  当前容量*负载因子
 */
int threshold;

/**
 * 负载因子
 */
final float loadFactor;
```

**DEFAULT_INITIAL_CAPACITY** 是创建HashMap的一个初始容量大小，默认是16，使用无参的构造函数初始化出来的HashMap的大小就是16。  而**loadFactor**就是负载因子，默认为0.75f。它是参与计算扩容大小的参数，当未指定容器容量时，扩容大小为当前容器大小*负载因子，当指定了容器大小时，扩容大小则是tableSizeFor()方法计算出来的值，当新增元素时，当前容器大小大于扩展容量了，HashMap就会进行扩容。



先从构造函数开始看起，HashMap提供了三种构造函数

- HashMap()
- HashMap(int initialCapacity)
- HashMap(int initialCapacity, float loadFactor)

initialCapacity为初始化容器的大小，loadFactor为负载因子

这里有一个需要注意的地方就是，不论你输入的容器大小是什么，ta都会帮你算成大于该值并且最接近的2的幂次方：

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

至于为什么这么做，就得从怎么根据key在数组中找到存放的位置开始说起了。

可以在源码中看到获取在tab中数组下标的函数为 **(n - 1) & hash**

这个函数是什么意思呢，这个函数其实等价于 **hash % n** , 也就是通过对hash值取余的方式，获取到数组中存放该节点的下标， 这也是为什么tableSizeFor方法会将传入的数字始终计算成2的整数倍的原因，在一个因为计算机对于&操作更好计算，于是就采用了这种方式初始化HashMap的初始容量，使计算key散列值的时候更加快速。



说到hashcode，来看一下HashMap中的hash()方法， 下面的这段代码是JDK1.8中的

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

可以看到这段代码 ** (h = key.hashCode()) ^ (h >>> 16)**， 这段代码其实是一个扰动函数，为了减少hashcode的碰撞率，jvm中算出来的hashcode是32位的，也就是说hashcode将自己的低16位和高16位进行了一个异或操作，这样能够大大减少hash碰撞的几率，在JDK1.7中还进行了4次扰动，在1.8中就减少到了1次，因为根据统计，一次扰动已经足以减少碰撞的几率。



基本的概念了解了，就来看一下常用的方法的源码吧~

#### put

```java
public V put(K key, V value) {
  	//计算出key的hash值，并当做参数传入
    return putVal(hash(key), key, value, false, true);
}


final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
  
    Node<K,V>[] tab; Node<K,V> p; int n, i;
  	//如果table为空，也就是数组还没有初始化过，就先对table进行初始化的操作
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
  	//计算散列到table中的位置，如果当前bin为空，则在当前bin新建一个Node节点作为头结点
  	//感觉这里不是线程安全的，并发的情况下有可能同时设置头结点
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
      	//如果当前bin中不为空
        Node<K,V> e; K k;
        //则判断头结点的key是否与当前key相等，如果相等则将头结点赋值给e
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
          	//如果该节点是一个红黑树的节点，则调用对应方法放置节点，并将对应节点返回，赋值给e
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
          	//如果还是一个链表，则遍历链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                  	//如果链表中当前节点的下一个节点为空，因为此时还没匹配到相同的key，所以新建一个Node
                    //此处也有可能会产生并发问题
                  	p.next = newNode(hash, key, value, null);
                  	//如果大于转成红黑树的临界值，则将链表转为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
              	//如果找到了相同的节点，则直接返回，此时e等于这个相同的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
      	//如果Node不为空，则需要对老的值进行替换
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
          	//普通情况或者放置方式为不存在才放入且旧值不存在时， 替换为新值
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
          	//LinkedHashMap实现的一个回调函数
            afterNodeAccess(e);
          	//返回旧值
            return oldValue;
        }
    }
  	//操作数自增1（这个值蛮重要的）
    ++modCount;
  	//size自增1，如果大于扩容的临界值，则对数组进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```



#### get

get方法就比较简单了，也来看一下

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}


final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
  	//如果table不为空并且对key散列后获得的bin中的节点也不为空时
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
      	//检查头结点是否是想要获取的节点，如果是的话直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
      	//否则遍历链表
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
              	//如果是红黑树，则在树中查找对应的节点
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
              	//是链表的话则遍历获取
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```



#### remove

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
  	//如果table非空且得到的bin非空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
      	//判断头结点是否相等，如果是，将头结点赋值给node
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
              	//在红黑树中查找对应的节点，找到后赋值给node
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
              	//否则在链表中遍历查找，找到后赋值给node
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
      	//如果找到了节点并且不需要匹配value，或者value相同的情况下
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
              	//在红黑树中移除该节点
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
              	//如果是头节点，将头结点指向当前节点的下一个节点
                tab[index] = node.next;
            else
              	//如果不是头节点则将指向当前节点的指针指向下一个节点
                p.next = node.next;
          	//操作数自增
            ++modCount;
          	//数组大小自减1
            --size;
          	//TODO LinkedHashMap的回调
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```



#### resize 

resize方法用于初始化table数组，或者是在数组需要扩容的时候调用, JDK1.7中在并发的情况下扩容链表有可能会形成环，而在JDK1.8中有可能丢失数据

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
  	//老table的容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    //老的数组扩容大小
  	int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
      	//如果已经是最大大小了，则直接返回老table
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
      	//如果能够扩容，则将数组大小扩展为原来的两倍， 扩容大小也相应的扩展两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
      	//数组为空的情况，扩容大小大于0，将扩容的大小的值赋值给新数组的长度
      	//带参数的构造函数会进入这个分支，扩容大小为2
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
      //此处为调用HashMap无参的构造函数，未指定数组大小的情况，使用默认值给数组大小和扩容大小赋值
      newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
  	//如果通过上面的条件计算后，新的数组扩容大小仍为0，则计算出新的扩容大小的值（首次初始化的情况）
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
  
    @SuppressWarnings({"rawtypes","unchecked"})
  	//这里生成了一个新的数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
      	//遍历老table
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
              	//老table相应bin中赋值null
                oldTab[j] = null;
                if (e.next == null)
                  	//如果只有一个节点，则将节点重新散列到新的table中
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                  	//如果是一个红黑树，重新构造这个红黑树？
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                  	//如果是一个链表 
                  	//下面的代码有些复杂，我们在下文中仔细分析一下
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
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```





```Java
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
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```



这段代码可以分为三个部分来看：



首先来看第一部分，可以看到其实是定义了两个双向链表。因为数组扩容时扩大到原来的两倍，所以这里定义了一个高位的链表和一个低位的链表，为什么要分成两个链表呢，看后面的代码就知道了。

```java
Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
```

下面的函数是为了将节点插入到新的链表中去，其中函数 **(e.hash & oldCap) == 0** 决定了该节点是放到低位的链表还是高位的链表。大家都知道，HashMap扩容时数组大小会变成原来的两倍，而key在进行散列的时候，是使用的hash&(cap - 1)， 比如原来的容量为16， 那么散列的值得范围就是hash的低4位，也就是0000~1111, 而将HashMap扩容2倍以后就会是去取hash的低五位，那么同一个hash的计算结果只有可能是0xxxx，或者是1xxxx, 只有第5位会有0，或者1两种选择，而等于0的话就相当于在原位置不动，等于1的话就相当于移动到扩容出来的链表，而高位和低位链表中间相差的距离恰好是一个老容量的大小（oldCap），着样大大减少了重新散列的成本，不得不说这里设计的也太精妙了。

```java
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
```



这里将对应的链表放到对应得数组中去

```java
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```





### HashSet

hashset完全就是在HashMap的基础上封装而成的，看下构造函数就知道了

```java
public HashSet() {
    map = new HashMap<>();
}

public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```



一些常用的方法也是直接调用的hashmap的方法，由于不需要存储value，所以就定义了一个空对象

```java
private static final Object PRESENT = new Object();

public Iterator<E> iterator() {
    return map.keySet().iterator();
}

public int size() {
    return map.size();
}

public boolean isEmpty() {
    return map.isEmpty();
}

public boolean contains(Object o) {
    return map.containsKey(o);
}

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}

public void clear() {
    map.clear();
}
```