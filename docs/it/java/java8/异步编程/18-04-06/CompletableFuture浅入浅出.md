### CompletableFuture浅入浅出

时间：2018年04月06日

> [前一篇博客](../18-04-05/CompletableFuture使用的简单小例.md)简单使用了CompletableFuture中的几个API，简单清爽地解决了一个多线程编程的小示例，不得不说该异步编程框架在实际开发中是起着极大的作用，最初在项目中使用时总是对thenApppy()和thenCompose()方法比较迷惑，什么时候该用前者，什么时候该用后者，后来大抵也了解了两者的使用场景，不过依然没有对该框架有一个整体的认识，借该博文整理一下个人对CompletableFuture框架的理解。

#### 1. 简谈Future接口

Future接口在Java5中已有引入，主要是解决需要获取多线程(实现Callable接口的线程)的返回值的问题，看一个简单示例：

```
Callable<String> callable = () -> "ok";
ExecutorService executor = Executors.newSingleThreadExecutor();
//主线程需要callable线程的结果，先拿到一个未来的Future
Future<String> future = executor.submit(callable); 
//有了结果后再根据get方法取真实的结果,当然如果此时callable线程如果没有执行完get方法会阻塞执行完，如果执行完则直接返回结果或抛出异常
System.out.println(future.get()); 
```

输出结果:
```
ok
```
Future能够正确获取到子线程产生的结果数据，但如果通过future.get()拿取数据时，callable线程并没有执行完，调用线程便会阻塞直到执行结束，或者通过isDone()方法轮询查看执行结果。

#### 2. CompletableFuture类
CompletableFuture类实现了Future和CompletionStage两个接口，提供了Future所有旧的获取线程的返回值的api，不过在CompletableFuture框架中，这种get()或isDone()的获取方式不建议使用。

个人把CompletableFuture框架的api进行如下划分：