# ThreadLocal 实现线程范围的共享变量

1. ThreadLocal 用于实现线程内的数据共享，即对于相同的程序代码，多个模块在同一线程中运行时要共享一份数据，而在另外线程中运行时又共享另外的数据。

2. 当使用 ThreadLocal 维护共享变量时，ThreadLocal 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。通俗一点，就是在每个 Thread 中，都有一个 ThreadLocalMap 集合，在该 map 中，key 为该ThreadLocal 对象，value 即为 ThreadLocal 中变量的副本，所以 ThreadLocal 只是操作每个线程的 ThreadLocalMap 而已。

3. 每个线程调用全局 ThreadLocal 对象的 set() 方法，就相当于在其内部的map中增加了一条记录，key 分别是各自的线程，value 是各自 set() 方法传的值。在线程结束时，可以调用 ThreadLocal.clear() 方法，这样会更快的释放内存。不调用也可以，因为线程结束后会自动释放相关的 TreadLocal 变量。

### ThreadLocal 应用场景

- 订单处理包含的一系列操作：减少库存量，增加一条流水台账，修改总账。这几个操作要在同一个事务中完成，通常即在同一个线程中进行处理，如果累加公司应收款的操作失败了，应把前面的操作回滚，否则提交所有操作。这要求这些操作使用相同的数据库连接对象，而这些操作的代码分别位于不同的模块类中。

- 银行转账包含的一系列操作：把转出账户的余额减少，把转入账户的余额增加，这两个操作要在同一个事务中完成，它们必须使用相同的数据库连接对象，转入和转出操作的代码分别是两个不同的账户对象的方法。

### TreadLocal 案例设计

- 实现对 ThreadLocal 对象的封装，让外界不要直接操作 TreadLocal 变量。

- 一个 TreadLocal 代表一个变量，故其中只能放一个数据。如果有多个数据变量要线程共享，可以先定义一个对象来封装这些数据，然后将这个对象存储到 TreadLocal 中。

```java
public class ThreadLocalTest {

    private static ThreadLocal<MyThreadData> myThreadDataScope=new ThreadLocal<>();

    public static void main(String[] args) {
       for(int i=0;i<2;i++){
           new Thread(()->{
               int data=new Random().nextInt(10000);
               MyThreadData myData = MyThreadData.getThreadInstance();
               myData.setName("name"+data);
               myData.setAge(data);
               // 将共享数据传入ThreadLocal对象中
               myThreadDataScope.set(myData);
               new A().get();
               new B().get();
           }).start();
       }
    }

    static class A{

        public void get(){
            /*MyThreadData myThreadData = myThreadDataScope.get();*/
            MyThreadData myThreadData = MyThreadData.getThreadInstance();
            System.out.println("A from "+Thread.currentThread().getName()+" get MyData:"+myThreadData.getName()+","+myThreadData.getAge());
        }

    }

    static class B{
        public void get(){
            /*MyThreadData myThreadData = myThreadDataScope.get();*/
            MyThreadData myThreadData = MyThreadData.getThreadInstance();
            System.out.println("B from "+Thread.currentThread().getName()+" get MyData:"+myThreadData.getName()+","+myThreadData.getAge());
        }
    }

}

/**
 将多个变量封装成一个对象
*/
class MyThreadData{
    private String name;

    private int age;

    private  MyThreadData(){}

    private static ThreadLocal<MyThreadData> myThreadLocal=new ThreadLocal<>();

    /**
     * 获取ThreadLocal对象
     * @return
     */
    public static MyThreadData getThreadInstance(){
        MyThreadData instance = myThreadLocal.get();
        if(instance==null){
            instance=new MyThreadData();
            myThreadLocal.set(instance);
        }
        return instance;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
