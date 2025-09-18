# Java Collections Framework - Advanced Guide for Experienced Developers

## Table of Contents
1. [Collection Framework Architecture](#collection-framework-architecture)
2. [Thread-Safe Collections](#thread-safe-collections)
3. [Concurrent Collections Deep Dive](#concurrent-collections-deep-dive)
4. [Performance Analysis & Optimization](#performance-analysis--optimization)
5. [Custom Collection Implementations](#custom-collection-implementations)
6. [Memory Management & GC Impact](#memory-management--gc-impact)
7. [Scenario-Based Design Questions](#scenario-based-design-questions)
8. [Production Best Practices](#production-best-practices)

---

## Collection Framework Architecture

### Internal Data Structures and Time Complexities
```java
/**
 * CollectionInternals - Deep dive into internal workings of Java collections
 * 
 * Purpose: Understanding internal data structures for optimal collection choice
 * Key Focus: Time complexities, memory usage, and performance characteristics
 */
public class CollectionInternals {
    
    /**
     * ArrayList Internal Analysis
     * 
     * Data Structure: Resizable array (Object[])
     * Default Capacity: 10
     * Growth Strategy: 1.5x current capacity (oldCapacity + (oldCapacity >> 1))
     * 
     * Time Complexities:
     * - get(index): O(1)
     * - add(element): O(1) amortized, O(n) worst case (resize)
     * - add(index, element): O(n) - requires shifting
     * - remove(index): O(n) - requires shifting
     * - contains(element): O(n) - linear search
     */
    public void arrayListInternals() {
        List<String> list = new ArrayList<>();
        
        // Growth demonstration
        for (int i = 0; i < 15; i++) {
            list.add("Item-" + i);
            // Internal array grows: 0 -> 10 -> 15 elements
        }
        
        // Insertion at beginning - expensive O(n) operation
        long start = System.nanoTime();
        list.add(0, "First");  // Shifts all elements
        long insertTime = System.nanoTime() - start;
        
        System.out.println("ArrayList insertion at beginning: " + insertTime + " ns");
    }
    
    /**
     * LinkedList Internal Analysis
     * 
     * Data Structure: Doubly-linked list
     * Node Structure: {prev, data, next}
     * Memory Overhead: 24 bytes per node (8 bytes object header + 8 bytes prev + 8 bytes next)
     * 
     * Time Complexities:
     * - get(index): O(n) - traversal required
     * - add(element): O(1) - add to tail
     * - add(index, element): O(n) - traversal to index
     * - remove(index): O(n) - traversal to index
     * - addFirst/addLast: O(1)
     * - removeFirst/removeLast: O(1)
     */
    public void linkedListInternals() {
        LinkedList<String> list = new LinkedList<>();
        
        // Efficient operations
        long start = System.nanoTime();
        list.addFirst("First");  // O(1)
        list.addLast("Last");    // O(1)
        long efficientOps = System.nanoTime() - start;
        
        // Inefficient operation
        start = System.nanoTime();
        list.get(list.size() / 2);  // O(n/2) traversal
        long inefficientOp = System.nanoTime() - start;
        
        System.out.println("LinkedList efficient ops: " + efficientOps + " ns");
        System.out.println("LinkedList random access: " + inefficientOp + " ns");
    }
    
    /**
     * HashMap Internal Analysis
     * 
     * Data Structure: Array of buckets (Node<K,V>[])
     * Default Capacity: 16
     * Load Factor: 0.75
     * Collision Resolution: Chaining (LinkedList -> TreeNode when threshold=8)
     * Hash Function: key.hashCode() ^ (key.hashCode() >>> 16)
     * 
     * Time Complexities:
     * - get(key): O(1) average, O(log n) worst case (tree)
     * - put(key, value): O(1) average, O(log n) worst case
     * - remove(key): O(1) average, O(log n) worst case
     * 
     * Tree Conversion: When bucket size >= 8 and table size >= 64
     */
    public void hashMapInternals() {
        Map<String, String> map = new HashMap<>();
        
        // Force hash collisions for demonstration
        Map<CollisionKey, String> collisionMap = new HashMap<>();
        
        // All these keys will hash to same bucket
        for (int i = 0; i < 10; i++) {
            collisionMap.put(new CollisionKey(i), "Value-" + i);
        }
        
        // Measure performance with collisions
        long start = System.nanoTime();
        String value = collisionMap.get(new CollisionKey(5));
        long collisionAccess = System.nanoTime() - start;
        
        System.out.println("HashMap with collisions access time: " + collisionAccess + " ns");
    }
    
    /**
     * CollisionKey - Forces hash collisions for testing
     */
    static class CollisionKey {
        private final int value;
        
        public CollisionKey(int value) {
            this.value = value;
        }
        
        @Override
        public int hashCode() {
            return 1; // All instances have same hash code
        }
        
        @Override
        public boolean equals(Object obj) {
            return obj instanceof CollisionKey && ((CollisionKey) obj).value == this.value;
        }
    }
    
    /**
     * TreeMap Internal Analysis
     * 
     * Data Structure: Red-Black Tree
     * Ordering: Natural ordering or Comparator
     * Balance: Self-balancing binary search tree
     * 
     * Time Complexities:
     * - get(key): O(log n)
     * - put(key, value): O(log n)
     * - remove(key): O(log n)
     * - firstKey/lastKey: O(log n)
     * - subMap operations: O(log n)
     */
    public void treeMapInternals() {
        TreeMap<Integer, String> treeMap = new TreeMap<>();
        
        // Add elements in random order
        int[] randomOrder = {5, 2, 8, 1, 9, 3, 7, 4, 6};
        for (int key : randomOrder) {
            treeMap.put(key, "Value-" + key);
        }
        
        // Demonstrate sorted iteration
        System.out.println("TreeMap sorted keys: " + treeMap.keySet());
        
        // Range operations
        SortedMap<Integer, String> subMap = treeMap.subMap(3, 7);
        System.out.println("SubMap [3,7): " + subMap);
    }
}
```

---

## Thread-Safe Collections

### Synchronized Collections vs Concurrent Collections
```java
/**
 * ThreadSafetyComparison - Comprehensive comparison of thread-safety approaches
 * 
 * Purpose: Understanding different thread-safety mechanisms and their trade-offs
 * Categories:
 * 1. Legacy synchronized collections (Collections.synchronizedXxx)
 * 2. Modern concurrent collections (java.util.concurrent)
 * 3. Copy-on-write collections
 * 4. Lock-free data structures
 */
public class ThreadSafetyComparison {
    
    /**
     * SynchronizedCollections - Legacy approach with wrapper synchronization
     * 
     * Mechanism: Synchronized wrapper around existing collections
     * Synchronization: Method-level synchronization using intrinsic locks
     * Problems:
     * 1. Iteration requires external synchronization
     * 2. Compound operations not atomic
     * 3. Poor concurrent performance
     * 4. Potential for deadlocks
     */
    public void synchronizedCollectionsDemo() {
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
        
        // Problem 1: Iteration requires external synchronization
        syncList.add("Item1");
        syncList.add("Item2");
        
        // WRONG - Can throw ConcurrentModificationException
        // for (String item : syncList) { ... }
        
        // CORRECT - External synchronization required
        synchronized (syncList) {
            for (String item : syncList) {
                System.out.println("Processing: " + item);
                // Any modification here requires careful handling
            }
        }
        
        // Problem 2: Compound operations not atomic
        // WRONG - Race condition between contains and add
        if (!syncList.contains("Item3")) {
            syncList.add("Item3");  // Another thread might add it here
        }
        
        // CORRECT - Atomic compound operation
        synchronized (syncList) {
            if (!syncList.contains("Item3")) {
                syncList.add("Item3");
            }
        }
    }
    
    /**
     * ConcurrentCollections - Modern approach with fine-grained locking
     * 
     * Key Collections:
     * 1. ConcurrentHashMap - Segment-based locking (Java 7), CAS operations (Java 8+)
     * 2. ConcurrentLinkedQueue - Lock-free queue using CAS
     * 3. ConcurrentSkipListMap - Lock-free sorted map
     * 4. BlockingQueue implementations
     */
    public void concurrentCollectionsDemo() {
        // ConcurrentHashMap - Thread-safe without external synchronization
        ConcurrentHashMap<String, String> concurrentMap = new ConcurrentHashMap<>();
        
        // Safe iteration without external synchronization
        concurrentMap.put("key1", "value1");
        concurrentMap.put("key2", "value2");
        
        // Iteration is safe - uses snapshot of current state
        for (Map.Entry<String, String> entry : concurrentMap.entrySet()) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
            // Can modify map during iteration
            concurrentMap.put("key3", "value3");
        }
        
        // Atomic operations
        String oldValue = concurrentMap.putIfAbsent("key4", "value4");
        boolean removed = concurrentMap.remove("key1", "value1");
        String replaced = concurrentMap.replace("key2", "newValue2");
        
        // Compute operations (Java 8+)
        concurrentMap.compute("counter", (key, val) -> val == null ? "1" : String.valueOf(Integer.parseInt(val) + 1));
    }
    
    /**
     * CopyOnWriteCollections - Optimized for read-heavy workloads
     * 
     * Mechanism: Creates copy of underlying array on every write operation
     * Use Cases: Event listener lists, configuration data, caches
     * Trade-offs:
     * - Excellent read performance (no synchronization)
     * - Expensive write operations (O(n) time and space)
     * - Memory overhead
     * - Eventual consistency for iterations
     */
    public void copyOnWriteDemo() {
        CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
        CopyOnWriteArraySet<String> cowSet = new CopyOnWriteArraySet<>();
        
        // Populate initial data
        cowList.addAll(Arrays.asList("Item1", "Item2", "Item3"));
        
        // Read operations are very fast (no locking)
        long readStart = System.nanoTime();
        for (String item : cowList) {
            System.out.println("Reading: " + item);
            // Iterator reflects snapshot at creation time
            // Modifications during iteration don't affect this iterator
        }
        long readTime = System.nanoTime() - readStart;
        
        // Write operations are expensive
        long writeStart = System.nanoTime();
        cowList.add("Item4");  // Creates copy of entire array
        long writeTime = System.nanoTime() - writeStart;
        
        System.out.println("CopyOnWrite - Read time: " + readTime + " ns, Write time: " + writeTime + " ns");
        
        // Demonstration of snapshot iteration
        Iterator<String> iterator = cowList.iterator();
        cowList.add("Item5");  // Won't appear in this iterator
        
        while (iterator.hasNext()) {
            System.out.println("Snapshot item: " + iterator.next());
        }
    }
    
    /**
     * PerformanceComparison - Benchmarking different thread-safe approaches
     * 
     * Scenarios tested:
     * 1. High read, low write (CopyOnWrite optimal)
     * 2. Balanced read/write (ConcurrentHashMap optimal)
     * 3. High contention (lock-free structures preferred)
     */
    public void performanceComparison() {
        int numThreads = 10;
        int operations = 100000;
        
        // Test synchronized collections
        Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());
        long syncTime = benchmarkMap(syncMap, numThreads, operations, "SynchronizedMap");
        
        // Test concurrent collections
        ConcurrentHashMap<String, String> concurrentMap = new ConcurrentHashMap<>();
        long concurrentTime = benchmarkMap(concurrentMap, numThreads, operations, "ConcurrentHashMap");
        
        System.out.println("Performance comparison:");
        System.out.println("Synchronized: " + syncTime + " ms");
        System.out.println("Concurrent: " + concurrentTime + " ms");
        System.out.println("Speedup: " + (double) syncTime / concurrentTime + "x");
    }
    
    private long benchmarkMap(Map<String, String> map, int numThreads, int operations, String mapType) {
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        CountDownLatch latch = new CountDownLatch(numThreads);
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < numThreads; i++) {
            final int threadId = i;
            executor.submit(() -> {
                try {
                    for (int j = 0; j < operations; j++) {
                        String key = "thread" + threadId + "_key" + j;
                        map.put(key, "value" + j);
                        map.get(key);
                        
                        if (j % 100 == 0) {
                            map.remove(key);
                        }
                    }
                } finally {
                    latch.countDown();
                }
            });
        }
        
        try {
            latch.await();
            long endTime = System.currentTimeMillis();
            return endTime - startTime;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return -1;
        } finally {
            executor.shutdown();
        }
    }
}
```

---

## Concurrent Collections Deep Dive

### ConcurrentHashMap Internal Architecture
```java
/**
 * ConcurrentHashMapDeepDive - Advanced analysis of ConcurrentHashMap implementations
 * 
 * Purpose: Understanding internal mechanisms across Java versions
 * Evolution:
 * - Java 7: Segment-based locking (16 segments by default)
 * - Java 8+: CAS operations + synchronized nodes, tree conversion
 */
public class ConcurrentHashMapDeepDive {
    
    /**
     * Java 8+ ConcurrentHashMap Architecture
     * 
     * Key Features:
     * 1. Node-level locking instead of segment locking
     * 2. CAS operations for lock-free updates
     * 3. Tree conversion for hash collision mitigation
     * 4. Parallel operations support (forEach, reduce, search)
     * 
     * Internal Structure:
     * - Node<K,V>[] table: Main hash table
     * - TreeNode<K,V>: Red-black tree nodes for collision resolution
     * - ForwardingNode<K,V>: Special node used during resize
     */
    public void java8ConcurrentHashMapFeatures() {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        // Populate with sample data
        for (int i = 0; i < 1000; i++) {
            map.put("key" + i, i);
        }
        
        // Atomic compute operations (Java 8+)
        map.compute("counter", (key, val) -> (val == null) ? 1 : val + 1);
        map.computeIfAbsent("newKey", key -> key.length());
        map.computeIfPresent("key1", (key, val) -> val * 2);
        
        // Merge operation - atomic put or update
        map.merge("mergeKey", 10, Integer::sum);  // Will put 10
        map.merge("mergeKey", 5, Integer::sum);   // Will update to 15
        
        // Bulk operations with parallelism threshold
        int parallelismThreshold = 100;
        
        // Parallel forEach
        map.forEach(parallelismThreshold, (key, value) -> {
            // Process key-value pairs in parallel
            if (value > 500) {
                System.out.println("High value: " + key + " = " + value);
            }
        });
        
        // Parallel search - returns first match
        String foundKey = map.search(parallelismThreshold, (key, value) -> 
            value > 750 ? key : null);
        
        // Parallel reduce
        Integer sum = map.reduce(parallelismThreshold,
            (key, value) -> value,        // Transformer
            0,                            // Basis
            Integer::sum);                // Reducer
        
        System.out.println("Found key: " + foundKey);
        System.out.println("Sum of values: " + sum);
    }
    
    /**
     * ConcurrentHashMap vs HashMap Performance Analysis
     * 
     * Test scenarios:
     * 1. Single-threaded performance
     * 2. Multi-threaded read-heavy workload
     * 3. Multi-threaded write-heavy workload
     * 4. Mixed read-write workload
     */
    public void performanceAnalysis() {
        int mapSize = 100000;
        int numThreads = 8;
        int operationsPerThread = 50000;
        
        System.out.println("=== ConcurrentHashMap vs HashMap Performance ===");
        
        // Single-threaded test
        singleThreadedTest(mapSize);
        
        // Multi-threaded tests
        multiThreadedReadTest(mapSize, numThreads, operationsPerThread);
        multiThreadedWriteTest(mapSize, numThreads, operationsPerThread);
        multiThreadedMixedTest(mapSize, numThreads, operationsPerThread);
    }
    
    private void singleThreadedTest(int mapSize) {
        Map<Integer, String> hashMap = new HashMap<>();
        ConcurrentHashMap<Integer, String> concurrentMap = new ConcurrentHashMap<>();
        
        // Warm up
        for (int i = 0; i < 1000; i++) {
            hashMap.put(i, "value" + i);
            concurrentMap.put(i, "value" + i);
        }
        hashMap.clear();
        concurrentMap.clear();
        
        // Test HashMap
        long start = System.nanoTime();
        for (int i = 0; i < mapSize; i++) {
            hashMap.put(i, "value" + i);
        }
        for (int i = 0; i < mapSize; i++) {
            hashMap.get(i);
        }
        long hashMapTime = System.nanoTime() - start;
        
        // Test ConcurrentHashMap
        start = System.nanoTime();
        for (int i = 0; i < mapSize; i++) {
            concurrentMap.put(i, "value" + i);
        }
        for (int i = 0; i < mapSize; i++) {
            concurrentMap.get(i);
        }
        long concurrentMapTime = System.nanoTime() - start;
        
        System.out.println("Single-threaded performance:");
        System.out.println("HashMap: " + hashMapTime / 1_000_000 + " ms");
        System.out.println("ConcurrentHashMap: " + concurrentMapTime / 1_000_000 + " ms");
        System.out.println("Overhead: " + (double) concurrentMapTime / hashMapTime + "x\n");
    }
    
    private void multiThreadedReadTest(int mapSize, int numThreads, int operationsPerThread) {
        ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
        
        // Populate map
        for (int i = 0; i < mapSize; i++) {
            map.put(i, "value" + i);
        }
        
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        CountDownLatch latch = new CountDownLatch(numThreads);
        Random random = new Random();
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < numThreads; i++) {
            executor.submit(() -> {
                try {
                    for (int j = 0; j < operationsPerThread; j++) {
                        int key = random.nextInt(mapSize);
                        map.get(key);
                    }
                } finally {
                    latch.countDown();
                }
            });
        }
        
        try {
            latch.await();
            long endTime = System.currentTimeMillis();
            long totalOps = (long) numThreads * operationsPerThread;
            System.out.println("Read-heavy test (" + numThreads + " threads):");
            System.out.println("Time: " + (endTime - startTime) + " ms");
            System.out.println("Throughput: " + (totalOps * 1000 / (endTime - startTime)) + " ops/sec\n");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
    
    private void multiThreadedWriteTest(int mapSize, int numThreads, int operationsPerThread) {
        ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
        
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        CountDownLatch latch = new CountDownLatch(numThreads);
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < numThreads; i++) {
            final int threadId = i;
            executor.submit(() -> {
                try {
                    for (int j = 0; j < operationsPerThread; j++) {
                        int key = threadId * operationsPerThread + j;
                        map.put(key, "value" + key);
                    }
                } finally {
                    latch.countDown();
                }
            });
        }
        
        try {
            latch.await();
            long endTime = System.currentTimeMillis();
            long totalOps = (long) numThreads * operationsPerThread;
            System.out.println("Write-heavy test (" + numThreads + " threads):");
            System.out.println("Time: " + (endTime - startTime) + " ms");
            System.out.println("Throughput: " + (totalOps * 1000 / (endTime - startTime)) + " ops/sec");
            System.out.println("Final map size: " + map.size() + "\n");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
    
    private void multiThreadedMixedTest(int mapSize, int numThreads, int operationsPerThread) {
        ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
        
        // Pre-populate for mixed operations
        for (int i = 0; i < mapSize / 2; i++) {
            map.put(i, "value" + i);
        }
        
        ExecutorService executor = Executors.newFixedThreadPool(numThreads);
        CountDownLatch latch = new CountDownLatch(numThreads);
        Random random = new Random();
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < numThreads; i++) {
            final int threadId = i;
            executor.submit(() -> {
                try {
                    for (int j = 0; j < operationsPerThread; j++) {
                        if (random.nextDouble() < 0.7) {  // 70% reads
                            int key = random.nextInt(mapSize);
                            map.get(key);
                        } else {  // 30% writes
                            int key = threadId * operationsPerThread + j;
                            map.put(key, "value" + key);
                        }
                    }
                } finally {
                    latch.countDown();
                }
            });
        }
        
        try {
            latch.await();
            long endTime = System.currentTimeMillis();
            long totalOps = (long) numThreads * operationsPerThread;
            System.out.println("Mixed workload test (70% read, 30% write, " + numThreads + " threads):");
            System.out.println("Time: " + (endTime - startTime) + " ms");
            System.out.println("Throughput: " + (totalOps * 1000 / (endTime - startTime)) + " ops/sec");
            System.out.println("Final map size: " + map.size() + "\n");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
}
```

---

## Performance Analysis & Optimization

### Collection Performance Characteristics
```java
/**
 * CollectionPerformanceOptimization - Advanced performance analysis and optimization techniques
 * 
 * Purpose: Understanding performance characteristics for optimal collection selection
 * Focus Areas:
 * 1. Time complexity analysis in practice
 * 2. Memory overhead and GC impact
 * 3. Cache locality and CPU performance
 * 4. Scaling characteristics under load
 */
public class CollectionPerformanceOptimization {
    
    /**
     * ListPerformanceComparison - Detailed analysis of List implementations
     * 
     * Test scenarios:
     * 1. Random access patterns
     * 2. Sequential iteration
     * 3. Insertion/deletion at different positions
     * 4. Memory usage patterns
     */
    public void listPerformanceAnalysis() {
        int[] sizes = {1000, 10000, 100000, 1000000};
        
        System.out.println("=== List Performance Analysis ===");
        System.out.printf("%-10s %-15s %-15s %-15s %-15s%n", 
            "Size", "ArrayList-Add", "LinkedList-Add", "ArrayList-Get", "LinkedList-Get");
        
        for (int size : sizes) {
            // ArrayList performance
            List<Integer> arrayList = new ArrayList<>();
            long arrayAddTime = measureListAddition(arrayList, size);
            long arrayGetTime = measureRandomAccess(arrayList, size / 10);
            
            // LinkedList performance
            List<Integer> linkedList = new LinkedList<>();
            long linkedAddTime = measureListAddition(linkedList, size);
            long linkedGetTime = measureRandomAccess(linkedList, size / 10);
            
            System.out.printf("%-10d %-15d %-15d %-15d %-15d%n",
                size, arrayAddTime, linkedAddTime, arrayGetTime, linkedGetTime);
        }
    }
    
    /**
     * MapPerformanceComparison - Comprehensive map performance analysis
     * 
     * Factors tested:
     * 1. Hash distribution quality
     * 2. Load factor impact
     * 3. Initial capacity optimization
     * 4. Key type performance (String vs Integer vs Custom)
     */
    public void mapPerformanceAnalysis() {
        System.out.println("\n=== Map Performance Analysis ===");
        
        int mapSize = 100000;
        
        // Test different key types
        testMapWithDifferentKeys(mapSize);
        
        // Test load factor impact
        testLoadFactorImpact(mapSize);
        
        // Test initial capacity impact
        testInitialCapacityImpact(mapSize);
        
        // Test hash quality impact
        testHashQualityImpact(mapSize);
    }
    
    private void testMapWithDifferentKeys(int size) {
        System.out.println("\n--- Key Type Performance ---");
        
        // Integer keys
        Map<Integer, String> intMap = new HashMap<>();
        long intTime = measureMapOperations(intMap, size, i -> i, "Integer keys");
        
        // String keys
        Map<String, String> stringMap = new HashMap<>();
        long stringTime = measureMapOperations(stringMap, size, i -> "key" + i, "String keys");
        
        // Custom object keys
        Map<CustomKey, String> customMap = new HashMap<>();
        long customTime = measureMapOperations(customMap, size, i -> new CustomKey(i), "Custom keys");
        
        System.out.printf("Integer: %d ms, String: %d ms, Custom: %d ms%n", 
            intTime, stringTime, customTime);
    }
    
    private void testLoadFactorImpact(int size) {
        System.out.println("\n--- Load Factor Impact ---");
        
        float[] loadFactors = {0.5f, 0.75f, 0.9f, 1.0f};
        
        for (float loadFactor : loadFactors) {
            Map<Integer, String> map = new HashMap<>(16, loadFactor);
            long time = measureMapOperations(map, size, i -> i, "Load factor " + loadFactor);
            System.out.printf("Load factor %.2f: %d ms%n", loadFactor, time);
        }
    }
    
    private void testInitialCapacityImpact(int size) {
        System.out.println("\n--- Initial Capacity Impact ---");
        
        // Default capacity (16)
        Map<Integer, String> defaultMap = new HashMap<>();
        long defaultTime = measureMapOperations(defaultMap, size, i -> i, "Default capacity");
        
        // Optimal capacity (no resizing needed)
        int optimalCapacity = (int) (size / 0.75f) + 1;
        Map<Integer, String> optimalMap = new HashMap<>(optimalCapacity);
        long optimalTime = measureMapOperations(optimalMap, size, i -> i, "Optimal capacity");
        
        System.out.printf("Default: %d ms, Optimal: %d ms, Improvement: %.2fx%n",
            defaultTime, optimalTime, (double) defaultTime / optimalTime);
    }
    
    private void testHashQualityImpact(int size) {
        System.out.println("\n--- Hash Quality Impact ---");
        
        // Good hash distribution
        Map<GoodHashKey, String> goodHashMap = new HashMap<>();
        long goodTime = measureMapOperations(goodHashMap, size, i -> new GoodHashKey(i), "Good hash");
        
        // Poor hash distribution (many collisions)
        Map<PoorHashKey, String> poorHashMap = new HashMap<>();
        long poorTime = measureMapOperations(poorHashMap, size, i -> new PoorHashKey(i), "Poor hash");
        
        System.out.printf("Good hash: %d ms, Poor hash: %d ms, Degradation: %.2fx%n",
            goodTime, poorTime, (double) poorTime / goodTime);
    }
    
    /**
     * Memory Usage Analysis
     * 
     * Measures actual memory consumption of different collection types
     * Factors: Object overhead, pointer overhead, load factor waste
     */
    public void memoryUsageAnalysis() {
        System.out.println("\n=== Memory Usage Analysis ===");
        
        Runtime runtime = Runtime.getRuntime();
        int elementCount = 100000;
        
        // Measure ArrayList memory usage
        runtime.gc();
        long beforeArrayList = runtime.totalMemory() - runtime.freeMemory();
        List<String> arrayList = new ArrayList<>();
        for (int i = 0; i < elementCount; i++) {
            arrayList.add("Item" + i);
        }
        runtime.gc();
        long afterArrayList = runtime.totalMemory() - runtime.freeMemory();
        long arrayListMemory = afterArrayList - beforeArrayList;
        
        // Measure LinkedList memory usage
        runtime.gc();
        long beforeLinkedList = runtime.totalMemory() - runtime.freeMemory();
        List<String> linkedList = new LinkedList<>();
        for (int i = 0; i < elementCount; i++) {
            linkedList.add("Item" + i);
        }
        runtime.gc();
        long afterLinkedList = runtime.totalMemory() - runtime.freeMemory();
        long linkedListMemory = afterLinkedList - beforeLinkedList;
        
        // Measure HashMap memory usage
        runtime.gc();
        long beforeHashMap = runtime.totalMemory() - runtime.freeMemory();
        Map<String, String> hashMap = new HashMap<>();
        for (int i = 0; i < elementCount; i++) {
            hashMap.put("Key" + i, "Value" + i);
        }
        runtime.gc();
        long afterHashMap = runtime.totalMemory() - runtime.freeMemory();
        long hashMapMemory = afterHashMap - beforeHashMap;
        
        System.out.printf("ArrayList: %d bytes (%.2f bytes/element)%n", 
            arrayListMemory, (double) arrayListMemory / elementCount);
        System.out.printf("LinkedList: %d bytes (%.2f bytes/element)%n", 
            linkedListMemory, (double) linkedListMemory / elementCount);
        System.out.printf("HashMap: %d bytes (%.2f bytes/element)%n", 
            hashMapMemory, (double) hashMapMemory / elementCount);
    }
    
    // Utility methods
    private long measureListAddition(List<Integer> list, int size) {
        long start = System.nanoTime();
        for (int i = 0; i < size; i++) {
            list.add(i);
        }
        return (System.nanoTime() - start) / 1_000_000; // Convert to milliseconds
    }
    
    private long measureRandomAccess(List<Integer> list, int accessCount) {
        Random random = new Random(42); // Fixed seed for consistency
        long start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            int index = random.nextInt(list.size());
            list.get(index);
        }
        return (System.nanoTime() - start) / 1_000_000;
    }
    
    private <K> long measureMapOperations(Map<K, String> map, int size, 
                                         IntFunction<K> keyGenerator, String description) {
        long start = System.nanoTime();
        
        // Insert operations
        for (int i = 0; i < size; i++) {
            map.put(keyGenerator.apply(i), "value" + i);
        }
        
        // Lookup operations
        for (int i = 0; i < size / 10; i++) {
            map.get(keyGenerator.apply(i));
        }
        
        return (System.nanoTime() - start) / 1_000_000;
    }
    
    // Test key classes
    static class CustomKey {
        private final int value;
        
        public CustomKey(int value) {
            this.value = value;
        }
        
        @Override
        public int hashCode() {
            return Integer.hashCode(value);
        }
        
        @Override
        public boolean equals(Object obj) {
            return obj instanceof CustomKey && ((CustomKey) obj).value == this.value;
        }
    }
    
    static class GoodHashKey {
        private final int value;
        
        public GoodHashKey(int value) {
            this.value = value;
        }
        
        @Override
        public int hashCode() {
            // Good hash function - distributes values well
            int h = value;
            h ^= (h >>> 20) ^ (h >>> 12);
            return h ^ (h >>> 7) ^ (h >>> 4);
        }
        
        @Override
        public boolean equals(Object obj) {
            return obj instanceof GoodHashKey && ((GoodHashKey) obj).value == this.value;
        }
    }
    
    static class PoorHashKey {
        private final int value;
        
        public PoorHashKey(int value) {
            this.value = value;
        }
        
        @Override
        public int hashCode() {
            // Poor hash function - causes many collisions
            return value % 100;
        }
        
        @Override
        public boolean equals(Object obj) {
            return obj instanceof PoorHashKey && ((PoorHashKey) obj).value == this.value;
        }
    }
}
```

}
```

---

## Custom Collection Implementations

### Lock-Striping and Advanced Patterns
```java
/**
 * CustomThreadSafeCollections - Advanced thread-safe collection implementations
 * 
 * Purpose: Building specialized collections for high-performance scenarios
 * Techniques: Lock striping, copy-on-write variants, bounded collections
 */
public class CustomThreadSafeCollections {
    
    /**
     * StripedHashMap - High-concurrency map using lock striping
     * 
     * Benefits: Better scalability than single-lock approaches
     * Design: Multiple locks (stripes) to reduce contention
     */
    public static class StripedHashMap<K, V> {
        private final Node<K, V>[] buckets;
        private final ReentrantReadWriteLock[] locks;
        private final int numStripes;
        private volatile int size = 0;
        
        @SuppressWarnings("unchecked")
        public StripedHashMap(int capacity, int numStripes) {
            this.numStripes = numStripes;
            this.buckets = new Node[capacity];
            this.locks = new ReentrantReadWriteLock[numStripes];
            
            for (int i = 0; i < numStripes; i++) {
                locks[i] = new ReentrantReadWriteLock();
            }
        }
        
        private ReentrantReadWriteLock getLock(Object key) {
            int hash = key != null ? key.hashCode() : 0;
            int stripe = Math.abs(hash % numStripes);
            return locks[stripe];
        }
        
        public V put(K key, V value) {
            ReentrantReadWriteLock lock = getLock(key);
            lock.writeLock().lock();
            try {
                // Implementation details...
                return null; // Simplified for brevity
            } finally {
                lock.writeLock().unlock();
            }
        }
        
        public V get(Object key) {
            ReentrantReadWriteLock lock = getLock(key);
            lock.readLock().lock();
            try {
                // Implementation details...
                return null; // Simplified for brevity
            } finally {
                lock.readLock().unlock();
            }
        }
        
        private static class Node<K, V> {
            final K key;
            volatile V value;
            volatile Node<K, V> next;
            
            Node(K key, V value, Node<K, V> next) {
                this.key = key;
                this.value = value;
                this.next = next;
            }
        }
    }
}
```

---

## Scenario-Based Design Questions

### Real-World System Design Challenges

#### Question 1: High-Frequency Trading System
**Scenario**: Design collections for a trading system processing 1M+ operations/second
- **Requirements**: Ultra-low latency (<1ms), memory efficient, GC-friendly
- **Solution**: Custom order book with TreeMap for price levels, object pooling
- **Key Decisions**: ConcurrentSkipListMap vs TreeMap, memory allocation patterns

#### Question 2: Distributed Cache
**Scenario**: Multi-node cache with TTL expiration and LRU eviction
- **Requirements**: Handle millions of entries, thread-safe, memory monitoring
- **Solution**: ConcurrentHashMap + background cleanup + LRU tracking
- **Key Decisions**: Cleanup frequency, eviction policies, memory bounds

#### Question 3: Real-time Analytics
**Scenario**: Process streaming data with time-window aggregations
- **Requirements**: High throughput, time-based partitioning, memory efficient
- **Solution**: Ring buffer + concurrent aggregators + sliding windows
- **Key Decisions**: Buffer sizing, partition strategies, GC impact

#### Question 4: Social Media Feed
**Scenario**: Timeline generation for millions of users
- **Requirements**: Real-time updates, personalization, scalable storage
- **Solution**: Multi-level caching + priority queues + async processing
- **Key Decisions**: Cache layers, update propagation, consistency models

#### Question 5: GPS Navigation System
**Scenario**: Real-time route optimization with live traffic data
- **Requirements**: Sub-second route calculation, millions of road segments, dynamic updates
- **Solution**: Graph representation + spatial indexing + priority queues
- **Key Decisions**: Adjacency list vs matrix, spatial partitioning, Dijkstra optimization

#### Question 6: E-commerce Recommendation Engine
**Scenario**: Real-time product recommendations based on user behavior
- **Requirements**: Process user clicks, collaborative filtering, personalized results
- **Solution**: Bloom filters + MinHash + approximate algorithms
- **Key Decisions**: Memory vs accuracy trade-offs, update frequency, similarity algorithms

#### Question 7: Log Analytics Platform
**Scenario**: Process billions of log entries with complex queries
- **Requirements**: High ingestion rate, time-range queries, aggregations
- **Solution**: Time-partitioned maps + inverted indices + compression
- **Key Decisions**: Partitioning strategy, index granularity, retention policies

#### Question 8: Multiplayer Game Server
**Scenario**: Real-time game state management for thousands of concurrent players
- **Requirements**: Low latency updates, spatial queries, collision detection
- **Solution**: Spatial data structures + event queues + state snapshots
- **Key Decisions**: Grid vs tree partitioning, update propagation, consistency models

---

## Detailed Real-World Data Structure Problems

### Problem 1: GPS Navigation System - Route Optimization
```java
/**
 * GPSNavigationSystem - Real-time route optimization with live traffic data
 * 
 * Problem: Calculate optimal routes for millions of requests per hour
 * Constraints: Sub-second response time, dynamic traffic updates, memory efficient
 * 
 * Data Structure Decisions:
 * 1. Graph representation: Adjacency list vs adjacency matrix
 * 2. Spatial indexing: Grid vs QuadTree vs R-Tree
 * 3. Priority queue: Binary heap vs Fibonacci heap
 * 4. Caching: Route cache vs precomputed distances
 */
public class GPSNavigationSystem {
    
    /**
     * RoadNetwork - Graph representation optimized for navigation
     * 
     * Choice: Adjacency List over Adjacency Matrix
     * Reason: Sparse graph (each intersection connects to few others)
     * Memory: O(V + E) vs O(V²), where V=intersections, E=roads
     */
    public static class RoadNetwork {
        // Map from intersection ID to adjacent roads
        private final Map<Long, List<Road>> adjacencyList = new ConcurrentHashMap<>();
        
        // Spatial index for geographic queries
        private final SpatialIndex spatialIndex = new GridSpatialIndex(0.001); // ~100m grid
        
        // Traffic data cache with TTL
        private final Map<Long, TrafficData> trafficCache = new ConcurrentHashMap<>();
        
        /**
         * addRoad() - Adds bidirectional road to network
         */
        public void addRoad(long fromIntersection, long toIntersection, 
                           double distance, int speedLimit, Coordinate[] geometry) {
            
            Road road = new Road(fromIntersection, toIntersection, distance, speedLimit, geometry);
            
            // Add to adjacency list (bidirectional)
            adjacencyList.computeIfAbsent(fromIntersection, k -> new ArrayList<>()).add(road);
            adjacencyList.computeIfAbsent(toIntersection, k -> new ArrayList<>())
                .add(road.reverse());
            
            // Add to spatial index for geographic queries
            spatialIndex.addRoad(road);
        }
        
        /**
         * findOptimalRoute() - Dijkstra's algorithm with traffic-aware costs
         * 
         * Data Structure Choice: PriorityQueue with custom comparator
         * Alternative considered: Fibonacci heap (better theoretical complexity)
         * Decision: PriorityQueue sufficient for typical graph sizes
         */
        public Route findOptimalRoute(long startId, long endId) {
            // Priority queue for Dijkstra's algorithm
            PriorityQueue<RouteNode> openSet = new PriorityQueue<>(
                Comparator.comparingDouble(RouteNode::getTotalCost)
            );
            
            // Visited set for processed nodes
            Set<Long> closedSet = new HashSet<>();
            
            // Distance map for shortest known distances
            Map<Long, Double> distances = new HashMap<>();
            
            // Parent tracking for path reconstruction
            Map<Long, Long> parentMap = new HashMap<>();
            
            // Initialize start node
            openSet.offer(new RouteNode(startId, 0.0, estimateDistance(startId, endId)));
            distances.put(startId, 0.0);
            
            while (!openSet.isEmpty()) {
                RouteNode current = openSet.poll();
                
                if (current.getIntersectionId() == endId) {
                    return reconstructPath(parentMap, startId, endId, distances.get(endId));
                }
                
                if (closedSet.contains(current.getIntersectionId())) {
                    continue;
                }
                
                closedSet.add(current.getIntersectionId());
                
                // Explore neighbors
                List<Road> neighbors = adjacencyList.get(current.getIntersectionId());
                if (neighbors != null) {
                    for (Road road : neighbors) {
                        long neighborId = road.getToIntersection();
                        
                        if (closedSet.contains(neighborId)) {
                            continue;
                        }
                        
                        // Calculate cost with current traffic
                        double edgeCost = calculateTrafficAwareCost(road);
                        double newDistance = distances.get(current.getIntersectionId()) + edgeCost;
                        
                        if (newDistance < distances.getOrDefault(neighborId, Double.MAX_VALUE)) {
                            distances.put(neighborId, newDistance);
                            parentMap.put(neighborId, current.getIntersectionId());
                            
                            double heuristic = estimateDistance(neighborId, endId);
                            openSet.offer(new RouteNode(neighborId, newDistance, newDistance + heuristic));
                        }
                    }
                }
            }
            
            return null; // No route found
        }
        
        /**
         * findNearbyIntersections() - Spatial query for nearby intersections
         * 
         * Data Structure Choice: Grid-based spatial index
         * Alternatives: QuadTree, R-Tree, KD-Tree
         * Decision: Grid optimal for uniform distribution and range queries
         */
        public List<Long> findNearbyIntersections(Coordinate location, double radiusKm) {
            return spatialIndex.findNearby(location, radiusKm);
        }
        
        /**
         * updateTrafficData() - Real-time traffic updates
         * 
         * Data Structure Choice: ConcurrentHashMap with TTL cleanup
         * Pattern: Write-through cache with background cleanup
         */
        public void updateTrafficData(long roadId, double currentSpeed, int congestionLevel) {
            TrafficData data = new TrafficData(currentSpeed, congestionLevel, System.currentTimeMillis());
            trafficCache.put(roadId, data);
            
            // Trigger cleanup if cache is getting large
            if (trafficCache.size() > 100000) {
                cleanupExpiredTrafficData();
            }
        }
        
        private double calculateTrafficAwareCost(Road road) {
            TrafficData traffic = trafficCache.get(road.getId());
            if (traffic == null || traffic.isExpired()) {
                // No traffic data, use speed limit
                return road.getDistance() / road.getSpeedLimit();
            }
            
            // Adjust cost based on current traffic
            double adjustedSpeed = Math.min(road.getSpeedLimit(), traffic.getCurrentSpeed());
            return road.getDistance() / Math.max(adjustedSpeed, 1.0); // Avoid division by zero
        }
        
        private double estimateDistance(long fromId, long toId) {
            // Simplified heuristic - in practice, use geographic coordinates
            return Math.abs(toId - fromId) * 0.001; // Rough estimate
        }
        
        private Route reconstructPath(Map<Long, Long> parentMap, long start, long end, double totalCost) {
            List<Long> path = new ArrayList<>();
            long current = end;
            
            while (current != start) {
                path.add(current);
                current = parentMap.get(current);
            }
            path.add(start);
            Collections.reverse(path);
            
            return new Route(path, totalCost);
        }
        
        private void cleanupExpiredTrafficData() {
            long expiredThreshold = System.currentTimeMillis() - 300000; // 5 minutes
            trafficCache.entrySet().removeIf(entry -> 
                entry.getValue().getTimestamp() < expiredThreshold);
        }
    }
    
    /**
     * SpatialIndex - Grid-based spatial indexing for geographic queries
     * 
     * Choice Rationale:
     * - Grid: O(1) insertion, O(k) query where k = items in nearby cells
     * - QuadTree: O(log n) insertion, O(log n + k) query
     * - Decision: Grid chosen for uniform road distribution and range query optimization
     */
    public static class GridSpatialIndex implements SpatialIndex {
        private final double cellSize;
        private final Map<GridCell, Set<Road>> grid = new ConcurrentHashMap<>();
        
        public GridSpatialIndex(double cellSize) {
            this.cellSize = cellSize;
        }
        
        @Override
        public void addRoad(Road road) {
            // Add road to all grid cells it intersects
            Set<GridCell> intersectedCells = getIntersectedCells(road);
            for (GridCell cell : intersectedCells) {
                grid.computeIfAbsent(cell, k -> ConcurrentHashMap.newKeySet()).add(road);
            }
        }
        
        @Override
        public List<Long> findNearby(Coordinate location, double radiusKm) {
            Set<Long> nearbyIntersections = new HashSet<>();
            
            // Calculate grid cells within radius
            int cellRadius = (int) Math.ceil(radiusKm / cellSize);
            GridCell centerCell = new GridCell(location, cellSize);
            
            for (int dx = -cellRadius; dx <= cellRadius; dx++) {
                for (int dy = -cellRadius; dy <= cellRadius; dy++) {
                    GridCell cell = new GridCell(centerCell.x + dx, centerCell.y + dy);
                    Set<Road> roads = grid.get(cell);
                    
                    if (roads != null) {
                        for (Road road : roads) {
                            // Check actual distance
                            if (road.distanceTo(location) <= radiusKm) {
                                nearbyIntersections.add(road.getFromIntersection());
                                nearbyIntersections.add(road.getToIntersection());
                            }
                        }
                    }
                }
            }
            
            return new ArrayList<>(nearbyIntersections);
        }
        
        private Set<GridCell> getIntersectedCells(Road road) {
            Set<GridCell> cells = new HashSet<>();
            Coordinate[] geometry = road.getGeometry();
            
            for (Coordinate coord : geometry) {
                cells.add(new GridCell(coord, cellSize));
            }
            
            return cells;
        }
    }
    
    // Supporting classes
    public interface SpatialIndex {
        void addRoad(Road road);
        List<Long> findNearby(Coordinate location, double radiusKm);
    }
    
    public static class Road {
        private final long id;
        private final long fromIntersection;
        private final long toIntersection;
        private final double distance;
        private final int speedLimit;
        private final Coordinate[] geometry;
        
        public Road(long fromIntersection, long toIntersection, double distance, 
                   int speedLimit, Coordinate[] geometry) {
            this.id = generateId(fromIntersection, toIntersection);
            this.fromIntersection = fromIntersection;
            this.toIntersection = toIntersection;
            this.distance = distance;
            this.speedLimit = speedLimit;
            this.geometry = geometry;
        }
        
        public Road reverse() {
            // Create reverse geometry
            Coordinate[] reverseGeometry = new Coordinate[geometry.length];
            for (int i = 0; i < geometry.length; i++) {
                reverseGeometry[i] = geometry[geometry.length - 1 - i];
            }
            
            return new Road(toIntersection, fromIntersection, distance, speedLimit, reverseGeometry);
        }
        
        public double distanceTo(Coordinate point) {
            // Calculate minimum distance from point to road geometry
            double minDistance = Double.MAX_VALUE;
            for (Coordinate coord : geometry) {
                double dist = Math.sqrt(Math.pow(coord.x - point.x, 2) + Math.pow(coord.y - point.y, 2));
                minDistance = Math.min(minDistance, dist);
            }
            return minDistance;
        }
        
        private long generateId(long from, long to) {
            return (Math.min(from, to) << 32) | Math.max(from, to);
        }
        
        // Getters
        public long getId() { return id; }
        public long getFromIntersection() { return fromIntersection; }
        public long getToIntersection() { return toIntersection; }
        public double getDistance() { return distance; }
        public int getSpeedLimit() { return speedLimit; }
        public Coordinate[] getGeometry() { return geometry; }
    }
    
    public static class Coordinate {
        public final double x, y; // Simplified 2D coordinates
        
        public Coordinate(double x, double y) {
            this.x = x;
            this.y = y;
        }
    }
    
    public static class GridCell {
        public final int x, y;
        
        public GridCell(int x, int y) {
            this.x = x;
            this.y = y;
        }
        
        public GridCell(Coordinate coord, double cellSize) {
            this.x = (int) Math.floor(coord.x / cellSize);
            this.y = (int) Math.floor(coord.y / cellSize);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            GridCell gridCell = (GridCell) obj;
            return x == gridCell.x && y == gridCell.y;
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(x, y);
        }
    }
    
    public static class RouteNode {
        private final long intersectionId;
        private final double costFromStart;
        private final double totalCost; // costFromStart + heuristic
        
        public RouteNode(long intersectionId, double costFromStart, double totalCost) {
            this.intersectionId = intersectionId;
            this.costFromStart = costFromStart;
            this.totalCost = totalCost;
        }
        
        public long getIntersectionId() { return intersectionId; }
        public double getCostFromStart() { return costFromStart; }
        public double getTotalCost() { return totalCost; }
    }
    
    public static class TrafficData {
        private final double currentSpeed;
        private final int congestionLevel;
        private final long timestamp;
        
        public TrafficData(double currentSpeed, int congestionLevel, long timestamp) {
            this.currentSpeed = currentSpeed;
            this.congestionLevel = congestionLevel;
            this.timestamp = timestamp;
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() - timestamp > 300000; // 5 minutes
        }
        
        public double getCurrentSpeed() { return currentSpeed; }
        public int getCongestionLevel() { return congestionLevel; }
        public long getTimestamp() { return timestamp; }
    }
    
    public static class Route {
        private final List<Long> intersections;
        private final double totalCost;
        
        public Route(List<Long> intersections, double totalCost) {
            this.intersections = intersections;
            this.totalCost = totalCost;
        }
        
        public List<Long> getIntersections() { return intersections; }
        public double getTotalCost() { return totalCost; }
    }
}
```

### Problem 2: E-commerce Recommendation Engine
```java
/**
 * RecommendationEngine - Real-time product recommendations
 * 
 * Problem: Generate personalized recommendations for millions of users
 * Constraints: Real-time response (<100ms), memory efficient, high accuracy
 * 
 * Data Structure Decisions:
 * 1. User similarity: MinHash for Jaccard similarity approximation
 * 2. Item filtering: Bloom filter for negative lookups
 * 3. Top-K recommendations: Bounded priority queue
 * 4. Feature vectors: Sparse representation with maps
 */
public class RecommendationEngine {
    
    /**
     * UserSimilarityIndex - MinHash-based similarity calculation
     * 
     * Choice: MinHash over exact Jaccard similarity
     * Reason: O(1) similarity estimation vs O(|union|) exact calculation
     * Trade-off: Small accuracy loss for massive performance gain
     */
    public static class UserSimilarityIndex {
        private final int numHashes;
        private final Map<Long, int[]> userMinHashes = new ConcurrentHashMap<>();
        private final Random[] hashFunctions;
        
        public UserSimilarityIndex(int numHashes) {
            this.numHashes = numHashes;
            this.hashFunctions = new Random[numHashes];
            
            // Initialize hash functions with different seeds
            for (int i = 0; i < numHashes; i++) {
                hashFunctions[i] = new Random(i);
            }
        }
        
        /**
         * updateUserProfile() - Updates user's MinHash signature
         * 
         * Time Complexity: O(items * numHashes)
         * Space Complexity: O(users * numHashes)
         */
        public void updateUserProfile(long userId, Set<Long> purchasedItems) {
            int[] minHashes = new int[numHashes];
            Arrays.fill(minHashes, Integer.MAX_VALUE);
            
            // Calculate MinHash signature
            for (long itemId : purchasedItems) {
                for (int i = 0; i < numHashes; i++) {
                    int hash = hash(itemId, i);
                    minHashes[i] = Math.min(minHashes[i], hash);
                }
            }
            
            userMinHashes.put(userId, minHashes);
        }
        
        /**
         * calculateSimilarity() - Estimates Jaccard similarity using MinHash
         * 
         * Time Complexity: O(numHashes) vs O(|union|) for exact calculation
         * Accuracy: Estimation error decreases with more hash functions
         */
        public double calculateSimilarity(long userId1, long userId2) {
            int[] hashes1 = userMinHashes.get(userId1);
            int[] hashes2 = userMinHashes.get(userId2);
            
            if (hashes1 == null || hashes2 == null) {
                return 0.0;
            }
            
            int matches = 0;
            for (int i = 0; i < numHashes; i++) {
                if (hashes1[i] == hashes2[i]) {
                    matches++;
                }
            }
            
            return (double) matches / numHashes;
        }
        
        /**
         * findSimilarUsers() - Find most similar users using MinHash
         */
        public List<SimilarUser> findSimilarUsers(long userId, int topK) {
            int[] targetHashes = userMinHashes.get(userId);
            if (targetHashes == null) {
                return Collections.emptyList();
            }
            
            // Bounded priority queue for top-K similar users
            PriorityQueue<SimilarUser> topSimilar = new PriorityQueue<>(
                topK, Comparator.comparingDouble(SimilarUser::getSimilarity)
            );
            
            for (Map.Entry<Long, int[]> entry : userMinHashes.entrySet()) {
                long otherUserId = entry.getKey();
                if (otherUserId == userId) continue;
                
                double similarity = calculateSimilarity(userId, otherUserId);
                
                if (topSimilar.size() < topK) {
                    topSimilar.offer(new SimilarUser(otherUserId, similarity));
                } else if (similarity > topSimilar.peek().getSimilarity()) {
                    topSimilar.poll();
                    topSimilar.offer(new SimilarUser(otherUserId, similarity));
                }
            }
            
            // Convert to list and reverse for descending order
            List<SimilarUser> result = new ArrayList<>(topSimilar);
            result.sort(Comparator.comparingDouble(SimilarUser::getSimilarity).reversed());
            return result;
        }
        
        private int hash(long itemId, int hashIndex) {
            hashFunctions[hashIndex].setSeed(itemId + hashIndex);
            return hashFunctions[hashIndex].nextInt();
        }
    }
    
    /**
     * ItemFilterIndex - Bloom filter for efficient negative lookups
     * 
     * Choice: Bloom filter over HashSet for item existence checks
     * Reason: Space efficiency when dealing with millions of items
     * Trade-off: False positives possible, but no false negatives
     */
    public static class ItemFilterIndex {
        private final BitSet bitSet;
        private final int size;
        private final int numHashFunctions;
        private final Random[] hashFunctions;
        
        public ItemFilterIndex(int expectedItems, double falsePositiveRate) {
            // Calculate optimal Bloom filter parameters
            this.size = (int) (-expectedItems * Math.log(falsePositiveRate) / Math.pow(Math.log(2), 2));
            this.numHashFunctions = (int) (size / expectedItems * Math.log(2));
            this.bitSet = new BitSet(size);
            this.hashFunctions = new Random[numHashFunctions];
            
            // Initialize hash functions
            for (int i = 0; i < numHashFunctions; i++) {
                hashFunctions[i] = new Random(i);
            }
        }
        
        /**
         * addItem() - Adds item to Bloom filter
         * 
         * Time Complexity: O(numHashFunctions)
         * Space Complexity: O(size) bits total
         */
        public void addItem(long itemId) {
            for (int i = 0; i < numHashFunctions; i++) {
                int hash = hash(itemId, i) % size;
                bitSet.set(Math.abs(hash));
            }
        }
        
        /**
         * mightContain() - Checks if item might be in set
         * 
         * Returns: false = definitely not in set, true = might be in set
         * Use case: Skip expensive database lookups for non-existent items
         */
        public boolean mightContain(long itemId) {
            for (int i = 0; i < numHashFunctions; i++) {
                int hash = hash(itemId, i) % size;
                if (!bitSet.get(Math.abs(hash))) {
                    return false; // Definitely not in set
                }
            }
            return true; // Might be in set
        }
        
        private int hash(long itemId, int hashIndex) {
            hashFunctions[hashIndex].setSeed(itemId + hashIndex);
            return hashFunctions[hashIndex].nextInt();
        }
    }
    
    /**
     * RecommendationGenerator - Combines similarity and filtering for recommendations
     */
    public static class RecommendationGenerator {
        private final UserSimilarityIndex similarityIndex;
        private final ItemFilterIndex itemFilter;
        private final Map<Long, Set<Long>> userPurchases = new ConcurrentHashMap<>();
        private final Map<Long, ItemMetadata> itemMetadata = new ConcurrentHashMap<>();
        
        public RecommendationGenerator(int expectedUsers, int expectedItems) {
            this.similarityIndex = new UserSimilarityIndex(128); // 128 hash functions
            this.itemFilter = new ItemFilterIndex(expectedItems, 0.01); // 1% false positive rate
        }
        
        /**
         * recordPurchase() - Records user purchase and updates indices
         */
        public void recordPurchase(long userId, long itemId, double rating) {
            // Update user purchases
            userPurchases.computeIfAbsent(userId, k -> ConcurrentHashMap.newKeySet()).add(itemId);
            
            // Update similarity index
            similarityIndex.updateUserProfile(userId, userPurchases.get(userId));
            
            // Add to item filter
            itemFilter.addItem(itemId);
            
            // Update item metadata
            itemMetadata.computeIfAbsent(itemId, k -> new ItemMetadata(k)).addRating(rating);
        }
        
        /**
         * generateRecommendations() - Generate top-K recommendations for user
         * 
         * Algorithm:
         * 1. Find similar users using MinHash
         * 2. Collect items purchased by similar users
         * 3. Filter out items already purchased by target user
         * 4. Score items based on similarity weights and item ratings
         * 5. Return top-K scored items
         */
        public List<Recommendation> generateRecommendations(long userId, int topK) {
            Set<Long> userItems = userPurchases.get(userId);
            if (userItems == null) {
                return Collections.emptyList();
            }
            
            // Find similar users
            List<SimilarUser> similarUsers = similarityIndex.findSimilarUsers(userId, 50);
            
            // Collect candidate items with scores
            Map<Long, Double> candidateScores = new HashMap<>();
            
            for (SimilarUser similarUser : similarUsers) {
                Set<Long> similarUserItems = userPurchases.get(similarUser.getUserId());
                if (similarUserItems == null) continue;
                
                for (long itemId : similarUserItems) {
                    // Skip items already purchased by target user
                    if (userItems.contains(itemId)) continue;
                    
                    // Quick filter using Bloom filter (skip if definitely not in catalog)
                    if (!itemFilter.mightContain(itemId)) continue;
                    
                    // Calculate item score based on user similarity and item rating
                    ItemMetadata metadata = itemMetadata.get(itemId);
                    if (metadata != null) {
                        double score = similarUser.getSimilarity() * metadata.getAverageRating();
                        candidateScores.merge(itemId, score, Double::sum);
                    }
                }
            }
            
            // Get top-K recommendations
            return candidateScores.entrySet().stream()
                .sorted(Map.Entry.<Long, Double>comparingByValue().reversed())
                .limit(topK)
                .map(entry -> new Recommendation(entry.getKey(), entry.getValue()))
                .collect(Collectors.toList());
        }
    }
    
    // Supporting classes
    public static class SimilarUser {
        private final long userId;
        private final double similarity;
        
        public SimilarUser(long userId, double similarity) {
            this.userId = userId;
            this.similarity = similarity;
        }
        
        public long getUserId() { return userId; }
        public double getSimilarity() { return similarity; }
    }
    
    public static class ItemMetadata {
        private final long itemId;
        private double totalRating = 0.0;
        private int ratingCount = 0;
        
        public ItemMetadata(long itemId) {
            this.itemId = itemId;
        }
        
        public synchronized void addRating(double rating) {
            totalRating += rating;
            ratingCount++;
        }
        
        public synchronized double getAverageRating() {
            return ratingCount > 0 ? totalRating / ratingCount : 0.0;
        }
        
        public long getItemId() { return itemId; }
    }
    
    public static class Recommendation {
        private final long itemId;
        private final double score;
        
        public Recommendation(long itemId, double score) {
            this.itemId = itemId;
            this.score = score;
        }
        
        public long getItemId() { return itemId; }
        public double getScore() { return score; }
        
        @Override
        public String toString() {
            return String.format("Recommendation{itemId=%d, score=%.3f}", itemId, score);
        }
    }
}
```

### Problem 3: Log Analytics Platform - Time-Series Data Processing
```java
/**
 * LogAnalyticsPlatform - Process billions of log entries with complex queries
 * 
 * Problem: Handle high ingestion rates with efficient time-range queries
 * Constraints: Billions of entries, complex aggregations, memory efficiency
 * 
 * Data Structure Decisions:
 * 1. Time partitioning: TreeMap for time-ordered partitions
 * 2. Inverted index: Map<String, Set<LogEntry>> for field searches
 * 3. Aggregation cache: LRU cache for frequent queries
 * 4. Compression: Custom encoding for repeated values
 */
public class LogAnalyticsPlatform {
    
    /**
     * TimePartitionedLogStore - Efficiently store and query time-series log data
     * 
     * Strategy: Partition logs by time windows for efficient range queries
     * Benefits: O(log partitions) for time range queries vs O(total logs)
     */
    public static class TimePartitionedLogStore {
        // Time-ordered partitions (key = partition start time)
        private final NavigableMap<Long, LogPartition> timePartitions = new ConcurrentSkipListMap<>();
        
        // Inverted indices for fast field searches
        private final Map<String, InvertedIndex> fieldIndices = new ConcurrentHashMap<>();
        
        // Query result cache with LRU eviction
        private final Map<String, QueryResult> queryCache = new LinkedHashMap<String, QueryResult>(1000, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<String, QueryResult> eldest) {
                return size() > 1000;
            }
        };
        
        private final long partitionDurationMs;
        
        public TimePartitionedLogStore(long partitionDurationMs) {
            this.partitionDurationMs = partitionDurationMs; // e.g., 1 hour = 3600000ms
        }
        
        /**
         * addLogEntry() - Adds log entry to appropriate time partition
         * 
         * Time Complexity: O(log partitions) for partition lookup + O(1) for insertion
         * Space Complexity: O(entries) with compression for repeated values
         */
        public void addLogEntry(LogEntry entry) {
            long partitionKey = getPartitionKey(entry.getTimestamp());
            
            // Get or create partition
            LogPartition partition = timePartitions.computeIfAbsent(partitionKey, 
                k -> new LogPartition(k, k + partitionDurationMs));
            
            // Add to partition
            partition.addEntry(entry);
            
            // Update inverted indices
            updateInvertedIndices(entry);
            
            // Clear related cached queries
            invalidateRelatedQueries(entry.getTimestamp());
        }
        
        /**
         * queryTimeRange() - Query logs within specific time range
         * 
         * Algorithm:
         * 1. Find overlapping partitions using NavigableMap
         * 2. Query each partition in parallel
         * 3. Merge and sort results
         * 4. Apply additional filters
         */
        public List<LogEntry> queryTimeRange(long startTime, long endTime, LogFilter filter) {
            String cacheKey = generateCacheKey(startTime, endTime, filter);
            QueryResult cached = queryCache.get(cacheKey);
            if (cached != null && !cached.isExpired()) {
                return cached.getResults();
            }
            
            // Find overlapping partitions
            long startPartition = getPartitionKey(startTime);
            long endPartition = getPartitionKey(endTime);
            
            NavigableMap<Long, LogPartition> overlappingPartitions = 
                timePartitions.subMap(startPartition, true, endPartition, true);
            
            // Query partitions in parallel
            List<CompletableFuture<List<LogEntry>>> futures = overlappingPartitions.values()
                .stream()
                .map(partition -> CompletableFuture.supplyAsync(() -> 
                    partition.queryRange(startTime, endTime, filter)))
                .collect(Collectors.toList());
            
            // Merge results
            List<LogEntry> allResults = futures.stream()
                .map(CompletableFuture::join)
                .flatMap(List::stream)
                .sorted(Comparator.comparingLong(LogEntry::getTimestamp))
                .collect(Collectors.toList());
            
            // Cache result
            queryCache.put(cacheKey, new QueryResult(allResults, System.currentTimeMillis()));
            
            return allResults;
        }
        
        /**
         * aggregateByField() - Perform aggregations over time ranges
         * 
         * Example: Count log levels in last hour, average response times, etc.
         */
        public Map<String, Long> aggregateByField(long startTime, long endTime, 
                                                  String fieldName, AggregationType aggType) {
            Map<String, Long> aggregation = new ConcurrentHashMap<>();
            
            // Find relevant partitions
            long startPartition = getPartitionKey(startTime);
            long endPartition = getPartitionKey(endTime);
            
            NavigableMap<Long, LogPartition> partitions = 
                timePartitions.subMap(startPartition, true, endPartition, true);
            
            // Parallel aggregation across partitions
            partitions.values().parallelStream()
                .forEach(partition -> {
                    Map<String, Long> partitionAgg = partition.aggregate(startTime, endTime, fieldName, aggType);
                    partitionAgg.forEach((key, value) -> 
                        aggregation.merge(key, value, Long::sum));
                });
            
            return aggregation;
        }
        
        /**
         * searchByField() - Fast field-based searches using inverted index
         */
        public List<LogEntry> searchByField(String fieldName, String fieldValue, 
                                          long startTime, long endTime) {
            InvertedIndex index = fieldIndices.get(fieldName);
            if (index == null) {
                return Collections.emptyList();
            }
            
            // Get candidate entries from inverted index
            Set<LogEntry> candidates = index.getEntries(fieldValue);
            
            // Filter by time range
            return candidates.stream()
                .filter(entry -> entry.getTimestamp() >= startTime && entry.getTimestamp() <= endTime)
                .sorted(Comparator.comparingLong(LogEntry::getTimestamp))
                .collect(Collectors.toList());
        }
        
        private long getPartitionKey(long timestamp) {
            return (timestamp / partitionDurationMs) * partitionDurationMs;
        }
        
        private void updateInvertedIndices(LogEntry entry) {
            for (Map.Entry<String, String> field : entry.getFields().entrySet()) {
                fieldIndices.computeIfAbsent(field.getKey(), k -> new InvertedIndex())
                    .addEntry(field.getValue(), entry);
            }
        }
        
        private void invalidateRelatedQueries(long timestamp) {
            // Simple strategy: clear all cache (could be more sophisticated)
            queryCache.clear();
        }
        
        private String generateCacheKey(long startTime, long endTime, LogFilter filter) {
            return String.format("range_%d_%d_%s", startTime, endTime, filter.hashCode());
        }
    }
    
    /**
     * LogPartition - Individual time partition with optimized storage
     */
    public static class LogPartition {
        private final long startTime;
        private final long endTime;
        
        // Compressed storage for log entries
        private final List<LogEntry> entries = new ArrayList<>();
        
        // Pre-computed aggregations for common queries
        private final Map<String, Map<String, AtomicLong>> fieldCounts = new ConcurrentHashMap<>();
        
        public LogPartition(long startTime, long endTime) {
            this.startTime = startTime;
            this.endTime = endTime;
        }
        
        public synchronized void addEntry(LogEntry entry) {
            entries.add(entry);
            
            // Update field counts for fast aggregations
            for (Map.Entry<String, String> field : entry.getFields().entrySet()) {
                fieldCounts.computeIfAbsent(field.getKey(), k -> new ConcurrentHashMap<>())
                    .computeIfAbsent(field.getValue(), k -> new AtomicLong(0))
                    .incrementAndGet();
            }
        }
        
        public List<LogEntry> queryRange(long startTime, long endTime, LogFilter filter) {
            return entries.stream()
                .filter(entry -> entry.getTimestamp() >= startTime && entry.getTimestamp() <= endTime)
                .filter(filter::matches)
                .collect(Collectors.toList());
        }
        
        public Map<String, Long> aggregate(long startTime, long endTime, 
                                         String fieldName, AggregationType aggType) {
            if (aggType == AggregationType.COUNT && 
                startTime <= this.startTime && endTime >= this.endTime) {
                // Use pre-computed counts for full partition queries
                Map<String, AtomicLong> counts = fieldCounts.get(fieldName);
                if (counts != null) {
                    return counts.entrySet().stream()
                        .collect(Collectors.toMap(
                            Map.Entry::getKey,
                            entry -> entry.getValue().get()
                        ));
                }
            }
            
            // Fallback to scanning entries
            Map<String, Long> result = new HashMap<>();
            for (LogEntry entry : entries) {
                if (entry.getTimestamp() >= startTime && entry.getTimestamp() <= endTime) {
                    String fieldValue = entry.getFields().get(fieldName);
                    if (fieldValue != null) {
                        result.merge(fieldValue, 1L, Long::sum);
                    }
                }
            }
            return result;
        }
        
        public long getStartTime() { return startTime; }
        public long getEndTime() { return endTime; }
        public int getEntryCount() { return entries.size(); }
    }
    
    /**
     * InvertedIndex - Fast field-based searches
     * 
     * Structure: fieldValue -> Set<LogEntry>
     * Trade-off: Memory usage for query speed
     */
    public static class InvertedIndex {
        private final Map<String, Set<LogEntry>> index = new ConcurrentHashMap<>();
        
        public void addEntry(String fieldValue, LogEntry entry) {
            index.computeIfAbsent(fieldValue, k -> ConcurrentHashMap.newKeySet()).add(entry);
        }
        
        public Set<LogEntry> getEntries(String fieldValue) {
            return index.getOrDefault(fieldValue, Collections.emptySet());
        }
        
        public Set<String> getValues() {
            return index.keySet();
        }
        
        public int getUniqueValueCount() {
            return index.size();
        }
    }
    
    // Supporting classes
    public static class LogEntry {
        private final long timestamp;
        private final String message;
        private final Map<String, String> fields;
        private final String level;
        
        public LogEntry(long timestamp, String level, String message, Map<String, String> fields) {
            this.timestamp = timestamp;
            this.level = level;
            this.message = message;
            this.fields = new HashMap<>(fields);
        }
        
        public long getTimestamp() { return timestamp; }
        public String getMessage() { return message; }
        public Map<String, String> getFields() { return fields; }
        public String getLevel() { return level; }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            LogEntry logEntry = (LogEntry) obj;
            return timestamp == logEntry.timestamp && Objects.equals(message, logEntry.message);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(timestamp, message);
        }
    }
    
    public interface LogFilter {
        boolean matches(LogEntry entry);
        
        static LogFilter levelEquals(String level) {
            return entry -> level.equals(entry.getLevel());
        }
        
        static LogFilter fieldEquals(String fieldName, String value) {
            return entry -> value.equals(entry.getFields().get(fieldName));
        }
        
        static LogFilter messageContains(String substring) {
            return entry -> entry.getMessage().contains(substring);
        }
        
        default LogFilter and(LogFilter other) {
            return entry -> this.matches(entry) && other.matches(entry);
        }
        
        default LogFilter or(LogFilter other) {
            return entry -> this.matches(entry) || other.matches(entry);
        }
    }
    
    public enum AggregationType {
        COUNT, SUM, AVG, MIN, MAX
    }
    
    public static class QueryResult {
        private final List<LogEntry> results;
        private final long timestamp;
        private final long ttlMs = 300000; // 5 minutes
        
        public QueryResult(List<LogEntry> results, long timestamp) {
            this.results = results;
            this.timestamp = timestamp;
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() - timestamp > ttlMs;
        }
        
        public List<LogEntry> getResults() { return results; }
    }
    
    /**
     * Usage Example
     */
    public static void main(String[] args) {
        LogAnalyticsPlatform.TimePartitionedLogStore logStore = 
            new LogAnalyticsPlatform.TimePartitionedLogStore(3600000); // 1 hour partitions
        
        // Add sample log entries
        long now = System.currentTimeMillis();
        Map<String, String> fields1 = Map.of("service", "auth", "responseTime", "150");
        logStore.addLogEntry(new LogEntry(now - 1000, "INFO", "User login", fields1));
        
        Map<String, String> fields2 = Map.of("service", "auth", "responseTime", "500");
        logStore.addLogEntry(new LogEntry(now - 500, "ERROR", "Login failed", fields2));
        
        // Query time range
        List<LogEntry> recentLogs = logStore.queryTimeRange(
            now - 2000, now, LogFilter.levelEquals("ERROR"));
        System.out.println("Error logs: " + recentLogs.size());
        
        // Aggregate by service
        Map<String, Long> serviceCounts = logStore.aggregateByField(
            now - 3600000, now, "service", AggregationType.COUNT);
        System.out.println("Service counts: " + serviceCounts);
        
        // Search by field
        List<LogEntry> authLogs = logStore.searchByField(
            "service", "auth", now - 3600000, now);
        System.out.println("Auth service logs: " + authLogs.size());
    }
}
```

### Problem 4: Real-Time Chat System - Message Distribution
```java
/**
 * RealTimeChatSystem - Scalable message distribution for millions of users
 * 
 * Problem: Distribute messages to active users with presence tracking
 * Constraints: Sub-100ms delivery, millions of concurrent users, message ordering
 * 
 * Data Structure Decisions:
 * 1. User sessions: ConcurrentHashMap for O(1) lookups
 * 2. Room membership: MultiMap pattern for user-to-rooms mapping
 * 3. Message queues: ConcurrentLinkedQueue per user for ordering
 * 4. Presence tracking: TTL-based cleanup with scheduled maintenance
 */
public class RealTimeChatSystem {
    
    /**
     * ChatSessionManager - Manages active user sessions and presence
     */
    public static class ChatSessionManager {
        // Active sessions: userId -> session info
        private final ConcurrentHashMap<String, UserSession> activeSessions = new ConcurrentHashMap<>();
        
        // Room memberships: roomId -> Set<userId>
        private final ConcurrentHashMap<String, Set<String>> roomMembers = new ConcurrentHashMap<>();
        
        // User rooms: userId -> Set<roomId> (reverse mapping for efficiency)
        private final ConcurrentHashMap<String, Set<String>> userRooms = new ConcurrentHashMap<>();
        
        // Message queues per user for guaranteed delivery
        private final ConcurrentHashMap<String, Queue<ChatMessage>> userMessageQueues = new ConcurrentHashMap<>();
        
        // Scheduled cleanup for inactive sessions
        private final ScheduledExecutorService cleanupExecutor = 
            Executors.newSingleThreadScheduledExecutor(r -> {
                Thread t = new Thread(r, "Chat-Cleanup");
                t.setDaemon(true);
                return t;
            });
        
        public ChatSessionManager() {
            // Cleanup inactive sessions every 30 seconds
            cleanupExecutor.scheduleAtFixedRate(this::cleanupInactiveSessions, 30, 30, TimeUnit.SECONDS);
        }
        
        /**
         * joinRoom() - User joins a chat room
         * 
         * Operations:
         * 1. Add user to room members
         * 2. Add room to user's room list
         * 3. Update user's last activity
         */
        public void joinRoom(String userId, String roomId) {
            // Update session activity
            UserSession session = activeSessions.computeIfAbsent(userId, k -> new UserSession(k));
            session.updateActivity();
            
            // Add to room members (thread-safe set)
            roomMembers.computeIfAbsent(roomId, k -> ConcurrentHashMap.newKeySet()).add(userId);
            
            // Add to user's rooms (reverse mapping)
            userRooms.computeIfAbsent(userId, k -> ConcurrentHashMap.newKeySet()).add(roomId);
            
            // Ensure message queue exists
            userMessageQueues.computeIfAbsent(userId, k -> new ConcurrentLinkedQueue<>());
        }
        
        /**
         * leaveRoom() - User leaves a chat room
         */
        public void leaveRoom(String userId, String roomId) {
            Set<String> members = roomMembers.get(roomId);
            if (members != null) {
                members.remove(userId);
                if (members.isEmpty()) {
                    roomMembers.remove(roomId);
                }
            }
            
            Set<String> rooms = userRooms.get(userId);
            if (rooms != null) {
                rooms.remove(roomId);
                if (rooms.isEmpty()) {
                    userRooms.remove(userId);
                }
            }
        }
        
        /**
         * broadcastMessage() - Send message to all room members
         * 
         * Algorithm:
         * 1. Get room members (O(1) lookup)
         * 2. Filter active users only
         * 3. Add message to each user's queue
         * 4. Notify delivery mechanisms
         */
        public void broadcastMessage(String roomId, ChatMessage message) {
            Set<String> members = roomMembers.get(roomId);
            if (members == null) return;
            
            // Get active members only
            List<String> activeMembers = members.stream()
                .filter(userId -> {
                    UserSession session = activeSessions.get(userId);
                    return session != null && session.isActive();
                })
                .collect(Collectors.toList());
            
            // Add message to each active user's queue
            for (String userId : activeMembers) {
                Queue<ChatMessage> userQueue = userMessageQueues.get(userId);
                if (userQueue != null) {
                    userQueue.offer(message);
                    
                    // Limit queue size to prevent memory issues
                    while (userQueue.size() > 1000) {
                        userQueue.poll(); // Remove oldest message
                    }
                }
            }
            
            // Update message delivery stats
            message.setDeliveredToCount(activeMembers.size());
        }
        
        /**
         * getMessagesForUser() - Get pending messages for user
         */
        public List<ChatMessage> getMessagesForUser(String userId) {
            Queue<ChatMessage> userQueue = userMessageQueues.get(userId);
            if (userQueue == null) {
                return Collections.emptyList();
            }
            
            List<ChatMessage> messages = new ArrayList<>();
            ChatMessage message;
            while ((message = userQueue.poll()) != null) {
                messages.add(message);
            }
            
            // Update user activity
            UserSession session = activeSessions.get(userId);
            if (session != null) {
                session.updateActivity();
            }
            
            return messages;
        }
        
        /**
         * getRoomMembers() - Get active members of a room
         */
        public Set<String> getRoomMembers(String roomId) {
            Set<String> members = roomMembers.get(roomId);
            if (members == null) {
                return Collections.emptySet();
            }
            
            // Filter to active users only
            return members.stream()
                .filter(userId -> {
                    UserSession session = activeSessions.get(userId);
                    return session != null && session.isActive();
                })
                .collect(Collectors.toSet());
        }
        
        /**
         * cleanupInactiveSessions() - Remove inactive sessions and clean up data structures
         */
        private void cleanupInactiveSessions() {
            long now = System.currentTimeMillis();
            
            // Find inactive sessions
            List<String> inactiveUsers = activeSessions.entrySet().stream()
                .filter(entry -> !entry.getValue().isActive(now))
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
            
            // Clean up inactive users
            for (String userId : inactiveUsers) {
                // Remove from all rooms
                Set<String> userRoomSet = userRooms.remove(userId);
                if (userRoomSet != null) {
                    for (String roomId : userRoomSet) {
                        Set<String> members = roomMembers.get(roomId);
                        if (members != null) {
                            members.remove(userId);
                            if (members.isEmpty()) {
                                roomMembers.remove(roomId);
                            }
                        }
                    }
                }
                
                // Remove session and message queue
                activeSessions.remove(userId);
                userMessageQueues.remove(userId);
            }
            
            System.out.println("Cleaned up " + inactiveUsers.size() + " inactive sessions");
        }
        
        /**
         * getSystemStats() - Get system statistics
         */
        public ChatSystemStats getSystemStats() {
            return new ChatSystemStats(
                activeSessions.size(),
                roomMembers.size(),
                userMessageQueues.values().stream().mapToInt(Queue::size).sum()
            );
        }
        
        public void shutdown() {
            cleanupExecutor.shutdown();
        }
    }
    
    /**
     * UserSession - Tracks user session and activity
     */
    public static class UserSession {
        private final String userId;
        private volatile long lastActivityTime;
        private final long sessionTimeoutMs = 300000; // 5 minutes
        
        public UserSession(String userId) {
            this.userId = userId;
            this.lastActivityTime = System.currentTimeMillis();
        }
        
        public void updateActivity() {
            this.lastActivityTime = System.currentTimeMillis();
        }
        
        public boolean isActive() {
            return isActive(System.currentTimeMillis());
        }
        
        public boolean isActive(long currentTime) {
            return (currentTime - lastActivityTime) < sessionTimeoutMs;
        }
        
        public String getUserId() { return userId; }
        public long getLastActivityTime() { return lastActivityTime; }
    }
    
    public static class ChatMessage {
        private final String messageId;
        private final String senderId;
        private final String roomId;
        private final String content;
        private final long timestamp;
        private volatile int deliveredToCount = 0;
        
        public ChatMessage(String messageId, String senderId, String roomId, String content) {
            this.messageId = messageId;
            this.senderId = senderId;
            this.roomId = roomId;
            this.content = content;
            this.timestamp = System.currentTimeMillis();
        }
        
        // Getters and setters
        public String getMessageId() { return messageId; }
        public String getSenderId() { return senderId; }
        public String getRoomId() { return roomId; }
        public String getContent() { return content; }
        public long getTimestamp() { return timestamp; }
        public int getDeliveredToCount() { return deliveredToCount; }
        public void setDeliveredToCount(int count) { this.deliveredToCount = count; }
    }
    
    public static class ChatSystemStats {
        private final int activeUsers;
        private final int activeRooms;
        private final int pendingMessages;
        
        public ChatSystemStats(int activeUsers, int activeRooms, int pendingMessages) {
            this.activeUsers = activeUsers;
            this.activeRooms = activeRooms;
            this.pendingMessages = pendingMessages;
        }
        
        @Override
        public String toString() {
            return String.format("ChatStats{users=%d, rooms=%d, pendingMessages=%d}", 
                activeUsers, activeRooms, pendingMessages);
        }
    }
}
```

### Problem 5: Stock Market Data Feed - Real-Time Price Updates
```java
/**
 * StockMarketDataFeed - High-frequency price update processing
 * 
 * Problem: Process millions of price updates per second for thousands of symbols
 * Constraints: Ultra-low latency, price history, technical indicators
 * 
 * Data Structure Decisions:
 * 1. Price storage: Circular buffer for OHLC data
 * 2. Symbol lookup: ConcurrentHashMap for O(1) access
 * 3. Time-series: Ring buffer with fixed-size windows
 * 4. Aggregation: Rolling calculations with efficient updates
 */
public class StockMarketDataFeed {
    
    /**
     * MarketDataProcessor - Core processor for price updates
     */
    public static class MarketDataProcessor {
        // Symbol data: symbol -> market data
        private final ConcurrentHashMap<String, SymbolData> symbolDataMap = new ConcurrentHashMap<>();
        
        // Price update listeners
        private final List<PriceUpdateListener> listeners = new CopyOnWriteArrayList<>();
        
        // Performance metrics
        private final AtomicLong totalUpdates = new AtomicLong(0);
        private final AtomicLong updateLatencies = new AtomicLong(0);
        
        /**
         * processPriceUpdate() - Process incoming price update
         * 
         * Time Complexity: O(1) for lookup + O(1) for buffer operations
         * Features: Thread-safe, lock-free for reads, minimal contention
         */
        public void processPriceUpdate(PriceUpdate update) {
            long startTime = System.nanoTime();
            
            // Get or create symbol data
            SymbolData symbolData = symbolDataMap.computeIfAbsent(
                update.getSymbol(), k -> new SymbolData(k, 1000)); // 1000 price points history
            
            // Update price data
            symbolData.addPriceUpdate(update);
            
            // Notify listeners asynchronously
            notifyListeners(update);
            
            // Update metrics
            long latency = System.nanoTime() - startTime;
            totalUpdates.incrementAndGet();
            updateLatencies.addAndGet(latency);
        }
        
        /**
         * getLatestPrice() - Get current market price for symbol
         */
        public PriceData getLatestPrice(String symbol) {
            SymbolData data = symbolDataMap.get(symbol);
            return data != null ? data.getLatestPrice() : null;
        }
        
        /**
         * getPriceHistory() - Get recent price history
         */
        public List<PriceData> getPriceHistory(String symbol, int count) {
            SymbolData data = symbolDataMap.get(symbol);
            return data != null ? data.getRecentPrices(count) : Collections.emptyList();
        }
        
        /**
         * getMovingAverage() - Calculate moving average
         */
        public double getMovingAverage(String symbol, int periods) {
            SymbolData data = symbolDataMap.get(symbol);
            return data != null ? data.calculateMovingAverage(periods) : 0.0;
        }
        
        /**
         * getTechnicalIndicators() - Get technical analysis indicators
         */
        public TechnicalIndicators getTechnicalIndicators(String symbol) {
            SymbolData data = symbolDataMap.get(symbol);
            return data != null ? data.calculateTechnicalIndicators() : null;
        }
        
        public void addPriceUpdateListener(PriceUpdateListener listener) {
            listeners.add(listener);
        }
        
        private void notifyListeners(PriceUpdate update) {
            // Async notification to avoid blocking price processing
            CompletableFuture.runAsync(() -> {
                for (PriceUpdateListener listener : listeners) {
                    try {
                        listener.onPriceUpdate(update);
                    } catch (Exception e) {
                        System.err.println("Error notifying listener: " + e.getMessage());
                    }
                }
            });
        }
        
        public MarketDataStats getStats() {
            long updates = totalUpdates.get();
            long avgLatency = updates > 0 ? updateLatencies.get() / updates : 0;
            return new MarketDataStats(symbolDataMap.size(), updates, avgLatency);
        }
    }
    
    /**
     * SymbolData - Efficient storage for individual symbol's market data
     * 
     * Design: Circular buffer for price history with O(1) operations
     */
    public static class SymbolData {
        private final String symbol;
        private final CircularBuffer<PriceData> priceHistory;
        private volatile PriceData latestPrice;
        
        // Rolling calculations for efficiency
        private final RollingCalculator volumeCalculator = new RollingCalculator(20);
        private final RollingCalculator priceCalculator = new RollingCalculator(20);
        
        public SymbolData(String symbol, int historySize) {
            this.symbol = symbol;
            this.priceHistory = new CircularBuffer<>(historySize);
        }
        
        /**
         * addPriceUpdate() - Add new price data point
         * 
         * Updates:
         * 1. Latest price reference
         * 2. Circular buffer history
         * 3. Rolling calculations
         */
        public synchronized void addPriceUpdate(PriceUpdate update) {
            PriceData priceData = new PriceData(
                update.getPrice(),
                update.getVolume(),
                update.getTimestamp()
            );
            
            latestPrice = priceData;
            priceHistory.add(priceData);
            
            // Update rolling calculations
            priceCalculator.addValue(update.getPrice());
            volumeCalculator.addValue(update.getVolume());
        }
        
        public PriceData getLatestPrice() {
            return latestPrice;
        }
        
        public List<PriceData> getRecentPrices(int count) {
            return priceHistory.getRecent(count);
        }
        
        public double calculateMovingAverage(int periods) {
            List<PriceData> recent = priceHistory.getRecent(periods);
            return recent.stream()
                .mapToDouble(PriceData::getPrice)
                .average()
                .orElse(0.0);
        }
        
        public TechnicalIndicators calculateTechnicalIndicators() {
            List<PriceData> recent = priceHistory.getRecent(50); // 50 periods
            if (recent.size() < 20) {
                return new TechnicalIndicators(0, 0, 0, 0);
            }
            
            double ma20 = recent.stream()
                .skip(recent.size() - 20)
                .mapToDouble(PriceData::getPrice)
                .average().orElse(0);
            
            double ma50 = recent.stream()
                .mapToDouble(PriceData::getPrice)
                .average().orElse(0);
            
            double currentPrice = latestPrice != null ? latestPrice.getPrice() : 0;
            
            // Simple RSI calculation
            double rsi = calculateRSI(recent);
            
            // Bollinger Bands
            double[] bollingerBands = calculateBollingerBands(recent, 20, 2.0);
            
            return new TechnicalIndicators(ma20, ma50, rsi, bollingerBands[0]);
        }
        
        private double calculateRSI(List<PriceData> prices) {
            if (prices.size() < 14) return 50; // Neutral RSI
            
            double gainSum = 0, lossSum = 0;
            for (int i = 1; i < Math.min(15, prices.size()); i++) {
                double change = prices.get(i).getPrice() - prices.get(i-1).getPrice();
                if (change > 0) {
                    gainSum += change;
                } else {
                    lossSum += Math.abs(change);
                }
            }
            
            if (lossSum == 0) return 100;
            double rs = gainSum / lossSum;
            return 100 - (100 / (1 + rs));
        }
        
        private double[] calculateBollingerBands(List<PriceData> prices, int periods, double stdDevMultiplier) {
            if (prices.size() < periods) return new double[]{0, 0, 0};
            
            List<PriceData> recent = prices.subList(prices.size() - periods, prices.size());
            double ma = recent.stream().mapToDouble(PriceData::getPrice).average().orElse(0);
            
            double variance = recent.stream()
                .mapToDouble(p -> Math.pow(p.getPrice() - ma, 2))
                .average().orElse(0);
            
            double stdDev = Math.sqrt(variance);
            
            return new double[]{
                ma + (stdDevMultiplier * stdDev), // Upper band
                ma,                               // Middle band (SMA)
                ma - (stdDevMultiplier * stdDev)  // Lower band
            };
        }
    }
    
    /**
     * CircularBuffer - Fixed-size circular buffer for price history
     * 
     * Features: O(1) insertions, efficient memory usage, thread-safe
     */
    public static class CircularBuffer<T> {
        private final Object[] buffer;
        private final int capacity;
        private volatile int head = 0;
        private volatile int size = 0;
        
        public CircularBuffer(int capacity) {
            this.capacity = capacity;
            this.buffer = new Object[capacity];
        }
        
        public synchronized void add(T item) {
            buffer[head] = item;
            head = (head + 1) % capacity;
            if (size < capacity) {
                size++;
            }
        }
        
        @SuppressWarnings("unchecked")
        public synchronized List<T> getRecent(int count) {
            int actualCount = Math.min(count, size);
            List<T> result = new ArrayList<>(actualCount);
            
            int startIndex = (head - actualCount + capacity) % capacity;
            for (int i = 0; i < actualCount; i++) {
                int index = (startIndex + i) % capacity;
                result.add((T) buffer[index]);
            }
            
            return result;
        }
        
        public int size() { return size; }
        public int capacity() { return capacity; }
    }
    
    /**
     * RollingCalculator - Efficient rolling window calculations
     */
    public static class RollingCalculator {
        private final double[] values;
        private final int windowSize;
        private int index = 0;
        private int count = 0;
        private double sum = 0.0;
        
        public RollingCalculator(int windowSize) {
            this.windowSize = windowSize;
            this.values = new double[windowSize];
        }
        
        public synchronized void addValue(double value) {
            if (count < windowSize) {
                sum += value;
                count++;
            } else {
                sum = sum - values[index] + value;
            }
            
            values[index] = value;
            index = (index + 1) % windowSize;
        }
        
        public synchronized double getAverage() {
            return count > 0 ? sum / count : 0.0;
        }
        
        public synchronized double getSum() {
            return sum;
        }
    }
    
    // Supporting classes
    public static class PriceUpdate {
        private final String symbol;
        private final double price;
        private final long volume;
        private final long timestamp;
        
        public PriceUpdate(String symbol, double price, long volume) {
            this.symbol = symbol;
            this.price = price;
            this.volume = volume;
            this.timestamp = System.currentTimeMillis();
        }
        
        public String getSymbol() { return symbol; }
        public double getPrice() { return price; }
        public long getVolume() { return volume; }
        public long getTimestamp() { return timestamp; }
    }
    
    public static class PriceData {
        private final double price;
        private final long volume;
        private final long timestamp;
        
        public PriceData(double price, long volume, long timestamp) {
            this.price = price;
            this.volume = volume;
            this.timestamp = timestamp;
        }
        
        public double getPrice() { return price; }
        public long getVolume() { return volume; }
        public long getTimestamp() { return timestamp; }
    }
    
    public static class TechnicalIndicators {
        private final double ma20;
        private final double ma50;
        private final double rsi;
        private final double bollingerUpper;
        
        public TechnicalIndicators(double ma20, double ma50, double rsi, double bollingerUpper) {
            this.ma20 = ma20;
            this.ma50 = ma50;
            this.rsi = rsi;
            this.bollingerUpper = bollingerUpper;
        }
        
        public double getMa20() { return ma20; }
        public double getMa50() { return ma50; }
        public double getRsi() { return rsi; }
        public double getBollingerUpper() { return bollingerUpper; }
    }
    
    public interface PriceUpdateListener {
        void onPriceUpdate(PriceUpdate update);
    }
    
    public static class MarketDataStats {
        private final int symbolCount;
        private final long totalUpdates;
        private final long avgLatencyNanos;
        
        public MarketDataStats(int symbolCount, long totalUpdates, long avgLatencyNanos) {
            this.symbolCount = symbolCount;
            this.totalUpdates = totalUpdates;
            this.avgLatencyNanos = avgLatencyNanos;
        }
        
        @Override
        public String toString() {
            return String.format("MarketStats{symbols=%d, updates=%d, avgLatency=%.2fμs}", 
                symbolCount, totalUpdates, avgLatencyNanos / 1000.0);
        }
    }
}
```

### Problem 6: Content Delivery Network (CDN) - Cache Management
```java
/**
 * CDNContentCache - Intelligent cache management for content delivery
 * 
 * Problem: Optimize content delivery with multi-level caching
 * Constraints: Memory limits, geographical distribution, cache coherence
 * 
 * Data Structure Decisions:
 * 1. L1 Cache: ConcurrentHashMap with LRU eviction
 * 2. L2 Cache: Compressed storage with background loading
 * 3. Cache statistics: Lock-free counters for performance metrics
 * 4. Invalidation: Pub-sub pattern for cache coherence
 */
public class CDNContentCache {
    
    /**
     * MultiLevelCache - Hierarchical cache with intelligent promotion/demotion
     */
    public static class MultiLevelCache {
        // L1 Cache: Hot content in memory
        private final BoundedLRUCache<String, ContentItem> l1Cache;
        
        // L2 Cache: Warm content with compression
        private final BoundedLRUCache<String, CompressedContent> l2Cache;
        
        // Cache statistics
        private final CacheStatistics stats = new CacheStatistics();
        
        // Background loading executor
        private final ExecutorService backgroundLoader = Executors.newFixedThreadPool(4);
        
        // Content invalidation subscribers
        private final Set<InvalidationListener> invalidationListeners = ConcurrentHashMap.newKeySet();
        
        public MultiLevelCache(int l1Size, int l2Size) {
            this.l1Cache = new BoundedLRUCache<>(l1Size, this::onL1Eviction);
            this.l2Cache = new BoundedLRUCache<>(l2Size, this::onL2Eviction);
        }
        
        /**
         * getContent() - Retrieve content with cache hierarchy
         * 
         * Cache Flow:
         * 1. Check L1 cache (hot)
         * 2. Check L2 cache (warm) -> promote to L1
         * 3. Load from origin -> store in L2
         * 4. Background promotion based on access patterns
         */
        public CompletableFuture<ContentItem> getContent(String contentId) {
            stats.recordRequest();
            
            // L1 Cache check
            ContentItem l1Item = l1Cache.get(contentId);
            if (l1Item != null) {
                stats.recordL1Hit();
                return CompletableFuture.completedFuture(l1Item);
            }
            
            // L2 Cache check
            CompressedContent l2Item = l2Cache.get(contentId);
            if (l2Item != null) {
                stats.recordL2Hit();
                
                // Decompress in background and promote to L1
                return CompletableFuture.supplyAsync(() -> {
                    ContentItem decompressed = l2Item.decompress();
                    l1Cache.put(contentId, decompressed);
                    return decompressed;
                }, backgroundLoader);
            }
            
            // Cache miss - load from origin
            stats.recordMiss();
            return loadFromOrigin(contentId)
                .thenApply(content -> {
                    if (content != null) {
                        // Store in L2 cache with compression
                        CompressedContent compressed = CompressedContent.compress(content);
                        l2Cache.put(contentId, compressed);
                        
                        // Hot content goes directly to L1
                        if (isHotContent(contentId)) {
                            l1Cache.put(contentId, content);
                        }
                    }
                    return content;
                });
        }
        
        /**
         * putContent() - Store content in cache
         */
        public void putContent(String contentId, ContentItem content) {
            if (isHotContent(contentId)) {
                l1Cache.put(contentId, content);
            }
            
            // Always store compressed version in L2
            CompressedContent compressed = CompressedContent.compress(content);
            l2Cache.put(contentId, compressed);
        }
        
        /**
         * invalidateContent() - Remove content from all cache levels
         */
        public void invalidateContent(String contentId) {
            l1Cache.remove(contentId);
            l2Cache.remove(contentId);
            
            // Notify other cache nodes
            notifyInvalidation(contentId);
        }
        
        /**
         * getHitRate() - Calculate overall cache hit rate
         */
        public double getHitRate() {
            return stats.getHitRate();
        }
        
        private CompletableFuture<ContentItem> loadFromOrigin(String contentId) {
            return CompletableFuture.supplyAsync(() -> {
                // Simulate origin loading
                try {
                    Thread.sleep(50); // 50ms origin latency
                    return new ContentItem(contentId, "Content data for " + contentId, 
                                         "text/html", System.currentTimeMillis());
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return null;
                }
            }, backgroundLoader);
        }
        
        private boolean isHotContent(String contentId) {
            // Simple heuristic: content accessed in last hour
            return stats.getRecentAccessCount(contentId) > 10;
        }
        
        private void onL1Eviction(String contentId, ContentItem content) {
            // When evicted from L1, consider storing in L2
            if (!l2Cache.containsKey(contentId)) {
                CompressedContent compressed = CompressedContent.compress(content);
                l2Cache.put(contentId, compressed);
            }
        }
        
        private void onL2Eviction(String contentId, CompressedContent content) {
            // Content completely evicted - update statistics
            stats.recordEviction(contentId);
        }
        
        private void notifyInvalidation(String contentId) {
            for (InvalidationListener listener : invalidationListeners) {
                try {
                    listener.onContentInvalidated(contentId);
                } catch (Exception e) {
                    System.err.println("Error notifying invalidation listener: " + e.getMessage());
                }
            }
        }
        
        public void addInvalidationListener(InvalidationListener listener) {
            invalidationListeners.add(listener);
        }
        
        public void shutdown() {
            backgroundLoader.shutdown();
        }
    }
    
    /**
     * BoundedLRUCache - Generic LRU cache with size limits and eviction callbacks
     */
    public static class BoundedLRUCache<K, V> {
        private final int maxSize;
        private final BiConsumer<K, V> evictionCallback;
        private final LinkedHashMap<K, V> cache;
        
        public BoundedLRUCache(int maxSize, BiConsumer<K, V> evictionCallback) {
            this.maxSize = maxSize;
            this.evictionCallback = evictionCallback;
            this.cache = new LinkedHashMap<K, V>(16, 0.75f, true) {
                @Override
                protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                    boolean shouldRemove = size() > maxSize;
                    if (shouldRemove && evictionCallback != null) {
                        evictionCallback.accept(eldest.getKey(), eldest.getValue());
                    }
                    return shouldRemove;
                }
            };
        }
        
        public synchronized V get(K key) {
            return cache.get(key);
        }
        
        public synchronized V put(K key, V value) {
            return cache.put(key, value);
        }
        
        public synchronized V remove(K key) {
            return cache.remove(key);
        }
        
        public synchronized boolean containsKey(K key) {
            return cache.containsKey(key);
        }
        
        public synchronized int size() {
            return cache.size();
        }
    }
    
    /**
     * CacheStatistics - Thread-safe cache performance metrics
     */
    public static class CacheStatistics {
        private final AtomicLong requests = new AtomicLong(0);
        private final AtomicLong l1Hits = new AtomicLong(0);
        private final AtomicLong l2Hits = new AtomicLong(0);
        private final AtomicLong misses = new AtomicLong(0);
        
        // Content access tracking
        private final ConcurrentHashMap<String, AtomicLong> accessCounts = new ConcurrentHashMap<>();
        private final ConcurrentHashMap<String, Long> lastAccessTime = new ConcurrentHashMap<>();
        
        public void recordRequest() {
            requests.incrementAndGet();
        }
        
        public void recordL1Hit() {
            l1Hits.incrementAndGet();
        }
        
        public void recordL2Hit() {
            l2Hits.incrementAndGet();
        }
        
        public void recordMiss() {
            misses.incrementAndGet();
        }
        
        public void recordAccess(String contentId) {
            accessCounts.computeIfAbsent(contentId, k -> new AtomicLong(0)).incrementAndGet();
            lastAccessTime.put(contentId, System.currentTimeMillis());
        }
        
        public void recordEviction(String contentId) {
            accessCounts.remove(contentId);
            lastAccessTime.remove(contentId);
        }
        
        public double getHitRate() {
            long totalRequests = requests.get();
            if (totalRequests == 0) return 0.0;
            
            long totalHits = l1Hits.get() + l2Hits.get();
            return (double) totalHits / totalRequests;
        }
        
        public long getRecentAccessCount(String contentId) {
            Long lastAccess = lastAccessTime.get(contentId);
            if (lastAccess == null || System.currentTimeMillis() - lastAccess > 3600000) { // 1 hour
                return 0;
            }
            
            AtomicLong count = accessCounts.get(contentId);
            return count != null ? count.get() : 0;
        }
        
        @Override
        public String toString() {
            return String.format("CacheStats{requests=%d, l1Hits=%d, l2Hits=%d, misses=%d, hitRate=%.2f%%}",
                requests.get(), l1Hits.get(), l2Hits.get(), misses.get(), getHitRate() * 100);
        }
    }
    
    // Supporting classes
    public static class ContentItem {
        private final String contentId;
        private final String data;
        private final String contentType;
        private final long lastModified;
        private final int size;
        
        public ContentItem(String contentId, String data, String contentType, long lastModified) {
            this.contentId = contentId;
            this.data = data;
            this.contentType = contentType;
            this.lastModified = lastModified;
            this.size = data.length() * 2; // Rough size estimation
        }
        
        public String getContentId() { return contentId; }
        public String getData() { return data; }
        public String getContentType() { return contentType; }
        public long getLastModified() { return lastModified; }
        public int getSize() { return size; }
    }
    
    public static class CompressedContent {
        private final String contentId;
        private final byte[] compressedData;
        private final String contentType;
        private final long lastModified;
        
        private CompressedContent(String contentId, byte[] compressedData, 
                                String contentType, long lastModified) {
            this.contentId = contentId;
            this.compressedData = compressedData;
            this.contentType = contentType;
            this.lastModified = lastModified;
        }
        
        public static CompressedContent compress(ContentItem content) {
            // Simplified compression (in practice, use gzip/deflate)
            byte[] compressed = content.getData().getBytes();
            return new CompressedContent(content.getContentId(), compressed, 
                                       content.getContentType(), content.getLastModified());
        }
        
        public ContentItem decompress() {
            // Simplified decompression
            String data = new String(compressedData);
            return new ContentItem(contentId, data, contentType, lastModified);
        }
        
        public String getContentId() { return contentId; }
        public int getCompressedSize() { return compressedData.length; }
    }
    
    public interface InvalidationListener {
        void onContentInvalidated(String contentId);
    }
}
```

I'll continue with more scenarios. Let me add the remaining 7 problems:

### Problem 7: Social Network Graph - Friend Recommendations
```java
/**
 * SocialNetworkGraph - Friend recommendation system using graph algorithms
 * 
 * Problem: Suggest friends based on mutual connections and interests
 * Constraints: Millions of users, real-time recommendations, privacy controls
 * 
 * Data Structure Decisions:
 * 1. Graph representation: Adjacency list with weighted edges
 * 2. Recommendation scoring: Priority queue for top-K results
 * 3. Interest matching: Bloom filters for privacy-preserving comparisons
 * 4. Graph traversal: BFS with depth limits for mutual friends
 */
public class SocialNetworkGraph {
    
    /**
     * FriendRecommendationEngine - Core recommendation logic
     */
    public static class FriendRecommendationEngine {
        // User connections: userId -> Set<Connection>
        private final ConcurrentHashMap<String, Set<Connection>> userConnections = new ConcurrentHashMap<>();
        
        // User interests: userId -> Set<Interest> (using Bloom filters for privacy)
        private final ConcurrentHashMap<String, BloomFilter> userInterests = new ConcurrentHashMap<>();
        
        // Recommendation cache with TTL
        private final ConcurrentHashMap<String, CachedRecommendations> recommendationCache = new ConcurrentHashMap<>();
        
        /**
         * addConnection() - Add friendship connection
         */
        public void addConnection(String userId1, String userId2, ConnectionType type, double strength) {
            Connection conn1 = new Connection(userId2, type, strength);
            Connection conn2 = new Connection(userId1, type, strength);
            
            userConnections.computeIfAbsent(userId1, k -> ConcurrentHashMap.newKeySet()).add(conn1);
            userConnections.computeIfAbsent(userId2, k -> ConcurrentHashMap.newKeySet()).add(conn2);
            
            // Invalidate recommendation cache for both users
            recommendationCache.remove(userId1);
            recommendationCache.remove(userId2);
        }
        
        /**
         * addUserInterests() - Add user interests using Bloom filter for privacy
         */
        public void addUserInterests(String userId, Set<String> interests) {
            BloomFilter filter = new BloomFilter(1000, 0.01); // 1000 expected items, 1% false positive
            for (String interest : interests) {
                filter.add(interest);
            }
            userInterests.put(userId, filter);
            
            // Invalidate cache
            recommendationCache.remove(userId);
        }
        
        /**
         * recommendFriends() - Generate friend recommendations
         * 
         * Algorithm:
         * 1. Find mutual friends (2nd degree connections)
         * 2. Calculate recommendation scores based on:
         *    - Number of mutual friends
         *    - Interest similarity
         *    - Connection strength
         * 3. Return top-K recommendations
         */
        public List<FriendRecommendation> recommendFriends(String userId, int count) {
            // Check cache first
            CachedRecommendations cached = recommendationCache.get(userId);
            if (cached != null && !cached.isExpired()) {
                return cached.getRecommendations().stream()
                    .limit(count)
                    .collect(Collectors.toList());
            }
            
            Set<Connection> userFriends = userConnections.get(userId);
            if (userFriends == null || userFriends.isEmpty()) {
                return Collections.emptyList();
            }
            
            // Map to store recommendation candidates and their scores
            Map<String, RecommendationScore> candidates = new HashMap<>();
            
            // Find mutual friends (2nd degree connections)
            for (Connection friend : userFriends) {
                Set<Connection> friendsFriends = userConnections.get(friend.getUserId());
                if (friendsFriends == null) continue;
                
                for (Connection mutualCandidate : friendsFriends) {
                    String candidateId = mutualCandidate.getUserId();
                    
                    // Skip if already friends or same user
                    if (candidateId.equals(userId) || isDirectlyConnected(userId, candidateId)) {
                        continue;
                    }
                    
                    // Calculate or update recommendation score
                    RecommendationScore score = candidates.computeIfAbsent(candidateId, 
                        k -> new RecommendationScore(k));
                    
                    // Add mutual friend contribution
                    score.addMutualFriend(friend.getUserId(), friend.getStrength() * mutualCandidate.getStrength());
                }
            }
            
            // Calculate interest similarity scores
            BloomFilter userInterestFilter = userInterests.get(userId);
            if (userInterestFilter != null) {
                for (RecommendationScore score : candidates.values()) {
                    BloomFilter candidateInterests = userInterests.get(score.getUserId());
                    if (candidateInterests != null) {
                        double similarity = calculateInterestSimilarity(userInterestFilter, candidateInterests);
                        score.setInterestSimilarity(similarity);
                    }
                }
            }
            
            // Generate final recommendations sorted by score
            List<FriendRecommendation> recommendations = candidates.values().stream()
                .map(score -> new FriendRecommendation(
                    score.getUserId(),
                    score.calculateFinalScore(),
                    score.getMutualFriends(),
                    score.getInterestSimilarity()
                ))
                .sorted(Comparator.comparingDouble(FriendRecommendation::getScore).reversed())
                .collect(Collectors.toList());
            
            // Cache recommendations
            recommendationCache.put(userId, new CachedRecommendations(recommendations));
            
            return recommendations.stream().limit(count).collect(Collectors.toList());
        }
        
        /**
         * getMutualFriends() - Get mutual friends between two users
         */
        public Set<String> getMutualFriends(String userId1, String userId2) {
            Set<Connection> friends1 = userConnections.get(userId1);
            Set<Connection> friends2 = userConnections.get(userId2);
            
            if (friends1 == null || friends2 == null) {
                return Collections.emptySet();
            }
            
            Set<String> friendIds1 = friends1.stream()
                .map(Connection::getUserId)
                .collect(Collectors.toSet());
            
            return friends2.stream()
                .map(Connection::getUserId)
                .filter(friendIds1::contains)
                .collect(Collectors.toSet());
        }
        
        private boolean isDirectlyConnected(String userId1, String userId2) {
            Set<Connection> friends = userConnections.get(userId1);
            return friends != null && friends.stream()
                .anyMatch(conn -> conn.getUserId().equals(userId2));
        }
        
        private double calculateInterestSimilarity(BloomFilter filter1, BloomFilter filter2) {
            // Estimate Jaccard similarity using Bloom filter intersection
            // This is approximate but preserves privacy
            return filter1.estimateIntersectionSize(filter2) / 
                   (filter1.estimateSize() + filter2.estimateSize());
        }
    }
    
    /**
     * RecommendationScore - Accumulates scoring factors for a candidate
     */
    public static class RecommendationScore {
        private final String userId;
        private final Set<String> mutualFriends = new HashSet<>();
        private double mutualFriendScore = 0.0;
        private double interestSimilarity = 0.0;
        
        public RecommendationScore(String userId) {
            this.userId = userId;
        }
        
        public void addMutualFriend(String mutualFriendId, double connectionStrength) {
            mutualFriends.add(mutualFriendId);
            mutualFriendScore += connectionStrength;
        }
        
        public void setInterestSimilarity(double similarity) {
            this.interestSimilarity = similarity;
        }
        
        public double calculateFinalScore() {
            // Weighted combination of factors
            double mutualWeight = 0.6;
            double interestWeight = 0.4;
            
            // Normalize mutual friend score by square root to prevent bias toward highly connected users
            double normalizedMutualScore = Math.sqrt(mutualFriendScore);
            
            return (mutualWeight * normalizedMutualScore) + (interestWeight * interestSimilarity);
        }
        
        public String getUserId() { return userId; }
        public Set<String> getMutualFriends() { return new HashSet<>(mutualFriends); }
        public double getInterestSimilarity() { return interestSimilarity; }
    }
    
    // Supporting classes
    public static class Connection {
        private final String userId;
        private final ConnectionType type;
        private final double strength; // 0.0 to 1.0
        
        public Connection(String userId, ConnectionType type, double strength) {
            this.userId = userId;
            this.type = type;
            this.strength = Math.max(0.0, Math.min(1.0, strength));
        }
        
        public String getUserId() { return userId; }
        public ConnectionType getType() { return type; }
        public double getStrength() { return strength; }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Connection that = (Connection) obj;
            return Objects.equals(userId, that.userId);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(userId);
        }
    }
    
    public enum ConnectionType {
        FRIEND, FAMILY, COLLEAGUE, ACQUAINTANCE
    }
    
    public static class FriendRecommendation {
        private final String userId;
        private final double score;
        private final Set<String> mutualFriends;
        private final double interestSimilarity;
        
        public FriendRecommendation(String userId, double score, Set<String> mutualFriends, 
                                  double interestSimilarity) {
            this.userId = userId;
            this.score = score;
            this.mutualFriends = mutualFriends;
            this.interestSimilarity = interestSimilarity;
        }
        
        public String getUserId() { return userId; }
        public double getScore() { return score; }
        public Set<String> getMutualFriends() { return mutualFriends; }
        public double getInterestSimilarity() { return interestSimilarity; }
        
        @Override
        public String toString() {
            return String.format("FriendRecommendation{userId='%s', score=%.3f, mutualFriends=%d}", 
                userId, score, mutualFriends.size());
        }
    }
    
    public static class CachedRecommendations {
        private final List<FriendRecommendation> recommendations;
        private final long timestamp;
        private final long ttlMs = 3600000; // 1 hour
        
        public CachedRecommendations(List<FriendRecommendation> recommendations) {
            this.recommendations = new ArrayList<>(recommendations);
            this.timestamp = System.currentTimeMillis();
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() - timestamp > ttlMs;
        }
        
        public List<FriendRecommendation> getRecommendations() {
            return recommendations;
        }
    }
    
    /**
     * Privacy-preserving BloomFilter for interest matching
     */
    public static class BloomFilter {
        private final BitSet bitSet;
        private final int size;
        private final int numHashFunctions;
        private int estimatedElements = 0;
        
        public BloomFilter(int expectedElements, double falsePositiveRate) {
            this.size = (int) (-expectedElements * Math.log(falsePositiveRate) / Math.pow(Math.log(2), 2));
            this.numHashFunctions = (int) (size / expectedElements * Math.log(2));
            this.bitSet = new BitSet(size);
        }
        
        public void add(String item) {
            for (int i = 0; i < numHashFunctions; i++) {
                int hash = hash(item, i) % size;
                bitSet.set(Math.abs(hash));
            }
            estimatedElements++;
        }
        
        public boolean mightContain(String item) {
            for (int i = 0; i < numHashFunctions; i++) {
                int hash = hash(item, i) % size;
                if (!bitSet.get(Math.abs(hash))) {
                    return false;
                }
            }
            return true;
        }
        
        public double estimateIntersectionSize(BloomFilter other) {
            // Estimate intersection size using Bloom filter properties
            BitSet intersection = (BitSet) this.bitSet.clone();
            intersection.and(other.bitSet);
            
            int intersectionBits = intersection.cardinality();
            return Math.max(0, (double) intersectionBits / size * estimatedElements);
        }
        
        public double estimateSize() {
            return estimatedElements;
        }
        
        private int hash(String item, int hashIndex) {
            return (item.hashCode() * (hashIndex + 1)) + hashIndex;
        }
    }
}
```

### Problem 8: Distributed Task Scheduler - Job Queue Management
```java
/**
 * DistributedTaskScheduler - Scalable job scheduling across multiple nodes
 * 
 * Problem: Schedule and execute tasks across distributed worker nodes
 * Constraints: Fault tolerance, load balancing, priority-based execution
 * 
 * Data Structure Decisions:
 * 1. Task queue: PriorityBlockingQueue for priority-based scheduling
 * 2. Worker tracking: ConcurrentHashMap for node health monitoring
 * 3. Task distribution: Round-robin with health-aware load balancing
 * 4. Failure handling: Exponential backoff with dead letter queue
 */
public class DistributedTaskScheduler {
    
    /**
     * TaskScheduler - Central scheduler managing task distribution
     */
    public static class TaskScheduler {
        // Priority queue for pending tasks
        private final PriorityBlockingQueue<ScheduledTask> taskQueue = 
            new PriorityBlockingQueue<>(1000, new TaskPriorityComparator());
        
        // Active worker nodes
        private final ConcurrentHashMap<String, WorkerNode> workerNodes = new ConcurrentHashMap<>();
        
        // Running tasks tracking
        private final ConcurrentHashMap<String, TaskExecution> runningTasks = new ConcurrentHashMap<>();
        
        // Failed tasks for retry
        private final DelayQueue<RetryableTask> retryQueue = new DelayQueue<>();
        
        // Dead letter queue for permanently failed tasks
        private final BlockingQueue<ScheduledTask> deadLetterQueue = new LinkedBlockingQueue<>();
        
        // Scheduler threads
        private final ExecutorService schedulerExecutor = Executors.newFixedThreadPool(4);
        private final ScheduledExecutorService maintenanceExecutor = 
            Executors.newSingleThreadScheduledExecutor();
        
        private volatile boolean running = true;
        
        public TaskScheduler() {
            startScheduler();
        }
        
        /**
         * scheduleTask() - Add task to scheduling queue
         */
        public String scheduleTask(Task task, Priority priority, long delayMs) {
            ScheduledTask scheduledTask = new ScheduledTask(
                generateTaskId(),
                task,
                priority,
                System.currentTimeMillis() + delayMs
            );
            
            taskQueue.offer(scheduledTask);
            return scheduledTask.getTaskId();
        }
        
        /**
         * registerWorker() - Register a new worker node
         */
        public void registerWorker(String workerId, WorkerCapabilities capabilities) {
            WorkerNode worker = new WorkerNode(workerId, capabilities);
            workerNodes.put(workerId, worker);
            
            // Start health monitoring for this worker
            monitorWorkerHealth(worker);
        }
        
        /**
         * unregisterWorker() - Remove worker node
         */
        public void unregisterWorker(String workerId) {
            WorkerNode worker = workerNodes.remove(workerId);
            if (worker != null) {
                // Reschedule any running tasks from this worker
                rescheduleWorkerTasks(workerId);
            }
        }
        
        /**
         * reportTaskCompletion() - Handle task completion from worker
         */
        public void reportTaskCompletion(String taskId, TaskResult result) {
            TaskExecution execution = runningTasks.remove(taskId);
            if (execution == null) return;
            
            WorkerNode worker = workerNodes.get(execution.getWorkerId());
            if (worker != null) {
                worker.taskCompleted(result.isSuccess());
            }
            
            if (!result.isSuccess() && execution.getRetryCount() < 3) {
                // Add to retry queue with exponential backoff
                long delay = (long) (Math.pow(2, execution.getRetryCount()) * 1000); // 1s, 2s, 4s
                RetryableTask retryTask = new RetryableTask(
                    execution.getScheduledTask(),
                    execution.getRetryCount() + 1,
                    delay
                );
                retryQueue.offer(retryTask);
            } else if (!result.isSuccess()) {
                // Max retries exceeded - move to dead letter queue
                deadLetterQueue.offer(execution.getScheduledTask());
            }
        }
        
        private void startScheduler() {
            // Main task dispatch loop
            schedulerExecutor.submit(this::taskDispatchLoop);
            
            // Retry task processor
            schedulerExecutor.submit(this::retryTaskProcessor);
            
            // Worker health monitor
            maintenanceExecutor.scheduleAtFixedRate(this::performMaintenance, 30, 30, TimeUnit.SECONDS);
        }
        
        /**
         * taskDispatchLoop() - Main loop for dispatching tasks to workers
         */
        private void taskDispatchLoop() {
            while (running) {
                try {
                    ScheduledTask task = taskQueue.take(); // Blocks until task available
                    
                    // Check if task is ready to execute
                    if (task.getExecutionTime() > System.currentTimeMillis()) {
                        // Put back with delay - will be naturally sorted by priority queue
                        taskQueue.offer(task);
                        Thread.sleep(100); // Brief pause to avoid busy waiting
                        continue;
                    }
                    
                    // Find available worker
                    WorkerNode selectedWorker = selectWorker(task);
                    if (selectedWorker != null) {
                        dispatchTaskToWorker(task, selectedWorker);
                    } else {
                        // No workers available - put task back
                        taskQueue.offer(task);
                        Thread.sleep(1000); // Wait before retrying
                    }
                    
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                } catch (Exception e) {
                    System.err.println("Error in task dispatch loop: " + e.getMessage());
                }
            }
        }
        
        /**
         * retryTaskProcessor() - Process failed tasks for retry
         */
        private void retryTaskProcessor() {
            while (running) {
                try {
                    RetryableTask retryTask = retryQueue.take(); // Blocks until retry ready
                    
                    // Re-queue the task for execution
                    taskQueue.offer(retryTask.getScheduledTask());
                    
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
        
        /**
         * selectWorker() - Choose best worker for task using load balancing
         */
        private WorkerNode selectWorker(ScheduledTask task) {
            List<WorkerNode> availableWorkers = workerNodes.values().stream()
                .filter(worker -> worker.isHealthy() && worker.canExecute(task.getTask()))
                .sorted(Comparator.comparingDouble(WorkerNode::getLoadFactor))
                .collect(Collectors.toList());
            
            return availableWorkers.isEmpty() ? null : availableWorkers.get(0);
        }
        
        private void dispatchTaskToWorker(ScheduledTask task, WorkerNode worker) {
            TaskExecution execution = new TaskExecution(task, worker.getWorkerId(), 0);
            runningTasks.put(task.getTaskId(), execution);
            
            // Send task to worker (in real implementation, this would be network call)
            worker.assignTask(task);
        }
        
        private void rescheduleWorkerTasks(String workerId) {
            List<TaskExecution> workerTasks = runningTasks.values().stream()
                .filter(exec -> exec.getWorkerId().equals(workerId))
                .collect(Collectors.toList());
            
            for (TaskExecution execution : workerTasks) {
                runningTasks.remove(execution.getScheduledTask().getTaskId());
                taskQueue.offer(execution.getScheduledTask());
            }
        }
        
        private void performMaintenance() {
            // Remove unhealthy workers
            List<String> unhealthyWorkers = workerNodes.entrySet().stream()
                .filter(entry -> !entry.getValue().isHealthy())
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
            
            for (String workerId : unhealthyWorkers) {
                unregisterWorker(workerId);
            }
        }
        
        private void monitorWorkerHealth(WorkerNode worker) {
            schedulerExecutor.submit(() -> {
                while (running && workerNodes.containsKey(worker.getWorkerId())) {
                    try {
                        // Simulate health check (ping worker)
                        Thread.sleep(10000); // Check every 10 seconds
                        worker.updateLastHeartbeat();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            });
        }
        
        private String generateTaskId() {
            return "task-" + System.currentTimeMillis() + "-" + 
                   Integer.toHexString(System.identityHashCode(this));
        }
        
        public SchedulerStats getStats() {
            return new SchedulerStats(
                taskQueue.size(),
                runningTasks.size(),
                retryQueue.size(),
                deadLetterQueue.size(),
                workerNodes.size()
            );
        }
        
        public void shutdown() {
            running = false;
            schedulerExecutor.shutdown();
            maintenanceExecutor.shutdown();
        }
    }
    
    /**
     * WorkerNode - Represents a distributed worker node
     */
    public static class WorkerNode {
        private final String workerId;
        private final WorkerCapabilities capabilities;
        private volatile long lastHeartbeat;
        private final AtomicInteger activeTasks = new AtomicInteger(0);
        private final AtomicInteger completedTasks = new AtomicInteger(0);
        private final AtomicInteger failedTasks = new AtomicInteger(0);
        
        public WorkerNode(String workerId, WorkerCapabilities capabilities) {
            this.workerId = workerId;
            this.capabilities = capabilities;
            this.lastHeartbeat = System.currentTimeMillis();
        }
        
        public boolean canExecute(Task task) {
            return capabilities.supports(task.getType()) && 
                   activeTasks.get() < capabilities.getMaxConcurrentTasks();
        }
        
        public void assignTask(ScheduledTask task) {
            activeTasks.incrementAndGet();
            // In real implementation, send task to worker via network
        }
        
        public void taskCompleted(boolean success) {
            activeTasks.decrementAndGet();
            if (success) {
                completedTasks.incrementAndGet();
            } else {
                failedTasks.incrementAndGet();
            }
        }
        
        public boolean isHealthy() {
            long timeSinceHeartbeat = System.currentTimeMillis() - lastHeartbeat;
            return timeSinceHeartbeat < 30000; // 30 seconds timeout
        }
        
        public double getLoadFactor() {
            int maxTasks = capabilities.getMaxConcurrentTasks();
            return (double) activeTasks.get() / maxTasks;
        }
        
        public void updateLastHeartbeat() {
            this.lastHeartbeat = System.currentTimeMillis();
        }
        
        public String getWorkerId() { return workerId; }
        public WorkerCapabilities getCapabilities() { return capabilities; }
    }
    
    /**
     * TaskPriorityComparator - Custom comparator for task prioritization
     */
    public static class TaskPriorityComparator implements Comparator<ScheduledTask> {
        @Override
        public int compare(ScheduledTask t1, ScheduledTask t2) {
            // Primary: Priority (higher priority first)
            int priorityComparison = Integer.compare(t2.getPriority().getValue(), t1.getPriority().getValue());
            if (priorityComparison != 0) {
                return priorityComparison;
            }
            
            // Secondary: Execution time (earlier first)
            int timeComparison = Long.compare(t1.getExecutionTime(), t2.getExecutionTime());
            if (timeComparison != 0) {
                return timeComparison;
            }
            
            // Tertiary: Task creation order
            return Long.compare(t1.getCreationTime(), t2.getCreationTime());
        }
    }
    
    // Supporting classes
    public static class ScheduledTask {
        private final String taskId;
        private final Task task;
        private final Priority priority;
        private final long executionTime;
        private final long creationTime;
        
        public ScheduledTask(String taskId, Task task, Priority priority, long executionTime) {
            this.taskId = taskId;
            this.task = task;
            this.priority = priority;
            this.executionTime = executionTime;
            this.creationTime = System.currentTimeMillis();
        }
        
        public String getTaskId() { return taskId; }
        public Task getTask() { return task; }
        public Priority getPriority() { return priority; }
        public long getExecutionTime() { return executionTime; }
        public long getCreationTime() { return creationTime; }
    }
    
    public static class RetryableTask implements Delayed {
        private final ScheduledTask scheduledTask;
        private final int retryCount;
        private final long retryTime;
        
        public RetryableTask(ScheduledTask scheduledTask, int retryCount, long delayMs) {
            this.scheduledTask = scheduledTask;
            this.retryCount = retryCount;
            this.retryTime = System.currentTimeMillis() + delayMs;
        }
        
        @Override
        public long getDelay(TimeUnit unit) {
            long remainingTime = retryTime - System.currentTimeMillis();
            return unit.convert(remainingTime, TimeUnit.MILLISECONDS);
        }
        
        @Override
        public int compareTo(Delayed other) {
            return Long.compare(this.retryTime, ((RetryableTask) other).retryTime);
        }
        
        public ScheduledTask getScheduledTask() { return scheduledTask; }
        public int getRetryCount() { return retryCount; }
    }
    
    public static class TaskExecution {
        private final ScheduledTask scheduledTask;
        private final String workerId;
        private final int retryCount;
        private final long startTime;
        
        public TaskExecution(ScheduledTask scheduledTask, String workerId, int retryCount) {
            this.scheduledTask = scheduledTask;
            this.workerId = workerId;
            this.retryCount = retryCount;
            this.startTime = System.currentTimeMillis();
        }
        
        public ScheduledTask getScheduledTask() { return scheduledTask; }
        public String getWorkerId() { return workerId; }
        public int getRetryCount() { return retryCount; }
        public long getStartTime() { return startTime; }
    }
    
    public interface Task {
        String getType();
        Map<String, Object> getParameters();
        void execute();
    }
    
    public enum Priority {
        LOW(1), NORMAL(2), HIGH(3), CRITICAL(4);
        
        private final int value;
        Priority(int value) { this.value = value; }
        public int getValue() { return value; }
    }
    
    public static class WorkerCapabilities {
        private final Set<String> supportedTaskTypes;
        private final int maxConcurrentTasks;
        
        public WorkerCapabilities(Set<String> supportedTaskTypes, int maxConcurrentTasks) {
            this.supportedTaskTypes = new HashSet<>(supportedTaskTypes);
            this.maxConcurrentTasks = maxConcurrentTasks;
        }
        
        public boolean supports(String taskType) {
            return supportedTaskTypes.contains(taskType);
        }
        
        public int getMaxConcurrentTasks() { return maxConcurrentTasks; }
    }
    
    public static class TaskResult {
        private final boolean success;
        private final String message;
        private final long executionTimeMs;
        
        public TaskResult(boolean success, String message, long executionTimeMs) {
            this.success = success;
            this.message = message;
            this.executionTimeMs = executionTimeMs;
        }
        
        public boolean isSuccess() { return success; }
        public String getMessage() { return message; }
        public long getExecutionTimeMs() { return executionTimeMs; }
    }
    
    public static class SchedulerStats {
        private final int pendingTasks;
        private final int runningTasks;
        private final int retryingTasks;
        private final int deadTasks;
        private final int activeWorkers;
        
        public SchedulerStats(int pendingTasks, int runningTasks, int retryingTasks, 
                            int deadTasks, int activeWorkers) {
            this.pendingTasks = pendingTasks;
            this.runningTasks = runningTasks;
            this.retryingTasks = retryingTasks;
            this.deadTasks = deadTasks;
            this.activeWorkers = activeWorkers;
        }
        
        @Override
        public String toString() {
            return String.format("SchedulerStats{pending=%d, running=%d, retrying=%d, dead=%d, workers=%d}",
                pendingTasks, runningTasks, retryingTasks, deadTasks, activeWorkers);
        }
    }
}
```

### Problem 9: Event Sourcing System - Event Store Management
```java
/**
 * EventSourcingSystem - Efficient event storage and replay for CQRS architecture
 * 
 * Problem: Store and replay events for aggregate reconstruction
 * Constraints: High write throughput, efficient replay, event ordering
 * 
 * Data Structure Decisions:
 * 1. Event storage: ConcurrentSkipListMap for ordered event storage
 * 2. Snapshots: LRU cache for aggregate snapshots
 * 3. Event indexing: ConcurrentHashMap for fast aggregate lookup
 * 4. Event streaming: BlockingQueue for real-time event publishing
 */
public class EventSourcingSystem {
    
    /**
     * EventStore - Core event storage and retrieval engine
     */
    public static class EventStore {
        // Global event sequence (ordered by sequence number)
        private final ConcurrentSkipListMap<Long, StoredEvent> eventSequence = new ConcurrentSkipListMap<>();
        
        // Events by aggregate ID for fast lookup
        private final ConcurrentHashMap<String, ConcurrentSkipListMap<Long, StoredEvent>> aggregateEvents = 
            new ConcurrentHashMap<>();
        
        // Aggregate snapshots for performance
        private final BoundedLRUCache<String, AggregateSnapshot> snapshotCache = 
            new BoundedLRUCache<>(1000, this::onSnapshotEviction);
        
        // Event subscribers for real-time processing
        private final ConcurrentHashMap<String, Set<EventSubscriber>> eventSubscribers = new ConcurrentHashMap<>();
        
        // Event publication queue
        private final BlockingQueue<StoredEvent> publicationQueue = new LinkedBlockingQueue<>();
        
        // Sequence generator
        private final AtomicLong sequenceGenerator = new AtomicLong(0);
        
        // Background publisher
        private final ExecutorService publisherExecutor = Executors.newSingleThreadExecutor();
        
        private volatile boolean running = true;
        
        public EventStore() {
            startEventPublisher();
        }
        
        /**
         * appendEvent() - Add new event to the store
         * 
         * Features:
         * 1. Atomic sequence number generation
         * 2. Ordered storage by sequence and aggregate version
         * 3. Asynchronous event publishing
         */
        public long appendEvent(String aggregateId, String eventType, Object eventData, long expectedVersion) {
            // Generate sequence number atomically
            long sequenceNumber = sequenceGenerator.incrementAndGet();
            
            // Create stored event
            StoredEvent storedEvent = new StoredEvent(
                sequenceNumber,
                aggregateId,
                eventType,
                eventData,
                expectedVersion + 1,
                System.currentTimeMillis()
            );
            
            // Store in global sequence
            eventSequence.put(sequenceNumber, storedEvent);
            
            // Store in aggregate-specific sequence
            aggregateEvents.computeIfAbsent(aggregateId, k -> new ConcurrentSkipListMap<>())
                .put(storedEvent.getAggregateVersion(), storedEvent);
            
            // Invalidate snapshot cache for this aggregate
            snapshotCache.remove(aggregateId);
            
            // Queue for asynchronous publishing
            try {
                publicationQueue.put(storedEvent);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            return sequenceNumber;
        }
        
        /**
         * getAggregateEvents() - Retrieve events for aggregate reconstruction
         * 
         * Optimization: Use snapshots when available to reduce event replay
         */
        public List<StoredEvent> getAggregateEvents(String aggregateId, long fromVersion) {
            ConcurrentSkipListMap<Long, StoredEvent> events = aggregateEvents.get(aggregateId);
            if (events == null) {
                return Collections.emptyList();
            }
            
            // Check if we can use a snapshot
            AggregateSnapshot snapshot = snapshotCache.get(aggregateId);
            long startVersion = fromVersion;
            
            if (snapshot != null && snapshot.getVersion() >= fromVersion) {
                startVersion = snapshot.getVersion() + 1;
            }
            
            // Get events from start version onwards
            return events.tailMap(startVersion, true).values()
                .stream()
                .collect(Collectors.toList());
        }
        
        /**
         * getEventsFromSequence() - Get events from global sequence (for replication)
         */
        public List<StoredEvent> getEventsFromSequence(long fromSequence, int limit) {
            return eventSequence.tailMap(fromSequence, true).values()
                .stream()
                .limit(limit)
                .collect(Collectors.toList());
        }
        
        /**
         * saveSnapshot() - Save aggregate snapshot for performance
         */
        public void saveSnapshot(String aggregateId, long version, Object aggregateState) {
            AggregateSnapshot snapshot = new AggregateSnapshot(
                aggregateId,
                version,
                aggregateState,
                System.currentTimeMillis()
            );
            
            snapshotCache.put(aggregateId, snapshot);
        }
        
        /**
         * subscribeToEvents() - Subscribe to events for real-time processing
         */
        public void subscribeToEvents(String eventType, EventSubscriber subscriber) {
            eventSubscribers.computeIfAbsent(eventType, k -> ConcurrentHashMap.newKeySet()).add(subscriber);
        }
        
        /**
         * unsubscribeFromEvents() - Remove event subscription
         */
        public void unsubscribeFromEvents(String eventType, EventSubscriber subscriber) {
            Set<EventSubscriber> subscribers = eventSubscribers.get(eventType);
            if (subscribers != null) {
                subscribers.remove(subscriber);
                if (subscribers.isEmpty()) {
                    eventSubscribers.remove(eventType);
                }
            }
        }
        
        /**
         * replayEvents() - Replay events for projections or debugging
         */
        public void replayEvents(long fromSequence, EventReplayHandler handler) {
            eventSequence.tailMap(fromSequence, true).values()
                .forEach(event -> {
                    try {
                        handler.handle(event);
                    } catch (Exception e) {
                        System.err.println("Error replaying event " + event.getSequenceNumber() + ": " + e.getMessage());
                    }
                });
        }
        
        private void startEventPublisher() {
            publisherExecutor.submit(() -> {
                while (running) {
                    try {
                        StoredEvent event = publicationQueue.take();
                        publishEventToSubscribers(event);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            });
        }
        
        private void publishEventToSubscribers(StoredEvent event) {
            Set<EventSubscriber> subscribers = eventSubscribers.get(event.getEventType());
            if (subscribers != null) {
                for (EventSubscriber subscriber : subscribers) {
                    try {
                        subscriber.onEvent(event);
                    } catch (Exception e) {
                        System.err.println("Error notifying subscriber: " + e.getMessage());
                    }
                }
            }
        }
        
        private void onSnapshotEviction(String aggregateId, AggregateSnapshot snapshot) {
            // Could implement snapshot persistence here
            System.out.println("Snapshot evicted for aggregate: " + aggregateId);
        }
        
        public EventStoreStats getStats() {
            return new EventStoreStats(
                eventSequence.size(),
                aggregateEvents.size(),
                snapshotCache.size(),
                publicationQueue.size()
            );
        }
        
        public void shutdown() {
            running = false;
            publisherExecutor.shutdown();
        }
    }
    
    /**
     * AggregateRepository - High-level repository for aggregate management
     */
    public static class AggregateRepository<T extends EventSourcedAggregate> {
        private final EventStore eventStore;
        private final AggregateFactory<T> aggregateFactory;
        private final int snapshotFrequency;
        
        public AggregateRepository(EventStore eventStore, AggregateFactory<T> aggregateFactory, 
                                 int snapshotFrequency) {
            this.eventStore = eventStore;
            this.aggregateFactory = aggregateFactory;
            this.snapshotFrequency = snapshotFrequency;
        }
        
        /**
         * load() - Load aggregate by reconstructing from events
         */
        public T load(String aggregateId) {
            // Try to get snapshot first
            AggregateSnapshot snapshot = eventStore.snapshotCache.get(aggregateId);
            
            T aggregate;
            long fromVersion = 0;
            
            if (snapshot != null) {
                // Restore from snapshot
                aggregate = aggregateFactory.fromSnapshot(snapshot);
                fromVersion = snapshot.getVersion() + 1;
            } else {
                // Create new aggregate
                aggregate = aggregateFactory.create(aggregateId);
            }
            
            // Apply events from the last snapshot
            List<StoredEvent> events = eventStore.getAggregateEvents(aggregateId, fromVersion);
            for (StoredEvent event : events) {
                aggregate.applyEvent(event);
            }
            
            return aggregate;
        }
        
        /**
         * save() - Save aggregate by appending uncommitted events
         */
        public void save(T aggregate) {
            List<UncommittedEvent> uncommittedEvents = aggregate.getUncommittedEvents();
            
            for (UncommittedEvent event : uncommittedEvents) {
                eventStore.appendEvent(
                    aggregate.getAggregateId(),
                    event.getEventType(),
                    event.getEventData(),
                    aggregate.getVersion()
                );
                aggregate.incrementVersion();
            }
            
            // Clear uncommitted events
            aggregate.clearUncommittedEvents();
            
            // Create snapshot if threshold reached
            if (aggregate.getVersion() % snapshotFrequency == 0) {
                eventStore.saveSnapshot(
                    aggregate.getAggregateId(),
                    aggregate.getVersion(),
                    aggregate.getState()
                );
            }
        }
    }
    
    // Supporting classes and interfaces
    public static class StoredEvent {
        private final long sequenceNumber;
        private final String aggregateId;
        private final String eventType;
        private final Object eventData;
        private final long aggregateVersion;
        private final long timestamp;
        
        public StoredEvent(long sequenceNumber, String aggregateId, String eventType, 
                          Object eventData, long aggregateVersion, long timestamp) {
            this.sequenceNumber = sequenceNumber;
            this.aggregateId = aggregateId;
            this.eventType = eventType;
            this.eventData = eventData;
            this.aggregateVersion = aggregateVersion;
            this.timestamp = timestamp;
        }
        
        public long getSequenceNumber() { return sequenceNumber; }
        public String getAggregateId() { return aggregateId; }
        public String getEventType() { return eventType; }
        public Object getEventData() { return eventData; }
        public long getAggregateVersion() { return aggregateVersion; }
        public long getTimestamp() { return timestamp; }
    }
    
    public static class AggregateSnapshot {
        private final String aggregateId;
        private final long version;
        private final Object aggregateState;
        private final long timestamp;
        
        public AggregateSnapshot(String aggregateId, long version, Object aggregateState, long timestamp) {
            this.aggregateId = aggregateId;
            this.version = version;
            this.aggregateState = aggregateState;
            this.timestamp = timestamp;
        }
        
        public String getAggregateId() { return aggregateId; }
        public long getVersion() { return version; }
        public Object getAggregateState() { return aggregateState; }
        public long getTimestamp() { return timestamp; }
    }
    
    public static class UncommittedEvent {
        private final String eventType;
        private final Object eventData;
        
        public UncommittedEvent(String eventType, Object eventData) {
            this.eventType = eventType;
            this.eventData = eventData;
        }
        
        public String getEventType() { return eventType; }
        public Object getEventData() { return eventData; }
    }
    
    public interface EventSourcedAggregate {
        String getAggregateId();
        long getVersion();
        void incrementVersion();
        Object getState();
        void applyEvent(StoredEvent event);
        List<UncommittedEvent> getUncommittedEvents();
        void clearUncommittedEvents();
    }
    
    public interface AggregateFactory<T extends EventSourcedAggregate> {
        T create(String aggregateId);
        T fromSnapshot(AggregateSnapshot snapshot);
    }
    
    public interface EventSubscriber {
        void onEvent(StoredEvent event);
    }
    
    public interface EventReplayHandler {
        void handle(StoredEvent event);
    }
    
    public static class EventStoreStats {
        private final long totalEvents;
        private final int aggregateCount;
        private final int snapshotCount;
        private final int pendingPublications;
        
        public EventStoreStats(long totalEvents, int aggregateCount, int snapshotCount, int pendingPublications) {
            this.totalEvents = totalEvents;
            this.aggregateCount = aggregateCount;
            this.snapshotCount = snapshotCount;
            this.pendingPublications = pendingPublications;
        }
        
        @Override
        public String toString() {
            return String.format("EventStoreStats{events=%d, aggregates=%d, snapshots=%d, pending=%d}",
                totalEvents, aggregateCount, snapshotCount, pendingPublications);
        }
    }
    
    /**
     * Reusable BoundedLRUCache from previous examples
     */
    public static class BoundedLRUCache<K, V> {
        private final int maxSize;
        private final BiConsumer<K, V> evictionCallback;
        private final LinkedHashMap<K, V> cache;
        
        public BoundedLRUCache(int maxSize, BiConsumer<K, V> evictionCallback) {
            this.maxSize = maxSize;
            this.evictionCallback = evictionCallback;
            this.cache = new LinkedHashMap<K, V>(16, 0.75f, true) {
                @Override
                protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                    boolean shouldRemove = size() > maxSize;
                    if (shouldRemove && evictionCallback != null) {
                        evictionCallback.accept(eldest.getKey(), eldest.getValue());
                    }
                    return shouldRemove;
                }
            };
        }
        
        public synchronized V get(K key) { return cache.get(key); }
        public synchronized V put(K key, V value) { return cache.put(key, value); }
        public synchronized V remove(K key) { return cache.remove(key); }
        public synchronized int size() { return cache.size(); }
    }
}
```

### Problem 10: Distributed Rate Limiter - Token Bucket Implementation
```java
/**
 * DistributedRateLimiter - High-performance rate limiting across multiple services
 * 
 * Problem: Prevent API abuse while maintaining low latency
 * Constraints: Distributed consistency, sub-millisecond decisions, burst handling
 * 
 * Data Structure Decisions:
 * 1. Token storage: AtomicLong for lock-free token management
 * 2. Time windows: ConcurrentSkipListMap for sliding window tracking
 * 3. Client tracking: ConcurrentHashMap for per-client rate limits
 * 4. Distributed state: Eventually consistent token synchronization
 */
public class DistributedRateLimiter {
    
    /**
     * TokenBucketRateLimiter - Core token bucket algorithm implementation
     */
    public static class TokenBucketRateLimiter {
        // Rate limiter configurations by client/API key
        private final ConcurrentHashMap<String, ClientTokenBucket> clientBuckets = new ConcurrentHashMap<>();
        
        // Global rate limits by rate limit type
        private final ConcurrentHashMap<String, RateLimitConfig> rateLimitConfigs = new ConcurrentHashMap<>();
        
        // Sliding window for burst detection
        private final ConcurrentHashMap<String, SlidingWindowCounter> slidingWindows = new ConcurrentHashMap<>();
        
        // Distributed state synchronization
        private final DistributedTokenSynchronizer synchronizer;
        
        // Background maintenance
        private final ScheduledExecutorService maintenanceExecutor = 
            Executors.newSingleThreadScheduledExecutor();
        
        public TokenBucketRateLimiter(DistributedTokenSynchronizer synchronizer) {
            this.synchronizer = synchronizer;
            startMaintenance();
        }
        
        /**
         * isAllowed() - Check if request is allowed under rate limits
         * 
         * Algorithm:
         * 1. Refill tokens based on elapsed time
         * 2. Check if tokens are available
         * 3. Consume token if available
         * 4. Update sliding window for burst detection
         */
        public RateLimitResult isAllowed(String clientId, String operation, int tokensRequired) {
            // Get or create client bucket
            ClientTokenBucket bucket = getOrCreateBucket(clientId, operation);
            
            // Refill tokens based on elapsed time
            bucket.refillTokens();
            
            // Check sliding window for burst detection
            SlidingWindowCounter window = getSlidingWindow(clientId, operation);
            boolean burstDetected = window.incrementAndCheckBurst(tokensRequired);
            
            if (burstDetected) {
                return new RateLimitResult(false, "Burst limit exceeded", 
                    bucket.getAvailableTokens(), bucket.getRefillTime());
            }
            
            // Try to consume tokens
            if (bucket.tryConsume(tokensRequired)) {
                // Update distributed state asynchronously
                synchronizer.updateTokenState(clientId, operation, bucket.getAvailableTokens());
                
                return new RateLimitResult(true, "Request allowed", 
                    bucket.getAvailableTokens(), bucket.getRefillTime());
            } else {
                return new RateLimitResult(false, "Rate limit exceeded", 
                    bucket.getAvailableTokens(), bucket.getRefillTime());
            }
        }
        
        /**
         * addRateLimitConfig() - Configure rate limits for operations
         */
        public void addRateLimitConfig(String operation, RateLimitConfig config) {
            rateLimitConfigs.put(operation, config);
        }
        
        /**
         * getClientStats() - Get current rate limiting statistics
         */
        public ClientRateLimitStats getClientStats(String clientId) {
            Map<String, TokenBucketStats> bucketStats = clientBuckets.entrySet().stream()
                .filter(entry -> entry.getKey().startsWith(clientId + ":"))
                .collect(Collectors.toMap(
                    entry -> entry.getKey().substring(clientId.length() + 1),
                    entry -> entry.getValue().getStats()
                ));
            
            return new ClientRateLimitStats(clientId, bucketStats);
        }
        
        private ClientTokenBucket getOrCreateBucket(String clientId, String operation) {
            String bucketKey = clientId + ":" + operation;
            return clientBuckets.computeIfAbsent(bucketKey, key -> {
                RateLimitConfig config = rateLimitConfigs.getOrDefault(operation, 
                    new RateLimitConfig(100, 10, 1000)); // Default: 100 tokens, 10/sec refill, 1000ms window
                return new ClientTokenBucket(clientId, operation, config);
            });
        }
        
        private SlidingWindowCounter getSlidingWindow(String clientId, String operation) {
            String windowKey = clientId + ":" + operation + ":window";
            return slidingWindows.computeIfAbsent(windowKey, key -> {
                RateLimitConfig config = rateLimitConfigs.getOrDefault(operation,
                    new RateLimitConfig(100, 10, 1000));
                return new SlidingWindowCounter(config.getBurstLimit(), config.getWindowSizeMs());
            });
        }
        
        private void startMaintenance() {
            // Cleanup expired buckets and synchronize state
            maintenanceExecutor.scheduleAtFixedRate(this::performMaintenance, 60, 60, TimeUnit.SECONDS);
        }
        
        private void performMaintenance() {
            long now = System.currentTimeMillis();
            
            // Remove inactive buckets
            List<String> inactiveBuckets = clientBuckets.entrySet().stream()
                .filter(entry -> now - entry.getValue().getLastAccessTime() > 300000) // 5 minutes
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
            
            for (String bucketKey : inactiveBuckets) {
                clientBuckets.remove(bucketKey);
                slidingWindows.remove(bucketKey + ":window");
            }
            
            // Synchronize token states across distributed nodes
            synchronizer.synchronizeTokenStates(clientBuckets);
        }
        
        public void shutdown() {
            maintenanceExecutor.shutdown();
        }
    }
    
    /**
     * ClientTokenBucket - Thread-safe token bucket for individual clients
     */
    public static class ClientTokenBucket {
        private final String clientId;
        private final String operation;
        private final RateLimitConfig config;
        
        // Token state (using atomic operations for lock-free access)
        private final AtomicLong availableTokens;
        private volatile long lastRefillTime;
        private volatile long lastAccessTime;
        
        // Statistics
        private final AtomicLong totalRequests = new AtomicLong(0);
        private final AtomicLong allowedRequests = new AtomicLong(0);
        private final AtomicLong deniedRequests = new AtomicLong(0);
        
        public ClientTokenBucket(String clientId, String operation, RateLimitConfig config) {
            this.clientId = clientId;
            this.operation = operation;
            this.config = config;
            this.availableTokens = new AtomicLong(config.getCapacity());
            this.lastRefillTime = System.currentTimeMillis();
            this.lastAccessTime = System.currentTimeMillis();
        }
        
        /**
         * refillTokens() - Add tokens based on elapsed time
         */
        public void refillTokens() {
            long now = System.currentTimeMillis();
            long elapsedMs = now - lastRefillTime;
            
            if (elapsedMs > 0) {
                // Calculate tokens to add based on refill rate
                long tokensToAdd = (elapsedMs * config.getRefillRate()) / 1000;
                
                if (tokensToAdd > 0) {
                    // Add tokens up to capacity
                    long currentTokens = availableTokens.get();
                    long newTokens = Math.min(config.getCapacity(), currentTokens + tokensToAdd);
                    
                    // Use compareAndSet for atomic update
                    while (!availableTokens.compareAndSet(currentTokens, newTokens)) {
                        currentTokens = availableTokens.get();
                        newTokens = Math.min(config.getCapacity(), currentTokens + tokensToAdd);
                    }
                    
                    lastRefillTime = now;
                }
            }
            
            lastAccessTime = now;
        }
        
        /**
         * tryConsume() - Attempt to consume tokens atomically
         */
        public boolean tryConsume(int tokensRequired) {
            totalRequests.incrementAndGet();
            
            long currentTokens = availableTokens.get();
            
            while (currentTokens >= tokensRequired) {
                long newTokens = currentTokens - tokensRequired;
                
                if (availableTokens.compareAndSet(currentTokens, newTokens)) {
                    allowedRequests.incrementAndGet();
                    return true;
                }
                
                // Retry with updated value
                currentTokens = availableTokens.get();
            }
            
            deniedRequests.incrementAndGet();
            return false;
        }
        
        public long getAvailableTokens() {
            return availableTokens.get();
        }
        
        public long getRefillTime() {
            long tokensNeeded = config.getCapacity() - availableTokens.get();
            return tokensNeeded > 0 ? (tokensNeeded * 1000) / config.getRefillRate() : 0;
        }
        
        public long getLastAccessTime() {
            return lastAccessTime;
        }
        
        public TokenBucketStats getStats() {
            return new TokenBucketStats(
                availableTokens.get(),
                config.getCapacity(),
                totalRequests.get(),
                allowedRequests.get(),
                deniedRequests.get()
            );
        }
    }
    
    /**
     * SlidingWindowCounter - Sliding window for burst detection
     */
    public static class SlidingWindowCounter {
        private final ConcurrentSkipListMap<Long, AtomicLong> timeWindows = new ConcurrentSkipListMap<>();
        private final long burstLimit;
        private final long windowSizeMs;
        
        public SlidingWindowCounter(long burstLimit, long windowSizeMs) {
            this.burstLimit = burstLimit;
            this.windowSizeMs = windowSizeMs;
        }
        
        /**
         * incrementAndCheckBurst() - Add request count and check for burst
         */
        public boolean incrementAndCheckBurst(int requestCount) {
            long now = System.currentTimeMillis();
            long windowStart = now - windowSizeMs;
            
            // Remove old windows
            timeWindows.headMap(windowStart).clear();
            
            // Add current request to current second
            long currentSecond = now / 1000;
            timeWindows.computeIfAbsent(currentSecond, k -> new AtomicLong(0))
                .addAndGet(requestCount);
            
            // Check if total requests in window exceed burst limit
            long totalRequests = timeWindows.tailMap(windowStart / 1000).values()
                .stream()
                .mapToLong(AtomicLong::get)
                .sum();
            
            return totalRequests > burstLimit;
        }
    }
    
    // Supporting classes
    public static class RateLimitConfig {
        private final long capacity;      // Maximum tokens in bucket
        private final long refillRate;    // Tokens per second refill rate
        private final long windowSizeMs;  // Sliding window size for burst detection
        private final long burstLimit;    // Maximum requests in window
        
        public RateLimitConfig(long capacity, long refillRate, long windowSizeMs) {
            this.capacity = capacity;
            this.refillRate = refillRate;
            this.windowSizeMs = windowSizeMs;
            this.burstLimit = capacity * 2; // Default burst limit
        }
        
        public long getCapacity() { return capacity; }
        public long getRefillRate() { return refillRate; }
        public long getWindowSizeMs() { return windowSizeMs; }
        public long getBurstLimit() { return burstLimit; }
    }
    
    public static class RateLimitResult {
        private final boolean allowed;
        private final String reason;
        private final long remainingTokens;
        private final long retryAfterMs;
        
        public RateLimitResult(boolean allowed, String reason, long remainingTokens, long retryAfterMs) {
            this.allowed = allowed;
            this.reason = reason;
            this.remainingTokens = remainingTokens;
            this.retryAfterMs = retryAfterMs;
        }
        
        public boolean isAllowed() { return allowed; }
        public String getReason() { return reason; }
        public long getRemainingTokens() { return remainingTokens; }
        public long getRetryAfterMs() { return retryAfterMs; }
    }
    
    public static class TokenBucketStats {
        private final long availableTokens;
        private final long capacity;
        private final long totalRequests;
        private final long allowedRequests;
        private final long deniedRequests;
        
        public TokenBucketStats(long availableTokens, long capacity, long totalRequests, 
                              long allowedRequests, long deniedRequests) {
            this.availableTokens = availableTokens;
            this.capacity = capacity;
            this.totalRequests = totalRequests;
            this.allowedRequests = allowedRequests;
            this.deniedRequests = deniedRequests;
        }
        
        public double getSuccessRate() {
            return totalRequests > 0 ? (double) allowedRequests / totalRequests : 0.0;
        }
        
        @Override
        public String toString() {
            return String.format("TokenBucketStats{tokens=%d/%d, requests=%d, success=%.2f%%}",
                availableTokens, capacity, totalRequests, getSuccessRate() * 100);
        }
    }
    
    public static class ClientRateLimitStats {
        private final String clientId;
        private final Map<String, TokenBucketStats> operationStats;
        
        public ClientRateLimitStats(String clientId, Map<String, TokenBucketStats> operationStats) {
            this.clientId = clientId;
            this.operationStats = operationStats;
        }
        
        public String getClientId() { return clientId; }
        public Map<String, TokenBucketStats> getOperationStats() { return operationStats; }
    }
    
    /**
     * Interface for distributed token synchronization
     */
    public interface DistributedTokenSynchronizer {
        void updateTokenState(String clientId, String operation, long availableTokens);
        void synchronizeTokenStates(Map<String, ClientTokenBucket> clientBuckets);
    }
}
```

### Problem 11: Time-Series Database - Efficient Data Aggregation
```java
/**
 * TimeSeriesDatabase - High-performance time-series data storage and querying
 * 
 * Problem: Store and aggregate time-series metrics with efficient querying
 * Constraints: High write throughput, fast range queries, memory efficiency
 * 
 * Data Structure Decisions:
 * 1. Time partitioning: ConcurrentSkipListMap for chronological ordering
 * 2. Data compression: Delta encoding with variable-length encoding
 * 3. Aggregation cache: Pre-computed aggregations at multiple resolutions
 * 4. Index structures: Bloom filters for fast existence checks
 */
public class TimeSeriesDatabase {
    
    /**
     * TimeSeriesStore - Core storage engine for time-series data
     */
    public static class TimeSeriesStore {
        // Time partitions: timestamp -> time partition
        private final ConcurrentSkipListMap<Long, TimePartition> timePartitions = new ConcurrentSkipListMap<>();
        
        // Series metadata: series_id -> metadata
        private final ConcurrentHashMap<String, SeriesMetadata> seriesMetadata = new ConcurrentHashMap<>();
        
        // Aggregation cache: (series_id, resolution, start_time) -> cached aggregation
        private final ConcurrentHashMap<String, CachedAggregation> aggregationCache = new ConcurrentHashMap<>();
        
        // Configuration
        private final long partitionSizeMs;
        private final int maxCacheEntries;
        
        // Background tasks
        private final ScheduledExecutorService maintenanceExecutor = 
            Executors.newSingleThreadScheduledExecutor();
        
        public TimeSeriesStore(long partitionSizeMs, int maxCacheEntries) {
            this.partitionSizeMs = partitionSizeMs; // e.g., 1 hour partitions
            this.maxCacheEntries = maxCacheEntries;
            startMaintenance();
        }
        
        /**
         * writeDataPoint() - Store a single data point
         * 
         * Features:
         * 1. Automatic partitioning by time
         * 2. Delta compression for storage efficiency
         * 3. Series metadata management
         */
        public void writeDataPoint(String seriesId, long timestamp, double value, Map<String, String> tags) {
            // Get or create series metadata
            SeriesMetadata metadata = seriesMetadata.computeIfAbsent(seriesId, 
                k -> new SeriesMetadata(k, tags));
            
            // Determine partition
            long partitionKey = timestamp / partitionSizeMs;
            TimePartition partition = timePartitions.computeIfAbsent(partitionKey, 
                k -> new TimePartition(k * partitionSizeMs, partitionSizeMs));
            
            // Write data point to partition
            partition.addDataPoint(seriesId, timestamp, value);
            
            // Update series statistics
            metadata.updateStats(timestamp, value);
            
            // Invalidate affected aggregations
            invalidateAggregations(seriesId, timestamp);
        }
        
        /**
         * writeBatch() - Efficient batch writing
         */
        public void writeBatch(List<DataPoint> dataPoints) {
            // Group by partition for efficient writing
            Map<Long, List<DataPoint>> partitionGroups = dataPoints.stream()
                .collect(Collectors.groupingBy(dp -> dp.getTimestamp() / partitionSizeMs));
            
            for (Map.Entry<Long, List<DataPoint>> entry : partitionGroups.entrySet()) {
                long partitionKey = entry.getKey();
                TimePartition partition = timePartitions.computeIfAbsent(partitionKey,
                    k -> new TimePartition(k * partitionSizeMs, partitionSizeMs));
                
                // Batch write to partition
                partition.addDataPoints(entry.getValue());
                
                // Update metadata for all series
                for (DataPoint dp : entry.getValue()) {
                    SeriesMetadata metadata = seriesMetadata.computeIfAbsent(dp.getSeriesId(),
                        k -> new SeriesMetadata(k, dp.getTags()));
                    metadata.updateStats(dp.getTimestamp(), dp.getValue());
                }
            }
        }
        
        /**
         * queryRange() - Query data points in time range
         */
        public List<DataPoint> queryRange(String seriesId, long startTime, long endTime) {
            List<DataPoint> results = new ArrayList<>();
            
            // Find relevant partitions
            long startPartition = startTime / partitionSizeMs;
            long endPartition = endTime / partitionSizeMs;
            
            for (long partitionKey = startPartition; partitionKey <= endPartition; partitionKey++) {
                TimePartition partition = timePartitions.get(partitionKey);
                if (partition != null) {
                    results.addAll(partition.queryRange(seriesId, startTime, endTime));
                }
            }
            
            return results;
        }
        
        /**
         * queryAggregation() - Get aggregated data with caching
         */
        public AggregationResult queryAggregation(String seriesId, long startTime, long endTime, 
                                                AggregationType aggregationType, long resolution) {
            
            // Check cache first
            String cacheKey = String.format("%s:%s:%d:%d:%d", seriesId, aggregationType, 
                startTime, endTime, resolution);
            
            CachedAggregation cached = aggregationCache.get(cacheKey);
            if (cached != null && !cached.isExpired()) {
                return cached.getResult();
            }
            
            // Compute aggregation
            AggregationResult result = computeAggregation(seriesId, startTime, endTime, 
                aggregationType, resolution);
            
            // Cache result
            if (aggregationCache.size() < maxCacheEntries) {
                aggregationCache.put(cacheKey, new CachedAggregation(result, 300000)); // 5 min TTL
            }
            
            return result;
        }
        
        /**
         * computeAggregation() - Compute aggregation from raw data
         */
        private AggregationResult computeAggregation(String seriesId, long startTime, long endTime,
                                                   AggregationType aggregationType, long resolution) {
            
            List<DataPoint> dataPoints = queryRange(seriesId, startTime, endTime);
            
            // Group data points into time buckets
            Map<Long, List<DataPoint>> timeBuckets = dataPoints.stream()
                .collect(Collectors.groupingBy(dp -> (dp.getTimestamp() / resolution) * resolution));
            
            List<AggregatedDataPoint> aggregatedPoints = new ArrayList<>();
            
            for (Map.Entry<Long, List<DataPoint>> bucket : timeBuckets.entrySet()) {
                long bucketTime = bucket.getKey();
                List<DataPoint> points = bucket.getValue();
                
                double aggregatedValue = calculateAggregation(points, aggregationType);
                aggregatedPoints.add(new AggregatedDataPoint(bucketTime, aggregatedValue, points.size()));
            }
            
            // Sort by timestamp
            aggregatedPoints.sort(Comparator.comparingLong(AggregatedDataPoint::getTimestamp));
            
            return new AggregationResult(seriesId, aggregationType, resolution, aggregatedPoints);
        }
        
        private double calculateAggregation(List<DataPoint> points, AggregationType type) {
            if (points.isEmpty()) return 0.0;
            
            switch (type) {
                case AVERAGE:
                    return points.stream().mapToDouble(DataPoint::getValue).average().orElse(0.0);
                case SUM:
                    return points.stream().mapToDouble(DataPoint::getValue).sum();
                case MIN:
                    return points.stream().mapToDouble(DataPoint::getValue).min().orElse(0.0);
                case MAX:
                    return points.stream().mapToDouble(DataPoint::getValue).max().orElse(0.0);
                case COUNT:
                    return points.size();
                case LAST:
                    return points.stream()
                        .max(Comparator.comparingLong(DataPoint::getTimestamp))
                        .map(DataPoint::getValue)
                        .orElse(0.0);
                default:
                    return 0.0;
            }
        }
        
        private void invalidateAggregations(String seriesId, long timestamp) {
            // Remove cached aggregations that might be affected by new data
            List<String> keysToRemove = aggregationCache.keySet().stream()
                .filter(key -> key.startsWith(seriesId + ":"))
                .collect(Collectors.toList());
            
            keysToRemove.forEach(aggregationCache::remove);
        }
        
        private void startMaintenance() {
            // Compact old partitions and clean up cache
            maintenanceExecutor.scheduleAtFixedRate(this::performMaintenance, 
                10, 10, TimeUnit.MINUTES);
        }
        
        private void performMaintenance() {
            long now = System.currentTimeMillis();
            
            // Remove expired aggregation cache entries
            List<String> expiredKeys = aggregationCache.entrySet().stream()
                .filter(entry -> entry.getValue().isExpired())
                .map(Map.Entry::getKey)
                .collect(Collectors.toList());
            
            expiredKeys.forEach(aggregationCache::remove);
            
            // Compact old partitions (older than 24 hours)
            long compactionThreshold = now - (24 * 60 * 60 * 1000);
            timePartitions.headMap(compactionThreshold / partitionSizeMs)
                .values()
                .forEach(TimePartition::compact);
        }
        
        public TimeSeriesStats getStats() {
            long totalDataPoints = timePartitions.values().stream()
                .mapToLong(TimePartition::getDataPointCount)
                .sum();
            
            return new TimeSeriesStats(
                seriesMetadata.size(),
                timePartitions.size(),
                totalDataPoints,
                aggregationCache.size()
            );
        }
        
        public void shutdown() {
            maintenanceExecutor.shutdown();
        }
    }
    
    /**
     * TimePartition - Efficient storage for a time partition
     */
    public static class TimePartition {
        private final long startTime;
        private final long partitionSize;
        
        // Series data: series_id -> compressed time series
        private final ConcurrentHashMap<String, CompressedTimeSeries> seriesData = new ConcurrentHashMap<>();
        
        // Partition bloom filter for fast existence checks
        private final BloomFilter seriesBloomFilter = new BloomFilter(10000, 0.01);
        
        public TimePartition(long startTime, long partitionSize) {
            this.startTime = startTime;
            this.partitionSize = partitionSize;
        }
        
        public void addDataPoint(String seriesId, long timestamp, double value) {
            CompressedTimeSeries series = seriesData.computeIfAbsent(seriesId, 
                k -> new CompressedTimeSeries(k));
            
            series.addDataPoint(timestamp, value);
            seriesBloomFilter.add(seriesId);
        }
        
        public void addDataPoints(List<DataPoint> dataPoints) {
            // Group by series for efficient batch processing
            Map<String, List<DataPoint>> seriesGroups = dataPoints.stream()
                .collect(Collectors.groupingBy(DataPoint::getSeriesId));
            
            for (Map.Entry<String, List<DataPoint>> entry : seriesGroups.entrySet()) {
                String seriesId = entry.getKey();
                CompressedTimeSeries series = seriesData.computeIfAbsent(seriesId,
                    k -> new CompressedTimeSeries(k));
                
                entry.getValue().forEach(dp -> series.addDataPoint(dp.getTimestamp(), dp.getValue()));
                seriesBloomFilter.add(seriesId);
            }
        }
        
        public List<DataPoint> queryRange(String seriesId, long startTime, long endTime) {
            // Quick bloom filter check
            if (!seriesBloomFilter.mightContain(seriesId)) {
                return Collections.emptyList();
            }
            
            CompressedTimeSeries series = seriesData.get(seriesId);
            if (series == null) {
                return Collections.emptyList();
            }
            
            return series.queryRange(startTime, endTime);
        }
        
        public void compact() {
            // Compress series data further for long-term storage
            seriesData.values().forEach(CompressedTimeSeries::performDeepCompression);
        }
        
        public long getDataPointCount() {
            return seriesData.values().stream()
                .mapToLong(CompressedTimeSeries::getDataPointCount)
                .sum();
        }
    }
    
    /**
     * CompressedTimeSeries - Delta-encoded time series storage
     */
    public static class CompressedTimeSeries {
        private final String seriesId;
        private final List<Long> timestamps = new ArrayList<>();
        private final List<Double> values = new ArrayList<>();
        
        // Delta compression state
        private long lastTimestamp = 0;
        private double lastValue = 0.0;
        
        public CompressedTimeSeries(String seriesId) {
            this.seriesId = seriesId;
        }
        
        public synchronized void addDataPoint(long timestamp, double value) {
            // Store deltas for compression
            if (timestamps.isEmpty()) {
                timestamps.add(timestamp);
                values.add(value);
            } else {
                timestamps.add(timestamp - lastTimestamp);
                values.add(value - lastValue);
            }
            
            lastTimestamp = timestamp;
            lastValue = value;
        }
        
        public synchronized List<DataPoint> queryRange(long startTime, long endTime) {
            List<DataPoint> results = new ArrayList<>();
            
            // Reconstruct original values from deltas
            long currentTime = 0;
            double currentValue = 0.0;
            
            for (int i = 0; i < timestamps.size(); i++) {
                if (i == 0) {
                    currentTime = timestamps.get(i);
                    currentValue = values.get(i);
                } else {
                    currentTime += timestamps.get(i);
                    currentValue += values.get(i);
                }
                
                if (currentTime >= startTime && currentTime <= endTime) {
                    results.add(new DataPoint(seriesId, currentTime, currentValue, Collections.emptyMap()));
                }
            }
            
            return results;
        }
        
        public void performDeepCompression() {
            // Additional compression for archival (could use advanced algorithms)
            // For now, just ensure the lists are trimmed to size
            if (timestamps instanceof ArrayList) {
                ((ArrayList<Long>) timestamps).trimToSize();
                ((ArrayList<Double>) values).trimToSize();
            }
        }
        
        public long getDataPointCount() {
            return timestamps.size();
        }
    }
    
    // Supporting classes
    public static class DataPoint {
        private final String seriesId;
        private final long timestamp;
        private final double value;
        private final Map<String, String> tags;
        
        public DataPoint(String seriesId, long timestamp, double value, Map<String, String> tags) {
            this.seriesId = seriesId;
            this.timestamp = timestamp;
            this.value = value;
            this.tags = new HashMap<>(tags);
        }
        
        public String getSeriesId() { return seriesId; }
        public long getTimestamp() { return timestamp; }
        public double getValue() { return value; }
        public Map<String, String> getTags() { return tags; }
    }
    
    public static class AggregatedDataPoint {
        private final long timestamp;
        private final double value;
        private final int count;
        
        public AggregatedDataPoint(long timestamp, double value, int count) {
            this.timestamp = timestamp;
            this.value = value;
            this.count = count;
        }
        
        public long getTimestamp() { return timestamp; }
        public double getValue() { return value; }
        public int getCount() { return count; }
    }
    
    public static class SeriesMetadata {
        private final String seriesId;
        private final Map<String, String> tags;
        private volatile long firstTimestamp = Long.MAX_VALUE;
        private volatile long lastTimestamp = Long.MIN_VALUE;
        private volatile double minValue = Double.MAX_VALUE;
        private volatile double maxValue = Double.MIN_VALUE;
        private final AtomicLong dataPointCount = new AtomicLong(0);
        
        public SeriesMetadata(String seriesId, Map<String, String> tags) {
            this.seriesId = seriesId;
            this.tags = new HashMap<>(tags);
        }
        
        public void updateStats(long timestamp, double value) {
            // Update timestamp bounds
            if (timestamp < firstTimestamp) {
                firstTimestamp = timestamp;
            }
            if (timestamp > lastTimestamp) {
                lastTimestamp = timestamp;
            }
            
            // Update value bounds
            if (value < minValue) {
                minValue = value;
            }
            if (value > maxValue) {
                maxValue = value;
            }
            
            dataPointCount.incrementAndGet();
        }
    }
    
    public enum AggregationType {
        AVERAGE, SUM, MIN, MAX, COUNT, LAST
    }
    
    public static class AggregationResult {
        private final String seriesId;
        private final AggregationType aggregationType;
        private final long resolution;
        private final List<AggregatedDataPoint> dataPoints;
        
        public AggregationResult(String seriesId, AggregationType aggregationType, 
                               long resolution, List<AggregatedDataPoint> dataPoints) {
            this.seriesId = seriesId;
            this.aggregationType = aggregationType;
            this.resolution = resolution;
            this.dataPoints = dataPoints;
        }
        
        public String getSeriesId() { return seriesId; }
        public AggregationType getAggregationType() { return aggregationType; }
        public long getResolution() { return resolution; }
        public List<AggregatedDataPoint> getDataPoints() { return dataPoints; }
    }
    
    public static class CachedAggregation {
        private final AggregationResult result;
        private final long expirationTime;
        
        public CachedAggregation(AggregationResult result, long ttlMs) {
            this.result = result;
            this.expirationTime = System.currentTimeMillis() + ttlMs;
        }
        
        public boolean isExpired() {
            return System.currentTimeMillis() > expirationTime;
        }
        
        public AggregationResult getResult() { return result; }
    }
    
    public static class TimeSeriesStats {
        private final int seriesCount;
        private final int partitionCount;
        private final long totalDataPoints;
        private final int cachedAggregations;
        
        public TimeSeriesStats(int seriesCount, int partitionCount, 
                             long totalDataPoints, int cachedAggregations) {
            this.seriesCount = seriesCount;
            this.partitionCount = partitionCount;
            this.totalDataPoints = totalDataPoints;
            this.cachedAggregations = cachedAggregations;
        }
        
        @Override
        public String toString() {
            return String.format("TimeSeriesStats{series=%d, partitions=%d, dataPoints=%d, cached=%d}",
                seriesCount, partitionCount, totalDataPoints, cachedAggregations);
        }
    }
    
    /**
     * Simple Bloom Filter implementation
     */
    public static class BloomFilter {
        private final BitSet bitSet;
        private final int size;
        private final int numHashFunctions;
        
        public BloomFilter(int expectedElements, double falsePositiveRate) {
            this.size = (int) (-expectedElements * Math.log(falsePositiveRate) / Math.pow(Math.log(2), 2));
            this.numHashFunctions = (int) (size / expectedElements * Math.log(2));
            this.bitSet = new BitSet(size);
        }
        
        public void add(String item) {
            for (int i = 0; i < numHashFunctions; i++) {
                int hash = hash(item, i) % size;
                bitSet.set(Math.abs(hash));
            }
        }
        
        public boolean mightContain(String item) {
            for (int i = 0; i < numHashFunctions; i++) {
                int hash = hash(item, i) % size;
                if (!bitSet.get(Math.abs(hash))) {
                    return false;
                }
            }
            return true;
        }
        
        private int hash(String item, int hashIndex) {
            return (item.hashCode() * (hashIndex + 1)) + hashIndex;
        }
    }
}
```

---

## Production Best Practices

### Enterprise Patterns and Guidelines

```java
/**
 * ProductionBestPractices - Enterprise-grade collection usage
 */
public class ProductionBestPractices {
    
    /**
     * Safe Collection Handling
     */
    public List<String> processItems(List<String> input) {
        // 1. Null safety
        if (input == null) return Collections.emptyList();
        
        // 2. Defensive copy
        List<String> workingCopy = new ArrayList<>(input);
        
        // 3. Input validation
        workingCopy.removeIf(item -> item == null || item.trim().isEmpty());
        
        // 4. Processing with immutable result
        return workingCopy.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toUnmodifiableList());
    }
    
    /**
     * Performance Monitoring
     */
    public static class InstrumentedMap<K, V> implements Map<K, V> {
        private final Map<K, V> delegate;
        private final AtomicLong getCount = new AtomicLong(0);
        private final AtomicLong putCount = new AtomicLong(0);
        
        public InstrumentedMap(Map<K, V> delegate) {
            this.delegate = delegate;
        }
        
        @Override
        public V get(Object key) {
            long start = System.nanoTime();
            try {
                return delegate.get(key);
            } finally {
                long duration = System.nanoTime() - start;
                getCount.incrementAndGet();
                if (duration > 1_000_000) { // 1ms threshold
                    logSlowOperation("GET", key, duration);
                }
            }
        }
        
        private void logSlowOperation(String op, Object key, long nanos) {
            System.err.printf("SLOW %s: key=%s duration=%.2fms%n",
                op, key, nanos / 1_000_000.0);
        }
        
        // Implement other Map methods...
        @Override public V put(K key, V value) { return delegate.put(key, value); }
        @Override public int size() { return delegate.size(); }
        @Override public boolean isEmpty() { return delegate.isEmpty(); }
        @Override public boolean containsKey(Object key) { return delegate.containsKey(key); }
        @Override public boolean containsValue(Object value) { return delegate.containsValue(value); }
        @Override public V remove(Object key) { return delegate.remove(key); }
        @Override public void putAll(Map<? extends K, ? extends V> m) { delegate.putAll(m); }
        @Override public void clear() { delegate.clear(); }
        @Override public Set<K> keySet() { return delegate.keySet(); }
        @Override public Collection<V> values() { return delegate.values(); }
        @Override public Set<Entry<K, V>> entrySet() { return delegate.entrySet(); }
    }
}
```

### Key Takeaways

1. **Choose the Right Collection**:
   - ArrayList: Fast random access, slow insertions
   - LinkedList: Fast insertions, slow random access
   - HashMap: O(1) average operations, not thread-safe
   - ConcurrentHashMap: Thread-safe with better performance than synchronized maps

2. **Thread Safety Considerations**:
   - Use concurrent collections for multi-threaded access
   - Avoid Collections.synchronizedXxx() in high-concurrency scenarios
   - Consider copy-on-write for read-heavy workloads

3. **Performance Optimization**:
   - Set initial capacity to avoid resizing
   - Monitor GC impact of collection operations
   - Use primitive collections for numeric data
   - Consider off-heap storage for large datasets

4. **Memory Management**:
   - Be aware of collection overhead (pointers, metadata)
   - Use weak references for caches
   - Implement cleanup strategies for long-lived collections
   - Monitor memory usage in production

5. **Production Readiness**:
   - Implement defensive programming practices
   - Add performance monitoring and alerting
   - Use configuration-based tuning
   - Plan for graceful degradation under load

This comprehensive guide provides the foundation for making informed decisions about Java collections in enterprise applications, with emphasis on scalability, performance, and maintainability.
