# Condition 实现线程同步通信

- Condition 的功能类似在传统线程技术中的 Object.wait 和 Object.notify 的功能。在等待 Condition 时，允许发生“虚假唤醒”，这通常作为对基础平台语义的让步。

```java
public class ConditionTest {
    public static void main(String[] args) {
        new ConditionTest().initThread();
    }

    private void initThread() {
        Business business = new Business();
        //主线程
        Thread mainThread=new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                business.mainThread(i);
            }
        });
        mainThread.start();

        //子线程
        Thread subThread = new Thread(() -> {
            for (int i = 1; i <= 50; i++) {
                business.subThread(i);
            }
        });
        subThread.start();
    }

    class Business {

        /**
         * 线程切换标志
         */
        private  boolean isStart = true;

        /**
         * 锁对象
         */
        private Lock lock = new ReentrantLock();

        /**
         * 获取condition对象
         */
        private Condition condition = lock.newCondition();

        /**
         * 子线程
         */
        public void subThread(int i) {
            lock.lock();
            try {
                while (isStart) {
                    condition.await();
                }
                for (int j = 1; j <= 20; j++) {
                    System.out.println("sub thread sequence of " + j + ",loop of " + i);
                }
                this.isStart = true;
                condition.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }

        /**
         * 主线程
         */
        public void mainThread(int i) {
            lock.lock();
            try {
                while (!isStart) {
                    condition.await();
                }
                for (int j = 1;j <= 10; j++) {
                    System.out.println("main thread sequence of " + j + ",loop of " + i);
                }
                this.isStart = false;
                condition.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
}
```
- 一个锁的内部可以有多个 condition，即多路等待和通知。在传统的线程机制中一个监视器对象上只能有一路等待和通知。

### 示例
作为一个示例，假定有一个绑定的缓冲区，它支持 put 和 take 方法。如果试图在空的缓冲区上执行 take 操作，
则在某一个项变得可用之前，线程将一直阻塞；如果试图在满的缓冲区上执行 put 操作，则在有空间变得可用之前，
线程将一直阻塞。我们喜欢在单独的等待 set 中保存 put 线程和 take 线程，这样就可以在缓冲区中的项或空间
变得可用时利用最佳规划，一次只通知一个线程。可以使用两个 Condition 实例来做到这一点。 

```java
 class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock();
     try {
       while (count == items.length) {
         notFull.await();
       }
       items[putptr] = x; 
       if (++putptr == items.length) {putptr = 0};
       ++count;
       notEmpty.signal();
     } finally {
       lock.unlock();
     }
   }

   public Object take() throws InterruptedException {
     lock.lock();
     try {
       while (count == 0) {
         notEmpty.await();
       }
       Object x = items[takeptr]; 
       if (++takeptr == items.length) {takeptr = 0};
       --count;
       notFull.signal();
       return x;
     } finally {
       lock.unlock();
     }
   } 
 }

```

