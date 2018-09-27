# 线程面试题
#### 多线程打印日志信息
题一：现有的程序代码模拟16个日志对象，并且需要运行16秒才能打印完，请在程序中添加4个线程去调用parseLog()方法打印这16条日志对象，程序只需运行4秒就打印完。
```java
public class MultiThreadPrintLog {
    public static void main(String[] args) {
        //创建阻塞队列
        BlockingQueue<String> queues = new ArrayBlockingQueue<>(1);
        for (int i=0;i<4;i++) {
            new Thread(()->{
                while(true){
                    try {
                        MultiThreadPrintLog.parseLog(queues.take());
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }

        System.out.println("begin:" + (System.currentTimeMillis() / 1000));
        //模拟处理16行日志，下面代码产生了16个日志对象

        //修改程序代码，开四个线程让16个日志对象4秒打印完
        for (int i=0;i<16;i++) {
            final String log=""+(i+1);
            {
                try {
                    queues.put(log);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //MultiThreadPrintLog.parseLog(log);
            }
        }
    }

    //parseLog方法内部的代码不能改动
    public static void parseLog(String log){
        System.out.println(log+":"+(System.currentTimeMillis()/1000));
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
#### 消费者和生产者
题二：现成的程序中代码不断产生数据，然后交给TestDo.doSome()方法处理，就好像生产者在不断产生数据，消费者不断消费数据。请将程序改造成有10个线程来消费生产的数据，这些消费则都调用TestDo.doSome()方法，地消费数据，保证消费则拿到的数据时有序的。
```java
public class MultiThreadCustomerAndProducer {
    public static void main(String[] args) {
        //创建阻塞队列
        BlockingQueue<String> queue=new SynchronousQueue<>();
        //锁对象
        //Lock lock=new ReentrantLock();
        //信号量
        Semaphore semaphore = new Semaphore(1);
        //产生10个线程
        for(int i=0;i<10;i++){
            new Thread(()->{
                try {
                    //获取信号量
                    semaphore.acquire();
                    String output=TestDo.doSome(queue.take());
                    System.out.println(Thread.currentThread().getName()+":"+output);
                    //释放信号量
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }

        System.out.println("begin:"+(System.currentTimeMillis()/1000));

            for(int i=0;i<10;i++){
                String input=i+"";
                try {
                    queue.put(input);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    }
}
class TestDo{
    public static String doSome(String input){
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        String output=input+":"+(System.currentTimeMillis()/1000);
        return output;
    }
}

```
#### 线程打印值
题三：现有程序同时启动4个线程去调用TestDo.doSome(key,value)方法，请修改代码，如果有几个线程调用TestDo.doSome(key,value)方法，传递的key相等，则这几个线程应互斥排队输出结果，即当有两个线程的key都是“1”时，它们中的一个比另一个线程要晚1秒输出。
```java
public class MultiThreadPrintValue extends Thread{
    private TestDo1 testDo;
    private String key;
    private String value;

    public MultiThreadPrintValue(String key,String key2,String value){
        this.testDo=TestDo1.getInstance();
        this.key=key+key2;
        this.value=value;
    }

    public static void main(String[] args) {
        MultiThreadPrintValue a = new MultiThreadPrintValue("1", "", "1");
        MultiThreadPrintValue b = new MultiThreadPrintValue("1", "", "2");
        MultiThreadPrintValue c = new MultiThreadPrintValue("2", "", "3");
        MultiThreadPrintValue d = new MultiThreadPrintValue("3", "", "4");
        System.out.println("begin:"+(System.currentTimeMillis()/1000));
        a.start();
        b.start();
        c.start();
        d.start();
    }

    @Override
    public void run() {
        testDo.doSome(key,value);
    }
}

class TestDo1{
    private TestDo1(){}
    private static TestDo1 _instance=new TestDo1();
    public static TestDo1 getInstance(){
        return  _instance;
    }

     //private ArrayList<Object> keys=new ArrayList<>();//线程不安全
     private CopyOnWriteArrayList<Object> keys=new CopyOnWriteArrayList<>();
    public void doSome(Object key,String value){
        if(!keys.contains(key)){
            keys.add(key);
        }else {
          for(Iterator it=keys.iterator();it.hasNext();){
              Object o=it.next();
                if(o.equals(key)){
                    key=o;
                }
          }
        }
        synchronized (key)
        {
            try {
                Thread.sleep(1000);
                System.out.println(key+":"+value+":"+(System.currentTimeMillis()/1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```