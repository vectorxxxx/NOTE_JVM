> 笔记来源：[尚硅谷 JVM 全套教程，百万播放，全网巅峰（宋红康详解 java 虚拟机）](https://www.bilibili.com/video/BV1PJ411n7xZ "尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）")
>
> 同步更新：https://gitee.com/vectorx/NOTE_JVM
>
> https://codechina.csdn.net/qq_35925558/NOTE_JVM
>
> https://github.com/uxiahnan/NOTE_JVM

[toc]

# 13. 垃圾回收器

## 13.1. GC 分类与性能指标

### 13.1.1. 垃圾回收器概述

垃圾收集器没有在规范中进行过多的规定，可以由不同的厂商、不同版本的 JVM 来实现。

由于 JDK 的版本处于高速迭代过程中，因此 Java 发展至今已经衍生了众多的 GC 版本。

从不同角度分析垃圾收集器，可以将 GC 分为不同的类型。

### 13.1.2. 垃圾收集器分类

按<mark>线程数</mark>分，可以分为<mark>串行垃圾回收器</mark>和<mark>并行垃圾回收器</mark>。

![image-20210512144253383](https://img-blog.csdnimg.cn/img_convert/ab10d1899d353ea14797f9ce1778503c.png)

串行回收指的是在同一时间段内只允许有一个 CPU 用于执行垃圾回收操作，此时工作线程被暂停，直至垃圾收集工作结束。

- 在诸如单 CPU 处理器或者较小的应用内存等硬件平台不是特别优越的场合，串行回收器的性能表现可以超过并行回收器和并发回收器。所以，<mark>串行回收默认被应用在客户端的 Client 模式下的 JVM 中</mark>
- 在并发能力比较强的 CPU 上，并行回收器产生的停顿时间要短于串行回收器。

和串行回收相反，并行收集可以运用多个 CPU 同时执行垃圾回收，因此提升了应用的吞吐量，不过并行回收仍然与串行回收一样，采用独占式，使用了“Stop-the-World”机制。

按照<mark>工作模式</mark>分，可以分为<mark>并发式垃圾回收器</mark>和<mark>独占式垃圾回收器</mark>。

- 并发式垃圾回收器与应用程序线程交替工作，以尽可能减少应用程序的停顿时间。
- 独占式垃圾回收器（Stop the world）一旦运行，就停止应用程序中的所有用户线程，直到垃圾回收过程完全结束。

![image-20200713083443486](https://img-blog.csdnimg.cn/img_convert/6e2c2869a4450dc405bda0ea8a8e7c31.png)

按<mark>碎片处理方式</mark>分，可分为<mark>压缩式垃圾回收器</mark>和<mark>非压缩式垃圾回收器</mark>。

- 压缩式垃圾回收器会在回收完成后，对存活对象进行压缩整理，消除回收后的碎片。
- 非压缩式的垃圾回收器不进行这步操作。

按<mark>工作的内存区间</mark>分，又可分为<mark>年轻代垃圾回收器</mark>和<mark>老年代垃圾回收器</mark>。

### 13.1.3. 评估 GC 的性能指标

- <mark>吞吐量</mark>：运行用户代码的时间占总运行时间的比例（总运行时间 = 程序的运行时间 + 内存回收的时间）
- <mark>垃圾收集开销</mark>：吞吐量的补数，垃圾收集所用时间与总运行时间的比例。
- <mark>暂停时间</mark>：执行垃圾收集时，程序的工作线程被暂停的时间。
- <mark>收集频率</mark>：相对于应用程序的执行，收集操作发生的频率。
- <mark>内存占用</mark>：Java 堆区所占的内存大小。
- <mark>快速</mark>：一个对象从诞生到被回收所经历的时间。

吞吐量、暂停时间、内存占用 这三者共同构成一个“不可能三角”。三者总体的表现会随着技术进步而越来越好。一款优秀的收集器通常最多同时满足其中的两项。

这三项里，暂停时间的重要性日益凸显。因为随着硬件发展，内存占用多些越来越能容忍，硬件性能的提升也有助于降低收集器运行时对应用程序的影响，即提高了吞吐量。而内存的扩大，对延迟反而带来负面效果。

简单来说，主要抓住两点：吞吐量、暂停时间

#### 吞吐量

吞吐量就是 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值，即吞吐量 = 运行用户代码时间 /（运行用户代码时间+垃圾收集时间）。比如：虚拟机总共运行了 100 分钟，其中垃圾收集花掉 1 分钟，那吞吐量就是 99%。

这种情况下，应用程序能容忍较高的暂停时间，因此，高吞吐量的应用程序有更长的时间基准，快速响应是不必考虑的

吞吐量优先，意味着在单位时间内，STW 的时间最短：0.2 + 0.2 = 0.4

![image-20200713084726176](https://img-blog.csdnimg.cn/img_convert/a05d48c1926a03c3acdebf74d10bf522.png)

#### 暂停时间

“暂停时间”是指一个时间段内应用程序线程暂停，让 GC 线程执行的状态。

例如，GC 期间 100 毫秒的暂停时间意味着在这 100 毫秒期间内没有应用程序线程是活动的。

暂停时间优先，意味着尽可能让单次 STW 的时间最短：0.1 + 0.1 + 0.1 + 0.1 + 0.1 = 0.5

![image-20200713085306400](https://img-blog.csdnimg.cn/img_convert/de90092e21cbff31926f7cc7dceebf25.png)

#### 吞吐量 vs 暂停时间

高吞吐量较好因为这会让应用程序的最终用户感觉只有应用程序线程在做“生产性”工作。直觉上，吞吐量越高程序运行越快。

低暂停时间（低延迟）较好因为从最终用户的角度来看不管是 GC 还是其他原因导致一个应用被挂起始终是不好的。这取决于应用程序的类型，<mark>有时候甚至短暂的 200 毫秒暂停都可能打断终端用户体验</mark>。因此，具有低的较大暂停时间是非常重要的，特别是对于一个<mark>交互式应用程序</mark>。

不幸的是”高吞吐量”和”低暂停时间”是一对相互竞争的目标（矛盾）。

- 因为如果选择以吞吐量优先，那么<mark>必然需要降低内存回收的执行频率</mark>，但是这样会导致 GC 需要更长的暂停时间来执行内存回收。
- 相反的，如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也<mark>只能频繁地执行内存回收</mark>，但这又引起了年轻代内存的缩减和导致程序吞吐量的下降。

在设计（或使用）GC 算法时，我们必须确定我们的目标：一个 GC 算法只可能针对两个目标之一（即只专注于较大吞吐量或最小暂停时间），或尝试找到一个二者的折衷。

现在标准：<mark>在最大吞吐量优先的情况下，降低停顿时间</mark>

## 13.2. 不同的垃圾回收器概述

垃圾收集机制是 Java 的招牌能力，极大地提高了开发效率。这当然也是面试的热点。

### 13.2.1. 垃圾回收器发展史

有了虚拟机，就一定需要收集垃圾的机制，这就是 Garbage Collection，对应的产品我们称为 Garbage Collector。

- 1999 年随 JDK1.3.1 一起来的是串行方式的 serialGc，它是第一款 GC。ParNew 垃圾收集器是 Serial 收集器的多线程版本
- 2002 年 2 月 26 日，Parallel GC 和 Concurrent Mark Sweep GC 跟随 JDK1.4.2 一起发布·
- Parallel GC 在 JDK6 之后成为 HotSpot 默认 GC。
- 2012 年，在 JDK1.7u4 版本中，G1 可用。
- 2017 年，JDK9 中 G1 变成默认的垃圾收集器，以替代 CMS。
- 2018 年 3 月，JDK10 中 G1 垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。
- 2018 年 9 月，JDK11 发布。引入 Epsilon 垃圾回收器，又被称为 "No-Op(无操作)“ 回收器。同时，引入 ZGC：可伸缩的低延迟垃圾回收器（Experimental）
- 2019 年 3 月，JDK12 发布。增强 G1，自动返回未用堆内存给操作系统。同时，引入 Shenandoah GC：低停顿时间的 GC（Experimental）。·
- 2019 年 9 月，JDK13 发布。增强 ZGC，自动返回未用堆内存给操作系统。
- 2020 年 3 月，JDK14 发布。删除 CMS 垃圾回收器。扩展 ZGC 在 macos 和 Windows 上的应用

### 13.2.2. 7 种经典的垃圾收集器

- 串行回收器：Serial、Serial Old
- 并行回收器：ParNew、Parallel Scavenge、Parallel old
- 并发回收器：CMS、G1

![image-20200713093551365](https://img-blog.csdnimg.cn/img_convert/90c3bcdc22cd0b49e10d702c608c4fc6.png)

官方手册：[https://www.oracle.com/technetwork/java/javase/tech/memorymanagement-whitepaper-1-150020.pdf](https://www.oracle.com/technetwork/java/javase/tech/memorymanagement-whitepaper-1-150020.pdf)

![image-20210512145950897](https://img-blog.csdnimg.cn/img_convert/c529d76b22212c44275b94675cc56760.png)

### 13.2.3. 7 款经典收集器与垃圾分代之间的关系

![image-20200713093757644](https://img-blog.csdnimg.cn/img_convert/fd16701d3e150d5e58d52b7306473a42.png)

- 新生代收集器：Serial、ParNew、Parallel Scavenge；

- 老年代收集器：Serial Old、Parallel Old、CMS；

- 整堆收集器：G1；

### 13.2.4. 垃圾收集器的组合关系

![image-20200713094745366](https://img-blog.csdnimg.cn/img_convert/b92c2212bea2907cb75ff9ef26f346fe.png)

1. 两个收集器间有连线，表明它们可以搭配使用：Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1；
2. 其中 Serial Old 作为 CMS 出现"`Concurrent Mode Failure`"失败的后备预案。
3. （红色虚线）由于维护和兼容性测试的成本，在 JDK 8 时将 Serial+CMS、ParNew+Serial Old 这两个组合声明为废弃（JEP173），并在 JDK9 中完全取消了这些组合的支持（JEP214），即：移除。
4. （绿色虚线）JDK14 中：弃用 Parallel Scavenge 和 Serialold GC 组合（JEP366）
5. （绿色虚框）JDK14 中：删除 CMS 垃圾回收器（JEP363）

### 13.2.5. 不同的垃圾收集器概述

为什么要有很多收集器，一个不够吗？因为 Java 的使用场景很多，移动端，服务器等。所以就需要针对不同的场景，提供不同的垃圾收集器，提高垃圾收集的性能。

虽然我们会对各个收集器进行比较，但并非为了挑选一个最好的收集器出来。没有一种放之四海皆准、任何场景下都适用的完美收集器存在，更加没有万能的收集器。所以<mark>我们选择的只是对具体应用最合适的收集器</mark>。

### 13.2.6. 如何查看默认垃圾收集器

`-XX:+PrintCommandLineFlags`：查看命令行相关参数（包含使用的垃圾收集器）

使用命令行指令：`jinfo -flag 相关垃圾回收器参数 进程ID`

## 13.3. Serial 回收器：串行回收

Serial 收集器是最基本、历史最悠久的垃圾收集器了。JDK1.3 之前回收新生代唯一的选择。

Serial 收集器作为 HotSpot 中 client 模式下的默认新生代垃圾收集器。

<mark>Serial 收集器采用复制算法、串行回收和"stop-the-World"机制的方式执行内存回收。</mark>

除了年轻代之外，Serial 收集器还提供用于执行老年代垃圾收集的 Serial Old 收集器。<mark>Serial Old 收集器同样也采用了串行回收和"Stop the World"机制，只不过内存回收算法使用的是标记-压缩算法。</mark>

- Serial old 是运行在 Client 模式下默认的老年代的垃圾回收器
- Serial 0ld 在 Server 模式下主要有两个用途：① 与新生代的 Parallel scavenge 配合使用 ② 作为老年代 CMS 收集器的后备垃圾收集方案

![image-20200713100703799](https://img-blog.csdnimg.cn/img_convert/d66b612e68381df2101c3e829a18b4f0.png)

这个收集器是一个单线程的收集器，但它的“单线程”的意义并不仅仅说明它只会<mark>使用一个 CPU 或一条收集线程去完成垃圾收集工作</mark>，更重要的是在它进行垃圾收集时，<mark>必须暂停其他所有的工作线程</mark>，直到它收集结束（Stop The World）

优势：<mark>简单而高效</mark>（与其他收集器的单线程比），对于限定单个 CPU 的环境来说，Serial 收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。运行在 Client 模式下的虚拟机是个不错的选择。

在用户的桌面应用场景中，可用内存一般不大（几十 MB 至一两百 MB），可以在较短时间内完成垃圾收集（几十 ms 至一百多 ms），只要不频繁发生，使用串行回收器是可以接受的。

在 HotSpot 虚拟机中，使用`-XX:+UseSerialGC`参数可以指定年轻代和老年代都使用串行收集器。等价于新生代用 Serial GC，且老年代用 Serial Old GC

**总结**

这种垃圾收集器大家了解，现在已经不用串行的了。而且在限定单核 cpu 才可以用。现在都不是单核的了。

对于交互较强的应用而言，这种垃圾收集器是不能接受的。一般在 Java web 应用程序中是不会采用串行垃圾收集器的。

## 13.4. ParNew 回收器：并行回收

如果说 Serial GC 是年轻代中的单线程垃圾收集器，那么 ParNew 收集器则是 Serial 收集器的多线程版本。Par 是 Parallel 的缩写，New：只能处理的是新生代

ParNew 收集器除了采用<mark>并行回收</mark>的方式执行内存回收外，两款垃圾收集器之间几乎没有任何区别。ParNew 收集器在年轻代中同样也是采用<mark>复制算法、"Stop-the-World"机制</mark>。

ParNew 是很多 JVM 运行在 Server 模式下新生代的默认垃圾收集器。

![image-20200713102030127](https://img-blog.csdnimg.cn/img_convert/187fdcd46a1cb35be6d88a01a433c0f3.png)

- 对于新生代，回收次数频繁，使用并行方式高效。
- 对于老年代，回收次数少，使用串行方式节省资源。（CPU 并行需要切换线程，串行可以省去切换线程的资源）

由于 ParNew 收集器是基于并行回收，那么是否可以断定 ParNew 收集器的回收效率在任何场景下都会比 serial 收集器更高效？

- ParNew 收集器运行在多 CPU 的环境下，由于可以充分利用多 CPU、多核心等物理硬件资源优势，可以更快速地完成垃圾收集，提升程序的吞吐量。
- 但是<mark>在单个 CPU 的环境下，ParNew 收集器不比 Serial 收集器更高效</mark>。虽然 Serial 收集器是基于串行回收，但是由于 CPU 不需要频繁地做任务切换，因此可以有效避免多线程交互过程中产生的一些额外开销。

因为除 Serial 外，目前只有 ParNew GC 能与 CMS 收集器配合工作

在程序中，开发人员可以通过选项"`-XX:+UseParNewGC`"手动指定使用 ParNew 收集器执行内存回收任务。它表示年轻代使用并行收集器，不影响老年代。

`-XX:ParallelGCThreads`限制线程数量，默认开启和 CPU 数据相同的线程数。

## 13.5. Parallel 回收器：吞吐量优先

HotSpot 的年轻代中除了拥有 ParNew 收集器是基于并行回收的以外，Parallel Scavenge 收集器同样也采用了<mark>复制算法、并行回收和"Stop the World"机制</mark>。

那么 Parallel 收集器的出现是否多此一举？

- 和 ParNew 收集器不同，ParallelScavenge 收集器的目标则是达到一个<mark>可控制的吞吐量</mark>（Throughput），它也被称为吞吐量优先的垃圾收集器。
- 自适应调节策略也是 Parallel Scavenge 与 ParNew 一个重要区别。

高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要<mark>适合在后台运算而不需要太多交互的任务</mark>。因此，常见在服务器环境中使用。<mark>例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序</mark>。

Parallel 收集器在 JDK1.6 时提供了用于执行老年代垃圾收集的 Parallel Old 收集器，用来代替老年代的 Serial Old 收集器。

Parallel Old 收集器采用了<mark>标记-压缩算法</mark>，但同样也是基于<mark>并行回收和"Stop-the-World"机制</mark>。

![image-20200713110359441](https://img-blog.csdnimg.cn/img_convert/8a4b655ee277aaf0f9a46754248ce05a.png)

在程序吞吐量优先的应用场景中，Parallel 收集器和 Parallel Old 收集器的组合，在 Server 模式下的内存回收性能很不错。在 Java8 中，默认是此垃圾收集器。

**参数配置**

- `-XX:+UseParallelGC` 手动指定年轻代使用 Parallel 并行收集器执行内存回收任务。

- `-XX:+UseParallelOldGC` 手动指定老年代都是使用并行回收收集器。

  - 分别适用于新生代和老年代。默认 jdk8 是开启的。
  - 上面两个参数，默认开启一个，另一个也会被开启。（互相激活）

- `-XX:ParallelGCThreads` 设置年轻代并行收集器的线程数。一般地，最好与 CPU 数量相等，以避免过多的线程数影响垃圾收集性能。

  $$ ParallelGCThreads = \begin{cases} CPU_Count & \text (CPU_Count <= 8) \\ 3 + (5 \* CPU＿Count / 8) & \text (CPU_Count > 8) \end{cases} $$

- `-XX:MaxGCPauseMillis` 设置垃圾收集器最大停顿时间（即 STw 的时间）。单位是毫秒。

  - 为了尽可能地把停顿时间控制在 MaxGCPauseMills 以内，收集器在工作时会调整 Java 堆大小或者其他一些参数。
  - 对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器端适合 Parallel，进行控制。
  - <mark>该参数使用需谨慎</mark>。

- `-XX:GCTimeRatio` 垃圾收集时间占总时间的比例（=1/（N+1））。用于衡量吞吐量的大小。

  - 取值范围（0, 100）。默认值 99，也就是垃圾回收时间不超过 1%。
  - 与前一个`-XX:MaxGCPauseMillis `参数有一定矛盾性。暂停时间越长，Radio 参数就容易超过设定的比例。

- `-XX:+UseAdaptivesizePolicy` 设置 Parallel Scavenge 收集器具有<mark>自适应调节策略</mark>

  - 在这种模式下，年轻代的大小、Eden 和 Survivor 的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。
  - 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（`GCTimeRatio`）和停顿时间（`MaxGCPauseMills`），让虚拟机自己完成调优工作。

## 13.6. CMS 回收器：低延迟

在 JDK1.5 时期，Hotspot 推出了一款在<mark>强交互应用</mark>中几乎可认为有划时代意义的垃圾收集器：CMS（Concurrent-Mark-Sweep）收集器，<mark>这款收集器是 HotSpot 虚拟机中第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程同时工作</mark>。

CMS 收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间。停顿时间越短（低延迟）就越适合与用户交互的程序，良好的响应速度能提升用户体验。

- <mark>目前很大一部分的 Java 应用集中在互联网站或者 B/S 系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短</mark>，以给用户带来较好的体验。CMS 收集器就非常符合这类应用的需求。

CMS 的垃圾收集算法采用<mark>标记-清除算法</mark>，并且也会"Stop-the-World"

不幸的是，CMS 作为老年代的收集器，却无法与 JDK1.4.0 中已经存在的新生代收集器 Parallel Scavenge 配合工作，所以在 JDK1.5 中使用 CMS 来收集老年代的时候，新生代只能选择 ParNew 或者 Serial 收集器中的一个。

在 G1 出现之前，CMS 使用还是非常广泛的。一直到今天，仍然有很多系统使用 CMS GC。

![image-20200713205154007](https://img-blog.csdnimg.cn/img_convert/f84a132db8c56a488e14f51e2c4d7fa7.png)

CMS 整个过程比之前的收集器要复杂，整个过程分为 4 个主要阶段，即初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段

- **初始标记**（Initial-Mark）阶段：在这个阶段中，程序中所有的工作线程都将会因为“Stop-the-World”机制而出现短暂的暂停，这个阶段的主要任务<mark>仅仅只是标记出 GCRoots 能直接关联到的对象</mark>。一旦标记完成之后就会恢复之前被暂停的所有应用线程。由于直接关联对象比较小，所以这里的<mark>速度非常快</mark>。
- **并发标记**（Concurrent-Mark）阶段：从 GC Roots 的<mark>直接关联对象开始遍历整个对象图的过程</mark>，这个过程耗时较长但是<mark>不需要停顿用户线程</mark>，可以与垃圾收集线程一起并发运行。
- **重新标记**（Remark）阶段：由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了<mark>修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录</mark>，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短。
- **并发清除**（Concurrent-Sweep）阶段：此阶段<mark>清理删除掉标记阶段判断的已经死亡的对象，释放内存空间</mark>。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

尽管 CMS 收集器采用的是并发回收（非独占式），但是<mark>在其初始化标记和再次标记这两个阶段中仍然需要执行“Stop-the-World”机制</mark>暂停程序中的工作线程，不过暂停时间并不会太长，因此可以说明目前所有的垃圾收集器都做不到完全不需要“stop-the-World”，只是尽可能地缩短暂停时间。

<mark>由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的。</mark>

另外，由于在垃圾收集阶段用户线程没有中断，<mark>所以在 CMS 回收过程中，还应该确保应用程序用户线程有足够的内存可用</mark>。因此，CMS 收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是<mark>当堆内存使用率达到某一阈值时，便开始进行回收</mark>，以确保应用程序在 CMS 工作过程中依然有足够的空间支持应用程序运行。要是 CMS 运行期间预留的内存无法满足程序需要，就会出现一次“`Concurrent Mode Failure`” 失败，这时虚拟机将启动后备预案：临时启用 Serial Old 收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。

CMS 收集器的垃圾收集算法采用的是标记清除算法，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，不可避免地将会<mark>产生一些内存碎片</mark>。那么 CMS 在为新对象分配内存空间时，将无法使用指针碰撞（Bump the Pointer）技术，而只能够选择空闲列表（Free List）执行内存分配。

![image-20200713212230352](https://img-blog.csdnimg.cn/img_convert/052d6ef7655e46e040729082ac36da30.png)

**有人会觉得既然 Mark Sweep 会造成内存碎片，那么为什么不把算法换成 Mark Compact？**

答案其实很简单，因为当并发清除的时候，用 Compact 整理内存的话，原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提的它运行的资源不受影响嘛。Mark Compact 更适合“Stop the World” 这种场景下使用

### 13.6.1. CMS 的优点

- 并发收集
- 低延迟

### 13.6.2. CMS 的弊端

- <mark>会产生内存碎片</mark>，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发 FullGC。
- <mark>CMS 收集器对 CPU 资源非常敏感</mark>。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。
- <mark>CMS 收集器无法处理浮动垃圾</mark>。可能出现“`Concurrent Mode Failure`"失败而导致另一次 Full GC 的产生。在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，那么<mark>在并发标记阶段如果产生新的垃圾对象，CMS 将无法对这些垃圾对象进行标记，最终会导致这些新产生的垃圾对象没有被及时回收</mark>，从而只能在下一次执行 GC 时释放这些之前未被回收的内存空间。

### 13.6.3. 设置的参数

- `-XX:+UseConcMarkSweepGC `手动指定使用 CMS 收集器执行内存回收任务。

  开启该参数后会自动将`-xx:+UseParNewGC`打开。即：ParNew（Young 区用）+CMS（Old 区用）+ Serial Old 的组合。

- `-XX:CMSInitiatingOccupanyFraction` 设置堆内存使用率的阈值，一旦达到该阈值，便开始进行回收。

  - JDK5 及以前版本的默认值为 68，即当老年代的空间使用率达到 68%时，会执行一次 CMS 回收。<mark>JDK6 及以上版本默认值为 92%</mark>
  - 如果内存增长缓慢，则可以设置一个稍大的值，大的阀值可以有效降低 CMS 的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁触发老年代串行收集器。因此通过该选项便可以有效降低 Ful1Gc 的执行次数。

- `-XX:+UseCMSCompactAtFullCollection` 用于指定在执行完 Full GC 后对内存空间进行压缩整理，以此避免内存碎片的产生。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。

- `-XX:CMSFullGCsBeforeCompaction` 设置在执行多少次 Full GC 后对内存空间进行压缩整理。

- `-XX:ParallelcMSThreads` 设置 CMS 的线程数量。

  - CMS 默认启动的线程数是（ParallelGCThreads+3）/4，ParallelGCThreads 是年轻代并行收集器的线程数。当 CPU 资源比较紧张时，受到 CMS 收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟糕。

### 小结

HotSpot 有这么多的垃圾回收器，那么如果有人问，Serial GC、Parallel GC、Concurrent Mark Sweep GC 这三个 Gc 有什么不同呢？

请记住以下口令：

- 如果你想要最小化地使用内存和并行开销，请选 Serial GC；
- 如果你想要最大化应用程序的吞吐量，请选 Parallel GC；
- 如果你想要最小化 GC 的中断或停顿时间，请选 CMS GC。

### 13.6.4. JDK 后续版本中 CMS 的变化

JDK9 新特性：CMS 被标记为 Deprecate 了（JEP291）

- 如果对 JDK9 及以上版本的 HotSpot 虚拟机使用参数`-XX: +UseConcMarkSweepGC`来开启 CMS 收集器的话，用户会收到一个警告信息，提示 CMS 未来将会被废弃。

JDK14 新特性：删除 CMS 垃圾回收器（JEP363）

- 移除了 CMS 垃圾收集器，如果在 JDK14 中使用 `-XX:+UseConcMarkSweepGC`的话，JVM 不会报错，只是给出一个 warning 信息，但是不会 exit。JVM 会自动回退以默认 GC 方式启动 JVM

## 13.7. G1 回收器：区域化分代式

**既然我们已经有了前面几个强大的 GC，为什么还要发布 Garbage First（G1）？**

原因就在于应用程序所应对的<mark>业务越来越庞大、复杂，用户越来越多</mark>，没有 GC 就不能保证应用程序正常进行，而经常造成 STW 的 GC 又跟不上实际的需求，所以才会不断地尝试对 GC 进行优化。G1（Garbage-First）垃圾回收器是在 Java7 update4 之后引入的一个新的垃圾回收器，是当今收集器技术发展的最前沿成果之一。

与此同时，为了适应现在<mark>不断扩大的内存和不断增加的处理器数量</mark>，进一步降低暂停时间（pause time），同时兼顾良好的吞吐量。

<mark>官方给 G1 设定的目标是在延迟可控的情况下获得尽可能高的吞吐量，所以才担当起“全功能收集器”的重任与期望。</mark>

**为什么名字叫 Garbage First(G1)呢？**

因为 G1 是一个并行回收器，它把堆内存分割为很多不相关的区域（Region）（物理上不连续的）。使用不同的 Region 来表示 Eden、幸存者 0 区，幸存者 1 区，老年代等。

G1 GC 有计划地避免在整个 Java 堆中进行全区域的垃圾收集。G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，<mark>每次根据允许的收集时间，优先回收价值最大的 Region</mark>。

由于这种方式的侧重点在于回收垃圾最大量的区间（Region），所以我们给 G1 一个名字：垃圾优先（Garbage First）。

G1（Garbage-First）是一款面向服务端应用的垃圾收集器，<mark>主要针对配备多核 CPU 及大容量内存的机器</mark>，以极高概率满足 GC 停顿时间的同时，还兼具高吞吐量的性能特征。

在 JDK1.7 版本正式启用，移除了 Experimenta1 的标识，是<mark>JDK9 以后的默认垃圾回收器</mark>，取代了 CMS 回收器以及 Parallel+Parallel Old 组合。被 Oracle 官方称为“<mark>全功能的垃圾收集器</mark>”。

与此同时，CMS 已经在 JDK9 中被标记为废弃（deprecated）。在 jdk8 中还不是默认的垃圾回收器，需要使用`-XX:+UseG1GC`来启用。

### 13.7.1. G1 回收器的特点（优势）

与其他 GC 收集器相比，G1 使用了全新的分区算法，其特点如下所示：

#### 并行与并发

- 并行性：G1 在回收期间，可以有多个 GC 线程同时工作，有效利用多核计算能力。此时用户线程 STW
- 并发性：G1 拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况

#### 分代收集

- 从分代上看，<mark>G1 依然属于分代型垃圾回收器</mark>，它会区分年轻代和老年代，年轻代依然有 Eden 区和 Survivor 区。但从堆的结构上看，它不要求整个 Eden 区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。
- 将<mark>堆空间分为若干个区域（Region），这些区域中包含了逻辑上的年轻代和老年代</mark>。
- 和之前的各类回收器不同，它同时<mark>兼顾年轻代和老年代</mark>。对比其他回收器，或者工作在年轻代，或者工作在老年代；

![image-20200713215105293](https://img-blog.csdnimg.cn/img_convert/9a71df3a4013da274aa3b28cd95e7d37.png)

![image-20200713215133839](https://img-blog.csdnimg.cn/img_convert/8bcd36541731eb308043eba968b7a828.png)

#### 空间整合

- CMS：“标记-清除”算法、内存碎片、若干次 Gc 后进行一次碎片整理
- G1 将内存划分为一个个的 region。内存的回收是以 region 作为基本单位的。<mark>Region 之间是复制算法</mark>，但整体上实际可看作是<mark>标记-压缩（Mark-Compact）算法</mark>，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次 GC。尤其是当 Java 堆非常大的时候，G1 的优势更加明显。

#### 可预测的停顿时间模型（即：软实时 soft real-time）

这是 G1 相对于 CMS 的另一大优势，G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒。

- 由于分区的原因，G1 可以只选取部分区域进行内存回收，这样缩小了回收的范围，因此对于全局停顿情况的发生也能得到较好的控制。
- G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，<mark>每次根据允许的收集时间，优先回收价值最大的 Region</mark>。保证了 G1 收集器在有限的时间内可以获取尽可能高的收集效率。
- 相比于 CMSGC，G1 未必能做到 CMS 在最好情况下的延时停顿，但是最差情况要好很多。

### 13.7.2. G1 垃圾收集器的缺点

相较于 CMS，G1 还不具备全方位、压倒性优势。比如在用户程序运行过程中，G1 无论是为了垃圾收集产生的内存占用（Footprint）还是程序运行时的额外执行负载（Overload）都要比 CMS 要高。

从经验上来说，在小内存应用上 CMS 的表现大概率会优于 G1，而 G1 在大内存应用上则发挥其优势。平衡点在 6-8GB 之间。

### 13.7.3. G1 回收器的参数设置

- `-XX:+UseG1GC`：手动指定使用 G1 垃圾收集器执行内存回收任务
- `-XX:G1HeapRegionSize` 设置每个 Region 的大小。值是 2 的幂，范围是 1MB 到 32MB 之间，目标是根据最小的 Java 堆大小划分出约 2048 个区域。默认是堆内存的 1/2000。
- `-XX:MaxGCPauseMillis` 设置期望达到的最大 GC 停顿时间指标（JVM 会尽力实现，但不保证达到）。默认值是 200ms（人的平均反应速度）
- `-XX:+ParallelGCThread` 设置 STW 工作线程数的值。最多设置为 8（上面说过 Parallel 回收器的线程计算公式，当 CPU_Count > 8 时，ParallelGCThreads 也会大于 8）
- `-XX:ConcGCThreads` 设置并发标记的线程数。将 n 设置为并行垃圾回收线程数（ParallelGCThreads）的 1/4 左右。
- `-XX:InitiatingHeapOccupancyPercent` 设置触发并发 GC 周期的 Java 堆占用率阈值。超过此值，就触发 GC。默认值是 45。

### 13.7.4. G1 收集器的常见操作步骤

G1 的设计原则就是简化 JVM 性能调优，开发人员只需要简单的三步即可完成调优：

- 第一步：开启 G1 垃圾收集器
- 第二步：设置堆的最大内存
- 第三步：设置最大的停顿时间

G1 中提供了三种垃圾回收模式：Young GC、Mixed GC 和 Full GC，在不同的条件下被触发。

### 13.7.5. G1 收集器的适用场景

面向服务端应用，针对具有大内存、多处理器的机器。（在普通大小的堆里表现并不惊喜）

最主要的应用是需要低 GC 延迟，并具有大堆的应用程序提供解决方案；如：在堆大小约 6GB 或更大时，可预测的暂停时间可以低于 0.5 秒；（G1 通过每次只清理一部分而不是全部的 Region 的增量式清理来保证每次 GC 停顿时间不会过长）。

用来替换掉 JDK1.5 中的 CMS 收集器；在下面的情况时，使用 G1 可能比 CMS 好：

- 超过 50%的 Java 堆被活动数据占用；
- 对象分配频率或年代提升频率变化很大；
- GC 停顿时间过长（长于 0.5 至 1 秒）

HotSpot 垃圾收集器里，除了 G1 以外，其他的垃圾收集器使用内置的 JVM 线程执行 GC 的多线程操作，而 G1 GC 可以采用应用线程承担后台运行的 GC 工作，即当 JVM 的 GC 线程处理速度慢时，系统会调用应用程序线程帮助加速垃圾回收过程。

### 13.7.6. 分区 Region：化整为零

使用 G1 收集器时，它将整个 Java 堆划分成约 2048 个大小相同的独立 Region 块，每个 Region 块大小根据堆空间的实际大小而定，整体被控制在 1MB 到 32MB 之间，且为 2 的 N 次幂，即 1MB，2MB，4MB，8MB，16MB，32MB。可以通过`-XX:G1HeapRegionSize`设定。<mark>所有的 Region 大小相同，且在 JVM 生命周期内不会被改变。</mark>

虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分 Region（不需要连续）的集合。通过 Region 的动态分配方式实现逻辑上的连续。

![image-20200713223244886](https://img-blog.csdnimg.cn/img_convert/74c611464ae6cdae3cbfffaef213cfd5.png)

一个 region 有可能属于 Eden，Survivor 或者 Old/Tenured 内存区域。但是一个 region 只可能属于一个角色。图中的 E 表示该 region 属于 Eden 内存区域，S 表示属于 survivor 内存区域，O 表示属于 Old 内存区域。图中空白的表示未使用的内存空间。

G1 垃圾收集器还增加了一种新的内存区域，叫做 Humongous 内存区域，如图中的 H 块。主要用于存储大对象，如果超过 1.5 个 region，就放到 H。

设置 H 的原因：对于堆中的对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象就会对垃圾收集器造成负面影响。为了解决这个问题，G1 划分了一个 Humongous 区，它用来专门存放大对象。<mark>如果一个 H 区装不下一个大对象，那么 G1 会寻找连续的 H 区来存储。</mark>为了能找到连续的 H 区，有时候不得不启动 Full GC。G1 的大多数行为都把 H 区作为老年代的一部分来看待。

每个 Region 都是通过指针碰撞来分配空间

![image-20200713223509993](https://img-blog.csdnimg.cn/img_convert/049520c5a004b63f750e04d5362a2992.png)

### 13.7.7. G1 垃圾回收器的回收过程

G1GC 的垃圾回收过程主要包括如下三个环节：

- 年轻代 GC（Young GC）

- 老年代并发标记过程（Concurrent Marking）

- 混合回收（Mixed GC）

  （如果需要，单线程、独占式、高强度的 Full GC 还是继续存在的。它针对 GC 的评估失败提供了一种失败保护机制，即强力回收。）

![image-20200713224113996](https://img-blog.csdnimg.cn/img_convert/e8f79c3cdeb0969981703f7a026cf581.png)

顺时针，Young gc -> Young gc + Concurrent mark->Mixed GC 顺序，进行垃圾回收。

应用程序分配内存，<mark>当年轻代的 Eden 区用尽时开始年轻代回收过程</mark>；G1 的年轻代收集阶段是一个<mark>并行的独占式</mark>收集器。在年轻代回收期，G1GC 暂停所有应用程序线程，启动多线程执行年轻代回收。然后<mark>从年轻代区间移动存活对象到 Survivor 区间或者老年区间，也有可能是两个区间都会涉及</mark>。

当堆内存使用达到一定值（默认 45%）时，开始老年代并发标记过程。

标记完成马上开始混合回收过程。对于一个混合回收期，G1 GC 从老年区间移动存活对象到空闲区间，这些空闲区间也就成为了老年代的一部分。和年轻代不同，老年代的 G1 回收器和其他 GC 不同，<mark>G1 的老年代回收器不需要整个老年代被回收，一次只需要扫描/回收一小部分老年代的 Region 就可以了</mark>。同时，这个老年代 Region 是和年轻代一起被回收的。

举个例子：一个 Web 服务器，Java 进程最大堆内存为 4G，每分钟响应 1500 个请求，每 45 秒钟会新分配大约 2G 的内存。G1 会每 45 秒钟进行一次年轻代回收，每 31 个小时整个堆的使用率会达到 45%，会开始老年代并发标记过程，标记完成后开始四到五次的混合回收。

### 13.7.8. Remembered Set

- 一个对象被不同区域引用的问题

- 一个 Region 不可能是孤立的，一个 Region 中的对象可能被其他任意 Region 中对象引用，判断对象存活时，是否需要扫描整个 Java 堆才能保证准确？

- 在其他的分代收集器，也存在这样的问题（而 G1 更突出）回收新生代也不得不同时扫描老年代？
- 这样的话会降低 MinorGC 的效率；

**解决方法：**

无论 G1 还是其他分代收集器，JVM 都是使用 Remembered Set 来避免全局扫描：

<mark>每个 Region 都有一个对应的 Remembered Set；</mark>

每次 Reference 类型数据写操作时，都会产生一个 Write Barrier 暂时中断操作；

然后检查将要写入的引用指向的对象是否和该 Reference 类型数据在不同的 Region（其他收集器：检查老年代对象是否引用了新生代对象）；

如果不同，通过 CardTable 把相关引用信息记录到引用指向对象的所在 Region 对应的 Remembered Set 中；

当进行垃圾收集时，在 GC 根节点的枚举范围加入 Remembered Set；就可以保证不进行全局扫描，也不会有遗漏。

![image-20200713224716715](https://img-blog.csdnimg.cn/img_convert/c97de4bdada1da5a9fdd344692ed0957.png)

### 13.7.9. G1 回收过程一：年轻代 GC

JVM 启动时，G1 先准备好 Eden 区，程序在运行过程中不断创建对象到 Eden 区，当 Eden 空间耗尽时，G1 会启动一次年轻代垃圾回收过程。

年轻代垃圾回收只会回收 Eden 区和 Survivor 区。

首先 G1 停止应用程序的执行（Stop-The-World），G1 创建回收集（Collection Set），回收集是指需要被回收的内存分段的集合，年轻代回收过程的回收集包含年轻代 Eden 区和 Survivor 区所有的内存分段。

![image-20200713225100632](https://img-blog.csdnimg.cn/img_convert/f29fa4dfe3abf4a77be06fdf3378aecf.png)

然后开始如下回收过程：

1. <mark>第一阶段，扫描根</mark>。根是指 static 变量指向的对象，正在执行的方法调用链条上的局部变量等。根引用连同 RSet 记录的外部引用作为扫描存活对象的入口。
2. <mark>第二阶段，更新 RSet</mark>。处理 dirty card queue（见备注）中的 card，更新 RSet。此阶段完成后，<mark>RSet 可以准确的反映老年代对所在的内存分段中对象的引用</mark>。
3. <mark>第三阶段，处理 RSet</mark>。识别被老年代对象指向的 Eden 中的对象，这些被指向的 Eden 中的对象被认为是存活的对象。
4. <mark>第四阶段，复制对象</mark>。此阶段，对象树被遍历，Eden 区内存段中存活的对象会被复制到 Survivor 区中空的内存分段，Survivor 区内存段中存活的对象如果年龄未达阈值，年龄会加 1，达到阀值会被会被复制到 Old 区中空的内存分段。如果 Survivor 空间不够，Eden 空间的部分数据会直接晋升到老年代空间。
5. <mark>第五阶段，处理引用</mark>。处理 Soft，Weak，Phantom，Final，JNI Weak 等引用。最终 Eden 空间的数据为空，GC 停止工作，而目标内存中的对象都是连续存储的，没有碎片，所以复制过程可以达到内存整理的效果，减少碎片。

### 13.7.10. G1 回收过程二：并发标记过程

1. <mark>初始标记阶段</mark>：标记从根节点直接可达的对象。这个阶段是 STW 的，并且会触发一次年轻代 GC。
2. <mark>根区域扫描（Root Region Scanning）</mark>：G1 GC 扫描 Survivor 区直接可达的老年代区域对象，并标记被引用的对象。这一过程必须在 YoungGC 之前完成。
3. <mark>并发标记（Concurrent Marking）</mark>：在整个堆中进行并发标记（和应用程序并发执行），此过程可能被 YoungGC 中断。在并发标记阶段，<mark>若发现区域对象中的所有对象都是垃圾，那这个区域会被立即回收</mark>。同时，并发标记过程中，会计算每个区域的对象活性（区域中存活对象的比例）。
4. <mark>再次标记（Remark）</mark>：由于应用程序持续进行，需要修正上一次的标记结果。是 STW 的。G1 中采用了比 CMS 更快的初始快照算法：snapshot-at-the-beginning（SATB）。
5. <mark>独占清理（cleanup，STW）</mark>：计算各个区域的存活对象和 GC 回收比例，并进行排序，识别可以混合回收的区域。为下阶段做铺垫。是 STW 的。这个阶段并不会实际上去做垃圾的收集
6. <mark>并发清理阶段</mark>：识别并清理完全空闲的区域。

### 13.7.11. G1 回收过程三：混合回收

当越来越多的对象晋升到老年代 o1d region 时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即 Mixed GC，该算法并不是一个 Old GC，除了回收整个 Young Region，还会回收一部分的 Old Region。这里需要注意：<mark>是一部分老年代，而不是全部老年代</mark>。可以选择哪些 Old Region 进行收集，从而可以对垃圾回收的耗时时间进行控制。也要注意的是 Mixed GC 并不是 Full GC。

![image-20200713225810871](https://img-blog.csdnimg.cn/img_convert/766b882cba7e709202005a3baeb596d0.png)

并发标记结束以后，老年代中百分百为垃圾的内存分段被回收了，部分为垃圾的内存分段被计算了出来。默认情况下，这些老年代的内存分段会分 8 次（可以通过`-XX:G1MixedGCCountTarget`设置）被回收

混合回收的回收集（Collection Set）包括八分之一的老年代内存分段，Eden 区内存分段，Survivor 区内存分段。混合回收的算法和年轻代回收的算法完全一样，只是回收集多了老年代的内存分段。具体过程请参考上面的年轻代回收过程。

由于老年代中的内存分段默认分 8 次回收，G1 会优先回收垃圾多的内存分段。垃圾占内存分段比例越高的，越会被先回收。并且有一个阈值会决定内存分段是否被回收，`-XX:G1MixedGCLiveThresholdPercent`，默认为 65%，意思是垃圾占内存分段比例要达到 65%才会被回收。如果垃圾占比太低，意味着存活的对象占比高，在复制的时候会花费更多的时间。

混合回收并不一定要进行 8 次。有一个阈值`-XX:G1HeapWastePercent`，默认值为 10%，意思是允许整个堆内存中有 10%的空间被浪费，意味着如果发现可以回收的垃圾占堆内存的比例低于 10%，则不再进行混合回收。因为 GC 会花费很多的时间但是回收到的内存却很少。

### 13.7.12. G1 回收可选的过程四：Full GC

G1 的初衷就是要避免 Full GC 的出现。但是如果上述方式不能正常工作，G1 会停止应用程序的执行（Stop-The-World），使用单线程的内存回收算法进行垃圾回收，性能会非常差，应用程序停顿时间会很长。

要避免 Full GC 的发生，一旦发生需要进行调整。什么时候会发生 Full GC 呢？比如<mark>堆内存太小</mark>，当 G1 在复制存活对象的时候没有空的内存分段可用，则会回退到 Full GC，这种情况可以通过增大内存解决。

导致 G1 Full GC 的原因可能有两个：

- Evacuation 的时候没有足够的 to-space 来存放晋升的对象；
- 并发处理过程完成之前空间耗尽。

### 13.7.13. 补充

从 Oracle 官方透露出来的信息可获知，回收阶段（Evacuation）其实本也有想过设计成与用户程序一起并发执行，但这件事情做起来比较复杂，考虑到 G1 只是回一部分 Region，停顿时间是用户可控制的，所以并不迫切去实现，而<mark>选择把这个特性放到了 G1 之后出现的低延迟垃圾收集器（即 ZGC）中</mark>。另外，还考虑到 G1 不是仅仅面向低延迟，停顿用户线程能够最大幅度提高垃圾收集效率，为了保证吞吐量所以才选择了完全暂停用户线程的实现方案。

### 13.7.14. G1 回收器优化建议

年轻代大小

- 避免使用`-Xmn`或`-XX:NewRatio`等相关选项显式设置年轻代大小
- 固定年轻代的大小会覆盖暂停时间目标

暂停时间目标不要太过严苛

- G1 GC 的吞吐量目标是 90%的应用程序时间和 10%的垃圾回收时间
- 评估 G1 GC 的吞吐量时，暂停时间目标不要太严苛。目标太过严苛表示你愿意承受更多的垃圾回收开销，而这些会直接影响到吞吐量。

## 13.8. 垃圾回收器总结

### 13.8.1. 7 种经典垃圾回收器总结

截止 JDK1.8，一共有 7 款不同的垃圾收集器。每一款的垃圾收集器都有不同的特点，在具体使用的时候，需要根据具体的情况选用不同的垃圾收集器。

| 垃圾收集器   | 分类           | 作用位置             | 使用算法                | 特点         | 适用场景                                 |
| :----------- | :------------- | :------------------- | :---------------------- | :----------- | :--------------------------------------- |
| Serial       | 串行运行       | 作用于新生代         | 复制算法                | 响应速度优先 | 适用于单 CPU 环境下的 client 模式        |
| ParNew       | 并行运行       | 作用于新生代         | 复制算法                | 响应速度优先 | 多 CPU 环境 Server 模式下与 CMS 配合使用 |
| Parallel     | 并行运行       | 作用于新生代         | 复制算法                | 吞吐量优先   | 适用于后台运算而不需要太多交互的场景     |
| Serial Old   | 串行运行       | 作用于老年代         | 标记-压缩算法           | 响应速度优先 | 适用于单 CPU 环境下的 Client 模式        |
| Parallel Old | 并行运行       | 作用于老年代         | 标记-压缩算法           | 吞吐量优先   | 适用于后台运算而不需要太多交互的场景     |
| CMS          | 并发运行       | 作用于老年代         | 标记-清除算法           | 响应速度优先 | 适用于互联网或 B／S 业务                 |
| G1           | 并发、并行运行 | 作用于新生代、老年代 | 标记-压缩算法、复制算法 | 响应速度优先 | 面向服务端应用                           |

GC 发展阶段：Serial => Parallel（并行）=> CMS（并发）=> G1 => ZGC

### 13.8.2. 垃圾回收器组合

不同厂商、不同版本的虚拟机实现差距比较大。HotSpot 虚拟机在 JDK7/8 后所有收集器及组合如下图

![image-20200714080151020](https://img-blog.csdnimg.cn/img_convert/5e57edaa3ec7295424480c67daee499e.png)

1. 两个收集器间有连线，表明它们可以搭配使用：Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1;

2. 其中 Serial Old 作为 CMS 出现＂`Concurrent Mode Failure`＂失败的后备预案。

3. （红色虚线）由于维护和兼容性测试的成本，在 JDK 8 时将 Serial ＋ CMS、ParNew ＋ Serial old 这两个组合声明为 Deprecated（JEP 173），并在 JDK 9 中

完全取消了这些组合的支持（JEP214），即：移除。

4. （绿色虚线）JDK 14 中：弃用 ParallelScavenge 和 SeriaOold GC 组合(JEP 366)

5. （绿色虚框）JDK 14 中：删除 CMS 垃圾回收器（JEP 363）

### 13.8.3. 怎么选择垃圾回收器

Java 垃圾收集器的配置对于 JVM 优化来说是一个很重要的选择，选择合适的垃圾收集器可以让 JVM 的性能有一个很大的提升。

怎么选择垃圾收集器？

1. 优先调整堆的大小让 JVM 自适应完成。

2. 如果内存小于 100M，使用串行收集器

3. 如果是单核、单机程序，并且没有停顿时间的要求，串行收集器

4. 如果是多 CPU、需要高吞吐量、允许停顿时间超过 1 秒，选择并行或者 JVM 自己选择

5. 如果是多 CPU、追求低停顿时间，需快速响应（比如延迟不能超过 1 秒，如互联网应用），使用并发收集器

   官方推荐 G1，性能高。<mark>现在互联网的项目，基本都是使用 G1</mark>。

最后需要明确一个观点：

1. 没有最好的收集器，更没有万能的收集
2. 调优永远是针对特定场景、特定需求，不存在一劳永逸的收集器

**面试**

对于垃圾收集，面试官可以循序渐进从理论、实践各种角度深入，也未必是要求面试者什么都懂。但如果你懂得原理，一定会成为面试中的加分项。 这里较通用、基础性的部分如下：

- 垃圾收集的算法有哪些？如何判断一个对象是否可以回收？

- 垃圾收集器工作的基本流程。

另外，大家需要多关注垃圾回收器这一章的各种常用的参数

## 13.9. GC 日志分析

通过阅读 Gc 日志，我们可以了解 Java 虚拟机内存分配与回收策略。 内存分配与垃圾回收的参数列表

- `-XX:+PrintGC` 输出 GC 日志。类似：`-verbose:gc`
- `-XX:+PrintGCDetails` 输出 GC 的详细日志
- `-XX:+PrintGCTimestamps` 输出 GC 的时间戳（以基准时间的形式）
- `-XX:+PrintGCDatestamps` 输出 GcC 的时间戳（以日期的形式，如 2013-05-04T21：53：59.234+0800）
- `-XX:+PrintHeapAtGC` 在进行 GC 的前后打印出堆的信息
- `-Xloggc:../logs/gc.log` 日志文件的输出路径

打开 GC 日志

```shell
-verbose:gc
```

这个只会显示总的 GC 堆的变化，如下：

```java
[GC (Allocation Failure) 80832K->19298K(227840K),0.0084018 secs]
[GC (Metadata GC Threshold) 109499K->21465K(228352K),0.0184066 secs]
[Full GC (Metadata GC Threshold) 21465K->16716K(201728K),0.0619261 secs]
```

参数解析

```java
GC、Full GC：GC的类型，GC只在新生代上进行，Full GC包括永生代，新生代，老年代。
Allocation Failure：GC发生的原因。
80832K->19298K：堆在GC前的大小和GC后的大小。
228840k：现在的堆大小。
0.0084018 secs：GC持续的时间。
```

打开 GC 日志

```shell
-verbose:gc -XX:+PrintGCDetails
```

输入信息如下

```java
[GC (Allocation Failure) [PSYoungGen:70640K->10116K(141312K)] 80541K->20017K(227328K),0.0172573 secs] [Times:user=0.03 sys=0.00,real=0.02 secs]
[GC (Metadata GC Threshold) [PSYoungGen:98859K->8154K(142336K)] 108760K->21261K(228352K),0.0151573 secs] [Times:user=0.00 sys=0.01,real=0.02 secs]
[Full GC (Metadata GC Threshold)[PSYoungGen:8154K->0K(142336K)]
[ParOldGen:13107K->16809K(62464K)] 21261K->16809K(204800K),[Metaspace:20599K->20599K(1067008K)],0.0639732 secs]
[Times:user=0.14 sys=0.00,real=0.06 secs]
```

参数解析

```java
GC，Full FC：同样是GC的类型
Allocation Failure：GC原因
PSYoungGen：使用了Parallel Scavenge并行垃圾收集器的新生代GC前后大小的变化
ParOldGen：使用了Parallel Old并行垃圾收集器的老年代GC前后大小的变化
Metaspace： 元数据区GC前后大小的变化，JDK1.8中引入了元数据区以替代永久代
xxx secs：指GC花费的时间
Times：user：指的是垃圾收集器花费的所有CPU时间，sys：花费在等待系统调用或系统事件的时间，real：GC从开始到结束的时间，包括其他进程占用时间片的实际时间。
```

打开 GC 日志

```shell
-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimestamps -XX:+PrintGCDatestamps
```

输入信息如下

```java
2019-09-24T22:15:24.518+0800: 3.287: [GC (Allocation Failure) [PSYoungGen:136162K->5113K(136192K)] 141425K->17632K(222208K),0.0248249 secs] [Times:user=0.05 sys=0.00,real=0.03 secs]

2019-09-24T22:15:25.559+0800: 4.329: [GC (Metadata GC Threshold) [PSYoungGen:97578K->10068K(274944K)] 110096K->22658K(360960K),0.0094071 secs] [Times: user=0.00 sys=0.00,real=0.01 secs]

2019-09-24T22:15:25.569+0800: 4.338: [Full GC (Metadata GC Threshold) [PSYoungGen:10068K->0K(274944K)][ParoldGen:12590K->13564K(56320K)] 22658K->13564K(331264K),[Metaspace:20590K->20590K(1067008K)],0.0494875 secs] [Times: user=0.17 sys=0.02,real=0.05 secs]
```

说明：带上了日期和实践

如果想把 GC 日志存到文件的话，是下面的参数：

```shell
-Xloggc:/path/to/gc.log
```

**日志补充说明**

- "`[GC`"和"`[Full GC`"说明了这次垃圾收集的停顿类型，如果有"Full"则说明 GC 发生了"Stop The World"

- 使用 Serial 收集器在新生代的名字是 Default New Generation，因此显示的是"`[DefNew`"

- 使用 ParNew 收集器在新生代的名字会变成"`[ParNew`"，意思是"Parallel New Generation"

- 使用 Parallel scavenge 收集器在新生代的名字是”`[PSYoungGen`"

- 老年代的收集和新生代道理一样，名字也是收集器决定的

- 使用 G1 收集器的话，会显示为"garbage-first heap"

- <mark>Allocation Failure</mark>

  表明本次引起 GC 的原因是因为在年轻代中没有足够的空间能够存储新的数据了。

- <mark>[PSYoungGen：5986K->696K(8704K) ] 5986K->704K(9216K)</mark>

  中括号内：GC 回收前年轻代大小，回收后大小，（年轻代总大小）

  括号外：GC 回收前年轻代和老年代大小，回收后大小，（年轻代和老年代总大小）

- <mark>user 代表用户态回收耗时，sys 内核态回收耗时，rea 实际耗时</mark>。由于多核的原因，时间总和可能会超过 real 时间

```java
Heap（堆）
PSYoungGen（Parallel Scavenge收集器新生代）total 9216K，used 6234K [0x00000000ff600000,0x0000000100000000,0x0000000100000000)
eden space（堆中的Eden区默认占比是8）8192K，768 used [0x00000000ff600000,0x00000000ffc16b08,0x00000000ffe00000)
from space（堆中的Survivor，这里是From Survivor区默认占比是1）1024K， 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
to space（堆中的Survivor，这里是to Survivor区默认占比是1，需要先了解一下堆的分配策略）1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)

ParOldGen（老年代总大小和使用大小）total 10240K， used 7001K ［0x00000000fec00000,0x00000000ff600000,0x00000000ff600000)
object space（显示个使用百分比）10240K，688 used [0x00000000fec00000,0x00000000ff2d6630,0x00000000ff600000)

PSPermGen（永久代总大小和使用大小）total 21504K， used 4949K [0x00000000f9a00000,0x00000000faf00000,0x00000000fec00000)
object space（显示个使用百分比，自己能算出来）21504K， 238 used [0x00000000f9a00000,0x00000000f9ed55e0,0x00000000faf00000)
```

### Minor GC 日志

![image-20200714082555688](https://img-blog.csdnimg.cn/img_convert/9364561fbb81a0e2f9aedc45a383972f.png)

### Full GC 日志

![image-20210512194815354](https://img-blog.csdnimg.cn/img_convert/502793e725122b958f2861932e5ef9c1.png)

**举例**

```java
private static final int _1MB = 1024 * 1024;public static void testAllocation() {    byte [] allocation1, allocation2, allocation3, allocation4;    allocation1 = new byte[2 *_1MB];    allocation2 = new byte[2 *_1MB];    allocation3 = new byte[2 *_1MB];    allocation4 = new byte[4 *_1MB];}public static void main(String[] args) {    testAllocation();}
```

设置 JVM 参数

```shell
-Xms10m -Xmx10m -XX:+PrintGCDetails
```

**图示**

![image-20200714083332238](https://img-blog.csdnimg.cn/img_convert/8dada6b73786eb693975593db10ce825.png)

![image-20200714083526790](https://img-blog.csdnimg.cn/img_convert/78f6489bd1a3b74e14ee4a95392df8c2.png)

可以用一些工具去分析这些 GC 日志

常用的日志分析工具有：GCViewer、GCEasy、GCHisto、GCLogViewer、Hpjmeter、garbagecat 等

## 13.X. 垃圾回收器的新发展

GC 仍然处于飞速发展之中，目前的默认选项<mark>G1 GC 在不断的进行改进</mark>，很多我们原来认为的缺点，例如串行的 Fu11GC、Card Table 扫描的低效等，都已经被大幅改进，例如，JDK10 以后，Fu11GC 已经是并行运行，在很多场景下，其表现还略优于 ParallelGC 的并行 Ful1GC 实现。

即使是 Serial GC，虽然比较古老，但是简单的设计和实现未必就是过时的，它本身的开销，不管是 GC 相关数据结构的开销，还是线程的开销，都是非常小的，所以随着云计算的兴起，<mark>在 Serverless 等新的应用场景下，Serial GC 找到了新的舞台</mark>。

比较不幸的是 CMSGC，因为其算法的理论缺陷等原因，虽然现在还有非常大的用户群体，但在 JDK9 中已经被标记为废弃，并在 JDK14 版本中移除

### 13.X.1. JDK11 新特性

Epsilon:A No-Op GarbageCollector（Epsilon 垃圾回收器，"No-Op（无操作）"回收器）[http://openidk.iava.net/jeps/318](http://openidk.iava.net/jeps/318)

ZGC:A Scalable Low-Latency Garbage Collector（Experimental）（ZGC：可伸缩的低延迟垃圾回收器，处于实验性阶段）[http://openidk.iava.net/jeps/333](http://openidk.iava.net/jeps/318)

![image-20210512195426194](https://img-blog.csdnimg.cn/img_convert/4aaef244379d75ba838e10d4178b2960.png)

现在 G1 回收器已成为默认回收器好几年了。

我们还看到了引入了两个新的收集器：ZGC（JDK11 出现）和 Shenandoah（Open JDK12）。主打特点：低停顿时间

![image-20210512195528695](https://img-blog.csdnimg.cn/img_convert/00920e4ae2b2c80a8016e6d8f4632545.png)

### 13.X.2. Open JDK12 的 Shenandoash GC

<mark>Open JDK12 的 Shenandoash GC：低停顿时间的 GC（实验性）</mark>

<mark>Shenandoah，无疑是众多 GC 中最孤独的一个。</mark>是第一款不由 oracle 公司团队领导开发的 Hotspot 垃圾收集器。不可避免的<mark>受到官方的排挤</mark>。比如号称 OpenJDK 和 OracleJDK 没有区别的 Oracle 公司仍拒绝在 OracleJDK12 中支持 Shenandoah。

Shenandoah 垃圾回收器最初由 RedHat 进行的一项垃圾收集器研究项目 Pauseless GC 的实现，<mark>旨在针对 JVM 上的内存回收实现低停顿的需求</mark>.。在 2014 年贡献给 OpenJDK。

Red Hat 研发 Shenandoah 团队对外宣称，<mark>Shenandoah 垃圾回收器的暂停时间与堆大小无关，这意味着无论将堆设置为 200MB 还是 200GB，99.9%的目标都可以把垃圾收集的停顿时间限制在十毫秒以内。</mark>不过实际使用性能将取决于实际工作堆的大小和工作负载。

![image-20200714090608807](https://img-blog.csdnimg.cn/img_convert/01f566c0db04f0e475db05addd94259f.png)

这是 RedHat 在 2016 年发表的论文数据，测试内容是使用 Es 对 200GB 的维基百科数据进行索引。从结果看：

- 停顿时间比其他几款收集器确实有了质的飞跃，但也未实现最大停顿时间控制在十毫秒以内的目标。
- 而吞吐量方面出现了明显的下降，总运行时间是所有测试收集器里最长的。

总结

- Shenandoah GC 的弱项：高运行负担下的吞吐量下降。
- Shenandoah GC 的强项：低延迟时间。
- Shenandoah GC 的工作过程大致分为九个阶段，这里就不再赘述。在之前 Java12 新特性视频里有过介绍。

【Java12 新特性地址】

[http://www.atguigu.com/download_detail.shtml?v=222](http://www.atguigu.com/download_detail.shtml?v=222)

或

[https://www.bilibili.com/video/BV1jJ411M7kQ?from=search&seid=12339069673726242866](https://www.bilibili.com/video/BV1jJ411M7kQ?from=search&seid=12339069673726242866)

### 13.X.3. 令人震惊、革命性的 ZGC

官方地址：[https://docs.oracle.com/en/java/javase/12/gctuning/](https://docs.oracle.com/en/java/javase/12/gctuning/)

![image-20210512200236647](https://img-blog.csdnimg.cn/img_convert/f78eebadbabf7450ded984f0f1e3a405.png)

ZGC 与 Shenandoah 目标高度相似，<mark>在尽可能对吞吐量影响不大的前提下，实现在任意堆内存大小下都可以把垃圾收集的停颇时间限制在十毫秒以内的低延迟。</mark>

《深入理解 Java 虚拟机》一书中这样定义 ZGC：ZGC 收集器是一款基于 Region 内存布局的，（暂时）不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现<mark>可并发的标记-压缩算法</mark>的，以<mark>低延迟为首要目标</mark>的一款垃圾收集器。

ZGC 的工作过程可以分为 4 个阶段：<mark>并发标记 - 并发预备重分配 - 并发重分配 - 并发重映射</mark> 等。

ZGC 几乎在所有地方并发执行的，除了初始标记的是 STw 的。所以停顿时间几乎就耗费在初始标记上，这部分的实际时间是非常少的。

测试数据：

![image-20200714091201073](https://img-blog.csdnimg.cn/img_convert/b08828b548a255493e934c6fe3308e50.png)

![image-20200714091401511](https://img-blog.csdnimg.cn/img_convert/4e648c4e37d84f5d83d965b55f6bc5f9.png)

在 ZGC 的强项停顿时间测试上，它毫不留情的将 Parallel、G1 拉开了两个数量级的差距。无论平均停顿、95％停顿、99％停顿、99.9％停顿，还是最大停顿时间，ZGC 都能毫不费劲控制在 10 毫秒以内。

虽然 ZGC 还在试验状态，没有完成所有特性，但此时性能已经相当亮眼，用“令人震惊、革命性”来形容，不为过。 <mark>未来将在服务端、大内存、低延迟应用的首选垃圾收集器。</mark>

![image-20200714093243028](https://img-blog.csdnimg.cn/img_convert/59b014ca2eeda5332fd40a4a2356f883.png)

<mark>JEP 364：ZGC 应用在 macos 上</mark>

<mark>JEP 365：ZGC 应用在 Windows 上</mark>

JDK14 之前，ZGC 仅 Linux 才支持。

尽管许多使用 zGc 的用户都使用类 Linux 的环境，但在 Windows 和 macos 上，人们也需要 ZGC 进行开发部署和测试。许多桌面应用也可以从 ZGC 中受益。因此，ZGC 特性被移植到了 Windows 和 macos 上。

现在 mac 或 Windows 上也能使用 zGC 了，示例如下：

```shell
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC
```

### 13.X.4. 其他垃圾回收器：AliGC

AliGC 是阿里巴巴 JVM 团队基于 G1 算法，面向大堆（LargeHeap）应用场景。指定场景下的对比：

![image-20200714093604012](https://img-blog.csdnimg.cn/img_convert/835fff5c3a7884de2a1d564b7f332900.png)

当然，其它厂商也提供了各种别具一格的 GC 实现，例如比较有名的低延迟 GC：Zing，有兴趣可以参考提供的链接 [https://www.infoq.com/articles/azul_gc_in_detail](https://www.infoq.com/articles/azul_gc_in_detail)
