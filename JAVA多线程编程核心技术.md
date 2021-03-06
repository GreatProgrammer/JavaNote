# JAVA多线程编程核心技术



## 第一章

终止线程的办法：

​		过期方法：stop()方法，目前已经被废弃，原因是它会直接杀死线程，让后续的一些结束工作无法进行。

​		推荐方法：interrupt()，只是将线程的状态属性设置为true，并不会终止线程。

interrupt()、interruped()、isInterruped()区别：

​		interrupt()：成员方法，将线程的中断状态设置为true。

​		interruped()：静态方法，返回当前线程的中断状态，并且重置中断状态。设置为false。

​		isInterruped()：成员方法，单纯的返回线程的中断状态。

注意点：

​		调用线程的interrupt()方法，如果线程是处于sleep状态，就会被抛出interruptException，并且重置中断状态为false。

停止线程方法：

​		外部线程调用interrupt()方法，内部一直用while(!interruped())判断。

暂停线程的办法：

​		过期方法：suspend()，目前已经已经被废弃。因为停止了之后还需要resume()去唤醒。一个线程的暂停之后什么时候再执行应该由cpu决定。

​		推荐方法：yield()，这个方法线程会暂停，但是什么时候唤醒是由底层cpu决定。

线程的优先级：通过setPriority()方法设置线程的优先级，十个优先级1~10，数字越大优先级越高。优先级较高的线程可以优先获得cpu等资源，但不代表一定比其他线程要快。

JDK内置三个优先级变量

```java
 /**
  * The minimum priority that a thread can have.
  */
 public final static int MIN_PRIORITY = 1;

/**
  * The default priority that is assigned to a thread.
  */
 public final static int NORM_PRIORITY = 5;

 /**
  * The maximum priority that a thread can have.
  */
 public final static int MAX_PRIORITY = 10;
```

守护线程：

1.所谓守护线程就是运行在程序后台的线程，程序的主线程Main（比方java程序一开始启动时创建的那个线程）不会是守护线程.

2.Daemon thread在Java里面的定义是，如果虚拟机中只有Daemon thread 在运行，则虚拟机退出。 

  虚拟机中可能会同时有很多个线程在运行，只有当所有的非守护线程都结束的时候，虚拟机的进程才会结束，不管在运行的线程是不是main()线程。

3.Main主线程结束了（Non-daemon thread）,如果此时正在运行的其他threads是daemon threads,JVM会使得这个threads停止,JVM也停下.

  如果此时正在运行的其他threads有Non-daemon threads,那么必须等所有的Non daemon线程结束了，JVM才会停下来.

4.总之,必须等所有的Non-daemon线程都运行结束了，只剩下daemon的时候，JVM才会停下来，注意Main主程序是Non-daemon线程.

```java
public class Thread3 {
    public static void main(String[] args) {
        Thread thread=new DaemonThread();
        thread.setDaemon(true);
        System.out.println(Thread.currentThread().getName()+" is daemon thread:"+Thread.currentThread().isDaemon());
        System.out.println(thread.getName()+" is daemon thread:"+thread.isDaemon());
        thread.start();
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class DaemonThread extends Thread{
    @Override
    public void run() {
        int i=0;
        while(true){
            try {
                Thread.sleep(1000);
                i++;
                System.out.println("i="+i);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

此时DaemonThread是守护线程，Main是用户线程。当Main线程睡眠五秒的时候，守护线程继续执行。当Main睡完执行结束之后，当前没有其他用户线程，守护线程就会退出。



## 第二章

synchronized锁重入：

​	内部方法可以获得外部方法的锁：

```java
    public synchronized void A(){
        System.out.println("Hello! This is A");
        B();
    }
    public synchronized void B(){
        System.out.println("Hello! This is B");
        C();
    }
    public synchronized void C(){
        System.out.println("Hello! This is C");
    }
```



虽然B方法需要得到锁，但是B方法在A方法内调用，所以可以获得锁

静态方法块使用synchronized的时候锁的是当前Class对象

当对volatile修饰的变量进行写操作，JVM会向处理器发送一条Lock前缀的指令，将CPU所在的缓存行数据写回到内存中，为了保证其他处理器的缓存和当前缓存一致，每个处理器通过嗅探在总线上传播的数据来检查自己缓存值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行置为无效状态，当处理器对这个数据进行读写操作时会重新从系统内存中读取到缓存行中。volatile只能保证每次使用变量的时候都是最新值。

 ## 第三章

wait()方法：让线程进入等待状态，并且释放锁。该方法必须要在synchronized里面执行，否则会抛出获取不到锁的异常。

notify()方法：随机唤醒一个wait状态的线程，但是不会释放锁。它要等当前线程执行完synchronized才会释放锁给唤醒的那个线程。同样该方法要在synchronized里面执行，否则会抛异常。

**释放和唤醒是由锁对象来调用，不是由线程对象调用，否则会报java.lang.IllegalMonitorStateException**

 wait(long timeout)：设置timeout毫秒之后如果没有被唤醒的话就自己唤醒。

#### 进程间通信

字节流：PipedInputStream和PiipedOutputStream

字符流：PipedReader 和PipedWriter

用法：输出流或者输入流调用connect方法即可。

join()：在main里面调用某个线程的join方法，表示等待此线程执行完后main线程再继续执行



 

