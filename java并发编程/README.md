## java并发编程基础
#### 1、什么是线程和进程
在介绍什么是线程之前，有必要对进程进行了解下，在操作系统中线程是进程中的一个实体，线程并不会独立存在，进程是资源分配和调度的基本单位，一个进程中最少有一个线程，多个线程共享一个进程内的资源。
总结
* 进程：程序运行资源分配的最小单位，进程内部有多个线程，会共享这个进程的资源
* 线程：CPU调度的最小单位，必须依赖进程而存在。

#### 2、什么是并发和并行
* 并行：指两个或多个事件在同一时间点发生。列如我们在学生时代，排队打饭，有多个窗口可以同时排队打饭。这就是并行。
* 并发：指单位时间内，处理事情的能力。列如我们在排队打饭时，单个窗口5分钟内可以处理10个人打饭，这就叫做并发。

#### 3、什么是同步和异步

* 同步：所谓同步，就是发出一个功能调用时，在没有得到结果之前，该调用就不返回或继续执行后续操作。
* 异步：异步与同步相对，当一个异步过程调用发出后，调用者在没有得到结果之前，就可以继续执行后续操作。当这个调用完成后，一般通过状态、通知和回调来通知调用者。对于异步调用，调用的返回并不受调用者控制。

#### 4、如何学习好java并发编程
是不是总有一种感觉，在项目开发遇到问题时，打比方在了解一些并发工具类的使用时，会查阅相关资料，但过段时间又忘了，总感觉我已经学习了好多知识，但还是搞不懂，有时候好不容易解决这个问题，但又不知道这样做是不是对的或者是最优方案，那怎么样才能学习好并发编程？ 其实在之前我也遇到过这样的问题，其实就2点，一个是从现象看本质，深入源码学习，二个是对整体并发工具类有个大体了解，最起码能知道有哪些工具类，在解决实际问题中，他们的优缺点是什么。有些人会问，那我已经知道这些并发工具类的使用用途，源码也看了，但他们为什么这样设计，这样设计有什么好处，我只能说这样的问题只有问并发编程大师Doug Lea了。附带一张java并发编程图谱(来自于网络)
![java并发编程图谱](https://github.com/ibc789/my-java-study/blob/master/img/thread/thread-1.png "java并发编程图谱")

#### 5、java多线程并发三个核心问题：分工、同步和互斥

* 分工：列如在装修房屋时，装修公司的工作就是把房屋装修好，交接给客户进行验收，装修公司在安排任务时为了提高效率，安排木工负责打衣柜、橱柜，电工负责电线铺设等，通俗点讲就是安排不同的人做不同的事。在Java世界里，Java SDK 并发包里的 Executor、Fork/Join、Future本质上也是一种分工。
* 同步：不过是一个线程执行完了一个任务，如何通知执行后续任务的线程开工而已
* 互斥：所谓互斥，指的是同一时刻，只允许一个线程访问共享变量。




#### 6、java线程创建的3种方式 实现Runable接口的run方法、继承Thread类重写run方法、实现Callable接口的call方法
* **6.1、通过实现Runable接口，如下**
```
public class RunableTest {

   public static class RunableTask implements Runnable{
       @Override
       public void run() {
           System.out.println("hello world");
       }
   }
    public static void main(String[] args) {
        RunableTask RunableTask = new RunableTask();
        Thread thread = new Thread(RunableTask);
        thread.start();
    }
}
```
   上述代码实现Runable接口的run方法，这样做的好处是RunableTask可以在继承其他类利于扩展，坏处是不能用this关键字获取当前线程相关信息，必须通过Thread.currentThread()方法来获取。

* **6.2、通过继承Thread类，如下**
```
public class ThreadTest extends Thread {

    public static class ThreadTask extends Thread{
        @Override
        public void run() {
            System.out.println("hello world");
        }
    }
    public static void main(String[] args) {
        ThreadTask threadTask = new ThreadTask();
        threadTask.start();
    }
}

```
   1、上述代码创建一个ThreadTest类，在main函数中创建ThreadTest的实例，并调用start方法启动线程。注意当然调用start方法后，线程并没有马上执行，而是处于就绪状态，这个就绪状态指线程除了获取CPU资源外已经获取了其他资源，等待获取CPU资源后才真正是运行状态，当然运行完run方法后，线程终止结束。

   2、继承的好处是可以在run方法中直接可以使用this关键字获取当前线程相关信息，而不用在使用Thread.currentThread()法，但坏处是，我们都知道java是单继承，这样做不利于扩展。

* **6.3、通过实现Callable接口，如下**
```
public class CallableTest {

   public static class CallableTask implements Callable<String>{
       @Override
       public String call() {
           return "hello world";
       }
   }

    public static void main(String[] args) {
       try {
           CallableTask callableTask = new CallableTask();
           FutureTask<String> futureTask = new FutureTask<>(callableTask);
           new Thread(futureTask).start();
           String res = futureTask.get();
           System.out.println(res);
       }catch (Exception e){
           e.printStackTrace();
       }
    }
}
```
   1、上述代码是通过FutureTask方式运行线程。

   2、它比Runable和Thread的优点是它可以有返回值，缺点是在使用FutureTask的get方法获取返回值时它是阻塞的。

   3、call接口可以抛出异常，而Runable必须通过setUncaughtExceptionHandler()设置异常，才能在主线程中捕获到子线程的异常。

   4、调用start方法后再次调用报IllegalThreadStateException异常，如下源码蓝色所示
   ```
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            $\color{#4285f4}{throw new IllegalThreadStateException();}$

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
   ```

   


* **6.4、他们3者关系如下**

| Runable接口 | Thread类 | Callable接口 |
| ------ | ------ | ------ |
| ![Runable接口](https://github.com/ibc789/my-java-study/blob/master/img/thread/thread-2.jpg "Runable接口") | ![Thread类](https://github.com/ibc789/my-java-study/blob/master/img/thread/thread-3.jpg "Thread类") | ![Callable接口](https://github.com/ibc789/my-java-study/blob/master/img/thread/thread-4.jpg "Callable接口") |

