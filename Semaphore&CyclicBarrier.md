# Semaphore&CyclicBarrier实现同步通信
### Semaphore信号灯
- Semaphore可以维护当前访问自身的线程个数，并提供了同步机制，使用Semaphore可以控制访问资源的线程个数。
- 单个信号量的Semaphore对象可以实现互斥锁（Synchronized）的功能，并且可以是由一个线程获得了“锁”，再由另一个线程的释放，这可应用于死锁恢复的场合。
```java
public class SemaphoreTest {
    public static void main(String[] args) {
        //创建线程池
        ExecutorService service = Executors.newCachedThreadPool();
        //创建信号量,true 代表公平
        Semaphore semaphore = new Semaphore(3,true);
        for(int i=1;i<=10;i++){
            Runnable runnable=()->{
                try {
                    //获取一个信号量
                    semaphore.acquire();
                    System.out.println("线程"+Thread.currentThread().getName()+"进入，当前已有"+(3-semaphore.availablePermits())+"个并发");
                    //间隔一秒
                    Thread.sleep((long)(Math.random()*10000));
                    System.out.println("线程"+Thread.currentThread().getName()+"即将离开");
                    //释放信号量
                    semaphore.release();
                    System.out.println("线程"+Thread.currentThread().getName()+"已离开，当前有"+(3-semaphore.availablePermits())+"个并发");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            };
            service.submit(runnable);
        }
        service.shutdown();
    }
}
```
### CyclicBarrier屏障点
表示线程彼此等待，所有线程都集合好后，才开始又进行彼此的任务。当任务执行完后，又彼此等待，所有线程都集合好后，又开始下一个任务。
```java
public class CyclicBarrierTest {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        CyclicBarrier barrier = new CyclicBarrier(3);
        for(int i=0;i<3;i++){
            Runnable runnable=new Runnable(){
                @Override
                public void run() {
                    try {
                        //间隔一秒
                        Thread.sleep((long)(Math.random()*10000));
                        System.out.println("线程"+Thread.currentThread().getName()
                                +"即将到达集合点1，当前已有"+(barrier.getNumberWaiting()+1)+"到达，"
                                +(barrier.getNumberWaiting()==2?"都到齐了，继续走":"正在等候"));
                        barrier.await();//屏障点

                        Thread.sleep((long)(Math.random()*10000));
                        System.out.println("线程"+Thread.currentThread().getName()
                                +"即将到达集合点2，当前已有"+(barrier.getNumberWaiting()+1)+"到达，"
                                +(barrier.getNumberWaiting()==2?"都到齐了，继续走":"正在等候"));
                        barrier.await();

                        Thread.sleep((long)(Math.random()*10000));
                        System.out.println("线程"+Thread.currentThread().getName()
                                +"即将到达集合点3，当前已有"+(barrier.getNumberWaiting()+1)+"到达，"
                                +(barrier.getNumberWaiting()==2?"都到齐了，继续走":"正在等候"));
                        barrier.await();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            };
            service.submit(runnable);
        }
    service.shutdown();
    }
}

```