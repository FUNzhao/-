### **生产者和消费者问题**

生产者和消费者问题 Synchronized 版

```java
package com.ztc.demo02;

/**
 * @Author ztc
 * Date on 2023/8/4  15:11
 */

public class A {

    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

// 判断等待，业务，通知
class Data { // 数字 资源类
    private int number = 0;

    //+1
    public synchronized void increment() throws InterruptedException {
        if (number != 0) { //0
// 等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "=>" + number);
// 通知其他线程，我+1完毕了
        this.notifyAll();
    }

    //-1
    public synchronized void decrement() throws InterruptedException {
        if (number == 0) { // 1
           // 问题存在，A B C D 4 个线程！虚假唤醒
            //if 改为 while 判断
// 等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "=>" + number);
// 通知其他线程，我-1完毕了
        this.notifyAll();
    }
}


```

