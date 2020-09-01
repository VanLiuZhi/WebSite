---
weight: 1
title: "Java-JVM 知识点总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java-JVM 知识点总结"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

JVM虚拟机总结笔记

## Java内存模型(内存结构)

1. 堆内存 Hape
2. 方法区(或者叫永久代) Method Area(PermGen)  jdk1.8更换为 元空间 Metaspace
3. 线程私有空间

Java内存结构分为三大块，1和2是线程共享的，我们需要理解这3块的含义和作用，基本上现代语言都是数据程序在内存中分离的，数据程序合并的在一些教学书本上有体现，比如8位，10位等处理器的汇编编程，程序简单，空间有限，所以不需要数据和程序分类，对于jvm，数据就是在1中，程序在3中执行，把类信息放在2上 --- `"算法+数据结构=程序"`

![image](/images/Java/Java-jvm-construction.png)

### 线程私有空间

每个线程都会分配一个独立的空间，内部又划分为: 

1. 程序计数器
2. 虚拟机栈
3. 本地方法栈

程序计数器存储当前线程执行的字节码行号；虚拟机栈存储当前线程方法栈帧，每个方法被执行的时候都会创建一个栈帧；本地方法栈和虚拟机栈类似，区别在在于存储的是native方法的栈帧，也就是jvm C++实现的调用，每个线程都有一个独立的程序计数器，虚拟机栈是在方法执行的时候创建栈帧的，不存在垃圾回收问题，线程结束就释放，生命周期和线程一致


虚拟机栈结构图

![image](/images/Java/Java-jvm-stack.png)

虚拟机栈的栈帧又分为: 

1. 局部变量表: 局部变量表是变量值的存储空间，用于存放方法参数和方法内部定义的局部变量，对象引用

2. 操作数栈: 在方法执行的过程中，会有各种字节码指令往操作数栈中入栈和出栈内容，操作数栈开辟的空间就是用来处理这些运算的

3. 方法出口: 一个方法开始执行后，只有两种方式可以退出这个方法

1）是执行引擎遇到任意一个方法返回的字节码指令: 传递给上层的方法调用者，是否有返回值和返回值类型将根据遇到何种方法来返回指令决定，这种退出的方法称为`正常完成出口`
2）方法执行过程中遇到异常: 无论是java虚拟机内部产生的异常还是代码中throw出的异常，只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，这种退出的方式称为`异常完成出口`

无论使用那种方式退出方法，都要返回到方法被调用的位置，程序才能继续执行

4. 动态链接: 每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，Class文件的常量池中存有大量的符号引用，字节码中的方法调用指令就以常量池中方法的符号引用为参数。这些符号引用一部分会在类加载阶段或者第一次使用的时候就转化为直接引用（静态方法，私有方法等），这种转化称为静态解析，另一部分将在每一次运行期间转化为直接引用，这部分称为动态连接

有一个异常StackOverflowError异常，就是方法的栈帧超过最大深度引起的

> 总结:
理解程序计数器和虚拟机栈的作用，虚拟机栈的栈帧，debug调试的时候就是走了一个方法入栈出栈的过程，理解这个栈帧特别重要。至于局部变量表，操作数栈等主要是分析字节码的时候用的，方法出口也是比较偏向底层，动态链接也是一个编程专用的术语，做了解即可

### 线程共享空间

堆内存是JVM中最大的一块由年轻代和老年代组成，而年轻代内存又被分成三部分，Eden空间(伊甸园区)、From Survivor空间(幸存者区)、To Survivor空间，默认情况下年轻代按照8:1:1的比例来分配，堆是垃圾回收重点关注的空间，之所以这么划分，是为了更好地回收内存，或者更快地分配内存。现代垃圾回收多采用分代回收算法

方法区存储类信息(类的元数据)、常量、静态变量等数据，jdk1.8改用元空间，关于这个元空间，还有很多可以展开的，元空间的出现，是因为以前永久代经常内存不足，所以改用了新的实现，最大的特点就是元空间是直接使用本地内存空间的，虽然如此，仍然是可配置的

控制参数

*   -Xms设置堆的最小空间大小。
*   -Xmx设置堆的最大空间大小。
*   -XX:NewSize设置新生代最小空间大小。
*   -XX:MaxNewSize设置新生代最大空间大小。
*   -XX:PermSize设置永久代最小空间大小。 1.8后不适用
*   -XX:MaxPermSize设置永久代最大空间大小。 1.8后不适用
*   -XX:SurvivorRatio设置Eden和其中一个Survivor的比值
*   -Xss设置每个线程的堆栈大小。

没有直接设置老年代的参数，但是可以设置堆空间大小和新生代空间大小两个参数来间接控制。若要配置元空间，Metaspace相关的参数进行配置

> 老年代空间大小=堆空间大小-年轻代大空间大小

### 垃圾回收

主要讨论Hotspot虚拟机的垃圾回收机制，虚拟机技术并不是Java的专利，很多语言都会使用虚拟机技术。gc是自动进行的，手动执行方法是`System.gc()`和`Runtime.gc()`，一般很少去用手动gc

垃圾回收大致经过这样一个流程: 新创建的对象都在Eden区分配，存在时间长的对象会进入From Surivor区(S0区)，更长的会进入To Survivor(S1区)，这是新生代的过程，存在更久的对象，进会进入老年代。永久代也是有垃圾回收的，比如类卸载，不过一般不关注，但是要知道，不是进入永久代的数据就是永久的

`这里要重点聊聊为什么新生代要这样分配`

详细流程：

1. 新对象都是进入Eden
2. 在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的
3. 执行GC，所有活下来的对象从Eden进入To区(使用复制算法)，From区的对象满足阈值的进入老年代，不满足的进入To区
4. GC执行完，From此时是空的，To区增加了对象
5. 这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”
6. 继续GC，不管怎样，都会保证名为To的Survivor区域是空的，直到To满了，所有对象移动到老年代

Survivor的存在意义，就是减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代

为什么要有两个survivor区，是为了避免碎片化，只有一个survivor区会导致survivor的空间使用不连续，因为survivor会有存活的对象，有些进入老年代后就空下了空间，导致内存不连续，通过2个survivor互相交换就很好的解决了这个问题

至于8:1:1的比例，也是因为复制算法，和需要2个survivor做的合理分配

**如何进行GC？这个问题和垃圾回收器相关，不同的垃圾回收器的策略不一样**

`有以下几种常见的策略:`

1. young GC: 

当Eden区满了触发，有些数据也会直接晋升到老年代，所以young GC时，老年代数据有增长是正常现象

2. full GC: 

当准备要触发一次young GC时，如果发现统计数据说之前young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC

3. Minor GC

Minor GC发生在Eden区；Young GC发生在Eden、S0、S1区；Major GC发生在Old区

`触发条件:`

1. Minor GC触发条件：当Eden区满时，触发Minor GC

2. Full GC触发条件：

1)调用System.gc时，系统建议执行Full GC，但是不必然执行

2)老年代空间不足

3)方法去空间不足

4)通过Minor GC后进入老年代的平均大小大于老年代的可用内存

5)由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

`对象什么时候进入老年代？`

1. 根据年龄

JVM会给对象增加一个年龄（age）的计数器，对象每“熬过”一次GC，年龄就要+1，待对象到达设置的阈值（默认为15岁）就会被移移动到老年代，可通过 -XX:MaxTenuringThreshold调整这个阈值

2. 动态年龄判断
  
根据对象年龄有另外一个策略也会让对象进入老年代，不用等待15次GC之后进入老年代，他的大致规则就是，假如当前放对象的Survivor，一批对象的总大小大于这块Survivor内存的50%，那么大于这批对象年龄的对象，就可以直接进入老年代了

举个例子，假如s1区大小是10M，有a=1M,b2M,c=1M,d=3M,e=1M 5个对象，把它们大小排序，然后从最小的开始相加，a+c+e+b+d a,c,e,b合计5M，达到50%了，那么d就直接进入老年代

3. 大对象直接进入老年代

如果设置了 -XX:PretenureSizeThreshold这个参数，那么如果你要创建的对象大于这个参数的值，比如分配一个超大的字节数组，此时就直接把这个大对象放入到老年代，不会经过新生代


### 什么是Stop the world？

Java中Stop-The-World机制简称STW，是在执行垃圾收集算法时，Java应用程序的其他所有线程都被挂起（除了垃圾收集帮助器之外）。Java中一种全局暂停现象，全局停顿，所有Java代码停止，native代码可以执行，但不能与JVM交互；这些现象多半是由于gc引起。

GC时的Stop the World(STW)是大家最大的敌人。但可能很多人还不清楚，除了GC，JVM下还会发生停顿现象。

JVM里有一条特殊的线程－－VM Threads，专门用来执行一些特殊的VM Operation，比如分派GC，thread dump等，这些任务，都需要整个Heap，以及所有线程的状态是静止的，一致的才能进行。所以JVM引入了安全点(Safe Point)的概念，想办法在需要进行VM Operation时，通知所有的线程进入一个静止的安全点。

除了GC，其他触发安全点的VM Operation包括：

1. JIT相关，比如Code deoptimization, Flushing code cache ；

2. Class redefinition (e.g. javaagent，AOP代码植入的产生的instrumentation) ；

3. Biased lock revocation 取消偏向锁 ；

4. Various debug operation (e.g. thread dump or deadlock check)；


### SafePoint 是什么?
    
GC不是什么时候都能进行的，必须要等程序进入一个“安全点”，Java 线程都进入到 safepoint 的时候 VMThread 才能开始执行 GC
    
1. 循环的末尾 (防止大循环的时候一直不进入 safepoint，而其他线程在等待它进入safepoint)

2. 方法返回前

3. 调用方法的 call 之后

4. 抛出异常的位置


### 垃圾回收算法

1. 标记—清除算法（Mark-Sweep）
把被标记的地方清楚掉，该区域就空余出来了，如果是连续的内存分配无法使用该区域

不足:
标记和清除过程效率都不高
会产生大量碎片，内存碎片过多可能导致无法给大对象分配内存。

2. 复制算法（Copying）
将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理

不足:
将内存缩小为原来的一半，浪费了一半的内存空间，代价太高；如果不想浪费一半的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法
复制收集算法在对象存活率较高时就要进行较多的复制操作，效率将会变低

3. 标记—整理算法（Mark-Compact）
与1类似，多了整理的过程

不足:
效率不高，不仅要标记存活对象，还要整理所有存活对象的引用地址，在效率上不如复制算法

4. 分代收集算法(Generational Collection)
是复制算法和标记算法的整合，对不同区域使用不同策略

新生代：由于新生代产生很多临时对象，大量对象需要进行回收，所以采用复制算法是最高效的。
老年代：回收的对象很少，都是经过几次标记后都不是可回收的状态转移到老年代的，所以仅有少量对象需要回收，故采用标记清除或者标记整理算法。


### 垃圾回收器

垃圾回收器才是真正执行GC的程序，在了解垃圾回收器之前，需要知道如何判断对象是否需要回收，有引用计数法和可达性分析算法

由于引用计数很难处理对象循环引用问题，多少采用可达性分析算法

有一个GC Roots的对象作为起点，以下对象可以作为起点

1. 虚拟机栈中引用的对象
        
2. 方法区中类静态属性引用的对象 
  
3. 方法区中的常量引用的对象
    
4. 本地方法栈中JNI（即一般说的Native方法）的引用的对象

从这些起点向下走过的路径，称为引用链，当一个对象到CG Root没有任何引用链的话，则说明此对象不可用

> 补充: 即使在可达性分析法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑阶段”，要真正宣告一个对象死亡，至少要经历两次标记过程；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize方法。当对象没有覆盖finalize方法，或finalize方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收

理解了对象如何被标记为可以，还有一个`对象引用类型`的概念

1. 强引用 一般我们new创建的，只有被标记为null了，才会被回收

2. 软引用 内存不足的时候就会回收，不管你对象还在不在用，软引用的访问可以直达内存，类似高速缓存的效果，不用从堆中查找

```java
Object obj = new Object(); 
SoftReference<Object> sf = new SoftReference<Object>(obj); 
obj = null; sf.get();//有时候会返回null
```

3. 弱引用 被gc线程找到就会回收，或者说存在感低，有一些资料上说，第二次gc的时候会回收，在ThreadLocal中有使用

```java
Object obj = new Object(); 
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null; 
wf.get();//有时候会返回null wf.isEnQueued();//返回是否被垃圾回收器标记为即将回收的垃圾
```

4. 虚引用 取不到，但是有相关的方法获取状态，可以用来判断对象是否已经从内存中删除

```java
Object obj = new Object(); 
PhantomReference<Object> pf = new PhantomReference<Object>(obj); 
obj=null; 
pf.get();//永远返回null
pf.isEnQueued();//返回是否从内存中已经删除
```

> 有人可能会问，这些类型有什么用呢？看上面的代码，除了强引用外，其它引用都是拿强引用创建的，比如软引用，我们就可以得到对象的一个缓存高速访问，不过很少见到代码中这样用，慎重

**接下来可以聊聊垃圾收集器了，它是回收算法的具体实现，由于gc的时候会swt，所以延伸了很多垃圾回收器，没有万能的垃圾回收器**

1. Serial收集器 简单高效，单线程，swt影响严重

2. ParNew收集器 Serial的多线程版

3. Parallel Scavenge收集器 它的特定就是注重CPU的利用率，就是尽量不swt

4. Serial Old收集器 Serial的老年代版

5. Parallel Old收集器 Parallel Old收集器 的老年代版

6. CMS收集器（Concurrent Mark Sweep） 特点就是swt占用时间短

7. G1 从整体来看是基于“标记—整理”算法实现的收集器，从局部（两个 Region 之间）上来看是基于“复制”算法实现的

## 类加载机制和流程

该主题主要讲class文件如何被加载到jvm中去的，整个流程为: 加载，验证，准备，解析，初始化，这里不展开了

需要重点了解的是加载器和加载器工作流程(使用了双亲委派)

加载器分类:

1. 启动类加载器 Bootstrap ClassLoader，由jvm来提供

2. 扩展类加载器 Extension ClassLoader

3. 应用程序加载器或系统类加载器 Application ClassLoader

加载过程(双亲委派):
appcl要加载一个类，不是它来加载，而是由它的父类就是extcl来加载，而extcl又把这个加载委托给它的父类bootcl，如果加载成功就返回，否则就重新由appcl来加载。appcl会委托父类进行加载的前提是缓存中没有找到这个类，比如第一次加载的时候

之所以使用双亲委派，是为了避免重复加载，先使用系统的。比如你自定义了java.util.HashMap类，执行的时候会去使用系统的

可以调用Class对象的getClassLoader方法获取加载器，调用getParent获取父加载器，启动类加载器java获取不到，会打印null

```java
import com.sun.javafx.PlatformUtil;

public class ClassLodersDemo {
   public static void main(String[] args) {
      Object object = new Object();
      ClassLodersDemo demo=new ClassLodersDemo();
      System.out.println(object.getClass().getClassLoader()); // null
      System.out.println(Object.class.getClassLoader());//java拿不到, null
      System.out.println(ClassLodersDemo.class.getClassLoader());//app
      System.out.println(PlatformUtil.class.getClassLoader());//ext
      System.out.println(demo.getClass().getClassLoader().getParent()); // ext
      System.out.println(demo.getClass().getClassLoader().getParent().getParent()); //null
   }
}
```

## 默认堆大小

默认堆大小：
若没有在命令行中指定了初始化和最大的堆大小，则取决于计算机上的的物理内存大小

服务器端的默认堆大小
初始化堆大小：客户端JVM相同
最大堆小大：
32位的JVM上，物理内存小于192MB时，为物理内存的一半；物理内存大192MB且小于4GB时，为物理内存的四分之一；大于等于4GB时，都为1GB
64位的JVM上，物理内存小于192MB时，为物理内存的一半；物理内存大192MB且小于128GB时，为物理内存的四分之一；大于等于128GB时，都为32GB

`System.out.println(Runtime.getRuntime().maxMemory());`可以打印，以我的计算机为例，结果是3817865216，差不多就是16GB的四分之一

经过测试，spring boot项目，什么都不配置，每次启动的Heap可用空间会有不同，而且可用保持不变的，下次启动会改变。最大空间固定

## 参考

https://www.cnblogs.com/duanxz/p/6076662.html
https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size

## 常量池、运行时常量池、字符串常量池

经常会看到这些概念，这里做个总结

`常量池`：即class文件常量池，是class文件的一部分，用于保存编译时确定的数据

class文件常量池在元空间中

![image](/images/Java/Java-jvm-const.png)

可以用命令javap -verbose class文件(比如Test) Classfile class文件地址 查看

`运行时常量池`：

Java语言并不要求常量一定只能在编译期产生，运行期间也可能产生新的常量，这些常量被放在运行时常量池中。

类加载后，常量池中的数据会在运行时常量池中存放！

这里所说的常量包括：基本类型包装类（包装类不管理浮点型，整形只会管理-128到127，不在这个范围的包装类型会创建新的对象，在堆空间）和String（也可以通过String.intern()方法可以强制将String放入常量池）

`字符串常量池`：

HotSpot VM里，记录interned string的一个全局表叫做StringTable，它本质上就是个HashSet<String>。注意它只存储对java.lang.String实例的引用，而不存储String对象的内容。就是我们创建String包装类型，如果是同样的内容，那么引入指向同一个地方，就在字符串常量池存储引用，当然数据本身是在方法区中分配的

jdk 1.7后，移除了方法区(可能这么说也不规范，方法区是虚拟机规范的称呼，应该说是移除永久代吧)，`运行时常量池`在`方法区`里，`字符串常量池`在`堆`中，数据在方法区中