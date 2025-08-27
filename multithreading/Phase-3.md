// =============================================================================
// PHASE 3: ADVANCED SYNCHRONIZATION - COMPREHENSIVE EXAMPLES
// =============================================================================

import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

// =============================================================================
// 7. OBJECT-LEVEL SYNCHRONIZATION METHODS (wait, notify, notifyAll)
// =============================================================================

/**
 * Complete demonstration of wait(), notify(), and notifyAll() methods
 * These methods MUST be called from within synchronized blocks/methods
 */
class WaitNotifyDemo {
    private final Object lock = new Object();
    private boolean dataReady = false;
    private String sharedData = null;
    
    /**
     * Demonstrates wait() - thread releases lock and waits for notification
     */
    public void waitForData() {
        synchronized (lock) {
            while (!dataReady) { // Always use while loop, not if!
                try {
                    System.out.println(Thread.currentThread().getName() + " waiting for data...");
                    lock.wait(); // Releases lock and waits
                    System.out.println(Thread.currentThread().getName() + " woke up!");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
            }
            
            // Data is ready, process it
            System.out.println(Thread.currentThread().getName() + " processing data: " + sharedData);
            dataReady = false; // Reset for next cycle
        }
    }
    
    /**
     * Demonstrates notify() - wakes up one waiting thread
     */
    public void produceData(String data) {
        synchronized (lock) {
            System.out.println("Producer creating data: " + data);
            sharedData = data;
            dataReady = true;
            
            lock.notify(); // Wake up ONE waiting thread
            System.out.println("Producer notified one waiting thread");
        }
    }
    
    /**
     * Demonstrates notifyAll() - wakes up all waiting threads
     */
    public void broadcastData(String data) {
        synchronized (lock) {
            System.out.println("Broadcaster creating data: " + data);
            sharedData = data;
            dataReady = true;
            
            lock.notifyAll(); // Wake up ALL waiting threads
            System.out.println("Broadcaster notified all waiting threads");
        }
    }
    
    public static void demonstrateWaitNotify() {
        System.out.println("=== WAIT/NOTIFY DEMONSTRATION ===");
        
        WaitNotifyDemo demo = new WaitNotifyDemo();
        
        // Create consumer threads
        Thread consumer1 = new Thread(() -> demo.waitForData(), "Consumer-1");
        Thread consumer2 = new Thread(() -> demo.waitForData(), "Consumer-2");
        Thread consumer3 = new Thread(() -> demo.waitForData(), "Consumer-3");
        
        // Start consumers first (they will wait)
        consumer1.start();
        consumer2.start();
        consumer3.start();
        
        try {
            Thread.sleep(1000); // Let consumers start waiting
            
            // Producer notifies one
            System.out.println("\n--- Using notify() ---");
            demo.produceData("First Data");
            Thread.sleep(1000);
            
            // Broadcaster notifies all remaining
            System.out.println("\n--- Using notifyAll() ---");
            demo.broadcastData("Broadcast Data");
            
            // Wait for all consumers to complete
            consumer1.join();
            consumer2.join();
            consumer3.join();
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Wait/Notify demonstration completed\n");
    }
}

/**
 * Classic Producer-Consumer Pattern Implementation
 * Demonstrates proper use of wait/notify for coordination
 */
class ProducerConsumerDemo {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int MAX_SIZE = 5;
    private final Object lock = new Object();
    
    /**
     * Producer adds items to queue
     */
    public void produce() {
        int item = 1;
        
        try {
            while (item <= 20) { // Produce 20 items
                synchronized (lock) {
                    // Wait if queue is full
                    while (queue.size() >= MAX_SIZE) {
                        System.out.println("Queue full, producer waiting...");
                        lock.wait();
                    }
                    
                    // Produce item
                    queue.offer(item);
                    System.out.println("🏭 Produced: " + item + " | Queue size: " + queue.size());
                    
                    // Notify consumers
                    lock.notifyAll();
                    
                    item++;
                }
                
                Thread.sleep(200); // Production delay
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("🏭 Producer finished");
    }
    
    /**
     * Consumer removes items from queue
     */
    public void consume(String consumerName) {
        try {
            while (true) {
                synchronized (lock) {
                    // Wait if queue is empty
                    while (queue.isEmpty()) {
                        System.out.println(consumerName + " waiting for items...");
                        lock.wait(5000); // Wait with timeout
                        
                        // Check if we should exit (producer might be done)
                        if (queue.isEmpty()) {
                            System.out.println(consumerName + " timed out, exiting");
                            return;
                        }
                    }
                    
                    // Consume item
                    Integer item = queue.poll();
                    System.out.println("🛒 " + consumerName + " consumed: " + item + " | Queue size: " + queue.size());
                    
                    // Notify producers
                    lock.notifyAll();
                }
                
                Thread.sleep(300); // Consumption delay
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("🛒 " + consumerName + " finished");
    }
    
    public static void demonstrateProducerConsumer() {
        System.out.println("=== PRODUCER-CONSUMER PATTERN ===");
        
        ProducerConsumerDemo demo = new ProducerConsumerDemo();
        
        // Create producer thread
        Thread producer = new Thread(() -> demo.produce(), "Producer");
        
        // Create consumer threads
        Thread consumer1 = new Thread(() -> demo.consume("Consumer-1"), "Consumer-1");
        Thread consumer2 = new Thread(() -> demo.consume("Consumer-2"), "Consumer-2");
        
        // Start all threads
        producer.start();
        consumer1.start();
        consumer2.start();
        
        try {
            producer.join();
            consumer1.join();
            consumer2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Producer-Consumer demonstration completed\n");
    }
}

// =============================================================================
// 8. VOLATILE KEYWORD - MEMORY VISIBILITY
// =============================================================================

/**
 * Demonstrates the volatile keyword and memory visibility issues
 */
class VolatileDemo {
    // Without volatile - may cause visibility issues
    private static boolean running = true;
    private static int counter = 0;
    
    // With volatile - guarantees visibility
    private static volatile boolean volatileRunning = true;
    private static volatile int volatileCounter = 0;
    
    /**
     * Demonstrates memory visibility problem without volatile
     */
    public static void demonstrateVisibilityProblem() {
        System.out.println("=== MEMORY VISIBILITY PROBLEM (without volatile) ===");
        
        Thread worker = new Thread(() -> {
            System.out.println("Worker thread started, running variable: " + running);
            
            while (running) { // May never see the update!
                counter++;
                // Uncomment next line to "fix" the problem (adds memory barrier)
                // System.out.println("Working... " + counter);
            }
            
            System.out.println("Worker thread stopped. Counter: " + counter);
        }, "Worker");
        
        worker.start();
        
        try {
            Thread.sleep(1000); // Let worker run
            
            System.out.println("Main thread setting running = false");
            running = false; // May not be visible to worker thread!
            
            // Wait a bit to see if worker stops
            worker.join(2000);
            
            if (worker.isAlive()) {
                System.out.println("⚠️ Worker thread didn't stop - visibility problem!");
                worker.interrupt(); // Force stop
            }
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Visibility problem demonstration completed\n");
    }
    
    /**
     * Demonstrates proper use of volatile keyword
     */
    public static void demonstrateVolatileSolution() {
        System.out.println("=== VOLATILE SOLUTION ===");
        
        volatileRunning = true; // Reset
        volatileCounter = 0;
        
        Thread volatileWorker = new Thread(() -> {
            System.out.println("Volatile worker started, volatileRunning: " + volatileRunning);
            
            while (volatileRunning) { // Will see updates immediately
                volatileCounter++;
            }
            
            System.out.println("Volatile worker stopped. Counter: " + volatileCounter);
        }, "VolatileWorker");
        
        volatileWorker.start();
        
        try {
            Thread.sleep(1000);
            
            System.out.println("Main thread setting volatileRunning = false");
            volatileRunning = false; // Visible immediately to worker
            
            volatileWorker.join(1000);
            
            if (volatileWorker.isAlive()) {
                System.out.println("❌ Something went wrong with volatile");
            } else {
                System.out.println("✅ Volatile worker stopped correctly");
            }
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Volatile solution demonstration completed\n");
    }
    
    /**
     * Double-Checked Locking Anti-Pattern (BROKEN without volatile)
     */
    private static ExpensiveResource instance;
    
    // BROKEN - without volatile
    public static ExpensiveResource getBrokenSingleton() {
        if (instance == null) { // First check (not synchronized)
            synchronized (VolatileDemo.class) {
                if (instance == null) { // Second check (synchronized)
                    instance = new ExpensiveResource(); // Problem: partial construction visible
                }
            }
        }
        return instance;
    }
    
    // CORRECT - with volatile
    private static volatile ExpensiveResource volatileInstance;
    
    public static ExpensiveResource getCorrectSingleton() {
        if (volatileInstance == null) { // First check (not synchronized)
            synchronized (VolatileDemo.class) {
                if (volatileInstance == null) { // Second check (synchronized)
                    volatileInstance = new ExpensiveResource(); // volatile prevents reordering
                }
            }
        }
        return volatileInstance;
    }
    
    static class ExpensiveResource {
        private final String data;
        
        public ExpensiveResource() {
            // Simulate expensive construction
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            this.data = "Resource created at " + System.currentTimeMillis();
        }
        
        public String getData() { return data; }
    }
    
    public static void demonstrateDoubleCheckedLocking() {
        System.out.println("=== DOUBLE-CHECKED LOCKING PATTERN ===");
        
        // Test the correct volatile version
        Thread[] threads = new Thread[5];
        
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                ExpensiveResource resource = getCorrectSingleton();
                System.out.println(Thread.currentThread().getName() + " got resource: " + resource.getData());
            }, "Thread-" + (i + 1));
        }
        
        // Start all threads simultaneously
        for (Thread thread : threads) {
            thread.start();
        }
        
        try {
            for (Thread thread : threads) {
                thread.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Double-checked locking demonstration completed\n");
    }
}

// =============================================================================
// 9. THREADLOCAL CLASS - THREAD-SPECIFIC DATA
// =============================================================================

/**
 * Demonstrates ThreadLocal for thread-specific data storage
 */
class ThreadLocalDemo {
    
    // ThreadLocal variable - each thread has its own copy
    private static final ThreadLocal<Integer> threadLocalCounter = new ThreadLocal<Integer>() {
        @Override
        protected Integer initialValue() {
            return 0; // Initial value for each thread
        }
    };
    
    // Alternative modern syntax (Java 8+)
    private static final ThreadLocal<String> threadLocalName = ThreadLocal.withInitial(() -> "Default");
    
    // Regular static variable for comparison
    private static int regularCounter = 0;
    
    /**
     * Demonstrates thread-specific storage
     */
    public static void demonstrateThreadLocalStorage() {
        System.out.println("=== THREADLOCAL STORAGE DEMONSTRATION ===");
        
        Thread[] workers = new Thread[3];
        
        for (int i = 0; i < workers.length; i++) {
            final int threadId = i + 1;
            
            workers[i] = new Thread(() -> {
                // Set thread-specific name
                threadLocalName.set("Worker-" + threadId);
                
                // Each thread works with its own counter
                for (int j = 1; j <= 5; j++) {
                    // Increment thread-local counter
                    int currentCount = threadLocalCounter.get();
                    threadLocalCounter.set(currentCount + 1);
                    
                    // Also increment regular counter (shared)
                    synchronized (ThreadLocalDemo.class) {
                        regularCounter++;
                    }
                    
                    System.out.println(threadLocalName.get() + 
                                     " - ThreadLocal counter: " + threadLocalCounter.get() + 
                                     " | Regular counter: " + regularCounter);
                    
                    try { Thread.sleep(100); } catch (InterruptedException e) { 
                        Thread.currentThread().interrupt(); 
                        return; 
                    }
                }
                
                System.out.println(threadLocalName.get() + " finished with ThreadLocal counter: " + threadLocalCounter.get());
                
            }, "Worker-" + threadId);
        }
        
        // Start all workers
        for (Thread worker : workers) {
            worker.start();
        }
        
        try {
            for (Thread worker : workers) {
                worker.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Final regular counter: " + regularCounter);
        System.out.println("ThreadLocal demonstration completed\n");
    }
    
    /**
     * Practical example: Database Connection per Thread
     */
    static class DatabaseConnection {
        private final String connectionId;
        
        public DatabaseConnection() {
            this.connectionId = "Connection-" + System.currentTimeMillis() + "-" + 
                              Thread.currentThread().getName();
            System.out.println("📦 Created database connection: " + connectionId);
        }
        
        public void executeQuery(String query) {
            System.out.println("🔍 [" + connectionId + "] Executing: " + query);
        }
        
        public void close() {
            System.out.println("🔒 Closed connection: " + connectionId);
        }
        
        public String getConnectionId() {
            return connectionId;
        }
    }
    
    // ThreadLocal database connection
    private static final ThreadLocal<DatabaseConnection> connectionThreadLocal = 
        ThreadLocal.withInitial(() -> new DatabaseConnection());
    
    public static void demonstrateThreadLocalConnections() {
        System.out.println("=== THREADLOCAL DATABASE CONNECTIONS ===");
        
        Thread[] dbWorkers = new Thread[3];
        
        for (int i = 0; i < dbWorkers.length; i++) {
            final int workerId = i + 1;
            
            dbWorkers[i] = new Thread(() -> {
                try {
                    // Get thread-specific database connection
                    DatabaseConnection conn = connectionThreadLocal.get();
                    
                    // Perform multiple database operations
                    for (int j = 1; j <= 3; j++) {
                        conn.executeQuery("SELECT * FROM table" + j + " WHERE worker_id = " + workerId);
                        Thread.sleep(200);
                    }
                    
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    // Clean up thread-local resource
                    DatabaseConnection conn = connectionThreadLocal.get();
                    if (conn != null) {
                        conn.close();
                        connectionThreadLocal.remove(); // Important: prevent memory leaks!
                    }
                }
                
            }, "DBWorker-" + workerId);
        }
        
        // Start all database workers
        for (Thread worker : dbWorkers) {
            worker.start();
        }
        
        try {
            for (Thread worker : dbWorkers) {
                worker.join();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("ThreadLocal database connections demonstration completed\n");
    }
    
    /**
     * InheritableThreadLocal demonstration
     */
    private static final InheritableThreadLocal<String> inheritableContext = 
        new InheritableThreadLocal<String>() {
            @Override
            protected String initialValue() {
                return "DefaultContext";
            }
        };
    
    public static void demonstrateInheritableThreadLocal() {
        System.out.println("=== INHERITABLE THREADLOCAL DEMONSTRATION ===");
        
        // Set context in main thread
        inheritableContext.set("MainThreadContext");
        System.out.println("Main thread context: " + inheritableContext.get());
        
        // Create child thread - inherits parent's ThreadLocal value
        Thread childThread = new Thread(() -> {
            System.out.println("Child thread inherited context: " + inheritableContext.get());
            
            // Child can modify its own copy
            inheritableContext.set("ChildThreadContext");
            System.out.println("Child thread modified context: " + inheritableContext.get());
            
            // Create grandchild thread
            Thread grandChildThread = new Thread(() -> {
                System.out.println("Grandchild inherited context: " + inheritableContext.get());
            }, "GrandChild");
            
            grandChildThread.start();
            try { grandChildThread.join(); } catch (InterruptedException e) { }
            
        }, "Child");
        
        childThread.start();
        try { childThread.join(); } catch (InterruptedException e) { }
        
        // Parent's context unchanged
        System.out.println("Main thread context after child: " + inheritableContext.get());
        
        System.out.println("InheritableThreadLocal demonstration completed\n");
    }
}

// =============================================================================
// COMPREHENSIVE DEMONSTRATION CLASS
// =============================================================================

public class Phase3AdvancedSynchronization {
    
    public static void main(String[] args) {
        System.out.println("=====================================");
        System.out.println("PHASE 3: ADVANCED SYNCHRONIZATION");
        System.out.println("=====================================\n");
        
        try {
            // Object-level synchronization methods
            WaitNotifyDemo.demonstrateWaitNotify();
            waitForUser();
            
            ProducerConsumerDemo.demonstrateProducerConsumer();
            waitForUser();
            
            // Volatile keyword
            VolatileDemo.demonstrateVisibilityProblem();
            waitForUser();
            
            VolatileDemo.demonstrateVolatileSolution();
            waitForUser();
            
            VolatileDemo.demonstrateDoubleCheckedLocking();
            waitForUser();
            
            // ThreadLocal
            ThreadLocalDemo.demonstrateThreadLocalStorage();
            waitForUser();
            
            ThreadLocalDemo.demonstrateThreadLocalConnections();
            waitForUser();
            
            ThreadLocalDemo.demonstrateInheritableThreadLocal();
            
            printPhase3Summary();
            
        } catch (Exception e) {
            System.err.println("Error in Phase 3 demonstration: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    private static void waitForUser() {
        System.out.println("Press Enter to continue...");
        try { System.in.read(); } catch (Exception e) { }
    }
    
    private static void printPhase3Summary() {
        System.out.println("\n=====================================");
        System.out.println("PHASE 3 SUMMARY");
        System.out.println("=====================================");
        System.out.println("✓ Object-Level Synchronization:");
        System.out.println("  • wait() - Release lock and wait for notification");
        System.out.println("  • notify() - Wake up one waiting thread");
        System.out.println("  • notifyAll() - Wake up all waiting threads");
        System.out.println("  • Producer-Consumer pattern implementation");
        System.out.println();
        System.out.println("✓ Volatile Keyword:");
        System.out.println("  • Guarantees memory visibility");
        System.out.println("  • Prevents instruction reordering");
        System.out.println("  • Essential for double-checked locking");
        System.out.println("  • NOT a replacement for synchronization");
        System.out.println();
        System.out.println("✓ ThreadLocal:");
        System.out.println("  • Thread-specific data storage");
        System.out.println("  • Eliminates need for synchronization");
        System.out.println("  • Remember to call remove() to prevent memory leaks");
        System.out.println("  • InheritableThreadLocal for parent-child inheritance");
        System.out.println();
        System.out.println("KEY RULES:");
        System.out.println("• Always use while loops with wait(), not if");
        System.out.println("• Call wait/notify only within synchronized blocks");
        System.out.println("• Use volatile for visibility, synchronized for atomicity");
        System.out.println("• Clean up ThreadLocal variables to prevent leaks");
        System.out.println();
        System.out.println("NEXT: Phase 4 - Java Concurrency Utilities");
        System.out.println("(ExecutorService, ReentrantLock, Atomic classes)");
    }
}