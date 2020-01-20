---
title: Java线程池的技术及优化
date: 2019-09-04 10:57:49
tags: ["Java", "线程池"]
categories: ["后端"]
---

### 创建线程池的demo

    private static final int POOL_CORE_SIZE = 10; //核心线程数
	private static final int POOL_MAX_SIZE = 20; //最大线程数
	private static final int KEEP_ALIVE_TIME = 2000; // 单个线程保活时间(毫秒)
	private static final int QUEUE_SIZE = 20; //等待队列大小

	// 创建有界队列
	BlockingQueue<Runnable> workQueue = new SynchronousQueue<Runnable>(QUEUE_SIZE);

	// 线程的创建工厂
	ThreadFactory threadFactory = new ThreadFactory() {
		private final AtomicInteger mCount = new AtomicInteger(1);

		public Thread newThread(Runnable r) {
			return new Thread(r, "AdvacnedAsyncTask #" + mCount.getAndIncrement());
		}
	};

	// 线程池任务满载后采取的任务拒绝策略
	RejectedExecutionHandler rejectHandler = new ThreadPoolExecutor.DiscardOldestPolicy();

	// 线程池对象, 创建线程
	ThreadPoolExecutor mExecute = new ThreadPoolExecutor(
		POOL_CORE_SIZE,
		POOL_MAX_SIZE,
		KEEP_ALIVE_TIME,
		TimeUnit.MILLISECONDS,
		workQueue,
		threadFactory,
		rejectHandler
	);

### 参数解析

- corePoolSize：核心线程数，会一直存活，即使没有任务，线程池也会维护线程的最少数量
- maximumPoolSize：线程池维护线程的最大数量
- keepAliveTime：指的是空闲线程结束的超时时间（当一个线程不工作时，过keepAliveTime 长时间将停止该线程）
- allowCoreThreadTimeout：设置为true，则所有线程均会退出直到线程数量为0
- unit：线程池维护线程所允许的空闲时间的单位、可选参数值为：TimeUnit中的几个静态属性：NANOSECONDS、MICROSECONDS、MILLISECONDS、SECONDS
- workQueue：线程池所使用的缓冲队列，常用的是：java.util.concurrent.ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue
- threadFactory：执行程序创建新线程时使用的工厂
- handler：线程池中的数量大于maximumPoolSize，对拒绝任务的处理策略，默认值ThreadPoolExecutor.AbortPolicy()

### 参数之间关系如下：

#### 第一类：

控制被处理线程：corePoolSize，maximumPoolSize，workQueue会决定你的线程处理顺序，首先进来线程会将该线程池的实际线程数变为corePoolSize，达到corePoolsize后，再进来的线程就会进入workQueue来排队；如果workQueue满了，再进来的线程就会继续创建线程，直到实际个数达到maximumPoolSize；

而关于workQueue有下面三种方式：

1、直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes， 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

2、无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

3、有界队列。当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折中：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。

