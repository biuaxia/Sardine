title: 【转载】聊聊传说中的 ThreadLocal
date: 2021-06-18 23:00:59
toc: true
category: 
 - 转载
 - Java
tags: 
 - 转载
 - ThreadLocal
 - Java
 - 传说
 - 聊聊
---

本文转载自：[聊聊传说中的 ThreadLocal | Nicksxs's Blog](https://nicksxs.me/2021/05/30/%E8%81%8A%E8%81%8A%E4%BC%A0%E8%AF%B4%E4%B8%AD%E7%9A%84-ThreadLocal/)

---

说来也惭愧，这个 ThreadLocal 其实一直都是一知半解，而且看了一下之后还发现记错了，所以还是记录下
原先记忆里的都是反过来，一个 ThreadLocal 是里面按照 thread 作为 key，存储线程内容的，真的是半解都米有，完全是错的，这样就得用 concurrentHashMap 这种去存储并且要锁定线程了，然后内容也只能存一个了，想想简直智障。


<!-- more -->


### 究竟是啥结构

比如我们在代码中 new 一个 ThreadLocal，

```java
public static void main(String[] args) {
        ThreadLocal<Man> tl = new ThreadLocal<>();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(tl.get());
        }).start();
        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            tl.set(new Man());
        }).start();
    }

    static class Man {
        String name = "nick";
    }
```

这里构造了两个线程，一个先往里设值，一个后从里取，运行看下结果，
[![](https://b3logfile.com/file/2021/06/solo-fetchupload-4364314004435277633-2aff4abf.png)](https://b3logfile.com/file/2021/06/solo-fetchupload-4364314004435277633-2aff4abf.png)
知道这个用法的话肯定知道是取不到值的，只是具体的原理原来搞错了，我们来看下设值 set 方法

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

写博客这会我才明白我原来咋会错得这么离谱，看到第一行代码 t 就是当前线程，然后第二行就是用这个线程去`getMap`，然后我是把这个当成从 map 里取值了，其实这里是

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

获取 t 的 threadLocals 成员变量，那这个 threadLocals 又是啥呢
[![](https://b3logfile.com/file/2021/06/solo-fetchupload-7542247463152618388-f3cdbad4.png)](https://b3logfile.com/file/2021/06/solo-fetchupload-7542247463152618388-f3cdbad4.png)
它其实是线程 Thread 中的一个类型是java.lang.ThreadLocal.ThreadLocalMap的成员变量
这是 ThreadLocal 的一个静态成员变量

```java
static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    }
```

全部代码有点长，只截取了一小部分，然后我们再回头来分析前面说的 set 过程，再 copy 下代码

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

获取到 map 以后呢，如果 map 不为空，就往 map 里 set，这里注意 key 是啥，其实是当前这个 ThreadLocal，这里就比较明白了究竟是啥结构，每个线程都会维护自身的 ThreadLocalMap，它是线程的一个成员变量，当创建 ThreadLocal 的时候，进行设值的时候其实是往这个 map 里以 ThreadLocal 作为 key，往里设 value。

### 内存泄漏是什么鬼

这里又要看下前面的 ThreadLocalMap 结构了，类似 HashMap，它有个 Entry 结构，在设置的时候会先包装成一个 Entry

```java
private void set(ThreadLocal<?> key, Object value) {

        // We don't use a fast path as with get() because it is at
        // least as common to use set() to create new entries as
        // it is to replace existing ones, in which case, a fast
        // path would fail more often than not.

        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);

        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();

            if (k == key) {
                e.value = value;
                return;
            }

            if (k == null) {
                replaceStaleEntry(key, value, i);
                return;
            }
        }

        tab[i] = new Entry(key, value);
        int sz = ++size;
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            rehash();
}
```

这里其实比较重要的就是前面的 Entry 的构造方法，Entry 是个 WeakReference 的子类，然后在构造方法里可以看到 key 会被包装成一个弱引用，这里为什么使用弱引用，其实是方便这个 key 被回收，如果前面的 ThreadLocal tl实例被设置成 null 了，如果这里是直接的强引用的话，就只能等到线程整个回收了，但是其实是弱引用也会有问题，主要是因为这个 value，如果在 ThreadLocal tl 被设置成 null 了，那么其实这个 value 就会没法被访问到，所以最好的操作还是在使用完了就 remove 掉

