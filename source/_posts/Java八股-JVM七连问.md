---
title: JVM七连问
tags:
  - JVM
categories: 八股
abbrlink: 64965
date: 2024-05-17 17:18:39
---



## JVM七连问

> 关于Object o = new Object()



### 1、请解释一下对象的创建过程（半初始化）

> 申请空间 - 初始化 - 引用

JVM首先检查当前类是否被加载解析和初始化过，如果没有就先执行对应的类加载器；如果加载了就为新对象分配内存空间，并且将分配的内存空间初始化为零值（成员变量，数值类型是 0，布尔类型是 false，对象类型是 null），接下来设置对象头，最后JVM执行构造方法（<init>），将成员变量赋值为预期的值。



### 2、DCL（double check lock）要不要加volatile问题（指令重排）

**synchronized** 内部可以重排序，锁的外部可以访问到中间状态

必须要加 **volatile**(保证线程之间的可见性和禁止指令重排序（内存屏障）)

```java
public class SingleObject {
    private static volatile SingleObject INSTANCE;

    private SingleObject() {
    }

    public static SingleObject getInstance() {
        if (INSTANCE == null) {
            synchronized (SingleObject.class) {
                if (INSTANCE == null) {
                    try {
                        Thread.sleep(1);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    INSTANCE = new SingleObject();
                }
            }
        }
        return INSTANCE;
    }
}
```



### 3、对象在内存中的存储布局（对象与数据的存储不同）

- Markword（64位机器8字节 32位机器4字节）
- 类型指针（4字节）
- 实例数据（String类型4字节）
- 对齐（被8整除）

### 4、对象头具体包括什么
**markword** 和 **class pointer**

#### markword
- 锁信息
- GC信息
- hashcode

### 5、对象怎么定位
**直接** 和 **间接**
- 直接（默认）
  直接指针，仅包含实例数据，实例数据中包含类型数据指针指向方法区中的T.class

  速度快

- 间接
  句柄方式，包含实例数据指针和类型数据指针

  对象在内存中移动位置，地址不用变



### 6、对象怎么分配

> 栈上 - 线程本地 - Eden - Old

- 尝试分配到栈上（逃逸分析、标量替换）
- 足够大直接分配到老年代
- TLAB（给每个线程在伊甸区的一个独立空间，线程同步）
- 分配到伊甸区



### 7、Object o = new Object()在内存中占多少字节

**16字节**



































