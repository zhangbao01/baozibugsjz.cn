---
title: HashMap源码put （链存储）
---
## 概述
	本篇主要讲述jdk1.8 hashmap链式存储的源码流程，和作者认为比较亮眼的地方
### put
```bash
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
### hash
```bash
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
这个是对key做hashcode处理，将hashcode右移16位原hashcode做一次异或(^)运算
由于计算key所在数组位置，是通过key的hashcode和数组长度减一做的与(&)运算
`tab[i = (n - 1) & hash]`所以高位在大多数情况下都是用不到的，右移16位是为了让高位参与到数组位置的计算。当然如果这个key算出来小于16位且大于数组长度，还是会有部分用不到的。
异或(^)，本身计算，相同为0，不同为1，1^0有四种情况（11，10，01，00），但是结果为1或者0的概率是相等的，并不会出现偏向1或偏向0的情况。

### putVal 
```bash
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
这个方法判断了有没有链表，是不是红黑树，即后续处理
主要说一下`tab[i = (n - 1) & hash]`
首先与(&)操作，`a&b`，当ab同时为1时，结果才为1，其他情况下为0，在1和0四种情况中，很明显是偏向于0的，但是hashmap初始长度是`16`二进制是`10000`，`n-1`的结果就是`15``1111`，`a&1111`结果只是取了a的后四位，并没有出现概率偏差的情况。所以hashmap的扩容是以左移一个单位，为了保证n-1的结果位数全部是1的情况

### resize
```bash
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
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
这个方法是扩容和初始化。
主要看一下`newTab[e.hash & (newCap - 1)]`和`(e.hash & oldCap) == 0`
第一个上面说了，只不过是重新计算了下key在数组的位置，
先看第二个`(e.hash & oldCap) == 0`，oldCap是旧数组的长度16(10000)，a&10000只有地五位是0的情况下结果才是0，原来取了后四位判断key所在的位置，现在扩容往左多移了一位自然要把原来这个链表拆成第五位是1和0的两种情况，并重新计算位置。
如果第五位是0和新的数组长度31(11111)做&运算依然是0，位置不变，如果是1，假设原来在1111这个位置下角标15，而现在是31(11111)，其中是多了一个扩容的长度16

这里说一个我认为比较矛盾的地方，当他判断只有一个节点没有链表的时候重新计算了地址，但是后面是通过判断扩容的位置是0和1的情况再进行地址的调整，如果说后者比前者效率更高的话可以把前面也改成先判断再调整位置，直接计算效率效率高的话也可以dowhile遍历同时计算位置然后赋值。知道答案的小伙伴可以给我解惑下
