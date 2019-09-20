# 线程的互斥与同步通信-Synchronized

多线程的同步机制对资源进行加锁，使得在同一个时间，只有一个线程可以进行操作，同步用以解决多个线程同时访问时可能出现的问题。
- 同步机制可以使用 synchronized 关键字实现。
- 当 synchronized 关键字修饰一个方法的时候，该方法叫做同步方法。
- 当 synchronized 关键字修饰一个代码片段的时候，该代码片段叫做同步代码块。
- 当 synchronized 方法执行完或发生异常时，会自动释放锁。

### Synchronized 示例

```java
/**
  多线程输出字符串，使用 Synchronized 关键字防止字符串被打乱。
**/
public class TraditionalThreadSynchronized {
    public static void main(String[] args) {
        new TraditionalThreadSynchronized().init();
    }

    private void init(){
        Outputer outputer = new Outputer();
        new Thread(()->{
            while (true) {
                try {
                    outputer.output1("zhanghaoxuan");
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }).start();

        new Thread(()->{
            while (true) {
                try {
                    Outputer.output3("lihuoming");
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    static class Outputer{
        public void output1(String name){
            int len=name.length();
            // 同步代码块
            synchronized (Outputer.class) {
                for (int i=0;i<len;i++){
                    System.out.print(name.charAt(i));
                }
                System.out.println();
            }
        }

        // 同步方法，相当于synchronized (this) {...}
        public synchronized void output2(String name){
            int len=name.length();
            for (int i=0;i<len;i++){
                System.out.print(name.charAt(i));
            }
            System.out.println();

        }

         // 同步方法，相当于synchronized (类对象的字节码 Class对象) {...}
        public static synchronized void output3(String name){
            int len=name.length();
            for (int i=0;i<len;i++){
                System.out.print(name.charAt(i));
            }
            System.out.println();

        }
    }
}
```
其中，output1 与 output3 同步，output2 与他们两个都不同步。

### Synchronized关键字使用注意

- 在一段代码中，synchronized 关键字最好使用一次，否则容易产生死锁。
- Synchronized 关键字是使用在代表要操作的资源的类的内部方法中，而不是线程代码中。
- 一般，Synchronized 关键字作用的代码范围越小越好，即使用同步代码块比同步方法更好。
- Synchronized 中的"锁"必须是同一对象才能实现同步互斥。
