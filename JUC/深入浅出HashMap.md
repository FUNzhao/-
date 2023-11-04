# 1、什么是HashMap

  `java.util.HashMap`是一个用于存储**键值对**的集合，每一个键值对称为 **Entry**。这些键值对**分散存储**在一个数组当中，这个数组就是 HashMap 的主干。

HashMap的基本使用：

```java
public class Test {
    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("klb", 18);
        map.put("smm", 12);
        System.out.println(map);
    }
}
```

![image-20230805144703614](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805144703614.png)

HashMap 数组每一个元素的初始值都是 Null。

![image-20230805144719276](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805144719276.png)

# 2、构造函数

  通过源码可以看出总共有四个构造函数：

![image-20230805144912772](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805144912772.png)

![image-20230805144931192](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805144931192.png)

![image-20230805144940355](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805144940355.png)

![image-20230805144952569](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805144952569.png)

可以看出，构造函数涉及到两个参数：`loadFactor`和`initialCapacity`，在`java.util.HashMap`中有两个属性：

![image-20230805145246561](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805145246561.png)

当两个参数没有指定值，则有默认值。两个参数分别叫做加载因子和初始化容量。

  这两个参数是干嘛的呢？

  在构造函数中只看到这些参数赋值给了属性，但没有使用，更没有看到有创建数组的地方。不是说 HashMap 是数组加链表么？

  其实，HashMap 可以看成懒加载，当没有数据 put 进来的时候，是不会创建数组的，防止浪费空间。

3、table 的创建
  我们使用map.put("klb",18)就把数据插入到 HashMap 当中了，这中间发生了什么呢？

  我们查看源码：

![image-20230805172352163](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172352163.png)

  table 指的就是 HashMap 的数组部分，第一个 if 判断就是判断这个 table 是否创建了，如果没有创建，则创建一个，我们进入这个inflateTable(threshold)查看源码：

![image-20230805172411118](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172411118.png)

  创建 table 的过程：

  1、没有指定的情况下，默认的loadFactor为0.75，默认的initialCapacity为16。首先是对 table 的容量进行纠正：

![image-20230805172429104](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172429104.png)

  这里的capacity就是大于等于toSize的数字中，最近的那个 2 的幂。默认值是 16 ，因此这里输出也是 16。如果在构造器中输入了initialCapacity为 18，那么大于 18 又是最近的 2 的幂，只能是 32，这个函数会返回 32 。总之，这个roundUpToPowerOf2就是为了保证 table 的容量既能满足程序员的要求，又必须是 2 的幂。

![image-20230805172449136](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172449136.png)

  3、更新阈值为capacity * loadFactor

  后面那个MAXIMUM_CAPACITY值为 1<<30，也就是 230= 1,073,741,824，一般是达不到这么大的数值。我们可以看成阈值就是：table 的容量和负载因子的乘积。

![image-20230805172527832](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172527832.png)

  4、确定了 table 的容量，接着就是直接创建容量大小的数组：

![image-20230805172541445](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172541445.png)

  可以得出：table 是一个 Entry[] 类型的数组，那 Entry 又是什么呢？查看 Entry 的定义：

![image-20230805172653409](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172653409.png)

  Entry 就是我们操作的键值对，他的 key 是 final 类型表示不可修改，有 next 属性表示可以多个 Entry 连城一个链表，带有 hash 值。

  因此，我们可以得出结论：JDK7 的 HashMap 就是数组+链表的形式。


# 4、Put 方法

![image-20230805172806354](C:/Users/DELL/AppData/Roaming/Typora/typora-user-images/image-20230805172806354.png)





## 并发下线程不安全

  我们上面分析了那么多源码，从来没看到任何同步机制在里面，当多个线程并发操作一个 HashMap 时，会引发线程不安全问题。

  那线程不安全会提现在哪里呢？

  首先，get方法肯定不会带来线程安全问题，问题就出在put方法中，上面仔细研究代码，如果两个线程要put的 key 是不同的，其实也没有线程不安全问题；如果要put的 key 是相同的，那更没有问题了，因为 HashMap 是保证 key 唯一的，后插入的数据直接覆盖前面插入的，不会出现两个一样 key 的数据。

  经过分析，线程不安全就出在 resize 当中。

  当两个线程操作一个 HashMap，且两个线程都判断出应该 resize 了，其中一个线程已经完成了 resize，并且把 table 更新为新的 newTable，但是另一个线程还处于transfer方法中，最后会导致循环引用的问题。
