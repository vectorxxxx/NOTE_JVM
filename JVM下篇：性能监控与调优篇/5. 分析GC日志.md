> 笔记来源：[尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）](https://www.bilibili.com/video/BV1PJ411n7xZ "尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）")
>
> 同步更新：https://gitee.com/vectorx/NOTE_JVM
>
> https://codechina.csdn.net/qq_35925558/NOTE_JVM
>
> https://github.com/uxiahnan/NOTE_JVM

[toc]

# 5. 分析GC日志

## 5.1. GC分类

针对HotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（Full GC）

- 部分收集（Partial GC）：不是完整收集整个Java堆的垃圾收集。其中又分为：
  - 新生代收集（Minor GC / Young GC）：只是新生代（Eden / S0, S1）的垃圾收集
  - 老年代收集（Major GC / Old GC）：只是老年代的垃圾收集。目前，只有CMS GC会有单独收集老年代的行为。<mark>注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。</mark>

- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。目前，只有G1 GC会有这种行为
- 整堆收集（Full GC）：收集整个java堆和方法区的垃圾收集。

## 5.2. GC日志分类

**MinorGC**

MinorGC（或young GC或YGC）日志：

```java
[GC (Allocation Failure) [PSYoungGen: 31744K->2192K (36864K) ] 31744K->2200K (121856K), 0.0139308 secs] [Times: user=0.05 sys=0.01, real=0.01 secs]
```

![image-20210506202126562](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210506202127.png)

![image-20210506202156090](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210506202158.png)

**FullGC**

```java
[Full GC (Metadata GC Threshold) [PSYoungGen: 5104K->0K (132096K) ] [Par01dGen: 416K->5453K (50176K) ]5520K->5453K (182272K), [Metaspace: 20637K->20637K (1067008K) ], 0.0245883 secs] [Times: user=0.06 sys=0.00, real=0.02 secs]
```

![image-20210506202330868](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210506202331.png)

![image-20210506202349072](https://gitee.com/vectorx/ImageCloud/raw/master/img/20210506202350.png)

## 5.3. GC日志结构剖析

**透过日志看垃圾收集器**

- Serial收集器：新生代显示 "[DefNew"，即 Default New Generation

- ParNew收集器：新生代显示 "[ParNew"，即 Parallel New Generation

- Parallel Scavenge收集器：新生代显示"[PSYoungGen"，JDK1.7使用的即PSYoungGen

- Parallel Old收集器：老年代显示"[ParoldGen"

- G1收集器：显示”garbage-first heap“

**透过日志看GC原因**

- Allocation Failure：表明本次引起GC的原因是因为新生代中没有足够的区域存放需要分配的数据
- Metadata GCThreshold：Metaspace区不够用了
- FErgonomics：JVM自适应调整导致的GC
- System：调用了System.gc()方法

**透过日志看GC前后情况**

通过图示，我们可以发现GC日志格式的规律一般都是：GC前内存占用-＞GC后内存占用（该区域内存总大小）

```java
[PSYoungGen: 5986K->696K (8704K) ] 5986K->704K (9216K)
```

- 中括号内：GC回收前年轻代堆大小，回收后大小，（年轻代堆总大小）

- 括号外：GC回收前年轻代和老年代大小，回收后大小，（年轻代和老年代总大小）

<mark>注意</mark>：Minor GC堆内存总容量 = 9/10 年轻代 + 老年代。原因是Survivor区只计算from部分，而JVM默认年轻代中Eden区和Survivor区的比例关系，Eden:S0:S1=8:1:1。

**透过日志看GC时间**

GC日志中有三个时间：user，sys和real

- user：进程执行用户态代码（核心之外）所使用的时间。这是执行此进程所使用的实际CPU 时间，其他进程和此进程阻塞的时间并不包括在内。在垃圾收集的情况下，表示GC线程执行所使用的 CPU 总时间。
- sys：进程在内核态消耗的 CPU 时间，即在内核执行系统调用或等待系统事件所使用的CPU 时间
- real：程序从开始到结束所用的时钟时间。这个时间包括其他进程使用的时间片和进程阻塞的时间（比如等待 I/O 完成）。对于并行gc，这个数字应该接近（用户时间＋系统时间）除以垃圾收集器使用的线程数。

由于多核的原因，一般的GC事件中，real time是小于sys time＋user time的，因为一般是多个线程并发的去做GC，所以real time是要小于sys＋user time的。如果real＞sys＋user的话，则你的应用可能存在下列问题：IO负载非常重或CPU不够用。

## 5.4. GC日志分析工具

**GCEasy**

GCEasy是一款在线的GC日志分析器，可以通过GC日志分析进行内存泄露检测、GC暂停原因分析、JVM配置建议优化等功能，大多数功能是免费的。

官网地址：[https://gceasy.io/](https://gceasy.io/)

**GCViewer**

GCViewer是一款离线的GC日志分析器，用于可视化Java VM选项 -verbose:gc 和 .NET生成的数据 -Xloggc:&lt;file&gt;。还可以计算与垃圾回收相关的性能指标（吞吐量、累积的暂停、最长的暂停等）。当通过更改世代大小或设置初始堆大小来调整特定应用程序的垃圾回收时，此功能非常有用。

源码下载：[https://github.com/chewiebug/GCViewer](https://github.com/chewiebug/GCViewer)

运行版本下载：[https://github.com/chewiebug/GCViewer/wiki/Changelog](https://github.com/chewiebug/GCViewer/wiki/Changelog)

**GChisto**

- 官网上没有下载的地方，需要自己从SVN上拉下来编译
- 不过这个工具似乎没怎么维护了，存在不少bug

**HPjmeter**

- 工具很强大，但是只能打开由以下参数生成的GC log，-verbose:gc  -Xloggc:gc.log。添加其他参数生成的gc.log无法打开
- HPjmeter集成了以前的HPjtune功能，可以分析在HP机器上产生的垃圾回收日志文件