# JUC 学习笔记

## Synchronized

使用**对象锁**来进行并发控制，最常见的一种并发控制方法，利用了操作系统提供的 monitor。

### wait-notify

一种**线程通信方式**，当一个线程抢到了锁，但由于某些条件无法执行，会使用`wait`方法，进入等待队列，并释放手中的锁。当条件满足时，会调用`notify`方法将等待的线程唤醒。

p.s. 这两个方法都只能在同步代码块里面使用

**代码示例**

```java
synchronized(lock) {
    while(条件不成立) {
        lock.wait();
    }
    // 干活
}
 //另一个线程
synchronized(lock) {
    // 发现条件满足
    lock.notifyAll();
 }
```



## 线程的六种状态

![image-20241224200228019](https://gitee.com/SheeepHappy/blog-pic/raw/master/20250331152235112.png)

waiting 和 blocked 的区别和联系：

- blocked 状态线程**没有竞争到锁**进入的阻塞状态，而 wating 状态是竞争到锁但由于缺少条件运行主动释放锁进入的等待状态
- waiting 状态的线程如果被唤醒，会去竞争锁，如果失败就转换到 blocked 状态





## 线程池



![image-20241225090413379](https://gitee.com/SheeepHappy/blog-pic/raw/master/20250331152235113.png)

### 线程池参数

![image-20241225100051889](https://gitee.com/SheeepHappy/blog-pic/raw/master/20250331152235114.png)

核心线程数

最大线程数

空闲时间

任务队列

线程工厂

拒绝策略

### 线程池的执行流程

1. 提交任务
2. 若有空闲线程，执行；否则，进入任务队列，若队列已满且线程全部忙碌会执行拒绝策略。
3. 线程执行完任务会回到线程池，满足一定条件时，空闲线程会被销毁。

### 线程池缩容如何感知

1. 定期查询线程池的 `getPoolSize()`、`getActiveCount()` 等属性；

2. 监控拒绝策略的触发情况；

3. 自定义钩子方法来跟踪线程池状态的变化。

### 关闭线程池

1. `shutdown`方法：优雅地关闭线程池，等待所有任务执行完毕，不再接收新任务，此方法**不会阻塞调用线程的执行**。
   - `awaitTermination`：可以使用这个方法等待线程池真正结束。
2. `shutdownNow` 方法：立即关闭线程池，尝试中断正在执行的任务，并返回未执行的任务列表。



## 多线程常用模式

### 保护性暂停

用于一个线程等待另一个线程的结果，主要利用一个中间对象实现。

**代码示例**

```java
@Slf4j
public class GuardedSuspension {
    public static void main(String[] args) {
        GuardedObject guardedObject = new GuardedObject();

        Thread t1 = new Thread(() -> {
            log.debug("等待结果...");
            String response = (String) guardedObject.getResponse(4000);
            log.debug("收到结果：{}", response);
        }, "等待线程");

        Thread t2 = new Thread(() -> {
            log.debug("正在执行下载...");
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("下载完成");
            guardedObject.setResponse("hello");


        }, "下载线程");

        t1.start();
        t2.start();
    }

}

// 保护性暂停的中间对象
class GuardedObject {
    // 某线程等待的结果
    private Object response;

    public Object getResponse() {
        synchronized (this) {
            while (response == null) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return response;
        }
    }

    public Object getResponse(long timeout) {
        synchronized (this) {
            long begin = System.currentTimeMillis();

            while (true) {
                long waitTime = timeout - (System.currentTimeMillis() - begin);
                if(waitTime <= 0){
                    break;
                }
                try {
                    // 防止被意外唤醒导致等待时间过长
                    this.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return response;
        }
    }

    public void setResponse(Object response) {
        synchronized (this) {
            this.response = response;
            this.notifyAll();
        }
    }
}
```



### 两阶段终止模式

使用 `interrput` 优雅地停止一个线程的活动。

**代码示例**

```java
@Slf4j
class TwoPhaseTermination {
    // 定期执行任务的监视器
    private Thread monitor;

    public void start() {
        monitor = new Thread(() -> {
            while(true){
                Thread thread = Thread.currentThread();
                // 检查是否被打断
                if(thread.isInterrupted()){
                    log.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(2500);
                } catch (InterruptedException e) {
                    log.debug("出现异常", e);
                    thread.interrupt();
                }
                log.debug("执行监控任务");
            }
        });

        monitor.start();
    }
    public void stop(){
        monitor.interrupt();
    }
}
```



### Balking 模式

和双检索单例模式很像，用于确保**某个任务只有一个线程会执行**。

```java
@Slf4j
class BalkingMonitor {

    private Thread monitor;

    private volatile boolean start = false;

    public void start() {
        if(start){
            return;
        }

        synchronized (this){
            if(start){
                return;
            }
            monitor = new Thread(() -> {
                while(true){
                    Thread thread = Thread.currentThread();
                    if(thread.isInterrupted()){
                        log.debug("料理后事");
                        break;
                    }
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        log.debug("被打断了");
                        thread.interrupt();
                    }
                    log.debug("执行监控任务");
                }
            });
            monitor.start();
            start = true;
        }

        
    }
    
    public void stop(){
        monitor.interrupt();
        start = false;
    }
}
```



### 顺序控制

三个线程轮流执行

#### wait-notify 实现

```java
class WaitNotify{
    private int loopNum;

    private int flag;

    public WaitNotify(int loopNum, int flag) {
        this.loopNum = loopNum;
        this.flag = flag;
    }

    public void print(String str, int waitFlag, int nextFlag){
        for (int i = 0; i < loopNum; i++) {
            synchronized (this){
                while(flag != waitFlag){
                    try {
                        this.wait();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }

                System.out.print(str);
                flag = nextFlag;
                this.notifyAll();
            }
        }
    }
}
```
#### await-signal 实现

```java
class AwaitSignal extends ReentrantLock {
    private int loopNum;

    public AwaitSignal(int loopNum) {
        this.loopNum = loopNum;
    }

    public void print(String str, Condition current, Condition next){
        for (int i = 0; i < loopNum; i++) {
            lock();
            try{
                current.await();
                System.out.print(str);
                next.signal();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                unlock();
            }
        }
    }
}
```




#### park-unpark 实现(最佳)

```java
class AwaitSignal extends ReentrantLock {
    private int loopNum;

    public AwaitSignal(int loopNum) {
        this.loopNum = loopNum;
    }

    public void print(String str, Condition current, Condition next){
        for (int i = 0; i < loopNum; i++) {
            lock();
            try{
                current.await();
                System.out.print(str);
                next.signal();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                unlock();
            }
        }
    }
}
```

> 从这开始就是异步的模式

### 生产者-消费者模式

```java
@Slf4j
class MessageQueue{

    private LinkedList<Message> list = new LinkedList<>();

    private int capacity;

    public MessageQueue(int capacity) {
        this.capacity = capacity;
    }

    public void put(Message message){
        synchronized (list){
            while (list.size() == capacity){
                try {
                    log.debug("消息队列已满，无法放入消息");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.add(message);
            log.debug("放入消息：{}", message);
            list.notifyAll();
        }
    }

    public Message take(){
        synchronized (list){
            while (list.isEmpty()){
                try {
                    log.debug("消息队列为空，无法获取消息");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            Message message = list.removeFirst();
            log.debug("获取消息：{}", message);
            list.notifyAll();
            return message;
        }
    }
}
```

### 工作线程模式

让**有限的工作线程**（Worker Thread）来轮流**异步处理无限多的任务**。也可以将其归类为分工模式，它的典型实现就是**线程池**，也体现了经典设计模式中的享元模式。

```java
/**
 * 工作线程模式
 * 不同类型的任务由不同的线程池执行
 */
@Slf4j
public class WorkerThread {
    static final List<String> MENU = Arrays.asList("地三鲜", "宫保鸡丁", "辣子鸡丁", "烤鸡翅");
    static Random RANDOM = new Random();
    static String cooking() {
        return MENU.get(RANDOM.nextInt(MENU.size()));
    }

    public static void main(String[] args) {
        ExecutorService waitPool = Executors.newFixedThreadPool(2);
        ExecutorService cookPool = Executors.newFixedThreadPool(1);
        waitPool.execute(() -> {
            log.debug("处理点餐...");
            Future<String> f = cookPool.submit(() -> {
                log.debug("做菜");
                return cooking();
            });
            try {
                log.debug("上菜: {}", f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });

        waitPool.execute(() -> {
            log.debug("处理点餐...");
            Future<String> f = cookPool.submit(() -> {
                log.debug("做菜");
                return cooking();
            });
            try {
                log.debug("上菜: {}", f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });

        waitPool.execute(() -> {
            log.debug("处理点餐...");
            Future<String> f = cookPool.submit(() -> {
                log.debug("做菜");
                return cooking();
            });
            try {
                log.debug("上菜: {}", f.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        });

    }
}
```

#### 线程池的参数设置

**CPU 密集型任务**

线程数设置为`cpu 核数 + 1`，既能充分利用 CPU，又能较少上下文切换带来的性能损耗。

**I/O 密集型任务**

可以适当增多线程数，参考公式：`线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间`

#### 自定义线程池

**阻塞队列**

```java
/**
 * 阻塞队列
 * @param <T>
 */
@Slf4j
public class BlockingQueue<T> {
    private Deque<T> queue = new ArrayDeque<>();

    private int capacity;

    private ReentrantLock lock = new ReentrantLock();
    // 队列为空时的休息室
    private Condition emptyWaitSet = lock.newCondition();
    // 队列满时的休息室
    private Condition fullWaitSet = lock.newCondition();

    public BlockingQueue(int capacity) {
        this.capacity = capacity;

    }

    // 添加任务（队列满时就阻塞）
    public void put(T t) {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                try {
                    fullWaitSet.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            queue.addLast(t);
            emptyWaitSet.signal();
        } finally {
            lock.unlock();
        }
    }

    /**
     * 添加任务，如果队列满了，则等待,等待超时就放弃
     * @param t
     * @param timeout
     * @param unit
     * @return
     */
    public boolean offer(T t, long timeout, TimeUnit unit) {
        lock.lock();
        try {
            long nanos = unit.toNanos(timeout);
            while (queue.size() == capacity) {
                try {
                    if(nanos <= 0){
                        log.debug("放弃添加任务，{}", t);
                        return false;
                    }
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            queue.addLast(t);
            log.debug("添加任务，{}", t);
            emptyWaitSet.signal();
            return true;
        } finally {
            lock.unlock();
        }
    }

    // 获取任务（阻塞）
    public T take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 获取任务 (带超时时间）
    public T poll(Long timeout, TimeUnit unit) {
        lock.lock();
        try {
            long nanos = unit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    if(nanos <= 0){
                        return null;
                    }
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 尝试放入，带拒绝策略
    public void tryPut(T task, RejectPolicy<T> rejectPolicy) {
        lock.lock();
        try {
            if (queue.size() == capacity) {
                rejectPolicy.reject(this, task);
            } else {
                queue.addLast(task);
                log.debug("新增任务{}，当前任务数量{}", task, queue.size());
            }

        } finally {
            lock.unlock();
        }
    }
}

```

**拒绝策略**

```java
@FunctionalInterface
public interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue, T task);
}
```

**线程池**

```java
/**
 * 自定义的线程池
 */
@Slf4j
public class ThreadPool {
    /**
     * 任务队列
     */
    private BlockingQueue<Runnable> taskQueue;

    // 线程集合
    private HashSet<Worker> workers = new HashSet<>();
    // 核心线程数
    private int coreSize;

    private long timeout;

    private TimeUnit timeUnit;

    private RejectPolicy rejectPolicy;

    public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity, RejectPolicy rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }

    public void execute(Runnable task){
        synchronized (workers){
            if(workers.size() < coreSize){
                Worker worker = new Worker(task);
                log.debug("新增 worker, {}", worker);
                workers.add(worker);
                worker.start();
            }else{
                taskQueue.tryPut(task, rejectPolicy);
            }
        }
    }

    class Worker extends Thread{

        Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            while(task != null || (task = taskQueue.poll(timeout, timeUnit)) != null){
                // 有任务
                try{
                    task.run();
                    log.debug("执行任务 {}", task);
                } finally {
                    task = null;
                }
            }
            // 无任务
            synchronized (workers){
                workers.remove(this);
                log.debug("移除 worker, {}", this);
            }

        }
    }
}

```

## 源码解读

1. [终于有人把 AQS 说清楚了！万字详解一、AQS 是啥？有啥用？ 一、AQS 是啥？有啥用？ 在 Java 并发编程的世 - 掘金](https://juejin.cn/post/7454026458250166272?searchId=20250331152418C8845647EF989E700C5F)
2. [从ReentrantLock的实现看AQS的原理及应用 - 美团技术团队](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)
