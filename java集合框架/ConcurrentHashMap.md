#### ConcurrentHashMap

* **ConcurrentHashMap底层结构**

  ​		ConcurrentHashMap类似于HashMap的底层结构

  * Entry数组相对于HashMap有一定改变,HashEntry的value以及next都被volatile修饰
  * 1.7额外采用了分段锁的涉及,只有在同一个分段内才存在竞争关系,相对于对整个Map加锁,分段锁减小了加锁粒度,大大提高了高并发环境下的处理能力
  * 1.8采用Node数组+CAS+synchronized

* **为什么ConcurrentHashMap的HashEntry节点的value和next都被volatile修饰?**

  * 为了在多线程读写的过程中保持他们的可见性
  * **为什么要保证他们的可见性?**
    * 因为采用了CAS操作吧,保证线程间目的地址数据均可见

* **ConcurrentHashMap分段锁用的是什么锁?**

  * 1.6 1.7使用继承自ReentrantLock的Segment
  * 1.8是synchronzied

* **为什么1.8摒弃了分段锁,改用了synchronized?**

  * 用到了synchronized+CAS操作,效率与ReentrantLock不相上下
  * 加入多个分段锁浪费内存空间。
  * 生产环境中， map 在放入时竞争同一个锁的概率非常小，分段锁反而会造成更新等操作的长时间等待。
  * 为了提高 GC 的效率

* **ConcurrentHashMap各个版本申请锁(put)有变化么?**

  * 1.7版本在申请Segment之前,put会通过tryLock()尝试获取锁,tryLock()的过程中会对对应的hashcode的链表进行遍历,如果遍历完毕仍然找不到与key相同的HashEntry节点,则为后续的put操作提前创建一个节点,当tryLock一定次数后仍无法获取锁,则去通过lock申请锁
  * **为什么要对链表进行遍历?**
    - 是为了希望遍历的链表被CPU cache缓存,为后续实际put过程中的链表遍历提升性能
  * 1.8版本先判断是否是当前数组下表的第一个object,如果是,cas插入无需加锁,如果不是直接用链表第一个object加锁,这里加的是synchronized,还有红黑树的不同(涉及put)
  * **为什么又用了synchronized?**
    * 减小内存开销,要获得reentrantlock支持,那么每个节点都需要继承AQS,但是不是每个节点都需要获取同步支持的,浪费内存
    * synchronized是jvm级别的,jvm在运行时能采取相应的优化措施,而reentrantlock是jdk级别的

* **了解过get,containsKey么,返回的数据有什么问题么?**

  * 可能在遍历过程中返回的数据已经被并发修改,这也是chm弱一致性的原因

* **ConcurrentHashMap并发度是多少?**

  * 默认16

* **为什么设置这么大?**

  * 如果并发度设置的过小,会带来严重的锁竞争问题(段内竞争),如果并发度过大,原本位于一个Segment中的数据,会分散开,导致线程过多,CPU cache命中率下降,引起性能下降

* **ConcurrentHashMap的一致性有了解么?**

  * 弱一致性吧,ConcurrentHashMap的弱一致性主要是为了提升效率，是一致性与效率之间的一种权衡。要成为强一致性，就得到处使用锁，甚至是全局锁，这就与Hashtable和同步的HashMap一样了。可能你期望往ConcurrentHashMap底层数据结构中加入一个元素后，立马能对get可见，但ConcurrentHashMap并不能如你所愿

* **ConcurrentHashMap能取代HashTable么?**

  * 不能,HashTable是强一致性的,ConcurrentHashMap是弱一致性的,不可替代

* **那有其他的办法让HashMap同步么?**

  ```
  Map m = Collections.synchronizeMap(hashMap);
  ```

*  **Collections.synchronizeMap()底层原理是什么?**
  
  * 对每一个方法都加了同步锁,变成了一个类似于hashtable的结构,不过特点是,他接受任何map类型的传入
* **ConcurrentHashMap允许key或者value为null么?**
  
* 不允许,这么设计的原因是因为,一旦value为null,则代表key-value还没有映射完成就被其他线程所见,虽然这种情况实际中几乎不可能发生
  
* **谈谈ConcurrentHashMap1.8吧,做了什么改动么?**

  * 和HashMap1.8类似,引入红黑树,摒弃Segment启用CAS+synchronized算法

* **聊聊怎么计算ConcurrentHashMap的size大小**

  * 他不可能出现STW这种,让所有线程都停下来
  * 1.7采用计算3次,这三次中如果前后两次计算结果一样,说明计算出来是准确的,如果三次都不满足条件,会加锁计算
  * 1.8采用一个volatile修饰的basecount记录元素个数,当插入或删除数据会通过addcount()CAS更新basecount和counterCell,最终通过baseCount和遍历CounterCell数组得出size,并且通过mappingCount方法计算出来size的返回值是long

* **聊聊ConcurrentHashMap的扩容方法transfer**

  * 基本思想和hashMap很像,但是由于他是支持并发扩容的,所以复杂得多,原因是它支持多线程扩容操作,但是却并没有加锁,这样做的目的不仅满足了concurrent的要求,而且是更希望利用并发处理减少扩容带来的事件影响,因为扩容的时候,总是涉及一个数组到另一个数组的拷贝操作,如果这个操作可以并发进行,效率会很高

  * 单线程扩容

    * 第一步,和hashmap扩容一样,构建一个nextTable,然后进行遍历,复制(原位置或者原位置+旧长度)
    * 第二步,让新的nextTable作为新的table

  * 多线程扩容

    * **如果是空**,那么直接插入

    * **如果非空也不是forward节点**,那么加锁处理,执行数据复制

    * 巧妙立用forward值,多线程遍历节点,如果一个节点**非空且不是forward节点**,加锁,处理完一个节点就把对应点的值set为forward,另一个线程看到forward,就向后遍历,这样交叉就完成了复制工作,而且还很好的解决了线程安全问题,尽管这个有一些影响效率，但是还是会比hashTable的synchronized要好得多。

      