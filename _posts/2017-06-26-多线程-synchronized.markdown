---
title: 多线程-synchronized
layout: post
guid: urn:uuid:39650954-625e-434b-98b5-0051dc5de831
tags:
  -
---
a. Synchronized的作用主要有三个：
（1）确保线程互斥的访问同步代码
（2）保证共享变量的修改能够及时可见
（3）有效解决重排序问题。

b. Synchronized总共有三种用法：
（1）修饰普通方法
（2）修饰静态方法
（3）修饰代码块

--

```java
    public class SynchronizedDemo {
      public void method() {
          synchronized (this) {
              System.out.println("Method 1 start");
          }
		}
	}
```
使用 `javap -c xxx.SynchronizedDemo` 进行反编译此Class文件，得到字节码：

```uncompiled
	Compiled from "SynchronizedDemo.java"
	public class com.hongv.test.concurrent.SynchronizedDemo {
	  public com.hongv.test.concurrent.SynchronizedDemo();
	    Code:
	       0: aload_0
	       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
	       4: return

	  public void method();
	    Code:
	       0: aload_0
	       1: dup
	       2: astore_1
	       3: monitorenter						// <--------!!!----------
	       4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
	       7: ldc           #3                  // String Method start
	       9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
	      12: aload_1
	      13: monitorexit						// <--------!!!----------
	      14: goto          22
	      17: astore_2
	      18: aload_1
	      19: monitorexit
	      20: aload_2
	      21: athrow
	      22: return
	    Exception table:
	       from    to  target type
	           4    14    17   any
	          17    20    17   any
	}
```
1. `monitorenter`
> 每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行`monitorenter`指令时尝试获取monitor的所有权，过程如下：
	* 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。
	* 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.Â
	* 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。
2. `monitorexit`
> 执行monitorexit的线程必须是objectref所对应的monitor的所有者。
指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。

> wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。


对同步方法进行反编译，方法的同步并没有通过指令`monitorenter`和`monitorexit`来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。


> Synchronized是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的Mutex Lock来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized效率低的原因。因此，这种依赖于操作系统Mutex Lock所实现的锁我们称之为“重量级锁”。JDK中对Synchronized做的种种优化，其核心都是为了减少这种重量级锁的使用。JDK1.6以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和“偏向锁”。

#### 线程状态及状态转换
> 当多个线程同时请求某个对象监视器时，对象监视器会设置几种状态用来区分请求的线程：
* Contention List：所有请求锁的线程将被首先放置到该竞争队列, ContentionList并不是一个真正的Queue，而只是一个虚拟队列，原因在于ContentionList是由Node及其next指针逻辑构成，并不存在一个Queue的数据结构。ContentionList是一个后进先出（LIFO）的队列，每次新加入Node时都会在队头进行，通过CAS改变第一个节点的的指针为新增节点，同时设置新增节点的next指向后续节点，而取得操作则发生在队尾。显然，该结构其实是个Lock-Free的队列。
因为只有Owner线程才能从队尾取元素，也即线程出列操作无争用，当然也就避免了CAS的ABA问题。
* Entry List：Contention List中那些有资格成为候选人的线程被移到Entry List, EntryList与ContentionList逻辑上同属等待队列，ContentionList会被线程并发访问，为了降低对ContentionList队尾的争用，而建立EntryList。Owner线程在unlock时会从ContentionList中迁移线程到EntryList，并会指定EntryList中的某个线程（一般为Head）为Ready（OnDeck）线程。Owner线程并不是把锁传递给OnDeck线程，只是把竞争锁的权利交给OnDeck，OnDeck线程需要重新竞争锁。这样做虽然牺牲了一定的公平性，但极大的提高了整体吞吐量，在Hotspot中把OnDeck的选择行为称之为“竞争切换”。
* Wait Set：那些调用wait方法被阻塞的线程被放置到Wait Set
* OnDeck：任何时刻最多只能有一个线程正在竞争锁，该线程称为OnDeck
* Owner：获得锁的线程称为Owner
* !Owner：释放锁的线程

##### 自旋锁
问题：
> 那些处于ContetionList、EntryList、WaitSet中的线程均处于阻塞状态，阻塞操作由操作系统完成（在Linxu下通过pthread_mutex_lock函数）。线程被阻塞后便进入内核（Linux）调度状态，这个会导致系统在用户态与内核态之间来回切换，严重影响锁的性能

缓解上述问题的办法便是自旋，其原理是：
> 当发生争用时，若Owner线程能在很短的时间内释放锁，则那些正在争用线程可以稍微等一等（自旋），在Owner线程释放锁后，争用线程可能会立即得到锁，从而避免了系统阻塞。但Owner运行的时间可能会超出了临界值，争用线程自旋一段时间后还是无法获得锁，这时争用线程则会停止自旋进入阻塞状态（后退）。基本思路就是自旋，不成功再阻塞，尽量降低阻塞的可能性，这对那些执行时间很短的代码块来说有非常重要的性能提高。自旋锁有个更贴切的名字：自旋-指数后退锁，也即复合锁。很显然，自旋在多处理器上才有意义。

那synchronized实现何时使用了自旋锁？
> 答案是在线程进入ContentionList时，也即第一步操作前。线程在进入等待队列时首先进行自旋尝试获得锁，如果不成功再进入等待队列。这对那些已经在等待队列中的线程来说，稍微显得不公平。还有一个不公平的地方是自旋线程可能会抢占了Ready线程的锁。自旋锁由每个监视对象维护，每个监视对象一个。

缺点： 自旋占用cpu


--------

  HotspotJVM中，锁的状态总共有四种：无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁（但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级）。JDK 1.6中默认是开启偏向锁和轻量级锁的，我们也可以通过-XX:-UseBiasedLocking来禁用偏向锁。锁的状态保存在对象的头文件中。参考[reference_1](http://www.infoq.com/cn/articles/java-se-16-synchronized),[reference_2](http://www.tuicool.com/articles/2aeAZn),[reference_3](http://www.cnblogs.com/paddix/p/5405678.html)

> 如果对象是数组类型，则虚拟机用3个Word（字宽）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，一字宽等于四字节，即32bit。

  |长度|内容|说明|
  |---|----|----|
  |32/64bit|Mark Word|存储对象的hashCode或锁信息等。|
  |32/64bit|Class Metadata Address|存储到对象类型数据的指针|
  |32/64bit|Array length|数组的长度（如果当前对象是数组）|

  <table style="width: 800px;" border="1" cellspacing="0" cellpadding="0">
  <tbody>
  <tr>
  <td rowspan="2" valign="top" width="76"><p><strong>锁状态</strong></p></td>
  <td colspan="3" valign="top" width="160"><p align="center">25 bit</p></td>
  <td rowspan="2" valign="top" width="120"><p align="center">4bit</p></td>
  <td valign="top" width="120"><p>1bit</p></td>
  <td valign="top" width="110"><p>2bit</p></td>
  </tr>
  <tr>
  <td colspan="2" valign="top" width="70"><p>23bit</p></td>
  <td valign="top" width="80"><p>2bit</p></td>
  <td valign="top" width="120"><p>是否是偏向锁</p></td>
  <td valign="top" width="110"><p>锁标志位</p></td>
  </tr>
  <tr>
  <td valign="top" width="105"><p>轻量级锁</p></td>
  <td colspan="5" valign="top" width="390"><p>指向栈中锁记录的指针</p></td>
  <td valign="top" width="110"><p>00</p></td>
  </tr>
  <tr>
  <td valign="top" width="105"><p>重量级锁</p></td>
  <td colspan="5" valign="top" width="390"><p>指向互斥量（重量级锁）的指针</p></td>
  <td valign="top" width="110"><p>10</p></td>
  </tr>
  <tr>
  <td valign="top" width="105"><p>GC标记</p></td>
  <td colspan="5" valign="top" width="390"><p>空</p></td>
  <td valign="top" width="110"><p>11</p></td>
  </tr>
  <tr>
  <td valign="top" width="105"><p>偏向锁</p></td>
  <td valign="top" width="70"><p>线程ID</p></td>
  <td colspan="2" valign="top" width="80"><p>Epoch</p></td>
  <td valign="top" width="120"><p>对象分代年龄</p></td>
  <td valign="top" width="120"><p>1</p></td>
  <td valign="top" width="110"><p>01</p></td>
  </tr>
  <tr>
  <td valign="top" width="105"><p>无锁</p></td>
  <td colspan="3" valign="top" width="150"><p>对象的hashCode</p></td>
  <td valign="top" width="120"><p>对象分代年龄</p></td>
  <td valign="top" width="120"><p>0</p></td>
  <td valign="top" width="110"><p>01</p></td>
  </tr>
  </tbody>
  </table>

####  轻量级锁的加锁过程 (参考：http://www.infoq.com/cn/articles/java-se-16-synchronized)
![](http://7u2nrz.com1.z0.glb.clouddn.com/light-weight-lock-acquire.png)
####  偏向锁的枷锁过程
![](http://7u2nrz.com1.z0.glb.clouddn.com/biase-lock-acquire.png)

##### 偏向锁 (https://blogs.oracle.com/dave/biased-locking-in-hotspot)
在JVM1.6中引入了偏向锁，偏向锁主要解决无竞争下的锁性能问题，首先我们看下无竞争下锁存在什么问题：
现在几乎所有的锁都是可重入的，也即已经获得锁的线程可以多次锁住/解锁监视对象，按照之前的HotSpot设计，每次加锁/解锁都会涉及到一些CAS操作（比如对等待队列的CAS操作），CAS操作会延迟本地调用，因此偏向锁的想法是一旦线程第一次获得了监视对象，之后让监视对象“偏向”这个线程，之后的多次调用则可以避免CAS操作，说白了就是置个变量，如果发现为true则无需再走各种加锁/解锁流程。

> CAS为什么会引入本地延迟？这要从SMP（对称多处理器）架构说起，下图大概表明了SMP的结构：
* CAS及SMP架构
 ![](http://7u2nrz.com1.z0.glb.clouddn.com/SMP_system_vmdc.png)
>> 其意思是所有的CPU会共享一条系统总线（BUS），靠此总线连接主存。每个核都有自己的一级缓存，各核相对于BUS对称分布，因此这种结构称为“对称多处理器”。
 而CAS的全称为Compare-And-Swap，是一条CPU的原子指令，其作用是让CPU比较后原子地更新某个位置的值，经过调查发现，其实现方式是基于硬件平台的汇编指令，就是说CAS是靠硬件实现的，JVM只是封装了汇编调用，那些AtomicInteger类便是使用了这些封装后的接口。
 Core1和Core2可能会同时把主存中某个位置的值Load到自己的L1 Cache中，当Core1在自己的L1 Cache中修改这个位置的值时，会通过总线，使Core2中L1 Cache对应的值“失效”，而Core2一旦发现自己L1 Cache中的值失效（称为Cache命中缺失）则会通过总线从内存中加载该地址最新的值，大家通过总线的来回通信称为“Cache一致性流量”，因为总线被设计为固定的“通信能力”，如果Cache一致性流量过大，总线将成为瓶颈。而当Core1和Core2中的值再次一致时，称为“Cache一致性”，从这个层面来说，锁设计的终极目标便是减少Cache一致性流量。
 而CAS恰好会导致Cache一致性流量，如果有很多线程都共享同一个对象，当某个Core CAS成功时必然会引起总线风暴，这就是所谓的本地延迟，本质上偏向锁就是为了消除CAS，降低Cache一致性流量。
* Cache一致性：
上面提到Cache一致性，其实是有协议支持的，现在通用的协议是MESI（最早由Intel开始支持），具体参考：http://en.wikipedia.org/wiki/MESI_protocol，
Cache一致性流量的例外情况：
其实也不是所有的CAS都会导致总线风暴，这跟Cache一致性协议有关，具体参考：http://blogs.oracle.com/dave/entry/biased_locking_in_hotspot
NUMA(Non Uniform Memory Access Achitecture）架构：
与SMP对应还有非对称多处理器架构，现在主要应用在一些高端处理器上，主要特点是没有总线，没有公用主存，每个Core有自己的内存，针对这种结构此处不做讨论。
(摘自：http://blog.csdn.net/chen77716/article/details/6618779)

##### 轻量级锁
> “轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的。但是，首先需要强调一点的是，轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用产生的性能消耗。
> 轻量级锁所适应的场景是线程交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。

##### 重量级锁
> 依赖于操作系统Mutex Lock所实现的锁我们称之为“重量级锁”, 需要使用操作系统进行阻塞的线程锁, 就要进行内核态和用户态之间的切换, 成本非常高

##### 锁的优缺点对比

|锁|优点|缺点|适用场景|
|---|---|---|------|
|偏向锁|加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。|如果线程间存在锁竞争，会带来额外的锁撤销的消耗。|适用于只有一个线程访问同步块场景。|
|轻量级锁|竞争的线程不会阻塞，提高了程序的响应速度。|如果始终得不到锁竞争的线程使用自旋会消耗CPU。|追求响应时间。同步块执行速度非常快。|
|重量级锁|线程竞争不使用自旋，不会消耗CPU。|线程阻塞，响应时间缓慢。|追求吞吐量。同步块执行速度较长。|



















_____
