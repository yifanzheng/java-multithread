# 线程的互斥与同步通信-wait-notify
wait()和notify()是直接隶属于Object类，也就是说，所有对象都拥有这一对方法。初看起来这十分不可思议，但是实际上却是很自然的，因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的wait()方法导致线程阻塞，并且该对象上的锁被释放。而调用任意对象的notify()方法则导致因调用该对象的wait()方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

其次，wait()和notify()可在任何位置调用，但是这一对方法却必须在 synchronized 方法或块中调用，理由也很简单，只有在synchronized方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，方法调用必须放置在这样的synchronized方法或块中，该方法或块的加锁对象就是调用这些方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException异常。

### wait()-notify()示例

- 子线程循环10次，主线程循环100次，接着子线程又循环10次，主线程又循环100次，如此循环50次。
```java
public class ThreadTest {
    public static void main(String[] args) {
        new ThreadTest().init();
    }

    private void init(){
        Business business = new Business();
        //主线程
        Thread mainThread=new Thread(()->{
           for(int i=1;i<=50;i++){
                business.main(i);
           }
        });
        mainThread.start();

        //子线程
        Thread subThread = new Thread(() -> {
            for(int i=1;i<=50;i++){
                business.sub(i);
            }
        });
        subThread.start();

    }

    class Business{
        /**
         *  线程切换标记
         */
        private boolean flag=true;

        /**
         * 子任务
         */
        public synchronized void sub(int i){
            try {
                while(!flag){//可以使用if(),但是效果没有while()好
                    this.wait();
                }
                for (int j=1;j<=10;j++) {
                    System.out.println("sub thread sequence of "+j+",loop of "+i);
                }
                flag=false;
                this.notify();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        /**
         * 主任务
         */
        public synchronized void main(int i){
            try {
                while(flag){
                    this.wait();
                }
                for (int j=1;j<=10;j++) {
                    System.out.println("main thread sequence of "+j+",loop of "+i);
                }
                flag=true;
                this.notify();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
### 线程同步设计原则
- 将互斥的线程任务，设计成类的内部方法，而不是在主线程中直接执行子线程。
- 线程同步通信中，防止线程被伪唤醒，使用while循环将wait()包住，效果比if更好。
