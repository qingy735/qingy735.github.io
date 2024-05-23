---
title: Java八股-并发编程篇
tags:
  - 并发编程
  - Java
categories: 八股
abbrlink: 27654
date: 2024-05-22 16:47:21
---

# Java八股-并发编程篇

## 基础

### 并行和并发区别

并行是指多个处理器同时执行多个任务，同一时间多个任务同时进行

并发是指在单处理器上一个时间段内多个任务同时执行，但是某一时刻只有一个任务执行



### 进程和线程

- 进程是CPU资源分配的最小单位，线程是CPU调度的最小单位
- 进程执行在线程中，一个进程可以拥有多个线程
- 进程之间的切换是耗时的，同一个进程间的线程进行切换很快
- 同一个进程内的线程共享进程中的资源
- 不同进程间的资源相互独立

## 线程创建方式

- 继承Thread类

> Java不支持多继承，如果已经继承了其他类则不能使用该方法

```java
class ThreadExam extends Thread {
    public void run() {
        System.out.println("继承Thread创建线程...");
    }
    
    public static void main(String[] args) {
        ThreadExam exam = new ThreadExam();
        exam.start();
    }
}
```



- 实现Runnable接口

```java
class RunnableExam implements Runnable {
    public void run() {
        System.out.println("实现Runnable接口创建线程...");
    }

    public static void main(String[] args) {
        RunnableExam exam = new RunnableExam();
        Thread thread = new Thread(exam);
        thread.start();
    }
}
```



- 实现Callable接口

> 可以结合FutureTask，通过get方法获取执行结果

```java
class CallableExam implements Callable<String> {
    public String call() {
        return "实现Callable接口创建线程..."";
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CallableExam exam = new CallableExam();
        FutureTask<String> futureTask = new FutureTask<>(exam);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
    }
}
```

### 为什么不直接调用线程的**run()**方法而去执行**start()**

通过调用**start()**方法来告知JVM需要创建新的线程并准备好需要的资源，然后在新建线程中执行**run()**方法，直接调用**run()**方法相当于只是调用了Thread中的一个普通方法，执行还是在主线程中。

### 线程常用调度方法

- wait()

  A线程调用该方法将被阻塞挂起直到其他线程调用notify（notifyAll）或调用A线程的interrupt方法。其使用前提必须是在同步代码块中，并且当前线程拥有该对象的锁

  ```java
  synchronized (obj) {
  	while (<condition does not hold>)
  		obj.wait();
  		... // Perform action appropriate to condition
  }
  ```

- wait(long timeout)

  相比于wait方法，在指定时间超时后会自动返回

- notify()

  A线程调用该方法将随机唤醒阻塞在当前加锁变量上的一个线程，直到当前线程放弃当前锁，唤醒的线程可以正常地争抢该锁。使用前提也必须是在同步代码块中，并且由加锁对象执行调用。

  ```java
  synchronized (obj) {
  	while (<condition does not hold>)
  		obj.notify();
  		... // Perform action appropriate to condition
  }
  ```

- notifyAll()

  相比于notify，该方法将唤醒所有阻塞在当前锁上的对象，使用条件相同

- sleep(long millis)

  暂时让出执行时间的执行权，但是获取的锁仍然保持，等到时间到了会继续获取CPU资源，然后正常运行。

- yield()

  Thread类的静态方法，一个线程调用yield后，表示当前线程请求让出CPU**（注意：不会释放锁，只是从运行状态转移到就绪状态）**

- join()

  A、B线程，B线程调用A.join()，此时B线程会进入阻塞队列，直到线程A运行结束或线程B中断。join方法会释放锁

### 线程状态

|     状态     |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|     NEW      |        初始状态：线程被创建，但还没有调用 start()方法        |
|   RUNNABLE   | 运行状态：Java 线程将操作系统中的就绪和运行两种状态笼统的称作“运行” |
|   BLOCKED    |                  阻塞状态：表示线程阻塞于锁                  |
|   WAITING    | 等待状态：表示线程进入等待状态，进入该状态表示当前线程需要等待其他线程做出一些特定动作（通知或中断） |
| TIME_WAITING | 超时等待状态：该状态不同于 WAITIND，它是可以在指定的时间自行返回的 |
|  TERMINATED  |              终止状态：表示当前线程已经执行完毕              |

![Java线程状态变化](https://gitee.com/qingy735/blogimg/raw/master/img/javathread-7.png)

### 线程间通信方式

- volatile和synchronized
- 等待/通知机制（wait、notify等）
- 管道输入/输出流
- Thread.join()
- ThreadLocal

## ThreadLocal

### 实现原理

```java
public
class Thread implements Runnable {
    ...
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    ....
}
```

#### set方法

每个线程都有一个ThreadLocalMap对象，在使用set方法时首先通过**Thread.currentThread()**获取当前线程，然后调用内部方法获取到当前线程的ThreadLocalMap对象，判断是否为空，为空则调用**createMap**方法直接创建并赋值；不为空则以当前ThreadLocal对象为key存储。

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

```java
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```

```java
void createMap(Thread t, T firstValue) {
	t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

#### get方法

同样首先获取当前线程，然后获得线程中的ThreadLocalMap对象，判断是否为空，为空直接调用**setInitialValue**方法（与set方法相同，仅将value修改为null）；不为空则通过当前ThreadLocal对象获取value并返回。

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

#### remove方法

通过**getMap**方法获取当前线程的ThreadLocalMap对象，非空则执行ThreadLocalMap内部**remove**方法

```java
public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
         m.remove(this);
 }
```

### ThreadLocalMap

ThreadLocalMap是ThreadLocal类的静态内部类，它是一个定制的哈希表，专门用于保存每个线程中的线程局部变量。

#### Entry

Entry继承了弱引用**WeakReference<ThreadLocal<?>>**，value用于存储与特定ThreadLocal对象关联的值。因为Entry的key为弱引用，所以当ThreadLocal外部的强引用被置为null，则根据可达性分析，ThreadLocal将会在下次GC中被回收，此时ThreadLocalMap就会出现key为null的Entry，如果线程不结束则key为null的value会一直存在一条强引用链，导致无法回收造成内存泄漏。**Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value**

![ThreadLocal各引用间的关系](https://gitee.com/qingy735/blogimg/raw/master/img/ThreadLocal-01.png)

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```



#### set方法

```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    // 获取当前key的hash值
    int i = key.threadLocalHashCode & (len-1);
	// 防止地址冲突
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        // 获取当前hash对应的Entry中的ThreadLocal对象
        ThreadLocal<?> k = e.get();
		// 为当前key则直接更新
        if (k == key) {
            e.value = value;
            return;
        }
		// 为空则替换掉当前槽位的key，防止空值key导致的内存泄漏
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
	// 当前位置未被初始化过
    tab[i] = new Entry(key, value);
    int sz = ++size;
    // 插入后再次清除一些key为null的“脏”entry,如果大于阈值就需要扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```

#### 扩容

ThreadLocalMap默认大小为16，阈值为2/3，超过阈值则准备扩容，首先会将key为null的entry的value设置为null便于垃圾回收，然后判断当前长度是否大于阈值的3/4，如果大于则进行扩容。

```java
private static final int INITIAL_CAPACITY = 16;

ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}

private void setThreshold(int len) {
    threshold = len * 2 / 3;
}
```



```java
private void rehash() {
    expungeStaleEntries();

    // 清理完key为null的Entry后长度仍大于阈值的3/4则进行扩容
    if (size >= threshold - threshold / 4)
        resize();
}
```



#### remove方法

首先获取当前ThreadLocal对应的key，然后执行clear方法，然后向后清理脏Entry数据

```java
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         // 哈希碰撞
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            // 从该hash值对应索引向后开始查询，因为与当前索引发生碰撞后只会向后赋值
            // 清除掉当前key后向后查询已有的key是否是通过rehash得到的，判断获取到的ThreadLocal是否为null，如果是则清除，
            // 不是则重新进行hash计算并赋值，直到下一个槽位为null
            expungeStaleEntry(i);
            return;
        }
    }
}

// 发生哈希碰撞，线性向后查询
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```

```java
private final int threadLocalHashCode = nextHashCode();
// 初始为0
private static AtomicInteger nextHashCode = new AtomicInteger();
// 增长步长
private static final int HASH_INCREMENT = 0x61c88647;
// 获取下一个hash值
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```



## Java内存模型

线程之间的共享变量存储在`主内存`（Main Memory）中，每个线程都有一个私有的`本地内存`（Local Memory），本地内存中存储了共享变量的副本，用来进行线程内部的读写操作。

- 当一个线程更改了本地内存中共享变量的副本后，它需要将这些更改刷新到主内存中，以确保其他线程可以看到这些更改
- 当一个线程需要读取共享变量时，它可能首先从本地内存中读取。如果本地内存中的副本是过时的，线程将从主内存中重新加载共享变量的最新值到本地内存中



![深入浅出 Java 多线程：Java内存模型](https://gitee.com/qingy735/blogimg/raw/master/img/jmm-f02219aa-e762-4df0-ac08-6f4cceb535c2.jpg)



### 原子性、可见性、有序性

>  JMM通过内存屏障来实现内存的可见性以及禁止重排序

指令重排序规则：

- happens-before

- as-if-serial

### synchronized

>  synchronized可以修饰`普通方法`（相当于对当前对象加锁）、`静态方法`（相当于对当前类加锁）、`代码块`（显式的指定对谁加锁）

#### synchronized特性

- 互斥

- 刷新内存（和volatile类似）

- 可重入

- 非公平锁

#### synchronized原理

1、synchronized修饰代码块时，JVM采用`monitorenter`、`monitorexit`两个指令来实现同步，`monitorenter`指令指向同步代码块的开始位置，`monitorexit`指令指向同步代码块的结束位置。

2、synchronized 修饰同步方法时，JVM 采用`ACC_SYNCHRONIZED`标记符来实现同步，这个标识指明了该方法是一个同步方法。

monitorenter、monitorexit 或者 ACC_SYNCHRONIZED 都是**基于 Monitor 实现**的。在 Java 虚拟机（HotSpot）中，Monitor 是由**ObjectMonitor 实现**的，可以叫做内部锁，或者 Monitor 锁。

ObjectMonitor：

- 两个队列\_WaitSet、\_EntryList，分别用来保存wait状态和block状态线程
- \_owner，获取到Monitor对象的线程进入\_owner区时，\_count+1，线程调用wait方法则会释放Monitor对象，_owner 恢复为空， _count - 1。同时该等待线程进入 _WaitSet 中，等待被唤醒。同时每进入一次\_count都会加一，从而实现了synchronized的可重入特性。

#### 锁级别

| 锁       | 优点                                                         | 缺点                                             | 使用场景                             |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------ |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 | 适用于只有一个线程访问同步块场景。   |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度。                   | 如果始终得不到锁竞争的线程使用自旋会消耗 CPU。   | 追求响应时间。同步块执行速度非常快。 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗 CPU。                           | 线程阻塞，响应时间缓慢。                         | 追求吞吐量。同步块执行时间较长。     |

#### 偏向锁

线程第一次进入同步块时，会在对象头和栈帧中的锁记录里存储锁偏向的线程 ID。当下次该线程进入这个同步块时，会去检查锁的 Mark Word 里面是不是放的自己的线程 ID。

如果是，表明该线程已经获得锁了，以后该线程在进入和退出同步代码块时不需要花费CAS操作来加锁和解锁

如果不是，就代表有另一个线程来竞争这个偏向锁。这个时候会尝试使用 CAS 来替换 Mark Word 里面的线程 ID 为新线程的 ID。

- 成功：表示之前的线程不存在了， Mark Word 里面的线程 ID 为新线程的 ID，锁不会升级，仍然为偏向锁；
- 失败：表示之前的线程仍然存在，那么暂停之前的线程，设置偏向锁标识为 0，并设置锁标志位为 00，升级为轻量级锁，会按照轻量级锁的方式进行竞争锁。



#### 轻量级锁

多个线程在不同时间段获取同一把锁，不存在锁竞争的情况，也就没有线程阻塞。针对这种情况，JVM 采用轻量级锁来避免线程的阻塞与唤醒。

#### 重量级锁

重量级锁依赖于操作系统的互斥锁（mutex，用于保证任何给定时间内，只有一个线程可以执行某一段特定的代码段） 实现，而操作系统中线程间状态的转换需要相对较长的时间，所以重量级锁效率很低，但被阻塞的线程不会消耗 CPU。



#### 锁升级过程

- 检查MarkWord中存放的是不是自己的ThreadId，如果是则表示当前线程处于偏向锁状态
- 如果不是自己的ThreadId，锁升级，此时通过CAS来执行切换，新的线程根据MarkWord里现有的ThreadId，通知之前线程暂停，之前线程将MarkWord中的内容置空
- 两个线程都把锁对象的 HashCode 复制到自己新建的用于存储锁的记录空间，接着开始通过 CAS 操作，把锁对象的 MarkWord 的内容修改为自己新建的记录空间的地址的方式竞争 MarkWord
- 成功执行 CAS 的获得资源，失败的则进入自旋
- 自旋的线程在自旋过程中，成功获得资源(即之前获的资源的线程执行完成并释放了共享资源)，则整个状态依然处于轻量级锁的状态，如果自旋失败
- 进入重量级锁的状态，这个时候，自旋的线程进行阻塞，等待之前线程执行完成并唤醒自己

> 注：一个对象在调用原生`hashCode`方法后（`来自Object的，未被重写过的`），**该对象将无法进入偏向锁状态，起步就会是轻量级锁**。若`hashCode`方法的调用是在对象已经处于偏向锁]状态时调用，**它的偏向状态会被立即撤销，并且锁会升级为重量级锁。**



#### synchronized锁对象

> synchronized锁住的是对象，如果锁住的这个对象在多线程中相同，那么这些线程访问synchronized修饰的代码块时总是互斥的。但是如果锁住的这个对象在多线程中是不同的，那么这些线程访问synchronized修饰的代码块时不会互斥的。



- 对象锁

  如果时同一个实例，就会按照顺序访问，如果时不同实例，就可以同时访问

  - 锁非静态变量
  - 锁this对象
  - 锁非静态方法

- 类锁

  所有实例按照顺序访问

  - 锁静态变量
  - 锁类的class
  - 锁静态方法



### volatile

> volatile可以保证可见性但是不保证原子性

- 当写一个`volatile`变量时，JMM会把该线程在本地内存中的变量强制刷新到主内存中
- 这个写操作会导致其他线程中的`volatile`变量缓存无效

`volatile`会禁止指令重排，当使用`volatile`修饰变量时，JMM会插入内存屏障来确保以下两点：

- 写屏障（Write Barrier）：当一个 volatile 变量被写入时，写屏障确保在该屏障之前的所有变量的写入操作都提交到主内存
- 读屏障（Read Barrier）：当读取一个 volatile 变量时，读屏障确保在该屏障之后的所有读操作都从主内存中读取

### ReentrantLock

> `ReentrantLock`是可重入的独占锁，只能由一个线程可以获取该锁，其他获取该锁的线程会被阻塞。



ReentrantLock加锁和解锁过程：

```java
// 创建非公平锁（默认创建的是非公平锁，传入参数true创建公平锁）
ReentrantLock lock = new ReentrantLock();
// 获取锁操作
lock.lock();
try {
    // 执行代码逻辑
} catch (Exception ex) {
    // ...
} finally {
    // 解锁操作
    lock.unlock();
}
```

#### 公平锁和非公平锁

- 公平锁意味着在多个线程竞争锁时，获取锁的顺序与线程请求的顺序相同，即先来先服务（FIFO）。虽然能保证锁的顺序，但是需要额外维护一个有序队列
- 非公平锁不保证线程获取锁的顺序，当锁被释放时，任何请求锁的线程都有机会获取锁，而不是按照请求的顺序。



#### synchronized和ReentrantLock

synchronized是一个关键字，而Lock属于一个接口

![三分恶面渣逆袭：synchronized和ReentrantLock的区别](https://gitee.com/qingy735/blogimg/raw/master/img/javathread-38.png)



- 使用方式不同

  synchronized可以直接在方法上加锁，也可以在代码块上加锁（不需要手动释放锁，锁会自动释放），而ReentrantLock必须手动声明来加锁和释放锁

- 功能特点不同

  如果需要更细粒度的控制（如可中断的锁操作、尝试非阻塞获取锁、超时获取锁或者使用公平锁等），可以使用 Lock



### AQS



> AQS，全称是 AbstractQueuedSynchronizer，中文意思是抽象队列同步器。AQS 的思想是，如果被请求的共享资源空闲，则当前线程能够成功获取资源；否则，它将进入一个等待队列，当有其他线程释放资源时，系统会挑选等待队列中的一个线程，赋予其资源。



![三分恶面渣逆袭：AQS抽象队列同步器](https://gitee.com/qingy735/blogimg/raw/master/img/javathread-39.png)



- 同步状态state由volatile修饰，保证多线程之间的可见性

  ```java
  private volatile int state;
  ```

- 同步队列时通过内部定义的Node类来实现的，每个Node包含等待状态、前后节点、线程的引用等

  ```java
  static final class Node {
      // 表示该结点（对应的线程）已被取消
      static final int CANCELLED =  1;
      // 表示后继结点（对应的线程）需要被唤醒
      static final int SIGNAL    = -1;
      // 表示该结点（对应的线程）在等待某一条件
      static final int CONDITION = -2;
      // 表示有资源可用，新head结点需要继续唤醒后继结点
      // （共享模式下，多线程并发释放资源，而head唤醒其后继结点后，需要把多出来的资源留给后面的结点；
      // 设置新的head结点时，会继续唤醒其后继结点）
      static final int PROPAGATE = -3;
  
      volatile Node prev;
  
      volatile Node next;
  
      volatile Thread thread;
  }
  ```

- 两种同步方式

  - 独占模式（Exclusive）：资源是独占的，一次只能有一个线程获取，如ReentrantLock。
  - 共享模式（Share）：同时可以被多个线程获取，具体的资源个数可以通过参数指定，如Semaphore、CountDownLatch



![preview](https://gitee.com/qingy735/blogimg/raw/master/img/view)



#### 加锁

`ReentrantLock.lock()`

```java
public void lock() {
    sync.lock();
}
```


```java
abstract static class Sync extends AbstractQueuedSynchronizer
```

sync是一个静态内部类，继承AQS，有两个实现NofairSync(非公平锁)，FailSync(公平锁)



`NonfairSync.lock`

```java
final void lock() {
    if (compareAndSetState(0, 1)) // 通过cas操作来修改state状态，表示争抢锁的操作
      setExclusiveOwnerThread(Thread.currentThread());// 设置当前获得锁状态的线程
    else
      acquire(1); //尝试去获取锁
}
```

`acquire`

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```



由于是非公平锁，所以调用lock方法时，先通过cas去抢占锁，如果抢占锁成功则保存获得锁成功的当前线程，否则调用acquire来走锁竞争逻辑。通过tryAcquire尝试获取独占锁，如果成功返回true，失败返回false，如果tryAcquire失败则会通过addWaiter方法将当前线程封装成Node添加到AQS队列尾部；acquireQueued将Node作为参数，通过自旋去尝试获取锁

![acquire流程](https://gitee.com/qingy735/blogimg/raw/master/img/aqs-a0689bb2-9b18-419d-9617-6d292fbd439d.jpg)

#### 释放锁

`ReentrantLock.unlock`

首先释放锁然后唤醒park的线程

```java
public void unlock() {
    sync.release(1);
}
```



```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```



### CAS

CAS（Compare-and-Swap）是一种乐观锁的实现方式，全称为“比较并交换”，是一种无锁的原子操作。

在CAS中有三个值：V（要更新的变量）、E（预期值）、N（新值），比较并交换的过程如下：判断V是否等于E，如果等于，将V的值设置为N；如果不等，说明已经有其他线程更新了V，于是当前线程放弃更新，什么也不做。

当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。



#### CAS三大问题

- ABA问题

  一个值原来是A，变成了B，又变回了A，这个时候CAS检查不出来。（栈顶元素判断）

  追加版本号或时间戳

- 长时间自旋

  CAS长时间自旋不成功会占用大量CPU资源

- 多个共享变量的原子操作

  当对一个共享变量执行操作时，CAS 能够保证该变量的原子性。但是对于多个共享变量，CAS 就无法保证操作的原子性，这时通常有两种做法：

  1. 使用`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行 CAS 操作；
  2. 使用锁。锁内的临界区代码可以保证只有当前线程能操作。



### 线程死锁

- 互斥条件
- 请求并持有
- 不可剥夺条件
- 循环等待条件

### 线程同步

- 互斥量
- 读写锁
- 条件变量
- 自旋锁
- 信号量

## 线程池

> 线程池其实是一种池化的技术实现，池化技术的核心思想就是实现资源的复用，避免资源的重复创建和销毁带来的性能开销。线程池可以管理一堆线程，让线程执行完任务之后不进行销毁，而是继续去处理其它线程已经提交的任务。

### 线程池参数

- **corePoolSize**：核心线程数

  线程池中核心线程数量，即使这些线程处于空闲状态也不会被回收。

- **maximumPoolSize**：最大线程数

  线程池允许的最大线程数量。当工作队列满了之后，线程池会创建新线程来处理任务，直到线程数达到这个最大值。

- **keepAliveTime**：非核心线程存活时间

  非核心线程的空闲存活时间。如果线程池中的线程数量超过了 corePoolSize，那么这些多余的线程在空闲时间超过 keepAliveTime 时会被终止。

- **unit**：非核心线程存活时间单位

- **workQueue**：等待队列

  用于存放待处理任务的阻塞队列。当所有核心线程都忙时，新任务会被放在这个队列里等待执行。

  - ArrayBlockingQueue：有界的先进先出的阻塞队列，底层是数组，适合固定大小的线程池
  - LinkedBlockingQueue：底层数据结构时链表，不指定大小默认是Integer.MAX_VALUE，相当于一个无界队列
  - PriorityBlockingQueue：支持优先队列排序的无界阻塞队列。任务按照其自然顺序或通过构造器给定的Comparator来排序
  - DelayQueue：由二叉堆实现的无界优先级阻塞队列
  - SynchronousQueue：实际上它不是一个真正的队列，因为没有容量。每个插入操作必须等待另一个线程的移除操作，同样任何一个移除操作都必须等待另一个线程的插入操作

- **threadFactory**：创建线程使用工厂

- **handler**：饱和拒绝策略

  定义了当线程池和工作队列都满了之后对新提交的任务的处理策略。

  - AbortPolicy：默认拒绝策略。抛出异常
  - CallerRunsPolicy：不抛出异常，会让提交任务的线程自己来执行任务
  - DiscardOldestPolicy：丢弃队列中最老的一个任务（在队列中等待最久的任务）
  - DiscardPolicy：直接丢弃该任务

### 常见线程池

- newFixedThreadPool (固定数目线程的线程池)

  核心线程数和最大线程数相同，阻塞队列为无界队列LinkedBlockingQueue，容易造成OOM

- newCachedThreadPool (可缓存线程的线程池)

  核心线程数为0，最大线程数为Integer.MAX_VALUE，可能无限创建线程导致OOM，阻塞队列为SynchronousQueue，非核心线程空闲存活时间为60秒

- newSingleThreadExecutor (单线程的线程池)

  核心线程数和最大线程数都为1，阻塞队列为无界队列LinkedBlockingQueue，可能导致OOM

- newScheduledThreadPool (定时及周期执行的线程池)

  最大线程数为Integer.MAX_VALUE，有OOM风险，阻塞队列为DelayedWorkQueue

### 动态调节参数

- 利用配置中心配置线程池参数，监听修改后通过对应的**set方法**重新配置线程池
- 自己实现线程池，监听参数变化，根据实际业务需求更改对应参数



## 并发容器

### ConcurrentHashMap

在JDK7时采用分段锁机制（Segment Locking），整个Map被分为若干段，每个段都可以独立地加锁，因此不同线程可以同时操作不同的段从而实现并发访问。



![初念初恋：JDK 7 ConcurrentHashMap](https://gitee.com/qingy735/blogimg/raw/master/img/map-20230816155810.png)

在 JDK 8 及以上版本中，ConcurrentHashMap 的实现进行了优化，不再使用分段锁，而是使用了一种更加精细化的锁——桶锁，以及 CAS 无锁算法。每个桶（Node 数组的每个元素）都可以独立地加锁，从而实现更高级别的并发访问。

![初念初恋：JDK 8 ConcurrentHashMap](https://gitee.com/qingy735/blogimg/raw/master/img/map-20230816155924.png)

同时，对于读操作，通常不需要加锁，可以直接读取，因为 ConcurrentHashMap 内部使用了 volatile 变量来保证内存可见性。

对于写操作，ConcurrentHashMap 使用 CAS 操作来实现无锁的更新，这是一种乐观锁的实现，因为它假设没有冲突发生，在实际更新数据时才检查是否有其他线程在尝试修改数据，如果有，采用悲观的锁策略，如 synchronized 代码块来保证数据的一致性。

### Hashtable

Hashtable 在任何时刻只允许一个线程访问整个 Map，通过对整个 Map 加锁来实现线程安全。而 ConcurrentHashMap（尤其是在 JDK 8 及之后版本）通过锁分离和 CAS 操作实现更细粒度的锁定策略，允许更高的并发。



### CopyOnWriteArrayList

CopyOnWriteArrayList 是一个线程安全的 ArrayList，它遵循写时复制（Copy-On-Write）的原则，即在写操作时，会先复制一个新的数组，然后在新的数组上进行写操作，写完之后再将原数组引用指向新数组。
