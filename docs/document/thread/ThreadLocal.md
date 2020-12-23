### ThreadLocal为什么会内存泄漏

-----

ThreadLocal是Java提供的线程安全类，其原理是每个线程都有自己的变量内存副本。实际上，每个线程Thread中都有一个ThreadLocalMap类，用于存储变量值。在更新和删除操作时，将操作每个线程中的readLocalMap类，而不会互相影响，从而实现线程安全。

ThreadLocal通常用于一个调用的上下文存储场景中，例如一个调用的令牌和traceId，它们可以在该调用的所有阶段使用，但是每个调用的值都不同，因此ThreadLocal派上了用场

但是，如果ThreadLocal使用不当，将导致内存溢出。实际上，ThreadLocal的作者Josh Bloch在编写ThreadLocal时就意识到了这一点，例如弱引用，但可能仍然存在内存溢出。让我们分析以下原因。

查看源代码，ThreadLocal的最底层是Thread中的ThreadLocalMap类。实际上，ThreadLocal只是一个工具类。最底层是Thread中的ThreadLocalMap类。

![img](https://www.programmersought.com/images/955/37d0b89fb6a06e70f4fe55a62462a683.png)

 

再次查看ThreadLocalMap的源代码![img](https://www.programmersought.com/images/298/496b6b0941e6a05c5f5a044be928e2ca.png)

 

当我们使用ThreadLocal.set（）时，集合的值和键（即企业定义的ThreadLocal类）将存储在ThreadLocalMap的Entry []数组中

在Entry实现弱引用WeakReference的情况下，Entry键（即业务方定义的ThreadLocal类）将被包装到弱引用中作为Entry键。Java中弱引用的定义是，当JVM执行垃圾回收扫描时，当它仅查找具有弱引用的对象时，它将立即回收该对象。这是在最初设计ThreadLocal时防止内存溢出的一种方法。

> ```java
> static class Entry extends WeakReference<ThreadLocal<?>> {
> 
> 
> 
>             /** The value associated with this ThreadLocal. */
> 
> 
> 
>             Object value;
> 
> 
> 
>  
> 
> 
> 
>             Entry(ThreadLocal<?> k, Object v) {
> 
> 
> 
>                 super(k);
> 
> 
> 
>                 value = v;
> 
> 
> 
>             }
> 
> 
> 
>         }
> ```

尽管键被打包为弱引用，并且将由垃圾回收机制收集，但是当值（线程）没有消失时，可能会有强引用链。

由于值是一个强引用，因此只要线程不死（例如线程池），该强引用链就会存在，那么值将无法恢复，这可能导致内存溢出

![img](https://www.programmersought.com/images/331/d47f92c7bfe9b1e600056bff6136fc33.png)

尽管ThreadLocal的作者对此进行了思考并进行了一些优化，例如，当在get过程中发现键为null时，它将遍历整个Entry数组，使用key为null删除条目，并将值指向null以消除此问题。强大的参考链。源代码方法是expungeStaleEntry

```java
        private int expungeStaleEntry(int staleSlot) {



            Entry[] tab = table;



            int len = tab.length;



 



            // expunge entry at staleSlot



            tab[staleSlot].value = null;



            tab[staleSlot] = null;



            size--;



 



            // Rehash until we encounter null



            Entry e;



            int i;



            for (i = nextIndex(staleSlot, len);



                 (e = tab[i]) != null;



                 i = nextIndex(i, len)) {



                ThreadLocal<?> k = e.get();



                if (k == null) {



                    e.value = null;



                    tab[i] = null;



                    size--;



                } else {



                    int h = k.threadLocalHashCode & (len - 1);



                    if (h != i) {



                        tab[i] = null;



 



                        // Unlike Knuth 6.4 Algorithm R, we must scan until



                        // null because multiple entries could have been stale.



                        while (tab[h] != null)



                            h = nextIndex(h, len);



                        tab[h] = e;



                    }



                }



            }



            return i;



        }
```

 

但是，这种消除强引用链的动作是由业务方在获取的情况下触发的。也许业务方没有得到它，或者get是键不是空的，并且它不会触发expungeStaleEntry类。**因此，开发人员应养成良好的习惯，记住在使用完ThreadLocal后，请调用ThreadLocal.remove（）方法或ThreadLocal.set（null）。**

在ThreadLocal的生命周期中，存在这些引用。（实线表示强引用，虚线表示弱引用）

![ ](https://www.programmersought.com/images/426/abf61a0dd3b9382e77fa2dc5ffa27e3a.png)

从ThreadLocal到Entry对象键的引用已损坏，并且Entry对象没有及时清理，这可能会导致OOM内存溢出！



### ThreadLocal OOM内存溢出场景和原理分析

## 1.案例代码

1.首先看一下代码。它模拟具有500个线程的线程池。所有线程共享一个ThreadLocal变量。每个线程在执行时都会插入一个大的List集合：
![ ](https://www.programmersought.com/images/646/37d45b701df46aa5879f6b28c23679b6.png)
![ ](https://www.programmersought.com/images/921/6ee04074a20237d8a66f3290662baa51.png)

2.设置JVM参数以将最大内存设置为256M，以模拟OOM：
![ ](https://www.programmersought.com/images/578/ada99fc4d894dcec1be2c25e791b0872.JPEG)

3.运行代码并输出结果：
![ ](https://www.programmersought.com/images/677/7a465e16cdf973205b49e2532005c7bd.JPEG)

可以看出，当单线程池达到212时，将报告错误并发生OOM内存溢出错误。
4.在运行代码时，同时打开JDK工具jConsole来监视内存更改：
![ ](https://www.programmersought.com/images/277/4766492ac22b5ccfe0963d85b6afdc85.JPEG)

可以看出，上面的内存已增加到JVM设置的最大值，然后引发异常并退出程序。
5.此示例可以很好地演示：线程池中的线程使用ThreadLocal对象后，将不再使用它，因为线程池中的线程不会退出，因此线程池中存在线程， ThreadLocal变量也将存在，占用内存！导致OOM溢出！

## 二，为什么ThreadLocal内存泄漏

在上一篇文章中，我简要介绍了不正确使用ThreadLocal导致OOM的原因。以下是详细的介绍：
1.首先看一下ThreadLocal的示意图：
这些引用在ThreadLocal的整个生命周期中都存在。参见下图：实线表示强引用，虚线表示弱引用。
![ ](https://www.programmersought.com/images/385/b4961793b01d3a51c98d5a32a19649b9.JPEG)
\2. ThreadLocal的实现是这样的：每个线程维护一个ThreadLocalMap映射表，此映射表的键是ThreadLocal实例本身，而值是真正需要存储的Object。

3.也就是说，ThreadLocal本身不存储值，它只是用作让线程从ThreadLocalMap获取值的键。值得注意的是，图中的虚线表示ThreadLocalMap使用ThreadLocal的弱引用作为Key，并且该弱引用的对象将在GC期间回收。

\4. ThreadLocalMap使用ThreadLocal的弱引用作为键。如果ThreadLocal没有外部强引用来引用它，则ThreadLocal势必在系统GC时被回收。这样，键为null的Entry将出现在ThreadLocalMap中。无法访问这些空的Entry键的值。如果当前线程没有再次结束，那么对于这些Entry键的值将始终存在一个强大的引用链：Thread Ref-> Thread-> ThreaLocalMap-> Entry-> value永远不会被回收，从而导致内存泄漏。

5.通常，在ThreadLocal中使用具有弱引用的映射。映射的类型是ThreadLocal.ThreadLocalMap。映射中的键是一个线程本地实例。该映射确实使用弱引用，但弱引用仅用于键。每个键对threadlocal的引用都很弱。当threadlocal实例设置为null时，没有对threadlocal实例的强引用，因此threadlocal将由gc回收。
但是，我们的值无法恢复，并且永远无法访问该值，因此会发生内存泄漏。因为从当前线程连接了一个强引用。仅在当前线程结束之后，当前线程才会在堆栈中不存在，强引用将被破坏，并且当前线程和映射值将全部由GC恢复。最好的方法是调用threadlocal的remove方法，这将在后面提到。

6.实际上，这种情况在ThreadLocalMap的设计中已经考虑到了，并增加了一些保护措施：当ThreadLocal get（），set（），去掉（） 。在上一节中也提到了这一点！

7.但是，这些被动预防措施不能保证不会出现内存泄漏：

（1）使用静态ThreadLocal会延长ThreadLocal的生命周期，并可能导致内存泄漏。
（2）如果使用ThreadLocal并且不再调用get（），set（），remove（）方法，则将导致内存泄漏，因为该内存始终存在。

## 3.为什么使用弱引用？OOM是一个弱参考吗？

1.从表面上看，内存泄漏的根本原因是使用弱引用。Internet上的大多数文章都专注于分析ThreadLocal对弱引用的使用导致内存泄漏，但是另一个问题也值得思考：为什么使用弱引用而不是强引用？

让我们看一下官方文档：
![ ](https://www.programmersought.com/images/312/e8c878502b5a8defe50717c101629228.png)
下面我们讨论两种情况：

> （1）密钥使用强引用：被引用的ThreadLocal对象被回收，但是ThreadLocalMap也持有对ThreadLocal的强引用。如果未手动删除，则不会回收ThreadLocal，从而导致条目内存泄漏。
>
> （2）密钥使用弱引用：被引用的ThreadLocal对象被回收。因为ThreadLocalMap持有对ThreadLocal的弱引用，所以即使未手动删除它，也将回收ThreadLocal。下次ThreadLocalMap调用设置，获取或删除时，将清除该值。

比较这两种情况，我们可以发现：由于ThreadLocalMap的生命周期与Thread一样长，因此，如果不手动删除相应的键，则会导致内存泄漏，但是使用弱引用可以提供额外的一层。保护：弱引用ThreadLocal不会泄漏内存，下次ThreadLocalMap调用设置，获取和删除时，将清除相应的值。

因此，ThreadLocal内存泄漏的根本原因是：由于ThreadLocalMap的生命周期与Thread一样长，因此，如果不手动删除相应的键，将导致内存泄漏，而不是因为引用弱。

## 四，ThreadLocal最佳实践

1.根据以上分析，我们可以了解ThreadLocal内存泄漏的原因和后果，那么如何避免内存泄漏呢？

答案是：每次使用ThreadLocal时，都调用其remove（）方法清除数据。

在使用线程池的情况下，不及时清除ThreadLocal不仅是内存泄漏的问题，而且更严重的是可能导致业务逻辑问题。因此，使用ThreadLocal等同于锁定后解锁以及使用后清理。

注意：

并非所有使用ThreadLocal的地方都在remove（）的结尾，它们的生命周期可能需要与项目的生命周期一样长，因此请做出适当的选择以避免业务逻辑错误！但是首先要确保的是，保存在ThreadLocal中的数据大小不是很大！

2.然后，我们将初始代码修改为：

取消注释：threadLocal.remove（）; 结果将不会显示为OOM，可以看到堆内存更改参差不齐，证明在每个remove（）之后，都会释放ThreadLocal内存！线程池中的线程数继续增加！
![ ](https://www.programmersought.com/images/670/f5206997fa27d66821ba1998510cd6b6.png)
取消注释：threadLocal.remove（）; 结果将不会显示为OOM，可以看到堆内存更改参差不齐，证明在每次remove（）之后，将释放ThreadLocal内存！线程池中的线程数继续增加！
![ ](https://www.programmersought.com/images/288/731debbf4f7fc887a137239505d983e0.JPEG)
![ ](https://www.programmersought.com/images/333/2d352a2086d56e392a7c39ef657ccc65.JPEG)