# 多线程的原子性操作之Atomic包
Java并发库java.util.concurrent是JDK5中引入到标准库中的。该包下有两个重要的子包：
java.util.concurrent.atomic包下是一组工具类，支持在单个变量上无锁线程安全编程。 
java.util.concurrent.locks包提供了一个用于锁定和等待条件的框架、不同于内建同步和监视器

### Atomic包下的类（AtomicInteger,AtomicLong等）
这组类使用了处理器CAS（比较并交换机制，一种乐观锁机制）指令来更新计数器值，它直接使用机器指令来设置值，因此对其他线程影响较小，但当在高并发条件下，与别的线程竞争赋值失败时会继续重试，这就会造成自旋锁，线程会在一个无限循环内不断尝试赋值，直到成功，对性能消耗非常大。

### CAS机制
- CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A与内存值V相等时，才将内存值V修改成B。
  - 主内存中存放的V值，所有线程共享。

  - 线程上次从内存中读取的V值A存放在线程的帧栈中，每个线程私有。

  - 需要写入内存中并改写V值的B值。也就是线程对A值操作后放入到主存V中。
  - 图解：
  ![CAS](/.assets/images/3.png)
  如图，主存中保存V值，线程中要使用V值要先从主存中读取V值到线程的工作内存A中，然后计算后变成B值，最后再把B值写回到内存V值中。多个线程共用V值都是如此操作。CAS的核心是在将B值写入到V之前要比较A值和V值是否相同，如果不相同证明此时V值已经被其他线程改变，重新将V值赋给A，并重新计算得到B，如果相同，则将B值赋给V。

- 使用这种机制编写的算法也叫非阻塞算法，标准定义为一个线程的失败或者挂起不影响其他线程的失败或者挂起的算法。
### CAS机制存在的问题
- ABA问题
因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。
------------
- 循环时间长，CPU开销大
自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
-----------
- 只能保证一个共享变量的原子操作
当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性。

### 使用LongAdder代替Atomic包下的类
LongAdder继承自Striped64，Striped64内部维护了一个懒加载的数组以及一个额外的base实力域，数组大小为2的N次方，使用每个线程内部的哈希值访问。核心思想是将AtomicLong/AtomicInteger的一个value的更新压力分散到多个value中去，从而降低更新热点。（适合统计计算场景）。
* 在高并发场景下效率比Atomic包下的类的效率快。


