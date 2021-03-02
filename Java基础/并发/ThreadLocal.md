## 														ThreadLocal

### 一、介绍

##### 		对于同一个threadLocal对象，在不同thread中，访问到的threadLocal.get()得到的值不一样，用于保存诸如uid，transactionId此类的值。

### 二、原理

#### 1、储存方式

​		以ThreadLocalMap的形式进行储存。ThreadLocalMap类似于HashMap，但是其Entry继承自WeakReference，当线程销毁时易于进行内存回收。entry的key为ThreadLocal自身，value为ThreadLocal中存放的变量。

#### 2、与Thread的关系

​		每个Thread中储存着一个ThreadLocalMap，保存当前线程中所有使用到的ThreadLocal。当程序需要保存/读取ThreadLocal中的值的时候，通过当前线程取到ThreadLocalMap，以当前ThreadLocal为key取出值。

​		**<font color="red">ThreadLocal和Thread是多对多的关系，一个线程中包含多个ThreadLocal变量；一个ThreadLocal变量也可以在多个线程中被读取。所以将ThreadLocalMap储存在每一个Thread中，是非常合适的选择</font>**

以下为伪代码

```java
public class ThreadLocal<T>{
	public T get(){
  	Thread thread = Thread.currentThread();
    ThreadLocalMap map = thread.threadLocalMap;
    if(map==null){
      return null;
		}
    return map.get(this);
	}
}
```

#### 3、为什么使用弱引用

​		有两个地方引用了ThreadLocal，一是创建ThreadLocal的类/方法；二是当前Thread中的ThreadLocalMap。**<font color="red">如果使用强引用，当创建ThreadLocal的类被回收，ThreadLocal无法被回收，因为被ThreadLocalMap中的Entry所引用，必须等到线程死亡才能被回收。</font>**如果使用线程池，则线程不会死亡，这部分永远不被回收，会越来越大。

所以必须使用弱引用

#### 4、内存泄漏

​	3中提到，Entry中使用了ThreadLocal弱引用，所以在ThreadLocal被销毁时，value并不会被销毁，从而造成了内存泄漏，暨key为空，无法通过key找到value，相应value的内存被泄露。虽然在java的实现方式上，每一次ThreadLocal的get、set、remove都主动删除了key为null的entry，我们仍然需要在结束时显式的进行remove操作，保证内存不被泄露，以防清除方法没有被调用



[可参考，说的很详细](https://blog.csdn.net/puppylpg/article/details/80433271)