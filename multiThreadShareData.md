# 多线程之间共享数据的方式探讨
### 方式一
- 如果每个线程执行的代码相同，可以用一个Runnable对象，这个Runnable对象中存放那个共享数据（卖票系统可以这样做）。
```java
public class MultiThreadShareData {
    public static void main(String[] args) {
        MyShareData shareData=new MyShareData();
        //放入不同的线程中
        new Thread(shareData).start();

        new Thread(shareData).start();

    }
}

class MyShareData implements Runnable{
    //共享的数据
    private int count=100;

    @Override
    public void run() {
        while (count>0){
            count--;
            System.out.println(Thread.currentThread().getName()+" 减了1，count还剩："+count);
        }
    }
}
```

### 方式二
- 如果每个线程执行的代码不同时，就需要不同的Runnable对象：
>a. 将共享数据封装在一个对象中，然后将这个对象逐一传递给各个Runnable对象，每个线程对共享数据操作的方法也分配到这个对象中，这样容易实现针对该数据进行的各个操作的互斥通信。
```java
public class MultiThreadShareData {

    private int shareData=0;

    public static void main(String[] args) {
        ShareData data = new ShareData();

        new Thread(new MyRunnable1(data)).start();

        new Thread(new MyRunnable2(data)).start();
    }

}

class MyRunnable1 implements Runnable{

    private ShareData data;

    public MyRunnable1(ShareData data){
        this.data=data;
    }

    @Override
    public void run() {
        for(int i=0;i<100;i++){
            //对数据进行增加
            this.data.increment();
        }
    }
}

class MyRunnable2 implements Runnable{

    private ShareData data;

    public MyRunnable2(ShareData data){
        this.data=data;
    }

    @Override
    public void run() {
        for(int i=0;i<100;i++){
            //对数据进行减少
            this.data.decrement();
        }
    }
}

/**
 将共享的数据封装成一个类
*/
class ShareData{
    //共享数据
    private int j=0;

    public synchronized void increment(){
        this.j++;
        System.out.println(Thread.currentThread().getName()+":j增加了1后j="+j);
    }

    public synchronized void decrement() {
        this.j--;
        System.out.println(Thread.currentThread().getName()+":j减少了1后j="+j);
    }
    public int getJ() {
        return j;
    }
}
```
  >b. 将Runnable对象作为某一个类的内部类，共享数据作为这个外部类的成员变量，每个线程对共享数据的操作方法也分配到外部类中，实现共享数据的互斥和通信操作，作为内部类的各个Runnable对象调用外部类的这些方法。
```java
public class MultiThreadShareData {

    private int shareData=0;

    public static void main(String[] args) {
        MultiThreadShareData m=new MultiThreadShareData();
        //初始化Runnable对象
        MyRunnable1 myRunnable1 = m.new MyRunnable1();
        MyRunnable2 myRunnable2=m.new MyRunnable2();

        //开启线程
        new Thread(myRunnable1).start();

        new Thread(myRunnable2).start();
    }

    private synchronized void increment(){
        this.shareData++;
        System.out.println(Thread.currentThread().getName()+":shareData增加了1后shareData="+shareData);
    }

    private synchronized void decrement() {
        this.shareData--;
        System.out.println(Thread.currentThread().getName()+":shareData减少了1后shareData="+shareData);
    }

    /**
     * 作为内部类的Runnable对象
     */
    class MyRunnable1 implements Runnable{

        @Override
        public void run() {
            for(int i=0;i<100;i++){
                increment();
            }
        }
    }

    class MyRunnable2 implements Runnable{

        @Override
        public void run() {
            for(int i=0;i<100;i++){
                decrement();
            }
        }
    }
}
```
 >c. 以上两种方法的组合：将共享数据封装到一个对象中，每个线程对共享数据的操作方法也分配到对象中，对象作为外部类的成员变量或方法中的局部变量，每个线程的Runnable作为成员内部类或局部内部类。
 ```java
   public class MultiThreadShareData {
    public static void main(String[] args) {

        ShareData data = new ShareData();

        new Thread(()->{
            for(int i=0;i<100;i++){
                data.increment();
            }
        }).start();

        new Thread(()->{
            for (int j=0;j<100;j++) {
                data.decrement();
            }
        }).start();
    }

}

/**
 封装共享数据的对象
*/
class ShareData{
    //共享数据
    private int j=0;

    /**
     对共享数据进行操作的方法
    */
    public synchronized void increment(){
        this.j++;
        System.out.println(Thread.currentThread().getName()+":j增加了1后j="+j);
    }

    public synchronized void decrement() {
        this.j--;
        System.out.println(Thread.currentThread().getName()+":j减少了1后j="+j);
    }

    public int getJ() {
        return j;
    }
}
 ```
总之， 要同步互斥的几段代码最好放在几个独立的方法中，这些方法在放入一个类中，这样比较容易实现它们之间的同步互斥和通信。

   
