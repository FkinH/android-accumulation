### JVM运行内存的分类
程序计数器：当前线程所执行的字节码的行号指示器，用于记录下一条要运行的指令，线程私有
注：如果正在执行的是Native方法，计数器值则为空
Java虚拟栈：存放基本数据类型、对象的引用、方法出口等，线程私有
Native方法栈：和虚拟栈相似，只不过它服务于Native方法，线程私有
Java堆：java内存最大的一块，所有对象实例、数组都存放在java堆，GC回收的地方，线程共享
方法区：存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码数据等。（即永久带），回收目标主要是常量池的回收和类型的卸载，各线程共享

### Java内存堆和栈区别
栈内存用来存储基本类型的变量和对象的引用变量，堆内存用来存储Java中的对象，无论是成员变量，局部变量，还是类变量，它们指向的对象都存储在堆内存中
栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存，堆内存中的对象对所有线程可见。堆内存中的对象可以被所有线程访问
如果栈内存没有可用的空间存储方法调用和局部变量，JVM会抛出java.lang.StackOverFlowError，如果是堆内存没有可用的空间存储生成的对象，JVM会抛出java.lang.OutOfMemoryError
栈的内存要远远小于堆内存，如果你使用递归的话，那么你的栈很快就会充满，-Xss选项设置栈内存的大小。-Xms选项可以设置堆的开始时的大小

### Java四引用

强引用（StrongReference）强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题

软引用（SoftReference）
如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中

弱引用（WeakReference）
弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中

虚引用（PhantomReference）
虚引用在任何时候都可能被垃圾回收器回收，主要用来跟踪对象被垃圾回收器回收的活动，被回收时会收到一个系统通知。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

### GC回收机制
Java中对象是采用new或者反射的方法创建的，这些对象的创建都是在堆（Heap）中分配的，所有对象的回收都是由Java虚拟机通过垃圾回收机制完成的。GC为了能够正确释放对象，会监控每个对象的运行状况，对他们的申请、引用、被引用、赋值等状况进行监控
Java程序员不用担心内存管理，因为垃圾收集器会自动进行管理
可以调用下面的方法之一：System.gc() 或Runtime.getRuntime().gc() ，但JVM可以屏蔽掉显示的垃圾回收调用
GC 标记对象的死活
引用计数法：给对象添加一个引用计数器,没当被引用的时候,计数器的值就加一。引用失效的时候减一,当计数器的值为 0 的时候就表示改对象可以被 GC 回收了，弊端:A->B,B->A,那么 AB 将永远不会被回收了。也就是引用有环的情况
根搜索算法(可达性算法) GC Roots Tracing：通过一个叫 GC Roots 的对象作为起点,从这些结点开始向下搜索,搜索所走过的路径称为引用链,当一个对象没有与任何的引用链相连的时候则改对象就可以被。 GC 回收回收了Roots 包括：java 虚拟机栈中引用的对象,本地方法栈中引用的对象,方法区中常量引用的对象,方法区中静态属性引用的对象
在Java语言里，可作为GC Roots的对象包括以下几种：
 虚拟机栈（栈帧中的本地变量表）中的引用的对象
 方法区中的类静态属性引用的对象
 方法区中的常量引用的对象。
 本地方法栈中JNI(即一般说的Native方法)的引用的对象。

GC回收算法
标记-清除法：标记出没有用的对象，然后一个一个回收掉
缺点：标记和清除两个过程效率不高，产生内存碎片导致需要分配较大对象时无法找到足够的连续内存而需要触发一次GC操作
复制算法: 按照容量划分二个大小相等的内存区域，当一块用完的时候将活着的对象复制到另一块上，然后再把已使用的内存空间一次清理掉
缺点：将内存缩小为了原来的一半
标记-整理法：标记出没有用的对象，让所有存活的对象都向一端移动，然后直接清除掉端边界以外的内
优点：解决了标记- 清除算法导致的内存碎片问题和在存活率较高时复制算法效率低的问题。
分代回收：根据对象存活周期的不同将内存划分为几块，一般是新生代和老年代，新生代基本采用复制算法，老年代采用标记整理算法