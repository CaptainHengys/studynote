LinkedHashMap与TreeMap



LinkedHashMap

* 结构

  * 一个linkedlist+hashmap
  * `linkedlist<key>`用来维持顺序,`hashmap<key,value>`用来存储数据

* 插入顺序

  * put的时候,直接加入linkedlist尾部,再放入hashmap存储
  * get的时候,对linkedlist不做处理,直接从hashmap拿数据

* LRU顺序

  * put的时候,直接加入linkedlist尾部,再放入hashmap存储

  * get的时候,从linkedlist中找到,删除,在加入linkedlist尾部,然后从hashmap中拿数据

    

* LRU顺序的get()方法保证了linkedlist前面都是不常用的,后面的是最近最常用的数据

* 插入顺序则记录了元素插入的顺序,和hashmap遍历时的存储顺序是不一样的

* 缓存会用到!LRUCache

TreeMap

* 红黑树实现的!
* ...以后再补充