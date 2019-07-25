# Callable&Future 的应用

- Future 取得的结果类型与 Callable 返回的结果类型必须一致，这是通过泛型实现的。
- Callable 要采用 ExecutorService 的 submit() 方法提交,返回的 Future 对象可以取消任务 cancel(true)。

```java
ExecutorService service = Executors.newFixedThreadPool(3);
Future<String> future = service.submit(new Callable<String>() {
    @Override
    public String call() throws Exception {
        Thread.sleep(100);
        return "hello";
    }
});
try {
    System.out.println("拿到结果：" + future.get());
} catch (Exception e) {
    e.printStackTrace();
}
service.shutdown();
```

- CompletionService 用于提交一组 Callable 任务，其 take() 方法返回已完成的一个 Callable 任务对应的 Future 对象。

```java
ExecutorService service = Executors.newFixedThreadPool(10);
CompletionService<Integer> completionService=new ExecutorCompletionService<Integer>(service);
for(int i=1;i<=10;i++){
    final int index=i;
    completionService.submit(new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            Thread.sleep(100);
            return index;
        }
    });

}
for(int i=0;i<10;i++){
    try {
        System.out.println(completionService.take().get());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
service.shutdown();
```
