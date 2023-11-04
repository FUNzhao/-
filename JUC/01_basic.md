#### 继承Thread类

- 子类继承Thread类具备多线程能力
- 启动线程：子类对象start()
- 不建议使用：避免OOP单继承局限性

#### 实现Runnable接口

- 实现接口Runnable具有多线程能力
- 启动线程：传入目标对象+Thread对象.start()
- 推荐使用：避免单继承局限性，灵活方便，方便同一个对象被多个线程使用

#### 关于OOP单继承局限性：

单继承的意思是，**一个子类只拥有一个父类**
具体解释：
若有一个A类，一个B类，且A,B类独立（没有共同父类，也相互不为父类）。
新建一个类C，想要同时继承A,B类，在JAVA中无法直接实现。
要实现，只能复制A类的内容，再继承B类，或者反过来，这将会很麻烦，而且代码会很丑陋。

局限性由上面可以得到：想要得到两个类的特性是不能通过继承实现。

而接口出现可以解决这个问题。
因为，一个类可以实现多个接口

最后解释一下Runnable是如何避免java单继承带来的局限性的。
如果你想写一个类C，但这个类C已经继承了一个类A，此时，你又想让C实现多线程。用继承Thread类的方式不行了。（因为单继承的局限性）
此时，只能用Runnable接口。
Runnable接口就是为了解决这种情境出现的。

```java
package com.ztc.demo01;

/**
 * @Author ztc
 * Date on 2023/7/12  17:01
 */

public class TestThread01 extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println("我在看代码--"+i);
        }
    }

    public static void main(String[] args) {

        //创建一个线程对象
        TestThread01 testThread01 = new TestThread01();

        //调用start()方法开启
        testThread01.start();

        for (int i = 0; i < 2000; i++) {
            System.out.println("我在学习多线程--"+i);
        }
    }
}

```

```java
package com.ztc.demo01;

/**
 * @Author ztc
 * Date on 2023/7/12  17:27
 */

public class TestThread02 implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            System.out.println("我在看代码---"+i);
        }
    }

    public static void main(String[] args) {
        //创建一个线程对象
        TestThread02 testThread02 = new TestThread02();

        new Thread(testThread02).start();
        for (int i = 0; i < 1000; i++) {
            System.out.println("我在学习多线程--"+i);
        }
    }
}

```



两个线程交替执行。线程开启不一定立即执行，由CPU调度执行

![image-20230712170905205](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230712170905205.png)



### 线程礼让

1. 线程礼让，让当前正在执行的线程暂停,但不阻塞；
2. 将线程从运行状态转为就绪状态；
3. 让CPU重新调度,礼让不一定成功,看CPU心情。



### 线程强制执行join

在线程操作中，可以使用 `join()` 方法让一个线程强制运行，线程强制运行期间，其他线程无法运行，必须等待此线程完成之后才可以继续执行



### 线程的优先级

优先级越高的线程，CPU越是尽量将资源给这个线程，但是并不代表优先级高，就要等着把优先级高的线程执行完了才会去执行优先级低的线程，不是这样的，只是说CPU会尽量执行高优先级线程，低优先级线程也有机会得到执行，只是机会少一些。

#### 1 优先级取值范围

Java 线程优先级使用 1 ~ 10 的整数表示：

- 最低优先级 1：`Thread.MIN_PRIORITY`
- 最高优先级 10：`Thread.MAX_PRIORITY`
- 普通优先级 5：`Thread.NORM_PRIORITY`

#### 2 获取线程优先级

```java
public static void main(String[] args) {
    System.out.println(Thread.currentThread().getPriority());
}
```

#### 3 设置优先级

Java 使用 `setPriority` 方法设置线程优先级，方法签名

```java
public final void setPriority(int newPriority)
```

newPriority 设置范围在 1-10，否则抛出 java.lang.IllegalArgumentException 异常



### Synchronized

**synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：** 

　　1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 
　　2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
　　3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
　　4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。

当两个并发线程(thread1和thread2)访问同一个对象(syncThread)中的synchronized代码块时，**在同一时刻只能有一个线程得到执行，另一个线程受阻塞，必须等待当前线程执行完这个代码块以后才能执行该代码块**。Thread1和thread2是互斥的，因为在执行synchronized代码块时会锁定当前的对象，只有执行完该代码块才能释放该对象锁，下一个线程才能执行并锁定该对象。 

eg.

```java
public class UnsafeBuyTicket {
    public static void main(String[] args) {
        BuyTicket station = new BuyTicket();

        new Thread(station,"苦逼的我").start();
        new Thread(station,"厉害的你们").start();
        new Thread(station,"黄牛").start();
    }
}

class BuyTicket implements Runnable{

    private int ticketNums = 10;
    boolean flag = true;

    @Override
    public void run() {
         //买票
        while (flag){
            try {
                buy();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    private synchronized void buy() throws InterruptedException {
        //判断是否有票
        if (ticketNums<=0){
            flag = false;
            return;
        }
        //模拟延时
        Thread.sleep(100);
        //买票
        System.out.println(Thread.currentThread().getName()+"拿到"+ticketNums--);
    }
}
```

![image-20230714174505394](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230714174505394.png)

#### 总结：

1、 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。 
2、每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。 
3、实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制



### Lock

lock是显示式锁（手动开启和关闭锁）synchronized是隐式锁，出了作用域自动释放

lock只有代码块锁，synchronized有代码块锁和方法锁

使用lock锁，jvm将花费较少的时间来调度线程，性能更好。而且具有更好的扩展性（提供更多的子类）



### 线程池及其应用

[Java并发系列终结篇：彻底搞懂Java线程池的工作原理 - 掘金 (juejin.cn)](https://juejin.cn/post/6983213662383112206/)
