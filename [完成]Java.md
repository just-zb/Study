Java
========
# JVM

https://claude.ai/share/560e512b-3e16-450e-8383-7bc1f7ac14c5

**类加载机制**把 .class 文件加载进 JVM → 加载后的数据存放在**内存模型**的各个区域 → 对象在堆中不断创建，内存不够了就触发**垃圾回收** → 当 GC 表现不佳（频繁 Full GC、OOM）时就需要**JVM 调优**来定位和解决问题。

所以问题是: 怎么进来->放在哪->怎么回收->出了问题怎么排查

## 类加载机制

一个 .java 文件经过编译变成 .class 字节码文件，JVM 怎么把它加载进内存变成可以使用的 Class 对象？

加载 、验证、准备、解析、初始化、使用和卸载。

首先要找到并读入class文件,通过类的全限定名（比如 `com.example.HelloWorld`）找到对应的 .class 文件,读取二进制流. 这是**加载**阶段,加载阶段把class文件加载到哪了? 这一阶段在方法区创建了数据结构,保存类的元信息和元空间,对应的是类的字段表,方法表,常量池. 但是例如反射无法直接获取方法区的数据,所以还需要在堆中创建一个class对象指向方法区,相当于一个获取方法区信息的接口对象.

方法区是jvm在运行时发挥作用,如方法调用查方法表,字节码验证查常量池. class对象服务于你的java代码,能够进行反射,类型判断等.

要确保class文件是有效的,所以下一步就是**验证**,校验字节码格式,语义的合法性,字节码指令是否有效.本质上是jvm防止字节码对自己造成破坏. 验证的是方法区里的数据是否合法

那么验证完毕之后,要创建对应的类,但是不会一次性把类初始化好,而是先给类变量(静态变量)占个位置,有一块内存是你的,但是仅仅是给内存,也就是类变量只有默认值,没有进行类中的赋值逻辑,因为类变量可能依赖其他类`static int count = OtherClass.getValue()`,其他依赖没准备好. 等准备就绪再赋初值. **准备**阶段分配内存的静态变量也在方法区

代码中引用一个其他对象,如`static int count = OtherClass.getValue()`,在字节码文件中只是一个符号,一个字符,这个对象没有解析,所以只是字符串而不是对应的对象,**解析（Resolution）**——解决的问题是"把符号引用变成直接引用"。 解析替换的也是方法区常量池中的符号引用。

问题: 常量池存放哪些东西?

在编译产生class文件时就有静态常量池,存放那些:凡是在编译期能确定、且需要被其他地方引用的信息.比如字面量和符号引用

当类被加载后,常量池被加载到方法区，变成运行时常量池**可以在运行期间动态添加新内容**。如`String.intern()` 方法,如果字符串常量池中还没有这个字符串，就会把它放进去。

字符串常量池的特殊之处:概念上属于运行时常量池的一部分,JDK 7 之前它在永久代（方法区的实现）中，JDK 7 起被移到了堆中。因为永久代空间有限且 GC 频率低，字符串又特别多，放在永久代容易导致 OOM.

常量池存的全是描述性信息,要么是固定不变的值（字符串、final 常量），要么是"谁是谁"的名字和签名,如符号引用. 

对于final static,会存放到常量池吗?

判断final static 是否在常量池，而是**编译期能不能确定值**。

基本类型的字面量赋值和字符串字面量赋值放到常量池方法调用,比如 `final static int MAX = 50` 和 `final static String STATUS = "PAID"`，它们的值在编译时就完全确定了，编译器会把它们作为字面量放进 .class 文件的常量池。而且有一个更彻底的优化：如果其他类引用了这些常量，编译器会直接把值内联到调用方的字节码中。比如别的类写了 `int x = Order.MAX_ITEMS`，编译后的字节码里直接就是 `int x = 50`，连 Order 类都不需要加载。这也是为什么改了一个 final static 常量的值后，引用它的类必须重新编译才能生效。

new 对象这些需要运行时才能算出来的就存放在堆中的class对象上.

无论是静态变量还是实例变量,描述信息都在方法区里，因为它们都是类的元信息。在JDK 7 之后，静态变量的值被移到了堆中的 Class 对象里,把静态变量的值放到堆里，就能被正常的垃圾回收管理。



```text
把一个类想象成一家连锁餐厅。
方法区就是这家餐厅的总部档案室。里面存着完整的运营手册：菜单有哪些菜（字段描述）、每道菜怎么做（方法的字节码）、餐厅叫什么名字、归哪个集团管（类名、继承关系）、员工行为规范（访问修饰符）等等。这是 JVM 自己管理用的，顾客（你的 Java 代码）进不去这个档案室。
常量池是档案室里的一个电话簿和价目表。电话簿记录的是"张三的电话是什么"（符号引用——类名、方法名、字段名），价目表记录的是固定不变的值（字面量——"PAID"、50 这些）。它是方法区的一部分，服务于 JVM 在运行时查找"谁是谁"和"固定值是多少"。
堆中的 Class 对象是餐厅的前台接待。顾客（你的 Java 代码）想了解这家餐厅有什么菜、营业时间是什么，不需要闯进档案室自己翻，问前台就行。前台知道档案室在哪（持有指向方法区的指针），能帮你查到你需要的信息（getMethods()、getFields()），还能帮你办事（newInstance()）。同时前台桌上还放着这家餐厅的公共物品（静态变量的值，JDK 7+ 之后挂在 Class 对象上）。
用这个比喻回答几个容易混的问题
"字段描述"和"字段的值"为什么分开存？——菜单上写着"宫保鸡丁，售价 38 元"（字段描述，在方法区），但每桌客人点的那盘具体的宫保鸡丁是各自独立的（实例变量的值，在堆中的对象实例里）。菜单只需要一份，但菜要按桌上。
常量池和 Class 对象有什么区别？——电话簿（常量池）是给 JVM 内部查名字和固定值用的，前台（Class 对象）是给你的 Java 代码用的。你的代码永远不会直接去翻常量池，但你会通过反射去调用 Class 对象的方法。
静态变量的值为什么不在方法区？——JDK 7 之前确实在方法区（相当于公共物品放在档案室里）。后来发现档案室空间太小、清理也不方便，就把公共物品搬到了前台桌上（堆中的 Class 对象），这样更容易管理和回收。
```



**初始化**:那么类的内存布局完成了,符号对象也变成了直接引用,那么对静态变量的赋值也就可以进行, 就可以执行静态变量赋值以及静态代码块了.没有创建类对象所以能进行的操作必须只和类相关.

**读进来 → 检查是否安全 → 先占位给零值 → 符号变地址 → 执行真正的初始化代码**。

上面是类加载的过程,是把类从字节码变成jvm内存区域的内存对象的过程, **但类什么时候会被加载呢**?

主动引用的时候:

`new` 一个对象的时候；读取或设置一个类的静态字段的时候（final static 的编译期常量除外，因为它已经内联到调用方了，根本不需要加载原来的类）；调用一个类的静态方法的时候；反射调用（`Class.forName()`）的时候；初始化一个类时发现它的父类还没初始化，会先初始化父类；程序入口的 main 方法所在的类。

"被动引用"不会触发初始化。最经典的考题是：`SubClass.parentStaticField` 通过子类引用父类的静态字段，只会初始化父类，不会初始化子类。定义一个类的数组 `Order[] arr = new Order[10]` 也不会初始化 Order



类什么时候被**卸载**: 这个类所有的实例都已经被回收了，加载这个类的 ClassLoader 也被回收了，这个类对应的 Class 对象没有被任何地方引用。

面试中经常问"永久代和元空间有什么区别"。它们都是方法区的实现方式，区别在于：永久代（JDK 7 及以前）使用 JVM 堆内存，大小通过 `-XX:MaxPermSize` 设置，容易因为加载的类太多导致 OOM（`java.lang.OutOfMemoryError: PermGen space`）。元空间（JDK 8 起）使用本地内存（Native Memory），默认不设上限（可以通过 `-XX:MaxMetaspaceSize` 限制），理论上只要操作系统还有内存就能用。这个改动的动机还是那个核心问题——永久代大小难以预估，设小了容易 OOM，设大了浪费堆空间。改用本地内存之后这个问题就缓解了。





有问题: 谁负责加载类? 先加载哪些类?

类加载器负责加载类对象,java有三层类加载器,为什么不是一个类加载器处理所有?

因为jdk核心类和自己写的业务类jvm的信任层级不同,对类加载器分层就是分权,最顶层的Bootstrap ClassLoader最可信,用来加载jdk核心类,自己的业务类jvm不信任,所以由Application ClassLoader加载. **Extension ClassLoader（扩展类加载器）**——负责加载 JDK 扩展目录下的类。

核心类如`java.lang.String` 必须由启动加载器加载, 但用户自己可以创建一样名称的类,出现混淆,为了保证加载的是jdk的类而不是用户的,使用双亲委派模型.当任何一个类加载器收到加载请求时，它**不会自己先尝试**，而是先委派给父加载器。父加载器又往上委派，一直到 Bootstrap。只有当父加载器说"我负责的范围内找不到这个类"时，才回到子加载器自己去加载。

解决了同一个类在 JVM 中的唯一性和安全性, 同一个类加载器不会重复加载一个类,所以自己写的`java.lang.String`会被bootstrap加载器丢弃. 但这样出现热部署的问题,如果一个累的实现要进行替换,必须要用新的类加载器了.



## JVM内存区域



类加载完成后,jvm会对内存进行分区,按照生命周期和共享性分区.

首先,创建线程,从main方法开始执行,现成的数据是私有的. 

为了记录线程执行到哪了,用程序计数器,它不会oom因为程序计数器只存储一个字节码地址. 

虚拟机栈:每个线程有自己的虚拟机栈，每调用一个方法，就会在栈顶压入一个**栈帧（Stack Frame）**方法返回时弹出, 栈帧里的内容和类加载阶段的知识直接相关.

- 一个栈帧有什么?
  - 局部变量存放在**局部变量表**, 包括方法中的局部变量和方法参数. 如果是实例方法，第 0 个 slot 固定是 `this` 引用。基本类型直接存值，引用类型存的是指向堆中对象的指针
  - **操作数栈**——方法执行过程中的临时计算空间, 存放计算过程中的中间结果
  - **动态链接**——这里就和类加载的解析阶段接上了. 解析阶段把符号引用变成直接引用，但并不是所有符号引用都在类加载时就解析了。有些方法调用（比如多态场景下，`animal.speak()` 到底调用 Dog 的还是 Cat 的 speak）要到运行时才能确定。栈帧中的动态链接就是持有一个指向运行时常量池的引用，在运行时把还没解析的符号引用转化为直接引用. 说白了就是把因为多态等原因没有解析的符号引用变成直接引用,不过是动态的.类加载时的解析是"静态解析"，这里是"动态链接"，两者是同一件事在不同时间点的完成
  - **方法出口**——记录方法返回后应该回到调用方的哪个位置继续执行。方法正常 return 或者抛异常，都需要知道回到哪里。
- 本地方法栈: 和虚拟机栈几乎一样，区别只是它服务的是 native 方法
- 虚拟机栈会抛出什么异常?
  - 最常见stackoverflow,无限递归导致超过栈深度限制
  - 如果栈尝试动态扩展但内存不够，抛 `OutOfMemoryError`



共享性: 堆是 JVM 管理的最大一块内存,由所有线程共享.

**创建堆对象的过程**:

- 当你执行 `new Order()` 时，JVM检查new指令的参数在常量池能否定位到类的符号引用,然后检查是否加载这个类,如果没有就类加载. 在堆中找一块足够大的空间，按照方法区中 Order 类的元信息（"图纸"）来分配内存——需要多少字节存 orderId 的引用、多少字节存 amount 的值，这些都是从方法区查到的。

- 局部变量表只负责"方法作用域内的变量"，对象与对象之间的引用关系全在堆内部通过实例字段维护。GC做可达性分析的时候，就是从GC Roots（包括栈帧中的局部变量表引用）出发，沿着堆中这些对象间的引用链往下找。

- 在对分配对象实例的内存后,所有实例变量都为零值,然后 设置对象头（记录这个对象属于哪个类、GC 年龄等信息）→ 执行构造方法 `<init>` ,最后才是把虚拟机栈中的局部变量表指向堆中的对象

- 继承关系中,类加载和实例化时,对应的执行顺序. **静态变量赋值和静态代码块之间**、**实例变量赋值和代码块之间**，谁先谁后取决于它们在源码中的书写顺序. 静态变量赋值和静态代码块会被编译器收集到 `<clinit>` 方法中，实例变量赋值和代码块会被编译器收集到 `<init>` 方法中

  - 当你写子类构造方法时，Java 要求第一件事必须是调用父类的构造方法。你可以显式写 `super()`，也可以不写——不写的话编译器会自动帮你加上一个无参的 `super()`。保证父类先初始化.

  - 编译器会把实例变量赋值和代码块"搬"到构造方法里，所以执行顺序是: super()（父类全部初始化完）→ 实例变量和代码块 → 你自己写的构造逻辑。生成的 `<init>` 方法实际上等价于：

    ```java
    Child() {
        super();                            // 1. 先调父类构造
        this.name = "bao";                  // 2. 实例变量赋值（编译器插入）
        System.out.println("代码块");        // 3. 代码块（编译器插入）
        System.out.println("构造逻辑");      // 4. 你写的构造逻辑
    }
    ```

    

  ```java
  class Parent {
      static String s1 = "父静态变量";       // 1
      static { System.out.println("父静态块"); } // 2
  
      String i1 = "父实例变量";              // 5（new的时候）
      { System.out.println("父代码块"); }     // 6
  
      Parent() { System.out.println("父构造"); } // 7
  }
  
  class Child extends Parent {
      static String s2 = "子静态变量";       // 3
      static { System.out.println("子静态块"); } // 4
  
      String i2 = "子实例变量";              // 8
      { System.out.println("子代码块"); }     // 9
  
      Child() { System.out.println("子构造"); } // 10
  }
  ```

  

为什么堆要分代?

**绝大多数对象都是"朝生夕灭"的**。比如方法里创建的临时对象，方法一结束就没用了。只有少部分对象会长期存活.

所以把堆分成两个区域:

- **新生代（Young Generation）**——存放刚创建的对象。因为大部分对象很快就死了，所以这个区域回收频率高，但每次回收很快
- **老年代（Old Generation）**——存放长期存活的对象。空间比新生代大（默认比例大约新生代:老年代 = 1:2），GC 频率低，但每次 GC 耗时更长，因为存活对象多，不能像新生代那样简单复制。

### 新生代

新生代还要进行分区,因为新生代用的是**复制算法**——把存活对象从一块区域复制到另一块，然后把原来的区域整体清空。这个算法需要一块"备用空间"来接收存活对象。如果把新生代对半分（50% 用来放对象，50% 留作备用），空间利用率太低。但实际上每次 GC 后存活的对象只有很少一部分（可能不到 10%），所以只留 10% 的空间（一个 Survivor 区）作为备用就够了。两个 Survivor 区是交替使用的：这次 GC 把存活对象从 Eden + S0 复制到 S1，下次 GC 再从 Eden + S1 复制到 S0。

所以有了elden区,和**两个 Survivor 区（S0、S1）**，默认比例 8:1:1。

**对象的流转路径**：新对象在 Eden 分配 → Eden 满了触发 Minor GC → 存活的对象复制到其中一个 Survivor 区 → 每熬过一次 Minor GC 对象的年龄加 1 → 年龄达到阈值（默认 15）就晋升到老年代。

例外情况:如果一个对象特别大（比如一个很大的数组），新生代放不下，JVM 会直接把它分配到老年代，跳过新生代。



### 老年代

因为老年代存活率高,所以如果用复制算法则会浪费内存空间,转而使用标记清除或者标记整理,都是在原地进行操作.老年代满了会触发 Major GC 或 Full GC。

但是G1 收集器打破了这个传统设计.



堆和元空间和虚拟机栈, 都会出现OOM, 堆中不断创建无法被回收的对象,元空间通过反射,CGLIB,动态代理等加载了过多的类. 以及虚拟机栈的stackoverflow,和过多线程,每个线程都需要分配空间.

## 垃圾回收

垃圾回收处理清理老年代对象的问题,需要确定哪些对象该回收,用什么算法回收以及用什么收集器.

判断对象是否应该回收:

- 引用计数法,但出现循环引用
- 可达性分析:从一组叫做 **GC Roots** 的起点出发，沿着引用链往下走，能走到的对象就是存活的，走不到的就是垃圾。解决了循环引用的问题
  - 选择GC roots, 一定是当前确定还在被使用的入口点. 如局部变量表的引用,类的静态变量引用和常量的引用,被 synchronized 锁持有的对象
  - 可达性分析是JVM的具体做法,不会一次就把不可达对象标记为删除,而是第一次不可达时,如果重写了finalize方法,放到队列中去执行该方法,如果该方法重新让对象变得可达,就不会删除. 如果第二次不可达,就会删除.

对象的四种引用类型:

- 强引用 不会被回收
- 弱引用:用 `WeakReference` 包装。不管内存够不够，下次 GC 一定会回收。`WeakHashMap` 就是利用这个特性，key 被回收后 entry 自动移除
- 软引用:用 `SoftReference` 包装。内存够的时候不回收，内存不够即将 OOM 的时候才回收。
- 虚引用:用 `PhantomReference` 包装。完全不影响对象的生命周期，甚至无法通过虚引用获取对象实例。唯一的作用是跟踪对象被 GC 回收的时机,在对象被回收时收到一个通知，用于做回收后的清理工作（比如管理堆外内存）。 必须配合 ReferenceQueue 使用：

  ```java
  Object obj = new Object();
  ReferenceQueue<Object> queue = new ReferenceQueue<>();
  PhantomReference<Object> phantomRef = new PhantomReference<>(obj, queue);
  
  obj = null;  // 去掉强引用
  System.gc();
  
  // GC 回收 obj 之后，phantomRef 会被放入 queue
  Reference<?> ref = queue.poll();
  if (ref != null) {
      // 说明对象已经被回收了，可以做清理工作
  }
  ```

  

引用的类型决定了对象什么时候被回收.

### 回收算法

知道新生代用的是标记复制算法,老年代用不同的算法. 最简单的是标记清除法,清除之后内存变得不连续,让大对象无法储存.

为了解决内存碎片问题,引入标记整理算法.把所有标记的对象移动到内存一端,清理之外的空间,但是移动对象开销大,并且因为对象地址的改变所以需要更新所有对象的引用,要暂停线程.

**复制算法（Copying）**——把内存分成两块，每次只用其中一块，GC 时把存活对象复制到另一块，然后把原来那块整体清空,代价是只能里用一半内存. 

新生代用复制算法,每次 GC 只需要复制少量存活对象. 并且配置elden和survivor区的比例以提高内存使用效率.把空间浪费从 50% 降到了只有 10%。

老年代对象存活率高，如果用复制算法要复制大量对象，开销太大。所以用**标记-清除**或**标记-整理**。标记-整理虽然慢一些，但没有碎片，长远来看更稳定。

三种基础算法都需要 STW Stop The World,停止线程,**标记-清除**——标记阶段要停.



### 收集器

新生代用复制算法,Serial是单线程收集器,GC 时必须暂停所有业务线程

**ParNew**——Serial 的多线程版本，用多个 GC 线程并行收集,能和CMS配合使用

**Parallel Scavenge**——也是多线程并行收集，但它的目标是**最大化吞吐量**（而不是最小化停顿时间）。提供了 `-XX:MaxGCPauseMillis`（控制最大停顿时间）和 `-XX:GCTimeRatio`（控制吞吐量目标）两个参数。适合后台计算型任务，不需要快速响应用户。



老年代收机器:

**Serial Old**——Serial 的老年代版本，单线程，标记-整理算法。

**Parallel Old**——Parallel Scavenge 的老年代版本，多线程，标记-整理。和 Parallel Scavenge 搭配使用，追求整体吞吐量最大化。

吞吐量就是 **CPU 用于执行用户代码的时间占总时间的比例**。

**吞吐量优先 vs 低延迟优先**是两种不同的取舍：

- **Parallel Scavenge** 追求高吞吐量——GC 总时间少，适合后台计算类任务（比如批处理、数据分析），用户感知不到停顿没关系，整体算得快就行
- **CMS / G1** 追求低延迟——每次 GC 停顿时间短，适合交互式应用（比如 Web 服务），用户不能等太久

**CMS（Concurrent Mark Sweep）**——第一个真正意义上追求**低停顿时间**的收集器。它用标记-清除算法，GC 过程分四步：初始标记（STW，很快，只标记 GC Roots 直接引用的对象,也就是只有一层）→ 并发标记（和业务线程同时运行，遍历整个引用链,也就是深度搜索）→ 最终标记（STW，修正并发标记期间因业务线程继续运行导致的引用变化）→ 并发清除（和业务线程同时运行，清除垃圾对象）

CMS 大部分工作和业务线程并发进行，STW 时间很短,但是用标记清除产生内存碎片,并发占用cpu资源; 并发清除阶段业务线程还在创建新对象，如果老年代空间不够了会触发"Concurrent Mode Failure"，退化为 Serial Old 来做一次全量 GC，这时候停顿就很长了。



CMS 在 JDK 9 被标记为废弃，JDK 14 被移除，被 G1 取代。

G1指的是garbage first. 它的设计目标是**在可控的停顿时间内，获得尽可能高的吞吐量**。

打破了固定的分代物理布局,它把整个堆划分成大量等大的 **Region**每个 Region 可以动态地充当 Eden、Survivor、Old 或 Humongous（存放大对象，超过 Region 大小 50% 的对象）

从要么回收新生代要么老年代变为回收高价值region,G1 可以用 `-XX:MaxGCPauseMillis`（默认 200ms）来设定一个停顿时间目标，然后在这个时间预算内尽可能多地回收垃圾。

回收过程: Young GC（回收所有 Eden 和 Survivor Region,在eden region满了之后触发）过程是 STW 的→ 

并发标记. 当老年代占用达到阈值（默认 45%）时触发，和 CMS 类似,包括初始标记（STW，搭在 Young GC 上顺便做）→ 并发标记 → 最终标记（STW）→ 筛选回收（统计每个 Region 的回收价值. 为了搞清楚老年代中各个 Region 的存活情况,目的不是回收，而是**摸底**,找出老年代的垃圾多,回收价值高→ 

Mixed GC并发标记结束后触发,（混合回收，既回收新生代也选择性回收高价值部分老年代 Region,根据你设定的停顿时间目标）。如果 Mixed GC 跟不上对象分配速度，会退化为 Full GC（单线程，很慢，要尽量避免）。

优势:不会产生内存碎片（Region 间用复制算法，Region 内整体回收）；可预测的停顿时间；不需要配合其他收集器使用，自己管理整个堆。

三个概念,这三个一定会STW, JVM 需要保证对象引用关系不变的时候

**Minor GC（Young GC）**——只回收新生代。触发条件是 Eden 区满了。频率高，但速度快。

**Major GC**——一般指只回收老年代。是 CMS 的特有行为。

**Full GC**——回收整个堆（新生代 + 老年代）和方法区。触发条件包括：老年代空间不足、元空间不足、调用了 `System.gc()`（只是建议，JVM 可以忽略）、CMS 的 Concurrent Mode Failure 等。Full GC 是最慢的，也是调优时最想要避免的。会从 GC Root 出发，标记所有可达对象。新生代使用复制算法，清空 Eden 区。老年代使用标记-整理算法，回收对象并消除碎片。



G1每次只回收一部分老年代 Region，把一次大的停顿拆成了多次小的停顿，并且可以通过停顿时间目标来控制每次停顿的上限。这就是"可预测的停顿时间"的实现原理。



对比G1和CMS,G1的停顿时间是可预测的,并且对内存进行压缩,但CMS能够最大限度减少应用暂停时间



## jvm调优

默认参数不适合业务,可能full GC频繁,也可能OOM,

先用工具查看jvm状态,先找到问题.然后定位原因,最后调整参数. 

### 查看jvm状态

有命令行工具和可视化工具.

**jps**——查看当前机器上有哪些 Java 进程和它们的 PID

**jstat**——查看 GC 统计信息，是判断 GC 是否健康的核心工具。

jmap

jstack

jinfo 查看和动态修改JVM参数

### FullGC频繁-应用响应变慢

先用jstat检查是full gc还是young gc,如果是full gc次数多占用时间长,则1. 可能是内存泄漏 2. 对象晋升过早

内存泄漏的原因,什么时候会内存泄漏?

内存泄漏的本质是：**对象已经没用了，但还有引用链连着 GC Roots，导致 GC 认为它是活的，无法回收。**

**ThreadLocal 使用不当**——ThreadLocal 的值存在线程的 ThreadLocalMap 中，如果线程是线程池中的长期存活线程，而你用完 ThreadLocal 后没有调 `remove()`，值对象就会随着线程一直存活，无法被回收。

**定位方法**：用 jmap 导出 heap dump，用 MAT 分析哪些对象占了老年代的大部分空间，找到它们的 GC Root 引用链，就能看出是谁在"抓着不放"。

如果是内存泄漏就修代码（比如集合忘记移除元素、静态变量持有了不该持有的引用）。如果是过早晋升就调参数——增大新生代比例（`-Xmn` 或 `-XX:NewRatio`）、提高晋升年龄阈值（`-XX:MaxTenuringThreshold`);

### OOM

堆,虚拟机栈,元空间都会OOM

堆满了,那么可能是内存泄漏或者堆设置太小.`-XX:+HeapDumpOnOutOfMemoryError` 参数，JVM 会在 OOM 时自动导出 heap dump，事后用 MAT 分析。

元空间满了，加载的类太多,因为使用动态代理或者反射生成类,或者热部署中的类加载器没有回收, 用 `-XX:MaxMetaspaceSize` 设置上限，同时排查是否有类加载器泄漏。

虚拟机栈溢出,一般是无限递归,如果是正常调用要调整栈大小,可以用 `-Xss` 增大每个线程的栈大小. 看 jstack 的调用栈就能定位





### GC停顿时间过长- 应用偶尔出现几百毫秒甚至几秒的卡顿

先开启 GC 日志（`-Xlog:gc*` 或 `-XX:+PrintGCDetails`），看停顿发生在哪个阶段。如果是 Young GC 慢，可能是新生代太大，存活对象太多，复制开销大。如果是 Mixed GC 或 Full GC 慢，考虑是不是老年代的回收策略不合适。

如果用的是 CMS，考虑换 G1。如果已经是 G1，调整 `-XX:MaxGCPauseMillis` 来控制目标停顿时间（但设太小会导致每次回收的 Region 太少，GC 频率升高，反而可能更快触发 Full GC）。如果堆很大（超过 16GB），考虑 ZGC。



核心参数:

`-X` 开头的是 JVM 的非标准参数（但主流 JVM 都支持），`-XX` 开头的是实验性或高级参数。

**-Xms**——`m` 是 memory，`s` 是 start。memory start，堆内存的起始大小。

**-Xmx**——`m` 是 memory，`x` 是 maximum。memory maximum，堆内存的最大值。

**-Xmn**——`m` 是 memory，`n` 是 new（新生代）。memory new，新生代的大小。

### 栈相关

**-Xss**——`s` 是 stack，第二个 `s` 是 size。stack size，每个线程的栈大小。

### 元空间相关

**-XX:MetaspaceSize** 和 **-XX:MaxMetaspaceSize**——这两个是 `-XX` 系列的，命名就直白多了，直接用完整单词，不需要猜缩写。MetaspaceSize 是触发 GC 的初始阈值，MaxMetaspaceSize 是上限。

元空间的回收是搭在 **Full GC** 上顺便做的。当元空间使用量达到 `MetaspaceSize` 阈值时，JVM 会触发一次 Full GC，在这次 Full GC 过程中检查有没有满足卸载条件的类，有就卸载并释放对应的元空间内存。如果释放之后空间还是不够，JVM 就扩容元空间；如果已经达到了 `MaxMetaspaceSize` 的上限且释放不出空间，就抛 OOM。

所以设置 `-XX:MetaspaceSize` **避免扩容过程中反复触发不必要的 Full GC 拖累堆的正常运行**。

### G1 相关

**-XX:MaxGCPauseMillis**——Max GC Pause in Milliseconds，最大 GC 停顿毫秒数。完整的英文句子，不需要记缩写。

**-XX:InitiatingHeapOccupancyPercent**——Initiating（触发）Heap Occupancy（占用率）Percent（百分比），堆占用率达到多少时触发并发标记。简称 IHOP。

**收集器选择**：`-XX:+UseG1GC`（使用 G1）、`-XX:+UseZGC`（使用 ZGC）。

### 诊断相关

**-XX:+HeapDumpOnOutOfMemoryError**——名字就是一句话：在 OOM 时做堆转储。

**-XX:HeapDumpPath**——堆转储文件的路径。

`-Xlog:gc*`（GC 日志） 告诉 JVM 把所有 GC 相关的事件都打印成日志。GC 日志给你的是**每一次 GC 的详细过程**——每次 GC 什么时候发生、为什么触发、每个阶段花了多久、回收了多少内存。它适合事后分析"某次停顿为什么特别长"或者"为什么突然触发了 Full GC"。





# Spring

## Spring学习

谈到Spring,先想到最重要的特性IOC和AOP

说一下什么是IOC? IOC与DI的关系?

控制反转,这里的“控制”指的是对象创建和依赖关系管理的控制权。是Spring中把对象创建和依赖关系的控制权交给容器,依赖注入是Spring实现IOC的手段,DI有字段注入,构造方法注入和setter注入.

IOC是如何实现的?

1. 加载beandefinition, 扫描包下的配置类,类的元信息变为BeanDefinition 对象。
2. 建立bean工厂,创建一个 DefaultListableBeanFactory 作为 Bean 工厂,用工厂负责创建和管理bean
3. 第三步是bean生命周期中的实例化和初始化

IOC是一个容器,内部包含各种bean,工厂里各种生产线，在 Spring 中就是各种 BeanPostProcessor。比如 `AutowiredAnnotationBeanPostProcessor` 专门负责处理 `@Autowired` 注解。

有各种缓存机制用来存放产品，比如说 singletonObjects 是成品仓库，存放完工的单例 Bean；earlySingletonObjects 是半成品仓库，用来解决循环依赖问题,这就是为什么单例bean用到三级缓存. 而bean的创建利用BeanFactory, 

BeanFactory与ApplicationContext类似,所以可以进行比较,它们的区别?

BeanFactory 算是 Spring 的“心脏”，而 ApplicantContext 可以说是 Spring 的完整“身躯”。BeanFactory 提供了最基本的 IoC 能力。它就像是一个 Bean 工厂，负责 Bean 的创建和管理。他采用的是懒加载的方式，也就是说只有当我们真正去获取某个 Bean 的时候，它才会去创建这个 Bean。

ApplicationContext 是 BeanFactory 的子接口，在 BeanFactory 的基础上扩展了很多企业级的功能。它不仅包含了 BeanFactory 的所有功能，还提供了国际化支持、事件发布机制、AOP、JDBC、ORM 框架集成等等。ApplicationContext 采用的是饿加载的方式，容器启动的时候就会把所有的单例 Bean 都创建好,所以性能好.对于生命周期,ApplicationContext 会自动调用 Bean 的初始化和销毁方法，而 BeanFactory 需要我们手动管理。

IOC在项目启动后做了什么?

1. 扫描和注册bean.这些类的元信息包装成 BeanDefinition 对象
2. Bean 的实例化和注入.IoC 容器会按照依赖关系的顺序开始创建 Bean 实例。



IOC的功能就是这样,除了bean之外IOC的功能还是比较简单的.

下面讲**aop**,它的作用是把一些业务逻辑中的相同代码抽取到一个独立的模块中,主要分为aop是什么,怎么实现的,用什么实现,aop应用场景

首先aop借鉴AspectJ的思想,有哪些基础概念?

AspectJ 是一个 AOP 框架,可以做到编译时、编译后和类加载时织入切面。AspectJ 可以拦截任何 Java 对象的方法调用.相比之下Spring AOP有很多限制.

一些AOP的术语:

**切面（Aspect）**：用`@Aspect`标识的类，是横切关注点的具体实现，把"在哪里拦截"和"执行什么逻辑"组织在一起。

**横切关注点（Cross-cutting Concern）**：散布在多个模块中的通用需求，比如日志、权限、事务。它是一个抽象概念，切面是它的代码实现。

**连接点（JoinPoint）**：程序执行中可以被拦截的点。Spring AOP中只支持方法执行。

**切点（Pointcut）**：用表达式定义"拦截哪些连接点"，相当于对连接点的筛选条件。比如`execution(* com.example.service.*.*(..))`匹配Service层所有方法。

**通知（Advice）**：切面中具体要执行的逻辑，按执行时机分五种：`@Before`（方法执行前）、`@After`（方法执行后，无论是否异常）、`@AfterReturning`（正常返回后）、`@AfterThrowing`（抛异常后）、`@Around`（环绕，最强大，可以控制是否执行目标方法）。

**目标对象（Target Object）**：被代理的原始对象，也就是真正的业务逻辑所在的那个bean。

**代理（Proxy）**：Spring AOP为目标对象生成的代理对象，负责在方法调用时插入通知逻辑。实现接口的用JDK动态代理，没实现接口的用CGLIB。

**织入（Weaving）**：把切面逻辑应用到目标对象的过程。Spring AOP在运行时通过代理织入，而AspectJ可以在编译期或类加载期织入。

Spring AOP 有哪些限制?

Spring 只支持方法类型的连接点,连接点就是被拦截的方法,而且只能拦截Bean对象中的方法.

```java
@Aspect
@Component
public class LoggingAspect {
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        log.info("开始执行方法: " + joinPoint.getSignature().getName());
    }
    @AfterReturning("execution(* com.example.service.*.*(..))")
    public void logAfterReturning(JoinPoint joinPoint) {
        log.info("方法执行成功: " + joinPoint.getSignature().getName());
    }
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))",
                   throwing = "ex")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable ex) {
        log.error("方法执行失败: " + joinPoint.getSignature().getName(), ex);
    }
}
```

Spring AOP的通知有哪些类型?

`@Before`是通知的一种类型,最常用`@Around`,下图展示不同的通知类型,

`@before`记录日志,`@After`只能做一些收尾工作.`@AfterReturning`可以获取方法返回值,做一些基于返回结果的处理.`@AfterThrowing`做异常处理和记录,不能处理异常,继续向上抛出.

环绕通知`@Around`可以控制方法是否执行,以及修改方法参数和返回值.必须接收一个 `ProceedingJoinPoint` 参数，通过调用其 `proceed()` 方法来执行目标方法。

![三分恶面渣逆袭：Spring AOP 通知方式](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/spring-320fa34f-6620-419c-b17a-4f516a83caeb.png)

有哪些方式可以实现织入?

`AspectJ`在编译期间织入字节码,性能好. 包括类加载期织入,它是在 `JVM` 加载 `class` 文件的时候进行织入,在类被加载到 `JVM` 之前修改字节码。运行时织入是我们在 `Spring` 中最常见,但是有性能开销,不需要特殊的编译器和jvm设置.

`Spring AOP`是如何实现织入的?

`Spring AOP` 依赖运行时织入,通过动态代理来实现,在目标对象上织入切面逻辑。所以所谓的织入,就是把切面逻辑放入目标类

`Spring AOP` 如何选择动态代理的方式?

如果目标类实现了接口，就用 `JDK` 动态代理；如果没有实现接口，就用 `CGLIB` 来创建子类代理

就像IOC一样,AOP也在项目启动后发挥功能,它是如何发挥功能的? 

在 Bean 的初始化阶段发生的,在bean的生命周期中的BPP阶段.在 Bean 实例化完成、属性注入完成之后，Spring 会调用所有 BeanPostProcessor 的 postProcessAfterInitialization 方法，AOP 代理的创建就是在这个阶段完成的。

下面说`aop`的应用场景有哪些? 

事务管理是用得最多的场景,日志记录,权限控制、性能监控,或者缓存管理

`Spring AOP`有一定的局限性,相比于另外一个强大工具AspectJ,它哪一些特点?

只支持方法级别,只拦截Spring的Bean. 但是AspectJ无所不能.AOP基于动态代理:JDK或者Cglib,AspectJ 是通过字节码织入.

只有弄明白动态代理JDK或者Cglib,才能搞清楚AOP,他们的区别?

JDK动态代理通过接口来创建代理,通过反射机制在运行时动态创建一个实现了指定接口的代理类,jdk动态代理是java原生,Service层一般实现接口,一般首选jdk; cglib通过继承目标类来创建代理.是一个第三方的字节码生成库,过 ASM 字节码框架动态生成目标类的子类，然后重写父类的方法来插入切面逻辑。Controller层不实现接口,会用cglib

在 Spring Boot 2.0 之后，Spring AOP 默认什么代理?

默认使用 CGLIB 代理,避免出现忘记实现接口导致aop不生效

`@EnableAspectJAutoProxy(proxyTargetClass = true, exposeProxy = true)` proxyTargetClass为true时,就会强制使用**CGLIB代理**

跟AOP密切相关的是Spring的事务,为什么?

Spring的事务通过AOP实现

spring如何管理事务?

通过编程式事务和声明式事务,声明式事务只需要加入`@Transactional`注解,事务的生命周期会自动处理.在方法执行前开启事务，方法正常返回时提交事务，如果方法抛出异常就回滚事务。

Spring事务的缺点:

由于aop只支持方法,所以无法做到更细粒度.

很明显,要明白`@Transactional`是如何发挥作用的?

spring在启动时扫描有该注解方法的bean,生成包裹了事务处理逻辑的代理bean. 这是第一阶段生成代理bean. 第二阶段是调用方法时,实际调用的是代理bean的方法, 在执行方法逻辑之前,**事务拦截器**根据注解参数获取事务属性,如传播行为、隔离级别.并通过**事务管理器**开启事务.从数据库连接池获取一个连接，关闭其自动提交。接着，代理对象会调用原始 Bean 实例中真正的业务方法,顺利返回则(事务管理器)提交事务;抛出异常被拦截器拦截,让事务管理器回滚事务. 结束后事务管理器释放数据库连接.

`@Transactional`注解不能保证正常,要遵循一些原则?

1 它跟aop紧密相连,所以aop失效的情况该注解也失效. 如 无法代理 private 方法; 对于`protected`方法,虽然cglib支持,但是在jdk动态代理情况下,它只能代理接口中的方法,所以protected无法生效. 2 一定要调用代理bean的方法事务才能生效,调用原对象的方法事务失效. 所以在同一个类中有AB两个方法,A有事务约束,B没有,B直接调用A方法会让事务失效. 3 只有在方法抛出异常才会回滚事务,所以如果异常在方法内被trycatch处理则无法回滚. 4 Spring 事务默认只对 RuntimeException 和 Error 类型的异常进行回滚,如果是编译型异常Checked Exception,需要通过 `@Transactional(rollbackFor = Exception.class)` 指定事务回归的异常类型,否则无法回滚

mysql事务有隔离级别,为什么spring 的事务也有隔离级别? 

有五个对应的隔离级别,DEFAULT 表示使用底层数据库的默认隔离级别,其他四个分别对应mysql的四个隔离级别,实现方式为:

`@Transactional(isolation = Isolation.READ_UNCOMMITTED)`

spring中方法会调用方法,事务方法A也会调用另外一个事务方法B,所以涉及到事务传播机制,有哪些?

很容易理解的是,B直接加入A的事务,这是一种传播机制, 其他机制还有:

![三分恶面渣逆袭：事务传播机制](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/spring-a6e2a8dc-9771-4d8b-9d91-76ddee98af1a.png)

需要对这些事务传播机制理解. 面试中最核心的是前三个（REQUIRED、REQUIRES_NEW、NESTED），尤其要能说清楚REQUIRES_NEW和NESTED的区别。

REQUIRES_NEW是创建一个事务,并且完全独立,外层回滚也不会影响,比如记录日志. NESTED是外层事务的"子事务"，命运上受外层控制；

require_new为什么要把原事务挂起?

因为数据库连接和事务是绑定的，一个连接同一时间只能有一个活跃事务。REQUIRES_NEW要开一个独立事务，就需要从连接池拿一个**新的数据库连接**。原事务的连接还占着资源，但又不能继续执行（要等新事务完成后才能继续后续逻辑），所以把它标记为"挂起"，本质上就是暂时搁置，等新事务提交或回滚后再恢复原事务继续执行。

事务传播在子线程生效吗?

事务传播不会在新线程中生效,事务传播机制是通过 [ThreadLocal](https://javabetter.cn/thread/ThreadLocal.html) 实现的



说完了AOP与IOC的基本原理,下面说bean

其实IOC最核心的功能是管理Bean,所以对Bean的了解非常重要. 跟Bean相关的有: 如何创建Bean,bean生命周期

Bean 本质上就是由 Spring 容器管理的 Java 对象,不需要通过new手动创建. 如何定义一个bean?  

1 `Service`、`Dao`、`Controller` 这些都是 Bean,spring自动创建他们的实例. 这是从注解创建,在这种情况,Spring 默认通过无参构造器来创建实例的,如果类只有一个有参构造方法，Spring 会自动进行构造方法注入:

```java
@Service
public class UserService {
    private UserDao userDao;
    
    public UserService(UserDao userDao) {  // 构造方法注入
        this.userDao = userDao;
    }
}
```



2 还可以用xml以及配置类创建bean.

3 用配置类创建bean:

```java
@Configuration
public class AppConfig {
    
    @Bean
    @Primary  // 主要候选者
    public DataSource primaryDataSource() {
        return new HikariDataSource();
    }
    
    @Bean
    @Qualifier("secondary")
    public DataSource secondaryDataSource() {
        return new BasicDataSource();
    }
}

@Autowired
private DataSource dataSource; // 会注入 primaryDataSource（因为有 @Primary）

@Autowired
@Qualifier("secondary")
private DataSource secondaryDataSource;
```

这里用到`@Bean`注解,与`@Component`不同,它作用在方法上,标记方法返回的对象为Bean,方法返回对象是手动创建的. 

用配置类创建bean,如果方法之间存在依赖关系会怎样?

`@Configuration`的定位就是**专门用来定义bean的配置类**，Spring认为在这种类里，`@Bean`方法之间互相调用是一个很常见的场景，所以用CGLIB代理来保证单例语义，这是合理的开销。`@Component`的定位是**普通的业务组件**，里面虽然也可以写`@Bean`方法（叫lite模式），但Spring认为这种类的主要职责不是定义bean，没必要为了一个不常见的场景给所有`@Component`都加上CGLIB代理的额外开销。

```java
@Configuration
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        // 直接调用上面的方法,就可以返回单例bean,依赖cglib代理
        return new JdbcTemplate(dataSource());
    }
}
```



```java
@Configuration
public class AppConfig {
    @Bean
    public ConnectionFactory connectionFactory() {
        return new ConnectionFactory();
    }
    
    @Bean
    public Connection createConnection(ConnectionFactory factory) { //通过实例工厂方法实例化,不依赖cglib代理.可以直接注入bean
        return factory.createConnection();
    }
}
```





Bean的生命周期有哪些?

经历五个阶段,分为实例化,属性赋值,初始化,使用和销毁.下面的图标识过程:

![三分恶面渣逆袭：Spring Bean生命周期](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/spring-942a927a-86e4-4a01-8f52-9addd89642ff.png)

首先是加载Bean definitions, 对每一个Bean,利用反射调用构造方法创建一个Bean对象, 

实例化阶段只是创建了一个空对象，属性赋值阶段才把它需要的依赖填充进去. 可以看出实例化bean和调用setters赋初值都有依赖注入: 如果是**构造器注入**，那实例化的时候就需要依赖注入了. 属性赋值是第二阶段,对于@autowired,@Resource,@Value注解定义的依赖对象用依赖注入,对setter方法也会依赖注入.

第三阶段是初始化,bean的属性设置好了,要调用一些初始化方法,主要是`@PostConstruct`方法. 并不只有这一个初始化方法,还有 InitializingBean 接口的 `afterPropertiesSet` 方法, 以及 通过 `@Bean` 的 `initMethod` 指定的初始化方法. 在初始化方法中,进行缓存预加载,DB配置工作. 调用完初始化方法后,得到的bean对象不是最终的bean对象, 需要进行BPP后置处理: 因为AOP会创建代理对象, BPP就是注册的 BeanPostProcessor 后置处理方法,经常用来创建代理对象.

最后是销毁阶段,容器关闭或者bean被移除, 调用:

- `@PreDestroy` 标注的方法
- DisposableBean 接口的 destroy 方法
- 通过 `@Bean` 的 destroyMethod 指定的销毁方法

所以看到@Bean的initMethod和destroyMethod,他们作为一种初始化方法以及销毁方法,init-method 是在所有其他初始化方法之后执行,destroyMethod也是一样.



说完了bean的生命周期,来看bean的一些作用域,有哪些?

最常见单例和原型,默认是singleton单例,单例的生命周期与IOC容器保持一致. scope为prototype时,每次获取bean都会创建一个新实例,适合处理bean有状态的情况. spring还有其他作用域,比如request,session,application,websocket. application与Singleton类似,但它的生命周期与 ServletContext 绑定。



单例bean依赖原型bean的情况,怎么解决?

使用@Lookup:

```java
@Component
public class SingletonService {
    // 错误的做法，prototypeBean只会注入一次
    @Autowired
    private PrototypeBean prototypeBean;
    
    // 正确的做法，每次调用都获取新实例
    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null;  // Spring会重写这个方法,方法体不会执行,而是会被子类重写
    }
}
```



作用域scope, 在单例scope会存在线程安全问题吗? 

如果bean是不可变的,不会有线程安全问题,但如果bean中有可变属性,就需要保证线程安全. 解决线程安全问题: 单例bean中通过方法传参,让所有状态用方法参数传递. 如果必须要维护状态,可以用threadlocal保存一个状态. 其他的情况,可以考虑JUC并发包,用原子类,或者线程安全集合,或者直接用锁.

Bean说的差不多了,现在你定义好一个bean之后,把它投入到使用,比如autowired,依赖注入,可以作用在方法和字段上,但是字段注入不推荐为什么?

原因是字段注入相比于构造方法注入有缺点:字段注入不利于进行测试,并且会隐藏循环依赖问题, 通过构造方法注入final字段,可以确保依赖在对象实例化时加载,也就是第一阶段.

`@Autowired` 和`@Resource`区别?

`@Autowired` 默认按照类型，也就是 byType 进行注入，如果有多个相同类型Bean则再按照名字进行匹配. 而 `@Resource` 默认按照名称，也就是 byName 进行注入。 

autowired是依赖注入核心注解,所以说下autowired原理?

基于反射机制和 BeanPostProcessor 接口,具体是`AutowiredAnnotationBeanPostProcessor`,第一阶段是依赖搜集,在实例化后属性赋值前,Processor扫描类,把标注了 `@Autowired` 注解的地方封装成Injection对象.第二阶段:Spring取出所有Injection对象,逐个处理注入点,根据类型去容器中查找匹配的 Bean,利用反射注入.

- Constructor.newInstance(args)
- Field.set()
- Method.invoke()

不同的bean的依赖注入存在依赖链条,依赖注入自动完成叫做自动装配,我们不用告诉 Spring 具体怎么注入，Spring 自己会想办法找到合适的 Bean 注入进来

自动装配的工作原理? 

Spring 容器在启动时自动扫描 `@ComponentScan` 指定包路径下的所有类，然后根据类上的注解，比如 `@Autowired`、`@Resource` 等，来判断哪些 Bean 需要被自动装配。之后分析每个 Bean 的依赖关系，在创建 Bean 的时候，根据装配规则自动找到合适的依赖 Bean，最后根据反射将这些依赖注入到目标 Bean 中。

在注解驱动时代，用得最多的是 `@Autowired` 注解，默认按照类型装配。还有 `@Resource` 注解，它默认按照名称装配，如果找不到对应名称的 Bean，就会按类型装配。

自动装配有更高级的用法:



> Spring Boot 的自动装配还有一套更高级的机制，通过 `@EnableAutoConfiguration` 和各种 `@Conditional` 注解来实现，这个是框架级别的自动装配，会根据 classpath 中的类和配置来自动配置 Bean。

![ShawnBlog：Spring Boot 的自动装配](https://cdn.paicoding.com/stutymore/spring-20250702101032.png)

在图中:通过@SPringbootApplication注解默认开启自动装配和组件扫描, `@SpringBootConfiguration`本身就是用`@Configuration`标注的，只是换了个名字。标识这是Spring Boot应用的主配置类，而不是普通的配置类。

自动装配的表现形式:

**classpath中的类**就是你项目里能加载到的所有类，包括你自己写的代码和maven/gradle引入的依赖jar包里的类,在pom.xml加入redis依赖后,Redis相关的类就出现在classpath里了。

**配置**就是`application.yml`里你写的那些配置项。Spring Boot的自动装配干的事情就是：扫描classpath发现你引入了Redis的依赖（类存在），再读你配置文件里的spring.redis.host等参数，自动帮你创建好RedisTemplate这个bean。你不需要自己写@Bean方法去new它。配置文件不是自动装配的触发条件,而是参数来源.

classpath是什么,@Conditional怎么用?

**classpath**就是JVM查找类的路径。在Spring Boot项目中，简单理解就是你编译后`target/classes`目录下的所有内容加上所有依赖jar包，JVM从这些地方加载类。

**@Conditional**是条件装配注解，Spring Boot提供了一系列派生注解:

- `@ConditionalOnClass`：classpath中存在某个类才生效.检测到依赖存在 → 触发自动配置 → 读配置文件里的值，读不到就用默认值 → 创建bean
- `@ConditionalOnMissingBean`：容器中不存在某个bean才生效。这就是为什么你自己手动定义了一个`DataSource`，Spring Boot就不会再自动创建默认的——它发现你已经有了
- `@ConditionalOnProperty`：配置文件中某个属性满足条件才生效。比如`@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")`，只有你在配置文件里写了`feature.enabled=true`才会创建这个bean。

Bean 自动装配会遇到循环依赖的问题,也就是出现了环, spring必须解决循环依赖问题,解决方法是三级缓存机制

三级缓存的工作过程? 

> 1. 一级缓存 singletonObject：存放完全初始化好的单例 Bean。实例化,注入,初始化.
> 2. 二级缓存earlySingletonObject：存放提前暴露的 Bean，实例化完成，但未初始化完成。
> 3. 三级缓存singletonFactories：存放 Bean 工厂，用于生成提前暴露的 Bean。

三级缓存解决循环依赖的过程?

> AB相互依赖,第一步创建A,此时 A 对象已存在，但 b属性还是 null
>
> 将 A 的对象工厂放入三级缓存,开始进行 A 的属性注入。
>
> 需要 B，但 B 还不存在，所以开始创建 B。
>
> 调用 B 的构造方法，创建 B 的实例。此时 B 对象已存在，但 a 属性还是 null。
>
> 将 B 的对象工厂放入三级缓存。开始进行 B 的属性注入。
>
> B注入A时,先从一级缓存找,没找到,二级缓存也没找到,三级缓存找到A对象工厂,并得到A的实例,A没有完全初始化.
>
> 然后把A的实例放到二级缓存,因为A暴露了. B属性注入完成. 之后进行初始化,完毕后放到一级缓存.
>
> 回到A,A拿到B的完整实例,属性注入,然后A初始化,把A从二级放到一级.

为什么三级缓存一定是单例对象?

> **只有单例bean才需要缓存**。单例在容器中只创建一次，创建过程中如果遇到循环依赖，需要提前暴露一个未完成的引用让对方先拿到，所以需要缓存来存这个中间状态。
>
> 原型（prototype）bean每次获取都new一个新对象，不存在"容器里已经有一个正在创建中的实例"这种情况。如果A依赖B，B依赖A，两个都是prototype，就会无限new下去，Spring直接抛异常，不尝试解决。 
>
> 所以只有在单例情况下提前暴露才有意义.

无法解决循环依赖的情况:

![三分恶面渣逆袭：循环依赖的几种情形](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/spring-37bb576d-b4af-42ed-91f4-d846ceb012b6.png)

构造方法注入发生在实例化阶段，创建 A 的时候必须先有 B，但创建 B又必须先有 A，这时候两个对象都还没创建出来，无法提前暴露到缓存中。

为什么需要第三级缓存存在疑惑,所以原因?

Bean的第三阶段初始化可能会创建新的代理对象,当 B 需要 A 的时候，会调用这个对象工厂的 getObject 方法，这个方法里面会判断 A 是否需要被代理。如果需要代理，就创建 A 的代理对象返回给 B；如果不需要代理，就返回 A 的原始对象。这样就保证了 B 拿到的 A 和最终放入容器的 A 是同一个对象。避免了B中的A对象是原始的,与代理A对象不一致.

为什么需要第二级缓存?

> 假设我们有 A、B、C 三个 Bean，A 依赖 B 和 C，B 和 C 都依赖 A,每次 B 或者 C 需要获取 A 的时候，都需要调用三级缓存中 A 的 `ObjectFactory.getObject()` 方法。这意味着如果 A 需要被代理的话，代理对象可能会被重复创建多次
> 二级缓存就是为了解决这个问题。当第一次通过对象工厂创建了 A 的早期引用之后，就把这个引用放到二级缓存中，同时从三级缓存中移除对象工厂。

Spring有很多特性, 组成部分有很多模块, 这些模块包含哪些?

要让模块发挥作用,用注解进行开发,spring的开发要了解不同的注解. 常用注解有哪些?

注解根据功能可以分为不同部分,要了解常用注解的功能

![三分恶面渣逆袭：Spring常用注解](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/spring-8d0a1518-a425-4887-9735-45321095d927.png)

## SpringMVC

Spring最重要的还是处理请求,也就是MVC. 要搞明白MVC的流程.

SpringMVC的工作流程?

![三分恶面渣逆袭：Spring MVC的工作流程](https://cdn.paicoding.com/tobebetterjavaer/images/sidebar/sanfene/spring-e29a122b-db07-48b8-8289-7251032e87a1.png)

**DispatcherServlet** 是整个 Spring MVC 的前端控制器（Front Controller）。所有 HTTP 请求先到达它，由它统一调度后续流程。它本身是一个标准的 Java Servlet，注册在 web.xml 或通过 Spring Boot 自动配置。可以把它理解为一个"总调度台"——自己不处理业务，只负责把请求分发给正确的处理者，再把结果组装返回。

**HandlerMapping** 负责"请求→处理器"的映射。DispatcherServlet 收到请求后，会问 HandlerMapping："这个 URL 该交给谁？" 常见实现是 `RequestMappingHandlerMapping`，它扫描 `@RequestMapping` / `@GetMapping` 等注解来建立映射关系。返回的结果是一个 **HandlerExecutionChain**，里面包含目标 Handler 以及一组拦截器。

**Handler（Controller）** 就是实际处理业务逻辑的地方，通常是你用 `@Controller` 或 `@RestController` 标注的类中的某个方法。

**HandlerAdapter** 是适配器层。因为 Handler 的形式可以多样（注解方法、实现 Controller 接口、HttpRequestHandler 等），DispatcherServlet 不直接调用 Handler，而是通过 HandlerAdapter 统一调用。最常用的是 `RequestMappingHandlerAdapter`，它负责参数解析（`@RequestParam`、`@RequestBody` 等）、数据绑定、调用方法、处理返回值。

**ModelAndView** 是 Handler 处理完后返回的结果容器，包含两部分：Model（数据，本质是一个 Map）和 View 的逻辑名（比如 `"user/list"`）。在 `@RestController` 场景下通常不走这个路径，而是直接通过 `HttpMessageConverter` 序列化返回 JSON。

**ViewResolver** 把逻辑视图名解析为具体的 View 对象。比如 `InternalResourceViewResolver` 会把 `"user/list"` 解析成 `/WEB-INF/views/user/list.jsp`。Thymeleaf、FreeMarker 各有自己的 ViewResolver 实现。

**View** 负责最终的渲染，把 Model 中的数据填充到模板里，生成 HTML 响应。

**HandlerInterceptor** 是拦截器，提供三个切入点：`preHandle`（Controller 执行前）、`postHandle`（Controller 执行后、视图渲染前）、`afterCompletion`（整个请求完成后）。常用于登录校验、日志记录、性能监控等。和 Servlet Filter 的区别在于它工作在 Spring MVC 层面，能拿到 Handler 信息。

**HttpMessageConverter** 在 REST 场景中极其重要。当方法标注了 `@ResponseBody` 或使用 `@RestController` 时，返回值不走 ViewResolver，而是由 `HttpMessageConverter` 直接序列化（比如 `MappingJackson2HttpMessageConverter` 把对象转成 JSON）。请求体的反序列化（`@RequestBody`）也是它完成的。

整个请求流程串起来就是：

请求 → **DispatcherServlet** → **HandlerMapping**（找到谁来处理）→ **HandlerInterceptor.preHandle** → **HandlerAdapter**（调用 Controller 方法）→ 返回 ModelAndView 或直接通过 **HttpMessageConverter** 写响应 → **ViewResolver** → **View** 渲染 → **HandlerInterceptor.afterCompletion** → 响应返回客户端



## SpringBoot

SpringBoot是践行了约定大于配置的思想的,对spring框架的增强. 它有很多默认的配置.

当我们引入依赖时,如Spring Data JPA,Spring会做什么?

做自动装配,根据项目中引入的依赖自动配置合适的 Bean

那么这个自动装配是很重要的功能,如何开启

开启自动装配的注解是`@EnableAutoConfiguration`这个注解会告诉 Spring 去扫描所有可用的自动配置类。

自动配置的原理:

> 当 main 方法运行的时候，Spring 会去类路径下找 `spring.factories` 这个文件,读取里面配置的自动配置类列表。每个自动配置类内部，通常会有一个 `@Configuration` 注解，同时结合各种 `@Conditional` 注解来做条件控制
> 另外一个常见的场景是自动注入 Bean，Spring Boot 项目在启动时加载所有的自动配置类，然后逐个检查它们的生效条件，当条件满足时就实例化并创建相应的 Bean。
>
> 自动装配的执行时机是在 Spring 容器启动的时候。具体来说是在 ConfigurationClassPostProcessor 这个 BeanPostProcessor 中处理的，它会解析 `@Configuration` 类，包括通过 `@Import` 导入的自动配置类。
>
> 它通过`SpringFactoriesLoader`加载`META-INF/spring.factories`文件中注册的所有自动配置类。
>
> `META-INF`是Java的约定目录，专门放元数据信息，jar包里都有这个目录。只有当你自己写一个自定义starter给别人用的时候，才需要自己创建`META-INF/spring.factories`文件。

`SpringBoot Starter`是什么,如何自定义一个 `SpringBoot Starter`?

<u>Starter 的核心思想是把相关的依赖打包在一起，让开发者只需要引入一个 starter 依赖，就能获得完整的功能模块。每个Starter包含自动配置类,通过条件注解判断是否生效.spring.factories 文件是 Spring Boot 自动装配的核心,位于META_INF目录下</u>

Springboot也增加了哪些功能?

<u>Actuator 监控、DevTools 开发工具、Spring Boot Starter 等等,Spring Boot Starter 则是一些预配置好的依赖集合</u>

相比spring,它增加了一些注解,主要有哪些? 以及他们的意义

- `@SpringBootApplication`：这是 Spring Boot 的核心注解，它是一个组合注解，包含了 `@Configuration`、`@EnableAutoConfiguration` 和 `@ComponentScan`。它标志着一个 Spring Boot 应用的入口。
- `@SpringBootTest`：用于测试 Spring Boot 应用的注解，它会加载整个 Spring 上下文，适合集成测试。

SpringBoot是怎么启动的?

依赖`@SpringBootApplication` 注解和`SpringApplication.run()` `@SpringBootApplication`标记当前类为配置类,能够开启自动装配和组件扫描

`SpringApplication.run()` 是应用程序入口,执行过程包含

- 创建 SpringApplication 实例,识别应用类型,如servlet web还是webflux
- 创建并准备 ApplicationContext,将主类作为配置源进行加载。
- 触发 Bean 的实例化，比如说扫描并注册 `@ComponentScan` 指定路径下的 Bean。
- 触发自动配置，在 Spring Boot 2.7 及之前是通过 spring.factories 加载的，3.x 是通过读取 `AutoConfiguration.imports`，并结合 `@ConditionalOn` 系列注解依据条件注册 Bean。
- 如果引入了 Web 相关依赖，会创建并启动 Tomcat 容器，完成 HTTP 端口监听。

SpringBoot 和Spring的区别?

Spring Boot 不是一个独立的框架，而是基于 Spring 框架的脚手架,让Spring开发变得简单高效.通过 starter 机制解决Spring依赖版本冲突的问题.





## 实际开发

Spring MVC获取 请求参数,路径变量,请求头,请求体:

@RequestParam,@PathVariable,@RequestHeader,@RequestBody



在Filter中,无法用到MVC,而是用HttpServletRequest以及HttpServletResponse，对请求体和响应进行读取和修改.

写入响应用ResponseEntity,如ResponseEntity<Map<String, Object>>



 @Slf4j 是 Lombok 提供的注解，自动生成一个日志对象，等于手写` private static final Logger log = LoggerFactory.getLogger(FileProcessingConsumer.class);  `
