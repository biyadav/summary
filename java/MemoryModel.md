<h5> Java Memory Model (JMM) </h5>
 
	1. Stack Memory:
		○ Each thread has its own stack memory.
		○ Used for method execution and local variables.
		○ Automatically managed (no garbage collection required).
		○ Thread-safe by nature (isolated to the thread).
	2. Heap Memory:
		○ Shared across all threads.
		○ Stores objects and class-level variables.
		○ Managed by the Garbage Collector (GC).
		○ Divided into:
			§ Young Generation: For newly allocated objects. Further divided into:
				□ Eden Space: Where objects are first allocated.
				□ Survivor Spaces (S0 and S1): Where objects that survive minor GC are moved.
			§ Old Generation (Tenured): For long-lived objects that survive multiple GC cycles.
	3. MetaSpace (Java 8+):
		○ Replaced the PermGen (Permanent Generation).
		○ Stores class metadata.
		○ Grows dynamically based on the application’s needs.
		○ Configurable using -XX:MetaspaceSize and -XX:MaxMetaspaceSize.

## Garbage Collection (GC) and Tuning
    Garbage Collection in Java is responsible for reclaiming memory by removing unused objects. 
    GC algorithms and configurations are critical for performance.

## Garbage Collection Algorithms
	1. Serial GC (-XX:+UseSerialGC):
		○ Single-threaded GC.
		○ Best for single-threaded applications or small heaps.
		○ Tuning:
			§ -XX:SurvivorRatio=<n>: Controls the size of the survivor spaces relative to Eden (default is 8:1).
	2. Parallel GC (Throughput Collector) (-XX:+UseParallelGC):
		○ Multi-threaded GC for both minor and major collections.
		○ Best for applications prioritizing throughput over pause time.
		○ Tuning:
			§ -XX:ParallelGCThreads=<n>: Number of threads for GC.
			§ -XX:MaxGCPauseMillis=<n>: Maximum desired pause time.
	3. CMS (Concurrent Mark-Sweep) GC (-XX:+UseConcMarkSweepGC):
		○ Low-latency collector with concurrent threads.
		○ Deprecated in Java 9 and removed in Java 14.
		○ Tuning:
			§ -XX:CMSInitiatingOccupancyFraction=<n>: Start CMS when heap is <n>% full.
			§ -XX:+UseCMSInitiatingOccupancyOnly: Prevent adaptive GC tuning.
	4. G1 GC (Garbage First) (-XX:+UseG1GC):
		○ Designed for low-latency applications.
		○ Partitions the heap into regions for more predictable performance.
		○ Tuning:
			§ -XX:MaxGCPauseMillis=<n>: Target pause time.
			§ -XX:G1HeapRegionSize=<n>: Size of each region (default 1MB, range 1MB–32MB).
	5. ZGC (Java 11+) (-XX:+UseZGC):
		○ Ultra-low-latency collector with pauses in milliseconds.
		○ Best for applications with very large heaps (>100GB).
		○ Tuning:
			§ Minimal tuning needed; designed to be mostly self-tuning.
	6. Shenandoah GC (Java 12+) (-XX:+UseShenandoahGC):
		○ Low-latency collector similar to ZGC.
		○ Suitable for applications requiring predictable latency.


## GC Logs and Monitoring
* Enable GC logging for performance analysis:
   -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log

* Heap Configuration Options
	1. Heap Size:
		○ -Xms<size>: Initial heap size.
		○ -Xmx<size>: Maximum heap size.
	2. New Generation Size:
		○ -XX:NewSize=<size>: Initial young generation size.
		○ -XX:MaxNewSize=<size>: Maximum young generation size.
	3. Survivor Ratio:
		○ -XX:SurvivorRatio=<n>: Eden-to-Survivor space ratio.
	4. Metaspace:
		○ -XX:MetaspaceSize=<size>: Initial metaspace size.
		○ -XX:MaxMetaspaceSize=<size>: Maximum metaspace size.

## Practical Tuning Example
 Scenario: High-throughput Web Service
	• Use Parallel GC for maximizing throughput:
	-XX:+UseParallelGC -Xms1g -Xmx4g -XX:NewRatio=2 -XX:SurvivorRatio=8 -XX:MaxGCPauseMillis=200

 Scenario: Low-latency Trading Application
	• Use G1 GC for predictable pause times:
         -XX:+UseG1GC -Xms4g -Xmx8g -XX:MaxGCPauseMillis=50 -XX:G1HeapRegionSize=16m
 
 Scenario: Real-time Analytics (Large Heap)
	• Use ZGC for ultra-low latency:
        -XX:+UseZGC -Xms16g -Xmx64g


# Example GC Log Snippet and Explanation


[0.238s][info][gc,start] GC(0) Pause Young (Normal) (G1 Evacuation Pause)
[0.238s][info][gc,task] GC(0) Using 8 workers for evacuation
[0.260s][info][gc,phases] GC(0)   Pre Evacuate Collection Set: 0.2ms
[0.260s][info][gc,phases] GC(0)   Evacuate Collection Set: 19.9ms
[0.260s][info][gc,phases] GC(0)   Post Evacuate Collection Set: 0.1ms
[0.260s][info][gc,phases] GC(0)   Other: 1.0ms
[0.260s][info][gc,heap] GC(0) Eden regions: 16->0(40)
[0.260s][info][gc,heap] GC(0) Survivor regions: 0->4(4)
[0.260s][info][gc,heap] GC(0) Old regions: 0->0
[0.260s][info][gc,heap] GC(0) Humongous regions: 0->0
[0.260s][info][gc,metaspace] GC(0) Metaspace: 10M->10M(105M)
[0.260s][info][gc] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 22.0ms


Breakdown of the Log
1. Time Stamps and Event Type
	• [0.238s][info][gc,start]:
		○ The GC starts at 0.238 seconds after the JVM process started.
		○ The event is categorized as info under the gc module.
	• GC(0): This is the first GC cycle, identified by the ID 0.
2. GC Type
	• Pause Young (Normal):
		○ Indicates a young generation collection (minor GC).
		○ "G1 Evacuation Pause" refers to the process of evacuating live objects from the young generation to the survivor or old generation.
3. Parallelism
	• Using 8 workers for evacuation:
		○ The GC uses 8 parallel threads for moving live objects to the survivor or old generation, optimizing performance.
4. Phases of GC
Each phase contributes to the overall pause time:
	• Pre Evacuate Collection Set: Prepares regions for evacuation (0.2ms).
	• Evacuate Collection Set: The main phase, moving live objects (19.9ms).
	• Post Evacuate Collection Set: Cleanup after evacuation (0.1ms).
	• Other: Miscellaneous tasks (1.0ms).
5. Heap Status
	• Eden regions:
		○ Before GC: 16 regions.
		○ After GC: 0 regions (all emptied).
		○ Available regions: Up to 40.
	• Survivor regions:
		○ Before GC: 0 regions.
		○ After GC: 4 regions.
		○ Maximum allowed: 4.
	• Old regions:
		○ No change (0 regions).
	• Humongous regions:
		○ None in use.
6. Metaspace
	• Metaspace: 10M->10M(105M):
		○ Metaspace usage remained unchanged during this collection.
		○ Current usage: 10 MB.
		○ Maximum capacity: 105 MB.
7. Pause Time
	• 22.0ms: The total pause time for this GC cycle was 22 milliseconds.

## Insights from the Log
	1. Performance:
		○ The GC pause time (22ms) is well within the target for applications aiming for low latency.
	2. Heap Usage:
		○ The Eden space was fully emptied, and surviving objects were promoted to the survivor regions.
		○ No promotion to the old generation, indicating most objects were short-lived.
	3. Concurrency:
		○ The GC efficiently utilized 8 parallel threads for evacuation.
	4. No Humongous Objects:
		○ No large objects (taking up an entire region) were detected during this GC cycle.

## Example Adjustments Based on Logs

If the GC Pause Time is High:
	1. Increase GC Threads:
		○ -XX:ParallelGCThreads=16: Increase the number of parallel threads for GC.
	2. Tune Region Size:
		○ -XX:G1HeapRegionSize=8m: Adjust region size to balance the number of regions processed.
	3. Optimize Pause Target:
		○ -XX:MaxGCPauseMillis=20: Set a stricter pause time goal.
  
If Frequent Minor GCs Occur:
	1. Increase Young Generation Size:
		○ -XX:NewRatio=2: Allocate more memory to the young generation.
	2. Survivor Space Tuning:
		○ -XX:SurvivorRatio=6: Adjust the Eden-to-Survivor space ratio
![image](https://github.com/user-attachments/assets/b5437478-5c02-44fd-98bc-19a0f002274f)
