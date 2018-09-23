# java多线程编程
Java 给多线程编程提供了内置的支持。 一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。
多线程是多任务的一种特别的形式，但多线程使用了更小的资源开销。
进程：一个进程包括由操作系统分配的内存空间，包含一个或多个线程。一个线程不能独立的存在，它必须是进程的一部分。一个进程一直运行，直到所有的非守护线程都结束运行后才能结束。
多线程能满足程序员编写高效率的程序来达到充分利用 CPU 的目的。

### 线程的生命周期
线程是一个动态执行的过程，它也有一个从产生到死亡的过程。
![image](/.assets/images/1.png)
- 新建状态
使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态。它保持这个状态直到程序 start() 这个线程。
- 就绪状态
当线程对象调用了start()方法之后，该线程就进入就绪状态。就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。
- 运行状态
如果就绪状态的线程获取 CPU 资源，就可以执行 run()，此时线程便处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。
- 阻塞状态
如果一个线程执行了sleep（睡眠）、suspend（挂起）等方法，失去所占用资源之后，该线程就从运行状态进入阻塞状态。在睡眠时间已到或获得设备资源后可以重新进入就绪状态。可以分为三种：
   - 等待阻塞：运行状态中的线程执行 wait() 方法，使线程进入等待阻塞状态。
   - 同步阻塞：线程在获取 synchronized 同步锁失败(因为同步锁被其他线程占用)。
   - 其他阻塞：通过调用线程的 sleep() 或 join() 发出了 I/O 请求时，线程就会进入到阻塞状态。当sleep() 状态超时，join() 等待线程终止或超时，或者 I/O 处理完毕，线程重新转入就绪状态。
- 死亡状态
一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态。

### 创建线程的两种传统方式
- 通过继承Thread类本身，在子类覆盖的run()方法中编写代码。
- 通过实现Runnable接口,重写run()方法，并传递给Thread对象。
```java
public class TraditionalThread {
    public static void main(String[] args) {
        //匿名内部类(继承Thread类)创建方式
        new Thread(){
            @Override
            public void run() {
                System.out.println(this.getName());

            }
        }.start();

        //传入Runnable对象
        Thread thread=new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        });
        thread.start();
    }
}
查看Thread类的run()方法源代码，可以知道以上两种方式都是在调用Thread对象的run()方法，
当Thread类的run()方法没有被子类覆盖时，并且为该Thread类传入了Runnable对象时，执行的是Runnable对象中的run()方法；反之，执行的
是Thread类自身的run()方法。
```
- 如果在Thread子类覆盖的run()方法中编写了运行代码，也为Thread类对象传入了一个Runnable对象。那么，线程运行时执行的代码是子类的run()方法代码，不是Runnable对象的run()方法代码。
利用面向对象原则分析：子类对象的run()方法覆盖了父类的run()方法，所以通过父类构造函数传入的Runnable对象的作用是无效的。
```java
public class TraditionalThread {
    public static void main(String[] args) {
        //匿名内部类创建方式
        new Thread(new Runnable() {//通过父类构造方法传入的Runnable对象
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName());
            }
        }){
            //运行的是这段子类的代码
            @Override
            public void run() {
                System.out.println(this.getName());

            }
        }.start();
    }
}
```
### 多线程机制会提高程序的运行效率吗？
在单核CPU中，多线线程机制不会提高运行效率，一般来说，反而会降低效率。因为CPU在各个线程上的切换会消耗时间。
### 为什么使用多线程下载网上资源会变快？
其实计算机本身没有变快，只是服务器为每个线程分配了带宽，服务器可同时为多个线程提供服务。
