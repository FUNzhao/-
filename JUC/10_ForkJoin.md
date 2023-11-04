# **ForkJoin**

在JDK中，提供了这样一种功能：它能够将复杂的逻辑拆分成一个个简单的逻辑来并行执行，待每个并行执行的逻辑执行完成后，再将各个结果进行汇总，得出最终的结果数据。有点像Hadoop中的MapReduce。



**ForkJoin**是由JDK1.7之后提供的多线程并发处理框架。ForkJoin框架的基本思想是分而治之。什么是分而治之？分而治之就是将一个复杂的计算，按照设定的阈值分解成多个计算，然后将各个计算结果进行汇总。相应的，ForkJoin将复杂的计算当做一个任务，而分解的多个计算则是当做一个个子任务来并行执行。

![image-20230807155351917](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230807155351917.png)

## Java并发编程的发展

对于Java语言来说，生来就支持多线程并发编程，在并发编程领域也是在不断发展的。Java在其发展过程中对并发编程的支持越来越完善也正好印证了这一点。

- Java 1 支持thread，synchronized。
- Java 5 引入了 thread pools， blocking queues, concurrent collections，locks, condition queues。
- Java 7 加入了fork-join库。
- Java 8 加入了 parallel streams。

#### 并发与并行

**并发和并行在本质上还是有所区别的。**

##### 并发

并发指的是在同一时刻，只有一个线程能够获取到CPU执行任务，而多个线程被快速的轮换执行，这就使得在宏观上具有多个线程同时执行的效果，并发不是真正的同时执行，并发可以使用下图表示。

![image-20230807160849047](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230807160849047.png)

##### 并行

并行指的是无论何时，多个线程都是在多个CPU核心上同时执行的，是真正的同时执行。

![image-20230807160903377](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230807160903377.png)

### 分治法

#### 基本思想

> 把一个规模大的问题划分为规模较小的子问题，然后分而治之，最后合并子问题的解得到原问题的解。

#### 步骤

①分割原问题；

②求解子问题；

③合并子问题的解为原问题的解。

我们可以使用如下伪代码来表示这个步骤。

```
if(任务很小）{
    直接计算得到结果
}else{
    分拆成N个子任务
    调用子任务的fork()进行计算
    调用子任务的join()合并计算结果
}
```

在分治法中，子问题一般是相互独立的，因此，经常通过递归调用算法来求解子问题。



### ForkJoin 特点：工作窃取

这个里面维护的都是双端队列

![image-20230807161113500](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230807161113500.png)

 

求和计算的任务:

```java
package com.ztc.ForkJoin;

import java.util.concurrent.RecursiveTask;

/**
 * @Author ztc
 * Date on 2023/8/7  16:12
 */

public class ForkJoinDemo extends RecursiveTask<Long> {

    private Long start;
    private Long end;

    //临界值
    private Long temp = 10000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }

    //计算方法
    @Override
    protected Long compute() {
        if ((end - start)<temp){
            Long sum = 0L;
            for (Long i = start; i < end; i++) {
                sum += i;
            }
            return sum;
        }else {
            //forkjoin递归
            long middle = (start + end) / 2;//中间值
            ForkJoinDemo task1 = new ForkJoinDemo(start, middle);
            task1.fork();//拆分任务，把任务压入线程队列
            ForkJoinDemo task2 = new ForkJoinDemo(middle + 1, end);
            task2.fork();
            return task1.join() + task2.join();
        }
    }
}

```

```java
package com.ztc.ForkJoin;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

/**
 * @Author ztc
 * Date on 2023/8/7  16:24
 */

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
//        test1();//时间5892
//        test2();//时间5099
        test3();//时间：140
    }
    public static void test1(){
        Long sum = 0L;
        long start = System.currentTimeMillis();
        for (Long i = 1L; i <= 10_0000_0000; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间" + (end-start));
    }

    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L, 10_0000_0000L);
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);
        Long sum = submit.get();

        long end = System.currentTimeMillis();

        System.out.println("sum="+sum+"时间"+(end-start));
    }

    public static void test3(){
        long start = System.currentTimeMillis();
        //Stream并行流
        Long sum = LongStream.range(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();

        System.out.println("sum="+sum+"时间："+(end-start));
    }

}

```

