# 										ConcurrentHashMap

## 一、基本结构

通过Node数组构成，相同hash值的数据放在数组的同一位上

Node有以下几种实现，分别对应链表node、红黑树的root、红黑树的node和占位node

| node        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| Node        | 默认实现，标识链表node                                       |
| TreeBin     | 红黑树的root节点，放在数组中                                 |
| TreeNode    | 红黑树的node节点，连接在TreeBin下面                          |
| ForwardNode | 在迁移时的占位node，标明当前Node数组正在迁移，当前位已经迁移完成，提供了新Node数组的引用，查数据去新Node数组查询 |
|             |                                                              |
|             |                                                              |



## 二、几种基本方法流程

### 1. get

当前hash>0，说明是正常的Node，按照链表遍历查询

当前hash<0，说明是TreeBin或ForwardNode，按照红黑树或去另一个数组中查询

<font color = "yellow" size = 5>因为Node的value和next指针都是volatile的，所以查询时无需加锁</font>



### 2. put

<font color = "yellow" size = 5>Key和Value都不允许为null,因为多线程场景，无法确定是不存在该key，还是该key的value为null。</font>

```java
public V put(K key,V value){
    if(node数组尚未初始化){
      if(cas 修改状态){//修改失败，说明被其他线程初始化
        初始化node数组
      }
    }else if(当前哈希值没有节点){
      通过cas直接写入;
    }else if(当前节点处于迁移状态){
      进行transfer，帮忙扩容;
    }else{//
      synchronized(node){
        按照链表/红黑树进行插入; 
      }
    }
    判断是否满足树化条件;
    进行累加以算出总数
    if(满足扩容条件){
      开始扩容
    }
}
```



### 3. size

有一个CellerCount数组，Node数组的每一位对应一个CellerCount，里面存着一个long类型对象进行个数统计。

size方法通过遍历CellerCount做sum，得到size

<font color = "yellow" size = 5>所以size不一定是准的</font>



### 4. resize

当前线程put后，检查大小，满足扩容条件，开始扩容。

其他线程put时，发现put的节点为ForwardNode，则帮忙扩容

每个线程分配数组中的16位，16位迁移完成后，发现还未结束，就一直重复，直到完成。



**摘抄自别人**

- **第一步**：计算出每个线程每次可以处理的个数，根据 Map 的长度，计算出每个线程（CPU）需要处理的桶（table数组的个数），默认每个线程每次处理 16 个桶，如果小于 16 个，则强制变成 16 个桶。
- **第二步**：对 nextTab 初始化，如果传入的新 table nextTab 为空，则对 nextTab 初始化，默认是原 table 的两倍
- **第三步**：引入 ForwardingNode、advance、finishing 变量来辅助扩容，ForwardingNode 表示该节点已经处理过，不需要在处理，advance 表示该线程是否可以下移到下一个桶（true：表示可以下移），finishing 表示是否结束扩容（true：结束扩容，false：未结束扩容） ，具体的逻辑就不说了
- **第四步**：跳过一些其他细节，直接到数据迁移这一块，**在数据转移的过程中会加 synchronized 锁，锁住头节点，同步化操作，防止 putVal 的时候向链表插入数据**
- **第五步**：进行数据迁移，**如果这个桶上的节点是链表或者红黑树，则会将节点数据分为低位和高位，计算的规则是通过该节点的 hash 值跟为扩容之前的 table 容器长度进行位运算（&），如果结果为 0 ，则将数据放在新表的低位（当前 table 中为 第 i 个位置，在新表中还是第 i 个位置），结果不为 0 ，则放在新表的高位（当前 table 中为第 i 个位置，在新表中的位置为 i + 当前 table 容器的长度）**。
- **第六步**：如果桶挂载的是红黑树，不仅需要分离出低位节点和高位节点，还需要判断低位和高位节点在新表以链表还是红黑树的形式存放。



## 三、常见问题

#### 1、如何保证线程安全

put时没有元素会通过cas设置元素

put时，链表/红黑树追加，会使用synchronize

put时发现正在扩容，会帮助扩容，并在扩容后进行put

红黑树旋转时，会锁根节点









## 四、别的blog

[扩容看这个](https://segmentfault.com/a/1190000021237438)

[其它的细节看这个](https://segmentfault.com/a/1190000022279729)

