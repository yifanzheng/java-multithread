# Lock实现线程同步通信

### Lock

Lock 比传统中的 Synchronized 更面向对象，与生活中的锁类似，锁本身也是一个对象。两个线程执行的代码片段要实现同步互斥的效果，它们必须使用同一个 Lock 对象。锁是要上在代表要操作的资源的类的内部方法中，而不是在线程中。
```java
public class LockTest {
    public static void main(String[] args) {
        new LockTest().init();
    }

    private void init() {
        Outputer outputer = new Outputer();
        new Thread(() -> {
            while (true) {
                try {
                    outputer.output("zhanghaoxuan");
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }).start();

        new Thread(() -> {
            while (true) {
                try {
                    outputer.output("lihuoming");
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    static class Outputer {
        //锁对象
        Lock lock = new ReentrantLock();

        public void output(String name) {
            int len = name.length();
            //加锁
            lock.lock();
             try {
                 for (int i = 0; i < len; i++) {
                     System.out.print(name.charAt(i));
                 }
                 System.out.println();
             }finally {
                 //解锁
                 lock.unlock();
             }

        }
    }
}
```
- 使用 try-finally 将要同步的代码包起来，是为了防止代码出现异常时无法释放锁对象。

### ReadWriteLock（读写锁）

读写锁：分为读锁和写锁,多个读锁不互斥，读锁与写锁互斥，写锁与写锁互斥，这是 JVM 自己控制的，只要上好相应的锁即可。读的时候上读锁，写的时候上写锁。

```java
public class ReadAndWriteLock {
    public static void main(String[] args) {
        Queue3 queue3 = new Queue3();
        for(int i = 0; i < 3; i++){
            new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true)
                    queue3.get();
                }
            }).start();
        }

        for(int j = 0; j < 3; j++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    while (true)
                    queue3.put((int)(Math.random() * 1000));
                }
            }).start();
        }
    }

   static class Queue3{
        private int data = 0;
        ReadWriteLock lock = new ReentrantReadWriteLock();
        // 读数据
        public void get() {
            lock.readLock().lock();
            try {
                System.out.println(Thread.currentThread().getName() + " be ready to get data!");

                Thread.sleep((long)(Math.random() * 1000));
                
                System.out.println(Thread.currentThread().getName() + " have read data:" + data);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.readLock().unlock();
            }
        }

        // 写数据
        public void put(int data){
            lock.writeLock().lock();
            try {
                System.out.println(Thread.currentThread().getName() + "be ready to write data!");

                Thread.sleep((long)(Math.random() * 1000));

                System.out.println(Thread.currentThread().getName() + "have write data：" + data);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                lock.writeLock().unlock();
            }
        }
    }
}
```
使用读写锁实现一个简单缓存类
```java
public class CacheDemo {
    // 缓存集合
    private Map<String,Object> cache=new HashMap<>();
    // 读写锁对象
    private ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    public static void main(String[] args) {
        // 执行方法
    }

    // 获取数据
    public Object getData(String key) {
        lock.readLock().lock();
        Object value = null;
        try {
            value = cache.get(key);
            if (Objects.equals(value,null)) {
                lock.readLock().unlock();
                lock.writeLock().lock();
                try {
                    // 防止当有多个线程阻塞在lock.writeLock().lock()时，其中一个线程修改了数据，释放锁后，后面的线程又重新对数据进行重复修改
                    if (Objects.equals(value,null)) {
                        value = "aaaa";//实际是从数据库query数据
                        cache.put(key,value);
                    }
                } finally {
                    lock.writeLock().unlock();
                }
                lock.readLock().lock();
            }
        } finally {
            lock.readLock().unlock();
        }
        return value;
    }
}
```
