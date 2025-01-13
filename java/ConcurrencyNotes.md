
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
# USer Thread Vs Daemon Thread 

User threads are high priority threads which always run in foreground. Where as Daemon threads are low priority threads which always run in background. User threads are designed to do some specific task where as daemon threads are used to perform some supporting tasks.

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

<h5> why do we need such thread pools when we can create objects of java.lang.Thread or implement Runnable/Callable interfaces to
  achieve parallelism? </h5>

1.)Creating a new thread for a new task leads to overhead of thread creation and tear-down. Managing this thread life-cycle significantly 
    adds to the execution time.
    
2.)Adding a new thread for each process without any throttling leads to the creation of a large number of threads.
These threads occupy memory and cause wastage ofresources. The CPU starts to spend too much time switching contexts when each thread is
swapped out and another thread comes in for execution.

Thread pools overcome this issue by keeping the threads alive and reusing the threads. Any excess tasks flowing in that the threads in the
pool can handle are held in a Queue. Once any of the threads get free, they pick up the next task from this queue. This task queue is
essentially unbounded for the out-of-box executors provided by the JDK.

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

