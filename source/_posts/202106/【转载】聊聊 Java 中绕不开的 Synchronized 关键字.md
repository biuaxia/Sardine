title: 【转载】聊聊 Java 中绕不开的 Synchronized 关键字
date: 2021-06-22 20:49:55
comment: false
toc: true
category: 
 - Java
tags:
 - 转载
 - Java
 - Synchronized
 - 锁
---

本文转载自：[聊聊 Java 中绕不开的 Synchronized 关键字 | Nicksxs's Blog](https://nicksxs.me/2021/06/20/%E8%81%8A%E8%81%8A-Java-%E4%B8%AD%E7%BB%95%E4%B8%8D%E5%BC%80%E7%9A%84-Synchronized-%E5%85%B3%E9%94%AE%E5%AD%97/)

---

Synchronized 关键字在 Java 的并发体系里也是非常重要的一个内容，首先比较常规的是知道它使用的方式，可以锁对象，可以锁代码块，也可以锁方法。


<!-- more -->


看一个简单的 demo

```java
public class SynchronizedDemo {

    public static void main(String[] args) {
        SynchronizedDemo synchronizedDemo = new SynchronizedDemo();
        synchronizedDemo.lockMethod();
    }

    public synchronized void lockMethod() {
        System.out.println("here i'm locked");
    }

    public void lockSynchronizedDemo() {
        synchronized (this) {
            System.out.println("here lock class");
        }
    }
}
```

然后来查看反编译结果，其实代码（日光）之下并无新事，即使是完全不懂的也可以通过一些词义看出一些意义

```java
  Last modified 2021年6月20日; size 729 bytes
  MD5 checksum dd9c529863bd7ff839a95481db578ad9
  Compiled from "SynchronizedDemo.java"
public class SynchronizedDemo
  minor version: 0
  major version: 53
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #2                          // SynchronizedDemo
  super_class: #9                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 4, attributes: 1
Constant pool:
   #1 = Methodref          #9.#22         // java/lang/Object."<init>":()V
   #2 = Class              #23            // SynchronizedDemo
   #3 = Methodref          #2.#22         // SynchronizedDemo."<init>":()V
   #4 = Methodref          #2.#24         // SynchronizedDemo.lockMethod:()V
   #5 = Fieldref           #25.#26        // java/lang/System.out:Ljava/io/PrintStream;
   #6 = String             #27            // here i\'m locked
   #7 = Methodref          #28.#29        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #8 = String             #30            // here lock class
   #9 = Class              #31            // java/lang/Object
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               lockMethod
  #17 = Utf8               lockSynchronizedDemo
  #18 = Utf8               StackMapTable
  #19 = Class              #32            // java/lang/Throwable
  #20 = Utf8               SourceFile
  #21 = Utf8               SynchronizedDemo.java
  #22 = NameAndType        #10:#11        // "<init>":()V
  #23 = Utf8               SynchronizedDemo
  #24 = NameAndType        #16:#11        // lockMethod:()V
  #25 = Class              #33            // java/lang/System
  #26 = NameAndType        #34:#35        // out:Ljava/io/PrintStream;
  #27 = Utf8               here i\'m locked
  #28 = Class              #36            // java/io/PrintStream
  #29 = NameAndType        #37:#38        // println:(Ljava/lang/String;)V
  #30 = Utf8               here lock class
  #31 = Utf8               java/lang/Object
  #32 = Utf8               java/lang/Throwable
  #33 = Utf8               java/lang/System
  #34 = Utf8               out
  #35 = Utf8               Ljava/io/PrintStream;
  #36 = Utf8               java/io/PrintStream
  #37 = Utf8               println
  #38 = Utf8               (Ljava/lang/String;)V
{
  public SynchronizedDemo();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 5: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #2                  // class SynchronizedDemo
         3: dup
         4: invokespecial #3                  // Method "<init>":()V
         7: astore_1
         8: aload_1
         9: invokevirtual #4                  // Method lockMethod:()V
        12: return
      LineNumberTable:
        line 8: 0
        line 9: 8
        line 10: 12

  public synchronized void lockMethod();
    descriptor: ()V
    flags: (0x0021) ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #6                  // String here i\'m locked
         5: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 13: 0
        line 14: 8

  public void lockSynchronizedDemo();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter
         4: getstatic     #5                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #8                  // String here lock class
         9: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit
        20: aload_2
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
      LineNumberTable:
        line 17: 0
        line 18: 4
        line 19: 12
        line 20: 22
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 17
          locals = [ class SynchronizedDemo, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
}
SourceFile: "SynchronizedDemo.java"
```

其中`lockMethod`中可以看到是通过 `ACC_SYNCHRONIZED` flag 来标记是被 synchronized 修饰，前面的 ACC 应该是 access 的意思，并且通过 `ACC_PUBLIC` 也可以看出来他们是同一类访问权限关键字来控制的，而修饰类则是通过`3: monitorenter`和`13: monitorexit`来控制并发，这个是原来就知道，后来看了下才知道修饰方法是不一样的，但是在前期都比较诟病是 synchronized 的性能，像 monitor 也是通过操作系统的`mutex lock`互斥锁来实现的，相对是比较重的锁，于是在 JDK 1.6 之后对 synchronized 做了一系列优化，包括偏向锁，轻量级锁，并且包括像 ConcurrentHashMap 这类并发集合都有在使用 synchronized 关键字配合 cas 来做并发保护，

jdk 对于 synchronized 的优化主要在于多重状态锁的升级，最初会使用偏向锁，当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，所以偏向锁的撤销操作的性能损耗必须小于节省下来的CAS原子指令的性能消耗）。
而当出现线程尝试进入同步块时发现已有偏向锁，并且是其他线程时，会将锁升级成轻量级锁，并且自旋尝试获取锁，如果自旋成功则表示获取轻量级锁成功，否则将会升级成重量级锁进行阻塞，当然这里具体的还很复杂，说的比较浅薄主体还是想将原先的阻塞互斥锁进行轻量化，区分特殊情况进行加锁。
