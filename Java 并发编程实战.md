### Java 并发编程实战读书笔记

- #### 同步容器类

  Jdk 提供了一些同步容器类，例如 Vector, HashTable,Collections.synchronizedXxx 等工厂方法创建的对象，

  还有 concurrent 包下如 ConcurrentHashMap 等。

  同步容器类都是线程安全的，但不代表基于同步容器的复合操作是同步的。

  常用的有：

  - ConcurrentHashMap

  - Collections.synchronizedXxx()

  - CopyOnWriteArrayList

  - CopyOnWriteArraySet

  - LinkedBlockingQueue

  - ArrayBlockingQueue

  - SynchronousQueue

    这其实不是一种真正意义上的队列，是一种线程间的任务交付机制，在 Executors.newCacheedThreadPool 方法创建的线程池中就使用的这种队列，它会直接将 Runnable 任务交给对应的线程执行，而不是将任务进入工作队列。这在一定程度上减少了开销。但是，这种情况仅仅试用于无界线程池。

  - ProorityBlockingQueue

  - Deque 

    双端队列，Stack 过时之后，官方推荐使用的栈实现（push、pop、peek）

- #### 同步工具类

  - 闭锁

    ~~~ java
    CountDownLatch latch = new CountDownLatch(3);
    
    ...
    //子线程中完成任务之后执行
      latch.countDown();
    ...
    //主线程等待任务执行完毕
    latch.await();
    ~~~

  - FutureTask

    执行可获取结果的异步任务

    Future.get() 将会阻塞到任务执行完成或者超时

  - 信号量

    ~~~ java
    Semaphore sem = new Semaphore(3);
    
    //子线程中获取许可 当资源超过上限时，将一直阻塞
    sem.acquire();
    
    //子线程中释放资源
    sem.release();
    
    ~~~

- #### Executor 框架

  ```
     /**
       * 各种线程池的实现：
       * newFixedThreadPool:
       * 固定长度的线程池，每提交一个任务创建一个线程，
       * 直到最大数量限制，然后规模不再变化，某个线程抛出异常时会创建新的代替
       * newCachedThreadPool:
       * 如果线程池的线程数量超过了任务规模将进行回收，当任务增加时添加新的线程，线程池规模没有限制
       * newSingleThreadExecutor:
       * 单工作线程的 executor,能保证任务的串行执行，如果此线程异常结束将创建一个新的代替
       * newScheduledThreadPool:
       * 创建一个固定长度的线程池，而且以延迟或者定时的方式来执行任务，类似于 Timer。
       */
  Executor exec =
          Executors.newFixedThreadPool(MAX_THREADS);
  ```

  ScheduledThreadPool 内部通过一个 WorkDelayQueue 延迟队列实现的延迟任务。jdk 也提供了 DelayQueue 工具类供使用, 这个队列只能加入实现了 Delayed 接口的对象。

  ~~~ java
  import java.util.concurrent.*;
  
  /**
   * ScheduledThreadPool 内部包含了一个类似于
   * DelayQueue 的延迟队列内部类
   * 借此实现了任务的延迟执行
   */
  public class DelayQueueCase {
  
      public static void main(String[] args) throws InterruptedException {
          DelayQueue<DelayMsg> queue = new DelayQueue<>();
          queue.put(new DelayMsg("111", 3000));
          queue.put(new DelayMsg("222", 6000));
          queue.put(new DelayMsg("333", 9000));
  
          while (true) {
              DelayMsg delayMsg = queue.take();
              System.out.println(delayMsg.getMessage());
          }
      }
  
  }
  
  class DelayMsg implements Delayed {
  
      private String message;
  
      private long time;
  
      public DelayMsg(String message, long delay) {
          this.message = message;
          this.time = System.currentTimeMillis() + delay;
      }
  
      public String getMessage() {
          return message;
      }
  
      @Override
      public long getDelay(TimeUnit unit) {
          return unit.convert(time - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
      }
  
      @Override
      public int compareTo(Delayed o) {
          return (int) (this.getDelay(TimeUnit.MILLISECONDS) - o.getDelay(TimeUnit.MILLISECONDS));
      }
  }
  ~~~

- #### ExecutorService 接口

  对 Executor 接口进行了拓展，增添了生命周期维护等功能。

  包括 shoudown、shutdownNow 等方法。

- #### CompletionService

  可用于批量执行多个异步任务，并获取返回结果。

  有点类似于 ExecutorService 的 invokeAll 方法，也是批量执行异步任务

  区别在于后者要等到所有任务执行完毕才能获取到 Future 结果集，

  而前者是先完成的先获取到。

  ~~~ java
  //传入一个线程池创建对象
  CompletionService<xxx> service = new ExecutorCompletionService<xxx>(executor);
  ...
  //提交任务
  service.submit(xxxCallable);
  
  //循环阻塞获取 先执行完成的先获取到
  Future<xxx> f = service.take();
  f.get()//获取结果
  ~~~

- #### 线程饥饿死锁

  线程饥饿死锁是指线程池中执行的任务之间存在依赖关系导致死锁的现象。当线程池中的任务有依赖关系，且线程池数量较小特别是单线程池时，比较容易发生。

  比如 SingleThreadExecutor 执行一个任务，这个任务里面又异步提交了两个任务到同一个线程池，并获取执行结果。由于只有一个线程，主任务会等待子任务执行返回结果，子任务又等着主任务执行完毕释放线程，这些就会死锁。

- #### ThreadPoolExecutor

  可以通过 Executors 的工厂方法来创建线程池，也可以通过构造函数来自定义创建线程池。

  Spring 中提供了一个 ThreadPoolTaskExecutor 的增强类 使用也比较方便

  ~~~ java
  /*
            创建一个固定线程池
            固定线程池的 core 和 max size 相等为入参
            存活时间为0 工作队列大小为默认的 Integer.MAX_VALUE
            等价于
            new ThreadPoolExecutor(10, 10, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())
           */
          ThreadPoolExecutor executor = (ThreadPoolExecutor) Executors.newFixedThreadPool(10);
          //设置饱和策略 abort 终止 | Discard 抛弃 | DiscardOldest 抛弃最老的 | callerRun 调用者线程执行
          executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
          //设置线程创建相关属性
          executor.setThreadFactory(
                  new ThreadFactoryBuilder()
                          //非守护线程
                          .setDaemon(false)
                          //名称前缀
                          .setNamePrefix("syozzz-")
                          .build()
          );
  ~~~

  当线程池的线程数量未达到 core 大小时，那么每接收一个任务就会创建一个新的线程，直到容量到达 core 。当数量达到 core 之后，新的任务将会开始添加到工作队列之中，如果工作队列也满了的话，那么会开始创建新的线程，直到达到 max 的大小，如果线程的空闲时间超过了 keepAliveTime, 那么此线程将被回收，直到线程池容量恢复到 core。

  特别注意：

  Executors.newFixedThreadPool 创建的现场池的工作队列默认大小为 Integer.MAX_VALUE，即工作队列为无界队列。

  Executors.newCachedThreadPool 创建的现场池，使用的队列为 SynchronousQueue，且 core 设置为0，max 设置为了 Interger.MAX_VALUE,  存活时间为 0。即此线程池为无界线程池。

- #### 线程池的饱和策略

  当有界队列被填满之后，就会触发饱和策略。

  - Abort 中止。默认，队列满了之后再提交，就会抛出 RejectedExecutionException。
  - Discard 抛弃，抛弃任务。
  - DiscardOldest 抛弃最老的，将抛弃优先级最高的等待任务。
  - Caller-Runs 将任务退回到调用者线程执行。
  
- #### 死锁

  - 锁顺序死锁

    锁顺序死锁是指两个线程试图以不同的顺序来获取相同的锁，而产生的死锁现象。

    比如有锁 A,B。线程1,2。线程1 中 以 A B 的顺序获取两个锁，线程2中以 BA的 顺序获取两个锁，那么就有可能存在两个线程交替执行，都获取到一个锁之后，等待对方释放另一个锁的现象。

    如果所有的线程都以固定的顺序来获取锁，就不会存在锁顺序死锁问题。

  - 动态的锁顺序死锁

    这是一种比较特殊的锁顺序死锁。以银行转账举例，账户 FROM 转账给账户 TO。代码中，先对 FROM 加锁，再对 TO 加锁，然后实现业务代码。这种情况表面上看都是固定的锁顺序没问题。但是，真实的锁对象确实动态的。比如 FROM 转账给 TO 的同时，TO 转账给 FROM，那么这种情况下，加锁的顺序刚好是相反的，就会出现死锁。可以考虑对两个账户求 hash ，然后根据 hash 大小的顺序来加锁，或者按账户ID顺序来加锁，避免死锁。

  - 资源死锁

    以不同的顺序获取两个资源，造成死锁。线程饥饿死锁也是一种资源死锁。

  - 减少死锁的策略

    - 锁范围
    - 锁粒度
    - 锁分段

- #### 显式锁

  Java 5.0 之前协调同步机制只有 synchronized 和 volatile。5.0 之后新增了显式锁。

  显式锁并不是一种替代内置锁的方法，而是当内置加锁机制不适用时，作为一种可选择的高级功能。

  

  
