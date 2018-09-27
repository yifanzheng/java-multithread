# BlockingQueue可阻塞队列
- 队列包含固定长度的队列和可变长度的队列。
- BlockingQueue是一个接口类，实现类有ArrayBlockingQueue和LinkedBlockingQueue。
- BlockingQueue实现的对象中，只有put()和take()方法具有阻塞功能。
#### 简单示例
```java
public class BlockedQueueTest {
    public static void main(String[] args) {
        //创建阻塞队列，长度为3
        BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(3);
        for (int i=0;i<2;i++) {
            new Thread(()->{
                while(true){
                    try {
                        Thread.sleep((long)(Math.random()*1000));
                        System.out.println("线程"+Thread.currentThread().getName()+"准备放数据");
                        queue.put(1);
                        System.out.println("线程"+Thread.currentThread().getName()+"已经放入了数据，目前队列中有"+queue.size()+"个数据");

                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }

        new Thread(()->{
            while (true) {
                try {
                    Thread.sleep(1000);
                    System.out.println("线程"+Thread.currentThread().getName()+"正准备取数据");
                    queue.take();
                    System.out.println("线程"+Thread.currentThread().getName()+"取走了一个数据，目前队列有"+queue.size()+"个数据");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

    }
}
```
#### 使用阻塞队列实现同步通信
主线程和子线程交替执行
```java
public class BlockingQueueComunication {
        public static void main(String[] args) {
            Business business = new Business();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i<20;i++){
                        business.mainThread(i);
                    }
                }
            }).start();

            new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i<20;i++){
                        business.subThread2(i);
                    }
                }
            }).start();

        }

        static class Business{
            /**
             * 创建阻塞线程
             */
            BlockingQueue<Integer> queue1=new ArrayBlockingQueue<Integer>(1);

            BlockingQueue<Integer> queue2=new ArrayBlockingQueue<Integer>(1);

            {
                try {
                    //先放入数据到队列1中
                    queue1.put(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }

            /**
             * 子线程
             */
            public void subThread2(int i){

                try {
                    //队列2放入数据
                    queue2.put(1);
                    for(int j=1;j<=20;j++){
                        System.out.println("sub2 thread sequence of "+j+",loop of "+i);
                    }
                    //从队列1中取数据
                    queue1.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            /**
             * 主线程
             */
            public void mainThread(int i){
                try {
                    queue1.put(1);
                    for(int j=1;j<=10;j++){
                        System.out.println("main thread sequence of "+j+",loop of "+i);
                    }
                    queue2.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
}

```
