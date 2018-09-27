# CountDownLatch&Exchanger实现同步通信
### CountDownLatch 计数器
犹如倒计时计数器，调用CountDownLatch对象的countDown()方法就可以将计数器减1；当计数到0时，则所有等待者或单个等待者开始执行。
类似裁判员一声令下，运动员同时开始奔跑，或所有运动员跑到终点后裁判员才开始公布结果。
```java
public class CountDownLatchTest {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();

        CountDownLatch latchOrder = new CountDownLatch(1);

        CountDownLatch latchAnswer = new CountDownLatch(3);
        for (int i=0;i<3;i++) {
            Runnable runnable=new Runnable(){
                @Override
                public void run() {
                    try {
                        System.out.println("线程"+Thread.currentThread().getName()+"正在等待命令！");
                        //等待命令
                        latchOrder.await();
                        System.out.println("线程"+Thread.currentThread().getName()+"已接收到命令");
                        Thread.sleep((long)(Math.random()*10000));
                        System.out.println("线程"+Thread.currentThread().getName()+"正在回应结果");
                        //计数器减1
                        latchAnswer.countDown();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }
            };
            service.submit(runnable);
        }

        try {
            Thread.sleep((long)(Math.random()*10000));
            System.out.println("线程"+Thread.currentThread().getName()+"正在发号命令");
            latchOrder.countDown();
            System.out.println("线程"+Thread.currentThread().getName()+"已经发号了命令，正在等待结果");
            latchAnswer.await();
            System.out.println("线程"+Thread.currentThread().getName()+"已经接收到结果");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        service.shutdown();
    }
}
```

### EXchanger 线程之间的数据交换
用于实现两个线程之间的数据交换，第一个拿出数据的线程一直要等到第二个线程拿出数据时，才能彼此交换数据。
```java
public class ExchangerTest {
    public static void main(String[] args) {
        ExecutorService service = Executors.newCachedThreadPool();
        Exchanger<String> exchanger = new Exchanger<>();

        service.submit(()->{
            String data1="hhhh";
            System.out.println("线程"+Thread.currentThread().getName()+"把数据"+data1+"拿出来准备交换！");
            try {
                Thread.sleep((long)(Math.random()*10000));
                String data2 = exchanger.exchange(data1);
                System.out.println("线程"+Thread.currentThread().getName()+"已经换到了数据"+data2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });

        service.submit(()->{
            String data2="ggg";
            System.out.println("线程"+Thread.currentThread().getName()+"把数据"+data2+"拿出来准备交换！");
            try {
                Thread.sleep((long)(Math.random()*10000));
                String data1 = exchanger.exchange(data2);
                System.out.println("线程"+Thread.currentThread().getName()+"已经换到了数据"+data1);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        });
        service.shutdown();
    }
}
````