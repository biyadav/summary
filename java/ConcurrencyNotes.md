java.util.concurrent package was added to Java 5.

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
