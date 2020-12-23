## JVM中一次完整的GC流程是怎样的

## 一、可达性分析算法（GC Roots）

有一种**引用计数法**，可以用来判断对象被引用的次数，在任何某一具体时刻，如果引用次数为0，则代表可以被回收。
这种实现方式比较简单，但对于循环引用的情况束手无策，所以 Java 采用了**可达性分析算法**。
即判断某个对象是否与 GC Roots 的这类对象之间的路径可达，若不可达，则有可能成为回收对象，被判定为不可达的对象要成为可回收对象必须**至少经历两次标记过程**，如果在这两次标记过程中仍然没有逃脱成为可回收对象的可能性，则基本上就真的成为可回收对象了。
具体详情可以看我另外一篇博客：[点这里：JAVA虚拟机–JVM之旅02 判断对象已”死“（想要对象留下，就给它一个自救的机会）](https://blog.csdn.net/fristjcjdncg/article/details/107725146)

在 Java 中，可作为 GC Roots 的对象包括以下几种：

**虚拟机栈**（本地变量表）中引用的对象
**方法区**中类**静态属性**引用的对象
**方法区**中**常量**引用的对象
**本地方法栈**中引用的对象

## 二、JVM中的堆结构

Java堆 = 老年代 + 新生代 | 大小比例为1:2，如下
新生代 = Eden + S0 + S1 也称 form和 to
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801204214577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyaXN0amNqZG5jZw==,size_16,color_FFFFFF,t_70)
当 Eden 区的空间满了， Java虚拟机会触发一次 Minor GC，以收集新生代的垃圾，存活下来的对象，则会转移到 Survivor 区。具体参考我的另外一篇：[点这里：深入讲解JVM垃圾回收算法思想及全过程）](https://blog.csdn.net/fristjcjdncg/article/details/107727764)

- **大对象**（需要大量连续内存空间的Java对象，如那种很长的字符串）**直接进入老年代**；
- 如果对象在Eden出生，并经过第一次Minor GC后仍然存活，并且被Survivor容纳的话，年龄设为1，每熬过一次Minor GC，年龄+1，熬过了一轮又一轮，若年龄超过一定限制（15），则被晋升到老年态。**即长期存活的对象进入老年代。**
- **Minor GC** 是清理新生代
- **Full GC**老年代满了而无法容纳更多的对象，Minor GC 之后通常就会进行**Full GC**，Full GC 清理整个内存堆 – 包括年轻代和老年代。
  **Major GC** 发生在老年代的GC，清理老年区，经常会伴随至少一次Minor GC，比Minor GC慢10倍以上。

**Minor GC触发机制**：
当年轻代满时就会触发Minor GC，这里的年轻代满指的是Eden代满，Survivor满不会引发GC。
**Full GC触发机制：**
（1）调用System.gc时，系统建议执行Full GC，但是不必然执行
（2）老年代空间不足
（3）方法区空间不足
（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存
（5）由Eden区、survivor space1（From Space）区向survivor space2（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小
当永久代满时也会引发Full GC，会导致Class、Method元信息的卸载