# JVM Heap and Thread Analysis Complete Guide

This comprehensive guide covers JVM memory and thread analysis for production troubleshooting, performance optimization, and issue resolution in Java applications.

## Table of Contents
1. [JVM Memory Fundamentals](#1-jvm-memory-fundamentals)
2. [Heap Dump Analysis](#2-heap-dump-analysis)
3. [Thread Dump Analysis](#3-thread-dump-analysis)
4. [Production Monitoring and Tools](#4-production-monitoring-and-tools)
5. [Live Production Analysis](#5-live-production-analysis)
6. [Issue Identification Patterns](#6-issue-identification-patterns)
7. [Production Troubleshooting Scenarios](#7-production-troubleshooting-scenarios)
8. [Quick Production Workarounds](#8-quick-production-workarounds)
9. [Automation and Monitoring](#9-automation-and-monitoring)
10. [Best Practices and Checklists](#10-best-practices-and-checklists)

---

## 1) JVM Memory Fundamentals

### 1.1 JVM Memory Structure

```
JVM Memory Layout:
┌─────────────────────────────────────────────────────────────┐
│                    JVM Process Memory                        │
├─────────────────────────────────────────────────────────────┤
│  Heap Memory (Garbage Collected)                           │
│  ┌─────────────────┬─────────────────┬─────────────────┐    │
│  │   Young Gen     │    Old Gen      │  Metaspace     │    │
│  │ ┌─────┬───────┐ │                 │  (Class Data)   │    │
│  │ │Eden │Survivor│ │                 │                 │    │
│  │ │Space│ S0|S1 │ │                 │                 │    │
│  │ └─────┴───────┘ │                 │                 │    │
│  └─────────────────┴─────────────────┴─────────────────┘    │
├─────────────────────────────────────────────────────────────┤
│  Non-Heap Memory                                           │
│  ┌─────────────────┬─────────────────┬─────────────────┐    │
│  │   Code Cache    │   Direct Memory │   Stack Memory  │    │
│  │                 │   (Off-heap)    │   (Per Thread)  │    │
│  └─────────────────┴─────────────────┴─────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Key Memory Areas:**

- **Heap**: Object allocation, garbage collected
- **Metaspace**: Class metadata (replaced PermGen in Java 8+)
- **Stack**: Method calls, local variables (per thread)
- **PC Register**: Current executing instruction (per thread)
- **Direct Memory**: Off-heap memory (NIO, unsafe operations)

### 1.2 Memory Issues Overview

```
Memory Issue Classification:
┌─────────────────────────────────────────────────────────┐
│                    Memory Issues                        │
├─────────────────┬─────────────────┬─────────────────────┤
│   Heap Issues   │ Non-Heap Issues │   System Issues     │
├─────────────────┼─────────────────┼─────────────────────┤
│ • OutOfMemory   │ • Metaspace OOM │ • Direct Memory OOM │
│ • Memory Leaks  │ • Code Cache    │ • Native Memory     │
│ • GC Pressure   │ • Class Loading │ • File Descriptors  │
│ • Large Objects │ • Reflection    │ • Socket Buffers    │
└─────────────────┴─────────────────┴─────────────────────┘
```

**Common Memory Problems:**
- **OutOfMemoryError**: Heap space exhausted
- **Memory Leaks**: Objects not properly released
- **High GC Pressure**: Frequent garbage collection
- **Direct Memory Issues**: Off-heap memory exhaustion
- **Metaspace Issues**: Too many classes loaded

```
Memory Problem Progression:
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Normal     │───▶│  Memory      │───▶│   High GC    │───▶│     OOM      │
│  Operation   │    │  Pressure    │    │   Activity   │    │    Error     │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
      ↑                     │                     │                     │
      │              ┌──────▼──────┐       ┌──────▼──────┐       ┌──────▼──────┐
      │              │ Monitor &   │       │ Take Action │       │ Emergency   │
      └──────────────│ Alert       │       │ (Scale/Tune)│       │ Response    │
                     └─────────────┘       └─────────────┘       └─────────────┘
```

---

## 2) Heap Dump Analysis

### 2.1 Taking Heap Dumps

#### Command Line Tools

```
Heap Dump Collection Methods:
┌─────────────────────────────────────────────────────────────────┐
│                        Heap Dump Sources                       │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Manual        │   Automatic     │      Programmatic           │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ • jcmd          │ • OOM Trigger   │ • JMX API                   │
│ • jmap          │ • Scheduled     │ • Spring Actuator          │
│ • VisualVM      │ • Threshold     │ • Custom Endpoints          │
│ • kill -3       │ • Monitoring    │ • Management Interface      │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

**1. jcmd (Recommended - Java 8+)**
```bash
# List Java processes
jcmd

# Take heap dump
jcmd <pid> GC.run_finalization
jcmd <pid> VM.gc
jcmd <pid> GC.run
jcmd <pid> VM.heapdump /path/to/heapdump.hprof

# Example
jcmd 12345 VM.heapdump /tmp/app-heapdump-$(date +%Y%m%d-%H%M%S).hprof
```

**2. jmap (Legacy)**
```bash
# Take heap dump
jmap -dump:format=b,file=/path/to/heapdump.hprof <pid>

# Live objects only (forces GC first)
jmap -dump:live,format=b,file=/path/to/heapdump.hprof <pid>

# Example with timestamp
jmap -dump:live,format=b,file=/tmp/heapdump-$(date +%Y%m%d-%H%M%S).hprof 12345
```

**3. Using kill signal (Linux)**
```bash
# Send SIGQUIT to trigger heap dump (if configured)
kill -3 <pid>

# Or using jstack
jstack <pid> > /tmp/thread-dump-$(date +%Y%m%d-%H%M%S).txt
```

#### JVM Arguments for Automatic Heap Dumps

```bash
# Automatic heap dump on OutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/dumps/

# Example startup command
java -Xmx4g -Xms1g \
     -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/var/logs/heapdumps/ \
     -jar application.jar
```

#### Programmatic Heap Dumps

```java
import com.sun.management.HotSpotDiagnosticMXBean;
import java.lang.management.ManagementFactory;
import javax.management.MBeanServer;

public class HeapDumpGenerator {
    
    private static final String HOTSPOT_BEAN_NAME = "com.sun.management:type=HotSpotDiagnostic";
    
    public static void generateHeapDump(String fileName, boolean live) {
        try {
            MBeanServer server = ManagementFactory.getPlatformMBeanServer();
            HotSpotDiagnosticMXBean hotspotMBean = 
                ManagementFactory.newPlatformMXBeanProxy(server, HOTSPOT_BEAN_NAME, 
                                                        HotSpotDiagnosticMXBean.class);
            hotspotMBean.dumpHeap(fileName, live);
            System.out.println("Heap dump created: " + fileName);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    // Usage example
    public static void main(String[] args) {
        String timestamp = java.time.LocalDateTime.now()
            .format(java.time.format.DateTimeFormatter.ofPattern("yyyyMMdd-HHmmss"));
        generateHeapDump("/tmp/app-heapdump-" + timestamp + ".hprof", true);
    }
}
```

#### Spring Boot Actuator

```java
@RestController
public class DiagnosticController {
    
    @PostMapping("/admin/heapdump")
    public ResponseEntity<String> createHeapDump() {
        try {
            String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd-HHmmss"));
            String fileName = "/tmp/heapdump-" + timestamp + ".hprof";
            
            HeapDumpGenerator.generateHeapDump(fileName, true);
            
            return ResponseEntity.ok("Heap dump created: " + fileName);
        } catch (Exception e) {
            return ResponseEntity.status(500).body("Failed to create heap dump: " + e.getMessage());
        }
    }
}
```

### 2.2 Heap Dump Analysis Tools

```
Heap Dump Analysis Workflow:
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Heap Dump   │───▶│   Load in    │───▶│   Analyze    │───▶│   Identify   │
│   (.hprof)   │    │     MAT      │    │   Objects    │    │  Root Cause  │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
        │                     │                     │                     │
        ▼                     ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Size &     │    │  Leak        │    │  Dominator   │    │  Memory      │
│   Location   │    │  Suspects    │    │    Tree      │    │   Leak       │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

#### Eclipse Memory Analyzer (MAT)

**Installation and Usage:**
```bash
# Download from https://www.eclipse.org/mat/
# Or use command line version
wget https://www.eclipse.org/downloads/download.php?file=/mat/1.14.0/rcp/MemoryAnalyzer-1.14.0.20230315-linux.gtk.x86_64.zip

# Unzip and run
./MemoryAnalyzer -vmargs -Xmx4g -data workspace
```

**MAT Analysis Commands:**
```bash
# Command line analysis
./ParseHeapDump.sh /path/to/heapdump.hprof org.eclipse.mat.api:suspects
./ParseHeapDump.sh /path/to/heapdump.hprof org.eclipse.mat.api:overview
./ParseHeapDump.sh /path/to/heapdump.hprof org.eclipse.mat.api:top_components
```

**Key MAT Reports:**
1. **Leak Suspects Report**: Automatically identifies potential memory leaks
2. **Dominator Tree**: Shows objects and their retained memory
3. **Histogram**: Object count and memory usage by class
4. **Thread Overview**: Thread-specific memory usage

#### VisualVM

```bash
# Start VisualVM
jvisualvm

# Or with specific heap dump
jvisualvm --jdkhome $JAVA_HOME /path/to/heapdump.hprof
```

**VisualVM Features:**
- Real-time heap monitoring
- Heap dump comparison
- Memory leak detection
- GC analysis

#### JProfiler

```bash
# Command line analysis
jprofiler -open /path/to/heapdump.hprof
```

#### jhat (Deprecated but still useful)

```bash
# Start heap analysis server
jhat -port 7000 /path/to/heapdump.hprof

# Access via browser: http://localhost:7000
```

### 2.3 What to Look for in Heap Dumps

#### Memory Leak Patterns

**1. Growing Collections**
```java
// Look for these patterns in MAT:
// - Large HashMap/ArrayList instances
// - Collections with unexpected sizes
// - Objects with many references

// Example OQL query in MAT:
SELECT * FROM java.util.HashMap WHERE size > 10000

// Find large collections
SELECT toString(s), s.elementData.@length 
FROM java.util.ArrayList s 
WHERE s.elementData.@length > 1000
```

**2. Retained Memory Analysis**
```java
// MAT OQL queries for memory analysis:

// Find objects with high retained memory
SELECT * FROM OBJECTS 2 WHERE toString(@).matches(".*YourClassName.*")

// Find instances of specific class
SELECT * FROM com.yourcompany.YourClass

// Find objects referencing a specific instance
SELECT * FROM OBJECTS WHERE <instance> IN dominators(@)
```

**3. String Duplicates**
```java
// Find duplicate strings
SELECT toString(s), count(*) as cnt FROM java.lang.String s 
GROUP BY toString(s) 
HAVING count(*) > 100 
ORDER BY count(*) DESC

// Find large char arrays
SELECT s, s.value.@length FROM java.lang.String s 
WHERE s.value.@length > 1000
```

#### Class Loader Leaks

```java
// Find class loaders and their classes
SELECT cl, cl.classes.@length as classCount 
FROM java.lang.ClassLoader cl 
ORDER BY classCount DESC

// Find duplicate classes
SELECT c.name, count(*) as instances 
FROM java.lang.Class c 
GROUP BY c.name 
HAVING count(*) > 1
```

---

## 3) Thread Dump Analysis

### 3.1 Taking Thread Dumps

#### Command Line Tools

**1. jstack (Most Common)**
```bash
# Take thread dump
jstack <pid> > thread-dump-$(date +%Y%m%d-%H%M%S).txt

# Take multiple samples for comparison
for i in {1..5}; do
  echo "Taking thread dump $i/5..."
  jstack 12345 > thread-dump-$i-$(date +%Y%m%d-%H%M%S).txt
  sleep 10
done
```

**2. jcmd**
```bash
# Thread dump with jcmd
jcmd <pid> Thread.print > thread-dump-$(date +%Y%m%d-%H%M%S).txt

# Include locked synchronizers
jcmd <pid> Thread.print -l > thread-dump-detailed-$(date +%Y%m%d-%H%M%S).txt
```

**3. kill signal**
```bash
# Send SIGQUIT (Linux/Unix)
kill -3 <pid>

# Output usually goes to application logs
```

**4. JVisualVM**
```bash
# GUI tool for thread dumps
jvisualvm
# Navigate to application -> Threads -> Thread Dump
```

#### Programmatic Thread Dumps

```java
import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;

public class ThreadDumpGenerator {
    
    public static String generateThreadDump() {
        StringBuilder sb = new StringBuilder();
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        
        // Get all thread information
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(true, true);
        
        for (ThreadInfo threadInfo : threadInfos) {
            sb.append("\"").append(threadInfo.getThreadName()).append("\"");
            sb.append(" #").append(threadInfo.getThreadId());
            sb.append(" ").append(threadInfo.getThreadState());
            
            if (threadInfo.getLockName() != null) {
                sb.append(" on ").append(threadInfo.getLockName());
            }
            
            if (threadInfo.getLockOwnerName() != null) {
                sb.append(" owned by \"").append(threadInfo.getLockOwnerName()).append("\"");
            }
            
            sb.append("\n");
            
            for (StackTraceElement stackTrace : threadInfo.getStackTrace()) {
                sb.append("   at ").append(stackTrace).append("\n");
            }
            sb.append("\n");
        }
        
        return sb.toString();
    }
    
    public static void saveThreadDump(String fileName) {
        try {
            String threadDump = generateThreadDump();
            java.nio.file.Files.write(java.nio.file.Paths.get(fileName), 
                                    threadDump.getBytes());
            System.out.println("Thread dump saved to: " + fileName);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### Spring Boot Actuator Thread Dump

```java
@RestController
public class ThreadDumpController {
    
    @GetMapping("/admin/threaddump")
    public ResponseEntity<String> getThreadDump() {
        String threadDump = ThreadDumpGenerator.generateThreadDump();
        return ResponseEntity.ok()
            .header("Content-Type", "text/plain")
            .body(threadDump);
    }
    
    @PostMapping("/admin/threaddump/save")
    public ResponseEntity<String> saveThreadDump() {
        String timestamp = LocalDateTime.now()
            .format(DateTimeFormatter.ofPattern("yyyyMMdd-HHmmss"));
        String fileName = "/tmp/threaddump-" + timestamp + ".txt";
        
        ThreadDumpGenerator.saveThreadDump(fileName);
        return ResponseEntity.ok("Thread dump saved: " + fileName);
    }
}
```

### 3.2 Thread Dump Analysis Tools

#### Eclipse MAT (Memory Analyzer)
```bash
# Load thread dumps in MAT
# File -> Open Thread Dumps...
```

#### VisualVM
- Built-in thread dump analysis
- Thread timeline visualization
- Deadlock detection

#### IBM Thread and Monitor Dump Analyzer (TMDA)
```bash
# Download from IBM Developer
# Specialized for thread dump analysis
```

#### FastThread.io (Online Tool)
- Upload thread dump file
- Automated analysis
- Generates reports with recommendations

#### Command Line Analysis

**Simple grep-based analysis:**
```bash
# Count threads by state
grep "java.lang.Thread.State:" thread-dump.txt | sort | uniq -c

# Find BLOCKED threads
grep -A 20 "BLOCKED" thread-dump.txt

# Find deadlocks
grep -A 30 -B 5 "Found deadlock" thread-dump.txt

# Count threads by name pattern
grep "Thread-" thread-dump.txt | wc -l

# Find long stack traces (potential issues)
awk '/^"/ {thread=$0; lines=0} /^   at/ {lines++} /^$/ {if(lines>50) print thread " - " lines " stack frames"}' thread-dump.txt
```

### 3.3 Thread States and Patterns

#### Thread States Explained

```
Java Thread State Diagram:
┌─────────────┐
│     NEW     │──────────────┐
└─────────────┘              │ start()
        │                    ▼
        │            ┌─────────────┐
        │            │  RUNNABLE   │◄──────────────┐
        │            └─────────────┘               │
        │                    │                    │
        │      ┌─────────────┼─────────────┐      │
        │      │             │             │      │
        │      ▼             ▼             ▼      │
        │ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
        │ │ BLOCKED  │ │ WAITING  │ │  TIMED   │  │
        │ │          │ │          │ │ WAITING  │  │
        │ └──────────┘ └──────────┘ └──────────┘  │
        │      │             │             │      │
        │      └─────────────┼─────────────┘      │
        │                    │                    │
        │                    └────────────────────┘
        │                                         
        ▼                                         
┌─────────────┐                                  
│ TERMINATED  │                                  
└─────────────┘                                  
```

```java
/*
RUNNABLE: Thread is executing or ready to execute
BLOCKED: Thread is blocked waiting for monitor lock
WAITING: Thread is waiting indefinitely for another thread
TIMED_WAITING: Thread is waiting for specified period
NEW: Thread created but not started
TERMINATED: Thread execution completed
*/
```

#### Common Thread Issues

```
Thread Issues Classification:
┌─────────────────────────────────────────────────────────────────┐
│                         Thread Issues                          │
├──────────────┬──────────────┬──────────────┬─────────────────────┤
│  Deadlocks   │ Contention   │  Resource    │    Performance      │
├──────────────┼──────────────┼──────────────┼─────────────────────┤
│ • Circular   │ • Lock       │ • Pool       │ • CPU Spinning      │
│   Dependencies│   Contention │   Exhaustion │ • Context Switching │
│ • Monitor    │ • Sync       │ • Connection │ • Inefficient Loops │
│   Locks      │   Blocks     │   Leaks      │ • GC Overhead       │
└──────────────┴──────────────┴──────────────┴─────────────────────┘
```

**Deadlock Detection Flow:**
```
Deadlock Analysis Process:
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Take Thread  │───▶│  Search      │───▶│   Analyze    │
│    Dump      │    │ "deadlock"   │    │  Lock Chain  │
└──────────────┘    └──────────────┘    └──────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Multiple    │    │  Identify    │    │  Fix Lock    │
│   Samples    │    │  Threads     │    │   Ordering   │
└──────────────┘    └──────────────┘    └──────────────┘
```

**1. Deadlocks**
```
Example deadlock pattern in thread dump:

"Thread-1" #10 prio=5 os_prio=0 tid=0x... nid=0x... waiting for monitor entry
   java.lang.Thread.State: BLOCKED (on object monitor)
   at com.example.ClassA.methodA(ClassA.java:25)
   - waiting to lock <0x000000076ab62208> (a java.lang.Object)
   - locked <0x000000076ab62218> (a java.lang.Object)

"Thread-2" #11 prio=5 os_prio=0 tid=0x... nid=0x... waiting for monitor entry
   java.lang.Thread.State: BLOCKED (on object monitor)
   at com.example.ClassB.methodB(ClassB.java:30)
   - waiting to lock <0x000000076ab62218> (a java.lang.Object)
   - locked <0x000000076ab62208> (a java.lang.Object)
```

**2. High CPU Consumption**
```bash
# Identify CPU-consuming threads
top -H -p <pid>

# Get thread dump and correlate with high CPU threads
# Convert thread ID from top (decimal) to hex for thread dump
printf "%x\n" <thread-id>
```

**3. Thread Pool Exhaustion**
```
Look for patterns like:
- Many threads in WAITING state on ThreadPoolExecutor
- Application threads waiting on queue.take()
- HTTP connector threads all busy
```

#### Analyzing Thread Dump Patterns

**1. Database Connection Pool Issues**
```
Pattern: Multiple threads waiting on:
at com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool.checkoutPooledConnection
at com.zaxxer.hikari.pool.HikariPool.getConnection
```

**2. HTTP Client Issues**
```
Pattern: Threads waiting on:
at java.net.SocketInputStream.socketRead0(Native Method)
at java.net.SocketInputStream.socketRead
at java.net.SocketInputStream.read
```

**3. Synchronization Issues**
```
Pattern: Multiple threads BLOCKED on same monitor:
- waiting to lock <0x00000000xyz12345> (a java.lang.Object)
```

---

## 4) Production Monitoring and Tools

```
Production Monitoring Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                    Application Layer                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │     JMX      │  │  Actuator    │  │   Custom     │          │
│  │   Metrics    │  │  Endpoints   │  │   Metrics    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
├─────────────────────────────────────────────────────────────────┤
│                    JVM Layer                                   │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Heap       │  │   Thread     │  │     GC       │          │
│  │   Memory     │  │   Dumps      │  │    Logs      │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
├─────────────────────────────────────────────────────────────────┤
│                   System Layer                                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │     CPU      │  │    Memory    │  │   Network    │          │
│  │    Usage     │  │    Usage     │  │   I/O        │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              Monitoring & Alerting Platform                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Prometheus  │  │   Grafana    │  │   AlertMgr   │          │
│  │  /DataDog    │  │  Dashboard   │  │  /PagerDuty  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### 4.1 JVM Monitoring Tools

#### Application Performance Monitoring (APM)

**1. New Relic Configuration**
```bash
# Java agent configuration
java -javaagent:/path/to/newrelic.jar \
     -Dnewrelic.config.file=/path/to/newrelic.yml \
     -jar application.jar
```

**2. AppDynamics Setup**
```bash
java -javaagent:/path/to/AppServerAgent/javaagent.jar \
     -Dappdynamics.agent.applicationName=YourApp \
     -Dappdynamics.agent.tierName=YourTier \
     -jar application.jar
```

**3. DataDog APM**
```bash
java -javaagent:/path/to/dd-java-agent.jar \
     -Ddd.service=your-service \
     -Ddd.env=production \
     -jar application.jar
```

#### Command Line Monitoring

**1. jstat - JVM Statistics**
```bash
# GC monitoring
jstat -gc <pid> 1s

# Memory usage
jstat -gccapacity <pid>

# GC performance
jstat -gcutil <pid> 5s

# Example output analysis script
#!/bin/bash
PID=$1
echo "Monitoring JVM $PID - Press Ctrl+C to stop"
echo "Time,S0,S1,E,O,M,CCS,YGC,YGCT,FGC,FGCT,GCT"
jstat -gc $PID 5s | awk 'NR>1 {printf "%s,%.1f,%.1f,%.1f,%.1f,%.1f,%.1f,%d,%.3f,%d,%.3f,%.3f\n", strftime("%H:%M:%S"), $3,$4,$6,$8,$10,$11,$13,$14,$15,$16,$17}'
```

**2. jmap - Memory Analysis**
```bash
# Heap summary
jmap -heap <pid>

# Class histogram
jmap -histo <pid> | head -20

# Monitor object creation
watch -n 5 'jmap -histo <pid> | head -20'
```

**3. jinfo - JVM Information**
```bash
# JVM flags
jinfo -flags <pid>

# System properties
jinfo -sysprops <pid>

# Set flag at runtime
jinfo -flag +PrintGCDetails <pid>
```

### 4.2 Docker and Kubernetes Monitoring

#### Docker Container Monitoring

```bash
# Container resource usage
docker stats <container-id>

# Execute jstack inside container
docker exec <container-id> jstack 1

# Copy heap dump from container
docker exec <container-id> jcmd 1 VM.heapdump /tmp/heapdump.hprof
docker cp <container-id>:/tmp/heapdump.hprof ./heapdump.hprof
```

#### Kubernetes Monitoring

```yaml
# Pod resource monitoring
apiVersion: v1
kind: Pod
metadata:
  name: java-app
spec:
  containers:
  - name: app
    image: your-app:latest
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
    env:
    - name: JAVA_OPTS
      value: "-Xmx1536m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/"
```

```bash
# Get heap dump from K8s pod
kubectl exec <pod-name> -- jcmd 1 VM.heapdump /tmp/heapdump.hprof
kubectl cp <pod-name>:/tmp/heapdump.hprof ./heapdump.hprof

# Monitor pod resources
kubectl top pod <pod-name>

# Get thread dump
kubectl exec <pod-name> -- jstack 1 > thread-dump.txt
```

---

## 5) Live Production Analysis

### 5.1 Safe Production Diagnostic Commands

```
Production Command Risk Assessment:
┌─────────────────────────────────────────────────────────────────┐
│                    Command Risk Matrix                          │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   LOW RISK      │  MEDIUM RISK    │      HIGH RISK              │
│   (< 1ms)       │  (< 100ms)      │      (> 1s)                 │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ • jps           │ • jstack        │ • jmap -dump                │
│ • jinfo         │ • jstat         │ • VM.heapdump               │
│ • VM.version    │ • class_histogram│ • GC.run (force)           │
│ • VM.uptime     │ • Thread.print  │ • Full thread analysis      │
│ • Process info  │ • Light GC      │ • Deep memory analysis      │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

```
Production Diagnostic Decision Tree:
┌──────────────┐
│   Issue      │
│  Detected    │
└──────┬───────┘
       │
       ▼
 ┌─────────────┐     No    ┌─────────────┐
 │  Critical   │──────────▶│  Use Safe   │
 │  Impact?    │           │  Commands   │
 └─────────────┘           └─────────────┘
       │ Yes                       │
       ▼                           ▼
 ┌─────────────┐           ┌─────────────┐
 │  Business   │           │  Monitor &  │
 │  Downtime   │           │   Alert     │
 │  Acceptable?│           └─────────────┘
 └─────────────┘
       │ Yes
       ▼
 ┌─────────────┐
 │  Use High   │
 │  Risk Cmds  │
 │ (with care) │
 └─────────────┘
```

#### Low Impact Commands

```bash
# Safe commands that don't affect performance significantly:

# 1. Process information
jcmd <pid> VM.version
jcmd <pid> VM.uptime
jcmd <pid> VM.flags
jcmd <pid> VM.system_properties

# 2. JVM information
jinfo -flags <pid>
jps -v

# 3. Quick heap info (minimal impact)
jcmd <pid> GC.class_histogram | head -20

# 4. Thread count
jcmd <pid> Thread.print | grep "java.lang.Thread.State" | wc -l
```

#### Medium Impact Commands

```bash
# Commands that may cause brief pauses:

# 1. Force GC (use carefully)
jcmd <pid> GC.run

# 2. Full class histogram
jcmd <pid> GC.class_histogram

# 3. Thread dump
jcmd <pid> Thread.print
```

#### High Impact Commands (Use with Caution)

```bash
# Commands that can cause significant pauses:

# 1. Heap dump (causes full GC)
jcmd <pid> VM.heapdump /tmp/heapdump.hprof

# 2. Live heap dump (longer pause)
jmap -dump:live,format=b,file=/tmp/heapdump.hprof <pid>
```

### 5.2 Production Monitoring Scripts

#### Automated Health Check Script

```bash
#!/bin/bash
# production-health-check.sh

PID=$1
THRESHOLD_CPU=80
THRESHOLD_MEMORY=85
LOG_FILE="/var/log/jvm-health-$(date +%Y%m%d).log"

# Function to log with timestamp
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Check if process exists
if ! kill -0 $PID 2>/dev/null; then
    log "ERROR: Process $PID not found"
    exit 1
fi

# CPU Usage
CPU_USAGE=$(ps -p $PID -o %cpu --no-headers | awk '{print int($1)}')
log "CPU Usage: ${CPU_USAGE}%"

if [ $CPU_USAGE -gt $THRESHOLD_CPU ]; then
    log "WARNING: High CPU usage detected (${CPU_USAGE}%)"
    # Take thread dump for analysis
    jstack $PID > "/tmp/threaddump-high-cpu-$(date +%Y%m%d-%H%M%S).txt"
    log "Thread dump saved for high CPU analysis"
fi

# Memory Usage
MEMORY_INFO=$(jstat -gc $PID | tail -1)
OLD_USED=$(echo $MEMORY_INFO | awk '{print $8}')
OLD_CAPACITY=$(echo $MEMORY_INFO | awk '{print $9}')
MEMORY_USAGE=$(echo "scale=2; $OLD_USED * 100 / $OLD_CAPACITY" | bc)

log "Memory Usage: ${MEMORY_USAGE}%"

if (( $(echo "$MEMORY_USAGE > $THRESHOLD_MEMORY" | bc -l) )); then
    log "WARNING: High memory usage detected (${MEMORY_USAGE}%)"
    # Take heap histogram
    jcmd $PID GC.class_histogram | head -20 > "/tmp/heap-histogram-$(date +%Y%m%d-%H%M%S).txt"
    log "Heap histogram saved for memory analysis"
fi

# GC Frequency Check
GC_COUNT=$(jstat -gc $PID | tail -1 | awk '{print $13}')
log "Total GC Count: $GC_COUNT"

# Log thread count
THREAD_COUNT=$(jstack $PID 2>/dev/null | grep "java.lang.Thread.State" | wc -l)
log "Thread Count: $THREAD_COUNT"

log "Health check completed"
```

#### Continuous Monitoring Script

```bash
#!/bin/bash
# continuous-monitor.sh

PID=$1
INTERVAL=30
OUTPUT_DIR="/var/monitoring/jvm"

mkdir -p $OUTPUT_DIR

while true; do
    TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    
    # Collect metrics
    echo "=== JVM Metrics - $TIMESTAMP ===" >> "$OUTPUT_DIR/metrics.log"
    
    # GC stats
    jstat -gc $PID >> "$OUTPUT_DIR/gc-stats.log"
    
    # Memory usage
    jcmd $PID VM.memory >> "$OUTPUT_DIR/memory-usage.log"
    
    # Thread count
    THREADS=$(jstack $PID 2>/dev/null | grep "java.lang.Thread.State" | wc -l)
    echo "$TIMESTAMP,$THREADS" >> "$OUTPUT_DIR/thread-count.log"
    
    # CPU usage
    CPU=$(ps -p $PID -o %cpu --no-headers)
    echo "$TIMESTAMP,$CPU" >> "$OUTPUT_DIR/cpu-usage.log"
    
    sleep $INTERVAL
done
```

### 5.3 Remote JMX Monitoring

#### JMX Configuration

```bash
# JVM startup arguments for JMX
-Dcom.sun.management.jmxremote=true
-Dcom.sun.management.jmxremote.port=9999
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
```

#### Secure JMX Setup

```bash
# Secure JMX configuration
-Dcom.sun.management.jmxremote=true
-Dcom.sun.management.jmxremote.port=9999
-Dcom.sun.management.jmxremote.ssl=true
-Dcom.sun.management.jmxremote.authenticate=true
-Dcom.sun.management.jmxremote.password.file=/path/to/jmxremote.password
-Dcom.sun.management.jmxremote.access.file=/path/to/jmxremote.access
```

#### JMX Client Example

```java
import javax.management.MBeanServerConnection;
import javax.management.remote.JMXConnector;
import javax.management.remote.JMXConnectorFactory;
import javax.management.remote.JMXServiceURL;
import java.lang.management.ManagementFactory;
import java.lang.management.MemoryMXBean;
import java.lang.management.ThreadMXBean;

public class JMXMonitor {
    
    public static void monitorRemoteJVM(String host, int port) {
        try {
            JMXServiceURL url = new JMXServiceURL(
                "service:jmx:rmi:///jndi/rmi://" + host + ":" + port + "/jmxrmi");
            
            JMXConnector connector = JMXConnectorFactory.connect(url);
            MBeanServerConnection connection = connector.getMBeanServerConnection();
            
            // Memory monitoring
            MemoryMXBean memoryBean = ManagementFactory.newPlatformMXBeanProxy(
                connection, ManagementFactory.MEMORY_MXBEAN_NAME, MemoryMXBean.class);
            
            System.out.println("Heap Memory: " + memoryBean.getHeapMemoryUsage());
            System.out.println("Non-Heap Memory: " + memoryBean.getNonHeapMemoryUsage());
            
            // Thread monitoring
            ThreadMXBean threadBean = ManagementFactory.newPlatformMXBeanProxy(
                connection, ManagementFactory.THREAD_MXBEAN_NAME, ThreadMXBean.class);
            
            System.out.println("Thread Count: " + threadBean.getThreadCount());
            System.out.println("Peak Thread Count: " + threadBean.getPeakThreadCount());
            
            connector.close();
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

## 6) Issue Identification Patterns

```
Issue Identification Framework:
┌─────────────────────────────────────────────────────────────────┐
│                    Symptom Analysis                            │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Symptoms      │   Data Points   │      Analysis Tools         │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ • High CPU      │ • Thread Dumps  │ • Thread Analysis           │
│ • High Memory   │ • Heap Dumps    │ • Memory Analysis           │
│ • Slow Response │ • GC Logs       │ • Performance Profiling    │
│ • App Hanging   │ • App Logs      │ • Deadlock Detection       │
│ • Errors        │ • System Metrics│ • Log Analysis             │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

```
Diagnostic Decision Flow:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Observe    │───▶│   Collect   │───▶│   Analyze   │───▶│   Action    │
│  Symptoms   │    │    Data     │    │    Data     │    │   Plan      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ • CPU High  │    │ • Dumps     │    │ • Patterns  │    │ • Immediate │
│ • Memory    │    │ • Logs      │    │ • Trends    │    │ • Long-term │
│ • Response  │    │ • Metrics   │    │ • Root      │    │ • Prevention│
│ • Errors    │    │ • Traces    │    │   Cause     │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### 6.1 Memory Issues

#### OutOfMemoryError Patterns

**1. Java Heap Space**
```
java.lang.OutOfMemoryError: Java heap space

Causes:
- Memory leaks
- Insufficient heap size
- Large object allocation
- Cache misconfigurations

Analysis Steps:
1. Take heap dump before crash
2. Analyze dominator tree in MAT
3. Look for unexpected large objects
4. Check for growing collections
```

**2. Metaspace/PermGen**
```
java.lang.OutOfMemoryError: Metaspace

Causes:
- Too many classes loaded
- Class loader leaks
- Dynamic proxy generation
- Reflection-heavy frameworks

Analysis Steps:
1. Check class count: jcmd <pid> GC.class_stats
2. Analyze class loaders in heap dump
3. Look for duplicate classes
4. Check framework configurations
```

**3. Direct Memory**
```
java.lang.OutOfMemoryError: Direct buffer memory

Causes:
- NIO operations
- Off-heap caches
- Native memory leaks
- Netty applications

Analysis Steps:
1. Monitor direct memory: -XX:NativeMemoryTracking=summary
2. Check NIO buffer usage
3. Analyze off-heap frameworks
```

#### Memory Leak Detection

**Heap Dump Analysis Workflow:**

```bash
# 1. Take multiple heap dumps over time
jcmd <pid> VM.heapdump /tmp/heapdump-1.hprof
sleep 300
jcmd <pid> VM.heapdump /tmp/heapdump-2.hprof
sleep 300
jcmd <pid> VM.heapdump /tmp/heapdump-3.hprof

# 2. Compare heap dumps in MAT
# Look for objects that consistently grow
```

**MAT Analysis Queries:**
```sql
-- Find objects that might be leaking
SELECT * FROM java.util.HashMap s WHERE s.@retainedHeapSize > 1000000

-- Find large arrays
SELECT s, s.@length FROM OBJECTS s WHERE s.@length > 10000

-- Find objects with many references
SELECT s, s.@referrers.size() as refCount FROM OBJECTS s WHERE s.@referrers.size() > 100
```

### 6.2 Thread Issues

```
Thread Issue Diagnostic Matrix:
┌────────────────┬─────────────────┬─────────────────┬─────────────────┐
│   Symptom      │   Thread State  │   Stack Trace   │   Resolution    │
├────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ App Hanging    │ BLOCKED         │ monitor locks   │ Fix deadlock    │
│ High CPU       │ RUNNABLE        │ tight loops     │ Optimize code   │
│ Slow Response  │ WAITING         │ I/O operations  │ Tune timeouts   │
│ Thread Leaks   │ Mix of states   │ thread creation │ Fix lifecycle   │
│ Pool Exhausted │ WAITING         │ queue.take()    │ Increase pool   │
└────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

```
Deadlock Analysis Flow:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Thread    │───▶│   Search    │───▶│   Found     │
│    Dump     │    │ "deadlock"  │    │ Deadlock?   │
└─────────────┘    └─────────────┘    └──────┬──────┘
                                            │ Yes
                                            ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Emergency  │◄───│   Analyze   │◄───│  Extract    │
│   Restart   │    │ Lock Chain  │    │ Lock Info   │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Implement  │    │  Fix Lock   │    │  Monitor    │
│  Timeouts   │    │  Ordering   │    │ for Repeat  │
└─────────────┘    └─────────────┘    └─────────────┘
```

#### Deadlock Detection

**Thread Dump Analysis:**
```bash
# Extract deadlock information
grep -A 50 "Found deadlock" thread-dump.txt

# Check for circular dependencies
awk '/Found deadlock/,/^$/' thread-dump.txt
```

**Deadlock Prevention Strategies:**
```java
// 1. Ordered locking
private static final Object lock1 = new Object();
private static final Object lock2 = new Object();

public void methodA() {
    synchronized(lock1) {
        synchronized(lock2) {
            // work
        }
    }
}

// 2. Timeout-based locking
if (lock.tryLock(5, TimeUnit.SECONDS)) {
    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

#### Thread Pool Issues

**Common Patterns:**
```
1. Thread Pool Exhaustion:
   - All threads in WAITING state
   - Tasks queued but not processing
   - High response times

2. Thread Leaks:
   - Thread count continuously growing
   - Threads not returning to pool
   - Resource exhaustion

3. Contention Issues:
   - Many threads BLOCKED on same monitor
   - High lock contention
   - Performance degradation
```

**Analysis Commands:**
```bash
# Count thread states
grep "java.lang.Thread.State:" thread-dump.txt | sort | uniq -c

# Find threads waiting on locks
grep -A 5 "waiting to lock" thread-dump.txt

# Identify hot locks
grep "waiting to lock" thread-dump.txt | sort | uniq -c | sort -nr
```

### 6.3 Performance Issues

#### High CPU Usage

**Analysis Steps:**
```bash
# 1. Identify high CPU threads
top -H -p <pid>

# 2. Convert thread ID to hex
printf "%x\n" <thread-id>

# 3. Find thread in thread dump
grep "nid=0x<hex-id>" thread-dump.txt -A 20

# 4. Analyze what the thread is doing
```

**Common High CPU Patterns:**
```
1. Infinite loops
2. Inefficient algorithms
3. Excessive GC activity
4. Lock spinning
5. Regex processing
6. String manipulation
```

#### GC Issues

**GC Analysis:**
```bash
# Monitor GC activity
jstat -gc <pid> 5s

# GC logs analysis (if enabled)
tail -f gc.log | grep "Full GC"

# Check GC efficiency
jstat -gcutil <pid>
```

**GC Tuning Recommendations:**
```bash
# G1 GC (recommended for most applications)
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m

# CMS GC (legacy)
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled

# ZGC (low latency)
-XX:+UseZGC
-XX:+UnlockExperimentalVMOptions
```

---

## 7) Production Troubleshooting Scenarios

```
Incident Response Workflow:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Detection  │───▶│ Assessment  │───▶│ Mitigation  │───▶│ Resolution  │
│  (0-2 min)  │    │ (2-5 min)   │    │ (5-15 min)  │    │ (15+ min)   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ • Alert     │    │ • Scope     │    │ • Scale     │    │ • Root      │
│ • Monitor   │    │ • Impact    │    │ • Restart   │    │   Cause     │
│ • Log       │    │ • Urgency   │    │ • Config    │    │ • Fix       │
│ • Notify    │    │ • Data      │    │ • Isolate   │    │ • Test      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

```
Troubleshooting Priority Matrix:
┌─────────────────────────────────────────────────────────────────┐
│                    Impact vs Urgency                           │
│                                                                 │
│    High │  P1: Critical     │  P2: High Priority               │
│  Impact │  • Service Down   │  • Performance Degraded         │
│         │  • Data Loss      │  • Partial Outage               │
│         │                   │                                  │
│  ───────┼───────────────────┼──────────────────────────────────│
│         │                   │                                  │
│    Low  │  P3: Medium       │  P4: Low Priority               │
│  Impact │  • Minor Issues   │  • Cosmetic Issues              │
│         │  • Workarounds    │  • Feature Requests             │
│         │                   │                                  │
│         └───────────────────┴──────────────────────────────────│
│             High Urgency         Low Urgency                   │
└─────────────────────────────────────────────────────────────────┘
```

### 7.1 High Memory Usage Scenario

**Symptoms:**
- Application slow response
- Frequent GC activity
- Memory alerts from monitoring

**Immediate Actions:**
```bash
# 1. Check current memory usage
jstat -gc <pid>

# 2. Get heap histogram to identify memory hogs
jcmd <pid> GC.class_histogram | head -20

# 3. If critical, force GC
jcmd <pid> GC.run

# 4. Take heap dump for analysis
jcmd <pid> VM.heapdump /tmp/emergency-heapdump.hprof
```

**Analysis Workflow:**
```bash
# 1. Open heap dump in MAT
# 2. Run Leak Suspects Report
# 3. Check Dominator Tree for large objects
# 4. Analyze object references
# 5. Identify root cause

# Example MAT OQL queries:
SELECT * FROM java.util.concurrent.ConcurrentHashMap WHERE size > 1000
SELECT * FROM java.lang.String s WHERE s.value.@length > 10000
```

### 7.2 Application Hanging Scenario

**Symptoms:**
- Application not responding
- Health checks failing
- No response to requests

**Immediate Actions:**
```bash
# 1. Take thread dump immediately
jstack <pid> > /tmp/hang-threaddump-$(date +%Y%m%d-%H%M%S).txt

# 2. Take multiple samples
for i in {1..3}; do
  jstack <pid> > /tmp/hang-threaddump-$i.txt
  sleep 10
done

# 3. Check for deadlocks
grep -A 20 "Found deadlock" /tmp/hang-threaddump-*.txt
```

**Analysis Steps:**
```bash
# 1. Identify main application threads
grep -A 10 "http-nio\|pool-\|main" threaddump.txt

# 2. Look for blocked threads
grep -B 2 -A 10 "BLOCKED" threaddump.txt

# 3. Find common wait points
grep -h "at " threaddump.txt | sort | uniq -c | sort -nr | head -10
```

### 7.3 High CPU Usage Scenario

**Symptoms:**
- CPU usage > 90%
- System becoming unresponsive
- Performance degradation

**Immediate Actions:**
```bash
# 1. Identify high CPU threads
top -H -p <pid> -n 1 | head -20

# 2. Take thread dump
jstack <pid> > /tmp/highcpu-threaddump.txt

# 3. Correlate high CPU threads with thread dump
for tid in $(top -H -p <pid> -n 1 | awk 'NR>7 {print $1}' | head -5); do
  hex_tid=$(printf "%x" $tid)
  echo "Thread $tid (0x$hex_tid):"
  grep -A 15 "nid=0x$hex_tid" /tmp/highcpu-threaddump.txt
done
```

**Quick Fixes:**
```bash
# 1. Restart application if critical
systemctl restart your-application

# 2. Reduce thread pool sizes temporarily
# 3. Enable GC logging for analysis
# 4. Scale horizontally if possible
```

### 7.4 Memory Leak Scenario

**Symptoms:**
- Memory usage continuously growing
- Frequent OutOfMemoryError
- GC overhead increasing

**Emergency Response:**
```bash
# 1. Increase heap size temporarily (if possible)
kill -TERM <pid>
# Restart with: -Xmx8g (instead of -Xmx4g)

# 2. Take heap dump before restart
jcmd <pid> VM.heapdump /tmp/memory-leak-heapdump.hprof

# 3. Monitor closely after restart
watch -n 30 'jstat -gc <new-pid>'
```

**Root Cause Analysis:**
```bash
# 1. Compare multiple heap dumps
# 2. Look for objects that grow consistently
# 3. Check for:
#    - Unclosed resources
#    - Event listeners not removed
#    - Cache misconfigurations
#    - Static collections growing
```

---

## 8) Quick Production Workarounds

```
Emergency Response Decision Tree:
┌─────────────┐
│   Issue     │
│  Severity   │
└──────┬──────┘
       │
       ▼
┌─────────────┐     P4/P3   ┌─────────────┐
│ Critical    │────────────▶│  Schedule   │
│ P1/P2?      │             │   Planned   │
└─────────────┘             │   Fix       │
       │ Yes                └─────────────┘
       ▼
┌─────────────┐     No      ┌─────────────┐
│ Immediate   │────────────▶│  Apply      │
│ Downtime    │             │  Hotfix     │
│ Acceptable? │             └─────────────┘
└─────────────┘
       │ Yes
       ▼
┌─────────────┐
│  Emergency  │
│  Restart/   │
│  Rollback   │
└─────────────┘
```

```
Workaround Strategy Matrix:
┌─────────────────┬─────────────────┬─────────────────────────────┐
│    Issue Type   │   Immediate     │       Long-term             │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ Memory Leak     │ Restart Service │ Fix resource management     │
│ High CPU        │ Scale/Throttle  │ Optimize algorithms         │
│ Deadlock        │ Restart + Alert │ Implement timeouts          │
│ Thread Leak     │ Increase Limits │ Fix thread lifecycle        │
│ Connection Pool │ Increase Pool   │ Fix connection leaks        │
│ GC Issues       │ Tune Parameters │ Optimize object allocation  │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

### 8.1 Immediate Response Actions

#### Memory Issues

**Out of Memory - Quick Fixes:**
```bash
# 1. Restart with increased heap
export JAVA_OPTS="$JAVA_OPTS -Xmx8g -Xms4g"
systemctl restart your-application

# 2. Clear application caches (if applicable)
curl -X POST http://localhost:8080/admin/cache/clear

# 3. Force garbage collection
jcmd <pid> GC.run

# 4. Scale horizontally (add more instances)
kubectl scale deployment your-app --replicas=3
```

**Memory Leak - Temporary Mitigation:**
```bash
# 1. Scheduled restarts
cat > /etc/cron.d/app-restart << EOF
0 2 * * * root systemctl restart your-application
EOF

# 2. Memory threshold restart script
#!/bin/bash
# auto-restart-on-memory.sh
THRESHOLD=85
while true; do
  MEMORY_USAGE=$(jstat -gc <pid> | tail -1 | awk '{printf "%.0f", $8*100/$9}')
  if [ $MEMORY_USAGE -gt $THRESHOLD ]; then
    echo "Memory usage $MEMORY_USAGE% exceeds threshold. Restarting..."
    systemctl restart your-application
    sleep 300
  fi
  sleep 60
done
```

#### CPU Issues

**High CPU - Quick Fixes:**
```bash
# 1. Reduce thread pool sizes
curl -X PUT http://localhost:8080/admin/config/threads/max/10

# 2. Enable circuit breakers
curl -X POST http://localhost:8080/admin/circuit-breaker/enable

# 3. Add rate limiting
# Configure at load balancer or API gateway

# 4. Profile and kill problematic threads (extreme cases)
jstack <pid> | grep "high-cpu-operation" -A 5 -B 5
```

#### Thread Issues

**Deadlock Resolution:**
```bash
# 1. Restart application immediately
systemctl restart your-application

# 2. Temporary monitoring
watch -n 10 'jstack <pid> | grep -c "Found deadlock"'

# 3. Implement timeout-based operations
```

**Thread Pool Exhaustion:**
```bash
# 1. Increase thread pool size temporarily
curl -X PUT http://localhost:8080/admin/config/threads/core/50
curl -X PUT http://localhost:8080/admin/config/threads/max/100

# 2. Clear any blocking operations
curl -X POST http://localhost:8080/admin/tasks/clear

# 3. Restart if pools can't recover
```

### 8.2 Configuration Adjustments

#### JVM Tuning for Emergency

```bash
# Emergency JVM settings for high load
JAVA_OPTS="
-Xmx8g
-Xms4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+DisableExplicitGC
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/
-XX:OnOutOfMemoryError='kill -3 %p'
"
```

#### Application Configuration

**Spring Boot Emergency Settings:**
```properties
# application-emergency.properties

# Reduce connection pools
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5

# Reduce timeouts
spring.datasource.hikari.connection-timeout=5000
spring.transaction.default-timeout=30

# Disable non-essential features
management.endpoints.web.exposure.include=health,info
logging.level.com.yourcompany=WARN

# Reduce thread pools
server.tomcat.threads.max=50
server.tomcat.threads.min-spare=10
```

### 8.3 Monitoring and Alerting

#### Emergency Monitoring Script

```bash
#!/bin/bash
# emergency-monitor.sh

PID=$1
ALERT_EMAIL="ops@company.com"
LOG_FILE="/var/log/emergency-monitor.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

while true; do
    # Check if process is running
    if ! kill -0 $PID 2>/dev/null; then
        log "CRITICAL: Process $PID is not running!"
        echo "Process $PID died" | mail -s "CRITICAL: Application Down" $ALERT_EMAIL
        exit 1
    fi
    
    # Memory check
    MEMORY_USAGE=$(jstat -gc $PID | tail -1 | awk '{printf "%.0f", $8*100/$9}')
    if [ $MEMORY_USAGE -gt 90 ]; then
        log "CRITICAL: Memory usage at ${MEMORY_USAGE}%"
        jcmd $PID VM.heapdump /tmp/critical-heapdump-$(date +%Y%m%d-%H%M%S).hprof
        echo "Memory usage critical: ${MEMORY_USAGE}%" | mail -s "CRITICAL: High Memory" $ALERT_EMAIL
    fi
    
    # CPU check
    CPU_USAGE=$(ps -p $PID -o %cpu --no-headers | awk '{print int($1)}')
    if [ $CPU_USAGE -gt 95 ]; then
        log "CRITICAL: CPU usage at ${CPU_USAGE}%"
        jstack $PID > /tmp/critical-threaddump-$(date +%Y%m%d-%H%M%S).txt
    fi
    
    # Thread count check
    THREAD_COUNT=$(jstack $PID 2>/dev/null | grep "java.lang.Thread.State" | wc -l)
    if [ $THREAD_COUNT -gt 500 ]; then
        log "WARNING: High thread count: $THREAD_COUNT"
    fi
    
    sleep 30
done
```

#### Quick Health Check

```bash
#!/bin/bash
# quick-health-check.sh

check_application() {
    local pid=$1
    local app_name=$2
    
    echo "=== Health Check for $app_name (PID: $pid) ==="
    
    # Process status
    if kill -0 $pid 2>/dev/null; then
        echo "✓ Process is running"
    else
        echo "✗ Process is not running"
        return 1
    fi
    
    # Memory usage
    local memory_usage=$(jstat -gc $pid | tail -1 | awk '{printf "%.1f", $8*100/$9}')
    echo "Memory Usage: ${memory_usage}%"
    
    # CPU usage
    local cpu_usage=$(ps -p $pid -o %cpu --no-headers)
    echo "CPU Usage: ${cpu_usage}%"
    
    # Thread count
    local thread_count=$(jstack $pid 2>/dev/null | grep "java.lang.Thread.State" | wc -l)
    echo "Thread Count: $thread_count"
    
    # GC frequency (last 5 minutes)
    local gc_count=$(jstat -gc $pid | tail -1 | awk '{print $13}')
    echo "Total GC Count: $gc_count"
    
    # Application-specific health check
    if curl -s http://localhost:8080/actuator/health | grep -q "UP"; then
        echo "✓ Application health check passed"
    else
        echo "✗ Application health check failed"
    fi
    
    echo "==========================================="
}

# Usage
if [ $# -eq 0 ]; then
    echo "Usage: $0 <pid> [app-name]"
    exit 1
fi

check_application $1 ${2:-"Java Application"}
```

---

## 9) Automation and Monitoring

### 9.1 Automated Dump Collection

#### Systemd Service for Monitoring

```ini
# /etc/systemd/system/jvm-monitor.service
[Unit]
Description=JVM Monitoring Service
After=network.target

[Service]
Type=simple
User=monitoring
ExecStart=/usr/local/bin/jvm-monitor.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### Monitoring Script

```bash
#!/bin/bash
# /usr/local/bin/jvm-monitor.sh

CONFIG_FILE="/etc/jvm-monitor.conf"
source $CONFIG_FILE

DUMP_DIR="/var/dumps"
LOG_FILE="/var/log/jvm-monitor.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

collect_dumps() {
    local pid=$1
    local app_name=$2
    local timestamp=$(date +%Y%m%d-%H%M%S)
    
    # Memory usage check
    local memory_usage=$(jstat -gc $pid | tail -1 | awk '{printf "%.0f", $8*100/$9}')
    
    if [ $memory_usage -gt $MEMORY_THRESHOLD ]; then
        log "High memory usage detected for $app_name: ${memory_usage}%"
        
        # Take heap dump
        jcmd $pid VM.heapdump "$DUMP_DIR/${app_name}-heapdump-${timestamp}.hprof"
        log "Heap dump collected: ${app_name}-heapdump-${timestamp}.hprof"
        
        # Take thread dump
        jstack $pid > "$DUMP_DIR/${app_name}-threaddump-${timestamp}.txt"
        log "Thread dump collected: ${app_name}-threaddump-${timestamp}.txt"
        
        # Notify operations team
        echo "High memory usage in $app_name: ${memory_usage}%" | \
            mail -s "JVM Alert: High Memory" $ALERT_EMAIL
    fi
}

# Main monitoring loop
while true; do
    for app in "${MONITORED_APPS[@]}"; do
        IFS=':' read -r app_name pid <<< "$app"
        
        if kill -0 $pid 2>/dev/null; then
            collect_dumps $pid $app_name
        else
            log "WARNING: Process $pid for $app_name is not running"
        fi
    done
    
    sleep $CHECK_INTERVAL
done
```

#### Configuration File

```bash
# /etc/jvm-monitor.conf

# Monitoring settings
MEMORY_THRESHOLD=85
CPU_THRESHOLD=90
CHECK_INTERVAL=300  # 5 minutes

# Applications to monitor (format: name:pid)
MONITORED_APPS=(
    "billing-service:$(pgrep -f billing-service.jar)"
    "user-service:$(pgrep -f user-service.jar)"
    "payment-service:$(pgrep -f payment-service.jar)"
)

# Alert settings
ALERT_EMAIL="ops@company.com"
SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
```

### 9.2 Integration with Monitoring Tools

#### Prometheus Integration

```java
@Component
public class JVMMetricsCollector {
    
    private final MeterRegistry meterRegistry;
    
    public JVMMetricsCollector(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        initializeMetrics();
    }
    
    private void initializeMetrics() {
        // Memory metrics
        Gauge.builder("jvm.memory.heap.used")
            .register(meterRegistry, this, obj -> getHeapUsed());
        
        Gauge.builder("jvm.memory.heap.max")
            .register(meterRegistry, this, obj -> getHeapMax());
        
        // Thread metrics
        Gauge.builder("jvm.threads.count")
            .register(meterRegistry, this, obj -> getThreadCount());
        
        // GC metrics
        Gauge.builder("jvm.gc.collection.time")
            .register(meterRegistry, this, obj -> getGCTime());
    }
    
    private double getHeapUsed() {
        return ManagementFactory.getMemoryMXBean().getHeapMemoryUsage().getUsed();
    }
    
    private double getHeapMax() {
        return ManagementFactory.getMemoryMXBean().getHeapMemoryUsage().getMax();
    }
    
    private double getThreadCount() {
        return ManagementFactory.getThreadMXBean().getThreadCount();
    }
    
    private double getGCTime() {
        return ManagementFactory.getGarbageCollectorMXBeans().stream()
            .mapToLong(gcBean -> gcBean.getCollectionTime())
            .sum();
    }
}
```

#### Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "title": "JVM Monitoring Dashboard",
    "panels": [
      {
        "title": "Heap Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "jvm_memory_heap_used / jvm_memory_heap_max * 100",
            "legendFormat": "Heap Usage %"
          }
        ]
      },
      {
        "title": "Thread Count",
        "type": "graph",
        "targets": [
          {
            "expr": "jvm_threads_count",
            "legendFormat": "Thread Count"
          }
        ]
      },
      {
        "title": "GC Collection Time",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(jvm_gc_collection_time[5m])",
            "legendFormat": "GC Time Rate"
          }
        ]
      }
    ]
  }
}
```

### 9.3 Alerting Rules

#### Prometheus Alerting Rules

```yaml
# jvm-alerts.yml
groups:
- name: jvm-alerts
  rules:
  - alert: HighMemoryUsage
    expr: (jvm_memory_heap_used / jvm_memory_heap_max) * 100 > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High JVM memory usage"
      description: "JVM memory usage is above 85% for more than 5 minutes"
      
  - alert: CriticalMemoryUsage
    expr: (jvm_memory_heap_used / jvm_memory_heap_max) * 100 > 95
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Critical JVM memory usage"
      description: "JVM memory usage is above 95%"
      
  - alert: HighThreadCount
    expr: jvm_threads_count > 500
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High thread count"
      description: "JVM thread count is above 500"
      
  - alert: FrequentGC
    expr: rate(jvm_gc_collection_time[5m]) > 0.1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Frequent garbage collection"
      description: "GC is running frequently, indicating memory pressure"
```

---

## 10) Best Practices and Checklists

### 10.1 Production Deployment Checklist

```
Production Readiness Framework:
┌─────────────────────────────────────────────────────────────────┐
│                     Pre-Production                             │
├──────────────┬──────────────┬──────────────┬──────────────────────┤
│    Code      │    Config    │   Testing    │      Monitoring      │
├──────────────┼──────────────┼──────────────┼──────────────────────┤
│ • Security   │ • JVM Flags  │ • Load Test  │ • Metrics Setup      │
│ • Performance│ • Resources  │ • Chaos Eng  │ • Alerting Rules     │
│ • Logging    │ • Env Vars   │ • Failover   │ • Dashboard Config   │
│ • Error      │ • Secrets    │ • Recovery   │ • Log Aggregation    │
│   Handling   │   Management │   Testing    │                      │
└──────────────┴──────────────┴──────────────┴──────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Production                                 │
├──────────────┬──────────────┬──────────────┬──────────────────────┤
│  Deployment  │  Monitoring  │   Support    │     Operations       │
├──────────────┼──────────────┼──────────────┼──────────────────────┤
│ • Blue/Green │ • Real-time  │ • Runbooks   │ • Backup/Recovery    │
│ • Canary     │ • Alerting   │ • Escalation │ • Capacity Planning  │
│ • Rolling    │ • Dashboards │ • On-call    │ • Performance Tuning │
│ • Rollback   │ • Log Review │ • Knowledge  │ • Security Updates   │
│   Strategy   │              │   Base       │                      │
└──────────────┴──────────────┴──────────────┴──────────────────────┘
```

#### JVM Configuration Checklist

```bash
# ✅ Essential JVM settings for production

# Memory settings
-Xmx4g                              # Maximum heap size
-Xms2g                              # Initial heap size
-XX:NewRatio=3                      # Old:Young generation ratio

# GC settings
-XX:+UseG1GC                        # Use G1 garbage collector
-XX:MaxGCPauseMillis=200           # Target GC pause time
-XX:G1HeapRegionSize=16m           # G1 region size

# Dump settings
-XX:+HeapDumpOnOutOfMemoryError    # Auto heap dump on OOM
-XX:HeapDumpPath=/var/dumps/       # Dump location
-XX:OnOutOfMemoryError="kill -3 %p" # Take thread dump on OOM

# Logging
-Xloggc:/var/log/gc.log            # GC log location
-XX:+UseGCLogFileRotation          # Rotate GC logs
-XX:NumberOfGCLogFiles=5           # Number of log files
-XX:GCLogFileSize=100M             # Max log file size

# JMX (for monitoring)
-Dcom.sun.management.jmxremote=true
-Dcom.sun.management.jmxremote.port=9999
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

#### Monitoring Setup Checklist

```bash
# ✅ Production monitoring essentials

□ APM tool configured (New Relic, AppDynamics, etc.)
□ JMX enabled for remote monitoring
□ GC logging enabled and rotated
□ Heap dump on OOM configured
□ Custom metrics exposed (Prometheus/Micrometer)
□ Health check endpoints available
□ Log aggregation configured (ELK, Splunk)
□ Alerting rules configured
□ Dashboard created (Grafana, DataDog)
□ Runbook documented for common issues
```

### 10.2 Troubleshooting Workflow

#### Step-by-Step Diagnostic Process

```
Diagnostic Process Timeline:
┌─────────────────────────────────────────────────────────────────┐
│  Phase 1: Initial Assessment (0-5 minutes)                     │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │
│  │   Check     │ │   Process    │ │   Quick     │ │  Alert  │ │
│  │  Service    │▶│   Status     │▶│   Health    │▶│  Team   │ │
│  │  Status     │ │              │ │   Check     │ │         │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  Phase 2: Data Collection (5-15 minutes)                       │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │
│  │   Thread    │ │   Heap      │ │   System    │ │   App   │ │
│  │   Dump      │▶│   Analysis  │▶│   Metrics   │▶│  Logs   │ │
│  │             │ │             │ │             │ │         │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  Phase 3: Root Cause Analysis (15-30 minutes)                  │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐ │
│  │  Correlate  │ │  Identify   │ │  Validate   │ │  Plan   │ │
│  │   Data      │▶│  Pattern    │▶│ Hypothesis  │▶│  Fix    │ │
│  │             │ │             │ │             │ │         │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# 1. Initial Assessment (2-5 minutes)
echo "=== Initial Assessment ==="
jps -v                              # List Java processes
ps aux | grep java                  # Process details
top -p <pid>                        # CPU/Memory usage
netstat -an | grep <port>          # Network status

# 2. Quick Health Check (5 minutes)
echo "=== Quick Health Check ==="
jstat -gc <pid>                     # GC status
jcmd <pid> VM.uptime               # JVM uptime
jcmd <pid> VM.flags                # JVM flags
curl http://localhost:8080/actuator/health  # App health

# 3. Detailed Analysis (10-15 minutes)
echo "=== Detailed Analysis ==="
jstack <pid> > threaddump.txt      # Thread dump
jcmd <pid> GC.class_histogram | head -20  # Memory usage
jstat -gc <pid> 5s 12              # GC monitoring

# 4. Data Collection (if needed)
echo "=== Data Collection ==="
jcmd <pid> VM.heapdump heapdump.hprof  # Heap dump
jcmd <pid> Thread.print > threaddump.txt  # Detailed thread dump
```

#### Common Issues Quick Reference

| **Symptom** | **Likely Cause** | **Quick Check** | **Resolution** |
|-------------|------------------|-----------------|----------------|
| High CPU | Infinite loop, inefficient algorithm | `top -H -p <pid>`, thread dump | Identify hot threads, optimize code |
| High Memory | Memory leak, large objects | Heap histogram, heap dump | Analyze dominator tree, fix leaks |
| App Hanging | Deadlock, blocking I/O | Thread dump, deadlock detection | Restart, fix synchronization |
| Slow Response | GC pressure, thread contention | GC stats, thread dump | Tune GC, optimize locks |
| OutOfMemory | Insufficient heap, memory leak | Heap dump, GC logs | Increase heap, fix leaks |

### 10.3 Emergency Response Procedures

#### Critical Issue Response (P0/P1)

```
Emergency Response Timeline:
┌─────────────────────────────────────────────────────────────────┐
│ 0-5 MIN: IMMEDIATE RESPONSE                                     │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │   Assess    │▶│   Alert     │▶│   Collect   │▶│   Decide    │ │
│ │   Impact    │ │    Team     │ │   Basic     │ │  Action     │ │
│ │             │ │             │ │   Data      │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ 5-15 MIN: STABILIZATION                                        │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │   Scale     │▶│   Circuit   │▶│   Restart   │▶│  Monitor    │ │
│ │ Resources   │ │  Breakers   │ │  Services   │ │ Recovery    │ │
│ │             │ │             │ │             │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ 15-30 MIN: ROOT CAUSE                                          │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │   Deep      │▶│  Identify   │▶│  Develop    │▶│   Test      │ │
│ │ Analysis    │ │ Root Cause  │ │    Fix      │ │    Fix      │ │
│ │             │ │             │ │             │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│ 30+ MIN: RESOLUTION & PREVENTION                               │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│ │   Deploy    │▶│   Verify    │▶│  Document   │▶│  Improve    │ │
│ │    Fix      │ │ Stability   │ │ Learnings   │ │ Process     │ │
│ │             │ │             │ │             │ │             │ │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# === EMERGENCY RESPONSE CHECKLIST ===

# 1. Immediate Actions (0-5 minutes)
□ Confirm issue scope and impact
□ Check if service is responding: curl -I http://app:8080/health
□ Take thread dump: jstack <pid> > emergency-threaddump.txt
□ Check system resources: top, df -h, free -m
□ Notify team via incident channel

# 2. Stabilization (5-15 minutes)
□ Take heap dump if memory related: jcmd <pid> VM.heapdump emergency.hprof
□ Scale horizontally if possible: kubectl scale deployment app --replicas=3
□ Apply circuit breakers: curl -X POST app:8080/admin/circuit-breaker/enable
□ Restart service if hanging: systemctl restart app

# 3. Investigation (15-30 minutes)
□ Analyze thread dump for deadlocks/blocking
□ Check recent deployments and changes
□ Review application logs for errors
□ Analyze heap dump if available
□ Monitor key metrics: CPU, memory, GC

# 4. Resolution and Recovery (30+ minutes)
□ Apply fix if root cause identified
□ Deploy hotfix if necessary
□ Verify system stability
□ Document incident and learnings
□ Schedule post-mortem review
```

#### Post-Incident Analysis Template

```
Post-Mortem Analysis Framework:
┌─────────────────────────────────────────────────────────────────┐
│                       INCIDENT TIMELINE                        │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Detection     │   Response      │         Resolution          │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ • When found    │ • Actions taken │ • Root cause identified    │
│ • How detected  │ • Tools used    │ • Fix implemented           │
│ • Initial       │ • Time to       │ • System restored           │
│   assessment    │   respond       │ • Verification completed    │
└─────────────────┴─────────────────┴─────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      LESSONS LEARNED                           │
├─────────────────┬─────────────────┬─────────────────────────────┤
│  What Worked    │ What Failed     │        Improvements         │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ • Good          │ • Gaps in       │ • Monitoring enhancements  │
│   responses     │   monitoring    │ • Process improvements      │
│ • Effective     │ • Slow          │ • Tool upgrades             │
│   tools         │   detection     │ • Training needs            │
│ • Team          │ • Missing       │ • Documentation updates     │
│   coordination  │   automation    │                             │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

```markdown
# Incident Post-Mortem

## Incident Summary
- **Date/Time**: 
- **Duration**: 
- **Severity**: 
- **Impact**: 

## Timeline
- **Detection**: How was the issue detected?
- **Response**: What actions were taken?
- **Resolution**: How was it resolved?

## Root Cause Analysis
- **Primary Cause**: 
- **Contributing Factors**: 
- **Evidence**: (heap dumps, thread dumps, logs)

## Lessons Learned
- **What went well**: 
- **What could be improved**: 
- **Action items**: 

## Prevention Measures
- **Monitoring improvements**: 
- **Code changes**: 
- **Process improvements**: 
```

---


