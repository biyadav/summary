Java provides its own multi-threading framework called the Executor Framework.
The Executor Framework contains a bunch of components that are used to efficiently manage worker threads. 
The Executor API de-couples the execution of task from the actual task to be executed via Executors.
This design is one of the implementations of the Producer-Consumer pattern.
The java.util.concurrent.Executors provide factory methods which are to be used to create ThreadPools of worker threads.
It is the job of the Executor Framework to schedule and execute the submitted tasks and return the results from the thread pool.

<h1>why do we need such thread pools when we can create objects of java.lang.Thread or implement Runnable/Callable interfaces to achieve parallelism?</h1>
