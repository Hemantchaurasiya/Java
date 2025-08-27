# Phase 5: Concurrent Collections

## 13. Thread-Safe Collections

### 13.1 ConcurrentHashMap: High-Performance Concurrent Map

**Key Features:**
- Segment-based locking (Java 7) evolved to CAS + synchronized (Java 8+)
- Non-blocking reads, minimal blocking writes
- No iteration interference
- Atomic operations support

**Core Methods:**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic operations
map.putIfAbsent("key", 1);
map.replace("key", oldValue, newValue);
map.compute("key", (k, v) -> v == null ? 1 : v + 1);
map.merge("key", 1, Integer::sum);

// Bulk operations (Java 8+)
map.forEach((k, v) -> System.out.println(k + "=" + v));
map.reduceValues(1, Integer::sum);
```

**Implementation Example:**
```java
public class ConcurrentCounter {
    private final ConcurrentHashMap<String, AtomicInteger> counters = new ConcurrentHashMap<>();
    
    public void increment(String key) {
        counters.computeIfAbsent(key, k -> new AtomicInteger(0)).incrementAndGet();
    }
    
    public int getCount(String key) {
        return counters.getOrDefault(key, new AtomicInteger(0)).get();
    }
}
```

### 13.2 CopyOnWriteArrayList: Read-Heavy Scenarios

**Characteristics:**
- Immutable snapshot for iterations
- Write operations create new array copy
- Perfect for read-heavy, write-light scenarios
- No synchronization needed for reads

**Usage Pattern:**
```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

// Multiple readers can iterate simultaneously
// without ConcurrentModificationException
for (String item : list) {
    // Safe iteration even if list is modified during iteration
    processItem(item);
}

// Write operations are expensive but thread-safe
list.add("new item"); // Creates new underlying array
```

### 13.3 BlockingQueue Implementations

#### ArrayBlockingQueue
- **Bounded queue** with array-based storage
- **FIFO ordering**
- **Blocking operations** for full/empty conditions

```java
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(100);

// Producer thread
queue.put(new Task()); // Blocks if queue is full

// Consumer thread
Task task = queue.take(); // Blocks if queue is empty
```

#### LinkedBlockingQueue
- **Optionally bounded** queue with linked nodes
- **Better throughput** than ArrayBlockingQueue
- **Separate locks** for head and tail operations

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(1000);

// Non-blocking operations
boolean added = queue.offer("item"); // Returns false if full
String item = queue.poll(); // Returns null if empty

// Timed operations
queue.offer("item", 5, TimeUnit.SECONDS);
String result = queue.poll(5, TimeUnit.SECONDS);
```

#### PriorityBlockingQueue
- **Unbounded queue** with priority ordering
- Elements must implement **Comparable** or use **Comparator**
- **Natural ordering** for priority

```java
PriorityBlockingQueue<Task> priorityQueue = new PriorityBlockingQueue<>();

class Task implements Comparable<Task> {
    private final int priority;
    private final String name;
    
    @Override
    public int compareTo(Task other) {
        return Integer.compare(other.priority, this.priority); // Higher priority first
    }
}
```

#### DelayQueue
- **Unbounded queue** for delayed elements
- Elements available only after their **delay expires**
- Implements **Delayed interface**

```java
DelayQueue<DelayedTask> delayQueue = new DelayQueue<>();

class DelayedTask implements Delayed {
    private final long executeTime;
    private final String taskName;
    
    public DelayedTask(String taskName, long delayInMillis) {
        this.taskName = taskName;
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
}
```

#### SynchronousQueue
- **Zero-capacity queue** for direct thread-to-thread handoff
- **Each put must wait** for corresponding take
- **Useful for thread coordination**

```java
SynchronousQueue<String> handoffQueue = new SynchronousQueue<>();

// Producer thread
handoffQueue.put("data"); // Blocks until another thread takes it

// Consumer thread  
String data = handoffQueue.take(); // Blocks until another thread puts data
```

## 14. Collection Synchronization

### 14.1 Collections.synchronizedXxx() Wrappers

**Legacy synchronization approach:**
```java
List<String> synchronizedList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> synchronizedMap = Collections.synchronizedMap(new HashMap<>());

// Important: Manual synchronization needed for iteration
synchronized (synchronizedList) {
    for (String item : synchronizedList) {
        processItem(item);
    }
}
```

**Limitations:**
- **Coarse-grained locking** reduces performance
- **Manual synchronization** required for compound operations
- **Iteration still needs** external synchronization

### 14.2 Concurrent vs Synchronized Collections

| Aspect | Synchronized Collections | Concurrent Collections |
|--------|-------------------------|----------------------|
| **Locking** | Single lock for entire collection | Fine-grained locking or lock-free |
| **Performance** | Poor under contention | Excellent scalability |
| **Iteration** | Manual synchronization needed | Safe concurrent iteration |
| **Null Values** | Generally supported | Often not supported |
| **Fail-Fast** | Yes (with external sync) | Weakly consistent |

### 14.3 Fail-Fast vs Fail-Safe Iterators

**Fail-Fast Iterators (Traditional Collections):**
```java
List<String> list = new ArrayList<>();
// Throws ConcurrentModificationException if modified during iteration
for (String item : list) {
    if (shouldRemove(item)) {
        list.remove(item); // Throws exception!
    }
}
```

**Fail-Safe Iterators (Concurrent Collections):**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
// Safe to modify during iteration - works on snapshot
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    if (entry.getValue() < 0) {
        map.remove(entry.getKey()); // Safe operation
    }
}
```

## Practical Implementation Examples

### Producer-Consumer with BlockingQueue
```java
public class ProducerConsumerExample {
    private final BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);
    private final AtomicBoolean shutdown = new AtomicBoolean(false);
    
    class Producer implements Runnable {
        @Override
        public void run() {
            int value = 0;
            try {
                while (!shutdown.get()) {
                    queue.put(value++);
                    Thread.sleep(100); // Simulate work
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                while (!shutdown.get() || !queue.isEmpty()) {
                    Integer value = queue.poll(1, TimeUnit.SECONDS);
                    if (value != null) {
                        processValue(value);
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        private void processValue(Integer value) {
            // Process the value
            System.out.println("Processed: " + value);
        }
    }
    
    public void start() {
        new Thread(new Producer()).start();
        new Thread(new Consumer()).start();
    }
    
    public void shutdown() {
        shutdown.set(true);
    }
}
```

### Cache Implementation with ConcurrentHashMap
```java
public class ThreadSafeCache<K, V> {
    private final ConcurrentHashMap<K, V> cache = new ConcurrentHashMap<>();
    private final Function<K, V> valueLoader;
    
    public ThreadSafeCache(Function<K, V> valueLoader) {
        this.valueLoader = valueLoader;
    }
    
    public V get(K key) {
        // Atomic operation - load if absent
        return cache.computeIfAbsent(key, valueLoader);
    }
    
    public void evict(K key) {
        cache.remove(key);
    }
    
    public void clear() {
        cache.clear();
    }
    
    // Bulk operations for efficiency
    public Map<K, V> getAll(Set<K> keys) {
        return keys.parallelStream()
                   .collect(Collectors.toConcurrentMap(
                       Function.identity(), 
                       this::get
                   ));
    }
}
```

## Performance Considerations

### Choosing the Right Collection

1. **High Read, Low Write**: `CopyOnWriteArrayList`, `CopyOnWriteArraySet`
2. **Balanced Read/Write**: `ConcurrentHashMap`, `ConcurrentLinkedQueue`
3. **Producer-Consumer**: `BlockingQueue` implementations
4. **Priority Processing**: `PriorityBlockingQueue`
5. **Timed Operations**: `DelayQueue`
6. **Thread Handoff**: `SynchronousQueue`

### Memory and GC Impact

- **CopyOnWriteArrayList**: High memory usage during writes
- **ConcurrentHashMap**: Memory-efficient with good GC behavior
- **LinkedBlockingQueue**: Can cause memory pressure if unbounded
- **PriorityBlockingQueue**: Additional overhead for maintaining order



## Next Steps: Phase 6 Preview

import java.util.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.util.function.Function;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * PHASE 5 REAL-WORLD SCENARIOS: CONCURRENT COLLECTIONS MASTERY
 * 
 * These scenarios simulate actual production problems:
 * 1. Web Server Request Cache (ConcurrentHashMap)
 * 2. Task Queue System (BlockingQueue)
 * 3. Event Notification System (CopyOnWriteArrayList)
 * 4. Rate Limiter (SynchronousQueue + Semaphore)
 * 5. Order Processing System (PriorityBlockingQueue)
 * 6. Delayed Job Scheduler (DelayQueue)
 * 7. Session Management (ConcurrentHashMap + ScheduledExecutor)
 * 8. Log Aggregation System (Multiple Collections)
 */

// ==================== SCENARIO 1: WEB SERVER REQUEST CACHE ====================
/**
 * Real Problem: High-traffic web server needs to cache expensive database queries
 * Requirements: Thread-safe, high-performance, automatic cleanup
 */
class WebServerCache<K, V> {
    private final ConcurrentHashMap<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
    private final Function<K, V> dataLoader;
    private final long ttlMillis;
    private final ScheduledExecutorService cleanupExecutor;
    
    // Statistics tracking
    private final AtomicLong hits = new AtomicLong(0);
    private final AtomicLong misses = new AtomicLong(0);
    
    public WebServerCache(Function<K, V> dataLoader, long ttlMillis) {
        this.dataLoader = dataLoader;
        this.ttlMillis = ttlMillis;
        this.cleanupExecutor = Executors.newScheduledThreadPool(1);
        
        // Periodic cleanup of expired entries
        cleanupExecutor.scheduleAtFixedRate(this::cleanup, ttlMillis, ttlMillis, TimeUnit.MILLISECONDS);
    }
    
    public V get(K key) {
        CacheEntry<V> entry = cache.get(key);
        
        if (entry != null && !entry.isExpired()) {
            hits.incrementAndGet();
            return entry.getValue();
        }
        
        // Cache miss - load data (potentially expensive operation)
        misses.incrementAndGet();
        V value = dataLoader.apply(key);
        
        // Use computeIfAbsent for atomic insertion, but handle expiration
        cache.compute(key, (k, oldEntry) -> {
            if (oldEntry == null || oldEntry.isExpired()) {
                return new CacheEntry<>(value, System.currentTimeMillis() + ttlMillis);
            }
            return oldEntry; // Another thread already updated
        });
        
        return value;
    }
    
    public void invalidate(K key) {
        cache.remove(key);
    }
    
    public void cleanup() {
        long now = System.currentTimeMillis();
        cache.entrySet().removeIf(entry -> entry.getValue().isExpiredAt(now));
    }
    
    public CacheStats getStats() {
        long totalRequests = hits.get() + misses.get();
        double hitRatio = totalRequests == 0 ? 0.0 : (double) hits.get() / totalRequests;
        return new CacheStats(hits.get(), misses.get(), hitRatio, cache.size());
    }
    
    private static class CacheEntry<V> {
        private final V value;
        private final long expirationTime;
        
        CacheEntry(V value, long expirationTime) {
            this.value = value;
            this.expirationTime = expirationTime;
        }
        
        V getValue() { return value; }
        boolean isExpired() { return System.currentTimeMillis() > expirationTime; }
        boolean isExpiredAt(long time) { return time > expirationTime; }
    }
    
    public static class CacheStats {
        public final long hits, misses;
        public final double hitRatio;
        public final int size;
        
        CacheStats(long hits, long misses, double hitRatio, int size) {
            this.hits = hits; this.misses = misses; this.hitRatio = hitRatio; this.size = size;
        }
        
        @Override
        public String toString() {
            return String.format("Cache Stats: Hits=%d, Misses=%d, Hit Ratio=%.2f%%, Size=%d", 
                               hits, misses, hitRatio * 100, size);
        }
    }
}

// Test the cache system
class CacheTest {
    public static void runCacheScenario() {
        System.out.println("=== WEB SERVER CACHE SCENARIO ===");
        
        // Simulate expensive database lookup
        Function<String, String> dbLoader = key -> {
            try {
                Thread.sleep(100); // Simulate DB query delay
                return "Data for " + key + " at " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return "Error";
            }
        };
        
        WebServerCache<String, String> cache = new WebServerCache<>(dbLoader, 2000); // 2 second TTL
        
        // Simulate concurrent web requests
        ExecutorService executor = Executors.newFixedThreadPool(10);
        CountDownLatch latch = new CountDownLatch(20);
        
        // Submit 20 concurrent requests for same keys
        for (int i = 0; i < 20; i++) {
            final int requestId = i;
            executor.submit(() -> {
                try {
                    String key = "user_" + (requestId % 5); // 5 different users
                    String result = cache.get(key);
                    System.out.println("Request " + requestId + ": " + result);
                } finally {
                    latch.countDown();
                }
            });
        }
        
        try {
            latch.await();
            Thread.sleep(1000);
            System.out.println(cache.getStats());
            
            // Test cache expiration
            Thread.sleep(2000);
            System.out.println("After expiration:");
            cache.get("user_0"); // Should reload
            System.out.println(cache.getStats());
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
}

// ==================== SCENARIO 2: TASK PROCESSING SYSTEM ====================
/**
 * Real Problem: Microservice needs to process tasks with different priorities
 * Requirements: Multiple producers, multiple consumers, backpressure handling
 */
class TaskProcessingSystem {
    
    enum TaskType {
        URGENT(1), NORMAL(5), LOW(10);
        final int priority;
        TaskType(int priority) { this.priority = priority; }
    }
    
    static class Task implements Comparable<Task> {
        final String id;
        final TaskType type;
        final String payload;
        final long createdAt;
        
        Task(String id, TaskType type, String payload) {
            this.id = id;
            this.type = type;
            this.payload = payload;
            this.createdAt = System.currentTimeMillis();
        }
        
        @Override
        public int compareTo(Task other) {
            // Higher priority tasks first, then FIFO for same priority
            int priorityDiff = Integer.compare(this.type.priority, other.type.priority);
            return priorityDiff != 0 ? priorityDiff : Long.compare(this.createdAt, other.createdAt);
        }
        
        @Override
        public String toString() {
            return String.format("Task{id='%s', type=%s, age=%dms}", 
                               id, type, System.currentTimeMillis() - createdAt);
        }
    }
    
    private final PriorityBlockingQueue<Task> taskQueue = new PriorityBlockingQueue<>();
    private final AtomicInteger processedCount = new AtomicInteger(0);
    private final AtomicInteger rejectedCount = new AtomicInteger(0);
    private final ExecutorService workers;
    private volatile boolean running = true;
    
    // Backpressure: reject tasks if queue is too full
    private static final int MAX_QUEUE_SIZE = 1000;
    
    public TaskProcessingSystem(int workerCount) {
        this.workers = Executors.newFixedThreadPool(workerCount);
        
        // Start worker threads
        for (int i = 0; i < workerCount; i++) {
            final int workerId = i;
            workers.submit(() -> workerLoop(workerId));
        }
    }
    
    public boolean submitTask(Task task) {
        if (taskQueue.size() >= MAX_QUEUE_SIZE) {
            rejectedCount.incrementAndGet();
            System.out.println("REJECTED: " + task + " (queue full)");
            return false;
        }
        
        boolean added = taskQueue.offer(task);
        if (added) {
            System.out.println("QUEUED: " + task + " (queue size: " + taskQueue.size() + ")");
        }
        return added;
    }
    
    private void workerLoop(int workerId) {
        while (running) {
            try {
                Task task = taskQueue.poll(1, TimeUnit.SECONDS);
                if (task != null) {
                    processTask(workerId, task);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
        System.out.println("Worker " + workerId + " shutting down");
    }
    
    private void processTask(int workerId, Task task) {
        try {
            // Simulate different processing times based on task type
            int processingTime = task.type == TaskType.URGENT ? 50 : 
                               task.type == TaskType.NORMAL ? 200 : 500;
            Thread.sleep(processingTime);
            
            processedCount.incrementAndGet();
            System.out.println(String.format("Worker-%d PROCESSED: %s (processed=%d, queued=%d)", 
                                            workerId, task, processedCount.get(), taskQueue.size()));
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public void shutdown() {
        running = false;
        workers.shutdown();
        try {
            if (!workers.awaitTermination(5, TimeUnit.SECONDS)) {
                workers.shutdownNow();
            }
        } catch (InterruptedException e) {
            workers.shutdownNow();
        }
    }
    
    public String getStats() {
        return String.format("Stats: Processed=%d, Rejected=%d, Queued=%d", 
                           processedCount.get(), rejectedCount.get(), taskQueue.size());
    }
}

// ==================== SCENARIO 3: EVENT NOTIFICATION SYSTEM ====================
/**
 * Real Problem: Multiple components need to listen to system events
 * Requirements: Thread-safe listener management, concurrent notifications
 */
class EventNotificationSystem<T> {
    
    // CopyOnWriteArrayList: Perfect for read-heavy (many notifications) vs write-light (rare listener changes)
    private final CopyOnWriteArrayList<EventListener<T>> listeners = new CopyOnWriteArrayList<>();
    private final AtomicLong eventCount = new AtomicLong(0);
    private final ExecutorService notificationExecutor;
    
    interface EventListener<T> {
        void onEvent(String eventType, T data);
        default String getListenerId() { return this.getClass().getSimpleName() + "@" + hashCode(); }
    }
    
    public EventNotificationSystem() {
        // Use cached thread pool for concurrent notifications
        this.notificationExecutor = Executors.newCachedThreadPool();
    }
    
    public void addEventListener(EventListener<T> listener) {
        listeners.addIfAbsent(listener); // Avoid duplicate listeners
        System.out.println("Added listener: " + listener.getListenerId() + " (total: " + listeners.size() + ")");
    }
    
    public void removeEventListener(EventListener<T> listener) {
        boolean removed = listeners.remove(listener);
        if (removed) {
            System.out.println("Removed listener: " + listener.getListenerId() + " (total: " + listeners.size() + ")");
        }
    }
    
    public void fireEvent(String eventType, T data) {
        long eventId = eventCount.incrementAndGet();
        System.out.println("FIRING EVENT #" + eventId + ": " + eventType);
        
        // Notify all listeners concurrently
        // CopyOnWriteArrayList provides snapshot iteration - safe even if listeners are added/removed during iteration
        for (EventListener<T> listener : listeners) {
            notificationExecutor.submit(() -> {
                try {
                    listener.onEvent(eventType, data);
                } catch (Exception e) {
                    System.err.println("Error notifying listener " + listener.getListenerId() + ": " + e.getMessage());
                }
            });
        }
    }
    
    public int getListenerCount() {
        return listeners.size();
    }
    
    public long getEventCount() {
        return eventCount.get();
    }
    
    public void shutdown() {
        notificationExecutor.shutdown();
    }
}

// ==================== SCENARIO 4: RATE LIMITER ====================
/**
 * Real Problem: API needs to limit requests per user to prevent abuse
 * Requirements: Thread-safe, configurable limits, time-based windows
 */
class RateLimiter {
    
    private final ConcurrentHashMap<String, UserLimitInfo> userLimits = new ConcurrentHashMap<>();
    private final int maxRequestsPerWindow;
    private final long windowSizeMillis;
    
    public RateLimiter(int maxRequestsPerWindow, long windowSizeMillis) {
        this.maxRequestsPerWindow = maxRequestsPerWindow;
        this.windowSizeMillis = windowSizeMillis;
        
        // Cleanup old entries periodically
        ScheduledExecutorService cleanup = Executors.newScheduledThreadPool(1);
        cleanup.scheduleAtFixedRate(this::cleanupExpiredEntries, windowSizeMillis, windowSizeMillis, TimeUnit.MILLISECONDS);
    }
    
    public boolean allowRequest(String userId) {
        long now = System.currentTimeMillis();
        
        return userLimits.compute(userId, (key, existingInfo) -> {
            if (existingInfo == null || existingInfo.isExpired(now)) {
                // New window
                return new UserLimitInfo(now, 1);
            } else {
                // Same window
                existingInfo.incrementRequests();
                return existingInfo;
            }
        }).requestCount <= maxRequestsPerWindow;
    }
    
    private void cleanupExpiredEntries() {
        long now = System.currentTimeMillis();
        userLimits.entrySet().removeIf(entry -> entry.getValue().isExpired(now));
    }
    
    private class UserLimitInfo {
        private final long windowStart;
        private int requestCount;
        
        UserLimitInfo(long windowStart, int initialCount) {
            this.windowStart = windowStart;
            this.requestCount = initialCount;
        }
        
        boolean isExpired(long now) {
            return now - windowStart >= windowSizeMillis;
        }
        
        void incrementRequests() {
            requestCount++;
        }
    }
    
    public String getStats() {
        return "Active users being rate limited: " + userLimits.size();
    }
}

// ==================== SCENARIO 5: DELAYED JOB SCHEDULER ====================
/**
 * Real Problem: System needs to execute jobs at specific future times
 * Requirements: Precise timing, persistent across restarts, thread-safe
 */
class DelayedJobScheduler {
    
    static class DelayedJob implements Delayed {
        final String jobId;
        final Runnable task;
        final long executeAt;
        final String description;
        
        DelayedJob(String jobId, Runnable task, long delayMillis, String description) {
            this.jobId = jobId;
            this.task = task;
            this.executeAt = System.currentTimeMillis() + delayMillis;
            this.description = description;
        }
        
        @Override
        public long getDelay(TimeUnit unit) {
            long remaining = executeAt - System.currentTimeMillis();
            return unit.convert(remaining, TimeUnit.MILLISECONDS);
        }
        
        @Override
        public int compareTo(Delayed other) {
            DelayedJob otherJob = (DelayedJob) other;
            return Long.compare(this.executeAt, otherJob.executeAt);
        }
        
        @Override
        public String toString() {
            return String.format("DelayedJob{id='%s', desc='%s', executeIn=%dms}", 
                               jobId, description, getDelay(TimeUnit.MILLISECONDS));
        }
    }
    
    private final DelayQueue<DelayedJob> jobQueue = new DelayQueue<>();
    private final ExecutorService jobExecutor;
    private final AtomicInteger completedJobs = new AtomicInteger(0);
    private volatile boolean running = true;
    
    public DelayedJobScheduler(int executorThreads) {
        this.jobExecutor = Executors.newFixedThreadPool(executorThreads);
        
        // Start the scheduler thread
        Thread schedulerThread = new Thread(this::schedulerLoop);
        schedulerThread.setDaemon(true);
        schedulerThread.start();
    }
    
    public void scheduleJob(String jobId, Runnable task, long delayMillis, String description) {
        DelayedJob job = new DelayedJob(jobId, task, delayMillis, description);
        jobQueue.offer(job);
        System.out.println("SCHEDULED: " + job + " (queue size: " + jobQueue.size() + ")");
    }
    
    private void schedulerLoop() {
        while (running) {
            try {
                // Take blocks until a job is ready to execute
                DelayedJob job = jobQueue.take();
                
                // Execute job in thread pool
                jobExecutor.submit(() -> {
                    try {
                        System.out.println("EXECUTING: " + job);
                        job.task.run();
                        completedJobs.incrementAndGet();
                        System.out.println("COMPLETED: " + job.jobId + " (total completed: " + completedJobs.get() + ")");
                    } catch (Exception e) {
                        System.err.println("ERROR executing job " + job.jobId + ": " + e.getMessage());
                    }
                });
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
        System.out.println("Scheduler loop terminated");
    }
    
    public int getPendingJobCount() {
        return jobQueue.size();
    }
    
    public int getCompletedJobCount() {
        return completedJobs.get();
    }
    
    public void shutdown() {
        running = false;
        jobExecutor.shutdown();
    }
}

// ==================== MAIN DEMONSTRATION CLASS ====================
public class Phase5RealWorldScenarios {
    
    public static void main(String[] args) {
        try {
            // Run all scenarios
            runWebServerCacheScenario();
            Thread.sleep(1000);
            
            runTaskProcessingScenario();
            Thread.sleep(1000);
            
            runEventNotificationScenario();
            Thread.sleep(1000);
            
            runRateLimiterScenario();
            Thread.sleep(1000);
            
            runDelayedJobScenario();
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    private static void runWebServerCacheScenario() {
        CacheTest.runCacheScenario();
        System.out.println();
    }
    
    private static void runTaskProcessingScenario() {
        System.out.println("=== TASK PROCESSING SYSTEM SCENARIO ===");
        
        TaskProcessingSystem processor = new TaskProcessingSystem(3);
        
        // Simulate task submissions from multiple threads
        ExecutorService producers = Executors.newFixedThreadPool(5);
        
        for (int i = 0; i < 20; i++) {
            final int taskId = i;
            producers.submit(() -> {
                TaskProcessingSystem.TaskType type = 
                    taskId < 5 ? TaskProcessingSystem.TaskType.URGENT :
                    taskId < 15 ? TaskProcessingSystem.TaskType.NORMAL :
                    TaskProcessingSystem.TaskType.LOW;
                
                TaskProcessingSystem.Task task = new TaskProcessingSystem.Task(
                    "TASK-" + taskId, type, "Payload for task " + taskId
                );
                
                processor.submitTask(task);
            });
        }
        
        try {
            Thread.sleep(5000); // Let tasks process
            System.out.println(processor.getStats());
            processor.shutdown();
            producers.shutdown();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println();
    }
    
    private static void runEventNotificationScenario() {
        System.out.println("=== EVENT NOTIFICATION SYSTEM SCENARIO ===");
        
        EventNotificationSystem<String> eventSystem = new EventNotificationSystem<>();
        
        // Add various listeners
        eventSystem.addEventListener((type, data) -> 
            System.out.println("  [EmailService] Processing " + type + ": " + data));
        
        eventSystem.addEventListener((type, data) -> {
            try {
                Thread.sleep(100); // Simulate slow processing
                System.out.println("  [SlowAnalytics] Analyzed " + type + ": " + data);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        eventSystem.addEventListener((type, data) -> 
            System.out.println("  [Logger] Logged " + type + ": " + data));
        
        // Fire events concurrently
        ExecutorService eventFirers = Executors.newFixedThreadPool(3);
        
        for (int i = 0; i < 10; i++) {
            final int eventId = i;
            eventFirers.submit(() -> {
                eventSystem.fireEvent("USER_ACTION", "User performed action " + eventId);
            });
        }
        
        try {
            Thread.sleep(2000);
            System.out.println("Events fired: " + eventSystem.getEventCount());
            eventSystem.shutdown();
            eventFirers.shutdown();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println();
    }
    
    private static void runRateLimiterScenario() {
        System.out.println("=== RATE LIMITER SCENARIO ===");
        
        RateLimiter rateLimiter = new RateLimiter(5, 2000); // 5 requests per 2 seconds
        
        ExecutorService requesters = Executors.newFixedThreadPool(10);
        
        // Simulate API requests from different users
        for (int i = 0; i < 30; i++) {
            final int requestId = i;
            requesters.submit(() -> {
                String userId = "user" + (requestId % 3); // 3 different users
                boolean allowed = rateLimiter.allowRequest(userId);
                
                System.out.println(String.format("Request %d from %s: %s", 
                                                requestId, userId, allowed ? "ALLOWED" : "RATE LIMITED"));
                
                try {
                    Thread.sleep(100); // Space out requests
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        try {
            Thread.sleep(3000);
            System.out.println(rateLimiter.getStats());
            requesters.shutdown();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println();
    }
    
    private static void runDelayedJobScenario() {
        System.out.println("=== DELAYED JOB SCHEDULER SCENARIO ===");
        
        DelayedJobScheduler scheduler = new DelayedJobScheduler(2);
        
        // Schedule jobs with different delays
        scheduler.scheduleJob("job1", 
            () -> System.out.println("    >>> Executing immediate job"), 
            100, "Immediate job");
            
        scheduler.scheduleJob("job2", 
            () -> System.out.println("    >>> Sending email notification"), 
            1000, "Email notification");
            
        scheduler.scheduleJob("job3", 
            () -> System.out.println("    >>> Generating daily report"), 
            2000, "Daily report");
            
        scheduler.scheduleJob("job4", 
            () -> System.out.println("    >>> Cleaning up temp files"), 
            1500, "Cleanup task");
            
        scheduler.scheduleJob("job5", 
            () -> System.out.println("    >>> Backup database"), 
            3000, "Database backup");
        
        try {
            Thread.sleep(4000); // Wait for all jobs to complete
            System.out.println("Final stats - Pending: " + scheduler.getPendingJobCount() + 
                             ", Completed: " + scheduler.getCompletedJobCount());
            scheduler.shutdown();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println();
    }
}