---
title: Springboot自定义连接池并开启定时任务
date: 2019-06-26 17:16:12
tags: ["Java", "Springboot", "线程池"]
categories: ["后端"]
---

### Springboot自定义线程池

#### 线程池参数配置

----------

	## 连接池
	threadPool:
	  corePoolSize: 20
	  maxPoolSize: 40
	  queueCapacity: 1000
	  # 线程前缀名
	  threadNamePrefix: 'Async-Service-'
	  allowCoreThreadTimeout: false
	  keepAliveSeconds: 300


#### 线程池配置类
创建一个配置类ExecutorConfig, 用来自定义如何创建一个ThreadPoolTaskExecutor, 在类上添加@Configuration和@EnableAsync注解, 表示是配置类.

----------

	@Configuration
	@EnableAsync
	public class ExecutorConfig implements AsyncConfigurer {
	
	    private static Logger log = LoggerFactory.getLogger(ExecutorConfig.class);
	
	    @Value("${threadPool.corePoolSize}")
	    int corePoolSize;
	    @Value("${threadPool.maxPoolSize}")
	    int maxPoolSize;
	    @Value("${threadPool.queueCapacity}")
	    int queueCapacity;
	    @Value("${threadPool.threadNamePrefix}")
	    String threadNamePrefix;
	    @Value("${threadPool.keepAliveSeconds}")
	    int keepAliveSeconds;
	
	    @Bean("asyncExecutor")
	    public Executor asyncExecutor() {
	        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
	        executor.setCorePoolSize(corePoolSize);
	        executor.setMaxPoolSize(maxPoolSize);
	        executor.setQueueCapacity(queueCapacity);
	        executor.setKeepAliveSeconds(keepAliveSeconds);
	        executor.setThreadNamePrefix(threadNamePrefix);
	
	        // 线程池对拒绝任务的处理策略
	        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
	        // 初始化
	        executor.initialize();
	        return executor;
	    }
	
	    @Override
	    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
	        return new AsyncUncaughtExceptionHandler() {
	            @Override
	            public void handleUncaughtException(Throwable ex, Method method, Object... params) {
	                log.error("==========================" + ex.getMessage() + "=======================", ex);
	                log.error("exception method:" + method.getName());
	            }
	        };
	    }
	}

### Springboot开启多线程定时任务

以下为定时任务类, 定时任务由此定义开启

----------
	@Component
	public class ComputersTask {
	
	    private static Logger log = LoggerFactory.getLogger(ComputersTask.class);
	
	    @Autowired
	    private IemComputerService iemComputerService;
	
		// asyncExecutor 为自定义的线程池
	    @Async("asyncExecutor")
	    @Scheduled(fixedDelay = 1000 * 60)
	    public void startListen() throws InterruptedException {
        	System.out.println("线程--"+Thread.currentThread().getName()+"--开始!");
        	Thread.sleep(1000*10);
        	System.out.println("线程--"+Thread.currentThread().getName()+"--结束!");
	    }
	}

一般情况下, 定时任务默认只使用单线程跑任务, 但在实际开发过程中, 单线程肯定不满足我们的需求, 所以我们需要开启定时任务多线程,
因为我们之前有自定义线程池, 所以我们直接在之前的线程池配置类中实现SchedulingConfigurer接口.


----------

	@Configuration
	@EnableAsync
	@EnableScheduling
	public class ExecutorConfig implements AsyncConfigurer, SchedulingConfigurer {

	    private static Logger log = LoggerFactory.getLogger(ExecutorConfig.class);
	
	    @Value("${threadPool.corePoolSize}")
	    int corePoolSize;
	    @Value("${threadPool.maxPoolSize}")
	    int maxPoolSize;
	    @Value("${threadPool.queueCapacity}")
	    int queueCapacity;
	    @Value("${threadPool.threadNamePrefix}")
	    String threadNamePrefix;
	    @Value("${threadPool.keepAliveSeconds}")
	    int keepAliveSeconds;
	
	    @Bean("asyncExecutor")
	    public Executor asyncExecutor() {
	        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
	        executor.setCorePoolSize(corePoolSize);
	        executor.setMaxPoolSize(maxPoolSize);
	        executor.setQueueCapacity(queueCapacity);
	        executor.setKeepAliveSeconds(keepAliveSeconds);
	        executor.setThreadNamePrefix(threadNamePrefix);
	
	        // 线程池对拒绝任务的处理策略
	        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
	        // 初始化
	        executor.initialize();
	        return executor;
	    }
	
	    @Override
	    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
	        return new AsyncUncaughtExceptionHandler() {
	            @Override
	            public void handleUncaughtException(Throwable ex, Method method, Object... params) {
	                log.error("==========================" + ex.getMessage() + "=======================", ex);
	                log.error("exception method:" + method.getName());
	            }
	        };
	    }	
	
	    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
	        TaskScheduler taskScheduler = taskScheduler();
	        scheduledTaskRegistrar.setTaskScheduler(taskScheduler);
	    }
	
	    @Bean(destroyMethod = "shutdown")
	    public ThreadPoolTaskScheduler taskScheduler() {
	        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
	        scheduler.setThreadNamePrefix(threadNamePrefix);
	        scheduler.setPoolSize(corePoolSize);
	        scheduler.setAwaitTerminationSeconds(60);
	        scheduler.setWaitForTasksToCompleteOnShutdown(true);
	        // 线程池对拒绝任务的处理策略
	        scheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
	        // 初始化
	        scheduler.initialize();
	        return scheduler;
	    }



	}

这样我们的定时任务启动之后就可以多线程运行了.
![](p1.png)