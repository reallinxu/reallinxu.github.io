---
title: "[数据结构]1.集合"
tags:
  - 数据结构
id: 2019
categories:
  - 数据结构
date: 2019-08-27 15:20:00
author: 
  - linxu
---


## Map
* HashMap  
基于数组+链表，默认大于8使用红黑树，小于6改为链表，默认容量是16，负载因子默认为0.75，HashMap每次put操作是都会检查一遍 size（当前容量）>initailCapacity*loadFactor 是否成立。如果不成立则HashMap扩容为以前的两倍，HashMap中计算Hash值封装为：(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)，原因是hashCode是32位的，length 绝大多数情况小于2的16次方。所以始终是hashcode 的低16位参与运算，这样做是使高16位也参与运算，会让得到的下标更加散列。
* LinkHashMap
继承于HashMap，是基于HashMap和双向链表来实现的。LinkedHashMap有序，可分为插入顺序和访问顺序两种（默认插入顺序，可在构造方法中指定），如果是访问顺序，那put和get操作已存在的Entry时，都会把Entry移动到双向链表的表尾(其实是先删除再插入)。
* Hashtable  
线程安全，TODO：
* ConcurrentMap  
ConcurrentHashmap和Hashtable都是支持并发的，这样会有一个问题，当你通过get(k)获取对应的value时，如果获取到的是null时，你无法判断，它是put（k,v）的时候value为null，还是这个key从来没有做过映射。HashMap是非并发的，可以通过contains(key)来做这个判断。而支持并发的Map在调用m.contains（key）和m.get(key),m可能已经不同了。

## List
* ArrayList 
基于数组实现，存储在连续的内存空间上，查询很快，但是中间部分的插入和删除很慢。默认数据长度为10，当第一个元素添加时候初始化（如果使用传参构造方法会直接初始化），每次扩容为增加50%(size >> 1)
* LinkedList 
基于双向链表实现，查询相对较慢(需移动指针)，删除和添加比较占优势。
* Vector
底层是基于数组实现，线程安全，默认长度为10，直接初始化，可以设置扩容增量，如果未设置，每次为增加原来的一倍。Vector和ArrayList最大容量都为（Integer.MAX_VALUE - 8），是因为一些虚拟机在数组中保留一些头字，会导致OOM。

## Set
* HashSet
HashMap实现，key为存入的元素，value为一个static的空对象。判断元素是否重复为 由于HashMap中的key值不能修改，所以HashSet不能进行修改元素的操作。
* LinkedHashSet
继承自HashSet，基于LinkedHashMap，继承了HashSet的全部特性，元素不重复，快速查找，快速插入，并且新增了一个重要特性，那就是有序。
* TreeSet  
TreeSet是基于TreeMap实现的，TreeSet的元素支持2种排序方式：自然排序或者根据提供的Comparator进行排序。TreeSet可以直接对其进行存储，如String，Integer等,因为这些类已经实现了Comparable接口，当元素是对象时需要手动实现Comparable接口，覆盖其compareTo方法。

## Queue
### 非阻塞队列
> 非阻塞队列在添加（移除）时如果队列满（空）时会返回异常或者直接返回null，方法未做同步措施，采用CAS机制  

非阻塞队列有：  

* PriorityQueue  
类实质上维护了一个有序列表。加入到 Queue 中的元素根据它们的天然排序（通过其 java.util.Comparable 实现）或者根据传递给构造函数的 java.util.Comparator 实现来定位  
* ConcurrentLinkedQueue  
ConcurrentLinkedQueue 是基于链接节点的、线程安全的队列。并发访问不需要同步。因为它在队列的尾部添加元素并从头部删除它们，所以只要不需要知道队列的大小，ConcurrentLinkedQueue 对公共集合的共享访问就可以工作得很好，收集关于队列大小的信息会很慢，需要遍历队列。  
 
非阻塞队列提供的主要方法如下：  

* add(E e)：将元素e插入到队列末尾，如果插入成功，则返回true；如果插入失败（即队列已满），则会抛出异常；  
* remove()：移除队首元素，若移除成功，则返回true；如果移除失败（队列为空），则会抛出异常；  
* offer(E e)：将元素e插入到队列末尾，如果插入成功，则返回true；如果插入失败（即队列已满），则返回false；  
* poll()：移除并获取队首元素，若成功，则返回队首元素；否则返回null；  
* peek()：获取队首元素，若成功，则返回队首元素；否则返回null  

对于非阻塞队列，一般情况下建议使用offer、poll和peek三个方法，不建议使用add和remove方法。因为使用offer、poll和peek三个方法可以通过返回值判断操作成功与否，而使用add和remove方法却不能达到这样的效果。注意，非阻塞队列中的方法都没有进行同步措施。

### 阻塞队列
> 阻塞队列在添加（移除）时如果队列满（空）时会等待（或超时退出），方法做同步措施，使用 ReentrantLock 锁

阻塞队列有：

* ArrayBlockingQueue  
ArrayListBlockingQueue是有界的，是一个有界缓存的等待队列。基于数组的阻塞队列，同LinkedBlockingQueue类似，内部维持着一个定长数据缓冲队列（该队列由数组构成）。ArrayBlockingQueue内部保存着两个整形变量标识着队列的头部和尾部在数组中的位置。生产者放入数据和消费者获取数据，都是共用同一个锁对象，由此两者无法真正并行运行，这点不同于LinkedBlockingQueue；ArrayBlockingQueue在插入或删除元素时不会产生或销毁任何额外的对象实例，而后者则会生成一个额外的Node对象。这在长时间内需要高效并发地处理大批量数据的系统中，其对于GC的影响还是存在一定的区别。
* LinkedBlockingQueue  
可以设定大小，可以理解为由链表结构组成的一个缓存的有界等待队列。生产者端和消费者端分别采用了独立的锁来控制数据同步。
* PriorityBlockingQueue  
一个支持优先级排序的无界阻塞队列。元素按优先级顺序被移除，该队列也没有上限，注意的是有序的队列需要进入的元素具有比较能力。
* DelayQueue  
一个使用优先级队列实现的无界阻塞队列。是一个存放 Delayed 元素的无界阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且poll将返回null。当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于或等于零的值时，则出现期满，poll就以移除这个元素了。此队列不允许使用 null 元素。
* SynchronousQueue  
一个不存储元素的阻塞队列。是一种无缓冲的等待队列，但是由于该Queue本身的特性，在某次添加元素后必须等待其他线程取走后才能继续添加；可以认为SynchronousQueue是一个缓存值为1的阻塞队列，但是 isEmpty()方法永远返回是true，remainingCapacity() 方法永远返回是0，remove()和removeAll() 方法永远返回是false，iterator()方法永远返回空，peek()方法永远返回null。没有存储功能，因此put和take会一直阻塞，直到有另一个线程已经准备好参与到交付过程中。
* LinkedTransferQueue  
一个由链表结构组成的无界阻塞队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。LinkedTransferQueue采用一种预占模式。意思就是消费者线程取元素时，如果队列不为空，则直接取走数据，若队列为空，那就生成一个节点（节点元素为null）入队，然后消费者线程被等待在这个节点上，后面生产者线程入队时发现有一个元素为null的节点，生产者线程就不入队了，直接就将元素填充到该节点，并唤醒该节点等待的线程，被唤醒的消费者线程取走元素，从调用的方法返回。我们称这种节点操作为“匹配”方式。
* LinkedBlockingDeque  
一个由链表结构组成的双向阻塞队列。即可以从队列的两端插入和移除元素。双向队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。 LinkedBlockingDeque多了addFirst、addLast、peekFirst、peekLast等方法，以first结尾的方法，表示插入、获取获移除双端队列的第一个元素。以last结尾的方法，表示插入、获取获移除双端队列的最后一个元素。 

 
阻塞队列提供的主要方法如下：  

* add：增加一个元索，如果队列已满，则抛出一个IIIegaISlabEepeplian异常  
* remove：移除并返回队列头部的元素，如果队列为空，则抛出一个NoSuchElementException异常  
* element：返回队列头部的元素，如果队列为空，则抛出一个NoSuchElementException异常  
* offer：添加一个元素并返回true，如果队列已满，则返回false  
* poll：移除并返问队列头部的元素，如果队列为空，则返回null  
* peek：返回队列头部的元素，如果队列为空，则返回null  
* put：添加一个元素，如果队列满，则阻塞  
* take：移除并返回队列头部的元素，如果队列为空，则阻塞  



> 参考博客：  
https://www.cnblogs.com/tiancai/p/9935770.html  
https://www.cnblogs.com/lemon-flm/p/7877898.html