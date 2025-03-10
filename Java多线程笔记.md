# Java并发理论基础

## 为什么需要多线程？

CPU、内存、I/O设备之间速度存在极大差异，为了合理利用CPU的高性能，**平衡这三者的速度差异**，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为：

- CPU增加了缓存，以均衡和内存之间的速度差异；-> 导致`可见性`问题
- 操作系统增加了进程、线程，以分时复用CPU，进而均衡CPU与I/O设备之间的速度差异; -> 导致`原子性`问题
- 编译程序优化指令执行次序，使得缓存能够得到更加合理的利用；-> 导致`有序性`问题

## 线程不安全是指什么？

如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作结果是不一致的。

## 并发出现线程不安全的本质是什么？

### 可见性：CPU缓存引起

**可见性**：一个线程对共享变量进行修改，另外一个线程能够立刻看到。 在并发时，可能存在可见性问题，即**线程1对变量i修改了以后，线程2没有立即看到线程1修改的值**。

### 原子性：分时复用引起

**原子性**：即一个操作或多个操作要么全部执行且执行的过程不会被任何因素打断，要么就都不执行。

### 有序性：重排序引起

**有序性**：即程序执行的顺序按照代码的先后顺序执行。

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序，重排序分为三种类型：

- **编译器优化的重排序**：编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序
- **指令级并行的重排序**：现代处理器采用了指令级并行技术来将多条指令重叠执行。
- **内存系统的重排序**：由于处理器使用缓存和I/O缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM 的处理器重排序规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）。

## Java是怎么解决并发问题的：JMM（Java Memory Model）

### 核心知识点

JMM本质上可以理解为，Java内存模型规范了JMM如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括：

- volatile、synchronized和final三个关键字
- Happens-Before规则

### 可见性、有序性、原子性

- 可见性

  Java提供了`volatile`关键字来保证可见性。当一个共享变量被`volatile`关键字修饰时，它会**保证修改的值立即被更新到主存**，当有其他线程需要读取时，它会去内存中读取新值。而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

  另外，通过`synchronized`和`Lock`也能保证可见性，`synchronized`和`Lock`能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存中，因此可以保证可见性。

- 有序性

  在Java里面，可以通过`volatile`关键字来保证一定的“有序性”。另外可以通过`synchronized`和`Lock`来保证有序性，很显然，`synchronized`和`Lock`保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。当然JMM是通过Happens-Before规则来保证有序性的。

- 原子性

  在Java中，对基本数据类型的变量的读取和赋值操作时原子性操作，即这些操作时不可被中断的，要么执行，要么不执行。也就是说，只有简单的读取、赋值（且必须是将数字赋值给变量）才是原子操作。

  如果要实现更大范围操作的原子性，可以通过`synchronized`和`Lock`来实现。由于`synchronized`和`Lock`能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。

### Happens-Before(先行发生)规则

- **单一线程规则**(Single Thread Rule)：在一个线程内，在程序前面的操作先行发生于后面的操作。
- **管程锁定规则**(Monitor Lock Rule)：一个unlock操作先行发生于后面对同一个锁的lock操作。
- **volatile变量规则**(Volatile Variable Rule)：对一个volatile变量的写操作先行发生于后面对这个变量的读操作。
- **线程启动规则**(Thread Start Rule)：Thread对象的start()方法调用先行发生于此线程的每一个动作。
- **线程加入规则**(Thread Join Rule)：Thread对象的结束先行发生于join()方法返回。
- **线程中断规则**(Thread Interruption Rule)：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过interrupted()方法检测到是否有中断发生。
- **对象终结规则**(Finalizer Rule)：一个对象的初始化完成（构造函数执行结束）先行发生于他的finalize()方法的开始。
- **传递性**(Transitivity)：如果操作A先行发生于操作B，操作B先行发生于操作C，那么操作A先行发生于操作C。

## 线程安全是不是非真即假？

**一个类在可以被多个线程安全调用时就是线程安全的。**

线程安全不是一个非真即假的命题，可以将共享数据按照安全程度的强弱顺序分为以下五类：不可变、绝对线程安全、相对线程安全、线程兼容和线程对立。

1. **不可变**

不可变(Immutable)的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型包括：

- final关键字修饰的基本数据类型
- String
- 枚举类型
- Number部分子类，如`Long`和`Double`等数值包装类型。但同为`Number`的原子类`AtomicInteger`和`AtomicLong`则是可变的。

对于集合类型，可以使用`Collections.unmodifiableXXX()`方法来获取一个不可变的集合。

2. **绝对线程安全**

不管运行时环境如何，调用者都不需要任何额外的同步措施。

3. **相对线程安全**

相对线程安全需要保证对这个对象单独的操作是线程安全的，在调用的时候不需要做额外的保障措施。但是对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。

在 Java 语言中，大部分的线程安全类都属于这种类型，例如 Vector、HashTable、Collections 的 synchronizedCollection() 方法包装的集合等。

4. **线程兼容**

线程兼容是指对象本身并不是线程安全的，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用，我们平常说一个类不是线程安全的，绝大多数时候指的是这一种情况。Java API 中大部分的类都是属于线程兼容的，如与前面的 Vector 和 HashTable 相对应的集合类 ArrayList 和 HashMap 等。

5. **线程对立**

线程对立是指无论调用端是否采取了同步措施，都无法在多线程中并发使用的代码。由于Java语言天生就具备多线程特性，线程对立很少出现，而且通常是有害的，应当尽量避免。

## 线程安全又哪些实现思路？

1. **互斥同步(阻塞同步)**

`synchronized`和`ReentrantLock`

2. **非阻塞同步**

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种**悲观的并发策略**，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁(这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁)、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

一些常见的乐观操作：`CAS`, `AtomicInteger`, `ABA`

3. **无同步方案**

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

**（一）栈封闭**

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

**（二）线程本地存储(Thread Local Storage)**

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式(如“生产者-消费者”模式)都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”(Thread-per-Request)的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

**（三）可重入代码(Reentrant Code)**

这种代码也叫做纯代码(Pure Code)，可以在代码执行的任何时刻中断它，转而去执行另外一段代码(包括递归调用它本身)，而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

## 如何理解并发和并行的区别？

# Java多线程的三种实现方式

- 集成Thread类并实现run()
- 实现Runnable()接口并实现run()
- 实现Callable<?>接口并实现call()

```java
public class MyThread extends Thread{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + "：Hello World!");
        }
    }
}
```

```java
public class MyRunnable implements Runnable{

    //只有run方法代码区里是新的线程，此处仍为main线程
    @Override
    public void run(){
        Thread t = Thread.currentThread();

        for (int i = 0; i < 100; i++) {
            System.out.println(t.getName() + "：Hello World!");
        }
    }
}
```

```java
public class MyCallable implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        //求1-100之间的和
        int sum = 0;
        for (int i = 1; i <= 100; i++) {
            sum += i;
        }
        return sum;
    }

}
```



# 多线程中常用的成员方法

| 方法名称                         | 说明                                       |
| -------------------------------- | ------------------------------------------ |
| String getName()                 | 获取当前线程名称                           |
| void setName(String name)        | 设置当前线程名称（构造方法也可以设置名字） |
| static Thread currentThread()    | 获取当前线程的对象                         |
| static void sleep(long time)     | 让线程休眠指定的时间（单位为毫秒）         |
| setPriority(int newPriority)     | 设置线程的优先级                           |
| final int getPriority()          | 获取线程的优先级                           |
| final void setDaemon(boolean on) | 设置为守护线程                             |
| public static void yield()       | 出让线程                                   |
| public static void join()        | 插入线程                                   |



## 线程的优先级

- Java的多线程为抢占式调度
- 优先级越大，线程抢到CPU时间片的概率越大
- 线程优先级为1(最小)到10(最大)，默认为5

```java
public class MyRunnable implements Runnable{
    @Override
    public void run(){
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + "------" + i);
        }
    }
}
```

```java
public class ThreadDemo1 {

    public static void main(String[] args) {
        MyRunnable mr = new MyRunnable();

        Thread t1 = new Thread(mr, "飞机");
        Thread t2 = new Thread(mr, "坦克");

        t1.setPriority(1);
        t2.setPriority(10);

        t1.start();
        t2.start();
    }
}
```



## 守护线程

当其他非守护线程结束时，守护线程也会陆陆续续结束。



## 出让线程

```java
Thread.yield(); //出让当前线程CPU的执行权
```



## 插入线程

```
Thread.join();	//将某线程插入到当前线程前执行
```



# 线程的生命周期

- 新建：创建线程对象
- 就绪：start()后进入就绪状态，线程不停尝试抢占CPU；当其他线程抢占CPU的执行权后，此线程再次进入就绪态
- 运行：线程抢占到CPU的执行权后进入运行态
- 死亡：run()中的全部内容执行完毕后，进入死亡态
- 阻塞：线程调用sleep()等阻塞方法后进入阻塞态，没有执行资格也没有执行权，sleep()等方法结束后，重新进入就绪态

