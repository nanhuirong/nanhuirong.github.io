---
layout: post
title: Java并发编程
---
### 并发编程问题思考
+ 内置锁的缺点: 引起混乱, 迫使JVM在对象大小和加锁性能之间进行权衡
+ 最低安全保证: 线程可能得到一个失效的值, 但是这个值肯定时之前某个线程设置的而并非一个随机值(如HBase 和 CopyWrite机制). 但是64位数值不满足这个条件, JVM允许将64位的读写分为两个32位读写
+ ThreadLocal与线程封闭, 将需要封闭的共享对象放入ThreadLocal中
+ final域无法进行修改, 但如果引用的时可变变量, 这些引用对象是可以被修改的, 多线程编程一般用final修饰
+ 不可变对象时实现并发的一种优秀的方法
+ 加锁必须保证是同一把锁, 否则没有任何意义
+ 可重入机制: 一个线程可以二次获得由他持有的锁, 为每一把锁关联一个计数器和所有者线程, 比如需要调用父类方法
+ 锁的力度是线程级的, 保证了互斥性 + 内存可见性
+ java并发编程包: java.util.concurrent
+ JMM: java 内存模型<br>
>在共享内存的多处理器体系架构, 每个处理器都由自己的缓存, 并定期与主内存进行协调<br>
>重排序<br>
>偏序关系 Happens-Before, 如果两个操作之间缺乏偏序关系, JVM可以对他们进行任意重拍序<br>
>借助同步(Piggyback)

***
java.util.concurrent中更高级别工具：<br>
>Executor Framework<br>
>并发集合<Concurrent Collection><br>
>同步器（Synchronizer）使线程能够等待另一个线程的对象允许他们协调动作，CountDownLatch、Semaphore、CyclicBarrier、Exchanger<br>

***

### 并发编程基础概念
1.Java同步机制
+ synchronized: 隐式独占锁<br>
>用在代码块或者方法上, 处理临界资源<br>
>只能锁住对象, 不能锁定原始数据类型<br>
>被锁定的数组的单个对象不会被锁定<br>
>静态方法锁定class对象<br>
+ Lock: 显式独占锁, 并不比隐式高明太多, 只是增加了异常控制功能
+ CAS: 不加锁的线程访问方式, 在竞争不是特别激烈的情况下有较高的效率<br>
>独占锁是一种悲观的技术<br>
>CAS包含3个操作, 需要读取的内存位置V, 进行比较的值A, 需要写入的值B, 当且仅当V==A时B才会执行. 缺点时会将调用者处于竞争问题中.<br>
>类似于交通问题的红绿灯和环岛, 在交通拥堵的情况下, 信号灯有利于提高吞吐量, 不是特别拥堵时, 环岛能实现更高的效率<br>
>通常不会出现死锁, 优先级反转等情况, 但是有可能出现饥饿或者活锁<br>
>ABA问题:加入版本号<br>
+ volatile: 内存可见性, 并不意味着原子操作, 轻量级的同步机制<br>
>线程所见值会在使用时从主内存区读出<br>
>线程所写的值会在指令完成前flush到主内存<br>
>当一个变量依赖另一个变量或者当前变量的值依赖于旧值, 不适合<br>
+ 原子变量: 包括数值型和非数值型<br>
>更好的volatile<br>
>标量:AtomicInteger, AtomicLong, AtomicBoolean, AtomicRefence<br>

2.监控
+ 监控CPU使用率: vmstat
+ dump线程信息: jstack -l PID > file
+ iostat: 监控磁盘利用率

3.任务执行的框架: Executor框架
+ 基于生产者消费者模式: 提交任务的线程相当于生产者, 执行任务的线程相当于消费者
+ Executors.newFixedThreadPool(): 创建一个固定长度的线程池 利用LinkedBlockingQueue 表示一个无界队列
+ Executors.newCachedThreadPool() 创建一个可缓存的线程, 动态扩张和缩小, 线程池的规模不受任何限制
+ Executors.newSingleThreadExecutor() 创建一个单线程的线程池利用LinkedBlockingQueue 表示一个无界队列
+ Executors.newScheduledThreadPool()  创建一个固定长度的线程池, 而且以延时或者定时的方式执行，替代java。util。Timer
+ 关闭线程池: ExecutorService<br>
>shutDown()  平滑结束线程池 不再接受新任务, 并等待已提交任务的完成<br>
>shutdownNow()   尝试取消所有的运行中的任务, 并不再执行已经提交的任务并将其返回<br>
>缺点: 没有办法返回一个值, 抛出一个受检测的异常, 已提交但是未开始的任务可以取消, 已开始的任务只有当他们可以响应中断时才可以取消<br>
>ThreadPoolExecutor支持更高程度的对线程池的控制<br>

4.管理延时任务和周期任务(有缺陷)
+ 替代方案:<br>
>考虑用newScheduledThreadPool()来替代<br>
>利用DelayQueue + SchedulerThreadPoolExecutor<br>

5.描述任务的生命周期(Callable 和 Future)
+ Runnable 与Callable<br>
>都是描述计算任务的抽象
>Callable: 会有一个返回值或者一个异常<br>
+ Future可用于管理定时任务<br>
>通常中断是取消最合理的方式<br>
>通过future来实现取消, Future.cancel, true 表示正在运行的任务可以被取消, false任务还没启动就不用取消<br>

6.Amdahl定律<br>
>speedup <= 1/(F + (1 - F)/ N)<br>
>N处理器个数<br>
>F必须被串行的部分<br>

7.锁分段与锁分解计数
+ 锁分段: 锁上的竞争呢个频率高于锁上数据发生竞争呢个的频率

8.Lock: 显式锁
+ ReentrantLock 实现了Lock接口, 并提供与synchronized相同的互斥性与内存可见性，支持轮询和定时操作,避免写写, 写读冲突<br>
>提供了可重入语义, (公平锁)如果一个线程获取读锁, 有一个线程等待写入锁, 那么其他线程均不能获取读锁,直到写线程释放写锁<br>
>(非公平锁)线程访问许可的顺序是不确定的, 写线程降级为读线程时可以的, 但是读线程升级为写线程是不可以的<br>
+ ReadWriteLock 读写锁, 允许多个线程同时读, 但是只允许一个线程写, 一个读锁, 一个写锁

### 代码实践
1.缓存最近执行的因式分解的数值及其计算结果
+ 基于可变对象
```java
@ThreadSafe
public class CachedFactorizer {
    @GuardedBy("this") private BigInteger lastNumber;
    @GuardedBy("this") private BigInteger[] lastFactors;
    @GuardedBy("this") private long hits;
    @GuardedBy("this") private long cacheHits;

    public synchronized long getHits(){
        return hits;
    }

    public synchronized double getCacheHitRatio(){
        return (double) cacheHits / (double) hits;
    }

    public void service(BigInteger number){
        BigInteger[] factors = null;
        synchronized (this){
            ++hits;
            if (number.equals(lastNumber)){
                ++cacheHits;
                factors = lastFactors;
            }
        }
        if (factors == null){
            factors = getFactors(number);
            synchronized (this){
                lastNumber = number;
                lastFactors = factors.clone();
            }
        }
    }

    private BigInteger[] getFactors(BigInteger number){
        BigInteger[] factors = null;
        return factors;
    }


    public static void main(String[] args) {
        final ConcurrentHashMap<Integer, Integer> conMap = new ConcurrentHashMap<>();
        conMap.put(1, 1);
        conMap.put(2, 2);
        final Map<Integer, Integer> unmodify = Collections.unmodifiableMap(conMap);
        System.out.println(unmodify);
        conMap.put(conMap.get(2), 3);
        System.out.println(conMap);
        System.out.println(unmodify);
    }
}
```
+ 基于不可变对象实现
```java
public class OneValueCache {
    private final BigInteger lastNumber;
    private final BigInteger[] lastFactors;

    public OneValueCache(BigInteger lastNumber, BigInteger[] lastFactors) {
        this.lastNumber = lastNumber;
        this.lastFactors = lastFactors;
    }
    public BigInteger[] getLastFactors(BigInteger number){
        if (lastNumber == null || !lastNumber.equals(number)){
            return null;
        }
        else {
            return Arrays.copyOf(lastFactors, lastFactors.length);
        }
    }
}
public class VolatileCacheFactor {
    private volatile OneValueCache cache = new OneValueCache(null, null);
    public void service(BigInteger number){
        BigInteger[] factors = cache.getLastFactors(number);
        if (factors == null){
            factors = getFactors(number);
            cache = new OneValueCache(number, factors);
        }
    }

    private BigInteger[] getFactors(BigInteger number){
        return null;
    }
}
```

2.车辆跟踪
+ 不可变对象
```java
@Immutable
public class Point {
    public final int x, y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
@ThreadSafe
public class DelegatingVehicleTracker {
    private final ConcurrentHashMap<String, Point> locations;
    private final Map<String, Point> unmodifiableMap;
    public DelegatingVehicleTracker(ConcurrentHashMap<String, Point> locations) {
        this.locations = locations;
        //维护location的一个只读模式的快照, 会实时进行更新
        this.unmodifiableMap = Collections.unmodifiableMap(locations);
    }
    public Map<String, Point> getLocations(){
        return unmodifiableMap;
    }
    //返回一个静态的拷贝
    public Map<String, Point> getStaticLocations(){
        return Collections.unmodifiableMap(new HashMap<>(locations));
    }
    public Point getLocation(String id){
        return locations.get(id);
    }
    public void setLocation(String id, int x, int y){
        if (locations.replace(id, new Point(x, y)) == null){
            throw new IllegalArgumentException("id 不存在");
        }
    }
}
```
+ 可变对象
```java
@ThreadSafe
public class SafePoint {
    @GuardedBy("this") private int x, y;
    private SafePoint(int[] a) {
        this(a[0], a[1]);
    }
    public SafePoint(SafePoint p) {
        this(p.get());
    }
    public SafePoint(int x, int y) {
        this.set(x, y);
    }
    public synchronized int[] get() {
        return new int[]{x, y};
    }
    public synchronized void set(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
@ThreadSafe
public class PublishingVehicleTracker {
    private final Map<String, SafePoint> locations;
    private final Map<String, SafePoint> unmodifiableMap;
    public PublishingVehicleTracker(Map<String, SafePoint> locations) {
        this.locations = new ConcurrentHashMap<String, SafePoint>(locations);
        this.unmodifiableMap = Collections.unmodifiableMap(this.locations);
    }
    public Map<String, SafePoint> getLocations() {
        return unmodifiableMap;
    }
    public SafePoint getLocation(String id) {
        return locations.get(id);
    }
    public void setLocation(String id, int x, int y) {
        if (!locations.containsKey(id))
            throw new IllegalArgumentException("invalid vehicle name: " + id);
        locations.get(id).set(x, y);
    }
}
```

3.闭锁
```java
public class TestHarness {
    public long timeTasks(int nThreads, final Runnable task)
            throws InterruptedException {
        final CountDownLatch startGate = new CountDownLatch(1);
        final CountDownLatch endGate = new CountDownLatch(nThreads);

        for (int i = 0; i < nThreads; i++) {
            Thread t = new Thread() {
                public void run() {
                    try {
                        startGate.await();
                        try {
                            task.run();
                        } finally {
                            endGate.countDown();
                        }
                    } catch (InterruptedException ignored) {
                    }
                }
            };
            t.start();
        }

        long start = System.nanoTime();
        startGate.countDown();
        endGate.await();
        long end = System.nanoTime();
        return end - start;
    }
}
```

4.栅栏
```java
public class CellularAutomata {
    private final Board mainBoard;
    private final CyclicBarrier barrier;
    private final Worker[] workers;

    public CellularAutomata(Board mainBoard) {
        this.mainBoard = mainBoard;
        int count = Runtime.getRuntime().availableProcessors() / 2;
        this.barrier = new CyclicBarrier(count, new Runnable() {
            @Override
            public void run() {
                mainBoard.commitNewValues();
            }
        });
        this.workers = new Worker[count];
        for (int i = 0; i < count; i++){
            workers[i] = new Worker(mainBoard.getSubBoard(count, i));
        }
    }
    private class Worker implements Runnable{
        private final Board board;
        public Worker(Board board){
            this.board = board;
        }

        @Override
        public void run() {
            while (!board.hasConverged()){
                for (int x = 0; x < board.getMaxX(); x++){
                    for (int y = 0; y < board.getMaxY(); y++){
                        board.setNewValue(x, y, computeValue(x, y));
                    }
                }
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        }
        private int computeValue(int x, int y){
            return 0;
        }
    }
    public void start(){
        for (int i = 0; i < workers.length; i++){
            new Thread(workers[i]).start();
        }
        mainBoard.waitForConvergence();
    }
    interface Board{
        int getMaxX();
        int getMaxY();
        int getValue(int x, int y);
        int setNewValue(int x, int y, int value);
        void commitNewValues();
        boolean hasConverged();
        void waitForConvergence();
        Board getSubBoard(int numPartitions, int index);
    }
}
```

5.并发缓存
```java
public class Memoizer <A, V> implements Computable<A, V> {
    private final ConcurrentMap<A, Future<V>> cache
            = new ConcurrentHashMap<A, Future<V>>();
    private final Computable<A, V> c;

    public Memoizer(Computable<A, V> c) {
        this.c = c;
    }

    public V compute(final A arg) throws InterruptedException {
        while (true) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> eval = new Callable<V>() {
                    public V call() throws InterruptedException {
                        return c.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<V>(eval);
                f = cache.putIfAbsent(arg, ft);
                if (f == null) {
                    f = ft;
                    ft.run();
                }
            }
            try {
                return f.get();
            } catch (CancellationException e) {
                cache.remove(arg, f);
            } catch (ExecutionException e) {
                throw LaunderThrowable.launderThrowable(e.getCause());
            }
        }
    }
}
```