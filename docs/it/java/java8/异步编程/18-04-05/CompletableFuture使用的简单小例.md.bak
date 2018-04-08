### CompletableFuture使用的简单小例

时间： 2018年04月05日

> 记得之前从知乎上看到过一个关于程序猿的笑话：有一个程序猿遇到一个问题题，后来决定用多线程的方式来解决，再后来他两个问题有了。

#### 1. 异步小案例

 刚入职公司时在公司的CE小组，那个时候刚毕业没多久，Java8这套以前没怎么用过(现在Java10都已经出来了)，记得印象比较深的一次，同组的远哥(现在ali大佬)给我和其他的三位小白同事抛了个多线程的问题，倒是不难，但也反映出了我等小白多线程编程的鸡肋。

**问题大致如下**：

- 有任务A、B、C以及D四个任务，它们之间的依赖性如下，A任务可单独执行，B和C任务依赖A任务产生的结果，B和C之间没有依赖性，D任务依赖B和C任务执行的结果，假使这几个任务执行的时间分别是5、5、4、3s，那么你能在多长时间内处理完所有的任务？

其实无论你是不是搞IT的，也估计想得出来最短时间应该是13s，因为关键点就是任务B和C可以同时执行嘛，是的，B和C同时执行取二者花费时间最长的5s作为这两个任务花费的总时间。
具体程序内容记得不是很清楚了，毕竟人年龄大易忘事，就大抵定义为这样子：

```
//任务A
public class TaskA {
    public Integer compute() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return 5;
    }
}

//任务B
public class TaskB {
    public Integer compute(Integer intA) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return (intA * 2);
    }
}

//任务C
public class TaskC  {
    public Integer compute(Integer intA) {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return (intA * 3);
    }
}

//任务D
public class TaskD {
    public Integer compute(Integer intB, Integer intC) {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return (intB + intC);
    }
}

```

拿到这个题，就觉得这个肯定是一个多线程问题，那到底如何设计线程去计算，捣腾了一会儿，远哥见我们几小白还没搞出来，说这个多线程你们可以用Callable试一下，或者Java8里面的方法（Java8不太熟，我不是一个合格的程序员）？对喔，Callable创建的线程是能够获取到其返回值的，可以利用多核CPU能力来为任务B和C分别创建两个线程来同时执行。

```
public static void main(String[] args) {
        long startTime = System.currentTimeMillis();
        TaskA taskA = new TaskA();
        int valA = taskA.compute();
        //定义B和C的异步执行任务
        Callable<Integer> taskBCallable = () -> {
            TaskB taskB = new TaskB();
            return taskB.compute(valA);
        };

        Callable<Integer> taskCCallable = () -> {
            TaskC taskC = new TaskC();
            return taskC.compute(valA);
        };

        //FutureTask以便于得到计算的结果
        FutureTask<Integer> futureTaskB = new FutureTask<>(taskBCallable);
        FutureTask<Integer> futureTaskC = new FutureTask<>(taskCCallable);

        //启动两个线程执行任务
        new Thread(futureTaskB).start();
        new Thread(futureTaskC).start();


        try {
            TaskD taskD = new TaskD();
            int valD = taskD.compute(futureTaskB.get(), futureTaskC.get());
            System.out.println("一共用时: " + (System.currentTimeMillis() - startTime) / 1000 +
                    "s; 计算结果为：" + valD);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
```

ok, 主线程写好了，看看执行结果

```
一共用时: 13s; 计算结果为：25
```

嗯，费劲千辛万苦终于写好了，有点小高兴，远哥过来看，好像确实没什么问题，但又动手撸起了代码来：

```
long startTime = System.currentTimeMillis();
        Integer intA = new TaskA().compute();
        try {
            CompletableFuture.supplyAsync(() -> new TaskB().compute(intA))
                    .thenCombine(CompletableFuture.supplyAsync(() -> new TaskC().compute(intA)), (x, y) -> new TaskD().compute(x, y))
                    .thenAccept(valD ->  System.out.println("一共用时: " + (System.currentTimeMillis() - startTime) / 1000 + "s; 计算结果为：" + valD)).get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
```

当然，执行结果仍然为：

```
一共用时: 13s; 计算结果为：25
```

代码思想如上哈，不保证百分百还原，我等一看，呀，远哥确实牛逼啊，Java8确实牛逼啊，代码如此干练清爽，短短几行核心代码就搞定这个“复杂”的计算过程了，整个计算过程比较符合该小案例的执行思想，先得到A的计算结果，然后以异步的方式开启一个新线程执行任务B同时又另开一个线程执行任务C，然后取任务B和任务C的计算结果给任务D，然后打印出计算结果......


#### 2. 下一步
Java8的异步编程用着是很爽，这一篇博客仅简单试用了一下8中异步编程的几个常用的API，希望能在下一篇中进一步了解：

- 理清CompletableFuture的家族史
- 对CompletableFuture多达50个的API的特性进行分析
- CompletableFuture的优缺点
- 。。。。。。

