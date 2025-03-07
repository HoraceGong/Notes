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

