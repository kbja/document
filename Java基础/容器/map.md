# 									Map

## 一、HashMap

### 1、基础知识

* 和HashTable类似，但是不支持并发、支持null值和键。
* 由Node<K,V>数组实现，数组的每一项为一个单向链表的头指针。**在JDK1.8后，会使用红黑树**
* threshold 阈值，当map大小超过threshold时，会进行resize
* loadFactor负载因子，默认为0.75
* 每次扩容都是容量*2，找到最接近当前需求容量(大)的2的幂次

### 2、重点

* hash值为当前对象的hash^(hash>>>16)，目的是让高16位也参与到计算中，减少冲突。之后再对结果result&(length-1)，本质是对result求余

* KeySet基于HashIterator实现，会数组头开始遍历，先遍历数组当前位的链表(树)，再遍历数组下一位。
* 树化后的node继承自LinkedHashMap的Entry，保留着之前链表的数据结构。所以在遍历时，通过Node的指针遍历即可。

[更多内容]: https://segmentfault.com/a/1190000012926722	"详细的HashMap介绍"



## 二、LinkedHashMap

### 1、基础知识

* 继承自HashMap，在HashMap的Node的基础上，增加了前后指针，可以将最近访问的数据通指针连接，形成双向链表，以实现**LRU**



## 三、TreeMap

使用场景：每小时生成一份文件，现需要按照给定时间取出这个时间之前和之后的两份文件



## 四、ConcurrentHashMap

* 并发容器，JDK1.8后后又取消了之前的分段锁设计，结构和HashMap类似，底层采用Unsafe类进行实现
* 设计使用的是乐观锁CAS（compare and swap），传过来三个值，之前拿到的预期值、当前值、希望更改的值。比较当前值和预期值是否相同，如果相同才进行更改。
* 每次上锁锁的是链表的头结点，所以每次扩容后，并发度都会提高。

