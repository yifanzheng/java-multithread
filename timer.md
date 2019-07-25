# 定时器-Timer

Timer 类主要负责完成定时计划任务的功能，就是在指定的时间的开始执行某个任务。
Timer 类的作用是设置计划任务，而封装任务内容的类是 TimerTask 类。此类是一个抽象类，继承需要实现一个 run 方法。

### 创建一个简单的 Timer
```java
public class TraditionalTimer{
    public static void main(String[] args){
        new Timer().schedule(new TimerTask(){
            @Override
            public void run(){
                System.out.prinln("bombing");
            }
        },2000,3000);
    }
}
2000:2秒后开始执行定时任务。
3000:定时任务开始执行后，每过3秒执行一次。
```
### 定时器案列
使一个任务先 2 秒后执行，再4秒后执行，依次交替。

```java
public class TraditionalTimer {
    /**
     * 次数
     */
    private static int count=0;

    public static void main(String[] args) {
        // 自定义内部定时任务
        class MyTimerTask extends TimerTask{

            @Override
            public void run() {
                System.out.println("bombing");
                count=(count+1)%2;
                new Timer().schedule(new MyTimerTask(),2000*(count+1));
            }
        }
        // 创建定时器
        new Timer().schedule(new MyTimerTask(),2000);

        while (true) {
            System.out.println(new Date().getSeconds());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```