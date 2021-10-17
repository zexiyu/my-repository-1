#  线程基础知识



## 程序，进程，线程，纤程（协程）

- 程序：就是最开始的时候安静的躺在磁盘中的.exe可执行文件，当我们点击运行的时候，系统将其load到内存中，之后cpu将其执行

- 进程：**操作系统资源分配的基本单位**。（一个程序可以有多个进程，比如一台电脑可以登录多个qq那就是多个进程）

- 线程：**调度执行的基本单位**。线程在进程的内部，线程是一个程序里不同的执行路径，多个线程共享同一进程中的资源

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210827215359498.png" alt="image-20210827215359498" style="zoom:60%;" />

  

## 纤程（协程）

纤程和线程本质上原理相同，线程的切换时通过栈实现，纤程亦如此，都是栈的记录和切换。纤程是用在用户空间之内的线程。

线程是由操作系统内核管理（内核态），协程是由程序进行控制（用户态）

区别:

1. **线程的切换通过内核空间，纤程的切换通过用户空间**。因此线程通过**内核栈**切换，协程通过**用户栈**切换。协程没有线程切换开销大
2. 纤程能启动的数量远多于线程数量

**使用纤程的好处就是简化线程的切换**



什么时候建议用协程：当有大量的短时间操作的线程不断切换时，我们采用协程替换线程效果最佳，一是降低了系统内存，二是减少了系统切换开销，提升系统性能

### 纤程在Java中的使用

JVM不支持纤程，Java要想使用纤程需要通过外部库 `Quaser` (纤程库)





## 大白话解释程序进程线程

- 比如用Intelligent Idea程序时，启动的整个idea就是一个进程，idea的整个进程中有语法检查线程，代码智能提示线程。

- JDK的一个进程中又有编译线程，垃圾自动回收线程
- 使用QQ，查看进程一定有一个 QQexe的进程，我可以用qq和A文字聊天，和B视频聊天，给C传文件，给D发一段语言，QQ支持录入信息的搜索。
- 大四的时候写论文，用word写论文，同时用QQ音乐放音乐，同时用QQ聊天，这是多个进程。
- word如没有保存，停电关机，再通电后打开word可以恢复之前未保存的文档，word也会检查你的拼写，两个线程：容灾备份，语法检查

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210827223028463.png" alt="image-20210827223028463" style="zoom:60%;" />





## 工作线程是不是越多越好，多少合适

工作线程并不是越多越好，因为线程的切换时消耗系统资源的

具体设多少线程合适还得看机器配置，看处理器核数，结合公式，并进行强力的压力测试才能得到。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210827225239604.png" alt="image-20210827225239604" style="zoom:30%;" />



## 并发concurrent和并行parallel

并发是指任务的提交，并行是指任务的执行。

并行是并发的子集。

从人的角度来看，就是一堆任务一起提交上去，机器可以同时执行。但是从计算机的角度来看，并发是指好多好多的任务同时提交上去。最终还是由多个cpu并行的去执行任务。每个cpu中又会有时间片快速的切换执行多个不同任务



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210906120000669.png" alt="image-20210906120000669" style="zoom:39%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210906120028963.png" alt="image-20210906120028963" style="zoom:38%;" />



# 锁的介绍

锁的本质不是对操作系统资源的锁定，更不是对线程的锁定，**而是哪个线程获得锁，持有锁**

我们知道对象头中包含8位（64bit）,其中有2bit表示锁的状态的



## synchornized



### synchornized的简单介绍

synchronized即保证原子性也保证可见性

程序中如果出异常锁会被释放

synchronized修饰的 代码块或方法执行结束会自动释放锁

synchronized static void m( )   ==> 类锁，相当于 T.class

synchronized  void m( )   ==>对象锁，相当于this



**synchornized锁对象必须要加final**

如`synchornized(o) { }`  就必须要` final Object o = new Object( );` 因为如果一个对象没有加final，当另一个线程改变了o这把锁，锁会失效。（改变了对象后对象头中锁的标志位会丢失）



**String类型不能用做锁**

是因为字符串常量池的特性：相同的字符串是同一个对象。如果用String做锁，就很可能不同类中的同步代码会持有相同的锁



### synchornized锁的可重入

有两个synchronized方法，加的是同一把锁，在一个线程中，这两个方法可以互相调用

```java
synchronized void m1(){
 m2();
}

synchronized void m2(){

}
```

m1调用m2，执行到m2时发现用的是同一把锁，而且锁的同一个对象，就不用再加锁



### synchornized锁的细化

有时候不需要将锁加在方法上，因为一个方法上可能有大量的执行逻辑，锁加在方法上粒度太大。有时并不需要对整个方法进行同步，对于加锁来说，代码能锁的少就尽量锁少一些。因此可以对锁进行更加细粒度的优化，在方法里面加个同步代码块即可







### synchornized锁升级

早期synchronized是重量级的 ，无论怎么设置synchronized都是重量级系统锁。效率低下（JDK1.5以前），

之后出了synchronized的改进，采用了锁升级机制

- 对象没有线程操作：无锁状态
- 当第一个线程访问资源时是偏向锁，在对象头上标有线程ID，记录对象为线程独有，下次第一个线程访问该对象会偏向于第一个线程（其实不是真正的加锁，只是设置了个标志位！） ----- 偏向锁
- 后面的线程来访问，发现线程ID和第一个线程不等，即进行锁升级为自旋锁。 -----自旋锁
- 当一个线程自旋10次以上，升级为重量级锁

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210830102304946.png" alt="image-20210830102304946" style="zoom:30%;" />



注：锁升级以后无法降级

**采用这种锁升级的原因是：**

- 大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。
- 自旋锁不会访问操作系统，它是用户态，不会访问内核态。可以提高响应速度。因此引入自旋锁

**什么时候适合使用自旋锁，什么时候适合使用重量级锁**

线程执行时间特别长，建议使用重量级锁。

线程执行时间短，而且线程数量不多，建议使用自旋锁。-----因为自旋锁基于用户态切换速度快，线程数量少自旋锁就少不浪费CPU资源



## Volatile

- 保证指令的有序性
- 保证线程间的可见性
- 不保证原子性



见JVM笔记



## ReentraintLock

synchornized和Lock都是可重入锁

使用tryLock进行尝试锁定，需要指定时间，不管锁定与否，过了尝试时间后方法都将继续执行
可以根据tryLock的返回值来判定是否锁定。
也可以指定tryLock的时间，由于tryLock(time)会抛出异常，所以要注意unclock的处理，必须放到finally中解锁

**synchronized与Lock的区别**  

1. Lock是基于CAS和AQS实现的，Synchornized有锁升级的过程。
2. Lock可以通过tryLock判断自己是否获取到锁
3. Lock可以在锁竞争的同时被interrupt打断
4. synchornized为非公平锁，Lock可实现公平和非公平
5. synchronized系统自动释放锁（程序执行完或出异常都会自动释放锁），Lock需手动加锁，在finally中使用unlock( )手动解锁



**公平锁和非公平锁**

公平锁底层有个等待队列，当一个线程进来以后会检查里面有没有等待队列，有等待队列就是公平锁，乖乖的去排队。 如果没有等待队列就是非公平锁，有的线程可能上来就直接抢到锁，有的线程可能半天也抢不到锁

```java
ReentrantLock lock = new ReentrantLock(true); //参数为true表示为公平锁，请对比输出结果
```





## CountDownLatch

CountDownLatch有一个数字参数。被countDown()计数的线程会先执行，直到这个计数参数变为0后，调用await()的线程才被执行。 

注意：

- countDown()和await() 必须配合使用。
- 且countDown数量必须要等于线程数量，否则后面的线程无法执行



```java
public class CloseDoorDemo {
    public static void main(String[] args) {
        Thread[] threads = new Thread[100];
        CountDownLatch countDownLatch = new CountDownLatch(threads.length);
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                System.out.println("the student number of " + Thread.currentThread().getName() + " left the classroom");
                countDownLatch.countDown();
            }, String.valueOf(i + 1));
        }
        // 保证了同学先关门班长后走人。保证了main方法后执行
        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("the classmate open the door last");
    }
}
```

```java
public class TestCountDownLatch {
    public static void main(String[] args) {
        usingCountDownLatch();
        usingJoin();
    }

    private static void usingCountDownLatch() {
        Thread[] threads = new Thread[10];
        CountDownLatch latch = new CountDownLatch(threads.length);
        Lock lock = new ReentrantLock(true);

        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                try {
                    lock.lock();
                    int result = 0;
                    System.out.println(Thread.currentThread().getName() + "\t" + result);
                    latch.countDown();
                } finally {
                    lock.unlock();
                }
            });
        }
        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }
        try {
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("end latch");
    }

    private static void usingJoin() {
        Thread[] threads = new Thread[10];

        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                int result = 0;
                System.out.println(Thread.currentThread().getName() + "\t" + result);
            }, "thread" + i);
        }

        for (int i = 0; i < threads.length; i++) {
            threads[i].start();
        }

        // 在main方法中加入每个线程的join，为了使得这些线程先于main方法执行
        for (int i = 0; i < threads.length; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("end join");
    }
}
```



## CyclicBarrier

原理：CyclicBarrier的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。线程进入屏障通过CyclicBarrier的await()方法。



```java
public class TestCyclicBarrier {
    public static void main(String[] args) {
        //CyclicBarrier barrier = new CyclicBarrier(20);

        CyclicBarrier barrier = new CyclicBarrier(20, () -> {
            System.out.println("满人，发车");
        });

        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```





## ReadWriteLock

- 读锁---共享锁

  如果一个读线程在读取数据，可以给其他读线程进行读取。但写线程不给

- 写锁---排他锁

  第一个写线程进来就会把资源锁定，其他读线程和写线程都不给

  

最终目的：读写分离

```java
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\tstart to write~~~");
            SleepHelper.sleep(1);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\tend write~~~");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }

    public void get(String key) {
        readWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\tstart read~~~");
            Object result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\tread end~~~" + result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
}
public class ReadWriteDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 1; i <= 5; i++) {
            final int tempInt = i;
            new Thread(() -> {
                myCache.put(String.valueOf(tempInt), Thread.currentThread().getName());
            }, String.valueOf(i)).start();
        }

        for (int i = 1; i <= 5; i++) {
            final int tempInt = i;
            new Thread(() -> {
                myCache.get(String.valueOf(tempInt));
            }, String.valueOf(i)).start();
        }
    }
}

```

**ReentraintLock底层是基于 volatile的state + AQS + CAS实现的**



## LockSupport

有两个方法：

- `park( Thread thread )`：阻塞线程
- `unpark( Thread thread )` ：解除阻塞指定线程

## Semaphore（信号灯）

**主要用于限流操作，用来限制多少个线程同时运行**

 * acquire（获取） 信号，每个线程进来信号数 + 1，直到加到自己设置的数量后。后面的线程不执行。等待这些线程执行完毕
 * release（释放）信号 。如果不释放后面批次的线程不会执行

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
      // 最多允许5个线程同时执行
        Semaphore semaphore = new Semaphore(5, true);
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + " is Running.....");
                    SleepHelper.sleep(2);
                    System.out.println(Thread.currentThread().getName() + " is end.....");
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            },"thread" + i).start();
        }
    }
}
```



## 死锁

**如何预防死锁？**

首先需要将死锁发生的是个必要条件讲出来: 

1. 互斥条件 同一时间只能有一个线程获取资源
2.  不可剥夺条件 一个线程已经占有的资源，在释放之前不会被其它线程抢占
3. 请求和保持条件 线程等待过程中不会释放已占有的资源
4. 循环等待条件 多个线程互相等待对方释放资源



 死锁预防，那么就是需要破坏这四个必要条件

- 资源互斥： 由于资源互斥是资源使用的固有特性，无法改变，我们不讨论

- 破坏不可剥夺条件

  一个进程不能获得所需要的全部资源时便处于等待状态，等待期间他占有的资源将被隐式的释放重新加入到系统的资源列表中，可以被其他的进程使用，而等待的进程只有重新获得自己原有的资源以及新申请的资源才可以重新启动、执行
  
- 破坏请求与保持条件

   第一种：静态分配，即每个进程在开始执行时就申请他所需要的全部资源

   第二种：动态分配，即每个进程在申请所需要的资源时他本身不占用系统资源

-  破坏循环等待条件

   采用资源有序分配其基本思想是将系统中的所有资源顺序编号，将紧缺的，稀少的采用较大的编号，在申请资源时必须按照编号的顺序进行，一个进程只有获得较小编号的进程才能申请较大编号的进程。

## 自旋锁

后面和CAS一起介绍



# 线程的七种状态

## 控制线程状态的三种方法

### sleep

sleep后线程处于TIMED WAITING状态 （不会释放锁）

### yield

 本质上是让出cpu给其他线程执行，线程从running -> ready就绪状态。至于后续什么再抢到并不关心 （不会释放锁）

### join

t1线程中出现t2.join( )后，t1线程变为Waiting状态，线程t2执行，待线程t2执行完毕后，之前的线程t1继续执行 （会释放锁，因为底层是wait，而wait是释放锁的）



### wait/notify

前面三种是Thread类中的方法。而wait/notify是Object类中的方法，它可以控制持有这个对象锁的线程进行等待（waiting）和通知其他线程（notify/notifyAll）。

**注：** wait( )会释放当前对象锁，但notify( )仅仅是通知唤醒其他线程，不释放锁



面试题：如何保证线程的执行顺序？

方法1：在main方法中分别调用 t1.join( )，t2.join( )，t3.join( )

方法2：t2中调用t1.join( )，t3中调用t2.join( )



**面试题：notify和notifyAll的区别**

锁池：假设线程A已经拥有对象锁，线程B、C想要获取锁就会被阻塞，进入一个地方去等待锁的等待，这个地方就是该对象的锁池；锁池中的线程会参与锁竞争

等待池：假设线程A调用某个对象的wait方法，线程A就会释放该对象锁，同时线程A进入该对象的等待池中，等待池中的线程不会参与锁的竞争。

 

notify和notifyAll的区别：

1、notify只会随机选取一个处于等待池中的线程进入锁池去竞争获取锁的机会；

2、notifyAll会让所有处于等待池的线程全部进入锁池去竞争获取锁的机会；



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210827224327205.png" alt="image-20210827224327205" style="zoom:50%;" />

- 线程就绪状态Ready：就是将线程扔到CPU的等待队列中
- 线程terminated结束后，是不能再start的，线程terminated也是线程死亡。正常结束run()/call()、出现异常、stop()都会使线程死亡
- block状态：线程在等待获取synchronized锁的过程是block的，其他都是blocked的。synchronized是由操作系统调度，只有被操作系统调度才是blocked
- timedwaiting：被指定等待时间，如sleep



**sleep和wait的区别**

1. sleep属于Thread类；wait属于Object类
2. sleep不释放锁，过了指定等待时间又恢复，wait释放锁，必须通过notify唤醒
3. sleep在同步状态和非同步状态下都可以调用，wait只能在同步代码中调用



# 创建线程的方式

1. 继承Thread类

2. 实现Runnable接口 

   变种： 使用lambda表达式 -> ，常用

3. 实现callable带返回值。配合FutureTask使用

   ```java
   public static void main(String[] args) throws Exception {
   		FutureTask<String> futureTask = new FutureTask<>(() -> {
               System.out.println(Thread.currentThread().getName() + "----------Implements Callable,use lambda...........");
               return "callable";
           });
       new Thread(futureTask, "lambdaCallable").start();
       System.out.println("获取到的Callable接口中call方法的返回值为：" + futureTask.get());
   }
   ```

   

4. 线程池

   ```java
   public static void main(String[] args) throws Exception {
     //=======================普通线程池的方式=======================
     		 ExecutorService executorService = Executors.newFixedThreadPool(10);
           for (int i = 0; i < 10; i++) {
               executorService.execute(() -> {
                   System.out.println(Thread.currentThread().getName() + "----------use ThreadPool...........");
               });
           }
     //==================线程池结合FutureTask，Callable的使用=================
           FutureTask<String> futureTask1 = new FutureTask<>(() -> {
               System.out.println(Thread.currentThread().getName() + "----------Use Callable In ThreadPool...........");
               return "callableInThreadPool";
           });
           executorService.execute(futureTask1);
           System.out.println(futureTask1.get());
           executorService.shutdown();
   }
   ```

   

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpem9uZ3hpYW8=,size_16,color_FFFFFF,t_70.png" alt="在这里插入图片描述" style="zoom:80%;" />

 

**面试题:callable接口与runnable接口的区别？**

1. 是否有返回值
2. 是否抛异常
3. 落地方法不一样，一个是run，一个是call



归根结底，本质上还是一种方式，就是new创建 Thread( )对象，然后调用start( )。





# 停止线程的方法

1. stop( ) ：

   已废弃，不建议使用。因为会产生数据不一致的问题，线程中途stop不会回滚到最初状态

2. suspend( ) 暂停线程 配合 resume( ) 恢复线程：

   不建议使用，暂停线程会给线程加一把锁，如果忘记恢复线程，该线程将永远锁住

3. 使用volatile关键字控制标志位

   通过` public static volatile boolean running = true` 控制。如果不依赖明确的中间执行状态是可以的。

   ```java
   public class ThreadInterrupt {
       public static volatile boolean RUNNING = true;
       public static void main(String[] args) {
           Thread t1 = new Thread(() -> {
               System.out.println("thread is start");
               while (RUNNING) {
               }
               System.out.println("thread is end");
           }, "thread1");
           t1.start();
           SleepHelper.sleep(3);
           RUNNING = false;
       }
   }
   ```

   

4. 使用interrupt设置标志位

   ```java
   public class ThreadInterrupt {
       public static void main(String[] args) {
           Thread t1 = new Thread(() -> {
               System.out.println("thread is start");
               while (!Thread.currentThread().isInterrupted()) { // 查询线程是否被打断
               }
               System.out.println("thread is end");
           }, "thread1");
           t1.start();
           SleepHelper.sleep(3);
           t1.interrupt();
       }
   }
   ```

   

   和volatile的区别：volatile是我们手工设置的标志位，interrupt是系统自带的标志位，interrupt更优雅





# CAS



CAS是乐观锁的一种实现方式，是一种轻量级锁，JUC 中很多工具类的实现就是基于 CAS 的。

底层用到CAS的类有很多，CAS就是无锁优化。



## CAS原理

线程在读取数据时不进行加锁，在准备写回数据时，先去查询原值，操作的时候比较原值是否修改，若未被其他线程修改则写回，若已被修改，则重新执行读取流程。此即为**无锁优化**

### 通过数据库操作认识CAS

以数据库操作为例：修改一个数据库中的数据name，先查寻到name = “帅群”，拿到值然后再去修改，修改之前判断是不是“帅群”？如果不是“帅群”的话说明已经被别的 线程/事务 修改过了，此次修改失败，如果是“帅群”的话就进行修改，改成“泽希”。

**注：** 上述只是通过mysql操作类比CAS，实际在CAS中 比较+更新 是一个原子操作，有CPU原语的支持

![img](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/b5e36029c0470944330d6b6e17a787af.png)



### Java中CAS的原理

CAS机制中使用了3个基本操作数：内存地址V，旧的预期值A，要修改的新值B。

更新一个变量的时候，只有当变量的预期值A和内存地址V当中的实际值相同时，才会将内存地址V对应的值修改为B。

修改一个值的时候，发现预期值和修改的内存地址中的值相同，就改成新值；如果预期值和修改的内存地址中的值不同，说明被别的线程修改了，此线程对此值的修改失败。

比如：将一个数从1改为2，改之前看看是不是1，1就是期望值

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210903151204430.png" alt="image-20210903151204430" style="zoom:30%;" />







## CAS举例：AtomicInteger

AtomicInger是常用的支持并发修改值的原子类，内部是基于CAS实现，有了它，对值的并发修改不需要加synchornized锁，加快了响应速度

AtomicInger中的incrementAndGet()方法调用图

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902155102391.png" alt="image-20210902155102391" style="zoom:39%;" />

**AtomicInteger代码**

```java
// AtomicInteger类-------------------
public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
// Unsafe类-------------------------
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
           // 如果要修改的值和期望值不等就循环判断（自旋）
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
					
        return var5;
 }

// 第一个参数为需要改变的对象，第二个参数为偏移量
public native int getIntVolatile(Object var1, long var2);

// 第一个参数为需要改变的对象，第二个参数为偏移量， 第三个参数为期待的值，第四个参数为更新后的值
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```



**注：判断和修改是原子操作，由cpu源语支持**



## 自旋锁

自旋锁并不是物理上的锁，而是逻辑上的锁，是基于CAS特性实现的锁，通过CAS的特性进入死循环，最终肯定要比较并交换成功才能退出，死循环这个过程中是逻辑上持有锁的。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210906193010073.png" alt="image-20210906193010073" style="zoom:50%;" />

自旋操作：先拿值 + 比较拿的值并判断 (原子)

**自旋操作就是通过 CAS的机制 + 死循环判断来实现的 。** 

这就是自旋锁的原理，通过它即可实现线程安全



**什么时候适合使用自旋锁，什么时候适合使用重量级锁**

线程执行时间特别长，建议使用重量级锁。

线程执行时间短，但线程数量不能太多，建议使用自旋锁。因为自旋操作占用cpu资源



自己手写个锁

```java
public class MyLockDemo {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();
    public static volatile int count = 0;
    public void myLock(){
        Thread thread = Thread.currentThread();
        while (!atomicReference.compareAndSet(null,thread)){
        }
    }

    public void myUnlock(){
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread,null);
    }

    public static void main(String[] args) {
        MyLockDemo spinLockDemo = new MyLockDemo();
        for (int i = 0; i < 100; i++) {
            new Thread(()->{
                for (int j = 0; j < 10000; j++) {
                    spinLockDemo.add();
                }
            }).start();
        }
        SleepHelper.sleep(3);
        System.out.println(count);
    }

    public void add(){
        myLock();
        count++;
        myUnlock();
    }
}
```



## 悲观锁和乐观锁

从思想上来说synchronized属于悲观锁，悲观的认为程序中的并发情况严重，一有操作一言不合就上锁，所以严防死守。CAS属于乐观锁，乐观地认为程序中的并发情况不那么严重，所以让线程不断去重试更新。

在java中除了上面提到的Atomic系列类，以及Lock系列类夺得底层实现，甚至在JAVA1.6以上版本，synchronized升级为重量级锁之前，也会采用CAS机制。



## CAS的缺点：

1.  CPU开销过大

   在并发量比较高的情况下，很多线程反复尝试检查并更新某一个变量，却一直不是期望值，长时间更新不成功，就会一直自旋，相当于死循环，会给CPU带来很大的压力

2. ABA问题

   这是CAS机制最大的问题所在。

### ABA问题

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/00831rSTly1gcegahfihuj30oa0ku0tg.jpg" alt="img" style="zoom:50%;" />

就是说来了一个线程把值改回了B，又来了一个线程把值又改回了A，对于这个时候判断的线程，就发现他的值还是A，所以他就不知道这个值到底有没有被人改过，其实很多场景如果只追求最后结果正确，这是没关系的。

但是实际过程中还是需要记录修改过程的，比如资金修改什么的，你每次修改的都应该有记录，方便回溯。


- 解决方法1，加stamp版本号。使用AtomicStampedReference类，每次操作不仅验证预期值，还会验证预期版本

  ```java
      private static AtomicInteger atomicInt = new AtomicInteger(100);
      private static AtomicStampedReference atomicStampedRef = new AtomicStampedReference(100, 0);
  
      public static void main(String[] args) throws InterruptedException {
          Thread intT1 = new Thread(new Runnable() {
              @Override
              public void run() {
                  atomicInt.compareAndSet(100, 101);
                  atomicInt.compareAndSet(101, 100);
              }
          });
  
          Thread intT2 = new Thread(() -> {
              SleepHelper.sleep(1);
              boolean c3 = atomicInt.compareAndSet(100, 101);
              System.out.println(c3); // true
          });
  
          intT1.start();
          intT2.start();
          intT1.join();
          intT2.join();
  
          Thread refT1 = new Thread(() -> {
              SleepHelper.sleep(1);
              atomicStampedRef.compareAndSet(100, 101, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
              System.out.println(atomicStampedRef.getReference() + "\t" + atomicStampedRef.getStamp());
              atomicStampedRef.compareAndSet(101, 100, atomicStampedRef.getStamp(), atomicStampedRef.getStamp() + 1);
              System.out.println(atomicStampedRef.getReference() + "\t" + atomicStampedRef.getStamp());
  
          });
  
          Thread refT2 = new Thread(() -> {
              int stamp = atomicStampedRef.getStamp();
           	  SleepHelper.sleep(2);
              boolean c3 = atomicStampedRef.compareAndSet(100, 101, stamp, stamp + 1);
              System.out.println(c3); // false
          });
  
          refT1.start();
          refT2.start();
      }
  }
  ```

- 解决方法2：加时间戳

  查询的时候把时间戳一起查出来，对的上才修改并且更新值的时候一起修改更新时间，这样也能保证，方法很多但是跟版本号都是异曲同工之妙
  
  稍微有点不靠谱，因为无法控制时间戳精度



## Unsafe

Java语言不像C，C++那样可以直接访问底层操作系统，但是JVM为我们提供了一个unsafe类里面调用了很多c++方法。

它可以

- **直接操作内存**

  如allocateMemory( )，put( )，freeMemory( ) .....

- 直接生成类的实例

  allocateInstance( )

- 直接操作类或实例变量

  getInt( )，getObject( ) 

- 进行CAS的操作

  compareAndSwapObject( )。正是unsafe的compareAndSwapInt方法保证了Compare和Swap操作之间的原子性操作





# AQS

核心：有一个共享的、volitale的state值 + 互相抢夺的线程（thread队列）

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902224216138.png" alt="image-20210902224216138" style="zoom:39%;" />

组成：

- thread队列：由Thread类型的node节点组成的双向链表，每个线程节点争用state，谁拿到state谁就获取到锁。

- state：对AQS不同的实现会代表着不同的含义

	例如 ： ReentrantLock通过state代表重入次数   0->1->2->3，释放锁  3->2->1->0。



## 以为ReentrantLock为例的AQS解读

```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
          selfInterrupt(); 
        }   	
    }
// tryAcquire:新来的线程尝试获取锁
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
// addWaiter:将线程加入队尾
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) { // 通过CAS的方式将线程加入到队列尾部
                pred.next = node;
                return node;
            }
        }
        enq(node);
 				// 将节点插入队列，必要时进行初始化
        return node;
    }
// acquireQueued队列中的线程尝试获取锁
final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) { 
              	 // 不断的获取node节点的前驱结点
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) { // 如果前驱结点是头结点，就通过tryAcquire尝试加锁
                    setHead(node); // 找到前驱结点是头节点后，将node设为头结点，
                    p.next = null; // 将之前头结点的引用去掉，并垃圾回收
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

**addWaiter:将线程加入队尾**

新来的一个线程执行任务先尝试获取锁，如果获取不到锁会加入AQS队列尾，加入队列的过程是基于CAS实现。目的：如果没用CAS就要锁住整个链表，效率低下。

注：如果加入队列尾的过程没有加锁或没有自旋操作的话可能会有多个阶段同时加入队尾，破坏链表数据结构

**acquireQueued队列中的线程尝试获取锁**

死循环，从队尾开始，每个节点都获取到它前面的节点，直到头结点最终获取到锁，才能结束循环。并将等待线程队列中的头结点进行垃圾回收。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902230700488.png" alt="image-20210902230700488" style="zoom:50%;" />



## VarHandle

可以和一个变量指向同一个引用

比如new了一个Object，`Object o = new Object()`，o指向了Object对象。可以创建VarHandle指向o指向的Object对象。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902231810643.png" alt="image-20210902231810643" style="zoom:35%;" />

可以做到引用本身做不到的操作，如CAS原子性操作

handle的实现C++,是JDK1.9以后出现的

举例：在CAS中addWaiter在jdk9以后是基于VarHandle实现



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902235328768.png" alt="image-20210902235328768" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902235552442.png" alt="image-20210902235552442" style="zoom:30%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210902235615525.png" alt="image-20210902235615525" style="zoom:50%;" />

jdk1.8：

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210903000146947.png" alt="image-20210903000146947" style="zoom:35%;" />

# ThreadLocal

就是线程本地变量,有了它每个线程都可以有自己独有的本地变量， 实际上就是个map。

## 基本使用

**本地变量不共享**

```java
public class ThreadLocalTest {
	static ThreadLocal<Person> tl = new ThreadLocal<>();

	public static void main(String[] args) {
		new Thread(()->{
			try {
				TimeUnit.SECONDS.sleep(2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

			System.out.println(tl.get());
		}).start();

		new Thread(()->{
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			tl.set(new Person());
		}).start();
	}

	static class Person {
		String name = "zhangsan";
	}
}
```

最终结果为`null`，因为设置在线程2的threadLocal变量和线程1中的threadLocal变量不是同一个

**本地变量共享**

```java
	static ThreadLocal tl = ThreadLocal.withInitial(() -> {
    return ...;
  });
```



## ThreadLocal原理



 ThreadLocal就是当前线程内部的一个map

这个map的KV 中K是当前Thread Local对象，V是设置的值

ThreadLocal的set和get如下

```java
 public void set(T value) {
        Thread t = Thread.currentThread(); // 获取当前线程
        ThreadLocalMap map = getMap(t); // 获取当前线程的Map
        if (map != null) // 如果有当前线程的map就往里添加
            map.set(this, value);
        else
            createMap(t, value);  // 如果没有当前线程的map就创建并将值加入threadLocal中
    }


	// 获取指定线程的map
	ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
   }

	 void createMap(Thread t, T firstValue) { // 传入当前线程和值，创建ThreadLocal对象
        t.threadLocals = new ThreadLocalMap(this, firstValue);
   }

  // ThreadLocal的构造器
	ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY]; // INITIAL_CAPACITY = 16; 初始化容量为16
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

// =========================================
public T get() {
    //获得当前线程
    Thread t = Thread.currentThread();
    //每个线程 都有一个自己的ThreadLocalMap，
    //ThreadLocalMap里就保存着所有的ThreadLocal变量
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        //ThreadLocalMap的key就是当前ThreadLocal对象实例，
        //多个ThreadLocal变量都是放在这个map中的
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            //从map里取出来的值就是我们需要的这个ThreadLocal变量
            T result = (T)e.value;
            return result;
        }
    }
    // 如果map没有初始化，那么在这里初始化一下
    return setInitialValue();
}
```





**ThreadLocal的用途**

声明式事务管理。比如获取每一个数据库连接，不可能每次操作都调用connection方法连接数据库，当这个线程第一次调用connection以后，获取的数据库连接信息就放到threadLocal中，后面该线程就不需要调用connection方法，以后该线程想要获取数据库连接直接从ThreadLocal中拿。

## 强弱软虚

引用就是变量指向对象

### 强引用

普通引用即为强引用，强引用在垃圾回收时会调用finalize方法



### 软引用(SoftReference)



一个对象被软引用指向时，只有内存不够才会被回收。内存够用的情况下，即使调用System.gc( ) 通过FGC 也不会被回收

软引用的作用：用作缓存



### 弱引用(WeakReference)

弱引用只要遭遇到GC就会回收。

弱引用的作用：一般用在容器



典型应用在ThreadLoacal中



**通过ThreadLocal中的弱引用发现其中的坑**

ThreadLocal和Thread和ThreadLocalMap的关系如图。每个线程都有一个指定的ThreadLocalMap，因此才会每个线程独有一份本地变量

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904230534820.png" alt="image-20210904230534820" style="zoom:50%;" />

ThreadLocal.ThreadLocalMap是一个比较特殊的Map，它的每个Entry中的key都是一个弱引用： 虽然我们new出来的ThreadLocal对象的引用指向了threadlocal对象，但是Entry中的key通过弱引用也指向了threadlocal对象



```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    //key就是一个弱引用
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```



#### ThreadLocal内存泄漏问题

**问题1：为啥要设置引用指向threadlocal对象？**

用大腿想一下并结合上图，如果没有引用指向threadLocal对象，那么new ThreadLocal( )就形同虚设。

若是强引用，即使tl＝null，但key的引用依然指向ThreadLocal对象，ThreadLocal永远不会被回收，所以会有内存泄漏。**使用弱引用解决了ThreadLocal永远不会回收的问题**。

**问题2：**

如果ThreadLocal被回收，key的值变成了null，导致整个value再也无法被访问到，因此依然存在内存泄漏

解决方法：ThreadLocal用完了，务必要手工remove掉！不然会导致内存泄漏。



### 虚引用(PhantomReference)

用来管理堆外内存。

外面有个队列，虚引用被回收的时候，这个虚引用会被装到队列里，如果检测到队列中有一个存在了，就证明这个已经被回收了

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904231454727.png" alt="image-20210904231454727" style="zoom:50%;" />

注：

- 虚引用是给写JVM的人用的
-   `PhantomReference（object o，QUEUE）`     使用的时候一定要有一个队列参数
- 用的值拿不到
- NIO中的directbytebuffer，直接内存，堆外内存
- Java中的堆外内存回收是使用Unsafe类，1.8的时候可以通过反射拿到Unsafe





# 并发容器

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210903234650387.png" alt="image-20210903234650387" style="zoom:50%;" />

HashMap在多线程环境下存在线程安全问题，一般怎么处理？

- 使用HashTable
- 使用Collections.synchornizedMap(map)
- 使用ConcurrentHashMap



## 并发修改异常



```java
public class TestConcurrentModification {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);

            },"AA").start();
        }
    }
}
```

运行后出现并发修改异常：ConcurrentModificationException 

因为ArrayList在迭代的时候如果同时对其进行修改就会抛出java.util.ConcurrentModificationException异常:并发修改异常

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210707170213253.png" alt="image-20210707170213253" style="zoom:50%;" />



## ThreadSafetyMap （线程安全的map）

### HashTable

Hashtable相比较于Hashmap中所有的操作都加锁

### synchornizedMap

通过 `Collections.synchronizedMap(Map<K,V> m)` 创建

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904094903733.png" alt="image-20210904094903733" style="zoom:30%;" />

 mutex相当于锁，和我们平时Object o = new Object())一样。之后的所有操作都是基于hashmap方法的基础上上synchornized(mutex)。

底层原理是通过代理



### ConcurrentHashMap

ConcurrentHashMap 底层结构是`数组 + 链表 + 红黑树`  。



**JDK1.7和JDK1.8中的ConcurrentHashmap对比**



- JDK1.7中的ConcurrentHashMap

  内部主要是一个Segment数组，而数组的每一项又是一个HashEntry数组，元素都存在HashEntry数组里。因为每次锁定的是Segment对象，也就是整个HashEntry数组，所以又叫分段锁。

- JDK1.8中的ConcurrentHashMap

  舍弃了分段锁的实现方式，元素都存在Node数组中，每次锁住的是一个Node对象，而不是某一段数组，所以支持的写的并发度更高。再者它引入了红黑树，在hash冲突严重时，读操作的效率更高。

![image-20211007002730165](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211007002730165.png)



**put操作**

ConcurrentHashMap在进行put操作的还是比较复杂的，大致可以分为以下步骤：

- 根据 key 计算出 hashcode 。

- 判断是否需要进行初始化。即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。

- 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容。

- 如果都不满足，则利用 synchronized 锁写入数据。

- 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树。
  
  
  
  **总的来说就是放入第一个数据的时候使用CAS自旋操作，后续的数据放入都是使用细粒度的synhronized锁**
  
  

同样是线程安全的Map，ConcurrentHashMap比HashTable效率要高。 因为写的时候采用**自旋锁 + 分段锁**，锁的粒度更小concurrenthashmap效率提高主要的地方是在读上，因为读方法没有加锁。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904105656108.png" alt="image-20210904105656108" style="zoom:30%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904104507675.png" alt="image-20210904104507675" style="zoom:39%;" />



**get操作**

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904161753204.png" alt="image-20210904161753204" style="zoom:50%;" />





**注：**TreeMap用的是红黑树，查找的时候效率高

没有ConcurrenTreeMap的原因是CAS在如果用在树形结构上会太复杂

因此出了个基于跳表结构实现的并发map：ConcurrentSkipListMap 跳表实现。



### CopyOnWrite

当写操作比较少读操作比较多的时候建议使用CopyOnWriteArrayList/CopyOnWriteArraySet。

读的时候不加锁，写的时候先copy出一个新的ArrayList，然后往后面添加，最后更换旧的引用。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904112228104.png" alt="image-20210904112228104" style="zoom:30%;" />



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210904112646052.png" alt="image-20210904112646052" style="zoom:39%;" />





**注：**Hashtable 和ConcurrentHashMap相比较于HashMap来说，前两者不允许key为null，也不允许值为null；HashMap允许键值为null。因为多线程情况下key为null到底是值为null还是没找到是模糊不清的



## Queue



Queue接口是继List和Set后新出的为了解决高并发问题的容器接口

### BlockingQueue

阻塞队列，核心就是：当阻塞队列为空的时候，从队列中取元素的操作就会被阻塞。当阻塞队列满的时候，往队列中放入元素的操作就会被阻塞。

常用的阻塞队列有

- BlockingQueue类
  - LinkedBlockingQueue ：基于链表结构的无界队列阻塞队列，容量必须要指定
  - ArrayBlockingQueue：   基于数组结构的有界阻塞队列，容量不必指定，最大为INTEGER.MAX_VALUE
  - PriorityBlockingQueue：一个**支持优先级排序**的**无界**阻塞队列，可以传入一个比较器，按制定规则对元素排序

- DelayQueue ：一个**支持优先级排序**的**无界**阻塞队列，只有等延时时间到了，才能取出来。可以精确的定时

  按延迟时间排序的队列。 往里面装任务的时候必须实现Delay这个接口，拿的时候，一般队列是先进先出，这个队列是按照等待时间来拿的，一般用来做定时任务，本质上是一个priorityqueue，是一个二叉树的模型，小顶堆

- SynchronusQueue 一个不存储元素的阻塞队列，线程池

  用于两个线程之间传递任务，容量永远为0，不能往里面装东西，用的时候要有线程等着才行，实际中线程池中用处最大的一个，很多线程池互相取任务的时候用的都是这个

- LinkedTransferQueue 一个由**链表**结构组成的**无界**阻塞队列

  前面队列的组合 和SynchronusQueue区别在于这个可以传递的内容可以是有长度的，NB的在于添加了一个transfer方法，含义是，当有一条线程给这个队列中添加的时候，如果是put方法，就是装完了这条线程就走了。transfer是装完了，在这等着，当有人来取走这个资源的时候，这条线程才走，一般用在做了一个操作后要求一定要有一个结果的情况下才能继续往下走，例如付款之后要等结果反馈给客户



**ArrBlockingQueue和LinkedBlockingQueue的区别**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210929225028446.png" alt="image-20210929225028446" style="zoom:39%;" />



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210929224908421.png" alt="image-20210929224908421" style="zoom:39%;" />

- 锁机制不同

  ​		ArrayBlockingQueue只有一把锁：生产者和消费者使用的是同一把锁；

  　　LinkedBlockingQueue中有两把锁：生产者的锁PutLock，消费者的锁takeLock　

- ArrBlockingQueue涉及到缓存行伪共享的问题，LinkedBlockingQueue基于链表，涉及到缓存行的对齐，不会将整个缓存行进行读取。在多线程读取时LinkedBlockingQueue效率更好



**Queue和List相比的区别**

Queue添加了很多对多线程情况下友好的API，如offer( ) peek( )、pool( ) 、put( ) 、take( ) 

- put/take 实现了阻塞，put一定要往里装，如果队列满了，这个线程会阻塞住，take是一定会往外取，如果没有了，就会阻塞住
- offer和add ：两者都是往队列尾部插入元素，不同的时候，当超出队列界限的时候，add（）方法是抛出异常让你处理，而offer（）方法是直接返回false
- peek是取元素，但是不会remove
- poll 是取并且remove掉
- 阻塞的原理用的都是park/unpark（await底层用的是park）
- 实际上用的是Condition的await和signal方法



至于BlockingQueue，对于它的put和take的阻塞方法，它是实现MQ (message queue) 的基础模型

![image-20210707193420928](https://img-blog.csdnimg.cn/img_convert/ff2ad79dbecf7938b7f3ef084a188a0b.png)


# 八锁理论

1 标准访问，先打印短信还是邮件

2 停4秒在短信方法内，先打印短信还是邮件   

3 普通的hello方法，是先打短信还是hello   

4 现在有两部手机，先打印短信还是邮件   

5 两个静态同步方法，1部手机，先打印短信还是邮件 

6 两个静态同步方法，2部手机，先打印短信还是邮件   

7 1个静态同步方法，1个普通同步方法，1部手机，先打印短信还是邮件  

8 1个静态同步方法，1个普通同步方法，2部手机，先打印短信还是邮件  

运行答案：

1、短信

2、短信

3、Hello

4、邮件

5、短信

6、短信

7、邮件

8、邮件



区分对象锁和类锁，synchornized加在非静态方法上的锁属于this（对象锁），synchornized加在静态方法上的锁属于Xxx.class（类锁）

```java
public class EightLocks {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        Phone phone2 = new Phone();

        new Thread(() -> {
            try {
                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "AA").start();

        Thread.sleep(100);

        new Thread(() -> {
            try {
                phone2.sendEmail();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "BB").start();
    }
}

class Phone {
    public static synchronized void sendSMS() throws Exception {
        SleepHelper.sleep(3);
        System.out.println("------sendSMS");
    }
    public synchronized void sendEmail() throws Exception {
        System.out.println("------sendEmail");
    }
    public void getHello() {
        System.out.println("------getHello");
    }
}
```

# 线程池

**线程池的优势：通过控制线程的数量来控制最大并发数，方便管理线程，线程复用，减少资源消耗**



线程池接口和类的继承关系

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210905225414159.png" alt="image-20210905225414159" style="zoom:50%;" />

- ThreadPoolExecutor :线程池的核心类
- ExcutorService：定义了一系列关于线程池生命周期的方法
- Executor：线程的执行接口，里面只定义了一个execute( )方法



```java
ExecutorService service = Executors.newCachedThreadPool();
Future<String> future = service.submit(c); //异步

System.out.println("hello");
System.out.println(future.get());//阻塞
```







## FutureTask



future：存储执行将来要产生的结果。

将程序放到FutureTask中，在线程启动的时候它会执行，并存储相应的结果

我们创建线程有一种基于的callable方式，配合FutureTask。

FutureTask实现了Runnable接口和Future接口







## CompletableFuture

用来异步处理多线程请求，管理多个Future的处理结果。**它的原理就是开启多个线程同时处理多个future**。



**场景案例：**假设你能够提供一个服务，这个服务查询各大电商网站同一类产品（比如天猫淘宝京东）的价格并汇总展示

有三个方法priceOfTM( ) 从天猫中查  、priceOfTB( )从淘宝中查、priceOfJD( )从京东查

传统的方法是让三个方法依次执行。现在使用CompletableFuture，可以异步进行请求的处理。CompletableFuture能管理多个future的处理结果。节省了程序运行的时间。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210905234852537.png" alt="image-20210905234852537" style="zoom:39%;" />

**CompletableFuture的使用方法：**

- 使用CompletableFuture异步执行多个方法

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210905234648030.png" alt="image-20210905234648030" style="zoom:50%;" />

-  使用CompletableFuture执行方法的过程中可以使用一系列的链式操作![image-20210905235152291](https://img-blog.csdnimg.cn/img_convert/78cb9d4f3fb28861b39620cf879a13d8.png)



## 线程池的创建

**方式1**

一池有N个固定的线程，有固定线程数的线程

```java
ExecutorService threadPool = Executors.newFixedThreadPool(5);
```

**方式2**

一个任务一个任务的执行，一池一线程


```java
ExecutorService service = Executors.newSingleThreadExecutor();
```

**方式3**

根据并发数量弹性调整线程池中线程数量

```java
 ExecutorService service = Executors.newCachedThreadPool();
```

**方法4**

使用ThreadPoolExecutor创建线程池，开发中一定要使用此方法。见下面。



注：  阿里开发手册上声明开发中**一定要使用ThreadPoolExecutor创建线程池**，不能使用Executors去创建，   而且不要显示创建线程new Thread( )

```
【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
说明：线程池的好处是减少在创建和销毁线程所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题 

【强制】线程池不允许使用 Executors去创建，而是通过 ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
说明： Executors返回的线程池对象的弊端如下：
1）FixedThreadPool SingleThreadPool：允许的请求等待队列长度为 Integer.MAX_ VALUE，能会堆积大量的请求，从而导致OOM
2）CachedThreadPool：允许的创建线程数量为 Inteaer.MAX WALUE，能会创建大量的线程、从而导致OOM
```





## ThreadPoolExecutor底层原理

使用newFixedThreadPool

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210707223034542.png" alt="image-20210707223034542" style="zoom:33%;" />

使用newSingleThreadExecutor

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210707223118016.png" alt="image-20210707223118016" style="zoom:33%;" />

使用newCachedThreadPool

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210707223104504.png" alt="image-20210707223104504" style="zoom:33%;" />

使用newScheduledThreadPool

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210929225746096.png" alt="image-20210929225746096" style="zoom:33%;" />

## 线程池7大参数

```java
  public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

七个参数

1. corePoolSize：线程池中核心线程数。
2. maximumPoolSize：最大线程数
3. keepAliveTime：非核心线程存活时间，超过该时间后将线程归还给操作系统，避免资源的浪费。简称灭活
4. TimeUnit：keepAliveTime时间单位
5. BlockingQueue：任务队列，被提交但尚未被执行的任务.
6. ThreadFactory：表示生成线程池中工作线程的线程工厂，用于创建线程，一般默认即可
7. handler：当队列塞满并且线程池也满了，就采用的拒绝策略。

**注：实际开发中maximumPoolSize，需要自己进行估算。通过机器配置+公式+大量压测判断**



**线程池的executor( ) 和submit( )的区别**

二者都是开启一个线程并异步执行。但是submit有返回值execute没有。submit的返回值是一个Future类，可以获取该FutureTask执行后的异常信息



## 线程池底层工作原理

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211006225031306.png" alt="image-20211006225031306" style="zoom:50%;" />



1. 在创建了线程池后，开始等待请求。

2. 当调用execute()方法添加一个请求任务时，线程池会做出如下判断：

   - 如果正在运行的线程数量小于corePoolSize，那么马上创建线程运行这个任务；

   - 如果正在运行的线程数量大于或等于corePoolSize，那么将这个任务放入队列；

   - 如果这个时候队列满了且正在运行的线程数量还小于maximumPoolSize，那么就要创建非核心线程立刻运行这个任务；

   - 如果队列满了且正在运行的线程数量大于或等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行。

3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做（闲置）超过一定的时间（大于keepAliveTime参数）时，线程会判断：
   - 如果当前运行的线程数大于corePoolSize，那么这个非核心线程就被停掉。
   -  所有线程池的所有任务完成后，它最终会收缩到corePoolSize的大小。

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2,
                5,
                2L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());
        try {
            for (int i = 0; i < 9; i++) {
                executor.execute(() ->{
                    System.out.println(Thread.currentThread().getName() + "办理业务中~~~");
                });
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            executor.shutdown();
        }
    }
}

```





## 线程池的拒绝策略

- AbortPolicy(默认)：直接抛出异常,RejectedExecutionException

- CallerRunsPolicy：不抛弃任务，也不抛异常，是将任务回退到调用者执行。如果任务被main方法调用则被拒绝的方法给main( )执行

- DiscardOldestPolicy：抛弃队列中等待最久的任务，并加入队列变成新的。

- DiscardPolicy：丢弃无法处理的任务，不予任何处理也不抛出异常。如果允许任务丢失，这是最好的一种策略。

  

  **注：以上内置拒绝策略均实现了RejectedExecutionHandle接口**
  
- 拒绝策略可以自定义

  开发中一般自定义线程池拒绝策略。场景：订单请求并发量过高时，等待队列和线程池满了，请求的订单被拒绝，这时可以将被拒绝的请求保存在kafka或rabbitmq或mysql中做好日志，等待并发量下来的时候再做处理



### 自定义线程池的拒绝策略

实现RejectedExecutionHandle方法

```java
public class MyRejectedHandler {
    public static void main(String[] args) {
        ExecutorService service = new ThreadPoolExecutor(4, 4,
                0, TimeUnit.SECONDS, new ArrayBlockingQueue<>(6),
                Executors.defaultThreadFactory(),
                new MyHandler());
    }

    static class MyHandler implements RejectedExecutionHandler {

        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            //log("r rejected")
            //save r kafka mysql redis
            //try 3 times
            if(executor.getQueue().size() < 10000) {
                //try put again();
            }
        }
    }
}
```

具体的策略是什么样，还是要看具体的业务进行处理。





## ForkJoinPool

分解汇总的任务为若干子任务，用多个线程执行若干个子任务。最终将结果合并汇总的过程。属于CPU密集型。注：做不到先执行子任务

属于 任务拆分+ 结果合并

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210906222943283.png" alt="image-20210906222943283" style="zoom:30%;" />



**特点：**

- ForkJoinPool的每个线程都有一个自己独有的工作队列，而ThreadPoolExecutor中所有的线程共用一个工作队列。
- 工作窃取：ForkJoinPool中的一个线程工作队列可以窃取其他线程的工作队列。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210906223315259.png" alt="image-20210906223315259" style="zoom:60%;" />

当一个线程执行较快的时候，为了防止线程空闲，最大化cpu的利用率，执行快的线程可以窃取执行速度较慢线程工作队列中的任务。从其他线程工作队列尾部窃取，并放到自己工作队列的尾部



**创建方法**

```java
ExecutorService service = Executors.newWorkStealingPool();
```

它的底层就是new 了一个ForkJoinPool，目的就是提供更方便的接口，将参数都定义好，仅此而已

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210906223705034.png" alt="image-20210906223705034" style="zoom:39%;" />



RecursiveTask:递归任务：继承后可以实现递归调用的任务，具有返回值

<img src="https://img-blog.csdnimg.cn/20200715151928617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpem9uZ3hpYW8=,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />





```java
public class ForkJoinPool {
	static int[] nums = new int[100_0000];
	static final int MAX_NUM = 50000;
	static Random r = new Random();

	static {
		for(int i=0; i<nums.length; i++) {
			nums[i] = r.nextInt(100);
		}
		long start = System.currentTimeMillis();
		System.out.println("---" + Arrays.stream(nums).sum()); //stream api
		long end = System.currentTimeMillis();
		System.out.println("time spend of stream method--" + (end - start));
	}
  

	static class AddTaskRet extends RecursiveTask<Long> {

		private static final long serialVersionUID = 1L;
		int start, end;

		AddTaskRet(int s, int e) {
			start = s;
			end = e;
		}

		@Override
		protected Long compute() {

			if(end-start <= MAX_NUM) {
				long sum = 0L;
				for(int i=start; i<end; i++) sum += nums[i];
				return sum;
			}

			int middle = start + (end-start)/2;

			AddTaskRet subTask1 = new AddTaskRet(start, middle);
			AddTaskRet subTask2 = new AddTaskRet(middle, end);
			subTask1.fork();
			subTask2.fork();
			return subTask1.join() + subTask2.join();
		}
	}

	public static void main(String[] args) throws IOException {

		ForkJoinPool fjp = new ForkJoinPool();
		AddTaskRet task = new AddTaskRet(0, nums.length);
		long start = System.currentTimeMillis();
		fjp.submit(task);
		long result = task.join();
		System.out.println(result);
		long end = System.currentTimeMillis();
		System.out.println("time spend of forkjoin method--" + (end - start));
	}
}

```



### 并行流 ParallelStream

并行流的底层就是用到了ForkJoinPool

```java
public class T13_ParallelStreamAPI {
	public static void main(String[] args) {
		List<Integer> nums = new ArrayList<>();
		Random r = new Random();
		for(int i=0; i<10000; i++) nums.add(1000000 + r.nextInt(1000000));
		
		long start = System.currentTimeMillis();
		nums.forEach(v->isPrime(v));
		long end = System.currentTimeMillis();
		System.out.println(end - start);
		
		//使用parallel stream api
		
		start = System.currentTimeMillis();
		nums.parallelStream().forEach(T13_ParallelStreamAPI::isPrime);
		end = System.currentTimeMillis();
		
		System.out.println(end - start);
	}
	
  // 求质数
	public static boolean isPrime(int num) {
		for(int i=2; i<=num/2; i++) {
			if(num % i == 0) return false;
		}
		return true;
	}
}
```



# ThreadPoolExecutor源码解析

作者：连神---连sir

### 1、常用变量的解释

```java
// 1. `ctl`，可以看做一个int类型的数字，高3位表示线程池状态，低29位表示worker数量
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
// 2. `COUNT_BITS`，`Integer.SIZE`为32，所以`COUNT_BITS`为29
private static final int COUNT_BITS = Integer.SIZE - 3;
// 3. `CAPACITY`，线程池允许的最大线程数。1左移29位，然后减1，即为 2^29 - 1
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// runState is stored in the high-order bits
// 4. 线程池有5种状态，按大小排序如下：RUNNING < SHUTDOWN < STOP < TIDYING < TERMINATED
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// Packing and unpacking ctl
// 5. `runStateOf()`，获取线程池状态，通过按位与操作，低29位将全部变成0
private static int runStateOf(int c)     { return c & ~CAPACITY; }
// 6. `workerCountOf()`，获取线程池worker数量，通过按位与操作，高3位将全部变成0
private static int workerCountOf(int c)  { return c & CAPACITY; }
// 7. `ctlOf()`，根据线程池状态和线程池worker数量，生成ctl值
private static int ctlOf(int rs, int wc) { return rs | wc; }

/*
 * Bit field accessors that don't require unpacking ctl.
 * These depend on the bit layout and on workerCount being never negative.
 */
// 8. `runStateLessThan()`，线程池状态小于xx
private static boolean runStateLessThan(int c, int s) {
    return c < s;
}
// 9. `runStateAtLeast()`，线程池状态大于等于xx
private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
}
```

### 2、构造方法

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    // 基本类型参数校验
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    // 空指针校验
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    // 根据传入参数`unit`和`keepAliveTime`，将存活时间转换为纳秒存到变量`keepAliveTime `中
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

### 3、提交执行task的过程

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
    // worker数量比核心线程数小，直接创建worker执行任务
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // worker数量超过核心线程数，任务直接进入队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 线程池状态不是RUNNING状态，说明执行过shutdown命令，需要对新加入的任务执行reject()操作。
        // 这儿为什么需要recheck，是因为任务入队列前后，线程池的状态可能会发生变化。
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 这儿为什么需要判断0值，主要是在线程池构造方法中，核心线程数允许为0
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 如果线程池不是运行状态，或者任务进入队列失败，则尝试创建worker执行任务。
    // 这儿有3点需要注意：
    // 1. 线程池不是运行状态时，addWorker内部会判断线程池状态
    // 2. addWorker第2个参数表示是否创建核心线程
    // 3. addWorker返回false，则说明任务执行失败，需要执行reject操作
    else if (!addWorker(command, false))
        reject(command);
}
```

### 4、addworker源码解析

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    // 外层自旋
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 这个条件写得比较难懂，我对其进行了调整，和下面的条件等价
        // (rs > SHUTDOWN) || 
        // (rs == SHUTDOWN && firstTask != null) || 
        // (rs == SHUTDOWN && workQueue.isEmpty())
        // 1. 线程池状态大于SHUTDOWN时，直接返回false
        // 2. 线程池状态等于SHUTDOWN，且firstTask不为null，直接返回false
        // 3. 线程池状态等于SHUTDOWN，且队列为空，直接返回false
        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        // 内层自旋
        for (;;) {
            int wc = workerCountOf(c);
            // worker数量超过容量，直接返回false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 使用CAS的方式增加worker数量。
            // 若增加成功，则直接跳出外层循环进入到第二部分
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            // 线程池状态发生变化，对外层循环进行自旋
            if (runStateOf(c) != rs)
                continue retry;
            // 其他情况，直接内层循环进行自旋即可
            // else CAS failed due to workerCount change; retry inner loop
        } 
    }
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            // worker的添加必须是串行的，因此需要加锁
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                // 这儿需要重新检查线程池状态
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    // worker已经调用过了start()方法，则不再创建worker
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // worker创建并添加到workers成功
                    workers.add(w);
                    // 更新`largestPoolSize`变量
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 启动worker线程
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // worker线程启动失败，说明线程池状态发生了变化（关闭操作被执行），需要进行shutdown相关操作
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

### 5、线程池worker任务单元

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable
{
    /**
     * This class will never be serialized, but we provide a
     * serialVersionUID to suppress a javac warning.
     */
    private static final long serialVersionUID = 6138294804551838833L;

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask;
    /** Per-thread task counter */
    volatile long completedTasks;

    /**
     * Creates with given first task and thread from ThreadFactory.
     * @param firstTask the first task (null if none)
     */
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        // 这儿是Worker的关键所在，使用了线程工厂创建了一个线程。传入的参数为当前worker
        this.thread = getThreadFactory().newThread(this);
    }

    /** Delegates main run loop to outer runWorker  */
    public void run() {
        runWorker(this);
    }

    // 省略代码...
}
```

### 6、核心线程执行逻辑-runworker

```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    // 调用unlock()是为了让外部可以中断
    w.unlock(); // allow interrupts
    // 这个变量用于判断是否进入过自旋（while循环）
    boolean completedAbruptly = true;
    try {
        // 这儿是自旋
        // 1. 如果firstTask不为null，则执行firstTask；
        // 2. 如果firstTask为null，则调用getTask()从队列获取任务。
        // 3. 阻塞队列的特性就是：当队列为空时，当前线程会被阻塞等待
        while (task != null || (task = getTask()) != null) {
            // 这儿对worker进行加锁，是为了达到下面的目的
            // 1. 降低锁范围，提升性能
            // 2. 保证每个worker执行的任务是串行的
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            // 如果线程池正在停止，则对当前线程进行中断操作
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            // 执行任务，且在执行前后通过`beforeExecute()`和`afterExecute()`来扩展其功能。
            // 这两个方法在当前类里面为空实现。
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                // 帮助gc
                task = null;
                // 已完成任务数加一 
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        // 自旋操作被退出，说明线程池正在结束
        processWorkerExit(w, completedAbruptly);
    }
}
```

# JMH Java准测试工具套件

## 什么是JMH

### 官网

 http://openjdk.java.net/projects/code-tools/jmh/ 

## 创建JMH测试

1. 创建Maven项目，添加依赖

   ```java
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <encoding>UTF-8</encoding>
           <java.version>1.8</java.version>
           <maven.compiler.source>1.8</maven.compiler.source>
           <maven.compiler.target>1.8</maven.compiler.target>
       </properties>
   
       <groupId>mashibing.com</groupId>
       <artifactId>HelloJMH2</artifactId>
       <version>1.0-SNAPSHOT</version>
   
   
       <dependencies>
           <!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-core -->
           <dependency>
               <groupId>org.openjdk.jmh</groupId>
               <artifactId>jmh-core</artifactId>
               <version>1.21</version>
           </dependency>
   
           <!-- https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess -->
           <dependency>
               <groupId>org.openjdk.jmh</groupId>
               <artifactId>jmh-generator-annprocess</artifactId>
               <version>1.21</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   
   
   </project>
   ```

2. idea安装JMH插件 JMH plugin v1.0.3

3. 由于用到了注解，打开运行程序注解配置

   > compiler -> Annotation Processors -> Enable Annotation Processing

4. 定义需要测试类PS (ParallelStream)

   ```java
   package com.mashibing.jmh;
   
   import java.util.ArrayList;
   import java.util.List;
   import java.util.Random;
   
   public class PS {
   
   	static List<Integer> nums = new ArrayList<>();
   	static {
   		Random r = new Random();
   		for (int i = 0; i < 10000; i++) nums.add(1000000 + r.nextInt(1000000));
   	}
   
   	static void foreach() {
   		nums.forEach(v->isPrime(v));
   	}
   
   	static void parallel() {
   		nums.parallelStream().forEach(PS::isPrime);
   	}
   	
   	static boolean isPrime(int num) {
   		for(int i=2; i<=num/2; i++) {
   			if(num % i == 0) return false;
   		}
   		return true;
   	}
   }
   ```

5. 写单元测试

   > 这个测试类一定要在test package下面
   >
   > ```java
   > package com.mashibing.jmh;
   > 
   > import org.openjdk.jmh.annotations.Benchmark;
   > 
   > import static org.junit.jupiter.api.Assertions.*;
   > 
   > public class PSTest {
   >  @Benchmark
   >  public void testForEach() {
   >      PS.foreach();
   >  }
   > }
   > ```

6. 运行测试类，如果遇到下面的错误：

   ```java
   ERROR: org.openjdk.jmh.runner.RunnerException: ERROR: Exception while trying to acquire the JMH lock (C:\WINDOWS\/jmh.lock): C:\WINDOWS\jmh.lock (拒绝访问。), exiting. Use -Djmh.ignoreLock=true to forcefully continue.
   	at org.openjdk.jmh.runner.Runner.run(Runner.java:216)
   	at org.openjdk.jmh.Main.main(Main.java:71)
   ```

   这个错误是因为JMH运行需要访问系统的TMP目录，解决办法是：

   打开RunConfiguration -> Environment Variables -> include system environment viables

7. 阅读测试报告

## JMH中的基本概念

1. Warmup
   预热，由于JVM中对于特定代码会存在优化（本地化），预热对于测试结果很重要
2. Mesurement
   总共执行多少次测试
3. Timeout
   
4. Threads
   线程数，由fork指定
5. Benchmark mode
   基准测试的模式
6. Benchmark
   测试哪一段代码



# Disruptor

作者：马士兵 http://www.mashibing.com

最近更新：2019年10月22日

## 介绍

主页：http://lmax-exchange.github.io/disruptor/

源码：https://github.com/LMAX-Exchange/disruptor

GettingStarted: https://github.com/LMAX-Exchange/disruptor/wiki/Getting-Started

api: http://lmax-exchange.github.io/disruptor/docs/index.html

maven: https://mvnrepository.com/artifact/com.lmax/disruptor

## Disruptor的特点

对比ConcurrentLinkedQueue : 链表实现

JDK中没有ConcurrentArrayQueue

Disruptor是数组实现的

无锁，高并发，使用环形Buffer，直接覆盖（不用清除）旧的数据，降低GC频率

实现了基于事件的生产者消费者模式（观察者模式）

## RingBuffer

环形队列

RingBuffer的序号，指向下一个可用的元素

采用数组实现，没有首尾指针

对比ConcurrentLinkedQueue，用数组实现的速度更快

> 假如长度为8，当添加到第12个元素的时候在哪个序号上呢？用12%8决定
>
> 当Buffer被填满的时候到底是覆盖还是等待，由Producer决定
>
> 长度设为2的n次幂，利于二进制计算，例如：12%8 = 12 & (8 - 1)  pos = num & (size -1)

## Disruptor开发步骤

1. 定义Event - 队列中需要处理的元素

2. 定义Event工厂，用于填充队列

   > 这里牵扯到效率问题：disruptor初始化的时候，会调用Event工厂，对ringBuffer进行内存的提前分配
   >
   > GC产频率会降低

3. 定义EventHandler（消费者），处理容器中的元素

## 事件发布模板

```java
long sequence = ringBuffer.next();  // Grab the next sequence
try {
    LongEvent event = ringBuffer.get(sequence); // Get the entry in the Disruptor
    // for the sequence
    event.set(8888L);  // Fill with data
} finally {
    ringBuffer.publish(sequence);
}
```

## 使用EventTranslator发布事件

```java
//===============================================================
        EventTranslator<LongEvent> translator1 = new EventTranslator<LongEvent>() {
            @Override
            public void translateTo(LongEvent event, long sequence) {
                event.set(8888L);
            }
        };

        ringBuffer.publishEvent(translator1);

        //===============================================================
        EventTranslatorOneArg<LongEvent, Long> translator2 = new EventTranslatorOneArg<LongEvent, Long>() {
            @Override
            public void translateTo(LongEvent event, long sequence, Long l) {
                event.set(l);
            }
        };

        ringBuffer.publishEvent(translator2, 7777L);

        //===============================================================
        EventTranslatorTwoArg<LongEvent, Long, Long> translator3 = new EventTranslatorTwoArg<LongEvent, Long, Long>() {
            @Override
            public void translateTo(LongEvent event, long sequence, Long l1, Long l2) {
                event.set(l1 + l2);
            }
        };

        ringBuffer.publishEvent(translator3, 10000L, 10000L);

        //===============================================================
        EventTranslatorThreeArg<LongEvent, Long, Long, Long> translator4 = new EventTranslatorThreeArg<LongEvent, Long, Long, Long>() {
            @Override
            public void translateTo(LongEvent event, long sequence, Long l1, Long l2, Long l3) {
                event.set(l1 + l2 + l3);
            }
        };

        ringBuffer.publishEvent(translator4, 10000L, 10000L, 1000L);

        //===============================================================
        EventTranslatorVararg<LongEvent> translator5 = new EventTranslatorVararg<LongEvent>() {

            @Override
            public void translateTo(LongEvent event, long sequence, Object... objects) {
                long result = 0;
                for(Object o : objects) {
                    long l = (Long)o;
                    result += l;
                }
                event.set(result);
            }
        };

        ringBuffer.publishEvent(translator5, 10000L, 10000L, 10000L, 10000L);
```

## 使用Lamda表达式

```java
package com.mashibing.disruptor;

import com.lmax.disruptor.RingBuffer;
import com.lmax.disruptor.dsl.Disruptor;
import com.lmax.disruptor.util.DaemonThreadFactory;

public class Main03
{
    public static void main(String[] args) throws Exception
    {
        // Specify the size of the ring buffer, must be power of 2.
        int bufferSize = 1024;

        // Construct the Disruptor
        Disruptor<LongEvent> disruptor = new Disruptor<>(LongEvent::new, bufferSize, DaemonThreadFactory.INSTANCE);

        // Connect the handler
        disruptor.handleEventsWith((event, sequence, endOfBatch) -> System.out.println("Event: " + event));

        // Start the Disruptor, starts all threads running
        disruptor.start();

        // Get the ring buffer from the Disruptor to be used for publishing.
        RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();


        ringBuffer.publishEvent((event, sequence) -> event.set(10000L));

        System.in.read();
    }
}
```

## ProducerType生产者线程模式

> ProducerType有两种模式 Producer.MULTI和Producer.SINGLE
>
> 默认是MULTI，表示在多线程模式下产生sequence
>
> 如果确认是单线程生产者，那么可以指定SINGLE，效率会提升
>
> 如果是多个生产者（多线程），但模式指定为SINGLE，会出什么问题呢？

## 等待策略

1，(常用）BlockingWaitStrategy：通过线程阻塞的方式，等待生产者唤醒，被唤醒后，再循环检查依赖的sequence是否已经消费。

2，BusySpinWaitStrategy：线程一直自旋等待，可能比较耗cpu

3，LiteBlockingWaitStrategy：线程阻塞等待生产者唤醒，与BlockingWaitStrategy相比，区别在signalNeeded.getAndSet,如果两个线程同时访问一个访问waitfor,一个访问signalAll时，可以减少lock加锁次数.

4，LiteTimeoutBlockingWaitStrategy：与LiteBlockingWaitStrategy相比，设置了阻塞时间，超过时间后抛异常。

5，PhasedBackoffWaitStrategy：根据时间参数和传入的等待策略来决定使用哪种等待策略

6，TimeoutBlockingWaitStrategy：相对于BlockingWaitStrategy来说，设置了等待时间，超过后抛异常

7，（常用）YieldingWaitStrategy：尝试100次，然后Thread.yield()让出cpu

8. （常用）SleepingWaitStrategy : sleep

## 消费者异常处理

默认：disruptor.setDefaultExceptionHandler()

覆盖：disruptor.handleExceptionFor().with()

## 依赖处理
