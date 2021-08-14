### 线程的进程的区别

a) 操作系统为正在运行的程序提供的抽象，就是所谓的进程（process）
	 1) 内存：指令, 地址空间，和程序读取和写入数据都在内存中
	 2）寄存器：指令的读取和更新
	 3) 程序计数器，栈指针，管理参数栈，局部变量和返回地址
	b) 线程（thread）是允许应用程序并发执行多个任务的一种机制
	 1) 如果线程出现IO阻塞，其他线程都可以继续运行
	-- 区别
	1) 进程之间的信息难以共享，没有共享内存 , (进程间的通信[IPC]),来进行进程间的交换
	2) 线程之前能够快速共享信息，只需要将数据复制到变量中(堆中)， 不过要避免多个线程同事修改同一份信息的情况

### Runable 和 callable 区别

1. 返回值： call 有，run 没有
2. 异常 : call 有， run 没有
3. 方法： call 方法，run 方法

### 集合

- List
  - ArrayList: 数组的方式实现，查询快，增加和删除的速度慢，线程不安全，创建时的容量为0，插入第一个元素的时候开始扩容，默认为10，以后每次的扩容因子为1.5
  - LinkedList: 双向循环列表的方式实现，有序可重复，查询速度慢，增加和删除速度快，线程不安全
  - Vector:数组的形式实现，有序可重复，查询速度快，增删速度慢，线程安全，效率低(synchronized方式实现)，创建时默认大小为10，扩容因子为2
  - Stack:Vector的子类，先进后出，数组实现，线程安全
  - CopyOnWriteList: 底层采用写时复制的方式实现,适用于读多写少的场景，线程安全，优点：无锁实现，缺点：只能保证数据的最终一致性，在添加到拷贝数组还没替换的时候读到的还是旧的数据，对象比较大的时候频繁的进行替换操作会导致大量GC
- Set
  - HashSet:底层使用了Hash表实现，内部使用了HashMap,无序不可重复，读写速度快，线程不安全
  - LinkedHashSet: 底层采用Hash表存储，并使用双向链表记录插入顺序，排列有序不可重复，存取速度比HashSet慢比TreeSet快，线程不安全
  - TreeSet: 底层采用红黑树实现，内部使用了NavigableMap 实现，按自然顺序或按指定顺序存放，不可重复，线程不安全
  - CopyOnWriteArraySet: 使用写时复制技术实现，适用于读多写少的场景，内部采用了CopyOnWriteList实现，同HashSet功能类似，线程安全
  - ConcurrentSkipListSet: 采用跳跃表的方式实现，适用于高并发的场景，内部使用了ConcurrentNavigableMap实现，同TreeSet功能相似，线程安全
- Queue
  - PriorityQueue:优先队列，底层基于优先堆的无界队列来实现，每次从队列中取出优先执行权最高的元素，线程不安全，不支持空值和不可比较的对象，默认字典顺序不是先进先出
  - ArrayBlockingQueue: 基于定长数组的阻塞队列的方式实现，线程安全的有界阻塞队列，内部通过互斥锁保护竞争资源，实现了多线程，竞争资源的互斥访问，锁是没有分离的，读取和写入不能同时进行，锁方面的性能不如LinekdBlockingQueue
  - LinkedBlockingQueue: 基于单向列表的阻塞队列实现，线程安全，锁分离，包含takeLock 和 putLock ，理论上性能是高于ArrayBlockingQueue
  - PriorityBlockingQueue
  - SynchronousQueue：线程安全无阻塞无边界的队列，每一个put必须等待take
  - DelayQueue: 延迟队列，有序无界阻塞队列，只有延迟期满才能获取队列元素，线程安全
- Map
  - HashMap:底层是用链表数组，Java8后又加了红黑树来实现，键无序不可重复可为null、值可重复可为null，存取速度快，线程不安全。
  - LinkedHashMap：底层是用链表数组存储，并用双向链表记录插入顺序，键有序不可重复可为null、值可重复可为null，存取速度快较HashMap略慢，比TreeMap快，线程不安全。
  - HashTable：底层是用链表数组，键无序不可重复可为null、值可重复可为null，存取速度较HashMap慢，线程安全。
  - TreeMap：底层使用红黑树来实现，内部使用了Comparator，按自然顺序或自定义顺序存放键，键不可重复不可为null、值可重复可为null，存取速度较HashMap慢，线程不安全。
  - WeakHashMap：同HashMap基本相似。区别在于，HashMap的key保留对象的强引用，这意味着只要该HashMap对象不被销毁，该HashMap对象所有key所引用的对象不会被垃圾回收，HashMap也不会自动删除这些key所对应的key-value对象；但WeakHashMap的key只保留对实际对象的弱引用，这意味着当垃圾回收了该key所对应的实际对象后，WeakHashMap会自动删除该key对应的key-value对象。
  - ConcurrentHashMap：底层使用锁分段技术来实现线程安全，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
  - ConcurrentSkipListMap：底层使用跳跃列表来实现，适用于高并发的场景，内部使用了ConcurrentNavigableMap，同TreeMap功能相似，是一个并发的、可排序的Map，线程安全。因此它可以在多线程环境中弥补ConcurrentHashMap不支持排序的问题。



### 线程池

- newCachedThreadPool：可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。工作线程的创建数量几乎没有限制，一定要注意控制任务的数量，否则，由于大量线程同时运行，很有会造成系统瘫痪
- newFixedThreadPool：指定工作线程数量的线程池，工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中，线程池中没有可运行任务时，它不会释放工作线程
- newSingleThreadExecutor：单线程化的Executor，保证顺序执行
- newScheduleThreadPool： 创建一个定长的线程池，而且支持定时的以及周期性的任务执行，支持定时及周期性任务执行

### volatile

- 保证可见性 ：保证内存可见性
- 不保证原子性：原子性是指一个操作是不可中断的，要全部执行完成，要不就都不执行
- 禁止指令重排：有序性即程序执行的顺序按照代码的先后顺序执行

### CAS（compare and Swap比较并交换）

实现：
1. Unsafe 类保证原子操作
2. 自旋锁do{}while 方式 保证版本号一致

缺点：

1. 连续的自旋会给CPU带来负担
2. 原子操作只能保证一个共享对象的原子操作，多个对象的时候无法保证，需要加锁
3. 引发ABA问题

### Synchronized 和 lock 区别

1.synchronized 是属于jvm层面的，底层是通过monitor对象来完成的，其实wait/notify等方法也依赖于monitor对象只有在同步块或方法中才能调用 wait/notify等方法
Lock是具体类api层面的锁:ReentrantLock

### 线程工具类

CountDownLatch:让一些线程阻塞直到另外一些完成后才被唤醒

CyclicBarrier:让一组线程到达一个屏障(也可以叫做同步点)时被阻塞,知道最后一个线程到达屏障时,屏障才会开门,所有被屏障拦截的线程才会继续干活

