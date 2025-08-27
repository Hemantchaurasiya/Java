// =============================================================================
// PHASE 2: CORE THREADING MECHANISMS - COMPREHENSIVE EXAMPLES
// =============================================================================

import java.util.ArrayList;
import java.util.List;
import java.util.Random;

// =============================================================================
// 4. THREAD CLASS METHODS - COMPLETE DEMONSTRATION
// =============================================================================

/**
 * Comprehensive demonstration of all essential Thread methods
 * Learning objectives:
 * - Understand start() vs run()
 * - Master sleep(), join(), interrupt()
 * - Learn yield(), daemon threads, priorities
 */
class ThreadMethodsDemo {
    
    /**
     * Demonstrates the critical difference between start() and run()
     */
    public static void demonstrateStartVsRun() {
        System.out.println("=== START() vs RUN() DEMONSTRATION ===");
        
        Thread thread1 = new Thread(() -> {
            System.out.println("Thread1 executing on: " + Thread.currentThread().getName());
            System.out.println("Thread1 ID: " + Thread.currentThread().getId());
        }, "MyCustomThread1");
        
        Thread thread2 = new Thread(() -> {
            System.out.println("Thread2 executing on: " + Thread.currentThread().getName());
            System.out.println("Thread2 ID: " + Thread.currentThread().getId());
        }, "MyCustomThread2");
        
        System.out.println("Current thread: " + Thread.currentThread().getName());
        
        // Using start() - Creates new thread
        System.out.println("\nCalling start() - creates new thread:");
        thread1.start();
        
        try { Thread.sleep(100); } catch (InterruptedException e) { }
        
        // Using run() - Executes in current thread
        System.out.println("\nCalling run() directly - executes in current thread:");
        thread2.run();
        
        try { thread1.join(); } catch (InterruptedException e) { }
        System.out.println("Start vs Run demonstration completed\n");
    }
    
    /**
     * Demonstrates Thread.sleep() behavior and InterruptedException
     */
    public static void demonstrateSleep() {
        System.out.println("=== THREAD.SLEEP() DEMONSTRATION ===");
        
        Thread sleepyThread = new Thread(() -> {
            try {
                System.out.println("Thread going to sleep for 3 seconds...");
                long startTime = System.currentTimeMillis();
                
                Thread.sleep(3000); // Sleep for 3 seconds
                
                long endTime = System.currentTimeMillis();
                System.out.println("Thread woke up after: " + (endTime - startTime) + "ms");
                
            } catch (InterruptedException e) {
                System.out.println("Thread was interrupted during sleep!");
                System.out.println("Interrupt status: " + Thread.currentThread().isInterrupted());
                Thread.currentThread().interrupt(); // Restore interrupt status
            }
        }, "SleepyThread");
        
        sleepyThread.start();
        
        // Let it sleep for a moment, then interrupt
        try {
            Thread.sleep(1500);
            System.out.println("Interrupting the sleeping thread...");
            sleepyThread.interrupt();
            sleepyThread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Sleep demonstration completed\n");
    }
    
    /**
     * Demonstrates join() method and coordination between threads
     */
    public static void demonstrateJoin() {
        System.out.println("=== THREAD.JOIN() DEMONSTRATION ===");
        
        Thread task1 = new Thread(() -> {
            System.out.println("Task 1 starting...");
            try {
                Thread.sleep(2000);
                System.out.println("Task 1 completed!");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Task1");
        
        Thread task2 = new Thread(() -> {
            System.out.println("Task 2 starting...");
            try {
                Thread.sleep(1500);
                System.out.println("Task 2 completed!");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Task2");
        
        Thread task3 = new Thread(() -> {
            System.out.println("Task 3 starting...");
            try {
                Thread.sleep(1000);
                System.out.println("Task 3 completed!");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Task3");
        
        long startTime = System.currentTimeMillis();
        
        // Start all tasks
        task1.start();
        task2.start();
        task3.start();
        
        try {
            // Wait for all tasks to complete
            System.out.println("Main thread waiting for all tasks to complete...");
            task1.join();
            task2.join();
            task3.join();
            
            long endTime = System.currentTimeMillis();
            System.out.println("All tasks completed in: " + (endTime - startTime) + "ms");
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Join demonstration completed\n");
    }
    
    /**
     * Demonstrates join() with timeout
     */
    public static void demonstrateJoinWithTimeout() {
        System.out.println("=== JOIN() WITH TIMEOUT DEMONSTRATION ===");
        
        Thread longRunningTask = new Thread(() -> {
            try {
                System.out.println("Long running task started...");
                Thread.sleep(5000); // 5 seconds
                System.out.println("Long running task completed!");
            } catch (InterruptedException e) {
                System.out.println("Long running task was interrupted!");
                Thread.currentThread().interrupt();
            }
        }, "LongTask");
        
        longRunningTask.start();
        
        try {
            System.out.println("Main thread will wait maximum 2 seconds...");
            boolean completed = longRunningTask.join(2000); // Wait max 2 seconds
            
            if (longRunningTask.isAlive()) {
                System.out.println("Task is still running after timeout!");
                System.out.println("Interrupting the long running task...");
                longRunningTask.interrupt();
            } else {
                System.out.println("Task completed within timeout!");
            }
            
            longRunningTask.join(); // Wait for actual completion
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Join with timeout demonstration completed\n");
    }
    
    /**
     * Demonstrates interrupt mechanism
     */
    public static void demonstrateInterrupt() {
        System.out.println("=== INTERRUPT MECHANISM DEMONSTRATION ===");
        
        Thread interruptibleTask = new Thread(() -> {
            System.out.println("Interruptible task started...");
            
            try {
                for (int i = 1; i <= 10; i++) {
                    // Check for interrupt
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println("Thread was interrupted at iteration: " + i);
                        return;
                    }
                    
                    System.out.println("Working... iteration " + i);
                    Thread.sleep(500);
                }
                System.out.println("Task completed normally");
                
            } catch (InterruptedException e) {
                System.out.println("Thread interrupted during sleep!");
                Thread.currentThread().interrupt(); // Restore interrupt status
            }
        }, "InterruptibleTask");
        
        interruptibleTask.start();
        
        try {
            Thread.sleep(2500); // Let it run for 2.5 seconds
            System.out.println("Main thread interrupting the task...");
            interruptibleTask.interrupt();
            
            interruptibleTask.join();
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Interrupt demonstration completed\n");
    }
    
    /**
     * Demonstrates yield() method
     */
    public static void demonstrateYield() {
        System.out.println("=== THREAD.YIELD() DEMONSTRATION ===");
        
        Thread politeThread = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println("Polite thread - iteration " + i);
                Thread.yield(); // Hint to scheduler to give other threads a chance
                
                // Small delay to make the effect visible
                try { Thread.sleep(100); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }, "PoliteThread");
        
        Thread greedyThread = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println("Greedy thread - iteration " + i);
                // No yield - keeps running
                
                try { Thread.sleep(100); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }, "GreedyThread");
        
        politeThread.start();
        greedyThread.start();
        
        try {
            politeThread.join();
            greedyThread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Yield demonstration completed\n");
    }
    
    /**
     * Demonstrates daemon threads
     */
    public static void demonstrateDaemonThreads() {
        System.out.println("=== DAEMON THREADS DEMONSTRATION ===");
        
        // Normal user thread
        Thread userThread = new Thread(() -> {
            try {
                for (int i = 1; i <= 3; i++) {
                    System.out.println("User thread - iteration " + i);
                    Thread.sleep(1000);
                }
                System.out.println("User thread completed");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "UserThread");
        
        // Daemon thread
        Thread daemonThread = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    System.out.println("Daemon thread - iteration " + i);
                    Thread.sleep(500);
                }
                System.out.println("Daemon thread completed (this might not print!)");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "DaemonThread");
        
        daemonThread.setDaemon(true); // Mark as daemon thread
        
        System.out.println("UserThread daemon status: " + userThread.isDaemon());
        System.out.println("DaemonThread daemon status: " + daemonThread.isDaemon());
        
        userThread.start();
        daemonThread.start();
        
        try {
            userThread.join();
            System.out.println("User thread finished - JVM may exit now, stopping daemon thread");
            Thread.sleep(500); // Brief pause to see if daemon continues
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Daemon thread demonstration completed\n");
    }
    
    /**
     * Demonstrates thread priorities
     */
    public static void demonstratePriorities() {
        System.out.println("=== THREAD PRIORITIES DEMONSTRATION ===");
        
        Thread lowPriority = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println("LOW priority thread - " + i);
                try { Thread.sleep(100); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }, "LowPriorityThread");
        
        Thread normalPriority = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println("NORMAL priority thread - " + i);
                try { Thread.sleep(100); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }, "NormalPriorityThread");
        
        Thread highPriority = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println("HIGH priority thread - " + i);
                try { Thread.sleep(100); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }, "HighPriorityThread");
        
        // Set priorities
        lowPriority.setPriority(Thread.MIN_PRIORITY);    // 1
        normalPriority.setPriority(Thread.NORM_PRIORITY); // 5
        highPriority.setPriority(Thread.MAX_PRIORITY);   // 10
        
        System.out.println("Priority ranges: MIN=" + Thread.MIN_PRIORITY + 
                          ", NORM=" + Thread.NORM_PRIORITY + 
                          ", MAX=" + Thread.MAX_PRIORITY);
        
        System.out.println("LowPriority: " + lowPriority.getPriority());
        System.out.println("NormalPriority: " + normalPriority.getPriority());
        System.out.println("HighPriority: " + highPriority.getPriority());
        
        // Start threads
        lowPriority.start();
        normalPriority.start();
        highPriority.start();
        
        try {
            lowPriority.join();
            normalPriority.join();
            highPriority.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Note: Priority is just a hint to the scheduler");
        System.out.println("Priorities demonstration completed\n");
    }
}

// =============================================================================
// 5. SYNCHRONIZATION FUNDAMENTALS
// =============================================================================

/**
 * Demonstrates race conditions and why synchronization is needed
 */
class RaceConditionDemo {
    private static int counter = 0;
    private static final int ITERATIONS = 100000;
    
    public static void demonstrateRaceCondition() {
        System.out.println("=== RACE CONDITION DEMONSTRATION ===");
        System.out.println("Two threads will increment a counter " + ITERATIONS + " times each");
        System.out.println("Expected final value: " + (ITERATIONS * 2));
        
        counter = 0; // Reset counter
        
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < ITERATIONS; i++) {
                counter++; // Race condition here!
            }
        }, "Thread1");
        
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < ITERATIONS; i++) {
                counter++; // Race condition here!
            }
        }, "Thread2");
        
        long startTime = System.currentTimeMillis();
        
        thread1.start();
        thread2.start();
        
        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        long endTime = System.currentTimeMillis();
        
        System.out.println("Actual final value: " + counter);
        System.out.println("Time taken: " + (endTime - startTime) + "ms");
        System.out.println("Lost increments: " + ((ITERATIONS * 2) - counter));
        System.out.println("This demonstrates the race condition problem!\n");
    }
}

/**
 * Critical Section demonstration
 */
class CriticalSectionDemo {
    private static int sharedResource = 0;
    private static final Object lock = new Object();
    
    public static void demonstrateCriticalSection() {
        System.out.println("=== CRITICAL SECTION DEMONSTRATION ===");
        
        // Thread that modifies shared resource without protection
        Thread unsafeThread = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                // CRITICAL SECTION - accessing shared resource
                int temp = sharedResource;
                System.out.println("Unsafe thread read: " + temp);
                
                try { Thread.sleep(100); } catch (InterruptedException e) { return; }
                
                temp = temp + 1;
                sharedResource = temp;
                System.out.println("Unsafe thread wrote: " + temp);
            }
        }, "UnsafeThread");
        
        // Thread that modifies shared resource with protection
        Thread safeThread = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                synchronized (lock) { // PROTECTED CRITICAL SECTION
                    int temp = sharedResource;
                    System.out.println("Safe thread read: " + temp);
                    
                    try { Thread.sleep(100); } catch (InterruptedException e) { return; }
                    
                    temp = temp + 1;
                    sharedResource = temp;
                    System.out.println("Safe thread wrote: " + temp);
                }
            }
        }, "SafeThread");
        
        sharedResource = 0;
        
        System.out.println("Starting threads that access shared resource...");
        unsafeThread.start();
        safeThread.start();
        
        try {
            unsafeThread.join();
            safeThread.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Final shared resource value: " + sharedResource);
        System.out.println("Critical section demonstration completed\n");
    }
}

// =============================================================================
// 6. SYNCHRONIZED KEYWORD - COMPREHENSIVE EXAMPLES
// =============================================================================

/**
 * Demonstrates all forms of synchronized keyword usage
 */
class SynchronizedDemo {
    private int instanceCounter = 0;
    private static int classCounter = 0;
    private final Object customLock = new Object();
    
    /**
     * Synchronized instance method
     * Locks on 'this' object
     */
    public synchronized void synchronizedInstanceMethod() {
        for (int i = 0; i < 3; i++) {
            instanceCounter++;
            System.out.println(Thread.currentThread().getName() + 
                             " - Instance method: " + instanceCounter);
            try { Thread.sleep(200); } catch (InterruptedException e) { 
                Thread.currentThread().interrupt(); 
                return; 
            }
        }
    }
    
    /**
     * Synchronized static method
     * Locks on class object (SynchronizedDemo.class)
     */
    public static synchronized void synchronizedStaticMethod() {
        for (int i = 0; i < 3; i++) {
            classCounter++;
            System.out.println(Thread.currentThread().getName() + 
                             " - Static method: " + classCounter);
            try { Thread.sleep(200); } catch (InterruptedException e) { 
                Thread.currentThread().interrupt(); 
                return; 
            }
        }
    }
    
    /**
     * Synchronized block on 'this'
     */
    public void synchronizedBlockOnThis() {
        System.out.println(Thread.currentThread().getName() + " - Before synchronized block");
        
        synchronized (this) {
            for (int i = 0; i < 3; i++) {
                instanceCounter++;
                System.out.println(Thread.currentThread().getName() + 
                                 " - Block on 'this': " + instanceCounter);
                try { Thread.sleep(200); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }
        
        System.out.println(Thread.currentThread().getName() + " - After synchronized block");
    }
    
    /**
     * Synchronized block on class
     */
    public void synchronizedBlockOnClass() {
        synchronized (SynchronizedDemo.class) {
            for (int i = 0; i < 3; i++) {
                classCounter++;
                System.out.println(Thread.currentThread().getName() + 
                                 " - Block on class: " + classCounter);
                try { Thread.sleep(200); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }
    }
    
    /**
     * Synchronized block on custom object
     */
    public void synchronizedBlockOnCustomObject() {
        synchronized (customLock) {
            for (int i = 0; i < 3; i++) {
                System.out.println(Thread.currentThread().getName() + 
                                 " - Block on custom lock: " + (i + 1));
                try { Thread.sleep(200); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        }
    }
    
    /**
     * Demonstration method
     */
    public static void demonstrateSynchronizedTypes() {
        System.out.println("=== SYNCHRONIZED KEYWORD DEMONSTRATION ===");
        
        SynchronizedDemo demo1 = new SynchronizedDemo();
        SynchronizedDemo demo2 = new SynchronizedDemo();
        
        // Test synchronized instance method
        System.out.println("\n1. Synchronized Instance Method Test:");
        Thread t1 = new Thread(() -> demo1.synchronizedInstanceMethod(), "Thread-1");
        Thread t2 = new Thread(() -> demo1.synchronizedInstanceMethod(), "Thread-2");
        Thread t3 = new Thread(() -> demo2.synchronizedInstanceMethod(), "Thread-3");
        
        t1.start(); t2.start(); t3.start();
        try { t1.join(); t2.join(); t3.join(); } catch (InterruptedException e) { }
        
        // Test synchronized static method
        System.out.println("\n2. Synchronized Static Method Test:");
        Thread t4 = new Thread(() -> SynchronizedDemo.synchronizedStaticMethod(), "Thread-4");
        Thread t5 = new Thread(() -> SynchronizedDemo.synchronizedStaticMethod(), "Thread-5");
        
        t4.start(); t5.start();
        try { t4.join(); t5.join(); } catch (InterruptedException e) { }
        
        // Test synchronized blocks
        System.out.println("\n3. Synchronized Block Tests:");
        Thread t6 = new Thread(() -> demo1.synchronizedBlockOnThis(), "Thread-6");
        Thread t7 = new Thread(() -> demo1.synchronizedBlockOnThis(), "Thread-7");
        
        t6.start(); t7.start();
        try { t6.join(); t7.join(); } catch (InterruptedException e) { }
        
        System.out.println("\nSynchronized demonstration completed\n");
    }
}

// =============================================================================
// MAIN DEMONSTRATION CLASS
// =============================================================================

public class Phase2CoreMechanisms {
    
    public static void main(String[] args) {
        System.out.println("=====================================");
        System.out.println("PHASE 2: CORE THREADING MECHANISMS");
        System.out.println("=====================================\n");
        
        try {
            // Thread Methods Demonstrations
            ThreadMethodsDemo.demonstrateStartVsRun();
            waitForUser();
            
            ThreadMethodsDemo.demonstrateSleep();
            waitForUser();
            
            ThreadMethodsDemo.demonstrateJoin();
            waitForUser();
            
            ThreadMethodsDemo.demonstrateJoinWithTimeout();
            waitForUser();
            
            ThreadMethodsDemo.demonstrateInterrupt();
            waitForUser();
            
            ThreadMethodsDemo.demonstrateYield();
            waitForUser();
            
            ThreadMethodsDemo.demonstrateDaemonThreads();
            waitForUser();
            
            ThreadMethodsDemo.demonstratePriorities();
            waitForUser();
            
            // Synchronization Fundamentals
            RaceConditionDemo.demonstrateRaceCondition();
            waitForUser();
            
            CriticalSectionDemo.demonstrateCriticalSection();
            waitForUser();
            
            // Synchronized Keyword
            SynchronizedDemo.demonstrateSynchronizedTypes();
            
            printPhase2Summary();
            
        } catch (Exception e) {
            System.err.println("Error in Phase 2 demonstration: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    private static void waitForUser() {
        System.out.println("Press Enter to continue...");
        try { System.in.read(); } catch (Exception e) { }
    }
    
    private static void printPhase2Summary() {
        System.out.println("=====================================");
        System.out.println("PHASE 2 SUMMARY");
        System.out.println("=====================================");
        System.out.println("✓ Thread Methods:");
        System.out.println("  • start() vs run() - ALWAYS use start()");
        System.out.println("  • sleep() - Thread.sleep(), handle InterruptedException");
        System.out.println("  • join() - Wait for thread completion");
        System.out.println("  • interrupt() - Cooperative thread cancellation");
        System.out.println("  • yield() - Hint to scheduler");
        System.out.println("  • setDaemon() - Background threads");
        System.out.println("  • setPriority() - Hint to scheduler");
        System.out.println();
        System.out.println("✓ Synchronization Fundamentals:");
        System.out.println("  • Race Conditions - Multiple threads, shared data");
        System.out.println("  • Critical Sections - Code requiring exclusive access");
        System.out.println("  • Mutual Exclusion - One thread at a time");
        System.out.println();
        System.out.println("✓ Synchronized Keyword:");
        System.out.println("  • Instance methods - lock on 'this'");
        System.out.println("  • Static methods - lock on class object");
        System.out.println("  • Synchronized blocks - custom lock objects");
        System.out.println();
        System.out.println("NEXT: Phase 3 - Advanced Synchronization");
        System.out.println("(wait/notify, volatile, ThreadLocal)");
    }
}

// =============================================================================
// PHASE 2 PRACTICAL EXERCISES - CORE THREADING MECHANISMS
// =============================================================================

import java.util.*;
import java.util.concurrent.atomic.AtomicBoolean;

// =============================================================================
// EXERCISE 1: Thread Coordination with join()
// =============================================================================

/**
 * Exercise 1: Download Manager Simulation
 * 
 * Create a download manager that:
 * 1. Downloads multiple files concurrently
 * 2. Shows progress for each file
 * 3. Waits for all downloads to complete
 * 4. Shows total time taken
 */
class Exercise1_DownloadManager {
    
    static class FileDownloader implements Runnable {
        private final String fileName;
        private final int fileSizeKB;
        private final int downloadSpeedKBps;
        
        public FileDownloader(String fileName, int fileSizeKB, int downloadSpeedKBps) {
            this.fileName = fileName;
            this.fileSizeKB = fileSizeKB;
            this.downloadSpeedKBps = downloadSpeedKBps;
        }
        
        @Override
        public void run() {
            try {
                System.out.println("Started downloading: " + fileName + " (" + fileSizeKB + " KB)");
                
                int downloaded = 0;
                while (downloaded < fileSizeKB) {
                    Thread.sleep(1000); // 1 second intervals
                    downloaded += downloadSpeedKBps;
                    
                    if (downloaded > fileSizeKB) {
                        downloaded = fileSizeKB;
                    }
                    
                    int progress = (downloaded * 100) / fileSizeKB;
                    System.out.println(fileName + " - Progress: " + progress + "% (" + downloaded + "/" + fileSizeKB + " KB)");
                }
                
                System.out.println("✓ Completed downloading: " + fileName);
                
            } catch (InterruptedException e) {
                System.out.println("Download interrupted: " + fileName);
                Thread.currentThread().interrupt();
            }
        }
    }
    
    public static void runDownloadManager() {
        System.out.println("=== EXERCISE 1: DOWNLOAD MANAGER ===");
        System.out.println("Starting multiple file downloads...\n");
        
        // Create different download tasks
        List<Thread> downloadThreads = Arrays.asList(
            new Thread(new FileDownloader("document.pdf", 500, 100), "PDF-Downloader"),
            new Thread(new FileDownloader("video.mp4", 2000, 200), "Video-Downloader"),
            new Thread(new FileDownloader("music.mp3", 300, 150), "Music-Downloader"),
            new Thread(new FileDownloader("software.zip", 1500, 80), "Software-Downloader")
        );
        
        long startTime = System.currentTimeMillis();
        
        // Start all downloads
        downloadThreads.forEach(Thread::start);
        
        try {
            // Wait for all downloads to complete
            System.out.println("Download manager waiting for all downloads to complete...");
            for (Thread thread : downloadThreads) {
                thread.join();
            }
            
            long endTime = System.currentTimeMillis();
            System.out.println("\n🎉 All downloads completed!");
            System.out.println("Total time: " + (endTime - startTime) / 1000.0 + " seconds");
            
        } catch (InterruptedException e) {
            System.out.println("Download manager was interrupted!");
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Download manager exercise completed\n");
    }
}

// =============================================================================
// EXERCISE 2: Interrupt Handling
// =============================================================================

/**
 * Exercise 2: Task Processor with Cancellation
 * 
 * Create a task processor that:
 * 1. Processes items in a queue
 * 2. Can be interrupted/cancelled gracefully
 * 3. Reports progress and handles cleanup
 */
class Exercise2_TaskProcessor implements Runnable {
    private final List<String> tasks;
    private final String processorName;
    private volatile boolean completed = false;
    private volatile int processedCount = 0;
    
    public Exercise2_TaskProcessor(String processorName, List<String> tasks) {
        this.processorName = processorName;
        this.tasks = new ArrayList<>(tasks);
    }
}