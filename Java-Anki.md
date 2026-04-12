# 基础



面向对象三大特性?

> 封装（Encapsulation）：
> 概念：是指将数据（属性）和操作数据的方法（行为）捆绑在一起形成一个独立的单元（类/对象），并隐藏其内部实现细节，而是仅通过公共方法对外提供访问接口。
> 好处：数据被保护在封装体内部，对外隐藏实现细节。
> 继承（Inheritance）：
> 概念：是指让类与类之间产生父子关系。它允许一个类（子类）从另一个已存在的类（父类）中继承属性和方法，从而实现代码的复用，并建立类之间的层级关系。
> 好处：提高代码复用性，类与类之间建立“is-a”的关系。
> 多态（Polymorphism）：
> 概念：是指允许父类引用指向子类对象，并在运行时根据对象的实际类型调用相应的方法。它使得“一个接口，多种实现”成为现实。
> 好处：提高代码的灵活性和可扩展性。



对多态的理解?

> 定义：
> 多态（Polymorphism）是指“一个接口，多种实现”。在 Java 中，多态主要体现在——允许父类引用指向子类对象，并在运行时根据对象的实际类型调用相应的方法，即动态绑定机制。
> 特点：
> 运行时绑定（动态绑定）：这是多态的核心机制，编译器在编译时只知道引用变量的类型（静态类型），但在程序运行时，JVM 会根据对象的实际类型（动态类型）来查找并调用相应的方法。
> 提高可扩展性：增加新的子类时，无需修改现有代码，只需让新子类重写父类方法。
> 实现多态的三个必要条件：
> 继承或实现关系：必须存在子父类继承关系或接口实现关系。
> 方法重写（Override）：子类必须重写父类的方法（或实现接口方法）。
> 父类引用指向子类对象。
> 优点：
> 可维护性高：修改具体实现不影响调用方。
> 可扩展性强：新增子类只需实现父类接口或重写方法，无需修改调用方代码。
> 缺点：
> 父类引用不能直接使用子类的特有成员：这是多态的一个限制。例如，Animal animal = new Dog();，animal 引用对象无法直接调用 Dog 类特有的方法，除非进行强制类型转换。

equals与==的区别?

> 使用"=="进行比较：
> ==是一个比较运算符，既可以判断基本类型，又可以判断引用类型。
> 如果判断基本类型，判断的是二者的值是否相等(eg : 判断1 == 1，结果为true；判断1 == 3，结果为false)；
> 如果判断引用类型，判断的是二者的地址是否相同，即判定是否为同一对象(eg : Student student1 = new Student();，Student student2 = student1；判断student2 == student1，结果为true)。
> 使用equals()方法进行比较：
> equals()方法是顶层父类Object类中的方法，
> 可以看到，Object类中的 equals 方法用来检测两个对象是否相等，即默认情况下比较的是两个对象的引用(地址)。这一点和 == 用于判断引用类型时一致。
> equals的特点在于，它是Object类中的方法，因此，equals方法往往在子类中被重写，例如在String类中，equals方法被重写去判断两个字符串的内容是否相等。并且，在我们自己创建的类中，equals方法也常常被重写，去判断两个对象的指定的具体内容是否一致。
> 还有一点要注意，“==”的运行速度通常比“equals方法”更快；因为==比较引用类型时，仅比较地址；而equals方法的性能要取决于具体实现。

介绍一下泛型以及其作用?

> 泛型的概念：
> 泛型（Generics）是 Java 的一种参数化类型机制，允许在定义类、接口或方法时声明类型参数，并在使用时指定具体类型，实现类型安全的代码复用。
> 泛型的作用：
> 类型安全：泛型强制在编译时检查类型匹配，避免运行时出现 ClassCastException。
> 消除强制转换：泛型自动处理类型转换，无需手动进行强制类型转换。
> 代码复用：适用于编写通用算法和数据结构（如 List\<T>、Map<K,V>）。
> 泛型的应用：
> 自定义泛型类：泛型最后代表的数据类型是在创建对象时确定的。
> 自定义泛型接口：泛型最终代表的数据类型是在继承该接口或者实现该接口时确定的。
> 自定义泛型方法：泛型最终代表的数据类型是在调用方法时确定的，每次调用泛型方法，都可以指定不同的泛型类型。
> 通过使用通配符（?）和边界（<? extends T>、<? super T>），可以实现灵活的类型匹配。
> ?extend T 是生产者，?super T是消费者
> List<String> 和 List<Object>实现方式完全相同，List<T>类在类型擦除后，T一定是object对象。所以他们两个相同，不过编译器为List<String>实现了强制转换。

为什么重写equals要重写hashcode？

> **两个对象equals相等，那么hashCode必须相同**。如果你只重写了equals没重写hashCode,equals相同的两个对象,hashcode不同,在使用HashMap、HashSet这类基于哈希的集合时会出问题。

方法重载和重写的区别？

> 方法重载（Overload）：在同一个类中定义多个同名方法，但参数列表（类型、数量或顺序）不同。
> 方法重写（Override）：在子类中重新定义父类的方法，要求方法签名（方法名、参数列表、返回类型）完全相同。

Stringbuilder原理?

> 内部维护char[],默认容量16，需要扩容时，核心就是原始容量翻倍加 2，如果不够，直接用实际需要的大小。
> Java 9 之后内部从 char[] 改为了 byte[]，配合一个 coder 标志位，对纯 Latin-1 字符串只用一个字节存储，否则用utf16，两个字节存储

String、StringBuffer与StringBuilder区别?

> **可变性**，**线程安全性**，**性能**，**应用场景**
> StringBuffer方法由synchronized同步。



BIO、NIO与AIO的区别:

> BIO、NIO、AIO是Java中用于处理I/O操作的几种不同的模型。
>
> Basic,同步阻塞,NIO: None-Blocking**同步非阻塞**模型.Async IO:异步非阻塞.
>
> 同步阻塞适合连接少且固定,NIO适合连接数较多且连接时间较短的情况.AIO适合连接数较多且连接时间较长的情况。
>
> 阻塞指的是执行到IO时,线程会一直阻塞,同步指的是**数据从内核空间拷贝到用户空间这个阶段，线程需要自己等待完成**。
>
> 一次IO分为:阶段一：等待数据就绪（数据到达网卡/磁盘准备好）,阶段二：将数据从内核缓冲区拷贝到用户缓冲区
>
> NIO的非阻塞,指的是不需要干等数据就绪，可以通过 Selector 轮询多个 Channel，哪个就绪了就处理哪个，没就绪就去干别的,相比于BIO的阻塞(在read时如果数据没就绪,线程挂起),非阻塞调用read如果数据没好就立刻返回去做别的事情.
>
> 同步指的是数据就绪后，从内核拷贝到用户空间这一步，还是线程自己去做的
>
> 而AIO作为异步非阻塞,不是线程自己执行数据拷贝,而是开启另一个线程，并将IO操作交给另一个线程执行,另一个线程完成后,执行回调函数.AIO基于事件和回调机制实现的。
>
> NIO比BIO更常用，而AIO的应用相对较少。
>
> ![image](./assets/20250925_NIO.jpg)
> NIO的Selector是如何实现“多路复用” 的？
>
> linux中依赖epoll,先将Channel**注册**到Selector中，操作系统内核会**监听**IO事件，当事件发生时，内核通知Selector，**唤醒**线程处理事件，这样可以**避免线程阻塞**在单个IO操作上，实现**多路复用**。
>

Error与Exception的区别?

> 都继承自throwable
> 1. Exception (异常) ：表示程序在运行过程中可能遇到的、可以被捕获和处理的异常情况。这些异常通常是由于外部因素或程序逻辑错误导致的，是可恢复的。
> 2. Error (错误) ：表示 JVM 内部或系统级别的严重问题，通常是致命的，应用程序无法预料和恢复。

Java异常类型有哪些？

> Java 中的异常都继承自 java.lang.Throwable 类，它分为两大类：Error 和 Exception。
> Error(错误) ：
> 表示 JVM 内部 或 系统级别的问题，通常是很严重的，程序无法恢复。例如：OutOfMemoryError (内存溢出)、StackOverflowError (栈溢出)。
> Exception(异常) ：
> 表示程序在 编译时 或 运行时 可能发生的问题，通常是可捕获和处理的。Exception 又分为两大子类：
>
> 1. 编译期异常 (Checked Exception) ：是指编译器强制检查的异常，必须进行 try-catch 捕获处理 或者 throws 声明处理，常见的编译期异常有 IOException (输入输出异常)、SQLException (数据库操作异常)。
> 2. 运行期异常 (Runtime Exception / Unchecked Exception) ：是指编译器不强制检查的异常，通常是程序逻辑错误导致的，可选择性处理，常见的运行期异常有 NullPointerException (空指针异常)、ArrayIndexOutOfBoundsException (数组越界异常)。

try-catch代码块的逻辑?

> finally 永远会执行，且在 return 之前执行，所以
> finally的return会覆盖try/catch 的 return，也会覆盖try中抛出的异常。 finally中抛出的异常会覆盖try中的异常
> return时，确定一个快照，返回值已经被复制到一个临时变量中。
> finally更改基本类型，则无法影响return，但更改引用类型，则可以影响return。

# 集合

常见集合类?

> Collection 接口（单列集合）：
>
> 1. List ：它的特点是“有序、可重复”。List接口的常见实现类有：ArrayList, Vector 和 LinkedList。
> 2. Set ：Set集合的特点是“无序、不可重复”。Set接口的常见实现类有：HashSet, TreeSet。
> 3. Queue ：队列，它的特点是先进先出（FIFO）。Queue接口的常见实现类有：PriorityQueue。
> 4. Map 接口（双列集合）：
>    Map接口的常见实现类有：：HashMap, Hashtable, TreeMap。

Map接口的实现类?

> HashMap: HashMap是基于哈希表实现的，可以提供快速的键值对存取，但是不保证迭代顺序。它允许 null 键和 null 值，但它是非线程安全的。
> LinkedHashMap: LinkedHashMap继承自 HashMap，它通过内部维护的双向链表来维护元素的插入顺序或访问顺序。LinkedHashMap也允许 null 键和 null 值，但它也是非线程安全的。
> TreeMap: TreeMap是基于红黑树实现的，它按照键的自然顺序 或者是 自定义的比较器进行排序。TreeMap并不允许 null 键，它也是非线程安全的。
> ConcurrentHashMap: ConcurrentHashMap是目前常用的高性能的线程安全 Map。它通过分段锁（JDK 1.7）或 CAS + synchronized（JDK 1.8+）来实现并发控制。ConcurrentHashMap不允许 null 键。
> Hashtable: Hashtable算是比较传统的线程安全 Map，它的所有操作都通过 synchronized 关键字同步，因此性能较低。Hashtable既不允许 null 键，也不允许 null 值。
> Properties: 它是 Hashtable 的子类，主要用于读写键和值都是 String 类型的配置文件。

ArrayList扩容机制?

> ArrayList 的扩容机制：
> 当其内部存储元素的数组，它的容量不足以容纳新元素时，ArrayList 会自动创建一个更大的新数组，并将原数组中的所有元素都复制到这个新数组中。
> 这个过程发生在当内部数组的 size 等于 elementData.length 并且还调用 add() 方法时触发。
> 默认的扩容策略是将当前容量扩大 1.5 倍（newCapacity = oldCapacity + (oldCapacity >> 1)）。虽然单次扩容涉及元素复制，时间复杂度为 O(N)，但由于容量是指数级增长的，因此向 ArrayList 中添加元素的均摊时间复杂度为 O(1) 。

HashMap实现原理?

> 底部存储entry数组，
> jdk1.7采用数组+链表形式，采用头插法。把新加入的entry放到链表头部，目的是后插入的元素先访问。
> jdk1.8之后采用数组+链表+红黑树，使用尾插法，当链表长度大于8，整体元素个数大于64进行树化，选用尾插法因为多线程下头插法会出现循环链表问题。

hashmap的hash函数?

> HashMap的hash函数，用到hashcode方法，并进行扰动处理。 (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); 前16位和后16位异或。 由hash值定位桶位置：index = hash & (table.length - 1)
> 当hashcode定位到桶位置，如果有多个元素，则调用equals比较，确定哪个是key。
> equals相同的两个对象则hashcode一定相同，只重写equals()而不重写hashCode()，会导致两个相同的对象分配到不同的桶，违反了hashmap的唯一性，出现能存进去但是取不出来。

HashMap的put方法?

> put首先计算hash值，定位存储索引位置，检查对应位置是否有值。 
> 如果有，调用equals方法检查key是否存在，如果存在则更新旧值，否则尾插。
> 插入完毕后检查是否树化，以及hashmap是否扩容。

HashMap扩容机制?

> 进行扩容时，遍历链表每个桶。
> - 没有元素，则跳过
> - 有一个元素，则直接计算hash&（newCap-1）
> - 是链表，则对链表每一个元素计算。 同一个链表中的元素的hash值不一定相同，只是在oldcap下分配到了一个链表。 现在是newcap，则拆分成两个链表：分为原位置和新位置。 根据用原先位置key的hash值与旧数组的长度进行“与”操作，为0则是原位置链表，为1则是新位置链表。
> - 是红黑树，则跟链表一样拆分成两个链表，不过进行额外的判断，如果链表长度<=6，则退化为链表，大于6则把链表进行树化。

在 resize() 方法中，为什么链表在扩容时，元素只会分到两个位置（原位置 j 或 j + oldCap），而不是完全重新计算哈希值？这种优化是如何实现的？

> resize会让容量翻倍，并且旧容量和新容量都是2的幂次方，检查元素的hash值在oldCap位的二进制是0还是1，(e.hash & oldCap) == 0 直接判断它在新数组的位置，避免重新计算

为什么HashMap数组长度是2的n次幂？

> 用位运算代替取模。index = hash & (length - 1)
> 扩容时拆分链表也是利用了这个特性，hash & oldCap

HashMap为什么不是线程安全?

> hashmap的操作不是原子的，多线程操作让hashmap的底层数组出现数据不一致。
> 从原子性，可见性，有序性出发：
> 原子性：HashMap的put()方法不是原子操作，并发时会被中断，导致数据覆盖。
> 可见性：快速失败设计模式：HashMap的modCount数组元素等共享变量未使用volatile修饰，线程A修改后，线程B可能看到旧值，导致迭代器判断错误。

为什么hashmap允许null但是concurrenthashmap不允许?

> HashMap 允许 null 是因为单线程下有办法消除歧义，调用containsKey在调用get。
> 但是如果concurrenthashmap允许null，则调用containsKey时，再执行get，在这个间隙中，key被移除，所以get得到的null值会有歧义，无法区分是否是不存在还是值为null。

Hash冲突解决方案?

> 开放地址法： 是指当发生冲突时，在数组中寻找下一个可用的空位，由线性探测，二次方探测，双重哈希。使用两个不同的哈希函数。第一个用于计算初始位置，第二个用于计算每次探测的步长
> 链地址法（拉链法）：是指在冲突位置拉出一个数据结构（通常是链表），然后将所有冲突的元素都存放在这个数据结构中。
>
> 为什么hashmap采用拉链法？
> 链地址法在删除元素时非常简单，只需要将链表中的对应节点移除即可，开放地址删除元素麻烦。
> 开放地址法对扩容因子（数组拥挤，导致冲突多），和哈希函数的均匀性（不匀均导致冲突多）要求高

## 并发安全集合

ConcurrentHashMap 的实现原理?

> jdk7采用分段锁，整个map被分为若干段，对不同的段加段锁来控制并发。段Segment集成reentrantLock，包含键值对数组。
> 而jdk8中，使用更细粒度的锁，是对桶加锁，配合 CAS + synchronized控制并发
> 对于读操作，使用volatile保证可见性，Node的val和next都是volatile修饰的.所以针对get不用加锁，保证读取的是最新的内容
> 对于写操作，优先使用cas尝试插入，插入不成功就用synchronized代码块加锁。

concurrenthashmap的put流程?

> 首先计算key的hash，定位到node数组桶位置，如果桶为空，直接cas插入节点，cas失败会退化为synchronized代码块来插入节点。插入链表，有必要时进行树化。

concurrenthashmap如何保证可见性?

> ConcurrentHashMap 中的 Node 节点中，value 和 next 都是 volatile 的，这样就可以保证对 value 或 next 的更新会被其他线程立即看到。

为什么concurrenthashmap比hashtable的效率高?

> Hashtable 在任何时刻只允许一个线程访问整个 Map，他是对整个map进行加锁，而concurrenthashmap非必要不加锁，首先用cas判断，cas失败再加锁。并且是对数组中的桶进行加锁，粒度细。get是无锁的，因为用volatile修饰。

CopyOnWriteArrayList 的实现原理?

> 适合读多写少，在读不加锁，内部使用 volatile 变量来修饰数组 array，以确保读操作的内存可见性。
> 写时加锁，用reentrantLock，创建新的数组，修改完之后替换旧数组。

什么是BlockQueue？

> 线程安全队列，支持阻塞的生产者和消费者模型。
> 它的实现类有：
> 1. ArrayBlockingQueue
> 2. LinkedBlockingQueue
> 3. PriorityBlockingQueue 按优先级取出
> 4. DelayQueue 元素到期后才能取出
> 5. SynchronousQueue 同步队列，容量为0，必须一对一交换数据，适用于高吞吐的任务提交。



# 并发

进程和线程的区别?

> 1. 进程是操作系统资源分配的基本单位，线程是 CPU 调度的基本单位。一个进程中可以包含多个线程。
> 2. 资源: 进程之间的资源互相独立,而同一进程下的线程共享资源,线程只有自己独立的栈和计数器
> 3. 开销: 进程的创建,切换,销毁开销大,线程开销小.
> 4. 通信方式: 进程之间通信用到IPC机制(Inter-Process Communication),如管道,消息队列和共享内存,比较复杂,而线程之间通过共享内存直接读写公共变量.
> 5. 安全性: 进程之间互不影响,但是多线程中一个线程的崩溃会让整个进程崩溃.
>
> 具体的例子:chrome的标签页之间属于进程,互相独立,而java线程池属于线程.

进程之间通信的方式?

> 1. 管道.本质上是内核开辟的共享内存缓冲区,数据单向流动,进程写和进程读.一头只能写,一头只能读.特点是单向和阻塞.
>    1. 是字节流,读取方只能按顺序读.
> 2. 消息队列:进程通过内核维护的消息队列收发消息,
>    1. 消息之间是独立的,进程可以按消息类型筛选,读取特定的消息.
> 3. 共享内存:多个进程直接读写同一块内存,速度最快,但要配合信号量做同步
> 4. socket:通用方式,不仅可以本机通信,也可以跨网络,HTTP/RPC的底层都是socket
>    1. 一个 Socket 由五元组唯一标识：**协议、源IP、源端口、目标IP、目标端口**。
>    2. 它是一层抽象,对外提供读写接口,屏蔽底层细节.调用传输层能力,介于应用层和传输层中间.

java创建的线程,跟操作系统层面的线程如何对应?

> 在主流的 JVM（HotSpot）中，Java 线程和操作系统线程是**一对一**的关系。java线程的调度和切换由操作系统,而不是jvm进行处理.就是1:1线程模型
>
> 实现简单,能利用多核cpu.但是线程的创建和切换要进入内核态,开销大.所以用线程池复用
>
> Java 21 正式引入了**虚拟线程**,这是一种 **N:M 模型**，大量的虚拟线程由 JVM 调度,适合IO密集场景.
>
> 1:1,N:1,N:M. 1:1属于内核线程,N:1则是用户级线程,线程全部由用户调度,内核无法感知,那么切换很快,不涉及内核,但是一个线程的阻塞,整个进程都阻塞,并且无法利用多核.
>
> N:M是混合线程,结合N:1和1:1的优点

java创建线程的方式:

> 1. 继承Thread类,重写run()方法。 创建对象就是创建线程对象.并调用start
> 2. 实现Runnable接口，实现run()方法。 new Thread(new Runnable());再调用start
> 3. 实现Callable接口（配合Future/FutureTask），实现call()方法。call可以有返回值和抛出异常
>    1. new Thread()只能传入runnable,不能直接传入callable接口,需要套一层,new FutureTask<>(callable),FutureTask实现了runnable接口,所以可以传入new Thread()
> 4. 使用线程池(Executor框架)。向线程池提交任务,线程池管理线程的生命周期.
>    1. 提交runnable,使用executor.execute
>    2. 提交callable,使用executor.submit

start和run的区别?

> **start()本质是native方法**，会启动一个新线程，把线程设置为就绪状态,调度器选择该线程之后,由JVM调用run()方法. start只能调用一次
>
> 直接调用run是作为普通方法在当前线程执行.

线程的状态有哪些?

> ![image](./assets/Thread_states-20260409164745505.jpg)

避免Future的get()方法长时间阻塞:

**get(long timeout, TimeUnit unit)**  先判断isDone,再调用get

手动创建线程是**即用即建**，线程池能够实现**预先创建和线程复用**



java通过那些方式实现线程安全？

> 实现Java线程安全，需要**控制多个线程对共享资源的并发访问**，避免出现**数据不一致**，**竞态条件**等问题
>
> 分为三种方式: 
>
> - 阻塞调用,通过“**加锁-释放锁**”控制线程对资源的访问，未获取锁的线程会阻塞等待。
>   - synchronized 隐式锁
>   - Lock显式锁
> - 非阻塞调用
>   - CAS原子类
>   - 版本号
> - 无同步方案(避免共享资源)
>   - 不可变对象
>   - 线程本地副本,线程有独立的资源副本,比如ThreadLocal

synchronized如何保证原子性,可见性,有序性?

> synchronized / Lock 保证 原子性,可见性,有序性
>
> 原子性: 本质是不被插入,而不是不中断.
>
> synchronized / Lock获得锁之后,除非主动释放锁,其他线程无法进入到代码块,无法看到和修改中间状态,线程切换不会释放锁.
>
> 可见性:JMM 定义:加锁时，清空工作内存，从主内存重新读取共享变量；解锁时，把工作内存中修改过的变量刷回主内存。所以synchronized的修改对下一个线程可见. 通过 JMM 的内存同步规则**硬性保证**
>
> 有序性:指的是**指令重排导致其他线程看到不合理的中间结果**。
>
> synchronized 并没有禁止内部的指令重排序,但因为同一时刻只有一个线程在执行这段代码，即使内部重排了，没有其他线程能观察到中间状态,所以有序性得到保证.

为什么volatile如何保证可见性和有序性,无法保证原子性?

> 可见性:强制读写主内存,读直接从主内存读,写直接写入主内存.普通变量操作的是工作内存的副本,刷回主内存不确定.
>
> 有序性: 编译器在 volatile 读写前后插入内存屏障，限制重排序. 
>
> 规则是: **volatile 写之前的操作不会被重排到写之后，volatile 读之后的操作不会被重排到读之前。**
>
> 不保证原子性: 以 `i++` 为例，它其实是三步操作,读取i,i+1,写入i. 两个线程都读到了 0，各自加 1 写回，第二次写覆盖了第一次。 volatile是无锁的,无法阻止其他线程插入一个线程的读和写中间.

volatile和synchronized的对比?

> volatile修饰变量, 修饰引用变量时,保证引用变量的指向的对象地址是最新的,但不能保证引用对象内部状态的线程安全.
>
> - synchronized是确保**数据的一致性**和**线程安全**，解决多个线程之间访问资源的同步性。而volatile是确保变量在多个线程的**可见性**和**有序性**。
> - volatile不需要获取/释放锁，性能较高且轻量级。
> - synchronized可以修饰**方法以及代码块**，volatile只能修饰**变量**。

Atomic不保证有序性?

> Atomic 底层用volatile修饰,对单个变量的读写是原子且可见的，但它没有像 synchronized 那样的互斥区域，无法保证多个变量之间的操作顺序

针对有序性和可见性,如何实现双重检查锁单例模式?

> 使用volatile和synchronized的单例模式:
>
> ```java
> public class Singleton {
>     // volatile 防止指令重排序
>     private static volatile Singleton instance;
> 
>     // 私有构造，防止外部 new
>     private Singleton() {}
> 
>     public static Singleton getInstance() {
>         // 第一次检查：避免每次都加锁，提升性能
>         if (instance == null) {
>             synchronized (Singleton.class) {
>                 // 第二次检查：防止多个线程同时通过第一次检查后重复创建
>                 if (instance == null) {
>                     instance = new Singleton();
>                 }
>             }
>         }
>         return instance;
>     }
> }
> ```
>
> 为什么需要volatile: instance = new Singleton() 分三步,分配内存,初始化对象和把引用指向对象地址.如果重拍为132,则
>
> 线程A 执行到第3步，引用已赋值但对象没初始化,线程B 在第一次检查时发现 instance != null,直接返回没初始化的对象.
>
> 为什么需要两次检查?
>
> 线程A 和 线程B 同时发现 instance == null 线程A 拿到锁，创建实例，释放锁; 线程B 拿到锁，又创建了一个实例 → 单例被破坏



synchronized的原理?

> 通过**互斥锁机制**保证多线程对共享资源的安全访问,使用synchronized后，会在编译之后在代码前后加上**monitorenter**和**monitorexit**字节码指令.
>
> **monitorenter**尝试获取对象锁,让锁的计数器+1,竞争锁的线程会进入阻塞. 执行**monitorexit**指令时会把计数器减一,计数器为0锁释放,等待队里中的线程竞争锁.
>
> synchronized修饰静态方法和普通方法的区别,修饰静态代码块和普通代码块的区别?
>
> 锁静态方法和静态代码块是对类对象加锁,对类的所有实例调用都排斥,类实例对象调用静态方法会互斥.而锁普通方法是锁对象实例,只对同一个对象互斥.

锁升级策略

> 无锁->有线程访问->偏向锁->产生锁竞争->轻量级锁->自旋失败->重量级锁
>
> 对象刚创建时，Mark Word没有锁标记.
>
> 第一个线程进入同步块时，通过 CAS 把自己的线程 ID 写入 Mark Word,相同线程再次进入只需要比对线程ID,不需cas
>
> 当第二个线程来竞争时，偏向锁撤销，升级为轻量级锁。每个线程在自己的栈帧中创建一个 Lock Record，然后通过 CAS 尝试把 Mark Word 替换为指向自己 Lock Record 的指针,CAS失败的会自旋重试.
>
> 自旋超过次数,或者等待线程过多(超过一个线程在自旋就直接升级),升级为重量锁,Mark Word 指向一个操作系统级别的 Monitor 对象.
>
> 线程获取不到锁就被操作系统挂起，需要内核态和用户态切换

对比synchronized和reentrantLock

> synchronized：是**基于JVM的内置锁**，只能**用于方法和代码块**，需要与**wait()** 和**notify()/notifyAll()** 方法一起使用，用于线程等待和通知。
>
> lock：是接口,是显式锁,需要手动获取和释放,更加灵活并且可以设置锁的公平性. 而与Condition接口结合可以实现**更细粒度的线程等待和通知机制**。
>
> ReentrantLock：是Lock接口的一个**具体实现类**
>
> 相同点:
>
> - 都是可重入锁,同一个线程可以多次获取同一个锁。
>
> 不同点:
>
> - synchronized是非公平锁,reentrantlock支持锁公平设置
> - **中断响应**: ReentrantLock支持响应中断，即线程在等待锁时可以中断。synchronized不支持
> - lock加锁可以超时lock.tryLock(2, TimeUnit.SECONDS);而synchronized不支持超时,等待锁的过程中加入到阻塞队列.
> - ReentrantLock可以**与多个Condition对象结合**，实现复杂的线程同步机制。synchronized不支持绑定多个条件
> - synchronized通过**互斥锁**保证共享数据不会被访问，ReentrantLock通过**AQS**来实现。

reentrantLock的实现原理,什么是AQS?

> 



CAS的原理

> 是一种乐观锁，用于比较一个变量的当前值是否等于预期值，如果相等，则更新值，否则重试。
>
> 这个比较和替换的操作需要是原子的，不可中断的。Java 中的 CAS 是由 Unsafe 类实现的。

CAS的问题

> ABA 问题、自旋开销大、只能操作一个变量
>
> 使用版本号/时间戳的方式来解决 ABA 问题。
>
> CAS 失败时会不断自旋重试,一直不成功，会给 CPU 带来非常大的执行开销
>
> 涉及到多个变量的原子更新:多个变量封装为一个对象，使用 AtomicReference 进行 CAS 更新。

原子类如何实现的?

> 是基于 CAS + volatile 实现的，底层依赖于 Unsafe 类

比较CAS和AQS?

> 

乐观锁和悲观锁的定义

> 悲观锁认为每次访问共享资源时都会发生冲突，所在在操作前一定要先加锁，防止其他线程修改数据。 如synchronized 和 ReentrantLock
>
> 乐观锁认为冲突不会总是发生，所以在操作前不加锁，而是在更新数据时检查是否有其他线程修改了数据。如果发现数据被修改了，就会重试。 CAS 是最典型的乐观锁实现
>
> 乐观锁发现有线程修改数据:重新读取数据，然后再尝试更新，直到成功为止或达到最大重试次数。
>
> 数据库层面例子:数据库的 `select ... for update` 是悲观锁，先锁住这行数据再操作。而版本号机制是乐观锁，更新时带上版本号，`update ... where version = 1`，如果版本对不上说明被别人改过，更新失败重试



wait/notify/notifyAll的作用

sleep和wait的区别

ReentrantLock的Condition的原理和作用

Condition与wait/notify的区别



ThreadLocal怎么用

实现方式

为什么会出现内存泄漏？



什么是CountDownLatch

什么是CyclicBarrier，和CountDownLatch的区别

什么是信号量Semaphore



为什么要有线程池，线程过多会怎样

> **线程过多会造成系统资源的大量占用**,线程池可以提高线程利用率.线程池可以**复用线程**，**避免频繁地创建和销毁线程**
>
> 线程池对线程统一管理,定义任务队列,拒绝策略,线程生命周期监控.
>
> 线程池可以用于**排队任务**

线程池工作流程

> ![image](./assets/20250917_ExecutorPool_work.jpg)

线程池参数

> 核心线程数（**corePoolSize**）、最大线程数（**maximumPoolSize**）、空闲线程存活时间（**keepAliveTime**）、时间单位（**TimeUnit**）、线程池任务队列（**workQueue**）、线程工厂（**ThreadFactory**）和拒绝策略（**RejectedExecutionHandler**）。
>
> 核心线程是线程池中长期存活的线程数,即便空闲也不会销毁. 设置为0时,新任务添加到队列
>
> **keepAliveTime**:当线程数大于核心线程数时,空闲时间超过定义,则销毁,只保留核心线程数.
>
> 通过**ThreadFactory**设置线程优先级,线程命名,以及线程类型

拒绝策略有哪些

> **CallerRunsPolicy**：使用线程池调用者所在的线程去执行被拒绝的任务
>
> **AbortPolicy**：直接抛出一个任务被线程池拒绝的异常。
>
> **DiscardPolicy**：默默丢弃新任务，不抛异常也不通知
>
> **DiscardOldestPolicy**：抛弃最老的任务，然后执行该任务。
>
> 自定义任务拒绝逻辑,实现 RejectedExecutionHandler 接口,比如把被拒绝的任务持久化到数据库或消息队列中,再补偿处理,确保任务不丢失.

线程池涉及到的方法:

> execute和submit
>
> shutdown(),shutdownNow():
>
> shutdown()使用后会**置状态为SHUTDOWN**,**正在执行的任务会继续**，没有被执行则中断,不再接受新任务.
>
> shutdownNow()使用后会**置状态为STOP**,**正在执行任务的线程会中断**，没有被执行的任务则返回。shutdownNow()终止线程调用的是Thread.interrupt()方法，如果线程中没有sleep,wait,Condition或者定时锁等应用，interrupt()方法是无法中断当前线程的，仍需等待正在执行的任务都执行完成了才能退出。

线程池调优

> 线程池参数设置需要结合任务特性和系统资源综合判断
>
> CPU密集型任务corePoolSize = CPU核心数 + 1 ：减少线程切换开销。
>
> IO密集型任务:corePoolSize = CPU核心数 * 2 

死锁出现的条件

预防死锁

排查死锁的思路



# JVM

**类加载机制**把 .class 文件加载进 JVM → 加载后的数据存放在**内存模型**的各个区域 → 对象在堆中不断创建，内存不够了就触发**垃圾回收** → 当 GC 表现不佳（频繁 Full GC、OOM）时就需要**JVM 调优**来定位和解决问题。

## 类加载机制

回答 类什么时候被加载,加载的流程,谁负责加载,

一个 .java 文件经过编译变成 .class 字节码文件，探讨JVM 怎么把它加载进内存变成可以使用的 Class 对象

类加载的过程?

> 加载 、验证、准备、解析、初始化、使用和卸载。

加载阶段做了什么?

> 找到并读入class文件,通过类的全限定名（比如 `com.example.HelloWorld`）找到对应的 .class 文件,读取二进制流.

加载阶段创建了什么?

> 在方法区创建了数据结构,保存类的元信息和元空间,对应的是类的字段表,方法表,常量池. 但是例如反射无法直接获取方法区的数据,所以还需要在堆中创建一个class对象指向方法区,Class对象为了获取方法区信息

验证为什么有必要?

> 确保class文件是有效的,不会破坏JVM环境,校验字节码格式,语义的合法性,字节码指令是否有效.验证的是方法区里的数据是否合法

下一步是创建类对象,也就是准备,准备阶段做了什么?

> 要创建对应的类,但是不会一次性把类初始化好,而是先给类变量(静态变量)占个位置,但是仅仅是给内存.因为类变量可能依赖其他类`static int count = OtherClass.getValue()`,其他依赖没准备好. 等准备就绪再初始化.

解析做了什么?

> 对于`static int count = OtherClass.getValue()`,这里的otherclass是一个符号引用,而不是直接引用, **解析（Resolution）**——解决的问题是"把符号引用变成直接引用"。 解析替换的也是方法区常量池中的符号引用。

在JDK 7 之后，静态变量的值被移到了堆中的 Class 对象里,把静态变量的值放到堆里，就能被正常的垃圾回收管理。

**初始化**做了什么?

> 类的内存布局完成了,符号对象也变成了直接引用,所以对静态变量的赋值也就可以进行,初始化会执行静态变量赋值以及静态代码块.

上面是类被加载的过程,类什么时候被加载?

> 主动引用的时候:
>
> `new` 一个对象的时候；读取或设置一个类的静态字段的时候（final static 的编译期常量除外，因为它已经内联到调用方了，根本不需要加载原来的类）；调用一个类的静态方法的时候；反射调用（`Class.forName()`）的时候；初始化一个类时发现它的父类还没初始化，会先初始化父类；程序入口的 main 方法所在的类。
>
> "被动引用"不会触发初始化。最经典的考题是：`SubClass.parentStaticField` 通过子类引用父类的静态字段，只会初始化父类，不会初始化子类。定义一个类的数组 `Order[] arr = new Order[10]` 也不会初始化 Order

类被加载,也就会被卸载,什么时候被卸载?

> 这个类所有的实例都已经被回收了，加载这个类的 ClassLoader 也被回收了，这个类对应的 Class 对象没有被任何地方引用。

永久代和元空间有什么区别?

> 它们都是方法区的实现方式，区别在于：永久代（JDK 7 及以前）使用 JVM 堆内存，大小通过 `-XX:MaxPermSize` 设置，容易因为加载的类太多导致 OOM（`java.lang.OutOfMemoryError: PermGen space`）。元空间（JDK 8 起）使用本地内存（Native Memory），默认不设上限（可以通过 `-XX:MaxMetaspaceSize` 限制），理论上只要操作系统还有内存就能用。这个改动的动机还是那个核心问题——永久代大小难以预估，设小了容易 OOM，设大了浪费堆空间。改用本地内存之后这个问题就缓解了。

负责加载类的对象叫做类加载器,为什么要分层?

> java有三层类加载器,jdk核心类和自己写的业务类jvm的信任层级不同,对类加载器分层就是分权,最顶层的**Bootstrap ClassLoader**最可信,用来加载jdk核心类,自己的业务类jvm不信任,所以由**Application ClassLoader**加载. **Extension ClassLoader（扩展类加载器）**——负责加载 JDK 扩展目录下的类。

类加载策略?

> 用父类加载机制,核心类如`java.lang.String` 必须由启动加载器加载, 但用户自己可以创建一样名称的类,出现混淆,为了保证加载的是jdk的类而不是用户的,使用父类加载机制.当任何一个类加载器收到加载请求时，它**不会自己先尝试**，而是先委派给父加载器。父加载器又往上委派，一直到 Bootstrap。只有当父加载器说"我负责的范围内找不到这个类"时，才回到子加载器自己去加载。
>
> 保证同一个类在 JVM 中的唯一性和安全性,同一个类不会重复加载一个类.

## JVM内存区域



# Spring

