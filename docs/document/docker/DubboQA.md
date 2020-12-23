\## 1 Dubbo



 1.1 服务调用超时问题怎么解决？

> dubbo服务调用：failover，failfast等失败策略，分别是超时重试次数，超时快速失败并返回

 1.2 Dubbo支持哪些序列化方式？

> hession2、kyro、pb

 1.3 Dubbo和SpringCloud的关系？

> dubbo是rpc框架，springcloud是微服务治理框架

 1.4 Dubbo的架构设计？一共划分了哪些层？

>dubbo架构一共分为10层，网络层，协议层，服务层、配置层，序列化层

 1.5 Dubbo的默认集群容错方案？

> ### 内置容错策略
>
> Dubbo主要内置了如下几种策略：
>
> - Failover(失败自动切换)
> - Failsafe(失败安全)
> - Failfast(快速失败)
> - Failback(失败自动恢复)
> - Forking(并行调用)
> - Broadcast(广播调用)
>
> 这些名称比较相似，概念也比较容易混淆，下面逐一进行解释。
>
> #### Failover(失败自动切换)
>
> `Failover`是高可用系统中的一个常用概念，服务器通常拥有主备两套机器配置，如果主服务器出现故障，则自动切换到备服务器中，从而保证了整体的高可用性。
>
> Dubbo也借鉴了这个思想，并且把它作为Dubbo`默认的容错策略`。当调用出现失败的时候，根据配置的重试次数，会自动从其他可用地址中重新选择一个可用的地址进行调用，直到调用成功，或者是达到重试的上限位置。
>
> Dubbo里默认配置的重试次数是2，也就是说，算上第一次调用，最多会调用3次。
>
> 其配置方法，容错策略既可以在服务提供方配置，也可以服务调用方进行配置。而重试次数的配置则更为灵活，既可以在服务级别进行配置，也可以在方法级别进行配置。具体优先顺序为：
>
> ```
> 服务调用方方法级配置 > 服务调用方服务级配置 > 服务提供方方法级配置 > 服务提供方服务级配置
> ```

 1.6 Dubbo使用的是什么通信框架?

> Dubbo 默认使用 Netty 框架，也是推荐的选择，另外内容还集成有Mina、Grizzly。

 1.7 Dubbo的主要应用场景？

> 使用 Dubbo 可以将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，可用于提高业务复用灵活扩展，使前端应用能更快速的响应多变的市场需求。

 1.8 Dubbo服务注册与发现的流程？流程说明。

> Dubbo 会在 Spring 实例化完 bean 之后，在刷新容器最后一步发布 ContextRefreshEvent 事件的时候，通知实现了 ApplicationListener 的 ServiceBean 类进行回调 onApplicationEvent 事件方法，Dubbo 会在这个方法中调用 ServiceBean 父类 ServiceConfig 的 export 方法，而该方法真正实现了服务的（异步或者非异步）发布。

 1.9 Dubbo的集群容错方案有哪些？

 1.10 Dubbo的四大组件

> - Container：服务运行容器，只启动一次
> - Provider：服务提供者
> - Registry：服务注册与发现的注册中心
> - Consumer：服务消费者
> - Monitor：统计服务的调用次数和调用时间的监控中心

 1.11 Dubbo在安全机制方面是如何解决的

> 

 1.12 Dubbo和SpringCloud的区别？

 1.13 Dubbo支持哪些协议，每种协议的应用场景，优缺点？

 1.14 Dubbo的核心功能有哪些？

 1.15 Dubbo的注册中心集群挂掉，发布者和订阅者之间还能通信么？

 1.16 Dubbo集群的负载均衡有哪些策略

 1.17 为什么需要服务治理？

 1.18 Dubbo超时时间怎样设置？

在使用过程中都遇到了些什么问题？

Dubbo 的设计目的是为了满足高并发小数据量的 rpc 调用，在大数据量下的性能表现并不好，建议使用 rmi 或 http 协议。

\## 2 ElasticSearch



 2.1 你们公司的ES集群，一个node一般会分配几个分片？

 2.2 Elasticsearch是如何实现Master选举的？

 2.3 你是如何做写入调优的？

 2.4 如何避免脑裂？

 2.5 19-Elasticsearch对于大数据量（上亿量级）的聚合如何实现？

 2.6 ES主分片数量可以在后期更改吗？为什么？

 2.7 如何监控集群状态？

 2.8 ElasticSearch中的副本是什么？

 2.9 20.ES更新数据的执行流程？

 2.10 shard里面是什么组成的？

 2.11 ElasticSearch中的分析器是什么？

 2.12 什么是脑裂？

 2.13 客户端在和集群连接时，如何选择特定的节点执行请求的？

 2.14 Elasticsearch中的倒排索引是什么？

 2.15 什么是索引？索引（名词） 一个索引(index)

 2.16 详细描述一下Elasticsearch更新和删除文档的过程

\## 3 JVM



 3.1 JVM参数主要有⼏种分类

> 指定JVM 参数我们就可以指定启动 JVM 进程以哪种模式（server 或 client），运行时分配的堆大小，栈大小，用什么垃圾收集器等等，JVM 参数主要分以下三类：
>
> **1. 标准参数（-）：**
> 所有的 JVM 实现都必须实现这些参数的功能，而且向后兼容；例如 -verbose:gc（输出每次GC的相关情况)，-server（一服务器模式启动jvm）
>
> **2. 非标准参数（-X）：**
> 默认 JVM 实现这些参数的功能，但是并不保证所有 JVM 实现都满足，且不保证向后兼容，栈，堆大小的设置都是通过这个参数来配置的,用得最多的如下：
>
> -Xms512m JVM 启动时设置的初始堆大小为 512M
> -Xmx512m JVM 可分配的最大堆大小为 512M
> -Xmn200m 设置的年轻代大小为 200M
> -Xss128k 设置每个线程的栈大小为 128k
>
> **3. 非Stable参数（-XX）：**
> 此类参数各个 jvm 实现会有所不同，将来可能会随时取消，需要慎重使用,非 Stable 参数主要有三大类：
>
> **1) 行为参数：**
> 用于改变JVM的一些基础行为，如使用什么垃圾收集器。
> -XX:+UseG1GC：使用G1
> -XX:+UseSerialGC：在新生代和老年代使用串行收集器
> -XX:+UseParNewGC：在新生代使用并行收集器
> -XX:+UseConcMarkSweepGC：对老生代采用CMS
> -XX:+UseParallelGC：启用并行收集器Parallel Scavenge
> -XX:+UseParallelOldGC：对Full GC启用并行，当-XX:+UseParallelGC启用时该项自动启用
>
> **2）追踪、调试参数：**
> -XX:+PrintGCDetails ：打印GC前后的详细信息以及对的使用情况
> -XX:+PrintGCDateStamps：在每次GC的打印信息前加上时间戳
> -Xloggc:文件路径 ：将GC日志保存到对应文件中，借助分析工具进行分析
> -XX:+HeapDumpOnOutOfMemoryError ：当首次遭遇OOM时导出此时堆中相关信息
>
> **3）性能调优（Performance Tuning）：**
> 用于 jvm 的性能调优，如设置新老生代内存容量比例。
>
> -XX:newSize=300m：设置年轻代的大小。
> -XX:newRatio=2：设置年轻代与老年代的比例。2表示，年轻代：老年代=1:2
> -XX:SurvivorRatio=3：设置年轻代中Eden区与两个Survivor区的比值。3表示 Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5。
> -XX:ParallelGCThreads=n：设置并行收集器收集时使用的CPU数。并行收集线程数。
> -XX:MaxGCPauseMillis=n：设置并行收集最大暂停时间，即响应时间。
> -XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比(即，吞吐量)。公式为1/(1+n)。

 3.2 3.Java中会存在内存泄漏吗，简述一下。

> 内存泄露场景：
>
> 1.静态集合类引起内存泄露： 如果仅仅释放引用本身（o=null），那么list仍然引用该对象，所以这个对象对GC 来说是不可回收的。
>
> [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)
>
> ```
>         LinkedList<Integer> list=new LinkedList<>();
>         int i=0;
>         while(true) {
>             Integer o=new Integer(i++);
>             list.add(o);
>             o=null;
>         }
> ```
>
> [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)
>
> 2.集合里面对象的属性被修改以后，remove方法不起作用
>
> [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)
>
> ```
> Set<Person> set = new HashSet<Person>();
> Person p1 = new Person("唐僧","pwd1",25);
> Person p2 = new Person("孙悟空","pwd2",26);
> Person p3 = new Person("猪八戒","pwd3",27);
> set.add(p1);
> set.add(p2);
> set.add(p3);
> p3.setAge(2);    //修改p3的年龄,此时p3元素对应的hashcode值发生改变
> set.remove(p3);//这个移除是不会发生作用的
> ```
>
> [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)
>
>  
>
> 3.数据库，sockect，io等连接没有及时关闭的也会造成内存泄漏，所以需要我们显式的close。
>
> 4.单例模式中，如果单例对象持有外部对象的引用的话，也会导致外部对象不能被回收。
>
> 5.监听器（未解决）
>
> 6.在使用线程池的时候：如果没有去给队列设置固定的长度或者没有给线程池设置规定的个数的话。
>
> 在流量峰值，大量的请求进入的时候，会导致队列塞满或者创建大量的线程，也会导致内存溢出。

 3.3 Java虚拟机是如何判定两个Java类是相同的？

> 通过重写类的equals、hashcode方法

 3.4 Java 中都有哪些引用类型

> 对象引用，强、软、弱、虚

 3.5 在 Java 中，对象什么时候可以被垃圾回收？

> 当一个对象到GC Roots不可达时，在下一个垃圾回收周期中尝试回收该对象，如果该对象重写了finalize()方法，并在这个方法中成功自救(将自身赋予某个引用)，那么这个对象不会被回收。但如果这个对象没有重写finalize()方法或者已经执行过这个方法，也自救失败，该对象将会被回收。

 3.6 19.StackOverflow异常有没有遇到过？一般你猜测会在什么情况下被触发？

> java虚拟机栈中内存溢出，一般由栈内存的局部变量过爆了，导致内存溢出。出现在递归方法，参数个数过多，递归过深，递归没有出口。

 3.7 堆空间如何划分？以及如何设置各个部分？

> java堆按照年龄分代划分：新生代、老年代、元空间
>
> 通过虚拟机参数进行设置：
>
> * 堆大小设置
>   1. -Xms堆的最小值
>   2. -Xmx堆空间的最大值
>
> * 新生代堆大小设置
>
>   1. -XX:NewSize新生代的最小值
>   2. -XX:MaxNewSize新生代的最大值
>   3. -XX:NewRatio设置新生代与老年代在堆空间的大小
>   4. -XX:SurvivorRatio新生代中Eden所占区域的大小
>
> * 对象晋升年龄
>
>   -XX:MaxTenuringThreshold,设置将新生代对象转到老年代时需要经过多少次垃圾回收，但是仍然没有被回收
>
> 

 3.8 什么是栈帧？栈帧存储了什么？

> 栈帧是虚拟机中存储方法出入口，局部变量表、操作数栈、返回地址、动态连接

 3.9 如何设置参数生成GC日志？

> -xx:printGCdetail、-xxprintGC

 3.10 GC 是什么？为什么要有 GC？

> java虚拟机垃圾回收机制，GC：主要是对javaheap内存的回收，回收无用不可达的对象，来释放内存空间

 3.11 使用过哪些jdk命令，并说明各个的作用是什么

> keytool:生成证书
>
> jstack:输出虚拟机线程堆栈信息
>
> jstat:统计jvm内存使用情况、gc回收次数
>
> jps：输出运行中的java进程

 3.12 JVM运行时数据区区域分为哪⼏部分？

> 栈：线程私有，主要存储：局部变量表、操作数栈、动态链接、方法返回地址
>
> 堆：共享
>
> 方法区：共享
>
> 程序计数器：私有
>
> 本地方法栈：私有

 3.13 是否了解类加载器双亲委派模型机制和破坏双亲委派模型？

> 1**.启动类加载器(Bootstrap ClassLoader)**，这个类加载器使用**C++语言实现**，是虚拟机自身的一部分；
>
> 2.**其它所有的类加载器**，这些类加载器都由**Java语言实现**，独立于虚拟机外部，并且全部继承自java.lang.ClassLoader。
>
> java为了保证类的唯一性与安全验证默认使用父类加载字节码对象，如果父类加载不了交由字类加载，如果都无法加载则抛出ClassNotFoundException
>
> 破坏双亲委派模型：字类重新loadClass、findClass方法
>
> **Spring破坏双亲委派模型** 
> Spring要对用户程序进行组织和管理，而用户程序一般放在WEB-INF目录下，由WebAppClassLoader类加载器加载，而Spring由Common类加载器或Shared类加载器加载。 
> 那么Spring是如何访问WEB-INF下的用户程序呢？ 
> 使用**线程上下文类加载器。** Spring加载类所用的classLoader都是通过Thread.currentThread().getContextClassLoader()获取的。当线程创建时会默认创建一个AppClassLoader类加载器（对应Tomcat中的WebAppclassLoader类加载器）： setContextClassLoader(AppClassLoader)。 
> 利用这个来加载用户程序。即任何一个线程都可通过getContextClassLoader()获取到WebAppclassLoader。

 3.14 逃逸分析有几种类型？

 3.15 -Xms这些参数的含义是什么？

 3.16你知道哪几种垃圾收集器,各自的优缺点，重点讲下cms和G1，包括原理，流程，优缺点。

 3.17JVM的内存结构，Eden和Survivor比例是多少？

\## 4 多线程/高并发



 4.1 负载平衡的意义什么？

 4.2 请说出同步线程及线程调度相关的方法？

 4.3 关于epoll和select的区别，哪些说法 是正确的？（多选） 

A. epoll 和 select 都是 I/O 多路复用的技术，都可以实现同时监听 多个I/O事件的状态。 

B. epoll 相比 select 效率更高，主要是基于其操作系统支持的 I/O 事件通知机制，而select是基于轮询机制。 

C. epoll支持水平触发和边沿触发两种模式。 

D. select能并行支持I/O比较小，且无法修改。

 4.4 启动一个线程是调用run()方法还是start()方法？

 4.5 如何确保N个线程可以访问N个资源同时又不导致死锁？

 4.6 编写多线程程序的几种实现方式（换个问法：创建多线程的方式）？

 4.7 线程和进程的区别？

 4.8 什么是线程池，有哪些常用线程池？

 4.9 什么是死锁？

 4.10 怎么保证缓存和数据库数据的一致性？

\## 5 消息中间件



 5.1 消费者获取消息有几种模式？

 5.2 17.RocketMQ的特点有哪些？

 5.3 kafka 同时设置了 7 天和 10G 清除数据，到第五天的时候消息达到了 10G，这个时候 kafka将如何处理？

 5.4 为何需要Kafka集群

 5.5 Kafka 数据存储设计

 5.6 Kafka如何判断一个节点是否存活？

 5.7 kafka消息发送的可靠性机制有几种

 5.8 请详细说一下推送模式和拉取模式。

 5.9 Kafka 与传统消息系统之间有三个关键区别

 5.10 RocketMQ 由哪些角色组成？

 5.11 23.Kafka的消费者如何消费数据

 5.12 Kafka的优点

 5.13 Kafka 的设计是什么样的呢？

 5.14 说说你对Consumer的了解？

 5.15 Kafka新建的分区会在哪个目录下创建

 5.16 说一下Kafka消费者消费过程

 5.17 介绍下Kafka

 5.18 什么情况会导致Kafka运行变慢？