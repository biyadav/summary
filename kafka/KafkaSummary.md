Apache Kafka Complete Study Guide
Table of Contents
Kafka Architecture
Important Configurations
Consumer Rebalancing
Transaction Handling
Acknowledgments and Delivery Guarantees
Exactly-Once Semantics
Idempotency
Consumer Lag Troubleshooting
Monitoring
Advanced Debugging and Troubleshooting
Performance Tuning
Best Practices
Common Use Cases
Kafka Architecture
Core Components
1. Broker
Definition: A Kafka server that stores and serves data
Role: Handles producer writes, consumer reads, and replication
Cluster: Multiple brokers form a Kafka cluster
Leader/Follower: Each partition has one leader broker and multiple follower brokers
2. Topics and Partitions
Topic: user-events
├── Partition 0: [msg1, msg2, msg3, msg4]
├── Partition 1: [msg5, msg6, msg7, msg8]
└── Partition 2: [msg9, msg10, msg11, msg12]
Key Concepts:

Topic: Logical category/feed name to which records are published
Partition: Ordered, immutable sequence of records
Offset: Unique identifier for each record within a partition
Retention: How long messages are kept (time or size-based)
3. Replication
Topic: orders (replication-factor=3)
Partition 0:
├── Leader: Broker 1
├── Follower: Broker 2 (In-Sync Replica)
└── Follower: Broker 3 (In-Sync Replica)
Replication Concepts:

Replication Factor: Number of copies of each partition
In-Sync Replicas (ISR): Replicas that are caught up with the leader
Leader Election: Process of choosing new leader when current leader fails
Unclean Leader Election: Allowing out-of-sync replicas to become leaders
4. ZooKeeper vs KRaft
ZooKeeper (Legacy)

External dependency for metadata management
Stores cluster metadata, configuration, and leader election
Single point of failure and complexity
KRaft (Kafka Raft) - New Default

Self-managed metadata using Raft consensus protocol
No external ZooKeeper dependency
Better scalability and simpler operations
KRaft Architecture:
Controller Nodes (Metadata management)
├── Controller 1 (Leader)
├── Controller 2 (Follower)
└── Controller 3 (Follower)

Broker Nodes (Data storage)
├── Broker 1
├── Broker 2
└── Broker 3
5. Producers
Producer Workflow:

Serialize key and value
Determine partition (via partitioner)
Add to batch for target broker
Send batch when full or linger time reached
Receive acknowledgment based on acks setting
Partitioning Strategies:

Round Robin: Default when no key is provided
Key-based: Hash of key determines partition
Custom Partitioner: User-defined partitioning logic
6. Consumers and Consumer Groups
Consumer Group: payment-processors
├── Consumer 1: Partitions [0, 1]
├── Consumer 2: Partitions [2, 3]
└── Consumer 3: Partitions [4, 5]
Consumer Concepts:

Consumer Group: Logical grouping of consumers
Partition Assignment: Each partition consumed by exactly one consumer in group
Offset Management: Tracking position in each partition
Commit Strategies: Auto-commit vs manual commit
Important Configurations
Broker Configurations
Essential Settings
# Server Basics
broker.id=1
listeners=PLAINTEXT://localhost:9092
log.dirs=/var/kafka-logs

# Replication
default.replication.factor=3
min.insync.replicas=2
unclean.leader.election.enable=false

# Log Management
log.retention.hours=168
log.retention.bytes=1073741824
log.segment.bytes=1073741824
log.cleanup.policy=delete

# Performance
num.network.threads=8
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Memory and Processing
num.replica.fetchers=4
replica.fetch.max.bytes=1048576
Advanced Broker Settings
# Compression
compression.type=snappy

# Transaction Support
transaction.state.log.replication.factor=3
transaction.state.log.min.isr=2

# Security
security.inter.broker.protocol=SASL_SSL
sasl.mechanism.inter.broker.protocol=PLAIN

# Monitoring
metric.reporters=io.confluent.metrics.reporter.ConfluentMetricsReporter
confluent.metrics.reporter.bootstrap.servers=localhost:9092
Producer Configurations
Performance-Critical Settings
# Acknowledgment
acks=all  # or -1 for strongest durability
retries=2147483647
enable.idempotence=true

# Batching and Compression
batch.size=16384
linger.ms=5
compression.type=snappy

# Memory Management
buffer.memory=33554432
max.block.ms=60000

# Reliability
max.in.flight.requests.per.connection=5
request.timeout.ms=30000
delivery.timeout.ms=120000

# Partitioning
partitioner.class=org.apache.kafka.clients.producer.internals.DefaultPartitioner
Producer Configuration Examples
// High Throughput Producer
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "1");
props.put("batch.size", 65536);
props.put("linger.ms", 10);
props.put("compression.type", "snappy");
props.put("buffer.memory", 67108864);

// High Reliability Producer
Properties reliableProps = new Properties();
reliableProps.put("bootstrap.servers", "localhost:9092");
reliableProps.put("acks", "all");
reliableProps.put("retries", Integer.MAX_VALUE);
reliableProps.put("enable.idempotence", true);
reliableProps.put("max.in.flight.requests.per.connection", 1);
Consumer Configurations
Essential Consumer Settings
# Connection
bootstrap.servers=localhost:9092
group.id=my-consumer-group

# Deserialization
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

# Offset Management
enable.auto.commit=false
auto.commit.interval.ms=5000
auto.offset.reset=earliest

# Session Management
session.timeout.ms=30000
heartbeat.interval.ms=3000
max.poll.interval.ms=300000
max.poll.records=500

# Fetch Settings
fetch.min.bytes=1
fetch.max.wait.ms=500
max.partition.fetch.bytes=1048576
Consumer Configuration Examples
// High Throughput Consumer
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "high-throughput-group");
props.put("max.poll.records", 1000);
props.put("fetch.max.bytes", 52428800);
props.put("max.partition.fetch.bytes", 2097152);

// Low Latency Consumer
Properties lowLatencyProps = new Properties();
lowLatencyProps.put("bootstrap.servers", "localhost:9092");
lowLatencyProps.put("group.id", "low-latency-group");
lowLatencyProps.put("fetch.min.bytes", 1);
lowLatencyProps.put("fetch.max.wait.ms", 1);
Consumer Rebalancing
Rebalancing Triggers
Consumer joins the group
Consumer leaves the group (graceful shutdown or crash)
Consumer is considered dead (missed heartbeats)
Topic partition count changes
Subscription pattern changes
Rebalancing Process
1. Consumer sends JoinGroup request to Group Coordinator
2. Group Coordinator selects a Consumer Leader
3. Consumer Leader receives group membership and subscription info
4. Consumer Leader runs partition assignment strategy
5. Consumer Leader sends assignment back to Group Coordinator
6. Group Coordinator sends assignments to all consumers
7. Consumers start consuming from assigned partitions
Partition Assignment Strategies
1. Range Assignor (Default)
Topic: events (6 partitions), 3 consumers
Consumer 1: [0, 1]
Consumer 2: [2, 3]
Consumer 3: [4, 5]
2. Round Robin Assignor
Topic A: [0, 1, 2], Topic B: [0, 1, 2], 3 consumers
Consumer 1: [A0, B1]
Consumer 2: [A1, B2]
Consumer 3: [A2, B0]
3. Sticky Assignor
# Minimizes partition movement during rebalancing
# Before rebalancing:
Consumer 1: [0, 1, 2]
Consumer 2: [3, 4, 5]

# After Consumer 3 joins (Sticky keeps existing assignments):
Consumer 1: [0, 1]
Consumer 2: [3, 4]
Consumer 3: [2, 5]
4. Cooperative Sticky Assignor
Incremental Rebalancing: Only revokes partitions that need to be moved
Reduced Downtime: Consumers continue processing unaffected partitions
Better Performance: Minimal disruption during rebalancing
Rebalancing Configuration
# Rebalancing Timeouts
session.timeout.ms=30000          # How long before consumer is considered dead
heartbeat.interval.ms=3000        # How often to send heartbeats
max.poll.interval.ms=300000       # Max time between poll() calls

# Assignment Strategy
partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor

# Rebalancing Listeners
Handling Rebalancing in Code
public class RebalanceListener implements ConsumerRebalanceListener {
    private Map<TopicPartition, OffsetAndMetadata> currentOffsets = new HashMap<>();
    
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        // Commit offsets for partitions being revoked
        consumer.commitSync(currentOffsets);
        System.out.println("Partitions revoked: " + partitions);
    }
    
    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        // Initialize processing for newly assigned partitions
        System.out.println("Partitions assigned: " + partitions);
    }
}

// Usage
consumer.subscribe(Arrays.asList("my-topic"), new RebalanceListener());
Minimizing Rebalancing Impact
Tune session timeout and heartbeat interval
Use cooperative assignors
Implement proper rebalance listeners
Graceful consumer shutdown
Monitor max.poll.interval.ms
Advanced Consumer Groups and Rebalancing
Consumer Group Lifecycle and States
Consumer Group States
Consumer Group States:
├── Empty: No active members
├── PreparingRebalance: Coordinator waiting for members to rejoin
├── CompletingRebalance: Coordinator waiting for assignment acknowledgment
├── Stable: All members have joined and received assignments
└── Dead: Group has been removed
Detailed Consumer Group Mechanics
public class ConsumerGroupLifecycle {
    
    public void demonstrateGroupStates() {
        // State 1: Empty -> PreparingRebalance
        // When first consumer joins
        consumer1.subscribe(Arrays.asList("my-topic"));
        consumer1.poll(Duration.ofMillis(100)); // Triggers join group
        
        // State 2: PreparingRebalance -> CompletingRebalance
        // Coordinator sends assignments to group leader
        
        // State 3: CompletingRebalance -> Stable
        // All members acknowledge assignments
        
        // State 4: Stable -> PreparingRebalance
        // When consumer2 joins the group
        consumer2.subscribe(Arrays.asList("my-topic"));
        consumer2.poll(Duration.ofMillis(100)); // Triggers rebalance
    }
}
Group Coordinator Selection
public class GroupCoordinatorLogic {
    
    public int determineGroupCoordinator(String groupId, int numBrokers) {
        // Coordinator is determined by hash of group ID
        int coordinatorId = Math.abs(groupId.hashCode()) % numBrokers;
        
        // The coordinator broker manages:
        // 1. Group membership
        // 2. Offset commits
        // 3. Rebalancing coordination
        
        return coordinatorId;
    }
    
    public void handleCoordinatorFailover() {
        // When coordinator fails:
        // 1. New coordinator is selected
        // 2. Group enters PreparingRebalance state
        // 3. All consumers must rejoin the group
        // 4. New assignments are calculated
        
        log.info("Coordinator failover triggers full rebalance");
    }
}
Comprehensive Rebalancing Causes
1. Consumer Lifecycle Events
public class RebalancingTriggers {
    
    // Cause 1: Consumer joins the group
    public void consumerJoinsGroup() {
        KafkaConsumer<String, String> newConsumer = new KafkaConsumer<>(props);
        newConsumer.subscribe(Arrays.asList("my-topic")); // Subscription
        newConsumer.poll(Duration.ofMillis(100)); // First poll triggers join
        
        log.info("New consumer joining triggers rebalance");
    }
    
    // Cause 2: Consumer leaves gracefully
    public void consumerLeavesGracefully() {
        consumer.close(); // Sends LeaveGroup request
        log.info("Graceful consumer shutdown triggers rebalance");
    }
    
    // Cause 3: Consumer crashes or becomes unresponsive
    public void consumerBecomesUnresponsive() {
        // If consumer doesn't send heartbeat within session.timeout.ms
        // Coordinator marks consumer as failed
        // Triggers rebalance to redistribute partitions
        
        simulateSlowProcessing(); // Causes missed heartbeats
        log.info("Consumer failure detected, rebalance triggered");
    }
    
    // Cause 4: Consumer processing takes too long
    public void consumerProcessingTimeout() {
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            
            // If this processing takes longer than max.poll.interval.ms
            // consumer is considered failed
            processRecordsVerySlowly(records); // > max.poll.interval.ms
            
            // Next poll() will trigger rebalance
        }
    }
}
2. Topic and Partition Changes
public class TopicChangeRebalancing {
    
    // Cause 5: Topic partition count increases
    public void partitionCountIncrease() {
        // Admin operation to increase partitions
        AdminClient adminClient = AdminClient.create(adminProps);
        
        Map<String, NewPartitions> partitionsToCreate = new HashMap<>();
        partitionsToCreate.put("my-topic", NewPartitions.increaseTo(10)); // from 6 to 10
        
        adminClient.createPartitions(partitionsToCreate);
        // All consumer groups subscribed to this topic will rebalance
        
        log.info("Partition increase triggers rebalance for all consumer groups");
    }
    
    // Cause 6: Topic deletion
    public void topicDeletion() {
        AdminClient adminClient = AdminClient.create(adminProps);
        adminClient.deleteTopics(Arrays.asList("my-topic"));
        
        // Consumers subscribed to deleted topic will rebalance
        log.info("Topic deletion triggers rebalance");
    }
    
    // Cause 7: Topic creation (for pattern subscription)
    public void newTopicCreatedWithPattern() {
        // Consumer subscribed with pattern
        consumer.subscribe(Pattern.compile("user-.*"));
        
        // When new topic "user-events-new" is created
        AdminClient adminClient = AdminClient.create(adminProps);
        NewTopic newTopic = new NewTopic("user-events-new", 3, (short) 3);
        adminClient.createTopics(Arrays.asList(newTopic));
        
        // Consumer will detect new topic and rebalance
        log.info("New topic matching pattern triggers rebalance");
    }
}
3. Configuration and Network Issues
public class ConfigurationRebalancing {
    
    // Cause 8: Subscription changes
    public void subscriptionChange() {
        // Initial subscription
        consumer.subscribe(Arrays.asList("topic-a", "topic-b"));
        consumer.poll(Duration.ofMillis(100));
        
        // Changed subscription triggers rebalance
        consumer.subscribe(Arrays.asList("topic-a", "topic-b", "topic-c"));
        consumer.poll(Duration.ofMillis(100)); // Rebalance triggered
        
        log.info("Subscription change triggers rebalance");
    }
    
    // Cause 9: Network partitions
    public void networkPartition() {
        // When consumer can't reach coordinator due to network issues
        // Consumer will be removed from group after session timeout
        
        simulateNetworkPartition();
        // After network recovers, consumer rejoin triggers rebalance
        
        log.info("Network partition recovery triggers rebalance");
    }
    
    // Cause 10: Coordinator broker failure
    public void coordinatorFailure() {
        // When the group coordinator broker fails:
        // 1. New coordinator is elected
        // 2. Group state is lost temporarily
        // 3. All consumers must rediscover coordinator
        // 4. Full rebalance occurs
        
        log.info("Coordinator failure triggers full group rebalance");
    }
}
Rebalancing Configuration Tuning
Essential Rebalancing Parameters
# Session Management
session.timeout.ms=30000              # How long before consumer is considered dead
heartbeat.interval.ms=3000            # How often to send heartbeats (should be 1/3 of session timeout)
max.poll.interval.ms=300000           # Max time between poll() calls

# Rebalancing Behavior
partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor
rebalance.timeout.ms=60000            # Max time for rebalance to complete

# Advanced Settings
metadata.max.age.ms=300000            # How often to refresh topic metadata
connections.max.idle.ms=540000        # Close idle connections after this time
request.timeout.ms=30000              # Client request timeout
Tuning for Different Scenarios
1. High-Throughput, Low-Latency Scenario

public class HighThroughputRebalancingConfig {
    
    public Properties getOptimizedConfig() {
        Properties props = new Properties();
        
        // Faster failure detection
        props.put("session.timeout.ms", "10000");        // 10 seconds
        props.put("heartbeat.interval.ms", "3000");      // 3 seconds
        props.put("max.poll.interval.ms", "30000");      // 30 seconds for fast processing
        
        // Cooperative rebalancing for minimal downtime
        props.put("partition.assignment.strategy", 
                 "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
        
        // Faster metadata refresh
        props.put("metadata.max.age.ms", "60000");       // 1 minute
        
        return props;
    }
}
2. Batch Processing Scenario

public class BatchProcessingRebalancingConfig {
    
    public Properties getBatchConfig() {
        Properties props = new Properties();
        
        // Longer timeouts for batch processing
        props.put("session.timeout.ms", "60000");        // 1 minute
        props.put("heartbeat.interval.ms", "20000");     // 20 seconds
        props.put("max.poll.interval.ms", "1800000");    // 30 minutes for long batch jobs
        
        // Longer rebalance timeout for large batches
        props.put("rebalance.timeout.ms", "300000");     // 5 minutes
        
        // Sticky assignor to minimize partition movement
        props.put("partition.assignment.strategy", 
                 "org.apache.kafka.clients.consumer.StickyAssignor");
        
        return props;
    }
}
3. Reliable Processing Scenario

public class ReliableProcessingRebalancingConfig {
    
    public Properties getReliableConfig() {
        Properties props = new Properties();
        
        // Conservative timeouts
        props.put("session.timeout.ms", "45000");        // 45 seconds
        props.put("heartbeat.interval.ms", "15000");     // 15 seconds
        props.put("max.poll.interval.ms", "600000");     // 10 minutes
        
        // Disable auto-commit for manual offset management
        props.put("enable.auto.commit", "false");
        
        // Longer request timeouts for reliability
        props.put("request.timeout.ms", "60000");        // 1 minute
        
        return props;
    }
}
Rebalancing Protocols
Eager Rebalancing Protocol (Legacy)
public class EagerRebalancingProtocol {
    
    /*
     * Eager Rebalancing Process:
     * 1. Stop-the-world: All consumers stop processing
     * 2. Revoke all partitions
     * 3. Rejoin group
     * 4. Receive new assignments
     * 5. Resume processing
     * 
     * Downside: Complete processing halt during rebalance
     */
    
    public void demonstrateEagerRebalancing() {
        Properties props = new Properties();
        props.put("partition.assignment.strategy", 
                 "org.apache.kafka.clients.consumer.RangeAssignor");
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        
        consumer.subscribe(Arrays.asList("my-topic"), new ConsumerRebalanceListener() {
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                log.info("EAGER: All partitions revoked: {}", partitions);
                // All partitions are revoked at once
                // Processing stops completely
                commitOffsets(partitions);
            }
            
            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                log.info("EAGER: New partitions assigned: {}", partitions);
                // Receive completely new assignment
                // Processing resumes
            }
        });
    }
}
Cooperative Rebalancing Protocol (Modern)
public class CooperativeRebalancingProtocol {
    
    /*
     * Cooperative Rebalancing Process:
     * 1. Identify partitions that need to move
     * 2. Revoke only those partitions
     * 3. Keep processing other partitions
     * 4. Assign revoked partitions to new consumers
     * 5. Minimal processing disruption
     */
    
    public void demonstrateCooperativeRebalancing() {
        Properties props = new Properties();
        props.put("partition.assignment.strategy", 
                 "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        
        consumer.subscribe(Arrays.asList("my-topic"), new ConsumerRebalanceListener() {
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                log.info("COOPERATIVE: Only moving partitions revoked: {}", partitions);
                // Only partitions that need to move are revoked
                // Other partitions continue processing
                commitOffsets(partitions);
            }
            
            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                log.info("COOPERATIVE: New partitions assigned: {}", partitions);
                // Receive additional partitions
                // Existing partitions continue processing
            }
            
            @Override
            public void onPartitionsLost(Collection<TopicPartition> partitions) {
                log.warn("COOPERATIVE: Partitions lost due to error: {}", partitions);
                // Handle partition loss scenario
                handlePartitionLoss(partitions);
            }
        });
    }
}
Custom Rebalancing Interfaces and Logic
Custom Partition Assignment Strategy
public class CustomPartitionAssignor implements ConsumerPartitionAssignor {
    
    @Override
    public String name() {
        return "custom-priority-assignor";
    }
    
    @Override
    public GroupAssignment assign(Cluster metadata, GroupSubscription subscriptions) {
        Map<String, Assignment> assignments = new HashMap<>();
        
        // Get all partitions for subscribed topics
        Set<String> allTopics = new HashSet<>();
        Map<String, Subscription> consumerSubscriptions = subscriptions.groupSubscription();
        
        for (Subscription subscription : consumerSubscriptions.values()) {
            allTopics.addAll(subscription.topics());
        }
        
        List<TopicPartition> allPartitions = new ArrayList<>();
        for (String topic : allTopics) {
            Integer partitionCount = metadata.partitionCountForTopic(topic);
            if (partitionCount != null) {
                for (int i = 0; i < partitionCount; i++) {
                    allPartitions.add(new TopicPartition(topic, i));
                }
            }
        }
        
        // Custom assignment logic
        assignments.putAll(assignWithPriority(consumerSubscriptions, allPartitions));
        
        return new GroupAssignment(assignments);
    }
    
    private Map<String, Assignment> assignWithPriority(
            Map<String, Subscription> subscriptions, 
            List<TopicPartition> partitions) {
        
        Map<String, Assignment> assignments = new HashMap<>();
        List<String> consumers = new ArrayList<>(subscriptions.keySet());
        
        // Sort consumers by priority (from user data)
        consumers.sort((c1, c2) -> {
            int priority1 = getPriority(subscriptions.get(c1));
            int priority2 = getPriority(subscriptions.get(c2));
            return Integer.compare(priority2, priority1); // Higher priority first
        });
        
        // Assign partitions round-robin style, but prioritize high-priority consumers
        int consumerIndex = 0;
        for (TopicPartition partition : partitions) {
            String consumer = consumers.get(consumerIndex % consumers.size());
            
            assignments.computeIfAbsent(consumer, k -> new Assignment(new ArrayList<>()))
                      .partitions().add(partition);
            
            consumerIndex++;
        }
        
        return assignments;
    }
    
    private int getPriority(Subscription subscription) {
        // Extract priority from user data
        ByteBuffer userData = subscription.userData();
        if (userData != null && userData.remaining() >= 4) {
            return userData.getInt();
        }
        return 0; // Default priority
    }
    
    @Override
    public void onAssignment(Assignment assignment, ConsumerGroupMetadata metadata) {
        // Called when assignment is received
        log.info("Custom assignment received for consumer {}: {}", 
                metadata.memberId(), assignment.partitions());
    }
}
Priority-Based Consumer Implementation
public class PriorityConsumer {
    
    public KafkaConsumer<String, String> createPriorityConsumer(int priority) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "priority-consumer-group");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", StringDeserializer.class.getName());
        
        // Use custom assignor
        props.put("partition.assignment.strategy", CustomPartitionAssignor.class.getName());
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        
        // Subscribe with priority in user data
        consumer.subscribe(
            Arrays.asList("priority-topic"), 
            new PriorityRebalanceListener(priority)
        );
        
        return consumer;
    }
    
    private static class PriorityRebalanceListener implements ConsumerRebalanceListener {
        private final int priority;
        
        public PriorityRebalanceListener(int priority) {
            this.priority = priority;
        }
        
        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            log.info("Priority {} consumer: partitions revoked {}", priority, partitions);
        }
        
        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            log.info("Priority {} consumer: partitions assigned {}", priority, partitions);
        }
    }
}
Advanced Custom Rebalancing Logic
public class AdvancedCustomRebalancing {
    
    // Custom assignor that considers consumer capacity
    public static class CapacityAwareAssignor implements ConsumerPartitionAssignor {
        
        @Override
        public String name() {
            return "capacity-aware-assignor";
        }
        
        @Override
        public GroupAssignment assign(Cluster metadata, GroupSubscription subscriptions) {
            Map<String, Assignment> assignments = new HashMap<>();
            Map<String, Subscription> consumerSubscriptions = subscriptions.groupSubscription();
            
            // Parse consumer capacities from user data
            Map<String, Integer> consumerCapacities = new HashMap<>();
            for (Map.Entry<String, Subscription> entry : consumerSubscriptions.entrySet()) {
                String consumerId = entry.getKey();
                ByteBuffer userData = entry.getValue().userData();
                int capacity = parseCapacity(userData);
                consumerCapacities.put(consumerId, capacity);
            }
            
            // Get all partitions
            List<TopicPartition> allPartitions = getAllPartitions(metadata, consumerSubscriptions);
            
            // Assign partitions based on capacity
            assignments.putAll(assignByCapacity(allPartitions, consumerCapacities));
            
            return new GroupAssignment(assignments);
        }
        
        private Map<String, Assignment> assignByCapacity(
                List<TopicPartition> partitions, 
                Map<String, Integer> capacities) {
            
            Map<String, Assignment> assignments = new HashMap<>();
            Map<String, Integer> currentLoads = new HashMap<>();
            
            // Initialize assignments and loads
            for (String consumer : capacities.keySet()) {
                assignments.put(consumer, new Assignment(new ArrayList<>()));
                currentLoads.put(consumer, 0);
            }
            
            // Assign each partition to the consumer with lowest load relative to capacity
            for (TopicPartition partition : partitions) {
                String bestConsumer = findBestConsumer(capacities, currentLoads);
                
                assignments.get(bestConsumer).partitions().add(partition);
                currentLoads.put(bestConsumer, currentLoads.get(bestConsumer) + 1);
            }
            
            return assignments;
        }
        
        private String findBestConsumer(Map<String, Integer> capacities, 
                                      Map<String, Integer> currentLoads) {
            String bestConsumer = null;
            double lowestLoadRatio = Double.MAX_VALUE;
            
            for (String consumer : capacities.keySet()) {
                int capacity = capacities.get(consumer);
                int load = currentLoads.get(consumer);
                double loadRatio = (double) load / capacity;
                
                if (loadRatio < lowestLoadRatio) {
                    lowestLoadRatio = loadRatio;
                    bestConsumer = consumer;
                }
            }
            
            return bestConsumer;
        }
        
        private int parseCapacity(ByteBuffer userData) {
            if (userData != null && userData.remaining() >= 4) {
                return userData.getInt();
            }
            return 1; // Default capacity
        }
    }
    
    // Consumer factory with capacity specification
    public KafkaConsumer<String, String> createCapacityAwareConsumer(
            String groupId, int capacity) {
        
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", groupId);
        props.put("partition.assignment.strategy", CapacityAwareAssignor.class.getName());
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        
        // Encode capacity in user data
        ByteBuffer userData = ByteBuffer.allocate(4);
        userData.putInt(capacity);
        userData.flip();
        
        consumer.subscribe(
            Arrays.asList("workload-topic"),
            new CapacityAwareRebalanceListener(capacity)
        );
        
        return consumer;
    }
    
    private static class CapacityAwareRebalanceListener implements ConsumerRebalanceListener {
        private final int capacity;
        
        public CapacityAwareRebalanceListener(int capacity) {
            this.capacity = capacity;
        }
        
        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            log.info("Capacity {} consumer: partitions revoked {}", capacity, partitions);
            // Commit offsets before losing partitions
        }
        
        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            log.info("Capacity {} consumer: assigned {} partitions (capacity: {})", 
                    capacity, partitions.size(), capacity);
            
            // Validate assignment doesn't exceed capacity
            if (partitions.size() > capacity) {
                log.warn("Assignment exceeds capacity: {} > {}", partitions.size(), capacity);
            }
        }
    }
}
Rebalancing Metrics and Monitoring
public class RebalancingMetrics {
    private final MeterRegistry meterRegistry;
    private final AtomicLong rebalanceCount = new AtomicLong(0);
    private final AtomicLong rebalanceDuration = new AtomicLong(0);
    
    public class MetricsRebalanceListener implements ConsumerRebalanceListener {
        private long rebalanceStartTime;
        
        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            rebalanceStartTime = System.currentTimeMillis();
            
            // Record rebalance event
            rebalanceCount.incrementAndGet();
            
            // Record partitions revoked
            Gauge.builder("kafka.consumer.partitions.revoked")
                 .register(meterRegistry, () -> partitions.size());
            
            log.info("Rebalance started - partitions revoked: {}", partitions.size());
        }
        
        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            long duration = System.currentTimeMillis() - rebalanceStartTime;
            rebalanceDuration.set(duration);
            
            // Record rebalance duration
            Timer.Sample sample = Timer.start(meterRegistry);
            sample.stop(Timer.builder("kafka.consumer.rebalance.duration")
                       .register(meterRegistry));
            
            // Record partitions assigned
            Gauge.builder("kafka.consumer.partitions.assigned")
                 .register(meterRegistry, () -> partitions.size());
            
            log.info("Rebalance completed in {}ms - partitions assigned: {}", 
                    duration, partitions.size());
        }
        
        @Override
        public void onPartitionsLost(Collection<TopicPartition> partitions) {
            // Record partition loss event
            Counter.builder("kafka.consumer.partitions.lost")
                   .register(meterRegistry)
                   .increment(partitions.size());
            
            log.error("Partitions lost: {}", partitions);
        }
    }
    
    public void registerRebalanceMetrics() {
        // Register rebalance count gauge
        Gauge.builder("kafka.consumer.rebalance.count")
             .register(meterRegistry, rebalanceCount::get);
        
        // Register rebalance frequency (rebalances per hour)
        Gauge.builder("kafka.consumer.rebalance.frequency")
             .register(meterRegistry, this::calculateRebalanceFrequency);
    }
    
    private double calculateRebalanceFrequency() {
        // Calculate rebalances per hour based on recent history
        // Implementation would track rebalances over time
        return rebalanceCount.get(); // Simplified
    }
}

## Comprehensive Interface Examples

### ConsumerPartitionAssignor Examples

#### 1. Geographic-Based Assignor
```java
/**
 * Custom assignor that assigns partitions based on consumer geographic location
 * to minimize latency by keeping consumers close to their data
 */
public class GeographicPartitionAssignor implements ConsumerPartitionAssignor {
    
    @Override
    public String name() {
        return "geographic-assignor";
    }
    
    @Override
    public List<String> supportedProtocols() {
        return Arrays.asList("range", "roundrobin");
    }
    
    @Override
    public ByteBuffer subscriptionUserData(Set<String> topics) {
        // Encode geographic information in user data
        String region = System.getProperty("consumer.region", "us-east-1");
        String datacenter = System.getProperty("consumer.datacenter", "dc1");
        
        GeographicInfo geoInfo = new GeographicInfo(region, datacenter);
        return serializeGeographicInfo(geoInfo);
    }
    
    @Override
    public GroupAssignment assign(Cluster metadata, GroupSubscription groupSubscription) {
        Map<String, Assignment> assignments = new HashMap<>();
        Map<String, Subscription> subscriptions = groupSubscription.groupSubscription();
        
        // Parse geographic information from all consumers
        Map<String, GeographicInfo> consumerLocations = new HashMap<>();
        for (Map.Entry<String, Subscription> entry : subscriptions.entrySet()) {
            String consumerId = entry.getKey();
            GeographicInfo location = deserializeGeographicInfo(entry.getValue().userData());
            consumerLocations.put(consumerId, location);
        }
        
        // Get all partitions
        List<TopicPartition> allPartitions = getAllPartitions(metadata, subscriptions);
        
        // Group partitions by geographic preference
        Map<String, List<TopicPartition>> partitionsByRegion = groupPartitionsByRegion(allPartitions, metadata);
        
        // Assign partitions to consumers in same region first
        assignments.putAll(assignGeographically(partitionsByRegion, consumerLocations));
        
        return new GroupAssignment(assignments);
    }
    
    private Map<String, Assignment> assignGeographically(
            Map<String, List<TopicPartition>> partitionsByRegion,
            Map<String, GeographicInfo> consumerLocations) {
        
        Map<String, Assignment> assignments = new HashMap<>();
        Map<String, List<String>> consumersByRegion = groupConsumersByRegion(consumerLocations);
        
        // Initialize assignments
        for (String consumer : consumerLocations.keySet()) {
            assignments.put(consumer, new Assignment(new ArrayList<>()));
        }
        
        // Assign partitions region by region
        for (Map.Entry<String, List<TopicPartition>> regionEntry : partitionsByRegion.entrySet()) {
            String region = regionEntry.getKey();
            List<TopicPartition> partitions = regionEntry.getValue();
            List<String> regionConsumers = consumersByRegion.getOrDefault(region, new ArrayList<>());
            
            if (regionConsumers.isEmpty()) {
                // Fall back to any available consumer
                regionConsumers = new ArrayList<>(consumerLocations.keySet());
            }
            
            // Round-robin assignment within region
            for (int i = 0; i < partitions.size(); i++) {
                String consumer = regionConsumers.get(i % regionConsumers.size());
                assignments.get(consumer).partitions().add(partitions.get(i));
            }
        }
        
        return assignments;
    }
    
    @Override
    public void onAssignment(Assignment assignment, ConsumerGroupMetadata metadata) {
        log.info("Geographic assignment for consumer {}: {} partitions in regions: {}",
                metadata.memberId(), 
                assignment.partitions().size(),
                getRegionsForPartitions(assignment.partitions()));
    }
    
    // Helper classes and methods
    private static class GeographicInfo {
        private final String region;
        private final String datacenter;
        
        public GeographicInfo(String region, String datacenter) {
            this.region = region;
            this.datacenter = datacenter;
        }
        
        // getters...
    }
    
    private ByteBuffer serializeGeographicInfo(GeographicInfo info) {
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(info);
            return ByteBuffer.wrap(baos.toByteArray());
        } catch (IOException e) {
            throw new RuntimeException("Failed to serialize geographic info", e);
        }
    }
    
    private GeographicInfo deserializeGeographicInfo(ByteBuffer buffer) {
        if (buffer == null) {
            return new GeographicInfo("unknown", "unknown");
        }
        
        try {
            byte[] bytes = new byte[buffer.remaining()];
            buffer.get(bytes);
            ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bytes));
            return (GeographicInfo) ois.readObject();
        } catch (Exception e) {
            return new GeographicInfo("unknown", "unknown");
        }
    }
}
2. Workload-Aware Assignor
/**
 * Assignor that considers consumer processing capacity and current workload
 * to distribute partitions optimally based on consumer capabilities
 */
public class WorkloadAwareAssignor implements ConsumerPartitionAssignor {
    
    @Override
    public String name() {
        return "workload-aware-assignor";
    }
    
    @Override
    public List<String> supportedProtocols() {
        return Arrays.asList("cooperative-sticky");
    }
    
    @Override
    public ByteBuffer subscriptionUserData(Set<String> topics) {
        ConsumerCapability capability = new ConsumerCapability(
            Runtime.getRuntime().availableProcessors(),  // CPU cores
            Runtime.getRuntime().maxMemory() / (1024 * 1024), // Memory in MB
            getCurrentWorkload(), // Current partition count
            getProcessingRate()   // Messages per second capability
        );
        return serializeCapability(capability);
    }
    
    @Override
    public GroupAssignment assign(Cluster metadata, GroupSubscription groupSubscription) {
        Map<String, Assignment> assignments = new HashMap<>();
        Map<String, Subscription> subscriptions = groupSubscription.groupSubscription();
        
        // Parse consumer capabilities
        Map<String, ConsumerCapability> consumerCapabilities = new HashMap<>();
        for (Map.Entry<String, Subscription> entry : subscriptions.entrySet()) {
            String consumerId = entry.getKey();
            ConsumerCapability capability = deserializeCapability(entry.getValue().userData());
            consumerCapabilities.put(consumerId, capability);
        }
        
        // Get partition workload information
        List<TopicPartition> allPartitions = getAllPartitions(metadata, subscriptions);
        Map<TopicPartition, PartitionWorkload> partitionWorkloads = getPartitionWorkloads(allPartitions);
        
        // Assign based on workload and capability
        assignments.putAll(assignByWorkload(partitionWorkloads, consumerCapabilities));
        
        return new GroupAssignment(assignments);
    }
    
    private Map<String, Assignment> assignByWorkload(
            Map<TopicPartition, PartitionWorkload> partitionWorkloads,
            Map<String, ConsumerCapability> consumerCapabilities) {
        
        Map<String, Assignment> assignments = new HashMap<>();
        Map<String, Double> consumerCurrentLoad = new HashMap<>();
        
        // Initialize assignments and current loads
        for (String consumer : consumerCapabilities.keySet()) {
            assignments.put(consumer, new Assignment(new ArrayList<>()));
            consumerCurrentLoad.put(consumer, 0.0);
        }
        
        // Sort partitions by workload (heaviest first)
        List<Map.Entry<TopicPartition, PartitionWorkload>> sortedPartitions = 
            partitionWorkloads.entrySet().stream()
                .sorted((e1, e2) -> Double.compare(e2.getValue().getWorkloadScore(), 
                                                  e1.getValue().getWorkloadScore()))
                .collect(Collectors.toList());
        
        // Assign each partition to the consumer with the best fit
        for (Map.Entry<TopicPartition, PartitionWorkload> entry : sortedPartitions) {
            TopicPartition partition = entry.getKey();
            PartitionWorkload workload = entry.getValue();
            
            String bestConsumer = findBestConsumerForWorkload(
                workload, consumerCapabilities, consumerCurrentLoad);
            
            assignments.get(bestConsumer).partitions().add(partition);
            consumerCurrentLoad.put(bestConsumer, 
                consumerCurrentLoad.get(bestConsumer) + workload.getWorkloadScore());
        }
        
        return assignments;
    }
    
    private String findBestConsumerForWorkload(
            PartitionWorkload workload,
            Map<String, ConsumerCapability> capabilities,
            Map<String, Double> currentLoads) {
        
        String bestConsumer = null;
        double bestScore = Double.MAX_VALUE;
        
        for (String consumer : capabilities.keySet()) {
            ConsumerCapability capability = capabilities.get(consumer);
            double currentLoad = currentLoads.get(consumer);
            
            // Calculate fit score (lower is better)
            double fitScore = calculateFitScore(workload, capability, currentLoad);
            
            if (fitScore < bestScore) {
                bestScore = fitScore;
                bestConsumer = consumer;
            }
        }
        
        return bestConsumer;
    }
    
    private double calculateFitScore(PartitionWorkload workload, 
                                   ConsumerCapability capability, 
                                   double currentLoad) {
        // Consider CPU utilization
        double cpuScore = (currentLoad + workload.getCpuRequirement()) / capability.getCpuCores();
        
        // Consider memory utilization
        double memoryScore = (currentLoad + workload.getMemoryRequirement()) / capability.getMemoryMB();
        
        // Consider processing rate
        double rateScore = (currentLoad + workload.getProcessingRate()) / capability.getProcessingRate();
        
        // Weighted combination
        return cpuScore * 0.4 + memoryScore * 0.3 + rateScore * 0.3;
    }
    
    @Override
    public void onAssignment(Assignment assignment, ConsumerGroupMetadata metadata) {
        double totalWorkload = assignment.partitions().stream()
            .mapToDouble(partition -> getPartitionWorkload(partition))
            .sum();
            
        log.info("Workload assignment for consumer {}: {} partitions, total workload: {:.2f}",
                metadata.memberId(), assignment.partitions().size(), totalWorkload);
    }
    
    // Helper classes
    private static class ConsumerCapability {
        private final int cpuCores;
        private final long memoryMB;
        private final int currentPartitions;
        private final double processingRate;
        
        public ConsumerCapability(int cpuCores, long memoryMB, int currentPartitions, double processingRate) {
            this.cpuCores = cpuCores;
            this.memoryMB = memoryMB;
            this.currentPartitions = currentPartitions;
            this.processingRate = processingRate;
        }
        
        // getters...
    }
    
    private static class PartitionWorkload {
        private final double workloadScore;
        private final double cpuRequirement;
        private final double memoryRequirement;
        private final double processingRate;
        
        public PartitionWorkload(double workloadScore, double cpuRequirement, 
                               double memoryRequirement, double processingRate) {
            this.workloadScore = workloadScore;
            this.cpuRequirement = cpuRequirement;
            this.memoryRequirement = memoryRequirement;
            this.processingRate = processingRate;
        }
        
        // getters...
    }
}
ConsumerRebalanceListener Examples
1. Stateful Processing Rebalance Listener
/**
 * Rebalance listener for stateful processing applications
 * Handles state transfer and cleanup during rebalancing
 */
public class StatefulProcessingRebalanceListener implements ConsumerRebalanceListener {
    
    private final KafkaConsumer<String, String> consumer;
    private final StateStore stateStore;
    private final OffsetManager offsetManager;
    private final MetricsRegistry metricsRegistry;
    
    public StatefulProcessingRebalanceListener(KafkaConsumer<String, String> consumer,
                                             StateStore stateStore,
                                             OffsetManager offsetManager,
                                             MetricsRegistry metricsRegistry) {
        this.consumer = consumer;
        this.stateStore = stateStore;
        this.offsetManager = offsetManager;
        this.metricsRegistry = metricsRegistry;
    }
    
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        log.info("Partitions being revoked: {}", partitions);
        long startTime = System.currentTimeMillis();
        
        try {
            // 1. Stop processing for these partitions
            stopProcessingForPartitions(partitions);
            
            // 2. Flush any pending state changes
            for (TopicPartition partition : partitions) {
                stateStore.flushPartition(partition);
                log.debug("Flushed state for partition: {}", partition);
            }
            
            // 3. Commit current offsets
            Map<TopicPartition, OffsetAndMetadata> offsetsToCommit = new HashMap<>();
            for (TopicPartition partition : partitions) {
                long currentOffset = offsetManager.getCurrentOffset(partition);
                if (currentOffset >= 0) {
                    offsetsToCommit.put(partition, new OffsetAndMetadata(currentOffset + 1));
                }
            }
            
            if (!offsetsToCommit.isEmpty()) {
                consumer.commitSync(offsetsToCommit);
                log.info("Committed offsets for {} partitions", offsetsToCommit.size());
            }
            
            // 4. Export state for potential transfer
            for (TopicPartition partition : partitions) {
                exportPartitionState(partition);
            }
            
            // 5. Clean up local state
            for (TopicPartition partition : partitions) {
                stateStore.clearPartition(partition);
                offsetManager.clearPartition(partition);
            }
            
            long duration = System.currentTimeMillis() - startTime;
            metricsRegistry.recordRebalanceRevokeDuration(duration);
            log.info("Partition revocation completed in {}ms", duration);
            
        } catch (Exception e) {
            log.error("Error during partition revocation", e);
            metricsRegistry.incrementRebalanceErrors();
            throw new RuntimeException("Failed to handle partition revocation", e);
        }
    }
    
    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        log.info("Partitions being assigned: {}", partitions);
        long startTime = System.currentTimeMillis();
        
        try {
            // 1. Initialize state for new partitions
            for (TopicPartition partition : partitions) {
                initializePartitionState(partition);
            }
            
            // 2. Import any transferred state
            for (TopicPartition partition : partitions) {
                importPartitionState(partition);
            }
            
            // 3. Seek to appropriate offsets
            Map<TopicPartition, Long> partitionOffsets = offsetManager.getStoredOffsets(partitions);
            for (TopicPartition partition : partitions) {
                Long storedOffset = partitionOffsets.get(partition);
                if (storedOffset != null) {
                    consumer.seek(partition, storedOffset);
                    log.debug("Seeking partition {} to offset {}", partition, storedOffset);
                } else {
                    // No stored offset, start from committed offset or beginning
                    OffsetAndMetadata committed = consumer.committed(Collections.singleton(partition)).get(partition);
                    if (committed != null) {
                        consumer.seek(partition, committed.offset());
                    } else {
                        consumer.seekToBeginning(Collections.singleton(partition));
                    }
                }
            }
            
            // 4. Start processing for new partitions
            startProcessingForPartitions(partitions);
            
            long duration = System.currentTimeMillis() - startTime;
            metricsRegistry.recordRebalanceAssignDuration(duration);
            log.info("Partition assignment completed in {}ms", duration);
            
        } catch (Exception e) {
            log.error("Error during partition assignment", e);
            metricsRegistry.incrementRebalanceErrors();
            throw new RuntimeException("Failed to handle partition assignment", e);
        }
    }
    
    @Override
    public void onPartitionsLost(Collection<TopicPartition> partitions) {
        log.warn("Partitions lost due to error: {}", partitions);
        
        try {
            // Emergency cleanup - no graceful state transfer possible
            for (TopicPartition partition : partitions) {
                stateStore.emergencyCleanup(partition);
                offsetManager.clearPartition(partition);
            }
            
            metricsRegistry.incrementPartitionsLost(partitions.size());
            
        } catch (Exception e) {
            log.error("Error during partition loss handling", e);
        }
    }
    
    private void stopProcessingForPartitions(Collection<TopicPartition> partitions) {
        for (TopicPartition partition : partitions) {
            // Signal processing threads to stop for these partitions
            ProcessingCoordinator.stopProcessing(partition);
        }
        
        // Wait for processing to stop gracefully
        ProcessingCoordinator.waitForStop(partitions, Duration.ofSeconds(30));
    }
    
    private void startProcessingForPartitions(Collection<TopicPartition> partitions) {
        for (TopicPartition partition : partitions) {
            ProcessingCoordinator.startProcessing(partition);
        }
    }
    
    private void exportPartitionState(TopicPartition partition) {
        try {
            StateSnapshot snapshot = stateStore.createSnapshot(partition);
            StateTransferService.exportState(partition, snapshot);
            log.debug("Exported state for partition: {}", partition);
        } catch (Exception e) {
            log.warn("Failed to export state for partition: {}", partition, e);
        }
    }
    
    private void importPartitionState(TopicPartition partition) {
        try {
            StateSnapshot snapshot = StateTransferService.importState(partition);
            if (snapshot != null) {
                stateStore.restoreFromSnapshot(partition, snapshot);
                log.debug("Imported state for partition: {}", partition);
            }
        } catch (Exception e) {
            log.warn("Failed to import state for partition: {}", partition, e);
        }
    }
    
    private void initializePartitionState(TopicPartition partition) {
        stateStore.initializePartition(partition);
        offsetManager.initializePartition(partition);
    }
}
2. Database Transaction Rebalance Listener
/**
 * Rebalance listener that handles database transactions during rebalancing
 * Ensures data consistency when consumer state is persisted in database
 */
public class DatabaseTransactionRebalanceListener implements ConsumerRebalanceListener {
    
    private final DataSource dataSource;
    private final KafkaConsumer<String, String> consumer;
    private final TransactionTemplate transactionTemplate;
    private final JdbcTemplate jdbcTemplate;
    
    public DatabaseTransactionRebalanceListener(DataSource dataSource, 
                                              KafkaConsumer<String, String> consumer) {
        this.dataSource = dataSource;
        this.consumer = consumer;
        this.transactionTemplate = new TransactionTemplate(new DataSourceTransactionManager(dataSource));
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }
    
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        log.info("Revoking partitions and committing database transaction: {}", partitions);
        
        // Commit any pending database transactions for these partitions
        transactionTemplate.execute(status -> {
            try {
                // 1. Complete any pending database operations
                for (TopicPartition partition : partitions) {
                    commitPendingOperations(partition);
                }
                
                // 2. Store current offsets in database
                Map<TopicPartition, OffsetAndMetadata> currentOffsets = getCurrentOffsets(partitions);
                storeOffsetsInDatabase(currentOffsets);
                
                // 3. Commit offsets to Kafka
                if (!currentOffsets.isEmpty()) {
                    consumer.commitSync(currentOffsets);
                }
                
                // 4. Mark partitions as inactive in database
                markPartitionsInactive(partitions);
                
                log.info("Successfully committed database transaction for {} partitions", partitions.size());
                return null;
                
            } catch (Exception e) {
                log.error("Error in database transaction during partition revocation", e);
                status.setRollbackOnly();
                throw new RuntimeException("Database transaction failed", e);
            }
        });
    }
    
    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        log.info("Assigning partitions and starting database transaction: {}", partitions);
        
        transactionTemplate.execute(status -> {
            try {
                // 1. Mark partitions as active in database
                markPartitionsActive(partitions);
                
                // 2. Retrieve stored offsets from database
                Map<TopicPartition, Long> storedOffsets = getStoredOffsetsFromDatabase(partitions);
                
                // 3. Seek to stored offsets
                for (TopicPartition partition : partitions) {
                    Long storedOffset = storedOffsets.get(partition);
                    if (storedOffset != null) {
                        consumer.seek(partition, storedOffset);
                        log.debug("Seeking partition {} to stored offset {}", partition, storedOffset);
                    } else {
                        // No stored offset, seek to committed offset
                        OffsetAndMetadata committed = consumer.committed(Collections.singleton(partition)).get(partition);
                        if (committed != null) {
                            consumer.seek(partition, committed.offset());
                        } else {
                            consumer.seekToBeginning(Collections.singleton(partition));
                        }
                    }
                }
                
                // 4. Initialize processing state for new partitions
                initializeProcessingState(partitions);
                
                log.info("Successfully initialized database state for {} partitions", partitions.size());
                return null;
                
            } catch (Exception e) {
                log.error("Error in database transaction during partition assignment", e);
                status.setRollbackOnly();
                throw new RuntimeException("Database initialization failed", e);
            }
        });
    }
    
    @Override
    public void onPartitionsLost(Collection<TopicPartition> partitions) {
        log.error("Partitions lost, performing emergency database cleanup: {}", partitions);
        
        transactionTemplate.execute(status -> {
            try {
                // Emergency cleanup - mark partitions as lost in database
                markPartitionsLost(partitions);
                
                // Clear any pending operations for lost partitions
                clearPendingOperations(partitions);
                
                return null;
            } catch (Exception e) {
                log.error("Error during emergency cleanup", e);
                status.setRollbackOnly();
                throw new RuntimeException("Emergency cleanup failed", e);
            }
        });
    }
    
    private void commitPendingOperations(TopicPartition partition) {
        String sql = "UPDATE pending_operations SET status = 'COMMITTED' " +
                    "WHERE topic = ? AND partition = ? AND status = 'PENDING'";
        
        int updated = jdbcTemplate.update(sql, partition.topic(), partition.partition());
        log.debug("Committed {} pending operations for partition {}", updated, partition);
    }
    
    private Map<TopicPartition, OffsetAndMetadata> getCurrentOffsets(Collection<TopicPartition> partitions) {
        Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
        
        for (TopicPartition partition : partitions) {
            long currentOffset = consumer.position(partition);
            offsets.put(partition, new OffsetAndMetadata(currentOffset));
        }
        
        return offsets;
    }
    
    private void storeOffsetsInDatabase(Map<TopicPartition, OffsetAndMetadata> offsets) {
        String sql = "INSERT INTO consumer_offsets (topic, partition, offset, committed_at) " +
                    "VALUES (?, ?, ?, ?) " +
                    "ON DUPLICATE KEY UPDATE offset = VALUES(offset), committed_at = VALUES(committed_at)";
        
        for (Map.Entry<TopicPartition, OffsetAndMetadata> entry : offsets.entrySet()) {
            TopicPartition partition = entry.getKey();
            long offset = entry.getValue().offset();
            
            jdbcTemplate.update(sql, 
                partition.topic(), 
                partition.partition(), 
                offset, 
                new Timestamp(System.currentTimeMillis()));
        }
    }
    
    private Map<TopicPartition, Long> getStoredOffsetsFromDatabase(Collection<TopicPartition> partitions) {
        Map<TopicPartition, Long> offsets = new HashMap<>();
        
        String sql = "SELECT topic, partition, offset FROM consumer_offsets " +
                    "WHERE topic = ? AND partition = ?";
        
        for (TopicPartition partition : partitions) {
            try {
                Long offset = jdbcTemplate.queryForObject(sql, Long.class, 
                    partition.topic(), partition.partition());
                if (offset != null) {
                    offsets.put(partition, offset);
                }
            } catch (EmptyResultDataAccessException e) {
                // No stored offset for this partition
                log.debug("No stored offset found for partition {}", partition);
            }
        }
        
        return offsets;
    }
    
    private void markPartitionsActive(Collection<TopicPartition> partitions) {
        String sql = "INSERT INTO active_partitions (topic, partition, consumer_id, assigned_at) " +
                    "VALUES (?, ?, ?, ?) " +
                    "ON DUPLICATE KEY UPDATE consumer_id = VALUES(consumer_id), assigned_at = VALUES(assigned_at)";
        
        String consumerId = getConsumerId();
        Timestamp now = new Timestamp(System.currentTimeMillis());
        
        for (TopicPartition partition : partitions) {
            jdbcTemplate.update(sql, 
                partition.topic(), 
                partition.partition(), 
                consumerId, 
                now);
        }
    }
    
    private void markPartitionsInactive(Collection<TopicPartition> partitions) {
        String sql = "DELETE FROM active_partitions WHERE topic = ? AND partition = ?";
        
        for (TopicPartition partition : partitions) {
            jdbcTemplate.update(sql, partition.topic(), partition.partition());
        }
    }
    
    private void markPartitionsLost(Collection<TopicPartition> partitions) {
        String sql = "INSERT INTO lost_partitions (topic, partition, consumer_id, lost_at) " +
                    "VALUES (?, ?, ?, ?)";
        
        String consumerId = getConsumerId();
        Timestamp now = new Timestamp(System.currentTimeMillis());
        
        for (TopicPartition partition : partitions) {
            jdbcTemplate.update(sql, 
                partition.topic(), 
                partition.partition(), 
                consumerId, 
                now);
        }
    }
    
    private void clearPendingOperations(Collection<TopicPartition> partitions) {
        String sql = "DELETE FROM pending_operations WHERE topic = ? AND partition = ?";
        
        for (TopicPartition partition : partitions) {
            jdbcTemplate.update(sql, partition.topic(), partition.partition());
        }
    }
    
    private void initializeProcessingState(Collection<TopicPartition> partitions) {
        // Initialize any processing state needed for new partitions
        for (TopicPartition partition : partitions) {
            ProcessingStateManager.initialize(partition);
        }
    }
    
    private String getConsumerId() {
        // Return unique consumer identifier
        return ManagementFactory.getRuntimeMXBean().getName();
    }
}
3. Monitoring and Alerting Rebalance Listener
/**
 * Rebalance listener that provides comprehensive monitoring and alerting
 * for rebalancing events and performance metrics
 */
public class MonitoringRebalanceListener implements ConsumerRebalanceListener {
    
    private final MeterRegistry meterRegistry;
    private final AlertingService alertingService;
    private final String consumerGroupId;
    private final String consumerId;
    
    private long rebalanceStartTime;
    private final AtomicLong totalRebalances = new AtomicLong(0);
    private final AtomicLong totalRevokedPartitions = new AtomicLong(0);
    private final AtomicLong totalAssignedPartitions = new AtomicLong(0);
    
    public MonitoringRebalanceListener(MeterRegistry meterRegistry,
                                     AlertingService alertingService,
                                     String consumerGroupId,
                                     String consumerId) {
        this.meterRegistry = meterRegistry;
        this.alertingService = alertingService;
        this.consumerGroupId = consumerGroupId;
        this.consumerId = consumerId;
        
        registerMetrics();
    }
    
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        rebalanceStartTime = System.currentTimeMillis();
        long rebalanceCount = totalRebalances.incrementAndGet();
        long revokedCount = totalRevokedPartitions.addAndGet(partitions.size());
        
        log.info("Rebalance #{} started - Revoking {} partitions: {}", 
                rebalanceCount, partitions.size(), partitions);
        
        // Record metrics
        Timer.Sample rebalanceSample = Timer.start(meterRegistry);
        rebalanceSample.stop(Timer.builder("kafka.consumer.rebalance.duration")
                           .tag("group", consumerGroupId)
                           .tag("consumer", consumerId)
                           .tag("phase", "revoke")
                           .register(meterRegistry));
        
        Counter.builder("kafka.consumer.partitions.revoked")
               .tag("group", consumerGroupId)
               .tag("consumer", consumerId)
               .register(meterRegistry)
               .increment(partitions.size());
        
        // Check for frequent rebalancing
        if (isFrequentRebalancing()) {
            alertingService.sendAlert(
                AlertLevel.WARNING,
                "Frequent rebalancing detected",
                String.format("Consumer group %s has had %d rebalances in the last hour", 
                             consumerGroupId, getRebalancesInLastHour())
            );
        }
        
        // Record partition details
        recordPartitionMetrics(partitions, "revoked");
        
        // Custom business logic hooks
        executeCustomRevokeLogic(partitions);
    }
    
    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        long rebalanceDuration = System.currentTimeMillis() - rebalanceStartTime;
        long assignedCount = totalAssignedPartitions.addAndGet(partitions.size());
        
        log.info("Rebalance completed in {}ms - Assigned {} partitions: {}", 
                rebalanceDuration, partitions.size(), partitions);
        
        // Record rebalance duration
        Timer.builder("kafka.consumer.rebalance.total.duration")
             .tag("group", consumerGroupId)
             .tag("consumer", consumerId)
             .register(meterRegistry)
             .record(rebalanceDuration, TimeUnit.MILLISECONDS);
        
        Counter.builder("kafka.consumer.partitions.assigned")
               .tag("group", consumerGroupId)
               .tag("consumer", consumerId)
               .register(meterRegistry)
               .increment(partitions.size());
        
        // Update current partition count gauge
        Gauge.builder("kafka.consumer.partitions.current")
             .tag("group", consumerGroupId)
             .tag("consumer", consumerId)
             .register(meterRegistry, () -> partitions.size());
        
        // Check for rebalance performance issues
        if (rebalanceDuration > getRebalanceThreshold()) {
            alertingService.sendAlert(
                AlertLevel.WARNING,
                "Slow rebalance detected",
                String.format("Rebalance took %dms for consumer group %s (threshold: %dms)", 
                             rebalanceDuration, consumerGroupId, getRebalanceThreshold())
            );
        }
        
        // Record partition assignment distribution
        recordAssignmentDistribution(partitions);
        
        // Custom business logic hooks
        executeCustomAssignLogic(partitions);
        
        // Send success notification for critical consumer groups
        if (isCriticalConsumerGroup()) {
            alertingService.sendAlert(
                AlertLevel.INFO,
                "Critical consumer group rebalanced",
                String.format("Consumer group %s successfully rebalanced with %d partitions", 
                             consumerGroupId, partitions.size())
            );
        }
    }
    
    @Override
    public void onPartitionsLost(Collection<TopicPartition> partitions) {
        log.error("Partitions lost: {}", partitions);
        
        // Record partition loss metrics
        Counter.builder("kafka.consumer.partitions.lost")
               .tag("group", consumerGroupId)
               .tag("consumer", consumerId)
               .register(meterRegistry)
               .increment(partitions.size());
        
        // Send critical alert
        alertingService.sendAlert(
            AlertLevel.CRITICAL,
            "Partitions lost",
            String.format("Consumer %s in group %s lost %d partitions: %s", 
                         consumerId, consumerGroupId, partitions.size(), partitions)
        );
        
        // Custom business logic for partition loss
        executeCustomLostLogic(partitions);
    }
    
    private void registerMetrics() {
        // Register gauges for totals
        Gauge.builder("kafka.consumer.rebalance.total.count")
             .tag("group", consumerGroupId)
             .tag("consumer", consumerId)
             .register(meterRegistry, totalRebalances::get);
        
        Gauge.builder("kafka.consumer.partitions.total.revoked")
             .tag("group", consumerGroupId)
             .tag("consumer", consumerId)
             .register(meterRegistry, totalRevokedPartitions::get);
        
        Gauge.builder("kafka.consumer.partitions.total.assigned")
             .tag("group", consumerGroupId)
             .tag("consumer", consumerId)
             .register(meterRegistry, totalAssignedPartitions::get);
        
        // Register rebalance frequency gauge
        Gauge.builder("kafka.consumer.rebalance.frequency")
             .tag("group", consumerGroupId)
             .tag("consumer", consumerId)
             .register(meterRegistry, this::calculateRebalanceFrequency);
    }
    
    private void recordPartitionMetrics(Collection<TopicPartition> partitions, String action) {
        // Record per-topic partition counts
        Map<String, Long> partitionsByTopic = partitions.stream()
            .collect(Collectors.groupingBy(TopicPartition::topic, Collectors.counting()));
        
        for (Map.Entry<String, Long> entry : partitionsByTopic.entrySet()) {
            Counter.builder("kafka.consumer.partitions.by.topic")
                   .tag("group", consumerGroupId)
                   .tag("consumer", consumerId)
                   .tag("topic", entry.getKey())
                   .tag("action", action)
                   .register(meterRegistry)
                   .increment(entry.getValue());
        }
    }
    
    private void recordAssignmentDistribution(Collection<TopicPartition> partitions) {
        // Record partition distribution statistics
        Map<String, List<TopicPartition>> partitionsByTopic = partitions.stream()
            .collect(Collectors.groupingBy(TopicPartition::topic));
        
        for (Map.Entry<String, List<TopicPartition>> entry : partitionsByTopic.entrySet()) {
            String topic = entry.getKey();
            int count = entry.getValue().size();
            
            Gauge.builder("kafka.consumer.partitions.per.topic")
                 .tag("group", consumerGroupId)
                 .tag("consumer", consumerId)
                 .tag("topic", topic)
                 .register(meterRegistry, () -> count);
        }
    }
    
    private boolean isFrequentRebalancing() {
        return getRebalancesInLastHour() > 10; // Threshold: 10 rebalances per hour
    }
    
    private int getRebalancesInLastHour() {
        // Implementation would track rebalances over time
        // Simplified for example
        return (int) (totalRebalances.get() % 100);
    }
    
    private long getRebalanceThreshold() {
        return 30000; // 30 seconds threshold
    }
    
    private boolean isCriticalConsumerGroup() {
        return consumerGroupId.contains("critical") || consumerGroupId.contains("prod");
    }
    
    private double calculateRebalanceFrequency() {
        // Calculate rebalances per hour
        // Implementation would use time-based sliding window
        return totalRebalances.get() / 24.0; // Simplified: rebalances per day / 24
    }
    
    private void executeCustomRevokeLogic(Collection<TopicPartition> partitions) {
        // Hook for custom business logic during partition revocation
        // Example: Pause data ingestion, flush caches, etc.
    }
    
    private void executeCustomAssignLogic(Collection<TopicPartition> partitions) {
        // Hook for custom business logic during partition assignment
        // Example: Initialize resources, warm up caches, etc.
    }
    
    private void executeCustomLostLogic(Collection<TopicPartition> partitions) {
        // Hook for custom business logic during partition loss
        // Example: Mark data as potentially inconsistent, trigger recovery, etc.
    }
}

## Consumer Group Eviction and Lifecycle Management

### Why Consumers Get Kicked Out of Groups

#### 1. Heartbeat Failures
```java
public class HeartbeatFailureScenarios {
    
    // Scenario 1: Network Issues
    public void networkPartitionScenario() {
        /*
         * When: Consumer loses network connectivity to coordinator
         * Cause: Network partition, firewall changes, DNS issues
         * Timeline:
         * - Consumer misses heartbeat window (session.timeout.ms)
         * - Coordinator marks consumer as failed
         * - Triggers rebalance to redistribute partitions
         */
        
        Properties props = new Properties();
        props.put("session.timeout.ms", "30000");      // 30 seconds to detect failure
        props.put("heartbeat.interval.ms", "3000");    // Send heartbeat every 3 seconds
        
        // Monitor network connectivity
        scheduleNetworkCheck();
    }
    
    // Scenario 2: GC Pauses
    public void gcPauseScenario() {
        /*
         * When: Long GC pauses prevent heartbeat sending
         * Cause: Large heap, inefficient GC configuration, memory pressure
         * Solution: Tune GC and increase session timeout
         */
        
        // JVM tuning for reduced GC impact
        String jvmArgs = """
            -XX:+UseG1GC
            -XX:MaxGCPauseMillis=200
            -XX:G1HeapRegionSize=16m
            -XX:+UseStringDeduplication
            -Xmx8g -Xms8g
            """;
        
        // Increase timeouts to accommodate GC pauses
        Properties props = new Properties();
        props.put("session.timeout.ms", "45000");      // Increased tolerance
        props.put("heartbeat.interval.ms", "15000");   // Less frequent heartbeats
    }
    
    // Scenario 3: Thread Blocking
    public void threadBlockingScenario() {
        /*
         * When: Consumer thread blocked and cannot send heartbeats
         * Cause: Synchronous I/O, lock contention, resource waiting
         * Solution: Asynchronous processing, proper thread management
         */
        
        // Bad: Blocking operation in consumer thread
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                // This blocks the consumer thread
                synchronousHttpCall(record.value()); // BAD!
            }
        }
        
        // Good: Asynchronous processing
        ExecutorService executor = Executors.newFixedThreadPool(10);
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                executor.submit(() -> processRecord(record)); // GOOD!
            }
        }
    }
}
2. Processing Timeout Violations
public class ProcessingTimeoutScenarios {
    
    // Scenario 1: max.poll.interval.ms exceeded
    public void pollIntervalExceeded() {
        /*
         * When: Time between poll() calls exceeds max.poll.interval.ms
         * Default: 5 minutes (300,000 ms)
         * Cause: Slow message processing, large batch sizes
         */
        
        Properties props = new Properties();
        props.put("max.poll.interval.ms", "300000");   // 5 minutes default
        props.put("max.poll.records", "500");          // Batch size
        
        long startTime = System.currentTimeMillis();
        
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            
            long processingStart = System.currentTimeMillis();
            
            // Process records
            for (ConsumerRecord<String, String> record : records) {
                processRecord(record); // If this takes too long...
            }
            
            long processingTime = System.currentTimeMillis() - processingStart;
            long timeSinceLastPoll = System.currentTimeMillis() - startTime;
            
            // Monitor processing time
            if (processingTime > 240000) { // 4 minutes warning
                log.warn("Processing time approaching max.poll.interval: {}ms", processingTime);
            }
            
            if (timeSinceLastPoll > 250000) { // 4 minutes 10 seconds
                log.error("Approaching max.poll.interval.ms limit!");
            }
            
            startTime = System.currentTimeMillis();
        }
    }
    
    // Solution: Chunked Processing
    public void chunkedProcessingPattern() {
        Properties props = new Properties();
        props.put("max.poll.interval.ms", "600000");   // Increased to 10 minutes
        props.put("max.poll.records", "100");          // Smaller batches
        
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            
            // Process in chunks to avoid timeout
            List<ConsumerRecord<String, String>> recordList = new ArrayList<>();
            records.forEach(recordList::add);
            
            int chunkSize = 50;
            for (int i = 0; i < recordList.size(); i += chunkSize) {
                int endIndex = Math.min(i + chunkSize, recordList.size());
                List<ConsumerRecord<String, String>> chunk = recordList.subList(i, endIndex);
                
                processChunk(chunk);
                
                // Check if we're approaching timeout
                if (shouldCommitEarly()) {
                    consumer.commitSync();
                    break; // Exit to poll again
                }
            }
        }
    }
}
3. Consumer Group Protocol Violations
public class ProtocolViolationScenarios {
    
    // Scenario 1: Multiple consumers with same client.id
    public void duplicateClientIdScenario() {
        /*
         * When: Multiple consumer instances use same client.id
         * Cause: Configuration error, deployment issues
         * Result: Coordinator cannot distinguish consumers
         */
        
        // BAD: Same client.id for multiple instances
        Properties props1 = new Properties();
        props1.put("client.id", "my-consumer"); // Same ID
        
        Properties props2 = new Properties();
        props2.put("client.id", "my-consumer"); // Same ID - CONFLICT!
        
        // GOOD: Unique client.id for each instance
        Properties goodProps1 = new Properties();
        goodProps1.put("client.id", "my-consumer-instance-1");
        
        Properties goodProps2 = new Properties();
        goodProps2.put("client.id", "my-consumer-instance-2");
    }
    
    // Scenario 2: Incompatible protocol versions
    public void protocolVersionMismatch() {
        /*
         * When: Consumer uses incompatible protocol version
         * Cause: Kafka version mismatch, deprecated assignors
         * Solution: Use compatible assignors and versions
         */
        
        // Check and use compatible assignors
        Properties props = new Properties();
        props.put("partition.assignment.strategy", 
                 "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
        
        // Avoid deprecated assignors
        // props.put("partition.assignment.strategy", "org.apache.kafka.clients.consumer.RangeAssignor"); // Old
    }
    
    // Scenario 3: Coordinator election issues
    public void coordinatorElectionProblems() {
        /*
         * When: Group coordinator broker fails during election
         * Cause: Broker failures, network partitions
         * Result: Temporary group unavailability
         */
        
        // Monitor coordinator health
        AdminClient adminClient = AdminClient.create(getAdminProps());
        
        try {
            DescribeConsumerGroupsResult result = adminClient.describeConsumerGroups(
                Arrays.asList("my-consumer-group"));
            
            ConsumerGroupDescription description = result.all().get().get("my-consumer-group");
            Node coordinator = description.coordinator();
            
            log.info("Current coordinator: {}", coordinator);
            
            // Check coordinator connectivity
            testCoordinatorConnectivity(coordinator);
            
        } catch (Exception e) {
            log.error("Coordinator election issue detected", e);
            handleCoordinatorFailure();
        }
    }
}
How to Minimize Rebalancing
1. Configuration Optimization
public class RebalancingMinimizationStrategies {
    
    // Strategy 1: Optimal Timeout Configuration
    public Properties getOptimalTimeoutConfig() {
        Properties props = new Properties();
        
        // Session management (prevents unnecessary member removal)
        props.put("session.timeout.ms", "45000");      // 45 seconds (conservative)
        props.put("heartbeat.interval.ms", "15000");   // 15 seconds (1/3 of session timeout)
        props.put("max.poll.interval.ms", "600000");   // 10 minutes (generous for processing)
        
        // Rebalancing optimization
        props.put("rebalance.timeout.ms", "300000");   // 5 minutes for rebalance completion
        
        // Use cooperative rebalancing
        props.put("partition.assignment.strategy", 
                 "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
        
        // Reduce metadata refresh frequency
        props.put("metadata.max.age.ms", "600000");    // 10 minutes
        
        return props;
    }
    
    // Strategy 2: Graceful Shutdown Pattern
    public void implementGracefulShutdown() {
        final AtomicBoolean shutdownRequested = new AtomicBoolean(false);
        
        // Register shutdown hook
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            log.info("Shutdown requested, initiating graceful shutdown");
            shutdownRequested.set(true);
        }));
        
        try {
            while (!shutdownRequested.get()) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                
                // Process records
                processRecords(records);
                
                // Commit periodically
                if (shouldCommit()) {
                    consumer.commitSync();
                }
            }
        } finally {
            try {
                // Graceful cleanup - sends LeaveGroup request
                consumer.close(Duration.ofSeconds(30));
                log.info("Consumer closed gracefully");
            } catch (Exception e) {
                log.error("Error during graceful shutdown", e);
            }
        }
    }
}
Complete Consumer Lifecycle and Rebalancing Process
1. Consumer Startup Lifecycle
public class ConsumerLifecycleManager {
    
    // Phase 1: Consumer Initialization
    public void phase1_ConsumerInitialization() {
        /*
         * Step 1: Create Consumer Instance
         * - Load configuration
         * - Initialize serializers/deserializers
         * - Set up metrics and monitoring
         */
        
        Properties props = loadConfiguration();
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        
        log.info("Consumer created with client.id: {}", props.get("client.id"));
        
        /*
         * Step 2: Subscribe to Topics
         * - Define topic subscription
         * - Register rebalance listener
         * - No network communication yet
         */
        
        consumer.subscribe(
            Arrays.asList("my-topic"),
            new DetailedRebalanceListener()
        );
        
        log.info("Subscribed to topics, no group coordination yet");
    }
    
    // Phase 2: Group Coordination Discovery
    public void phase2_GroupCoordinatorDiscovery() {
        /*
         * Step 1: First poll() call triggers group coordination
         * - Consumer sends FindCoordinator request
         * - Discovers which broker is the group coordinator
         * - Coordinator determined by: hash(group.id) % number_of_brokers
         */
        
        log.info("First poll() - discovering group coordinator");
        ConsumerRecords<String, String> firstPoll = consumer.poll(Duration.ofMillis(100));
        
        /*
         * Step 2: JoinGroup Protocol
         * - Consumer sends JoinGroup request to coordinator
         * - Includes: group.id, member.id, protocol type, subscription info
         * - Coordinator collects all group members
         */
        
        /*
         * Internal Process:
         * 1. Coordinator receives JoinGroup request
         * 2. If first member or rebalance needed:
         *    - Coordinator waits for session.timeout.ms for all members
         *    - Moves group to PreparingRebalance state
         * 3. Coordinator selects group leader (usually first to join)
         * 4. Sends JoinGroup response with member list to leader
         * 5. Sends JoinGroup response with assignment to other members
         */
    }
    
    // Phase 3: Partition Assignment
    public void phase3_PartitionAssignment() {
        /*
         * Step 1: Leader Assignment Calculation
         * - Group leader receives all member subscriptions
         * - Runs partition assignment strategy
         * - Creates assignment for all group members
         */
        
        /*
         * Step 2: SyncGroup Protocol
         * - Leader sends SyncGroup request with assignments
         * - Other members send empty SyncGroup request
         * - Coordinator distributes assignments to all members
         */
        
        /*
         * Step 3: Assignment Acknowledgment
         * - All members receive their partition assignments
         * - Group moves to Stable state
         * - Consumer ready to start processing
         */
        
        log.info("Consumer received partition assignment and is ready to process");
    }
    
    // Phase 4: Normal Processing State
    public void phase4_NormalProcessing() {
        /*
         * Ongoing Activities:
         * 1. Heartbeat Thread: Sends heartbeats every heartbeat.interval.ms
         * 2. Main Thread: Polls for messages and processes them
         * 3. Offset Management: Commits offsets periodically
         * 4. Metadata Refresh: Updates topic/partition metadata
         */
        
        // Heartbeat monitoring
        scheduleHeartbeatMonitoring();
        
        // Main processing loop
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            
            // Process records
            processRecords(records);
            
            // Commit offsets
            if (shouldCommit()) {
                consumer.commitAsync();
            }
            
            // Monitor processing health
            monitorProcessingHealth();
        }
    }
    
    private class DetailedRebalanceListener implements ConsumerRebalanceListener {
        
        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            log.info("=== REBALANCE PHASE 1: Partitions Revoked ===");
            log.info("Revoked partitions: {}", partitions);
            
            /*
             * What happens during revocation:
             * 1. Consumer stops fetching from revoked partitions
             * 2. Application should commit current offsets
             * 3. Clean up any partition-specific state
             * 4. Prepare for new assignment
             */
            
            // Commit current offsets
            try {
                Map<TopicPartition, OffsetAndMetadata> currentOffsets = getCurrentOffsets(partitions);
                consumer.commitSync(currentOffsets);
                log.info("Committed offsets for revoked partitions");
            } catch (Exception e) {
                log.error("Failed to commit offsets during revocation", e);
            }
            
            // Clean up partition-specific resources
            cleanupPartitionResources(partitions);
        }
        
        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            log.info("=== REBALANCE PHASE 2: Partitions Assigned ===");
            log.info("Assigned partitions: {}", partitions);
            
            /*
             * What happens during assignment:
             * 1. Consumer receives new partition assignment
             * 2. Initializes state for new partitions
             * 3. Seeks to appropriate offsets
             * 4. Resumes processing
             */
            
            // Initialize resources for new partitions
            initializePartitionResources(partitions);
            
            // Seek to appropriate offsets
            for (TopicPartition partition : partitions) {
                long storedOffset = getStoredOffset(partition);
                if (storedOffset >= 0) {
                    consumer.seek(partition, storedOffset);
                    log.debug("Seeking partition {} to stored offset {}", partition, storedOffset);
                } else {
                    // No stored offset, use committed offset or start from beginning
                    OffsetAndMetadata committed = consumer.committed(Collections.singleton(partition)).get(partition);
                    if (committed != null) {
                        consumer.seek(partition, committed.offset());
                    } else {
                        consumer.seekToBeginning(Collections.singleton(partition));
                    }
                }
            }
            
            log.info("Consumer ready to process {} partitions", partitions.size());
        }
        
        @Override
        public void onPartitionsLost(Collection<TopicPartition> partitions) {
            log.error("=== REBALANCE ERROR: Partitions Lost ===");
            log.error("Lost partitions: {}", partitions);
            
            /*
             * Partition loss scenarios:
             * 1. Consumer took too long to rejoin after revocation
             * 2. Network issues during rebalancing
             * 3. Coordinator failures
             * 
             * Recovery: Clean up state and wait for new assignment
             */
            
            // Emergency cleanup
            emergencyCleanup(partitions);
        }
    }
}
2. Detailed Rebalancing Process Timeline
public class RebalancingProcessTimeline {
    
    public void completeRebalancingWalkthrough() {
        /*
         * TIMELINE: Consumer Group Rebalancing Process
         * 
         * T0: Trigger Event (new consumer joins, existing leaves, or failure detected)
         * T1: Coordinator initiates rebalance
         * T2: Group state changes to PreparingRebalance
         * T3: Existing consumers receive rebalance signal
         * T4: Consumers stop processing and join rebalance
         * T5: Leader selection and assignment calculation
         * T6: Assignment distribution to all members
         * T7: Group state changes to Stable
         * T8: Consumers resume processing with new assignments
         */
        
        demonstrateRebalanceSteps();
    }
    
    private void demonstrateRebalanceSteps() {
        log.info("=== T0: TRIGGER EVENT ===");
        // New consumer joins group
        triggerRebalance();
        
        log.info("=== T1-T2: COORDINATOR INITIATES REBALANCE ===");
        // Coordinator detects trigger and changes group state
        coordinatorInitiatesRebalance();
        
        log.info("=== T3: REBALANCE SIGNAL SENT ===");
        // Existing consumers receive rebalance signal in heartbeat response
        sendRebalanceSignal();
        
        log.info("=== T4: CONSUMERS JOIN REBALANCE ===");
        // All consumers stop processing and send JoinGroup requests
        consumersJoinRebalance();
        
        log.info("=== T5: LEADER SELECTION AND ASSIGNMENT ===");
        // Coordinator selects leader and leader calculates assignments
        leaderCalculatesAssignments();
        
        log.info("=== T6: ASSIGNMENT DISTRIBUTION ===");
        // Coordinator distributes assignments to all members
        distributeAssignments();
        
        log.info("=== T7-T8: RESUME PROCESSING ===");
        // Group becomes stable and consumers resume processing
        resumeProcessing();
    }
    
    // Detailed implementation of each phase
    private void triggerRebalance() {
        /*
         * Rebalance Triggers:
         * 1. Consumer joins: First poll() call triggers JoinGroup
         * 2. Consumer leaves: close() sends LeaveGroup request
         * 3. Consumer fails: Missed heartbeats trigger timeout
         * 4. Topic changes: Partition count increases
         * 5. Subscription changes: Different topic subscription
         */
        
        log.info("Rebalance triggered by: Consumer joining group");
    }
    
    private void coordinatorInitiatesRebalance() {
        /*
         * Coordinator Actions:
         * 1. Changes group state to PreparingRebalance
         * 2. Sets rebalance timeout timer
         * 3. Prepares to collect JoinGroup requests
         * 4. Increment generation ID for this rebalance
         */
        
        log.info("Group coordinator initiating rebalance");
        log.info("Group state: Empty -> PreparingRebalance");
    }
    
    private void sendRebalanceSignal() {
        /*
         * Signal Distribution:
         * 1. Existing consumers receive REBALANCE_IN_PROGRESS in heartbeat response
         * 2. Consumers stop fetching new messages
         * 3. Consumers prepare to send JoinGroup requests
         */
        
        log.info("Heartbeat responses include REBALANCE_IN_PROGRESS signal");
    }
    
    private void consumersJoinRebalance() {
        /*
         * Consumer Actions:
         * 1. Stop processing current messages
         * 2. Commit current offsets (in onPartitionsRevoked)
         * 3. Send JoinGroup request with subscription info
         * 4. Wait for JoinGroup response
         */
        
        log.info("All consumers sending JoinGroup requests");
        log.info("Consumers waiting for assignment...");
    }
    
    private void leaderCalculatesAssignments() {
        /*
         * Leader Selection:
         * 1. Coordinator selects first consumer to join as leader
         * 2. Leader receives member list and subscription info
         * 3. Leader runs partition assignment strategy
         * 4. Leader creates assignment map for all members
         */
        
        log.info("Group leader selected and calculating assignments");
        log.info("Running partition assignment strategy");
    }
    
    private void distributeAssignments() {
        /*
         * Assignment Distribution:
         * 1. Leader sends SyncGroup with assignment data
         * 2. Other members send empty SyncGroup requests
         * 3. Coordinator distributes assignments to all members
         * 4. All members receive their partition assignments
         */
        
        log.info("Coordinator distributing assignments to all members");
        log.info("Group state: PreparingRebalance -> Stable");
    }
    
    private void resumeProcessing() {
        /*
         * Resume Processing:
         * 1. Consumers receive assignments (onPartitionsAssigned)
         * 2. Consumers seek to appropriate offsets
         * 3. Consumers resume polling and processing
         * 4. Heartbeat thread resumes normal operation
         */
        
        log.info("All consumers resuming processing with new assignments");
        log.info("Rebalance complete - group is stable");
    }
}
Transaction Handling
Overview of Kafka Transactions
Kafka transactions provide atomicity guarantees across multiple partitions and topics, enabling exactly-once processing semantics for stream processing applications.

Transaction Components
Transaction Coordinator
├── Transaction Log Topic (__transaction_state)
├── Producer ID Management
├── Transaction State Management
└── Transaction Timeout Handling

Producer Instance
├── Transactional ID (unique across restarts)
├── Producer ID (assigned by coordinator)
├── Epoch (prevents zombie producers)
└── Transaction State (ONGOING, PREPARE_COMMIT, etc.)
Transactional Producer
Basic Transactional Producer Setup
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

// Essential transactional properties
props.put("transactional.id", "my-transactional-producer-1");
props.put("enable.idempotence", true);
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("max.in.flight.requests.per.connection", 5);

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// Initialize transactions
producer.initTransactions();
Transaction Lifecycle
public class TransactionalService {
    private final KafkaProducer<String, String> producer;
    
    public void processOrderWithTransaction(Order order) {
        try {
            // Begin transaction
            producer.beginTransaction();
            
            // Send multiple messages atomically
            producer.send(new ProducerRecord<>("order-events", order.getId(), 
                objectMapper.writeValueAsString(new OrderCreatedEvent(order))));
            
            producer.send(new ProducerRecord<>("inventory-events", order.getId(),
                objectMapper.writeValueAsString(new InventoryReservedEvent(order))));
            
            producer.send(new ProducerRecord<>("payment-events", order.getId(),
                objectMapper.writeValueAsString(new PaymentProcessedEvent(order))));
            
            // Commit transaction
            producer.commitTransaction();
            
        } catch (Exception e) {
            // Abort transaction on any error
            producer.abortTransaction();
            throw new TransactionException("Failed to process order", e);
        }
    }
}
Acknowledgments and Delivery Guarantees
Producer Acknowledgment Levels
acks=0 (Fire and Forget)
Properties props = new Properties();
props.put("acks", "0");
// Producer doesn't wait for any acknowledgment
// Highest throughput, no durability guarantee
// Messages can be lost if broker fails

// Use case: Metrics, logs where some data loss is acceptable
Characteristics:

Latency: Lowest (no waiting)
Throughput: Highest
Durability: None (messages can be lost)
Use Cases: High-volume metrics, non-critical logging
acks=1 (Leader Acknowledgment)
Properties props = new Properties();
props.put("acks", "1");
// Producer waits for leader replica acknowledgment only
// Balanced throughput and some durability
// Messages can be lost if leader fails before replication

// Use case: Most common setting for general applications
Characteristics:

Latency: Medium
Throughput: Good
Durability: Moderate (can lose messages if leader fails)
Use Cases: General application messages, events
acks=all/-1 (Full ISR Acknowledgment)
Properties props = new Properties();
props.put("acks", "all"); // or "-1"
props.put("min.insync.replicas", "2"); // Minimum replicas that must acknowledge
// Producer waits for all in-sync replicas to acknowledge
// Strongest durability guarantee
// Lower throughput due to waiting for all replicas

// Use case: Critical data that cannot be lost
Characteristics:

Latency: Highest (waits for all ISR)
Throughput: Lower
Durability: Strongest (no data loss if min.insync.replicas maintained)
Use Cases: Financial transactions, critical business events
Exactly-Once Semantics
Understanding Exactly-Once Semantics (EOS)
What is Exactly-Once?
Exactly-once semantics ensures that each message is processed exactly one time, even in the presence of failures, retries, or rebalancing.

Without EOS:
Producer → [Retry] → Broker → Consumer → [Duplicate Processing]

With EOS:
Producer → [Idempotent] → Broker → [Transactional] → Consumer → [Exactly-Once Processing]
EOS Components
Idempotent Producer: Prevents duplicate messages on retry
Transactions: Atomic writes across partitions
Transactional Consumer: Reads only committed messages
Offset Management: Transactional offset commits
Idempotency
Producer Idempotency
Enabling Idempotent Producer
Properties props = new Properties();
props.put("enable.idempotence", "true");
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("max.in.flight.requests.per.connection", 5);

// Idempotent producer prevents duplicate messages during retries
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
How Idempotency Works
Producer Request includes:
├── Producer ID (PID) - assigned by broker
├── Sequence Number - per partition sequence
└── Producer Epoch - prevents zombie producers

Broker maintains:
├── Last sequence number per PID per partition
└── Duplicate detection logic
Consumer Lag Troubleshooting
Understanding Consumer Lag
What is Consumer Lag?
Consumer lag is the difference between the latest offset in a partition and the consumer's current offset.

# Check consumer lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group my-group --describe

# Output shows:
# TOPIC    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# events   0          1000           1500            500
# events   1          800            1200            400
Lag Analysis Tools
Advanced Lag Monitoring Script
#!/bin/bash
# advanced-lag-monitor.sh

BOOTSTRAP_SERVERS="localhost:9092"
THRESHOLD=1000
ALERT_EMAIL="admin@company.com"

monitor_consumer_lag() {
    local group=$1
    
    kafka-consumer-groups.sh --bootstrap-server $BOOTSTRAP_SERVERS \
        --group $group --describe --offsets | \
    awk -v threshold=$THRESHOLD -v group=$group '
    NR > 1 && $6 != "-" {
        lag = $6
        if (lag > threshold) {
            print "ALERT: Group " group " partition " $2 " lag: " lag
            alert_count++
        }
        total_lag += lag
    }
    END {
        print "Total lag for group " group ": " total_lag
        if (alert_count > 0) {
            system("echo \"High lag detected for group " group "\" | mail -s \"Kafka Lag Alert\" " ALERT_EMAIL)
        }
    }'
}

# Monitor all consumer groups
for group in $(kafka-consumer-groups.sh --bootstrap-server $BOOTSTRAP_SERVERS --list); do
    echo "=== Monitoring group: $group ==="
    monitor_consumer_lag $group
    echo ""
done
Consumer Lag Metrics Collection
@Component
public class ConsumerLagMonitor {
    private final AdminClient adminClient;
    private final MeterRegistry meterRegistry;
    
    @Scheduled(fixedRate = 30000) // Every 30 seconds
    public void collectLagMetrics() {
        try {
            Map<String, ConsumerGroupDescription> groups = adminClient
                .describeConsumerGroups(getConsumerGroups())
                .all()
                .get();
            
            for (String groupId : groups.keySet()) {
                Map<TopicPartition, OffsetAndMetadata> offsets = adminClient
                    .listConsumerGroupOffsets(groupId)
                    .partitionsToOffsetAndMetadata()
                    .get();
                
                Map<TopicPartition, Long> endOffsets = getEndOffsets(offsets.keySet());
                
                for (Map.Entry<TopicPartition, OffsetAndMetadata> entry : offsets.entrySet()) {
                    TopicPartition tp = entry.getKey();
                    long currentOffset = entry.getValue().offset();
                    long endOffset = endOffsets.get(tp);
                    long lag = endOffset - currentOffset;
                    
                    // Record metrics
                    Gauge.builder("kafka.consumer.lag")
                        .tag("group", groupId)
                        .tag("topic", tp.topic())
                        .tag("partition", String.valueOf(tp.partition()))
                        .register(meterRegistry, () -> lag);
                }
            }
        } catch (Exception e) {
            log.error("Failed to collect consumer lag metrics", e);
        }
    }
}
Lag Troubleshooting Strategies
1. Identify Root Causes
public class LagDiagnostics {
    
    public void diagnoseLag(String consumerGroup) {
        // Check 1: Consumer processing speed
        measureProcessingSpeed(consumerGroup);
        
        // Check 2: Producer rate vs Consumer rate
        compareProducerConsumerRates();
        
        // Check 3: Consumer configuration issues
        validateConsumerConfiguration(consumerGroup);
        
        // Check 4: Rebalancing frequency
        checkRebalancingFrequency(consumerGroup);
        
        // Check 5: Resource constraints
        checkResourceUtilization(consumerGroup);
    }
    
    private void measureProcessingSpeed(String consumerGroup) {
        // Measure time between polls and processing
        long startTime = System.currentTimeMillis();
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
        long pollTime = System.currentTimeMillis() - startTime;
        
        startTime = System.currentTimeMillis();
        for (ConsumerRecord<String, String> record : records) {
            processRecord(record);
        }
        long processingTime = System.currentTimeMillis() - startTime;
        
        log.info("Group: {} - Poll time: {}ms, Processing time: {}ms, Records: {}", 
                consumerGroup, pollTime, processingTime, records.count());
        
        if (processingTime > pollTime * 10) {
            log.warn("Slow processing detected - consider optimization or scaling");
        }
    }
}
2. Scaling Solutions
public class LagResolutionStrategies {
    
    // Strategy 1: Increase consumer instances
    public void scaleConsumers(String consumerGroup, int targetInstances) {
        int currentPartitions = getPartitionCount();
        int currentConsumers = getCurrentConsumerCount(consumerGroup);
        
        if (currentConsumers < currentPartitions && currentConsumers < targetInstances) {
            log.info("Scaling consumers from {} to {}", currentConsumers, 
                    Math.min(targetInstances, currentPartitions));
            // Trigger consumer scaling (e.g., via Kubernetes)
            scaleConsumerDeployment(consumerGroup, Math.min(targetInstances, currentPartitions));
        }
    }
    
    // Strategy 2: Optimize consumer configuration
    public void optimizeConsumerConfig(String consumerGroup) {
        Properties optimizedProps = new Properties();
        
        // Increase fetch size for better throughput
        optimizedProps.put("fetch.min.bytes", "50000");
        optimizedProps.put("max.partition.fetch.bytes", "2097152");
        optimizedProps.put("max.poll.records", "1000");
        
        // Reduce session timeout if processing is fast
        optimizedProps.put("session.timeout.ms", "10000");
        optimizedProps.put("heartbeat.interval.ms", "3000");
        
        // Enable auto-commit for better performance (if acceptable)
        optimizedProps.put("enable.auto.commit", "true");
        optimizedProps.put("auto.commit.interval.ms", "1000");
        
        log.info("Applied optimized configuration for group: {}", consumerGroup);
    }
    
    // Strategy 3: Parallel processing within consumer
    public void enableParallelProcessing() {
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            
            if (!records.isEmpty()) {
                List<CompletableFuture<Void>> futures = new ArrayList<>();
                
                for (ConsumerRecord<String, String> record : records) {
                    CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
                        processRecord(record);
                    }, executor);
                    futures.add(future);
                }
                
                // Wait for all records in batch to complete
                CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
                
                // Commit after all processing is done
                consumer.commitSync();
            }
        }
    }
}
Emergency Lag Recovery
Fast-Forward Strategy
public class EmergencyLagRecovery {
    
    public void fastForwardConsumer(String consumerGroup, String topic) {
        log.warn("Executing emergency fast-forward for group: {} topic: {}", 
                consumerGroup, topic);
        
        // Get current lag
        Map<TopicPartition, Long> currentLag = getCurrentLag(consumerGroup, topic);
        
        if (shouldFastForward(currentLag)) {
            // Option 1: Reset to latest offset (lose messages)
            resetToLatest(consumerGroup, topic);
            
            // Option 2: Skip to reasonable lag threshold
            skipToThreshold(consumerGroup, topic, 1000); // Keep only last 1000 messages lag
        }
    }
    
    private void resetToLatest(String consumerGroup, String topic) {
        String[] cmd = {
            "kafka-consumer-groups.sh",
            "--bootstrap-server", "localhost:9092",
            "--group", consumerGroup,
            "--topic", topic,
            "--reset-offsets",
            "--to-latest",
            "--execute"
        };
        
        try {
            Process process = Runtime.getRuntime().exec(cmd);
            process.waitFor();
            log.info("Reset consumer group {} to latest offset for topic {}", 
                    consumerGroup, topic);
        } catch (Exception e) {
            log.error("Failed to reset consumer group", e);
        }
    }
    
    private void skipToThreshold(String consumerGroup, String topic, long maxLag) {
        AdminClient adminClient = AdminClient.create(getAdminConfig());
        
        try {
            // Get current end offsets
            Set<TopicPartition> partitions = getTopicPartitions(topic);
            Map<TopicPartition, Long> endOffsets = getEndOffsets(partitions);
            
            // Calculate target offsets (end - maxLag)
            Map<TopicPartition, OffsetAndMetadata> targetOffsets = new HashMap<>();
            for (Map.Entry<TopicPartition, Long> entry : endOffsets.entrySet()) {
                long targetOffset = Math.max(0, entry.getValue() - maxLag);
                targetOffsets.put(entry.getKey(), new OffsetAndMetadata(targetOffset));
            }
            
            // Alter consumer group offsets
            adminClient.alterConsumerGroupOffsets(consumerGroup, targetOffsets).all().get();
            
            log.info("Reset consumer group {} to maintain max lag of {} for topic {}", 
                    consumerGroup, maxLag, topic);
            
        } catch (Exception e) {
            log.error("Failed to skip to threshold", e);
        } finally {
            adminClient.close();
        }
    }
}
Monitoring
Key Metrics to Monitor
Broker Metrics
# Throughput Metrics
kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec
kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec
kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec

# Request Metrics
kafka.network:type=RequestMetrics,name=RequestsPerSec,request=Produce
kafka.network:type=RequestMetrics,name=RequestsPerSec,request=FetchConsumer
kafka.network:type=RequestMetrics,name=TotalTimeMs,request=Produce

# Partition Metrics
kafka.server:type=ReplicaManager,name=PartitionCount
kafka.server:type=ReplicaManager,name=LeaderCount
kafka.server:type=ReplicaManager,name=IsrShrinksPerSec
kafka.server:type=ReplicaManager,name=IsrExpandsPerSec

# Log Metrics
kafka.log:type=LogSize,name=Size,topic=*,partition=*
kafka.log:type=LogEndOffset,name=Value,topic=*,partition=*
Producer Metrics
# Throughput
kafka.producer:type=producer-metrics,client-id=*,attribute=record-send-rate
kafka.producer:type=producer-metrics,client-id=*,attribute=byte-rate

# Latency
kafka.producer:type=producer-metrics,client-id=*,attribute=record-queue-time-avg
kafka.producer:type=producer-metrics,client-id=*,attribute=request-latency-avg

# Errors
kafka.producer:type=producer-metrics,client-id=*,attribute=record-error-rate
kafka.producer:type=producer-metrics,client-id=*,attribute=record-retry-rate
Consumer Metrics
# Lag Monitoring
kafka.consumer:type=consumer-fetch-manager-metrics,client-id=*,attribute=records-lag-max
kafka.consumer:type=consumer-fetch-manager-metrics,client-id=*,attribute=records-lag-avg

# Throughput
kafka.consumer:type=consumer-fetch-manager-metrics,client-id=*,attribute=records-consumed-rate
kafka.consumer:type=consumer-fetch-manager-metrics,client-id=*,attribute=bytes-consumed-rate

# Processing Time
kafka.consumer:type=consumer-coordinator-metrics,client-id=*,attribute=commit-latency-avg
Monitoring Tools
1. JMX Monitoring
# Enable JMX on Kafka broker
export KAFKA_JMX_OPTS="-Dcom.sun.management.jmxremote \
  -Dcom.sun.management.jmxremote.authenticate=false \
  -Dcom.sun.management.jmxremote.ssl=false \
  -Dcom.sun.management.jmxremote.port=9999"

# Query JMX metrics
jconsole localhost:9999
2. Kafka Manager / AKHQ
# AKHQ Configuration
akhq:
  connections:
    my-cluster:
      properties:
        bootstrap.servers: "localhost:9092"
      
  security:
    default-group: admin
3. Prometheus + Grafana Setup
# docker-compose.yml for monitoring stack
version: '3'
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kafka'
    static_configs:
      - targets: ['localhost:9999']
    metrics_path: /metrics
4. Consumer Lag Monitoring Script
#!/bin/bash
# consumer-lag-monitor.sh

KAFKA_HOME=/opt/kafka
BOOTSTRAP_SERVERS="localhost:9092"

echo "Consumer Group Lag Report:"
echo "=========================="

for group in $($KAFKA_HOME/bin/kafka-consumer-groups.sh \
    --bootstrap-server $BOOTSTRAP_SERVERS --list); do
    
    echo "Group: $group"
    $KAFKA_HOME/bin/kafka-consumer-groups.sh \
        --bootstrap-server $BOOTSTRAP_SERVERS \
        --group $group --describe
    echo ""
done
Alert Thresholds
# Sample alerting rules
groups:
  - name: kafka.rules
    rules:
      - alert: KafkaConsumerLag
        expr: kafka_consumer_lag_sum > 1000
        for: 5m
        annotations:
          summary: "High consumer lag detected"
          
      - alert: KafkaBrokerDown
        expr: up{job="kafka"} == 0
        for: 1m
        annotations:
          summary: "Kafka broker is down"
          
      - alert: KafkaUnderReplicatedPartitions
        expr: kafka_server_replica_manager_under_replicated_partitions > 0
        for: 5m
        annotations:
          summary: "Under-replicated partitions detected"
Advanced Debugging and Troubleshooting
Common Issues and Solutions
1. Consumer Lag Issues
Symptoms:

High consumer lag
Slow message processing
Timeouts
Debugging Steps:

# Check consumer group status
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group my-group --describe

# Check topic details
kafka-topics.sh --bootstrap-server localhost:9092 \
  --topic my-topic --describe

# Monitor consumer metrics
kafka-run-class.sh kafka.tools.ConsumerPerformance \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --messages 10000
Solutions:

Increase number of consumers
Optimize consumer processing logic
Increase max.poll.records
Tune fetch.min.bytes and fetch.max.wait.ms
2. Rebalancing Issues
Symptoms:

Frequent rebalancing
Consumer timeouts
Processing interruptions
Debugging:

# Enable detailed consumer logging
log4j.logger.org.apache.kafka.clients.consumer=DEBUG
log4j.logger.org.apache.kafka.clients.consumer.internals=DEBUG

# Monitor rebalancing metrics
kafka.consumer:type=consumer-coordinator-metrics,client-id=*,attribute=rebalance-rate-per-hour
Solutions:

// Proper session timeout configuration
Properties props = new Properties();
props.put("session.timeout.ms", "30000");
props.put("heartbeat.interval.ms", "3000");
props.put("max.poll.interval.ms", "300000");

// Graceful shutdown
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    consumer.wakeup();
}));

try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        // Process records
    }
} catch (WakeupException e) {
    // Expected for shutdown
} finally {
    consumer.close();
}
3. Message Loss/Duplication
Message Loss Debugging:

// Check producer acknowledgment settings
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("enable.idempotence", true);

// Verify min.insync.replicas
kafka-configs.sh --bootstrap-server localhost:9092 \
  --entity-type topics --entity-name my-topic --describe
Duplication Debugging:

// Enable idempotent producer
props.put("enable.idempotence", true);
props.put("max.in.flight.requests.per.connection", 5);

// Implement idempotent consumer processing
@Component
public class IdempotentMessageProcessor {
    private final Set<String> processedMessageIds = new ConcurrentHashMap<>();
    
    public void processMessage(String messageId, String payload) {
        if (processedMessageIds.contains(messageId)) {
            log.info("Duplicate message detected: {}", messageId);
            return;
        }
        
        // Process message
        processedMessageIds.add(messageId);
    }
}
4. Performance Issues
Throughput Debugging:

# Producer performance test
kafka-producer-perf-test.sh --topic my-topic \
  --num-records 100000 \
  --record-size 1000 \
  --throughput 10000 \
  --producer-props bootstrap.servers=localhost:9092

# Consumer performance test
kafka-consumer-perf-test.sh --topic my-topic \
  --bootstrap-server localhost:9092 \
  --messages 100000
Network Issues:

# Check network configuration
netstat -tulpn | grep 9092
ss -tulpn | grep 9092

# Test connectivity
telnet broker-host 9092
kafka-broker-api-versions.sh --bootstrap-server localhost:9092
Debugging Tools
1. Kafka Scripts
# Topic management
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-topics.sh --bootstrap-server localhost:9092 --topic my-topic --describe

# Consumer group management
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-group --describe

# Log analysis
kafka-dump-log.sh --files /var/kafka-logs/my-topic-0/00000000000000000000.log --print-data-log

# Configuration
kafka-configs.sh --bootstrap-server localhost:9092 --entity-type brokers --entity-name 1 --describe
2. Log Analysis
# Broker logs
tail -f /var/log/kafka/server.log

# Key patterns to look for
grep "ERROR" /var/log/kafka/server.log
grep "WARN" /var/log/kafka/server.log
grep "rebalance" /var/log/kafka/server.log
grep "ISR" /var/log/kafka/server.log
3. JVM Debugging
# GC analysis
-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps

# Heap dump on OOM
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/kafka-heap-dump.hprof

# JFR profiling
-XX:+FlightRecorder -XX:StartFlightRecording=duration=60s,filename=kafka-profile.jfr
Advanced Troubleshooting Scenarios
Broker-Level Issues
1. Split Brain / Zombie Leaders

# Detect split brain scenario
kafka-topics.sh --bootstrap-server localhost:9092 --describe | grep "Leader: none"

# Check ISR status
kafka-topics.sh --bootstrap-server localhost:9092 --describe --under-replicated-partitions

# Force leader election if needed (use with caution)
kafka-leader-election.sh --bootstrap-server localhost:9092 --election-type preferred --all-topic-partitions
2. Disk Space Issues

public class DiskSpaceMonitor {
    @Scheduled(fixedRate = 60000) // Every minute
    public void checkDiskSpace() {
        File logDir = new File("/var/kafka-logs");
        long totalSpace = logDir.getTotalSpace();
        long freeSpace = logDir.getFreeSpace();
        double usagePercent = ((double)(totalSpace - freeSpace) / totalSpace) * 100;
        
        if (usagePercent > 85) {
            log.error("Disk usage critical: {}%", usagePercent);
            // Trigger log cleanup or alerting
            triggerLogCleanup();
        } else if (usagePercent > 75) {
            log.warn("Disk usage warning: {}%", usagePercent);
        }
    }
    
    private void triggerLogCleanup() {
        // Force log cleanup
        try {
            Process process = Runtime.getRuntime().exec(
                "kafka-configs.sh --bootstrap-server localhost:9092 " +
                "--entity-type topics --alter --add-config cleanup.policy=delete,retention.ms=3600000"
            );
            process.waitFor();
        } catch (Exception e) {
            log.error("Failed to trigger cleanup", e);
        }
    }
}
3. Memory Issues and GC Problems

public class MemoryDiagnostics {
    
    public void analyzeMemoryUsage() {
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        MemoryUsage nonHeapUsage = memoryBean.getNonHeapMemoryUsage();
        
        long heapUsed = heapUsage.getUsed();
        long heapMax = heapUsage.getMax();
        double heapUtilization = (double) heapUsed / heapMax * 100;
        
        log.info("Heap utilization: {}% ({}/{})", 
                heapUtilization, formatBytes(heapUsed), formatBytes(heapMax));
        
        if (heapUtilization > 85) {
            log.error("High heap utilization detected");
            dumpThreadStacks();
            analyzeTopObjects();
        }
    }
    
    private void dumpThreadStacks() {
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        ThreadInfo[] threadInfos = threadBean.dumpAllThreads(true, true);
        
        for (ThreadInfo threadInfo : threadInfos) {
            if (threadInfo.getThreadState() == Thread.State.BLOCKED ||
                threadInfo.getThreadState() == Thread.State.WAITING) {
                log.warn("Thread {} in state {} for {}ms", 
                        threadInfo.getThreadName(), 
                        threadInfo.getThreadState(),
                        threadInfo.getBlockedTime());
            }
        }
    }
}
Network and Connectivity Issues
1. Network Partitions

public class NetworkDiagnostics {
    
    public void diagnoseNetworkIssues(String brokerHost, int brokerPort) {
        // Test basic connectivity
        try (Socket socket = new Socket()) {
            socket.connect(new InetSocketAddress(brokerHost, brokerPort), 5000);
            log.info("Network connectivity to {}:{} - OK", brokerHost, brokerPort);
        } catch (IOException e) {
            log.error("Network connectivity failed to {}:{}", brokerHost, brokerPort, e);
            return;
        }
        
        // Test Kafka protocol
        Properties props = new Properties();
        props.put("bootstrap.servers", brokerHost + ":" + brokerPort);
        props.put("request.timeout.ms", "5000");
        props.put("connections.max.idle.ms", "10000");
        
        try (AdminClient adminClient = AdminClient.create(props)) {
            DescribeClusterResult clusterResult = adminClient.describeCluster();
            clusterResult.nodes().get(5, TimeUnit.SECONDS);
            log.info("Kafka protocol connectivity - OK");
        } catch (Exception e) {
            log.error("Kafka protocol connectivity failed", e);
            diagnoseKafkaConnectivity(props);
        }
    }
    
    private void diagnoseKafkaConnectivity(Properties props) {
        // Check for common issues
        checkDNSResolution(props.getProperty("bootstrap.servers"));
        checkFirewallRules();
        checkKafkaVersionCompatibility();
    }
}
2. SSL/TLS Issues

public class SSLDiagnostics {
    
    public void diagnoseSSLIssues(String keystorePath, String truststorePath) {
        try {
            // Validate keystore
            KeyStore keystore = KeyStore.getInstance("JKS");
            keystore.load(new FileInputStream(keystorePath), "password".toCharArray());
            log.info("Keystore validation - OK");
            
            // Validate truststore
            KeyStore truststore = KeyStore.getInstance("JKS");
            truststore.load(new FileInputStream(truststorePath), "password".toCharArray());
            log.info("Truststore validation - OK");
            
            // Check certificate expiration
            Enumeration<String> aliases = keystore.aliases();
            while (aliases.hasMoreElements()) {
                String alias = aliases.nextElement();
                X509Certificate cert = (X509Certificate) keystore.getCertificate(alias);
                if (cert != null) {
                    Date expiry = cert.getNotAfter();
                    long daysUntilExpiry = (expiry.getTime() - System.currentTimeMillis()) / (1000 * 60 * 60 * 24);
                    
                    if (daysUntilExpiry < 30) {
                        log.warn("Certificate {} expires in {} days", alias, daysUntilExpiry);
                    }
                }
            }
            
        } catch (Exception e) {
            log.error("SSL configuration issue", e);
        }
    }
}
Consumer-Specific Troubleshooting
1. Rebalancing Storms

public class RebalancingDiagnostics {
    private final AtomicLong rebalanceCount = new AtomicLong(0);
    private final AtomicLong lastRebalanceTime = new AtomicLong(System.currentTimeMillis());
    
    public class DiagnosticRebalanceListener implements ConsumerRebalanceListener {
        
        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            long currentTime = System.currentTimeMillis();
            long timeSinceLastRebalance = currentTime - lastRebalanceTime.get();
            long count = rebalanceCount.incrementAndGet();
            
            log.warn("Rebalance #{} - Partitions revoked: {} - Time since last: {}ms", 
                    count, partitions.size(), timeSinceLastRebalance);
            
            if (timeSinceLastRebalance < 30000) { // Less than 30 seconds
                log.error("Frequent rebalancing detected - possible configuration issue");
                diagnoseRebalanceIssues();
            }
            
            lastRebalanceTime.set(currentTime);
        }
        
        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            log.info("Partitions assigned: {}", partitions.size());
        }
    }
    
    private void diagnoseRebalanceIssues() {
        // Check common causes
        checkSessionTimeout();
        checkHeartbeatInterval();
        checkMaxPollInterval();
        checkProcessingTime();
    }
    
    private void checkMaxPollInterval() {
        // Monitor if processing takes longer than max.poll.interval.ms
        long maxPollInterval = getConsumerConfig("max.poll.interval.ms", 300000);
        long processingTime = measureProcessingTime();
        
        if (processingTime > maxPollInterval * 0.8) {
            log.warn("Processing time {}ms is close to max.poll.interval {}ms", 
                    processingTime, maxPollInterval);
        }
    }
}
2. Offset Management Issues

public class OffsetDiagnostics {
    
    public void diagnoseOffsetIssues(KafkaConsumer<String, String> consumer, String groupId) {
        // Check for offset out of range
        try {
            consumer.poll(Duration.ofMillis(1000));
        } catch (OffsetOutOfRangeException e) {
            log.error("Offset out of range for group {}", groupId);
            handleOffsetOutOfRange(consumer, e.offsetOutOfRangePartitions());
        }
        
        // Check for committed vs current offset drift
        checkOffsetDrift(consumer, groupId);
    }
    
    private void handleOffsetOutOfRangeException(KafkaConsumer<String, String> consumer, 
                                               Set<TopicPartition> partitions) {
        for (TopicPartition partition : partitions) {
            // Get available offset range
            Map<TopicPartition, Long> beginningOffsets = consumer.beginningOffsets(Collections.singleton(partition));
            Map<TopicPartition, Long> endOffsets = consumer.endOffsets(Collections.singleton(partition));
            
            long beginningOffset = beginningOffsets.get(partition);
            long endOffset = endOffsets.get(partition);
            
            log.info("Partition {} offset range: {} to {}", partition, beginningOffset, endOffset);
            
            // Reset to appropriate offset
            consumer.seek(partition, beginningOffset);
            log.warn("Reset partition {} to beginning offset {}", partition, beginningOffset);
        }
    }
    
    private void checkOffsetDrift(KafkaConsumer<String, String> consumer, String groupId) {
        AdminClient adminClient = AdminClient.create(getAdminConfig());
        
        try {
            Map<TopicPartition, OffsetAndMetadata> committedOffsets = adminClient
                .listConsumerGroupOffsets(groupId)
                .partitionsToOffsetAndMetadata()
                .get();
            
            for (Map.Entry<TopicPartition, OffsetAndMetadata> entry : committedOffsets.entrySet()) {
                TopicPartition tp = entry.getKey();
                long committedOffset = entry.getValue().offset();
                long currentPosition = consumer.position(tp);
                
                long drift = currentPosition - committedOffset;
                if (Math.abs(drift) > 1000) {
                    log.warn("Large offset drift detected for {}: committed={}, current={}, drift={}", 
                            tp, committedOffset, currentPosition, drift);
                }
            }
        } catch (Exception e) {
            log.error("Failed to check offset drift", e);
        } finally {
            adminClient.close();
        }
    }
}
Producer-Specific Troubleshooting
1. Producer Metadata Issues

public class ProducerDiagnostics {
    
    public void diagnoseMetadataIssues(KafkaProducer<String, String> producer) {
        // Force metadata refresh
        try {
            List<PartitionInfo> partitions = producer.partitionsFor("test-topic");
            if (partitions == null || partitions.isEmpty()) {
                log.error("No partition metadata available for topic");
                diagnoseMetadataRefresh();
            } else {
                log.info("Partition metadata: {} partitions available", partitions.size());
            }
        } catch (Exception e) {
            log.error("Metadata fetch failed", e);
        }
    }
    
    private void diagnoseMetadataRefresh() {
        // Check metadata refresh settings
        Properties props = getProducerConfig();
        long metadataMaxAge = Long.parseLong(props.getProperty("metadata.max.age.ms", "300000"));
        long retryBackoff = Long.parseLong(props.getProperty("retry.backoff.ms", "100"));
        
        log.info("Metadata config - max.age: {}ms, retry.backoff: {}ms", 
                metadataMaxAge, retryBackoff);
        
        if (metadataMaxAge > 600000) { // 10 minutes
            log.warn("Metadata max age is quite high: {}ms", metadataMaxAge);
        }
    }
    
    public void monitorProducerMetrics(KafkaProducer<String, String> producer) {
        Map<MetricName, ? extends Metric> metrics = producer.metrics();
        
        // Key metrics to monitor
        double recordErrorRate = getMetricValue(metrics, "record-error-rate");
        double recordRetryRate = getMetricValue(metrics, "record-retry-rate");
        double batchSizeAvg = getMetricValue(metrics, "batch-size-avg");
        double requestLatencyAvg = getMetricValue(metrics, "request-latency-avg");
        
        log.info("Producer metrics - Error rate: {}, Retry rate: {}, Batch size: {}, Latency: {}ms",
                recordErrorRate, recordRetryRate, batchSizeAvg, requestLatencyAvg);
        
        if (recordErrorRate > 0.01) { // 1% error rate
            log.warn("High error rate detected: {}", recordErrorRate);
        }
        
        if (recordRetryRate > 0.1) { // 10% retry rate
            log.warn("High retry rate detected: {}", recordRetryRate);
        }
    }
}
2. Serialization Issues

public class SerializationDiagnostics {
    
    public void diagnoseSerializationIssues() {
        // Test serialization/deserialization
        testStringSerialization();
        testJsonSerialization();
        testAvroSerialization();
    }
    
    private void testJsonSerialization() {
        try {
            ObjectMapper mapper = new ObjectMapper();
            TestObject testObj = new TestObject("test", 123);
            
            // Test serialization
            String json = mapper.writeValueAsString(testObj);
            log.info("JSON serialization successful: {}", json);
            
            // Test deserialization
            TestObject deserialized = mapper.readValue(json, TestObject.class);
            log.info("JSON deserialization successful: {}", deserialized);
            
        } catch (Exception e) {
            log.error("JSON serialization/deserialization failed", e);
        }
    }
    
    private void testAvroSerialization() {
        try {
            // Test Avro schema compatibility
            Schema schema = new Schema.Parser().parse(getClass().getResourceAsStream("/user.avsc"));
            
            // Create test record
            GenericRecord record = new GenericData.Record(schema);
            record.put("name", "Test User");
            record.put("age", 25);
            
            // Serialize
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            DatumWriter<GenericRecord> writer = new GenericDatumWriter<>(schema);
            Encoder encoder = EncoderFactory.get().binaryEncoder(out, null);
            writer.write(record, encoder);
            encoder.flush();
            
            byte[] serialized = out.toByteArray();
            log.info("Avro serialization successful: {} bytes", serialized.length);
            
            // Deserialize
            DatumReader<GenericRecord> reader = new GenericDatumReader<>(schema);
            Decoder decoder = DecoderFactory.get().binaryDecoder(serialized, null);
            GenericRecord deserialized = reader.read(null, decoder);
            
            log.info("Avro deserialization successful: {}", deserialized);
            
        } catch (Exception e) {
            log.error("Avro serialization/deserialization failed", e);
        }
    }
}
Troubleshooting Tools and Scripts
Comprehensive Health Check Script
#!/bin/bash
# kafka-health-check.sh

KAFKA_HOME="/opt/kafka"
BOOTSTRAP_SERVERS="localhost:9092"
LOG_FILE="/var/log/kafka-health-check.log"

echo "=== Kafka Health Check - $(date) ===" | tee -a $LOG_FILE

# 1. Broker connectivity
echo "1. Testing broker connectivity..." | tee -a $LOG_FILE
timeout 10 $KAFKA_HOME/bin/kafka-broker-api-versions.sh --bootstrap-server $BOOTSTRAP_SERVERS > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "✓ Broker connectivity OK" | tee -a $LOG_FILE
else
    echo "✗ Broker connectivity FAILED" | tee -a $LOG_FILE
fi

# 2. Topic listing
echo "2. Testing topic operations..." | tee -a $LOG_FILE
TOPIC_COUNT=$($KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BOOTSTRAP_SERVERS --list 2>/dev/null | wc -l)
echo "✓ Found $TOPIC_COUNT topics" | tee -a $LOG_FILE

# 3. Under-replicated partitions
echo "3. Checking under-replicated partitions..." | tee -a $LOG_FILE
UNDER_REPLICATED=$($KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BOOTSTRAP_SERVERS --describe --under-replicated-partitions 2>/dev/null | wc -l)
if [ $UNDER_REPLICATED -eq 0 ]; then
    echo "✓ No under-replicated partitions" | tee -a $LOG_FILE
else
    echo "✗ Found $UNDER_REPLICATED under-replicated partitions" | tee -a $LOG_FILE
fi

# 4. Consumer group lag
echo "4. Checking consumer group lag..." | tee -a $LOG_FILE
CONSUMER_GROUPS=$($KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server $BOOTSTRAP_SERVERS --list 2>/dev/null)
for group in $CONSUMER_GROUPS; do
    MAX_LAG=$($KAFKA_HOME/bin/kafka-consumer-groups.sh --bootstrap-server $BOOTSTRAP_SERVERS --group $group --describe 2>/dev/null | awk 'NR>1 && $6 != "-" {if($6 > max) max=$6} END {print max+0}')
    if [ $MAX_LAG -gt 10000 ]; then
        echo "⚠ Group $group has high lag: $MAX_LAG" | tee -a $LOG_FILE
    fi
done

# 5. Disk space
echo "5. Checking disk space..." | tee -a $LOG_FILE
DISK_USAGE=$(df /var/kafka-logs | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 80 ]; then
    echo "⚠ High disk usage: ${DISK_USAGE}%" | tee -a $LOG_FILE
else
    echo "✓ Disk usage OK: ${DISK_USAGE}%" | tee -a $LOG_FILE
fi

echo "=== Health Check Complete ===" | tee -a $LOG_FILE
Performance Diagnostic Script
#!/bin/bash
# kafka-performance-diagnostic.sh

KAFKA_HOME="/opt/kafka"
BOOTSTRAP_SERVERS="localhost:9092"
TEST_TOPIC="performance-test-topic"

echo "=== Kafka Performance Diagnostic ==="

# Create test topic
$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BOOTSTRAP_SERVERS \
    --create --topic $TEST_TOPIC --partitions 6 --replication-factor 3 \
    --if-not-exists

# Producer performance test
echo "Running producer performance test..."
$KAFKA_HOME/bin/kafka-producer-perf-test.sh \
    --topic $TEST_TOPIC \
    --num-records 100000 \
    --record-size 1000 \
    --throughput 10000 \
    --producer-props bootstrap.servers=$BOOTSTRAP_SERVERS \
    acks=all retries=2147483647 enable.idempotence=true

# Consumer performance test
echo "Running consumer performance test..."
$KAFKA_HOME/bin/kafka-consumer-perf-test.sh \
    --topic $TEST_TOPIC \
    --bootstrap-server $BOOTSTRAP_SERVERS \
    --messages 100000 \
    --threads 1

# Cleanup
echo "Cleaning up test topic..."
$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server $BOOTSTRAP_SERVERS \
    --delete --topic $TEST_TOPIC

echo "Performance diagnostic complete"
Performance Tuning
Producer Tuning
Throughput Optimization
Properties highThroughputProps = new Properties();
// Batching
highThroughputProps.put("batch.size", 65536);          // 64KB batches
highThroughputProps.put("linger.ms", 10);              // Wait 10ms for batching
highThroughputProps.put("buffer.memory", 67108864);    // 64MB buffer

// Compression
highThroughputProps.put("compression.type", "snappy"); // or "lz4"

// Acknowledgment
highThroughputProps.put("acks", "1");                  // Leader acknowledgment only

// Parallelism
highThroughputProps.put("max.in.flight.requests.per.connection", 5);
Latency Optimization
Properties lowLatencyProps = new Properties();
// Immediate sending
lowLatencyProps.put("batch.size", 1);
lowLatencyProps.put("linger.ms", 0);

// No compression (trades throughput for latency)
lowLatencyProps.put("compression.type", "none");

// Network settings
lowLatencyProps.put("request.timeout.ms", 10000);
Consumer Tuning
High Throughput Consumer
Properties props = new Properties();
// Fetch settings
props.put("fetch.min.bytes", 50000);        // Wait for 50KB
props.put("fetch.max.wait.ms", 500);        // Or 500ms timeout
props.put("max.partition.fetch.bytes", 2097152); // 2MB per partition
props.put("max.poll.records", 1000);        // Process 1000 records per poll

// Auto-commit for performance (trade-off with reliability)
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "1000");
Parallel Processing
@Component
public class ParallelConsumer {
    private final ExecutorService executor = Executors.newFixedThreadPool(10);
    
    @KafkaListener(topics = "my-topic")
    public void consume(List<ConsumerRecord<String, String>> records) {
        // Process records in parallel
        records.parallelStream().forEach(record -> {
            executor.submit(() -> processRecord(record));
        });
    }
    
    private void processRecord(ConsumerRecord<String, String> record) {
        // Processing logic
    }
}
Broker Tuning
Hardware Recommendations
CPU: 24+ cores for high throughput
Memory: 64GB+ RAM
Storage: NVMe SSDs for logs
Network: 10GbE+ for high throughput clusters
OS-Level Tuning
# File descriptor limits
echo "* soft nofile 100000" >> /etc/security/limits.conf
echo "* hard nofile 100000" >> /etc/security/limits.conf

# VM settings
echo "vm.swappiness=1" >> /etc/sysctl.conf
echo "vm.dirty_ratio=80" >> /etc/sysctl.conf
echo "vm.dirty_background_ratio=5" >> /etc/sysctl.conf
echo "net.core.wmem_default=262144" >> /etc/sysctl.conf
echo "net.core.rmem_default=262144" >> /etc/sysctl.conf
JVM Tuning
# Heap size (typically 50% of system memory, max 32GB)
export KAFKA_HEAP_OPTS="-Xmx16g -Xms16g"

# GC tuning for Kafka
export KAFKA_JVM_PERFORMANCE_OPTS="-XX:+UseG1GC \
  -XX:MaxGCPauseMillis=20 \
  -XX:InitiatingHeapOccupancyPercent=35 \
  -XX:+ExplicitGCInvokesConcurrent \
  -XX:MaxInlineLevel=15 \
  -Djava.awt.headless=true"

# Additional JVM options
export KAFKA_JVM_PERFORMANCE_OPTS="$KAFKA_JVM_PERFORMANCE_OPTS \
  -XX:+UnlockExperimentalVMOptions \
  -XX:+UseCGroupMemoryLimitForHeap"
Broker Configuration Tuning
# Network threads (1 per core)
num.network.threads=16

# I/O threads (1 per disk)
num.io.threads=8

# Socket buffer sizes
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600

# Log configuration
log.segment.bytes=1073741824        # 1GB segments
log.retention.check.interval.ms=300000
log.cleaner.enable=true
log.cleanup.policy=delete

# Replication settings
replica.fetch.max.bytes=2097152
replica.socket.timeout.ms=30000
replica.socket.receive.buffer.bytes=65536
Partition Strategy
Optimal Partition Count
Guidelines:
- Start with: (Target Throughput) / (Partition Throughput)
- Consider: Number of consumers in largest consumer group
- Account for: Future growth and rebalancing overhead
- Limit: Don't exceed 4000 partitions per broker

Example:
Target: 100MB/s
Per partition: 10MB/s
Partitions needed: 10-15 partitions
Custom Partitioner
public class CustomPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes, Cluster cluster) {
        if (key == null) {
            return ThreadLocalRandom.current().nextInt(cluster.partitionCountForTopic(topic));
        }
        
        // Custom logic based on key
        if (key.toString().startsWith("priority-")) {
            return 0; // High priority partition
        }
        
        return Math.abs(key.hashCode()) % cluster.partitionCountForTopic(topic);
    }
}
Monitoring Performance
# Throughput monitoring script
#!/bin/bash
while true; do
    echo "=== $(date) ==="
    kafka-run-class.sh kafka.tools.JmxTool \
        --object-name kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec \
        --jmx-url service:jmx:rmi:///jndi/rmi://localhost:9999/jmxrmi \
        --date-format "yyyy-MM-dd HH:mm:ss" \
        --attributes Count | tail -1
    sleep 10
done
Best Practices
Design Patterns
1. Event Sourcing
@Entity
public class OrderAggregate {
    private String orderId;
    private OrderStatus status;
    private List<OrderEvent> events = new ArrayList<>();
    
    public void addItem(String itemId, int quantity) {
        OrderEvent event = new ItemAddedEvent(orderId, itemId, quantity);
        apply(event);
        events.add(event);
    }
    
    public void publishEvents(KafkaTemplate<String, OrderEvent> producer) {
        events.forEach(event -> 
            producer.send("order-events", orderId, event));
        events.clear();
    }
}
2. CQRS (Command Query Responsibility Segregation)
// Command side
@Service
public class OrderCommandService {
    @Autowired
    private KafkaTemplate<String, OrderCommand> producer;
    
    public void createOrder(CreateOrderCommand command) {
        producer.send("order-commands", command.getOrderId(), command);
    }
}

// Query side
@Service
public class OrderQueryService {
    @KafkaListener(topics = "order-events")
    public void updateReadModel(OrderEvent event) {
        // Update read-optimized database
        readModelRepository.update(event);
    }
}
3. Saga Pattern
@Component
public class OrderSagaOrchestrator {
    
    @KafkaListener(topics = "order-created")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Step 1: Reserve inventory
        producer.send("inventory-commands", 
            new ReserveInventoryCommand(event.getOrderId(), event.getItems()));
    }
    
    @KafkaListener(topics = "inventory-reserved")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        // Step 2: Process payment
        producer.send("payment-commands",
            new ProcessPaymentCommand(event.getOrderId(), event.getAmount()));
    }
    
    @KafkaListener(topics = "payment-failed")
    public void handlePaymentFailed(PaymentFailedEvent event) {
        // Compensating action: Release inventory
        producer.send("inventory-commands",
            new ReleaseInventoryCommand(event.getOrderId()));
    }
}
Security Best Practices
1. SSL/TLS Configuration
# Broker SSL configuration
listeners=SSL://localhost:9093
security.inter.broker.protocol=SSL
ssl.keystore.location=/etc/kafka/ssl/kafka.server.keystore.jks
ssl.keystore.password=password
ssl.key.password=password
ssl.truststore.location=/etc/kafka/ssl/kafka.server.truststore.jks
ssl.truststore.password=password
ssl.client.auth=required
2. SASL Authentication
# SASL_PLAINTEXT configuration
listeners=SASL_PLAINTEXT://localhost:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# JAAS configuration
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin-secret"
    user_admin="admin-secret"
    user_alice="alice-secret";
};
3. ACL (Access Control Lists)
# Create ACLs
kafka-acls.sh --bootstrap-server localhost:9092 \
  --add --allow-principal User:alice \
  --operation Read --topic my-topic

kafka-acls.sh --bootstrap-server localhost:9092 \
  --add --allow-principal User:bob \
  --operation Write --topic my-topic

# List ACLs
kafka-acls.sh --bootstrap-server localhost:9092 --list
Operational Best Practices
1. Capacity Planning
Calculation Framework:
1. Message size: Average and peak message sizes
2. Message rate: Messages per second
3. Retention: How long to keep data
4. Replication: Factor for redundancy
5. Growth: Expected growth rate

Example:
- 1000 msgs/sec × 1KB avg = 1MB/s
- 24 hours retention = 86.4GB/day
- 3x replication = 259.2GB/day
- 30 days retention = 7.77TB
2. Backup and Recovery
# Topic backup script
#!/bin/bash
BACKUP_DIR="/backup/kafka/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup topic metadata
kafka-topics.sh --bootstrap-server localhost:9092 --list > $BACKUP_DIR/topics.txt

# Backup configurations
kafka-configs.sh --bootstrap-server localhost:9092 \
  --entity-type topics --describe > $BACKUP_DIR/topic-configs.txt

# Backup consumer group offsets
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --all-groups --describe > $BACKUP_DIR/consumer-groups.txt
3. Rolling Updates
# Rolling broker update procedure
1. Update broker configuration
2. Restart one broker at a time
3. Wait for ISR to be fully replicated
4. Monitor metrics and logs
5. Proceed to next broker

# Script for controlled restart
#!/bin/bash
for broker in 1 2 3; do
    echo "Restarting broker $broker"
    systemctl restart kafka-$broker
    
    # Wait for broker to rejoin
    sleep 30
    
    # Check cluster health
    kafka-topics.sh --bootstrap-server localhost:9092 \
      --describe --under-replicated-partitions
    
    read -p "Continue with next broker? (y/n): " confirm
    [[ $confirm != "y" ]] && exit 1
done
Common Use Cases
1. Real-time Analytics Pipeline
// Stream processing with Kafka Streams
@Component
public class AnalyticsStreamProcessor {
    
    @Bean
    public KStream<String, ClickEvent> processClickStream(StreamsBuilder builder) {
        return builder.stream("click-events")
            .groupByKey()
            .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
            .count()
            .toStream()
            .to("click-analytics");
    }
}
2. Microservices Communication
// Order service publishes events
@Service
public class OrderService {
    @Autowired
    private KafkaTemplate<String, OrderEvent> producer;
    
    public Order createOrder(CreateOrderRequest request) {
        Order order = new Order(request);
        orderRepository.save(order);
        
        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        producer.send("order-events", order.getId(), event);
        
        return order;
    }
}

// Inventory service consumes events
@Service
public class InventoryService {
    @KafkaListener(topics = "order-events")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Update inventory
        inventoryService.reserveItems(event.getItems());
    }
}
3. Log Aggregation
// Application logs to Kafka
@Component
public class KafkaLogAppender extends AppenderBase<ILoggingEvent> {
    private KafkaTemplate<String, String> producer;
    
    @Override
    protected void append(ILoggingEvent event) {
        LogEntry logEntry = new LogEntry(
            event.getLevel().toString(),
            event.getMessage(),
            event.getTimeStamp()
        );
        
        producer.send("application-logs", 
            event.getLoggerName(), 
            objectMapper.writeValueAsString(logEntry));
    }
}
4. CDC (Change Data Capture)
// Debezium connector configuration
{
  "name": "mysql-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-server",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "password",
    "database.server.id": "1",
    "database.server.name": "inventory",
    "database.include.list": "inventory",
    "database.history.kafka.bootstrap.servers": "localhost:9092",
    "database.history.kafka.topic": "schema-changes"
  }
}
This comprehensive study guide covers all the essential aspects of Apache Kafka that you requested. Each section provides both theoretical understanding and practical examples that you can use in real-world scenarios.
