# CAS(Compare-And-Swap)



### 什么是CAS机制

CAS机制是一种数据更新的方式。在具体讲什么是CAS机制之前，我们先来聊下在多线程环境下，对共享变量进行数据更新的两种模式：悲观锁模式和乐观锁模式。



**悲观锁**更新的方式认为：在更新数据的时候大概率会有其他线程去争夺共享资源，所以悲观锁的做法是：第一个获取资源的线程会将资源锁定起来，其他没争夺到资源的线程只能进入阻塞队列，等第一个获取资源的线程释放锁之后，这些线程才能有机会重新争夺资源。synchronized就是java中悲观锁的典型实现，synchronized使用起来非常简单方便，但是会使没争抢到资源的线程进入阻塞状态，线程在阻塞状态和Runnable状态之间切换效率较低（比较慢）。比如你的更新操作其实是非常快的，这种情况下你还用synchronized将其他线程都锁住了，线程从Blocked状态切换回Runnable华的时间可能比你的更新操作的时间还要长。

**乐观锁**更新方式认为:在更新数据的时候其他线程争抢这个共享变量的概率非常小，所以更新数据的时候不会对共享数据加锁。但是在正式更新数据之前会检查数据是否被其他线程改变过，如果未被其他线程改变过就将共享变量更新成最新值，如果发现共享变量已经被其他线程更新过了，就重试，直到成功为止。CAS机制就是乐观锁的典型实现。



**CAS的全称**：`Compare-And-Swap（比较并交换）`。比较变量的现在值与之前的值是否一致，若一致则替换，否则不替换。

**CAS的作用**：`原子性`更新变量值，保证线程安全。

**CAS指令**：需要有三个操作数，变量的`当前值（V）`，`旧的预期值（A）`，`准备设置的新值（B）`。

```java
//CAS:Compare-and-Swap比较并交换
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2023);

//public final boolean compareAndSet(int expect, int update)
        atomicInteger.compareAndSet(2023,2024);
        System.out.println(atomicInteger.get());
    }
}
```



## CAS机制优缺点

### 缺点

**1. ABA问题**
ABA问题：CAS在操作的时候会检查变量的值是否被更改过，如果没有则更新值，但是带来一个问题，最开始的值是A，接着变成B，最后又变成了A。经过检查这个值确实没有修改过，因为最后的值还是A，但是实际上这个值确实已经被修改过了。为了解决这个问题，在每次进行操作的时候加上一个版本号，每次操作的就是两个值，一个版本号和某个值，A——>B——>A问题就变成了1A——>2B——>3A。在jdk中提供了AtomicStampedReference类解决ABA问题，用Pair这个内部类实现，包含两个属性，分别代表版本号和引用，在compareAndSet中先对当前引用进行检查，再对版本号标志进行检查，只有全部相等才更新值。

```java
package com.ztc.cas;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * @Author ztc
 * Date on 2023/8/9  12:58
 */

//CAS:Compare-and-Swap比较并交换
public class CASDemo {
    public static void main(String[] args) {

        AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1, 1);

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();//获得版本号
            System.out.println("a1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicStampedReference.compareAndSet(1,2,
                    atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);
            System.out.println("a2=>"+atomicStampedReference.getStamp());

            System.out.println(atomicStampedReference.compareAndSet(2, 1,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1));

            System.out.println("a3=>"+atomicStampedReference.getStamp());

        },"a").start();

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();//获得版本号
            System.out.println("b1=>"+stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(atomicStampedReference.compareAndSet(1,6,
                    stamp,stamp+1));

            System.out.println("b2=>"+atomicStampedReference.getStamp());

        },"b").start();
    }
}

```

![image-20230809163921401](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230809163921401.png)

![image-20230809164814811](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230809164814811.png)

如果泛型是个包装类，注意对象的引用问题

![image-20230809163823475](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230809163823475.png)

**2. 可能会消耗较高的CPU**
看起来CAS比锁的效率高，从阻塞机制变成了非阻塞机制，减少了线程之间等待的时间。每个方法不能绝对的比另一个好，在线程之间竞争程度大的时候，如果使用CAS，每次都有很多的线程在竞争，也就是说CAS机制不能更新成功。这种情况下CAS机制会一直重试，这样就会比较耗费CPU。因此可以看出，如果线程之间竞争程度小，使用CAS是一个很好的选择；但是如果竞争很大，使用锁可能是个更好的选择。在并发量非常高的环境中，如果仍然想通过原子类来更新的话，可以使用AtomicLong的替代类：LongAdder。

**3. 不能保证代码块的原子性**
Java中的CAS机制只能保证共享变量操作的原子性，而不能保证代码块的原子性。



### 优点

- 可以保证变量操作的原子性；
- 并发量不是很高的情况下，使用CAS机制比使用锁机制效率更高；
- 在线程对共享资源占用时间较短的情况下，使用CAS机制效率也会较高。



### Java提供的CAS操作类--Unsafe类

从Java5开始引入了对CAS机制的底层的支持，在这之前需要开发人员编写相关的代码才可以实现CAS。在原子变量类Atomic*中（例如AtomicInteger、AtomicLong）可以看到CAS操作的代码，在这里的代码都是调用了底层（核心代码调用native修饰的方法）的实现方法。在AtomicInteger源码中可以看getAndSet方法和compareAndSet方法之间的关系，compareAndSet方法调用了底层的实现，该方法可以实现与一个volatile变量的读取和写入相同的效果。在前面说到了volatile不支持例如i++这样的复合操作，在Atomic中提供了实现该操作的方法。JVM对CAS的支持通过这些原子类（Atomic*）暴露出来，供我们使用。

而Atomic系类的类底层调用的是Unsafe类的API，Unsafe类提供了一系列的compareAndSwap*方法，下面就简单介绍下Unsafe类的API：

- long objectFieldOffset（Field field）方法：返回指定的变量在所属类中的内存偏移地址，该偏移地址仅仅在该Unsafe函数中访问指定字段时使用。如下代码使用Unsafe类获取变量value在AtomicLong对象中的内存偏移。

```java
static {
   try {
       valueOffset = unsafe.objectFieldOffset
           (AtomicInteger.class.getDeclaredField("value"));
   } catch (Exception ex) { throw new Error(ex); }
}
```

- int arrayBaseOffset（Class arrayClass）方法：获取数组中第一个元素的地址。
- int arrayIndexScale（Class arrayClass）方法：获取数组中一个元素占用的字节。
- boolean compareAndSwapLong（Object obj, long offset, long expect, long update）方法：比较对象obj中偏移量为offset的变量的值是否与expect相等，相等则使用update值更新，然后返回true，否则返回false。
- public native long getLongvolatile（Object obj, long offset）方法：获取对象obj中偏移量为offset的变量对应volatile语义的值。
- void putLongvolatile（Object obj, long offset, long value）方法：设置obj对象中offset偏移的类型为long的field的值为value，支持volatile语义。
- void putOrderedLong（Object obj, long offset, long value）方法：设置obj对象中offset偏移地址对应的long型field的值为value。这是一个有延迟的putLongvolatile方法，并且不保证值修改对其他线程立刻可见。只有在变量使用volatile修饰并且预计会被意外修改时才使用该方法。
- void park（boolean isAbsolute, long time）方法：阻塞当前线程，其中参数isAbsolute等于false且time等于0表示一直阻塞。time大于0表示等待指定的time后阻塞线程会被唤醒，这个time是个相对值，是个增量值，也就是相对当前时间累加time后当前线程就会被唤醒。如果isAbsolute等于true，并且time大于0，则表示阻塞的线程到指定的时间点后会被唤醒，这里time是个绝对时间，是将某个时间点换算为ms后的值。另外，当其他线程调用了当前阻塞线程的interrupt方法而中断了当前线程时，当前线程也会返回，而当其他线程调用了unPark方法并且把当前线程作为参数时当前线程也会返回。
- void unpark（Object thread）方法：唤醒调用park后阻塞的线程。