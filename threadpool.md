# 线程池
使用线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或 者“过度切换”的问题。 

### Executors方式创建各种线程池
##### 1、newFixedThreadPool
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
```java
//创建固定线程数量的线程池
ExecutorService service = Executors.newFixedThreadPool(3);
for(int j=1;j<=10;j++){
            int task=j;
            service.execute(new Runnable() {
                @Override
                public void run() {
                    for(int i=1;i<=10;i++){
                        try {
                            Thread.sleep(20);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread().getName()+" is looping of "+i+" for task of "+task);
                    }
                }
            });
        }
System.out.println("all of 10 tasks is committed");
service.shutdown();
```
##### 2、newCachedThreadPool
创建一个可缓存线程池，线程数量是动态变化的，应用中存在的线程数可以无限大。
```java
ExecutorService service = Executors.newCachedThreadPool();
```
##### 3、newSingleThreadExecutor
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
```java
ExecutorService service = Executors.newSingleThreadExecutor();
```
##### 4、newScheduledThreadPool
创建一个定长线程池，支持定时及周期性任务执行。
- 在多线程并行处理定时任务时，Timer 运行多个 TimeTask 时，只要其中之一没有捕获 抛出的异常，其它任务便会自动终止运行，使用 ScheduledExecutorService 则没有这个问题。
- ScheduleExecutorService接口中有四个重要的方法，其中scheduleAtFixedRate和scheduleWithFixedDelay在实现定时程序时比较方便。

###### scheduleAtFixedRate 按指定频率周期执行某个任务
```java
//定时的线程
ScheduledExecutorService scheduledService = Executors.newScheduledThreadPool(3);
    scheduledService.scheduleAtFixedRate(new Runnable() {
         @Override
         public void run() {
                System.out.println("bombing!");
         }
    },6,2,TimeUnit.SECONDS);
    6：表示6秒后开始执行任务
    2：表示以2秒的间隔执行一次
```
需要注意的是，当执行任务的时间大于指定的间隔时间时，它并不会在指定间隔时开辟一个新的线程并发执行这个任务。而是等待该线程执行完毕后才开始执行下一个任务。
```java
//定时的线程
ScheduledExecutorService scheduledService = Executors.newScheduledThreadPool(3);
//时间格式化对象（保证线程安全）
ThreadLocal<DateFormat> df=ThreadLocal.withInitial(new Supplier<DateFormat>() {
    @Override
    public DateFormat get() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
});
scheduledService.scheduleAtFixedRate(new Runnable() {

    @Override
    public void run() {
        System.out.println("bombing!"+df.get().format(new Date()));
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}, 6, 2, TimeUnit.SECONDS);
在3秒后才开始执行下一个线程任务。
```
###### scheduleWithFixedDelay 按指定频率间隔执行某个任务（以固定延迟时间进行执行）
```java
//定时的线程
ScheduledExecutorService scheduledService = Executors.newScheduledThreadPool(3);
//时间格式化对象（保证线程安全）
ThreadLocal<DateFormat> df=ThreadLocal.withInitial(new Supplier<DateFormat>() {
    @Override
    public DateFormat get() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
});
scheduledService.scheduleWithFixedDelay(new Runnable() {

    @Override
    public void run() {
        System.out.println("bombing!"+df.get().format(new Date()));
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}, 6, 2, TimeUnit.SECONDS);
2：表示当前任务执行完成后再延迟2秒执行下一个任务。
所以,以上一次任务的结束时间 + 延迟时间 = 下一次任务的开始时间。
本程序是任务耗时3秒，延迟2秒，下一个任务是在（3+2）5秒后开始。
```
### ThreadPoolExecutor创建线程
##### 构造方法参数说明
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) //后两个参数为可选参数
```
`corePoolSize`：指的是保留的线程池的大小  

`maximumPoolSize`：指的是线程池的最大大小  

`keepAliveTime`：指的是空闲线程结束的超时时间  

`TimeUnit`：是一个枚举类，表示keepAliveTime的单位  

`workQueue`：表示存放任务的队列  

`threadFactory`:使用ThreadFactory创建新线程，默认使用defaultThreadFactory创建线程  

`handler`:定义处理被拒绝任务的策略，默认使用ThreadPoolExecutor.AbortPolicy,任务被拒绝时将抛出RejectExecutorException  

##### 工作原理
- 线程池刚创建时，里面没有一个线程，任务队列作为参数传进来。
- 当调用`execute()`方法时，添加一个任务会有如下判断：
   - a.如果正在运行的线程数量小于corePoolSize,会立即创建线程运行任务。
   - b.如果正在运行的线程数大于或等于corePoolSize,会将超出的任务放到队列中。
   - c.如果队列满了，线程数量小于maximumPoolSize，会创建线程运行这个任务。
   - d.如果队列满了，线程数量大于等于maximumPoolSize，会抛出异常，告诉调用者“我不再接受任务”。
- 当一个线程完成任务时，它会从队列中取下一个任务执行。
- 当线程无事可做，超过keepAliveTime时，线程池会判断当前运行线程数大于corePoolSize，那会被停掉，线程数会回缩到corePoolSize大小。
```java
BlockingQueue<Runnable> queue=new ArrayBlockingQueue<>(10);
    ThreadPoolExecutor executor=new ThreadPoolExecutor(3,10,3,TimeUnit.SECONDS,queue);
    for(int i=0;i<20;i++){
        final int task=i;
        executor.execute(()->{
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+":"+(task+1));
        });
    }
    executor.shutdown();
```
### 最后
线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。



