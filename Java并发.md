## Java并发
jdk1.8
- AQS原理
- JUC包

### Java同步容器和并发容器
1. 同步容器
同步容器默认是线程安全的，对它的操作都是已经默认加了同步锁的，同步容器主要包括2类：
- Vector、Stack、HashTable
	- Vector实现了List接口，Vector底层是一个数组，其对于数组的各种操作和ArrayList几乎一样，唯一不同的在于大部分线程不安全的方法都加了syncrhoized关键字去限定；
	- Stack底层也是一个数组，它继承于Vector类，很多方法也用syncrhoized关键字加了锁；
	- HashTable实现了Map接口，它的实现原理几乎和HashMap一样。但是HashTable对很多方法都加了syncrhoized关键字进行限定
- Collections 工具类中提供的同步集合类
Collections类是一个工具类，相当于Arrays类对于Array的支持，Collections类中提供了大量对集合或者容器进行排序、查找的方法；
2. 并发容器
同步容器是通过syncrhoized关键字对线程不安全的操作进行加锁来保证线程安全的，其原理是使得多线程轮流获取同步锁进行对集合的操作，所以性能有所下降；juc提供了多种并发容器，在原有集合上的拷贝进行操作，用修改后的集合替换原有集合，以达到并发安全的使用集合类的目的；
根据接口的类型，主要有以下四种接口，其他具体的容器均是对这些接口的实现类:
- Queue: 阻塞队列BlockingQueue，非阻塞队列ConcurrentLinkedQueue;
- Map: ConcurrentMap
- Set: ConcurrentSkipListSet, CopyOnWriteArraySet
- List: CopyOnWriteArrayList

### juc - BlockingQueue
