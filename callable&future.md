# Callable&Future的应用
- Future取得的结果类型与Callable返回的结果类型必须一致，这是通过泛型实现的。
- Callable要采用ExecutorService的submit()方法提交,返回的Future对象可以取消任务cancel(true)。
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
    System.out.println("拿到结果："+future.get());
} catch (Exception e) {
    e.printStackTrace();
}
service.shutdown();
```
- CompletionService用于提交一组Callable任务，其take()方法返回已完成的一个Callable任务对应的Future对象。
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
