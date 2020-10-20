#### HashTable与Collections.synchronizedMap()

hashtable基本已经不推荐使用了,这里简单介绍一下经常被问到的问题

* hashtable底层结构?
  * 与hashmap类似,数组+链表的形式,不同的是在几乎每个方法都加上了synchronized,保证同步安全
  * 初始容量为11,如果传入容量初始值,那就直接用,不会扩充为2的幂次
  * 每次扩容扩为原来的2*n+1
* hashtable为什么不推荐使用了?
  * 因为每个方法都加上同步,会让效率变得很低
* 那你推荐用什么?
  * 可以使用ConcurrentHashMap或者Collections.SynchronizedMap()
* ConcurrentHashMap可以完全替代HashTable么?
  * 不可以1.7的ConcurrentHashMap是弱一致性,HashTable是强一致性,比如想找到集合中最大的元素,可能chm在这个段中找到了,但是另一个段同时修改了数据,最大值出现在了另一个个段,这就是由于分段锁的存在
  * 自我感觉1.8的ConcurrentHashMap貌似已经完全可以了



* 谈谈Collections.synchronizedMap()吧,他和hashtable有什么区别
  * hashtable锁级别是方法级别的,Collections.synchronizedMap()是代码块级别的锁方法,并且可以锁对象
  * 两者性能相近,但是Collections.synchronizedMap()允许null作为key或value(锁对象),这个时候只允许同时存在一个操作,有点串行化的感觉