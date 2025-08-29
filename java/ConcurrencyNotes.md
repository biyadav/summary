
<h5>  Why Multithreading? </h5>

 Common reasons for using multithreading :
  a. Better utilization of a single CPU.
  b. Better utilization of multiple CPUs or CPU cores.
  c. Better user experience with regards to responsiveness.
  d. Better user experience with regards to fairness.
 New functional programming parallelism has been introduced with the Fork and Join framework in Java 7,
 and the collection streams API in java 8 .


<h5>  java.lang.Thread Class: Thread implements Runnable  </h5>

Provides a way to create and manage threads.
Key Methods:
 • public synchronized void start(): Starts the thread.
 • public void run(): Defines the code executed by the thread.
 • public final void join() throws InterruptedException: Waits for a thread to finish execution.

Example:
```
import java.lang.Thread;
public class SimpleThreadExample {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> System.out.println("Thread running: " + Thread.currentThread().getName()));
        thread.start();
    }
}
```

<h5> Thread methods </h5>

1. public synchronized void start : Begins the execution of the thread. The Java Virtual Machine (JVM) calls the run() method of the 
   thread.
2. public void run(): The entry point for the thread. When the thread is started, the run() method is invoked. If the thread was created 
   using a class that implements Runnable, the run() method will execute the run() method of that Runnable object.
3. public static native void sleep(long millis): Causes the currently executing thread to sleep (temporarily cease execution) for the 
   specified number of milliseconds.
4. public final void join( ): Waits for this thread to die. When one thread calls the join() method of another thread, it pauses the 
   execution of the current thread until the thread being joined has completed its execution. public final synchronized void join(final 
   long millis)
5. public final void setPriority(int newPriority): Changes the priority of the thread. The priority is a value between Thread.MIN_PRIORITY 
    (1) and Thread.MAX_PRIORITY (10).

    Thread.currentThread() : return the current associated Thread 

6. public void interrupt(): Interrupts the thread. If the thread is blocked in a call to wait(), sleep(), or join(), it will throw an InterruptedException.

7. public static native void yield(): Thread.yield() is a static method that suggests the current thread temporarily pause its execution 
  to allow other threads of the same or higher priority to execute. It’s important to note that yield() is just a hint to the thread 
  scheduler, and the actual behavior may vary depending on the JVM and OS. Thread.yield();

```
public class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " is running...");
            Thread.yield();
        }
    }

    public static void main(String[] args) {
        MyThread t1 = new MyThread(); 
        MyThread t2 = new MyThread();
        t1.start();
        t2.start();
    }
}

```

8. public final void setDaemon(boolean on) : Marks the thread as either a daemon thread or a user thread. The Java Virtual Machine exits 
   when the only threads running are all daemon threads.This method must be invoked before the thread is started
9. public final boolean isDaemon()
10. public static void sleep(long millis) throws InterruptedException
    Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds,When the thread is going for sleep, it does not release the      synchronized locks it holds.

<h5>    Runnable Interface  </h5>
Runnable is a functional interface representing a task that does not return a result and cannot throw checked exceptions.


```
When to Use:
	• You need to execute a task without expecting any result.
	• The task does not throw checked exceptions.
	• You’re working with APIs like Thread or Executor that only require a Runnable.

Runnable vs Thread:

Use Runnable when you want to separate the task from the thread, allowing the class to extend another class if needed.
Extend Thread if you need to override Thread methods or if the task inherently requires direct control over the thread itself,
though this limits inheritance.

```

```
import java.lang.Runnable;

 public class RunnableExample {
  public static void main(String[] args) { 
        Runnable task = () -> System.out.println("Runnable task running"); Thread thread = new Thread(task); thread.start();
   } 
}

Using Runnable with ExecutorService

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class RunnableWithExecutorExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Runnable task = () -> System.out.println("Task executed via ExecutorService");
        executor.execute(task);
        executor.shutdown();
    }
}

```

![image](https://github.com/user-attachments/assets/30cfacf8-f18e-45ec-8e7e-098eb4dd0546)


<h5>    Guidelines for Choosing  </h5>

```
Use Runnable when:

The task is fire-and-forget.
You don’t care about the result of the computation.
The task doesn’t involve checked exceptions.

Use Callable when:

The task must return a result (e.g., fetching data or computing a value).
The task may throw checked exceptions.
You’re working with concurrency utilities like Future.

```

<h5>  Extending Thread Vs implementing Runnable  </h5>
 
Java doesn’t have multiple inheritance so this option will limit your ability to extend anything else.
More importantly, it’s just not a very pretty way of doing it. As good developers we aim to favour composition
over inheritance.Also it seperate the task which needs to be executed from  the class maintaining thread life cycle .
A thread implementation is more related to how to start a new thread instead of mixing the task logic .

java.util.concurrent package was added to Java 5.


<h5>The Callable Interface</h5>

Callable interface is an improved version of Runnable. Callable is a functional interface introduced in Java 5
that represents a task capable of returning a result and throwing checked exceptions.

```
When to Use:
	• You need a task to return a result.
	• The task might throw a checked exception.
	• You're working with ExecutorService methods like submit(Callable<T>).
Key Method:
	• public abstract V call() throws Exception: The method to define the task logic and return a result.
 
public class DataReader implements Callable {
    @Override
    public String call() throws Exception {
        System.out.println("Reading data...");
        TimeUnit.SECONDS.sleep(5);
        return "Data reading finished";
    }
}
```

```
public static void main(String[] args) throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(2);

    Future<String> dataReadFuture = executorService.submit(new DataReader());
    Future<String> dataProcessFuture = executorService.submit(new DataProcessor());

    while (!dataReadFuture.isDone() && !dataProcessFuture.isDone()) {
            System.out.println("Reading and processing not yet finished.");
            // Do some other things that don't depend on these two processes
            // Simulating another task
            TimeUnit.SECONDS.sleep(1);
        }
    System.out.println(dataReadFuture.get());
    System.out.println(dataProcessFuture.get());
}

OR WITH TIMEOUT

public static void main(String[] args) throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(2);

    Future<String> dataReadFuture = executorService.submit(new DataReader());
    Future<String> dataProcessFuture = executorService.submit(new DataProcessor());

    System.out.println("Doing another task in anticipation of the results.");
    // Simulating another task
    TimeUnit.SECONDS.sleep(1);
    System.out.println(dataReadFuture.get());
    System.out.println(dataProcessFuture.get());
}

```
If we submit more than these two tasks into this pool, the additional tasks will wait in the queue until a free spot emerges.

 <h5>Combining Callable and Runnable </h5>
 
 You can combine Callable and Runnable using Executors.callable(Runnable task, T result) 
 if you need a Callable wrapper for a Runnable task.
 
 ```
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
public class CallableFromRunnableExample {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();
Runnable runnableTask = () -> System.out.println("Runnable task executed");
        Callable<String> callableTask = Executors.callable(runnableTask, "Success");
Future<String> future = executor.submit(callableTask);
        System.out.println("Result: " + future.get());
executor.shutdown();
    }
}
Summary
  • Use Runnable for simple tasks that don’t require results or checked exception handling.
  • Use Callable for tasks where you need to compute and retrieve a result or handle exceptions.

```
## USer Thread Vs Daemon Thread 

User threads are high priority threads which always run in foreground. Where as Daemon threads are low priority threads which always run in background. User threads are designed to do some specific task where as daemon threads are used to perform some supporting tasks.
Default daemon status of a thread is inherited from it’s parent thread i.e a thread created by user thread will be a user thread and a thread created by a daemon thread will be a daemon thread.You can convert user thread into daemon thread and vice-versa using setDaemon() method but before starting the thread.After starting the thread, calling setDaemon throws java.lang.IllegalThreadStateException.

isDaemon() method is used to check whether a thread is daemon thread or not.

1. JVM waits for user threads to finish their work. JVM will not exit until all user threads finish their tasks.JVM will not wait for 
    daemon threads to finish their work. It will exit as soon as all user threads finish their work.
2. User threads are foreground threads.	Daemon threads are background threads.
3. User threads are high priority threads.Daemon threads are low priority threads.
4. User threads are created by the application.	Daemon threads, in most of time, are created by the JVM.
5. User threads are mainly designed to do some specific task.Daemon threads are designed to support the user threads.
6. JVM will not force the user threads to terminate. It will wait for user threads to terminate themselves.JVM will force the daemon 
   threads to terminate if all user threads have finished their work.

<h4> Executor Framework </h4>

Java provides its own multi-threading framework called the Executor Framework.
The Executor Framework contains a bunch of components that are used to efficiently manage worker threads. 
The Executor API de-couples the execution of task from the actual task to be executed via Executors.
This design is one of the implementations of the Producer-Consumer pattern.
The java.util.concurrent.Executors provide factory methods which are to be used to create ThreadPools of worker threads.
It is the job of the Executor Framework to schedule and execute the submitted tasks and return the results from the thread pool.

<h4> why do we need such thread pools when we can create objects of java.lang.Thread or implement Runnable/Callable interfaces to
  achieve parallelism? </h5>

1.)Creating a new thread for a new task leads to overhead of thread creation and tear-down. Managing this thread life-cycle significantly 
    adds to the execution time.
    
2.)Adding a new thread for each process without any throttling leads to the creation of a large number of threads.
These threads occupy memory and cause wastage ofresources. The CPU starts to spend too much time switching contexts when each thread is
swapped out and another thread comes in for execution.

Thread pools overcome this issue by keeping the threads alive and reusing the threads. Any excess tasks flowing in that the threads in the
pool can handle are held in a Queue. Once any of the threads get free, they pick up the next task from this queue. This task queue is
essentially unbounded for the out-of-box executors provided by the JDK.

### Benefits of Executors framework
* Avoiding Manual Thread management
* Resource management
* Scalability
* Thread reuse
* Error handling

<h5> java.util.concurrent.Executor </h5>
This interface provides a way of decoupling task submission from the mechanics of how each task will be run, 
void execute(Runnable command); 

<h5> java.util.concurrent.ExecutorService</h5>
  extends the Executor interface 

An Executor that provides methods to manage termination and methods that can produce a Future for tracking progress of one or more asynchronous tasks.
An ExecutorService can be shut down, which will cause it to reject new tasks. The shutdown method will allow previously submitted tasks to execute before terminating, while the shutdownNow method prevents waiting tasks from starting and attempts to stop currently executing tasks.  An unused ExecutorService should be shut down to allow reclamation of its resources.
Method submit extends base method Executor.execute(Runnable) by creating and returning a Future that can be used to cancel execution and/ or wait for completion.
Methods invokeAny and invokeAll perform the most commonly useful forms of bulk execution, executing a collection of tasks and then waiting for at least one, or all, to complete. 

 *  void shutdown() Initiates an orderly shutdown in which previously submitted tasks are executed, but no new tasks will be accepted.
 *  List<Runnable> shutdownNow() Attempts to stop all actively executing tasks, halts the processing of waiting tasks, and returns a list of the tasks that were awaiting execution.
 *  boolean isShutdown()
 *  boolean isTerminated() isTerminated is never true unless either shutdown or shutdownNow was called first. Returns true if all tasks have completed following shut down
 *  <T> Future<T> submit(Callable<T> task) Submits a value-returning task for execution and returns a Future representing the pending results of the task.
 *  <T> Future<T> submit(Runnable task, T result)
 *  Future<?> submit(Runnable task)
 *  <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
 *  <T> T invokeAny(Collection<? extends Callable<T>> tasks)


<h5>  SingleThreadExecutor  </h5>
 Has only a single thread. It is used to execute tasks in a sequential manner. If the thread dies due to an exception while executing a task,
 a new thread is created to replace the old thread and the subsequent tasks are executed in the new one.

 ExecutorService executorService = Executors.newSingleThreadExecutor()

 <h5>   FixedThreadPool(n)   </h5>

Is a thread pool of a fixed number of threads. The tasks submitted to the executor are executed by the n threads and if there are more tasks
they are stored on a LinkedBlockingQueue
ExecutorService executorService = Executors.newFixedThreadPool(4);

<h5>  CachedThreadPool  </h5>

This thread pool is mostly used where there are lots of short-lived parallel tasks to be executed. Unlike the fixed thread pool,
the number of threads of this executor pool is not bounded. If all the threads are busy executing some tasks and a new task comes,
the pool will create and add a new thread to the executor. As soon as one of the threads becomes free, it will take up the execution
of the new tasks. If a thread remains idle for sixty seconds, they are terminated and removed from cache.

However, if not managed correctly, or the tasks are not short-lived, the thread pool will have lots of live threads.
This may lead to resource thrashing and hence performance drop.


ExecutorService executorService = Executors.newCachedThreadPool();

<h5>  ScheduledExecutor  </h5>

When we have a task that needs to be run at regular intervals or if we wish to delay a certain task.
ScheduledExecutorService scheduledExecService = Executors.newScheduledThreadPool(1);
scheduledExecService.scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit)
scheduledExecService.scheduleWithFixedDelay(Runnable command, long initialDelay, long period, TimeUnit unit)

<h5>  The Future Object  </h5>

The result of the task submitted for execution to an executor can be accessed using the java.util.concurrent.Future
Future can be thought of as a promise made to the caller by the executor.
Future<String> result = executorService.submit(callableTask);
A task submitted to the executor, like above, is asynchronous i.e. the program execution does not wait for the completion
of task execution to proceed to the next step. Instead, whenever the task execution is completed, it is set in this Future object by the executor.
The caller can continue executing the main program and when the result of the submitted task is needed he can call .get() on this Future object.
If the task is complete the result is immediately returned to the caller or else the caller is blocked until the execution of this is completed by
the executor and the result is computed.
To avoid indefinite wait Use Future.get(long timeout, TimeUnit unit) method which throws a TimeoutException if the result is not returned in the stipulated time frame. 
If there is an exception when executing the task, the call to get method will throw an ExecutionException.
if the submitted task implements java.util.concurrent.Callable it will return result. If the task implements the Runnable interface, the call to .get() will
return null once the task is complete.
Future.cancel(boolean mayInterruptIfRunning) can be used to cancel the execution of a submitted task. 

```
public class Task implements Callable<String> {

    private String message;
    public Task(String message) {
        this.message = message;
    }
    @Override
    public String call() throws Exception {
        return "Hello " + message + "!";
    }
}

public class ExecutorExample {
    public static void main(String[] args) {

        Task task = new Task("World");
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        Future<String> result = executorService.submit(task);
        try {
            System.out.println(result.get());
        } catch (InterruptedException | ExecutionException e) {
            System.out.println("Error occurred while executing the submitted task");
            e.printStackTrace();
        }
        executorService.shutdown();
    }
}

```
we call the shutdown on the executorService object to terminate all the threads and return the resources back to the OS.
The .shutdown() method waits for the completion of currently submitted tasks to the executor. However, if the requirement
is to immediately shut down the executor without waitingthen we can use the .shutdownNow() method instead.
Any tasks pending for execution will be returned back in a java.util.List object.
```
public interface Future<V> {
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
    boolean isCancelled();
    boolean isDone();
    boolean cancel(boolean mayInterruptIfRunning)
}

```
if you don't receive a result within n seconds, you might just decide to not use the result at all. 
```
boolean cancelled = false;
if (dataReadFuture.isDone()) {
    try {
        dataReadResult = dataReadFuture.get();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
} else {
cancelled = dataReadFuture.cancel(true);
}
if (!cancelled) {
    System.out.println(dataReadResult);
} else {
    System.out.println("Task was cancelled.");
}
```

The cancel() method accepts a boolean parameter. This boolean defines whether we allow the cancel() method to interrupt the task execution or not.
And if we try getting the data from a canceled task, a CancellationException is generated:
```
if (dataReadFuture.cancel(true)) {
    dataReadFuture.get();
}
```

You can use synchronized keyword only with methods but not with variables, constructors, static initializers and instance initializers.
you can use synchronised blocks within it 


##  Object lock or monitor?

The synchronization in Java is built around an entity called object lock or monitor. Below is the brief description about lock or monitor.

Whenever an object is created to any class, an object lock is created and is stored inside the object.
One object will have only one object lock associated with it.
Any thread wants to enter into synchronized methods or blocks of any object, they must acquire object lock associated with that object and release the lock after they are done with the execution.
The other threads which wants to enter into synchronized methods of that object have to wait until the currently executing thread releases the object lock.
To enter into static synchronized methods or blocks, threads have to acquire class lock associated with that class as static members are stored inside the class memory.

##  Mutex
synchronized block takes one argument and it is called mutex. If synchronized block is defined inside non-static definition blocks like non-static methods, instance initializer or constructors, then this mutex must be an instance of that class. If synchronized block is defined inside static definition blocks like static methods or static initializer, then this mutex must be like ClassName.class.

## Synchrozied 

Synchronized keyword can not be used with constructors. But, constructors can have synchronized blocks.You can use synchronized keyword only with methods but not with variables, constructors, static initializers and instance initializers.

##  Java Lock   java.util.concurrent.locks.Lock    Implementations java.util.concurrent.locks.ReentrantLock

The Java Lock interface, java.util.concurrent.locks.Lock, represents a concurrent lock which can be used to guard against race conditions inside critical sections. 
The main differences between a Lock and a synchronized block are:

Synchronized locks the entire method or block, leading to potential performance issues. It lacks a try-lock mechanism, causing threads to block indefinitely,
increasing the risk of deadlocks.A synchronized block makes no guarantees about the sequence in which threads waiting to entering it are granted access.
You cannot pass any parameters to the entry of a synchronized block. Thus, having a timeout trying to get access to a synchronized block is not possible.
The synchronized block must be fully contained within a single method. A Lock can have it's calls to lock() and unlock() in separate methods.

## Reentrant Lock
A Reentrant Lock in Java is a type of lock that allows a thread to acquire the same lock multiple times without causing a deadlock. If a thread already holds the lock,
it can re-enter the lock without being blocked. This is useful when a thread needs to repeatedly enter synchronized blocks or methods within the same execution flow. 

```
public class ReentrantExample {
    private final Lock lock = new ReentrantLock();

    public void outerMethod() {
        lock.lock();
        try {
            System.out.println("Outer method");
            innerMethod();
        } finally {
            lock.unlock();
        }
    }

    public void innerMethod() {
        lock.lock();
        try {
            System.out.println("Inner method");
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ReentrantExample example = new ReentrantExample();
        example.outerMethod();
    }
}

```

##  Methods of ReentrantLock

* lock() blocks until lock acquired
* tryLock() unblocking wont wait for lock  return true or false whether lock acquired 
* tryLock(long timeout, TimeUnit unit)
* unlock()
# Locking and unLocking java Lock 

Obviously all threads must share the same Lock instance. If each thread creates its own Lock instance, then they will be locking on different locks, and thus not be blocking each other from access. <h2>To avoid exceptions locking a Lock forever, you should lock and unlock it from within a try-finally block,</h2> like this:
```
Lock lock = new ReentrantLock();

try{
    lock.lock();
      //critical section
} finally {
    lock.unlock();
}

```

## Read Write Lock

A Read-Write Lock allows multiple threads to read shared data simultaneously while restricting write access to a single thread at a time.  ReentrantReadWriteLock class in Java optimizes performance in scenarios with frequent read operations and infrequent writes. Multiple readers can acquire the read lock without blocking each other, but when a thread needs to write, it must acquire the write lock, ensuring exclusive access .

```

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteCounter {
    private int count = 0;
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Lock readLock = lock.readLock();
    private final Lock writeLock = lock.writeLock();

    public void increment() {
        writeLock.lock();
        try {
            count++;
            Thread.sleep(50);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            writeLock.unlock();
        }
    }

    public int getCount() {
        readLock.lock();
        try {
            return count;
        } finally {
            readLock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ReadWriteCounter counter = new ReadWriteCounter();

        Runnable readTask = new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println(Thread.currentThread().getName() + " read: " + counter.getCount());
                }
            }
        };

        Runnable writeTask = new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    counter.increment();
                    System.out.println(Thread.currentThread().getName() + " incremented");
                }
            }
        };

        Thread writerThread = new Thread(writeTask);
        Thread readerThread1 = new Thread(readTask);
        Thread readerThread2 = new Thread(readTask);

        writerThread.start();
        readerThread1.start();
        readerThread2.start();

        writerThread.join();
        readerThread1.join();
        readerThread2.join();

        System.out.println("Final count: " + counter.getCount());
    }
}
```

## Fairness of Locks

A fair lock ensures that threads acquire the lock in the order they requested it, preventing thread starvation. With a fair lock, if multiple threads are waiting, the longest-waiting thread is granted the lock next. However, fairness can lead to lower throughput due to the overhead of maintaining the order. Non-fair locks, in contrast offer better performance but at the risk of some threads waiting indefinitely.   private final Lock fairLock = new ReentrantLock(true); // pass true in constructor






## Is it possible to declare a constructor as synchronized?

No, constructors cannot be synchronized because object locks do not exist until the object is created. However, synchronized blocks can be used inside constructors.
```
class Test {
    Test() {
        synchronized (this) {
            System.out.println("Synchronized block in constructor");
        }
    }
}
```

##  Java Concurrency – Synchronizers
The java.util.concurrent package contains several classes that help manage a set of threads that collaborate with each other.

- CyclicBarrier
- Semaphore
- CountDownLatch
- SynchronousQueue
- Phaser
- Exchanger


##  java.util.concurrent.CountDownLatch 

A CountDownLatch is a synchronization aid in Java that allows one or more threads to wait until a set of operations being performed in other threads completes.

1.  Initialization: You initialize a CountDownLatch with a given count. This count is the number of times the countDown() method must be invoked before the threads waiting on the 
     latch can proceed.
2.  Waiting: Threads call the await() method on the latch to wait until the count reaches zero.
3.  Counting Down: Other threads call the countDown() method to decrease the count. When the count reaches zero, all waiting threads are released.

USE CASE :

-  Starting Multiple Threads at the Same Time: You can use a CountDownLatch to ensure that multiple threads start executing at the same time. This is useful in performance testing 
   where you want to simulate concurrent user activity.
-  Waiting for Multiple Threads to Complete: If you have a main thread that needs to wait for several worker threads to complete their tasks before proceeding, a CountDownLatch can 
   be used to block the main thread until all worker threads have finished.
-  Dividing a Task Among Multiple Threads: When a large task can be divided into smaller subtasks that can be executed in parallel, you can use a CountDownLatch to wait for all 
   subtasks to complete before combining the results.
-  Deadlock Detection: In complex systems, CountDownLatch can help in detecting deadlocks by ensuring that certain operations are completed before others start.
-  Resource Initialization: If multiple threads need to wait for a resource to be initialized before they can proceed, a CountDownLatch can be used to block the threads until the 
   resource is ready.

```

In this Example all worker thread will wait untill Main thread call countdown on startSignal so all start at same time .
Then main method will wait when all thread complete their task collectively making count zero  calling coutdown on doneSignal.

import java.util.concurrent.CountDownLatch;

public class StartTogetherExample {
    public static void main(String[] args) {
        int numberOfThreads = 5;
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(numberOfThreads);

        for (int i = 0; i < numberOfThreads; i++) {
            new Thread(new Worker(startSignal, doneSignal)).start();
        }

        try {
            System.out.println("Ready... Set... Go!");
            startSignal.countDown(); // Release all threads to start at the same time
            doneSignal.await(); // Wait for all threads to finish
            System.out.println("All threads have finished.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Worker implements Runnable {
    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;

    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }

    @Override
    public void run() {
        try {
            startSignal.await(); // Wait for the start signal
            // Simulate some work
            Thread.sleep((long) (Math.random() * 1000));
            System.out.println(Thread.currentThread().getName() + " finished work.");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            doneSignal.countDown(); // Signal that this thread has finished
        }
    }
}

```

# Complex Scenario :Parallel Data Processing

Imagine we have a large list of numbers, and we want to calculate the sum of squares of these numbers using multiple threads. Each thread will process a portion of the list, and the main thread will wait for all threads to complete before combining the results.
<h4> Step-by-Step Implementation </h4>

* Initialize the CountDownLatch: Create a CountDownLatch with the count equal to the number of worker threads.
* Divide the Task: Split the list into smaller sublists, each to be processed by a separate worker thread.
* Process the Data: Each worker thread calculates the sum of squares for its sublist and stores the result.
* Combine the Results: The main thread waits for all threads to complete and then combines the results.

```

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class ParallelDataProcessingExample {
    public static void main(String[] args) {
        int numberOfThreads = 4;
        List<Integer> data = generateData(1000); // Generate a list of 1000 numbers
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        List<Worker> workers = new ArrayList<>();

        // Divide the data into sublists and create worker threads
        int chunkSize = data.size() / numberOfThreads;
        for (int i = 0; i < numberOfThreads; i++) {
            int start = i * chunkSize;
            int end = (i == numberOfThreads - 1) ? data.size() : start + chunkSize;
            List<Integer> sublist = data.subList(start, end);
            Worker worker = new Worker(sublist, latch);
            workers.add(worker);
            new Thread(worker).start();
        }

        try {
            latch.await(); // Wait for all threads to finish
            int totalSumOfSquares = workers.stream().mapToInt(Worker::getSumOfSquares).sum();
            System.out.println("Total sum of squares: " + totalSumOfSquares);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static List<Integer> generateData(int size) {
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            data.add(i + 1);
        }
        return data;
    }
}

class Worker implements Runnable {
    private final List<Integer> data;
    private final CountDownLatch latch;
    private int sumOfSquares;

    Worker(List<Integer> data, CountDownLatch latch) {
        this.data = data;
        this.latch = latch;
    }

    @Override
    public void run() {
        sumOfSquares = data.stream().mapToInt(i -> i * i).sum();
        System.out.println(Thread.currentThread().getName() + " calculated sum of squares: " + sumOfSquares);
        latch.countDown(); // Signal that this thread has finished
    }

    public int getSumOfSquares() {
        return sumOfSquares;
    }
}

```

## Handling Exception in Worker Threads 



```
/*
Strategy 1: Catch and Log Exceptions in Worker Threads:

Ensure that each worker thread catches and logs any exceptions that occur during its execution.
This prevents the thread from terminating abruptly and allows you to handle the error appropriately.
*/
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class ExceptionHandlingExample {
    public static void main(String[] args) {
        int numberOfThreads = 4;
        List<Integer> data = generateData(1000); // Generate a list of 1000 numbers
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        List<Worker> workers = new ArrayList<>();

        // Divide the data into sublists and create worker threads
        int chunkSize = data.size() / numberOfThreads;
        for (int i = 0; i < numberOfThreads; i++) {
            int start = i * chunkSize;
            int end = (i == numberOfThreads - 1) ? data.size() : start + chunkSize;
            List<Integer> sublist = data.subList(start, end);
            Worker worker = new Worker(sublist, latch);
            workers.add(worker);
            new Thread(worker).start();
        }

        try {
            latch.await(); // Wait for all threads to finish
            int totalSumOfSquares = workers.stream().mapToInt(Worker::getSumOfSquares).sum();
            System.out.println("Total sum of squares: " + totalSumOfSquares);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static List<Integer> generateData(int size) {
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            data.add(i + 1);
        }
        return data;
    }
}

class Worker implements Runnable {
    private final List<Integer> data;
    private final CountDownLatch latch;
    private int sumOfSquares;

    Worker(List<Integer> data, CountDownLatch latch) {
        this.data = data;
        this.latch = latch;
    }

    @Override
    public void run() {
        try {
            sumOfSquares = data.stream().mapToInt(i -> i * i).sum();
            System.out.println(Thread.currentThread().getName() + " calculated sum of squares: " + sumOfSquares);
        } catch (Exception e) {
            System.err.println(Thread.currentThread().getName() + " encountered an error: " + e.getMessage());
        } finally {
            latch.countDown(); // Signal that this thread has finished
        }
    }

    public int getSumOfSquares() {
        return sumOfSquares;
    }
}

```

```
/*
Strategy 2: Use a Shared Data Structure for Error Reporting 

We can use a shared data structure (e.g., a ConcurrentLinkedQueue) to collect exceptions from worker threads.
The main thread can then check this structure after all threads have completed to handle any errors.
*/

import java.util.ArrayList;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.CountDownLatch;

public class ExceptionHandlingWithQueueExample {
    public static void main(String[] args) {
        int numberOfThreads = 4;
        List<Integer> data = generateData(1000); // Generate a list of 1000 numbers
        CountDownLatch latch = new CountDownLatch(numberOfThreads);
        Queue<Exception> exceptions = new ConcurrentLinkedQueue<>();
        List<Worker> workers = new ArrayList<>();

        // Divide the data into sublists and create worker threads
        int chunkSize = data.size() / numberOfThreads;
        for (int i = 0; i < numberOfThreads; i++) {
            int start = i * chunkSize;
            int end = (i == numberOfThreads - 1) ? data.size() : start + chunkSize;
            List<Integer> sublist = data.subList(start, end);
            Worker worker = new Worker(sublist, latch, exceptions);
            workers.add(worker);
            new Thread(worker).start();
        }

        try {
            latch.await(); // Wait for all threads to finish
            if (!exceptions.isEmpty()) {
                System.err.println("Errors occurred during processing:");
                exceptions.forEach(e -> System.err.println(e.getMessage()));
            } else {
                int totalSumOfSquares = workers.stream().mapToInt(Worker::getSumOfSquares).sum();
                System.out.println("Total sum of squares: " + totalSumOfSquares);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private static List<Integer> generateData(int size) {
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            data.add(i + 1);
        }
        return data;
    }
}

class Worker implements Runnable {
    private final List<Integer> data;
    private final CountDownLatch latch;
    private final Queue<Exception> exceptions;
    private int sumOfSquares;

    Worker(List<Integer> data, CountDownLatch latch, Queue<Exception> exceptions) {
        this.data = data;
        this.latch = latch;
        this.exceptions = exceptions;
    }

    @Override
    public void run() {
        try {
            sumOfSquares = data.stream().mapToInt(i -> i * i).sum();
            System.out.println(Thread.currentThread().getName() + " calculated sum of squares: " + sumOfSquares);
        } catch (Exception e) {
            exceptions.add(e);
        } finally {
            latch.countDown(); // Signal that this thread has finished
        }
    }

    public int getSumOfSquares() {
        return sumOfSquares;
    }
}

```

## java.util.concurrent.CyclicBarrier


A CyclicBarrier allows a set of threads to all wait for each other to reach a common barrier point.
Once all threads have reached this point, the barrier is "broken," and all threads are released to continue their execution.
CyclicBarriers are used in programs in which we have a fixed number of threads that must wait for each other to reach a common point before continuing execution.
The barrier can be reused, making it cyclic.

The threads that need to synchronize their execution are also called parties and calling the await() method is how we can register that a certain thread has reached the barrier point.
This call is synchronous and the thread calling this method suspends execution till a specified number of threads have called the same method on the barrier. This situation where the required number of threads have called await(), is called tripping the barrier.

Optionally, we can pass the second argument to the constructor, which is a Runnable instance. This has logic that would be run by the last thread that trips the barrier:
public CyclicBarrier(int parties, Runnable barrierAction)

##  Scenario: Multi-Phase Computation

Imagine we have a large dataset that needs to be processed in multiple phases. Each phase involves some computation, and all threads must synchronize at the end of each phase before moving on to the next phase.
1. Initialize the CyclicBarrier: Create a CyclicBarrier with the number of threads and a barrier action.
2. Divide the Task: Split the dataset into smaller chunks, each to be processed by a separate thread.
3. Process the Data in Phases: Each thread processes its chunk in multiple phases, synchronizing at the end of each phase.
4. Barrier Action: Define a barrier action to be executed at the end of each phase when all thread reach to barrier to be executed by last thread

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class MultiPhaseComputationExample {
    public static void main(String[] args) {
        int numberOfThreads = 4;
        int numberOfPhases = 3;
        List<Integer> data = generateData(1000); // Generate a list of 1000 numbers
        CyclicBarrier barrier = new CyclicBarrier(numberOfThreads, new BarrierAction());

        for (int i = 0; i < numberOfThreads; i++) {
            int start = i * (data.size() / numberOfThreads);
            int end = (i == numberOfThreads - 1) ? data.size() : start + (data.size() / numberOfThreads);
            List<Integer> sublist = data.subList(start, end);
            new Thread(new Worker(sublist, barrier, numberOfPhases)).start();
        }
    }

    private static List<Integer> generateData(int size) {
        List<Integer> data = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            data.add(i + 1);
        }
        return data;
    }
}

class Worker implements Runnable {
    private final List<Integer> data;
    private final CyclicBarrier barrier;
    private final int numberOfPhases;

    Worker(List<Integer> data, CyclicBarrier barrier, int numberOfPhases) {
        this.data = data;
        this.barrier = barrier;
        this.numberOfPhases = numberOfPhases;
    }

    @Override
    public void run() {
        try {
            for (int phase = 1; phase <= numberOfPhases; phase++) {
                processPhase(phase);
                System.out.println(Thread.currentThread().getName() + " completed phase " + phase);
                barrier.await(); // Wait for other threads reach the barrier
                System.out.println(" ");
            }
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }

    private void processPhase(int phase) {
        // Simulate some work for the phase
        for (int i = 0; i < data.size(); i++) {
            data.set(i, data.get(i) + phase);
        }
    }
}

class BarrierAction implements Runnable {
    @Override
    public void run() {
        System.out.println("All threads have reached the barrier. Proceeding to the next phase.");
     // We can use to combine results of all thread which they put to some concurrent collection at the end of each phase 
    }
}

Explanation:
Data Generation: The generateData method creates a list of integers from 1 to 1000.
Task Division: The data is divided into sublists, each assigned to a worker thread.
Worker Threads: Each worker thread processes its sublist in multiple phases. After each phase, the thread calls barrier.await() to wait for other threads.
Barrier Action: The BarrierAction class defines an action that is executed once all threads have reached the barrier at the end of each phase.

Key Points:
Reusability: The CyclicBarrier is reused for each phase, making it suitable for multi-phase computations.
Synchronization: All threads must reach the barrier before any of them can proceed to the next phase.
Barrier Action: The barrier action provides a way to perform additional operations once all threads have synchronized.

```

## Atomic classes  java.util.concurrent.atomic.*

Atomic classes provide classes to read and write atomically, and also contains advanced atomic operations like compareAndSet(). 
* AtomicBoolean  * AtomicInteger  * AtomicLong * AtomicReference * AtomicIntegerArray * AtomicLongArray * AtomicReferenceArray


### java.util.concurrent.atomic.AtomicBoolean
AtomicBoolean atomicBoolean = new AtomicBoolean();  initial value false
AtomicBoolean atomicBoolean = new AtomicBoolean(true); initial true
boolean value = atomicBoolean.get();
atomicBoolean.set(false);

Swapping Atomic values 
AtomicBoolean atomicBoolean = new AtomicBoolean(true);
boolean oldValue = atomicBoolean.getAndSet(false);

Compare and Set 
AtomicBoolean atomicBoolean = new AtomicBoolean(true);

boolean expectedValue = true;
boolean newValue      = false;
boolean wasNewValueSet = atomicBoolean.compareAndSet( expectedValue, newValue);  compares the current value of the AtomicBoolean to true and if the two values are equal, sets the new value of the AtomicBoolean to false .

###  java.util.concurrent.atomic.AtomicInteger

AtomicInteger atomicInteger = new AtomicInteger(); initial value 0 
AtomicInteger atomicInteger = new AtomicInteger(123);initial value 123
int theValue = atomicInteger.get(); retrun 123
atomicInteger.set(234); // set value to 234

AtomicInteger atomicInteger = new AtomicInteger(123);
int expectedValue = 123;
int newValue      = 234;
atomicInteger.compareAndSet(expectedValue, newValue);

addAndGet(n)  first add return addedValue
getAndAdd(n)  first return then add 
getAndIncrement()  return current value and increment by 1
incrementAndGet()   increment by 1 and the return incremented value

## java.util.concurrent.atomic.AtomicReference
The AtomicReference class provides an object reference variable which can be read and written atomically which means if multiple threads attemp to set the reference it will not be in inconsistent state

AtomicReference atomicReference = new AtomicReference();
String initialReference = "the initially referenced string";
AtomicReference atomicReference = new AtomicReference(initialReference);
String reference = (String) atomicReference.get();

AtomicReference<String> atomicStringReference = new AtomicReference<String>();
String initialReference = "the initially referenced string";
AtomicReference<String> atomicStringReference = new AtomicReference<String>(initialReference);
String reference = atomicReference.get();
