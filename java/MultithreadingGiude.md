# Java Multithreading Complete Study Guide

## Table of Contents
1. [Thread Fundamentals](#thread-fundamentals)
2. [Executor Framework](#executor-framework)
3. [Custom Thread Pools](#custom-thread-pools)
4. [CompletableFuture](#completablefuture)
5. [ForkJoin Framework](#forkjoin-framework)
6. [Scenario-Based Questions](#scenario-based-questions)
7. [Best Practices](#best-practices)
8. [Performance Tuning](#performance-tuning)

---

## Thread Fundamentals

### Thread Lifecycle and States
```java
/**
 * ThreadLifecycleDemo - Demonstrates the complete lifecycle of a Java thread
 * 
 * Purpose: Shows how threads transition between different states (NEW, RUNNABLE, 
 * WAITING, TIMED_WAITING, BLOCKED, TERMINATED) and how to observe these transitions.
 * 
 * Key Learning: Understanding thread states is crucial for debugging multithreaded
 * applications and optimizing thread pool behavior.
 */
public class ThreadLifecycleDemo {
    
    /**
     * demonstrateThreadStates() - Creates a thread and shows its state transitions
     * 
     * What it does:
     * 1. Creates a thread in NEW state
     * 2. Starts the thread (moves to RUNNABLE)
     * 3. Thread goes to TIMED_WAITING during sleep()
     * 4. Thread could go to WAITING if wait() is called
     * 5. Finally moves to TERMINATED when thread completes
     */
    public void demonstrateThreadStates() {
        Thread thread = new Thread(() -> {
            try {
                // RUNNABLE state
                System.out.println("Thread is running");
                
                // TIMED_WAITING state
                Thread.sleep(2000);
                
                // WAITING state (if another thread calls join())
                synchronized (this) {
                    wait(); // Would cause WAITING state
                }
                
            } catch (InterruptedException e) {
                // TERMINATED state when exception occurs
                Thread.currentThread().interrupt();
            }
        });
        
        // NEW state
        System.out.println("Thread state: " + thread.getState()); // NEW
        
        thread.start();
        // RUNNABLE state
        System.out.println("Thread state: " + thread.getState()); // RUNNABLE
        
        try {
            thread.join();
            // TERMINATED state
            System.out.println("Thread state: " + thread.getState()); // TERMINATED
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Thread Synchronization Mechanisms
```java
/**
 * SynchronizationDemo - Comprehensive examples of Java synchronization mechanisms
 * 
 * Purpose: Shows different ways to coordinate access to shared resources between
 * multiple threads to prevent race conditions and ensure thread safety.
 * 
 * Covers: synchronized methods/blocks, volatile variables, atomic operations,
 * ReentrantLock, and ReadWriteLock patterns.
 */
public class SynchronizationDemo {
    private final Object lock = new Object();
    private volatile boolean flag = false;
    private final AtomicInteger counter = new AtomicInteger(0);
    
    /**
     * synchronizedMethod() - Method-level synchronization
     * 
     * What it does: Only one thread can execute this method at a time per object instance
     * Use case: When entire method needs to be atomic
     * Performance: Can cause contention if method is long-running
     */
    public synchronized void synchronizedMethod() {
        // Only one thread can execute this at a time
        counter.incrementAndGet();
    }
    
    /**
     * synchronizedBlock() - Block-level synchronization with custom lock
     * 
     * What it does: Only synchronizes the critical section, not entire method
     * Use case: When only part of method needs synchronization
     * Advantage: Better performance than method-level sync, custom lock object
     */
    public void synchronizedBlock() {
        synchronized (lock) {
            // Critical section - only one thread can execute this block
            flag = !flag;
        }
    }
    
    /**
     * volatile variable - Ensures visibility across threads
     * 
     * What it does: Guarantees that reads/writes are directly from/to main memory
     * Use case: Simple flags, counters that don't need atomic operations
     * Important: Only ensures visibility, NOT atomicity of compound operations
     */
    public volatile int volatileCounter = 0;
    
    /**
     * atomicOperations() - Lock-free atomic operations
     * 
     * What it does: Performs thread-safe operations without explicit locking
     * Use case: Counters, flags, simple numeric operations
     * Advantage: Better performance than synchronized for simple operations
     */
    public void atomicOperations() {
        counter.compareAndSet(0, 1);  // Atomic compare-and-swap
        counter.addAndGet(5);         // Atomic addition
    }
    
    /**
     * ReentrantLock - Explicit locking with advanced features
     * 
     * What it does: Provides more flexibility than synchronized blocks
     * Features: timeout attempts, interruptible locking, condition variables
     * Use case: When you need more control over locking behavior
     */
    private final ReentrantLock reentrantLock = new ReentrantLock();
    
    /**
     * reentrantLockExample() - Shows proper ReentrantLock usage pattern
     * 
     * Pattern: Always use try-finally to ensure lock is released
     * Advantage: Can attempt lock with timeout, can be interrupted
     */
    public void reentrantLockExample() {
        reentrantLock.lock();
        try {
            // Critical section
            performCriticalOperation();
        } finally {
            reentrantLock.unlock();  // Always unlock in finally block
        }
    }
    
    /**
     * ReadWriteLock - Separate locks for read and write operations
     * 
     * What it does: Allows multiple readers OR single writer
     * Use case: Data structures with frequent reads, infrequent writes
     * Advantage: Better throughput for read-heavy workloads
     */
    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    
    /**
     * readOperation() - Multiple threads can read simultaneously
     * 
     * What it does: Acquires read lock, allowing concurrent reads
     * Performance: Much better than exclusive locking for read operations
     */
    public void readOperation() {
        readWriteLock.readLock().lock();
        try {
            // Multiple threads can read simultaneously
            readData();
        } finally {
            readWriteLock.readLock().unlock();
        }
    }
    
    /**
     * writeOperation() - Exclusive write access
     * 
     * What it does: Acquires write lock, blocking all other readers/writers
     * Behavior: Waits for all readers to finish before acquiring write lock
     */
    public void writeOperation() {
        readWriteLock.writeLock().lock();
        try {
            // Only one thread can write at a time
            writeData();
        } finally {
            readWriteLock.writeLock().unlock();
        }
    }
}
```

---

## Executor Framework

### Types of Executors
```java
/**
 * ExecutorTypes - Demonstrates different types of thread pools and their use cases
 * 
 * Purpose: Shows when and how to use different executor types for various scenarios
 * Key Concept: Choosing the right executor type is crucial for application performance
 * 
 * Executor Types Covered:
 * 1. SingleThreadExecutor - Sequential task execution
 * 2. FixedThreadPool - Fixed number of worker threads
 * 3. CachedThreadPool - Dynamic thread creation/reuse
 * 4. ScheduledThreadPool - Time-based task execution
 * 5. WorkStealingPool - Fork-join based parallel processing
 */
public class ExecutorTypes {
    
    /**
     * singleThreadExecutorDemo() - Sequential task execution with single worker thread
     * 
     * What it does: Creates executor with exactly one worker thread
     * Use case: When tasks must be executed in order, or for simple background processing
     * Advantage: Guarantees task execution order, simple thread management
     * Disadvantage: No parallelism, can become bottleneck
     */
    public void singleThreadExecutorDemo() {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        try {
            // Tasks executed sequentially by single thread
            for (int i = 0; i < 5; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    System.out.println("Task " + taskId + " executed by " + 
                                     Thread.currentThread().getName());
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
        } finally {
            shutdownExecutorGracefully(executor);
        }
    }
    
    /**
     * fixedThreadPoolDemo() - Fixed number of worker threads
     * 
     * What it does: Creates executor with fixed number of threads (3 in this example)
     * Use case: When you know the optimal number of threads for your workload
     * Advantage: Predictable resource usage, good for CPU-bound tasks
     * Thread behavior: Threads are reused, excess tasks wait in queue
     */
    public void fixedThreadPoolDemo() {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            // Fixed number of threads handle all tasks
            for (int i = 0; i < 10; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    System.out.println("Task " + taskId + " executed by " + 
                                     Thread.currentThread().getName());
                    simulateWork();
                });
            }
        } finally {
            shutdownExecutorGracefully(executor);
        }
    }
    
    /**
     * cachedThreadPoolDemo() - Dynamic thread creation and reuse
     * 
     * What it does: Creates new threads as needed, reuses idle threads
     * Use case: For I/O-bound tasks with unpredictable workload patterns
     * Advantage: Automatically scales with demand, reuses threads efficiently
     * Risk: Can create unlimited threads if tasks arrive faster than completion
     */
    public void cachedThreadPoolDemo() {
        ExecutorService executor = Executors.newCachedThreadPool();
        
        try {
            // Creates new threads as needed, reuses idle threads
            for (int i = 0; i < 20; i++) {
                final int taskId = i;
                executor.submit(() -> {
                    System.out.println("Task " + taskId + " executed by " + 
                                     Thread.currentThread().getName());
                    simulateWork();
                });
                
                // Small delay between submissions
                Thread.sleep(50);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            shutdownExecutorGracefully(executor);
        }
    }
    
    /**
     * scheduledThreadPoolDemo() - Time-based task scheduling
     * 
     * What it does: Executes tasks at scheduled times or with fixed delays/rates
     * Use case: Periodic tasks, delayed execution, cron-like functionality
     * Methods:
     * - schedule(): One-time delayed execution
     * - scheduleAtFixedRate(): Fixed rate execution (starts every X time units)
     * - scheduleWithFixedDelay(): Fixed delay between task completions
     */
    public void scheduledThreadPoolDemo() {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
        
        try {
            // Schedule one-time task - executes once after specified delay
            scheduler.schedule(() -> {
                System.out.println("One-time task executed after 2 seconds");
            }, 2, TimeUnit.SECONDS);
            
            // Schedule repeating task - executes every 3 seconds regardless of execution time
            // If task takes longer than period, next execution may start immediately
            scheduler.scheduleAtFixedRate(() -> {
                System.out.println("Repeating task: " + new Date());
            }, 1, 3, TimeUnit.SECONDS);
            
            // Schedule with fixed delay - waits for task completion before starting delay
            // Always waits 2 seconds after task completes before next execution
            scheduler.scheduleWithFixedDelay(() -> {
                System.out.println("Fixed delay task: " + new Date());
                simulateWork(); // Delay starts after task completion
            }, 1, 2, TimeUnit.SECONDS);
            
            // Let it run for a while
            Thread.sleep(15000);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            shutdownExecutorGracefully(scheduler);
        }
    }
    
    /**
     * workStealingPoolDemo() - Fork-join based thread pool with work stealing
     * 
     * What it does: Creates a pool where idle threads can "steal" work from busy threads
     * Use case: CPU-intensive tasks that can be parallelized, recursive algorithms
     * Advantage: Better load balancing, efficient for divide-and-conquer algorithms
     * Based on: ForkJoinPool implementation with work-stealing algorithm
     */
    public void workStealingPoolDemo() {
        ExecutorService executor = Executors.newWorkStealingPool();
        
        try {
            List<Future<Integer>> futures = new ArrayList<>();
            
            // Submit tasks with varying workloads
            for (int i = 0; i < 20; i++) {
                final int taskId = i;
                Future<Integer> future = executor.submit(() -> {
                    // Simulate varying work loads
                    int workLoad = (taskId % 3 + 1) * 1000;
                    Thread.sleep(workLoad);
                    return taskId * 2;
                });
                futures.add(future);
            }
            
            // Collect results
            for (Future<Integer> future : futures) {
                System.out.println("Result: " + future.get());
            }
            
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        } finally {
            shutdownExecutorGracefully(executor);
        }
    }
    
    /**
     * shutdownExecutorGracefully() - Proper executor shutdown pattern
     * 
     * What it does: Safely shuts down executor service with timeout handling
     * Process:
     * 1. shutdown() - stops accepting new tasks
     * 2. awaitTermination() - waits for existing tasks to complete
     * 3. shutdownNow() - forcefully interrupts running tasks if timeout exceeded
     * 
     * Why important: Prevents resource leaks and ensures clean application shutdown
     */
    private void shutdownExecutorGracefully(ExecutorService executor) {
        executor.shutdown();  // Disable new tasks from being submitted
        try {
            // Wait for existing tasks to terminate
            if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                executor.shutdownNow();  // Cancel currently executing tasks
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();  // Re-cancel if current thread also interrupted
            Thread.currentThread().interrupt();  // Preserve interrupt status
        }
    }
    
    /**
     * simulateWork() - Simulates variable-duration work
     * 
     * What it does: Sleeps for random duration between 0.5-1.5 seconds
     * Use case: Testing thread pool behavior with realistic work patterns
     * Pattern: Always handle InterruptedException properly in utility methods
     */
    private void simulateWork() {
        try {
            Thread.sleep(500 + (int)(Math.random() * 1000));
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();  // Restore interrupt status
        }
    }
}
```

### Future and Callable
```java
/**
 * FutureCallableDemo - Demonstrates asynchronous task execution with results
 * 
 * Purpose: Shows how to execute tasks asynchronously and retrieve results later
 * Key Concepts:
 * - Callable: Task that returns a result (vs Runnable which returns void)
 * - Future: Handle to retrieve result of asynchronous computation
 * - Blocking vs non-blocking result retrieval
 * 
 * Use Cases: Long-running computations, I/O operations, parallel processing
 */
public class FutureCallableDemo {
    
    /**
     * basicFutureDemo() - Basic pattern for asynchronous task with result
     * 
     * What it demonstrates:
     * 1. Submit Callable task to executor
     * 2. Continue other work while task executes
     * 3. Retrieve result when needed (blocking call)
     * 
     * Pattern: Submit -> Do other work -> Get result
     */
    public void basicFutureDemo() {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            // Submit Callable task
            Future<String> future = executor.submit(new Callable<String>() {
                @Override
                public String call() throws Exception {
                    Thread.sleep(2000);
                    return "Task completed by " + Thread.currentThread().getName();
                }
            });
            
            // Do other work while task executes
            System.out.println("Task submitted, doing other work...");
            doOtherWork();
            
            // Get result (blocks until completion)
            String result = future.get();
            System.out.println("Result: " + result);
            
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
    
    /**
     * futureWithTimeoutDemo() - Handling timeouts in asynchronous operations
     * 
     * What it demonstrates:
     * 1. Setting maximum wait time for task completion
     * 2. Cancelling tasks that exceed timeout
     * 3. Handling TimeoutException gracefully
     * 
     * Use case: Preventing application from hanging on slow operations
     * Important: Always specify timeouts for external service calls
     */
    public void futureWithTimeoutDemo() {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        try {
            Future<String> future = executor.submit(() -> {
                Thread.sleep(5000); // Long running task
                return "Long task completed";
            });
            
            try {
                // Wait maximum 3 seconds - throws TimeoutException if task takes longer
                String result = future.get(3, TimeUnit.SECONDS);
                System.out.println("Result: " + result);
            } catch (TimeoutException e) {
                System.out.println("Task timed out, cancelling...");
                // cancel(true) attempts to interrupt the running task
                // cancel(false) only cancels if task hasn't started yet
                future.cancel(true); // Interrupt if running
            }
            
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
    
    // Multiple Futures with invokeAll
    public void multipleFuturesDemo() {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            List<Callable<Integer>> tasks = Arrays.asList(
                () -> { Thread.sleep(1000); return 1; },
                () -> { Thread.sleep(2000); return 2; },
                () -> { Thread.sleep(1500); return 3; }
            );
            
            // Submit all tasks and wait for completion
            List<Future<Integer>> futures = executor.invokeAll(tasks);
            
            // Collect results
            for (Future<Integer> future : futures) {
                System.out.println("Result: " + future.get());
            }
            
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
    
    // invokeAny - returns first completed result
    public void invokeAnyDemo() {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            List<Callable<String>> tasks = Arrays.asList(
                () -> { Thread.sleep(3000); return "Task 1"; },
                () -> { Thread.sleep(1000); return "Task 2"; }, // This will complete first
                () -> { Thread.sleep(2000); return "Task 3"; }
            );
            
            // Returns result of first completed task
            String result = executor.invokeAny(tasks);
            System.out.println("First completed: " + result);
            
        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
    
    private void doOtherWork() {
        try {
            Thread.sleep(1000);
            System.out.println("Other work completed");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

---

## Custom Thread Pools

### ThreadPoolExecutor Configuration
```java
public class CustomThreadPools {
    
    // Basic custom thread pool
    public ThreadPoolExecutor createBasicThreadPool() {
        return new ThreadPoolExecutor(
            2,                              // corePoolSize
            4,                              // maximumPoolSize
            60L,                            // keepAliveTime
            TimeUnit.SECONDS,              // time unit
            new LinkedBlockingQueue<>(10), // work queue
            new ThreadFactory() {          // thread factory
                private final AtomicInteger threadNumber = new AtomicInteger(1);
                @Override
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(r, "CustomPool-" + threadNumber.getAndIncrement());
                    t.setDaemon(false);
                    return t;
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
        );
    }
    
    // Production-ready thread pool with monitoring
    public class MonitoredThreadPool {
        private final ThreadPoolExecutor executor;
        private final ScheduledExecutorService monitor;
        
        public MonitoredThreadPool(String poolName, int coreSize, int maxSize, int queueSize) {
            this.executor = new ThreadPoolExecutor(
                coreSize,
                maxSize,
                60L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(queueSize),
                new NamedThreadFactory(poolName),
                new LoggingRejectionHandler()
            );
            
            // Enable core thread timeout
            executor.allowCoreThreadTimeOut(true);
            
            // Start monitoring
            this.monitor = Executors.newSingleThreadScheduledExecutor(
                new NamedThreadFactory(poolName + "-monitor"));
            startMonitoring();
        }
        
        private void startMonitoring() {
            monitor.scheduleAtFixedRate(() -> {
                System.out.printf(
                    "Pool Stats - Active: %d, Pool Size: %d, Queue Size: %d, " +
                    "Completed: %d, Total Tasks: %d%n",
                    executor.getActiveCount(),
                    executor.getPoolSize(),
                    executor.getQueue().size(),
                    executor.getCompletedTaskCount(),
                    executor.getTaskCount()
                );
            }, 5, 5, TimeUnit.SECONDS);
        }
        
        public Future<?> submit(Runnable task) {
            return executor.submit(task);
        }
        
        public <T> Future<T> submit(Callable<T> task) {
            return executor.submit(task);
        }
        
        public void shutdown() {
            executor.shutdown();
            monitor.shutdown();
        }
    }
    
    // Named thread factory
    private static class NamedThreadFactory implements ThreadFactory {
        private final String namePrefix;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        
        public NamedThreadFactory(String namePrefix) {
            this.namePrefix = namePrefix;
        }
        
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, namePrefix + "-" + threadNumber.getAndIncrement());
            t.setDaemon(false);
            t.setPriority(Thread.NORM_PRIORITY);
            
            // Set uncaught exception handler
            t.setUncaughtExceptionHandler((thread, exception) -> {
                System.err.println("Uncaught exception in thread " + thread.getName());
                exception.printStackTrace();
            });
            
            return t;
        }
    }
    
    // Custom rejection handler
    private static class LoggingRejectionHandler implements RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            System.err.println("Task rejected: " + r.toString() + 
                             " - Pool: " + executor.toString());
            
            // Alternative strategies:
            // 1. Log and discard
            // 2. Run in caller thread (CallerRunsPolicy)
            // 3. Throw exception (AbortPolicy - default)
            // 4. Discard oldest (DiscardOldestPolicy)
            
            // Custom strategy: Try to queue again after short delay
            try {
                Thread.sleep(100);
                if (!executor.isShutdown()) {
                    executor.submit(r);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

### Specialized Thread Pools
```java
public class SpecializedThreadPools {
    
    // CPU-intensive tasks pool
    public static class CPUIntensivePool {
        private final ExecutorService executor;
        
        public CPUIntensivePool() {
            int processors = Runtime.getRuntime().availableProcessors();
            this.executor = new ThreadPoolExecutor(
                processors,                    // Core size = CPU count
                processors,                    // Max size = CPU count
                0L,                           // Keep alive time
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(100),
                new NamedThreadFactory("CPU-Intensive"),
                new ThreadPoolExecutor.CallerRunsPolicy()
            );
        }
        
        public <T> Future<T> submitCPUTask(Callable<T> task) {
            return executor.submit(task);
        }
        
        public void shutdown() {
            executor.shutdown();
        }
    }
    
    // I/O-intensive tasks pool
    public static class IOIntensivePool {
        private final ExecutorService executor;
        
        public IOIntensivePool() {
            int processors = Runtime.getRuntime().availableProcessors();
            this.executor = new ThreadPoolExecutor(
                processors * 2,               // Core size = 2 * CPU count
                processors * 4,               // Max size = 4 * CPU count
                60L,                          // Keep alive time
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(200),
                new NamedThreadFactory("IO-Intensive"),
                new ThreadPoolExecutor.CallerRunsPolicy()
            );
        }
        
        public <T> Future<T> submitIOTask(Callable<T> task) {
            return executor.submit(task);
        }
        
        public void shutdown() {
            executor.shutdown();
        }
    }
    
    // Priority-based thread pool
    public static class PriorityThreadPool {
        private final ThreadPoolExecutor executor;
        
        public PriorityThreadPool() {
            this.executor = new ThreadPoolExecutor(
                2, 4, 60L, TimeUnit.SECONDS,
                new PriorityBlockingQueue<>(100, new TaskComparator()),
                new NamedThreadFactory("Priority-Pool"),
                new ThreadPoolExecutor.CallerRunsPolicy()
            );
        }
        
        public Future<?> submitTask(Runnable task, int priority) {
            return executor.submit(new PriorityTask(task, priority));
        }
        
        private static class PriorityTask implements Runnable, Comparable<PriorityTask> {
            private final Runnable task;
            private final int priority;
            
            public PriorityTask(Runnable task, int priority) {
                this.task = task;
                this.priority = priority;
            }
            
            @Override
            public void run() {
                task.run();
            }
            
            @Override
            public int compareTo(PriorityTask other) {
                return Integer.compare(other.priority, this.priority); // Higher priority first
            }
        }
        
        private static class TaskComparator implements Comparator<Runnable> {
            @Override
            public int compare(Runnable r1, Runnable r2) {
                if (r1 instanceof PriorityTask && r2 instanceof PriorityTask) {
                    return ((PriorityTask) r1).compareTo((PriorityTask) r2);
                }
                return 0;
            }
        }
        
        public void shutdown() {
            executor.shutdown();
        }
    }
}
```

---

## CompletableFuture

### Basic CompletableFuture Operations
```java
/**
 * CompletableFutureBasics - Modern asynchronous programming with CompletableFuture
 * 
 * Purpose: Demonstrates advanced asynchronous programming patterns introduced in Java 8
 * Advantages over Future:
 * 1. Non-blocking operations with callbacks
 * 2. Composable and chainable operations
 * 3. Better error handling and recovery
 * 4. Support for combining multiple async operations
 * 
 * Key Concepts: Completion stages, transformations, compositions, error handling
 */
public class CompletableFutureBasics {
    
    /**
     * creationMethods() - Different ways to create CompletableFuture instances
     * 
     * Methods demonstrated:
     * 1. completedFuture() - Already completed with a value
     * 2. supplyAsync() - Asynchronous computation with result
     * 3. supplyAsync(supplier, executor) - Custom executor
     * 4. Manual completion - Complete programmatically
     * 5. runAsync() - Asynchronous execution without result
     */
    public void creationMethods() {
        // 1. Already completed future
        CompletableFuture<String> completed = CompletableFuture.completedFuture("Hello");
        
        // 2. Asynchronous computation
        CompletableFuture<String> async = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
                return "Async result";
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                throw new RuntimeException(e);
            }
        });
        
        // 3. Asynchronous execution with custom executor
        ExecutorService executor = Executors.newFixedThreadPool(3);
        CompletableFuture<String> asyncWithExecutor = CompletableFuture.supplyAsync(() -> {
            return "Custom executor result";
        }, executor);
        
        // 4. Manual completion
        CompletableFuture<String> manual = new CompletableFuture<>();
        // Later in code: manual.complete("Manual result");
        
        // 5. Runnable-based (no return value)
        CompletableFuture<Void> runAsync = CompletableFuture.runAsync(() -> {
            System.out.println("Async task executed");
        });
    }
    
    /**
     * chainingOperations() - Demonstrates CompletableFuture chaining and composition
     * 
     * Operations explained:
     * 1. thenApply() - Transform result (like map() in streams)
     * 2. thenCompose() - Chain another CompletableFuture (like flatMap())
     * 3. thenAccept() - Consume result without returning value
     * 
     * Advantage: Builds processing pipeline without blocking intermediate steps
     */
    public void chainingOperations() {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
            .thenApply(String::toUpperCase)        // Transform: "Hello" -> "HELLO"
            .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World!"))  // Chain another async operation
            .thenApply(s -> s + " from CompletableFuture");  // Final transformation
        
        future.thenAccept(System.out::println);  // Consume final result
    }
    
    /**
     * errorHandling() - Exception handling in asynchronous operations
     * 
     * Methods demonstrated:
     * 1. exceptionally() - Provides fallback value on exception
     * 2. handle() - Process both success and error cases
     * 3. whenComplete() - Perform action on completion (success or failure)
     * 
     * Key concept: Exceptions in async operations must be handled explicitly
     */
    public void errorHandling() {
        CompletableFuture<String> futureWithError = CompletableFuture.supplyAsync(() -> {
            throw new RuntimeException("Something went wrong!");
        }).exceptionally(throwable -> "Fallback value");  // Provides default value on exception
        
        futureWithError.thenAccept(System.out::println);  // Will print "Fallback value"
    }
}
```

---

## ForkJoin Framework

### Understanding ForkJoin
```java
/**
 * ArraySumTask - Demonstrates divide-and-conquer with ForkJoin framework
 * 
 * Purpose: Shows how to implement parallel algorithms using work-stealing approach
 * Key Concepts:
 * 1. RecursiveTask - For tasks that return a result
 * 2. Work stealing - Idle threads steal work from busy threads
 * 3. Divide and conquer - Break large problems into smaller ones
 * 
 * When to use: CPU-intensive tasks that can be parallelized (sorting, searching, mathematical computations)
 * Performance: Best with recursive algorithms that can split work evenly
 */
public class ArraySumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10000;
    private final int[] array;
    private final int start;
    private final int end;
    
    public ArraySumTask(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }
    
    /**
     * compute() - Core ForkJoin algorithm implementation
     * 
     * Algorithm steps:
     * 1. Check if task is small enough for direct computation (base case)
     * 2. If too large, split into two subtasks (divide)
     * 3. Fork one subtask to run in parallel
     * 4. Compute the other subtask in current thread
     * 5. Join results from both subtasks (conquer)
     * 
     * Key pattern: fork() one task, compute() the other, then join()
     */
    @Override
    protected Long compute() {
        int length = end - start;
        
        // Base case: small enough to compute directly
        // THRESHOLD prevents excessive task splitting overhead
        if (length <= THRESHOLD) {
            return computeDirectly();
        }
        
        // Divide: split the task into two halves
        int mid = start + length / 2;
        ArraySumTask leftTask = new ArraySumTask(array, start, mid);
        ArraySumTask rightTask = new ArraySumTask(array, mid, end);
        
        // Conquer: fork the left task (runs in parallel)
        leftTask.fork();  // Adds to work-stealing queue
        
        // Compute right task in current thread (work stealing optimization)
        Long rightResult = rightTask.compute();
        
        // Join left task result (blocks until completion)
        Long leftResult = leftTask.join();
        
        // Combine results
        return leftResult + rightResult;
    }
    
    /**
     * computeDirectly() - Sequential computation for small tasks
     * 
     * What it does: Simple loop-based summation when task size is below threshold
     * Why separate method: Keeps the recursive logic clean and testable
     */
    private Long computeDirectly() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += array[i];
        }
        return sum;
    }
}
```

---

## Scenario-Based Questions

### Scenario 1: E-commerce Order Processing
```java
/**
 * EcommerceOrderProcessor - Real-world example of complex asynchronous processing
 * 
 * Business Problem: Process e-commerce orders efficiently with multiple validation steps
 * 
 * Architecture decisions:
 * 1. Separate thread pools for different operation types (payment, inventory, shipping, email)
 * 2. Pipeline processing with CompletableFuture composition
 * 3. Parallel execution where operations are independent
 * 4. Graceful error handling and fallback strategies
 * 
 * Performance requirements:
 * - Handle 1000+ orders per minute
 * - Sub-second response times
 * - Fail fast on validation errors
 * - Asynchronous email sending (don't block order confirmation)
 */
public class EcommerceOrderProcessor {
    /*
     * SCENARIO: Design a multithreaded system for processing e-commerce orders:
     * - Validate payment (100ms)
     * - Check inventory (150ms) 
     * - Calculate shipping (50ms)
     * - Send confirmation email (200ms)
     * 
     * REQUIREMENTS:
     * - Process 1000+ orders per minute
     * - Maintain order of operations where necessary
     * - Handle failures gracefully
     */
    
    private final ExecutorService paymentPool = Executors.newFixedThreadPool(10);
    private final ExecutorService inventoryPool = Executors.newFixedThreadPool(8);
    private final ExecutorService shippingPool = Executors.newFixedThreadPool(5);
    private final ExecutorService emailPool = Executors.newFixedThreadPool(3);
    
    /**
     * processOrder() - Main order processing pipeline
     * 
     * Processing flow:
     * 1. Payment validation (blocking step - must succeed to continue)
     * 2. Parallel inventory check and shipping calculation (independent operations)
     * 3. Combine results and validate availability
     * 4. Send confirmation email asynchronously (fire-and-forget)
     * 
     * Design patterns used:
     * - Pipeline processing with CompletableFuture.thenCompose()
     * - Parallel execution with thenCombine()
     * - Fire-and-forget with runAsync()
     * - Fail-fast validation with early returns
     */
    public CompletableFuture<OrderResult> processOrder(Order order) {
        return CompletableFuture
            // Step 1: Validate payment (must complete before proceeding)
            .supplyAsync(() -> validatePayment(order), paymentPool)
            
            // Step 2: Parallel inventory check and shipping calculation
            .thenCompose(paymentResult -> {
                // Fail fast if payment validation fails
                if (!paymentResult.isValid()) {
                    return CompletableFuture.completedFuture(
                        OrderResult.failed("Payment validation failed"));
                }
                
                // Execute inventory check and shipping calculation in parallel
                // These operations are independent and can run simultaneously
                CompletableFuture<InventoryResult> inventoryCheck = 
                    CompletableFuture.supplyAsync(() -> checkInventory(order), inventoryPool);
                
                CompletableFuture<ShippingResult> shippingCalc = 
                    CompletableFuture.supplyAsync(() -> calculateShipping(order), shippingPool);
                
                // Combine results when both operations complete
                return inventoryCheck.thenCombine(shippingCalc, 
                    (inventory, shipping) -> {
                        if (!inventory.isAvailable()) {
                            return OrderResult.failed("Item not available");
                        }
                        return OrderResult.success(paymentResult, inventory, shipping);
                    });
            })
            
            // Step 3: Send confirmation email asynchronously (don't wait for completion)
            .thenCompose(orderResult -> {
                if (orderResult.isSuccess()) {
                    // Fire-and-forget email sending - doesn't block order confirmation
                    CompletableFuture.runAsync(() -> sendConfirmationEmail(order), emailPool);
                }
                return CompletableFuture.completedFuture(orderResult);
            })
            
            // Centralized error handling for the entire pipeline
            .exceptionally(throwable -> {
                System.err.println("Order processing failed for order: " + order.getId());
                throwable.printStackTrace();
                return OrderResult.failed("Processing error: " + throwable.getMessage());
            });
    }
    
    private PaymentResult validatePayment(Order order) {
        try { Thread.sleep(100); } catch (InterruptedException e) { 
            Thread.currentThread().interrupt(); 
        }
        return new PaymentResult(true);
    }
    
    private InventoryResult checkInventory(Order order) {
        try { Thread.sleep(150); } catch (InterruptedException e) { 
            Thread.currentThread().interrupt(); 
        }
        return new InventoryResult(true);
    }
    
    private ShippingResult calculateShipping(Order order) {
        try { Thread.sleep(50); } catch (InterruptedException e) { 
            Thread.currentThread().interrupt(); 
        }
        return new ShippingResult(9.99);
    }
    
    private void sendConfirmationEmail(Order order) {
        try { Thread.sleep(200); } catch (InterruptedException e) { 
            Thread.currentThread().interrupt(); 
        }
        System.out.println("Confirmation email sent for order: " + order.getId());
    }
    
    // Supporting classes
    static class Order {
        private String id;
        public String getId() { return id; }
    }
    
    static class PaymentResult {
        private boolean valid;
        public PaymentResult(boolean valid) { this.valid = valid; }
        public boolean isValid() { return valid; }
    }
    
    static class InventoryResult {
        private boolean available;
        public InventoryResult(boolean available) { this.available = available; }
        public boolean isAvailable() { return available; }
    }
    
    static class ShippingResult {
        private double cost;
        public ShippingResult(double cost) { this.cost = cost; }
        public double getCost() { return cost; }
    }
    
    static class OrderResult {
        private boolean success;
        private String message;
        
        private OrderResult(boolean success, String message) {
            this.success = success;
            this.message = message;
        }
        
        public static OrderResult success(PaymentResult payment, InventoryResult inventory, ShippingResult shipping) {
            return new OrderResult(true, "Order processed successfully");
        }
        
        public static OrderResult failed(String message) {
            return new OrderResult(false, message);
        }
        
        public boolean isSuccess() { return success; }
        public String getMessage() { return message; }
    }
}
```

---

## Best Practices

### Thread Pool Best Practices
```java
public class ThreadPoolBestPractices {
    
    // 1. Proper thread pool sizing
    public ExecutorService createOptimalThreadPool(String poolName, TaskType taskType) {
        int processors = Runtime.getRuntime().availableProcessors();
        
        switch (taskType) {
            case CPU_INTENSIVE:
                // CPU-bound tasks: pool size = number of processors
                return new ThreadPoolExecutor(
                    processors, processors, 0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<>(100),
                    new NamedThreadFactory(poolName + "-cpu"),
                    new ThreadPoolExecutor.CallerRunsPolicy()
                );
                
            case IO_INTENSIVE:
                // I/O-bound tasks: pool size = 2 * number of processors
                return new ThreadPoolExecutor(
                    processors * 2, processors * 4, 60L, TimeUnit.SECONDS,
                    new LinkedBlockingQueue<>(200),
                    new NamedThreadFactory(poolName + "-io"),
                    new ThreadPoolExecutor.CallerRunsPolicy()
                );
                
            case MIXED:
                // Mixed workload: balanced configuration
                return new ThreadPoolExecutor(
                    processors, processors * 2, 60L, TimeUnit.SECONDS,
                    new LinkedBlockingQueue<>(150),
                    new NamedThreadFactory(poolName + "-mixed"),
                    new ThreadPoolExecutor.CallerRunsPolicy()
                );
                
            default:
                throw new IllegalArgumentException("Unknown task type: " + taskType);
        }
    }
    
    // 2. Graceful shutdown
    public void shutdownGracefully(ExecutorService executor, long timeout, TimeUnit unit) {
        executor.shutdown(); // Disable new tasks from being submitted
        
        try {
            // Wait for existing tasks to terminate
            if (!executor.awaitTermination(timeout, unit)) {
                executor.shutdownNow(); // Cancel currently executing tasks
                
                // Wait for tasks to respond to being cancelled
                if (!executor.awaitTermination(timeout, unit)) {
                    System.err.println("Executor did not terminate gracefully");
                }
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
    
    // 3. Thread factory with proper naming and exception handling
    public static class NamedThreadFactory implements ThreadFactory {
        private final String namePrefix;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final boolean daemon;
        
        public NamedThreadFactory(String namePrefix, boolean daemon) {
            this.namePrefix = namePrefix;
            this.daemon = daemon;
        }
        
        public NamedThreadFactory(String namePrefix) {
            this(namePrefix, false);
        }
        
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, namePrefix + "-" + threadNumber.getAndIncrement());
            t.setDaemon(daemon);
            t.setPriority(Thread.NORM_PRIORITY);
            
            t.setUncaughtExceptionHandler((thread, exception) -> {
                System.err.println("Uncaught exception in thread " + thread.getName());
                exception.printStackTrace();
                // Could also log to monitoring system
            });
            
            return t;
        }
    }
    
    enum TaskType {
        CPU_INTENSIVE, IO_INTENSIVE, MIXED
    }
}
```

---

## Performance Tuning

### Key Performance Considerations
```java
public class PerformanceTuning {
    
    // 1. Optimal queue sizing
    public ThreadPoolExecutor createTunedThreadPool() {
        int processors = Runtime.getRuntime().availableProcessors();
        
        // Queue size calculation:
        // Too small: frequent rejections
        // Too large: high memory usage, poor GC performance
        int optimalQueueSize = processors * 25; // Rule of thumb
        
        return new ThreadPoolExecutor(
            processors,
            processors * 2,
            60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(optimalQueueSize),
            new NamedThreadFactory("tuned-pool"),
            new ThreadPoolExecutor.CallerRunsPolicy()
        );
    }
    
    // 2. Avoiding context switching overhead
    public void minimizeContextSwitching() {
        // Use fewer threads for CPU-intensive tasks
        int cpuThreads = Runtime.getRuntime().availableProcessors();
        ExecutorService cpuPool = Executors.newFixedThreadPool(cpuThreads);
        
        // Batch similar tasks together
        List<Callable<Integer>> cpuTasks = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            final int taskId = i;
            cpuTasks.add(() -> performCPUIntensiveWork(taskId));
        }
        
        try {
            List<Future<Integer>> results = cpuPool.invokeAll(cpuTasks);
            // Process results...
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            cpuPool.shutdown();
        }
    }
    
    // 3. Memory-efficient task processing
    public void memoryEfficientProcessing() {
        // Process large datasets in chunks to avoid OOM
        int chunkSize = 1000;
        List<Integer> largeDataset = generateLargeDataset();
        
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        for (int i = 0; i < largeDataset.size(); i += chunkSize) {
            int endIndex = Math.min(i + chunkSize, largeDataset.size());
            List<Integer> chunk = largeDataset.subList(i, endIndex);
            
            executor.submit(() -> processChunk(chunk));
        }
        
        executor.shutdown();
    }
    
    private Integer performCPUIntensiveWork(int taskId) {
        // Simulate CPU work
        long result = 0;
        for (int i = 0; i < 100000; i++) {
            result += Math.sqrt(i * taskId);
        }
        return (int) result;
    }
    
    private List<Integer> generateLargeDataset() {
        return IntStream.range(0, 1000000).boxed().collect(Collectors.toList());
    }
    
    private void processChunk(List<Integer> chunk) {
        // Process chunk of data
        chunk.forEach(this::processItem);
    }
    
    private void processItem(Integer item) {
        // Process individual item
    }
}
```

---

## Producer-Consumer Patterns

### Basic Producer-Consumer with BlockingQueue
```java
/**
 * BasicProducerConsumer - Classic producer-consumer pattern using BlockingQueue
 * 
 * Purpose: Demonstrates thread-safe communication between producer and consumer threads
 * Key Concepts:
 * 1. BlockingQueue - Thread-safe queue with blocking operations
 * 2. Producer threads add items to queue
 * 3. Consumer threads remove items from queue
 * 4. Automatic blocking when queue is full/empty
 * 
 * Use Cases: Data processing pipelines, event handling, work distribution
 */
public class BasicProducerConsumer {
    
    // Shared queue between producers and consumers
    private final BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>(100);
    private volatile boolean running = true;
    
    /**
     * Producer - Generates tasks and adds them to the queue
     * 
     * What it does:
     * 1. Creates tasks continuously
     * 2. Puts tasks into blocking queue (blocks if queue is full)
     * 3. Handles interruption gracefully
     * 
     * Pattern: put() blocks when queue is full, preventing memory overflow
     */
    public class Producer implements Runnable {
        private final String producerName;
        private final AtomicInteger taskCounter = new AtomicInteger(0);
        
        public Producer(String name) {
            this.producerName = name;
        }
        
        @Override
        public void run() {
            try {
                while (running && !Thread.currentThread().isInterrupted()) {
                    // Create a new task
                    Task task = new Task(
                        producerName + "-Task-" + taskCounter.incrementAndGet(),
                        "Data from " + producerName
                    );
                    
                    // Put task into queue (blocks if queue is full)
                    taskQueue.put(task);
                    System.out.println(producerName + " produced: " + task.getId());
                    
                    // Simulate production time
                    Thread.sleep(100 + (int)(Math.random() * 200));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println(producerName + " interrupted");
            }
        }
    }
    
    /**
     * Consumer - Processes tasks from the queue
     * 
     * What it does:
     * 1. Takes tasks from blocking queue (blocks if queue is empty)
     * 2. Processes each task
     * 3. Handles interruption and shutdown gracefully
     * 
     * Pattern: take() blocks when queue is empty, eliminating busy waiting
     */
    public class Consumer implements Runnable {
        private final String consumerName;
        
        public Consumer(String name) {
            this.consumerName = name;
        }
        
        @Override
        public void run() {
            try {
                while (running && !Thread.currentThread().isInterrupted()) {
                    // Take task from queue (blocks if queue is empty)
                    Task task = taskQueue.take();
                    
                    // Process the task
                    processTask(task);
                    
                    System.out.println(consumerName + " consumed: " + task.getId());
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println(consumerName + " interrupted");
            }
        }
        
        private void processTask(Task task) throws InterruptedException {
            // Simulate task processing time
            Thread.sleep(50 + (int)(Math.random() * 100));
            // Actual task processing would go here
        }
    }
    
    /**
     * Task - Represents work item in the producer-consumer system
     * 
     * Purpose: Encapsulates data and metadata for processing
     * Thread Safety: Immutable design ensures thread safety
     */
    public static class Task {
        private final String id;
        private final String data;
        private final long timestamp;
        
        public Task(String id, String data) {
            this.id = id;
            this.data = data;
            this.timestamp = System.currentTimeMillis();
        }
        
        public String getId() { return id; }
        public String getData() { return data; }
        public long getTimestamp() { return timestamp; }
        
        @Override
        public String toString() {
            return "Task{id='" + id + "', data='" + data + "'}";
        }
    }
    
    /**
     * Demo method showing producer-consumer in action
     * 
     * Configuration:
     * - 2 producer threads
     * - 3 consumer threads
     * - Runs for 10 seconds then shuts down gracefully
     */
    public void runDemo() {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        try {
            // Start producers
            executor.submit(new Producer("Producer-1"));
            executor.submit(new Producer("Producer-2"));
            
            // Start consumers
            executor.submit(new Consumer("Consumer-1"));
            executor.submit(new Consumer("Consumer-2"));
            executor.submit(new Consumer("Consumer-3"));
            
            // Let it run for 10 seconds
            Thread.sleep(10000);
            
            // Graceful shutdown
            running = false;
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            shutdownGracefully(executor);
            System.out.println("Queue size at shutdown: " + taskQueue.size());
        }
    }
    
    private void shutdownGracefully(ExecutorService executor) {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
```

### Priority-Based Producer-Consumer
```java
/**
 * PriorityProducerConsumer - Producer-consumer with task prioritization
 * 
 * Purpose: Demonstrates priority-based task processing using PriorityBlockingQueue
 * Key Features:
 * 1. High-priority tasks processed before low-priority ones
 * 2. Thread-safe priority queue implementation
 * 3. Dynamic priority assignment
 * 4. Starvation prevention mechanisms
 * 
 * Use Cases: Critical system tasks, emergency processing, SLA-based processing
 */
public class PriorityProducerConsumer {
    
    // Priority queue that orders tasks by priority (high to low)
    private final PriorityBlockingQueue<PriorityTask> priorityQueue = 
        new PriorityBlockingQueue<>(100, new TaskPriorityComparator());
    
    private volatile boolean running = true;
    private final AtomicLong taskIdGenerator = new AtomicLong(0);
    
    /**
     * PriorityTask - Task with priority level and anti-starvation mechanism
     * 
     * Features:
     * 1. Priority levels (CRITICAL, HIGH, MEDIUM, LOW)
     * 2. Timestamp for preventing starvation
     * 3. Age calculation for priority boost
     * 4. Immutable design for thread safety
     */
    public static class PriorityTask {
        public enum Priority {
            CRITICAL(4), HIGH(3), MEDIUM(2), LOW(1);
            
            private final int value;
            Priority(int value) { this.value = value; }
            public int getValue() { return value; }
        }
        
        private final long id;
        private final String name;
        private final String data;
        private final Priority priority;
        private final long submissionTime;
        
        public PriorityTask(String name, String data, Priority priority) {
            this.id = System.nanoTime(); // Unique ID
            this.name = name;
            this.data = data;
            this.priority = priority;
            this.submissionTime = System.currentTimeMillis();
        }
        
        public long getId() { return id; }
        public String getName() { return name; }
        public String getData() { return data; }
        public Priority getPriority() { return priority; }
        public long getSubmissionTime() { return submissionTime; }
        
        /**
         * getEffectivePriority() - Calculates priority with age boost
         * 
         * Anti-starvation mechanism:
         * - Tasks waiting longer than 5 seconds get priority boost
         * - Prevents low-priority tasks from being starved indefinitely
         */
        public int getEffectivePriority() {
            long age = System.currentTimeMillis() - submissionTime;
            int basePriority = priority.getValue();
            
            // Boost priority for old tasks (anti-starvation)
            if (age > 5000) { // 5 seconds
                basePriority += 1;
            }
            if (age > 10000) { // 10 seconds
                basePriority += 2;
            }
            
            return basePriority;
        }
        
        @Override
        public String toString() {
            return String.format("PriorityTask{id=%d, name='%s', priority=%s, age=%dms}", 
                id, name, priority, System.currentTimeMillis() - submissionTime);
        }
    }
    
    /**
     * TaskPriorityComparator - Defines task ordering in priority queue
     * 
     * Ordering rules:
     * 1. Higher effective priority first
     * 2. If same priority, older tasks first (FIFO within priority)
     * 3. If same time, lower ID first (deterministic ordering)
     */
    public static class TaskPriorityComparator implements Comparator<PriorityTask> {
        @Override
        public int compare(PriorityTask t1, PriorityTask t2) {
            // Primary: Compare by effective priority (higher first)
            int priorityComparison = Integer.compare(t2.getEffectivePriority(), t1.getEffectivePriority());
            if (priorityComparison != 0) {
                return priorityComparison;
            }
            
            // Secondary: Compare by submission time (older first)
            int timeComparison = Long.compare(t1.getSubmissionTime(), t2.getSubmissionTime());
            if (timeComparison != 0) {
                return timeComparison;
            }
            
            // Tertiary: Compare by ID (deterministic)
            return Long.compare(t1.getId(), t2.getId());
        }
    }
    
    /**
     * PriorityProducer - Generates tasks with different priorities
     * 
     * Task generation strategy:
     * - 10% CRITICAL tasks
     * - 20% HIGH priority tasks
     * - 40% MEDIUM priority tasks
     * - 30% LOW priority tasks
     */
    public class PriorityProducer implements Runnable {
        private final String producerName;
        
        public PriorityProducer(String name) {
            this.producerName = name;
        }
        
        @Override
        public void run() {
            try {
                while (running && !Thread.currentThread().isInterrupted()) {
                    // Generate task with random priority distribution
                    PriorityTask.Priority priority = generateRandomPriority();
                    
                    PriorityTask task = new PriorityTask(
                        producerName + "-Task-" + taskIdGenerator.incrementAndGet(),
                        "Data-" + System.currentTimeMillis(),
                        priority
                    );
                    
                    // Add to priority queue
                    priorityQueue.put(task);
                    System.out.println(producerName + " produced: " + task);
                    
                    // Variable production rate
                    Thread.sleep(50 + (int)(Math.random() * 150));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println(producerName + " interrupted");
            }
        }
        
        /**
         * generateRandomPriority() - Generates priority based on realistic distribution
         * 
         * Distribution:
         * - CRITICAL: 10% (emergency tasks)
         * - HIGH: 20% (important business tasks)
         * - MEDIUM: 40% (regular business tasks)
         * - LOW: 30% (background/maintenance tasks)
         */
        private PriorityTask.Priority generateRandomPriority() {
            double random = Math.random();
            if (random < 0.1) return PriorityTask.Priority.CRITICAL;
            if (random < 0.3) return PriorityTask.Priority.HIGH;
            if (random < 0.7) return PriorityTask.Priority.MEDIUM;
            return PriorityTask.Priority.LOW;
        }
    }
    
    /**
     * PriorityConsumer - Processes tasks in priority order
     * 
     * Processing behavior:
     * 1. Always takes highest priority task available
     * 2. Simulates different processing times based on priority
     * 3. Logs processing details for monitoring
     */
    public class PriorityConsumer implements Runnable {
        private final String consumerName;
        
        public PriorityConsumer(String name) {
            this.consumerName = name;
        }
        
        @Override
        public void run() {
            try {
                while (running && !Thread.currentThread().isInterrupted()) {
                    // Take highest priority task
                    PriorityTask task = priorityQueue.take();
                    
                    // Process task (different processing times by priority)
                    processTask(task);
                    
                    long waitTime = System.currentTimeMillis() - task.getSubmissionTime();
                    System.out.println(String.format(
                        "%s processed: %s (waited %dms)", 
                        consumerName, task, waitTime
                    ));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                System.out.println(consumerName + " interrupted");
            }
        }
        
        /**
         * processTask() - Simulates priority-based processing time
         * 
         * Processing times:
         * - CRITICAL: 50-100ms (fast processing)
         * - HIGH: 100-200ms
         * - MEDIUM: 200-400ms
         * - LOW: 400-800ms (can take longer)
         */
        private void processTask(PriorityTask task) throws InterruptedException {
            int baseTime = switch (task.getPriority()) {
                case CRITICAL -> 50;
                case HIGH -> 100;
                case MEDIUM -> 200;
                case LOW -> 400;
            };
            
            int processingTime = baseTime + (int)(Math.random() * baseTime);
            Thread.sleep(processingTime);
        }
    }
    
    /**
     * Demo method showing priority-based processing
     * 
     * Configuration:
     * - 3 producer threads (mixed priority generation)
     * - 2 consumer threads (priority-based processing)
     * - Runs for 15 seconds with monitoring
     */
    public void runDemo() {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        ScheduledExecutorService monitor = Executors.newSingleThreadScheduledExecutor();
        
        try {
            // Start producers
            executor.submit(new PriorityProducer("Producer-1"));
            executor.submit(new PriorityProducer("Producer-2"));
            executor.submit(new PriorityProducer("Producer-3"));
            
            // Start consumers
            executor.submit(new PriorityConsumer("Consumer-1"));
            executor.submit(new PriorityConsumer("Consumer-2"));
            
            // Start monitoring
            monitor.scheduleAtFixedRate(this::printQueueStats, 2, 2, TimeUnit.SECONDS);
            
            // Let it run for 15 seconds
            Thread.sleep(15000);
            
            // Graceful shutdown
            running = false;
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            shutdownGracefully(executor);
            shutdownGracefully(monitor);
            printFinalStats();
        }
    }
    
    /**
     * printQueueStats() - Monitors queue statistics during execution
     * 
     * Metrics tracked:
     * - Queue size
     * - Priority distribution
     * - Average wait times by priority
     */
    private void printQueueStats() {
        int queueSize = priorityQueue.size();
        if (queueSize > 0) {
            Map<PriorityTask.Priority, Long> priorityCounts = priorityQueue.stream()
                .collect(Collectors.groupingBy(
                    PriorityTask::getPriority, 
                    Collectors.counting()
                ));
            
            System.out.println("Queue Stats - Size: " + queueSize + 
                             ", Priorities: " + priorityCounts);
        }
    }
    
    private void printFinalStats() {
        System.out.println("Final queue size: " + priorityQueue.size());
        if (!priorityQueue.isEmpty()) {
            System.out.println("Remaining tasks by priority:");
            priorityQueue.stream()
                .collect(Collectors.groupingBy(
                    PriorityTask::getPriority, 
                    Collectors.counting()
                ))
                .forEach((priority, count) -> 
                    System.out.println("  " + priority + ": " + count));
        }
    }
    
    private void shutdownGracefully(ExecutorService executor) {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
```

### Advanced Producer-Consumer with Different Queue Types
```java
/**
 * MultiQueueProducerConsumer - Demonstrates different BlockingQueue implementations
 * 
 * Purpose: Shows characteristics and use cases of different queue types
 * Queue types demonstrated:
 * 1. ArrayBlockingQueue - Fixed capacity, bounded
 * 2. LinkedBlockingQueue - Optionally bounded, linked nodes
 * 3. PriorityBlockingQueue - Unbounded, priority-ordered
 * 4. SynchronousQueue - Zero capacity, direct handoff
 * 5. DelayQueue - Time-based delayed elements
 */
public class MultiQueueProducerConsumer {
    
    /**
     * ArrayBlockingQueue Demo - Fixed size, bounded queue
     * 
     * Characteristics:
     * - Fixed capacity (specified at creation)
     * - Blocks producers when full
     * - Blocks consumers when empty
     * - Good for memory-bounded scenarios
     */
    public void arrayBlockingQueueDemo() {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(5);
        
        // Producer that will block when queue is full
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    String item = "Item-" + i;
                    System.out.println("Producing: " + item);
                    queue.put(item); // Blocks when queue is full
                    System.out.println("Produced: " + item + " (queue size: " + queue.size() + ")");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        // Consumer with slower processing
        Thread consumer = new Thread(() -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    String item = queue.take(); // Blocks when queue is empty
                    System.out.println("Consuming: " + item);
                    Thread.sleep(1000); // Slow consumer
                    System.out.println("Consumed: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        try {
            producer.join();
            Thread.sleep(2000);
            consumer.interrupt();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * SynchronousQueue Demo - Zero capacity, direct handoff
     * 
     * Characteristics:
     * - No internal capacity
     * - Producer blocks until consumer takes item
     * - Consumer blocks until producer provides item
     * - Direct thread-to-thread handoff
     * - Good for immediate processing requirements
     */
    public void synchronousQueueDemo() {
        SynchronousQueue<String> queue = new SynchronousQueue<>();
        
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    String item = "SyncItem-" + i;
                    System.out.println("Producer waiting to hand off: " + item);
                    queue.put(item); // Blocks until consumer takes it
                    System.out.println("Producer handed off: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    Thread.sleep(2000); // Consumer delay
                    System.out.println("Consumer ready to take item");
                    String item = queue.take(); // Blocks until producer provides
                    System.out.println("Consumer received: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        try {
            producer.join();
            consumer.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * DelayQueue Demo - Time-based delayed processing
     * 
     * Characteristics:
     * - Elements can only be taken when their delay has expired
     * - Unbounded queue
     * - Elements must implement Delayed interface
     * - Good for scheduled tasks, time-based processing
     */
    public void delayQueueDemo() {
        DelayQueue<DelayedTask> delayQueue = new DelayQueue<>();
        
        // Producer adding tasks with different delays
        Thread producer = new Thread(() -> {
            delayQueue.offer(new DelayedTask("Task-1", 1000));  // 1 second delay
            delayQueue.offer(new DelayedTask("Task-2", 3000));  // 3 second delay
            delayQueue.offer(new DelayedTask("Task-3", 2000));  // 2 second delay
            delayQueue.offer(new DelayedTask("Task-4", 500));   // 0.5 second delay
            System.out.println("All delayed tasks added to queue");
        });
        
        // Consumer that processes tasks only when delay expires
        Thread consumer = new Thread(() -> {
            try {
                while (!Thread.currentThread().isInterrupted()) {
                    DelayedTask task = delayQueue.take(); // Blocks until delay expires
                    System.out.println("Processing: " + task);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
        
        try {
            producer.join();
            Thread.sleep(5000); // Let delayed tasks be processed
            consumer.interrupt();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    /**
     * DelayedTask - Task that can be delayed in DelayQueue
     * 
     * Implementation requirements:
     * 1. Implement Delayed interface
     * 2. Provide getDelay() method
     * 3. Implement Comparable for ordering
     */
    public static class DelayedTask implements Delayed {
        private final String name;
        private final long executeTime;
        
        public DelayedTask(String name, long delayInMillis) {
            this.name = name;
            this.executeTime = System.currentTimeMillis() + delayInMillis;
        }
        
        @Override
        public long getDelay(TimeUnit unit) {
            long remainingTime = executeTime - System.currentTimeMillis();
            return unit.convert(remainingTime, TimeUnit.MILLISECONDS);
        }
        
        @Override
        public int compareTo(Delayed other) {
            return Long.compare(this.executeTime, ((DelayedTask) other).executeTime);
        }
        
        @Override
        public String toString() {
            return "DelayedTask{name='" + name + "', remaining=" + 
                   getDelay(TimeUnit.MILLISECONDS) + "ms}";
        }
    }
    
    /**
     * Main demo method that runs all queue type demonstrations
     */
    public static void main(String[] args) {
        MultiQueueProducerConsumer demo = new MultiQueueProducerConsumer();
        
        System.out.println("=== ArrayBlockingQueue Demo ===");
        demo.arrayBlockingQueueDemo();
        
        System.out.println("\n=== SynchronousQueue Demo ===");
        demo.synchronousQueueDemo();
        
        System.out.println("\n=== DelayQueue Demo ===");
        demo.delayQueueDemo();
    }
}
```


