- [线程安全问题](#线程安全问题)
  - [原子性](#原子性)
  - [可见性](#可见性)
  - [活跃性](#活跃性)
    - [死锁](#死锁)
    - [活锁](#活锁)
    - [饥饿](#饥饿)
- [性能问题](#性能问题)
- [JMM （Java Memory Model）](#jmm-java-memory-model)
  - [Java 内存模型](#java-内存模型)
- [volatile 关键字](#volatile-关键字)
  - [禁止指令重排](#禁止指令重排)
- [synchronized 关键字](#synchronized-关键字)
  - [synchronized同步方法](#synchronized同步方法)
  - [synchronized同步静态方法](#synchronized同步静态方法)
  - [synchronized同步代码块](#synchronized同步代码块)
  - [synchronized属于可重入锁](#synchronized属于可重入锁)

# 线程安全问题

## 原子性

```java
int i = 0; // 操作1
i++;   // 操作2 这不是原子操作。这实际上是一个 "read-modify-write" 操作，它包括了读取 i 的值，增加 i，然后写回 i。
int j = i; // 操作3
i = i + 1; // 操作4
```

## 可见性

每个线程都有属于自己的工作内存

为了解决多线程的可见性问题，Java 提供了volatile这个关键字。当一个共享变量被 volatile 修饰时，它会保证修改的值立即更新到主存当中，这样的话，当有其他线程需要读取时，就会从内存中读到新值。普通的共享变量不能保证可见性，因为变量被修改后什么时候刷回到主存是不确定的，因此另外一个线程读到的可能就是旧值。

当然 Java 的锁机制如 synchronized 和 lock 也是可以保证可见性的。

## 活跃性

### 死锁

死锁是指多个线程因为环形等待锁的关系而永远地阻塞下去。

线程1持有锁A，线程2持有锁B，线程1等待线程2释放锁B，而线程2也等待线程1释放锁A。

### 活锁

死锁是两个线程都在等待对方释放锁导致阻塞。而活锁的意思是线程没有阻塞，还活着呢。当多个线程都在运行并且都在修改各自的状态，而其他线程又依赖这个状态，就导致任何一个线程都无法继续执行，只能重复着自身的动作，于是就发生了活锁。

### 饥饿

如果一个线程无其他异常却迟迟不能继续运行，那基本上是处于饥饿状态了。

常见的有几种场景:

- 高优先级的线程一直在运行消耗 CPU，所有的低优先级线程一直处于等待；
- 一些线程被永久堵塞在一个等待进入同步块的状态，而其他线程总是能在它之前持续地对该同步块进行访问；

# 性能问题

前面讲到了线程安全和死锁、活锁这些问题，如果这些都没有发生，多线程并发一定比单线程串行执行快吗？答案是不一定，因为多线程有创建线程和线程上下文切换的开销。

CPU 是很宝贵的资源，速度非常快，为了保证雨露均沾，通常会给不同的线程分配时间片，当 CPU 从执行一个线程切换到执行另一个线程时，CPU 需要保存当前线程的本地数据，程序指针等状态，并加载下一个要执行线程的本地数据，程序指针等，也就是『上下文切换』。

一般减少上下文切换的方法有：

- 无锁并发编程：可以参照 ConcurrentHashMap 锁分段的思想，不同的线程处理不同段的数据，这样在多线程竞争的条件下，可以减少上下文切换的时间。
- CAS 算法，利用 Atomic + CAS 算法来更新数据，采用乐观锁的方式，可以有效减少一部分不必要的锁竞争带来的上下文切换。
- 使用最少线程：避免创建不必要的线程，如果任务很少，但创建了很多的线程，这样就会造成大量的线程都处于等待状态。
- 协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。

# JMM （Java Memory Model）

并发编程的线程之间存在两个问题：

- 线程间如何通信？即：线程之间以何种机制来交换信息
- 线程间如何同步？即：线程以何种机制来控制不同线程间发生的相对顺序


有两种并发模型可以解决这两个问题：

- 消息传递并发模型
- 共享内存并发模型

## Java 内存模型

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406182036425.png)

线程 A 无法直接访问线程 B 的工作内存，线程间通信必须经过主存。

线程 B 并不是直接去主存中读取共享变量的值，而是先在本地内存 B 中找到这个共享变量，发现这个共享变量已经被更新了，然后本地内存 B 去主存中读取这个共享变量的新值，并拷贝到本地内存 B 中，最后线程 B 再读取本地内存 B 中的新值。

本地内存相当于 CPU的L1&L2 Cache

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406182038032.png)

Java 中的 volatile 关键字可以保证多线程操作共享变量的可见性以及禁止指令重排序，synchronized 关键字不仅保证可见性，同时也保证了原子性（互斥性）。

# volatile 关键字

- 当写一个 volatile 变量时，JMM 会把该线程在本地内存中的变量强制刷新到主内存中去；
- 这个写操作会导致其他线程中的 volatile 变量缓存无效。

## 禁止指令重排

重排序不会对存在数据依赖关系的操作进行重排序。比如：a=1;b=a; 这个指令序列，因为第二个操作依赖于第一个操作，所以在编译时和处理器运行时这两个操作不会被重排序。

重排序是为了优化性能，但是不管怎么重排序，单线程下程序的执行结果不能被改变。比如：a=1;b=2;c=a+b 这三个操作，第一步 (a=1) 和第二步 (b=2) 由于不存在数据依赖关系，所以可能会发生重排序，但是 c=a+b 这个操作是不会被重排序的，因为需要保证最终的结果一定是 c=a+b=3。

# synchronized 关键字

synchronized 关键字最主要有以下 3 种应用方式：

- 同步方法，为当前对象（this）加锁，进入同步代码前要获得当前对象的锁；
- 同步静态方法，为当前类加锁（锁的是 Class 对象），进入同步代码前要获得当前类的锁；
- 同步代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

## synchronized同步方法

```Java
public class AccountingSync implements Runnable {
    //共享资源(临界资源)
    static int i = 0;
    // synchronized 同步方法
    public synchronized void increase() {
        i ++;
    }
    @Override
    public void run() {
        for(int j=0;j<1000000;j++){
            increase();
        }
    }
    public static void main(String args[]) throws InterruptedException {
        AccountingSync instance = new AccountingSync();
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println("static, i output:" + i);
    }
}
```


## synchronized同步静态方法

```Java
public class AccountingSyncClass implements Runnable {
    static int i = 0;
    /**
     * 同步静态方法,锁是当前class对象，也就是
     * AccountingSyncClass类对应的class对象
     */
    public static synchronized void increase() {
        i++;
    }
    // 非静态,访问时锁不一样不会发生互斥
    public synchronized void increase4Obj() {
        i++;
    }
    @Override
    public void run() {
        for(int j=0;j<1000000;j++){
            increase();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        //new新实例
        Thread t1=new Thread(new AccountingSyncClass());
        //new新实例
        Thread t2=new Thread(new AccountingSyncClass());
        //启动线程
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
/**
 * 输出结果:
 * 2000000
 */
```

## synchronized同步代码块

```Java
public class AccountingSync2 implements Runnable {
    static AccountingSync2 instance = new AccountingSync2(); // 饿汉单例模式

    static int i=0;

    @Override
    public void run() {
        //省略其他耗时操作....
        //使用同步代码块对变量i进行同步操作,锁对象为instance
        synchronized(instance){
            for(int j=0;j<1000000;j++){
                i++;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(instance);
        Thread t2=new Thread(instance);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```

## synchronized属于可重入锁

synchronized 就是可重入锁，因此一个线程调用 synchronized 方法的同时，在其方法体内部调用该对象另一个 synchronized 方法是允许的

```Java
public class AccountingSync implements Runnable{
    static AccountingSync instance=new AccountingSync();
    static int i=0;
    static int j=0;

    @Override
    public void run() {
        for(int j=0;j<1000000;j++){
            //this,当前实例对象锁
            synchronized(this){
                i++;
                increase();//synchronized的可重入性
            }
        }
    }

    public synchronized void increase(){
        j++;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(instance);
        Thread t2=new Thread(instance);
        t1.start();t2.start();
        t1.join();t2.join();
        System.out.println(i);
    }
}
```