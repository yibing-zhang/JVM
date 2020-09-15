# 硬件层面是如何保证数据一致性的？
# 1.存储器的层次结构

![图片](https://uploader.shimo.im/f/uAlySNZBDjag2yvY.png!thumbnail)

距离CPU越近，速度越快，越远速度越慢

CPU速度特别快，比内存快100个数量级，比硬盘快100万个数量级，最慢的是远程存储

# 2.多核CPU导致数据不一致问题

加入一个数x被load到L3缓存上，这时候有CPU1 和CPU2把这个数都加载进了各自的CPU缓存里，CPU1把x改为了1，CPU2把X改为了2，由于是在不同的CPU缓存中，对方无法感知对方对X做的操作，如果把数据同时写会L3，就会产生数据不一致的问题

# 3.缓存行(x86:64字节)


* 缓存行概念？

我们把内存里某些数据放进CPU缓存里，他不是只把这一个数据放进去，假如CPU用到了int类型的一个数据，他不会只加载这4个字节的数，他会把这个数据所在行的所有数据读取进来，这就是缓存行(Cache Line)


* 缓存行对齐？

使用缓存行能够提高效率，在我本机上测试使用缓存行的效率大概是不使用缓存行效率的3倍

```java
public class CacheLinePadding {
  private static class Padding {
      //创建一个7个long类型变量的类，由于long类型是8个字节，先补全56个字节
      public volatile long pa, p2, p3, p4, p5, p6, p7;
  }
  private static class T extends Padding {
      public volatile long x = 0L;
  }
  public static T[] arr = new T[2];
  static {
      arr[0] = new T();
      arr[1] = new T();
  }
  public static void main(String[] args) throws InterruptedException {
      Thread t1 = new Thread(() -> {
          for (int i = 0; i < 1000_0000L; i++) {
              arr[0].x = i;
          }
      });
      Thread t2 = new Thread(() -> {
          for (int i = 0; i < 1000_0000L; i++) {
              arr[1].x = i;
          }
      });
      final long start = System.nanoTime();
      t1.start();
      t2.start();
      t1.join();
      t2.join();
      System.out.println((System.nanoTime() - start)/100_0000);
  }
}
```

* 伪共享？

    * 读取缓存以cache line为单位读取，目前为**64**bytes
    * 位于同一缓存行的不同数据，被两个不同的CPU锁定，产生互相影响的伪共享问题

缓存行经典使用案例Disruptor


* 如何解决数据一致性？

    1. 锁总线(效率低下)：在L2和L3中间加个锁，第一个CPU访问数据的时候把总线锁住，不允许另外一个线程去访问，这是老CPU干的事情
    2. MESI协议 + 锁总线：

---


# 乱序问题
CPU为了提高指令的执行效率，会在一条指令执行过程中（比如去内存读数据，这个过程比CPU的执行效率慢100倍），去同时执行另一条指令，前提是两条指令没有依赖关系

#### 1.硬件层面如何保证有序性(X86CPU内存屏障)？


* sfence：save 屏障
* Ifence：load 屏障
* mfence：modify 屏障
* 原子指令，如x86上的lock...，指令是一个Full Barrier，执行的时候会锁住内存子系统来确保执行顺序，甚至多个CPU，Software Locks通常使用了内存屏障或原子指令来实现变量可见性和保持顺序
#### 2.JVM层面如何保证有序性？


* LoadLoad
>在load2以及后续读取操作要读取的数据被访问之前，要保证load1要读取的数据被读取完毕
* StoreStore
>在store2以及后续写入操作要读取的数据被访问之前，要保证store1写入的数据对其他处理器可见
* LoadStore
>在store2以及后续写入操作要读取的数据被访问之前，保证Load1要读取的数据被读取完毕
* StoreLoad
>在load2以及后续所有读取操作执行前，保证store1的写入对所有的处理器可见
#### 3.Volatile是如何实现内存屏障的？（有同学面试被问）


* **字节码层面**：加ACC_VOLATILE标记
* **JVM层面**：volatile内存区的读写，都加屏障
>StoreStoreBarrier
>volatile 写操作
>StoreLoadBarrier
>LoadLoadBarrier
>volatile 读操作
>LoadStoreBarrie
* **OS和硬件层面**
#### 4.Synchronize的实现细节


* **字节码层面**：

    * 修饰方法名：ACC_SYNCHRONIZED
    * 修饰代码块：monitorenter/ monitorexit
* **JVM层面**：c和c++调用了操作系统提供的同步机制
* **OS和硬件层面（x86）：**基本上是一个lock指令

