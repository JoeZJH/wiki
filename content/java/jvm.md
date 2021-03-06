---
title: "深入理解Java虚拟机：JVM高级特性与最佳实践（第2版）"
layout: page
date: 2019-10-25
---
[TOC]


# 第二部分

## 第2章 Java内存区域与内存溢出异常
- 垃圾回收机制
- 运行时数据区域
    - 程序计数器：一块较小的内存，记录当前字节码的行号指示器，修改该计数器可以实现分支、循环、跳转、异常处理等。每个线程都有一个独有的程序计数器
    - Java虚拟机栈
        - 每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息
        - 常说的栈内存指的就是这个
        - 局部变量表存放基本数据类型、引用和returnAddress
        - StackOverflowError
    - 本地方法栈，Native方法调用的栈
    - Java堆
        - 所有线程共享
        - 存放对象实例
        - 垃圾收集器管理的主要区域，GC堆（垃圾堆，哈哈）
        - 细分为：新生代和老年代
        - Eden空间、From Survivor空间、To Survivor空间
        - 通过`-Xmx` 和 `-Xms` 控制堆大小
    - 方法区
        - 也有叫做「永久代」Permanent Generation
        - `-XX:MaxPermSize` 设定上限
    - 运行时常量池
        - 编译时可以决定
        - `String.intern()` 可以动态创建字符串常量
    - 直接内存
- HotSpot 对象
    - 对象创建
        - new指令
        - 检查参数是否在常量池中
        - 分配新对象内存
        - 将分配到的内存空间初始化为零值
        - 设置对象头，元数据、哈希吗、对象GC分代年龄
        - 调用<init>方法，即构造函数
    - 对象内存布局
        - 对象头
            - 运行时数据：hash码、GC分代、锁状态标志etc。长度在32位或64位。Mark Word
            - 类型指针，即指向它的类元数据的指针，通过这个指针确定这个对象是哪个类的实例。
            - 数组长度：如果对象是一个Java数组
        - 实例数据：真正有效信息
            - 相同宽度字段总是被分配到一起，父类的放在子类前面
        - 对齐填充：满足对象起始地址必须是8字节整数倍的要求
    - 对象的访问定位
        - 句柄：Java堆中将画出一块内存作为句柄池，对象reference中存的是句柄地址，而句柄包含了对象实例数据和类型数据的具体地址信息
        - 直接指针：reference就是对象实例的地址，速度快，用得多

## 第3章 垃圾收集器与内存分配策略
### 概述
- GC需要完成的3件事
    - 哪些内存需要回收
    - 什么时候回收？
    - 如何回收？
- 对象已死吗
    - 引用计数算法
        - 当有个地方应用，就加1，否则减1
        - 当计数为0时，对象就不可能再被使用
        - 循环引用问题，两个对象互相引用，但外部来看，这两个对象还是不可访问，应该被GC收集
    - 可达性分析算法
        - 从`GC Roots`对象作为起始点，向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots对象没有任何引用链，就证明此对象是不可用的。复杂度是不是太高了？？？
        - GC Roots包括
            - 虚拟机栈中引用的对象
            - 方法区类静态属性引用的对象
            - 方法区常量引用的对象
            - 本地方栈中JNI引用的对象
    - 引用
        - 强引用：`Object obj =  new Object()`
        - 软引用：第二次回收
        - 弱引用：垃圾回收的主要部分
        - 虚引用：
    - 生存还是死亡：
        - `finalize`方法在对象可GC时调用，且只调用一次，在这个方法可以逃离GC，比如将this赋值给静态变量。但第二次GC的时候任然会被GC掉，因为它只调用一次。**这个方法不要使用，它不是C++中的析构函数**
    - 回收方法区：回收效率低，能回收的有：废弃常量，类
        - 判断类是否可以GC
            - 类的实例都被回收
            - 类的ClassLoader已被回收
            - 类的class对象没在任何地方被引用，无法在任何地方通过反射访问该类的方法
- 垃圾收集算法
    - 标记-清除算法：效率低、碎片多
    - 复制算法：将内存分为两块，每次用一块，GC时只讲存活的对象复制到另一块上。内存使用效率低
    - 标记-整理算法：标记完了后，让存活的对象往一端移动，让内存保持连续
    - 分代收集算法：将Java堆分为新生代和老年代，对不同的代采用不同的算法
        - 新生代：死亡率高，用复制算法
        - 老年代：存活率高，用标记-整理（清理）
- HotSpot算法实现
    - 枚举根节点
        - 可达性分析要停止所有的线程
        - 通过OopMap数据结构来获取对应引用，在类加载完后，将引用标记出来了
    - 安全点：程序执行是并非在所有地方都停下来GC，只有在到达安全点时才暂停
    - 安全区域：指一段代码片段中，引用关系不会发生变化。在这个区域任何地方开始GC都是安全的。
- 垃圾收集器
    - Serial收集器，单线程，对用户带来不良体验的中断
    - Parallel收集器
    - Concurrent Mark Sweep（CMS）
    - Garbage First
    - ParNew收集器，Serial收集器的多线程版
- 虚拟机区分新生代和老年代的方法：
    - 每个对象有一个年龄计数器
    - 每一次新生代GC（Minor GC）仍然存活，那么年龄加1
    - 默认增加到15就晋升为老年代，参数可以通过 `-XX:MaxTenuringThreshold` 设置
    - 大对象直接进入老年代
    - 动态对象年龄判定：如果在Survivor空间中，相同年龄所有对象大小综合大于Survivor空间一半，年龄大于或等于该年龄的对象就可以直接进入老年代
    - 空间分配担保，FULL GC 老年代也要GC
    
    
       
## 第4章 虚拟机性能监控与故障处理工具
- 系统定位问题用到的数据：运行日志、异常堆栈、GC日志、线程快照、堆转储快照
- JDK命令行工具，正真实现的代码在 jdk/lib/tools.jar ，好处是这里面的工具可以直接在应用程序中实现监控分析
    - `jps` JVM Process Status 显示进程状态
        - `-l` 显示进程主类
        - `-q` 只显示LVMID
        - `-m` 显示传给主类的参数
        - `-v` 输出启动JVM参数
    - `jstat` JVM Statistics Monitoring Tool，用于手机虚拟机运行数据
        - `-class` 件事类装载、加载数量etc
        - `-gc` 监事gc
        - `-gcnew` 新生代GC
        - `-gcold` 老年代GC
        - `-compiler` 输出JIT编译器编译过的方法、耗时
    - jinfo Configuration Info for Java 显示虚拟机配置信息
    - jmap Memory Map for Java 生成内存快照heapdump文件
        - `jmap -dump:file=<filename> pid`
    - jhat Heap Dump Browser 用于分析heapdump文件，它会建立一个HTTP服务，用户可以在网页上查看分析结果
        - VisualVM, Eclipse Memory Analyzer, IBM HeapAnalyzer
        ```bash
        $jhat test.bin 
        Reading from test.bin...
        Dump file created Tue Oct 29 22:56:59 CST 2019
        Snapshot read, resolving...
        Resolving 199895 objects...
        Chasing references, expect 39 dots.......................................
        Eliminating duplicate references.......................................
        Snapshot resolved.
        Started HTTP server on port 7000
        Server is ready.
        ```
    - jstack Stack Trace for Java
        - `jstack -l <pid>`
- JDK可视化工具：JConsole 和 VisualVM
    - jconsole 命令行启动即可
    - Visual VM通过插件安装
    
