// =============================================================================
// PHASE 1: FOUNDATION CONCEPTS - COMPLETE EXAMPLES AND EXPLANATIONS
// =============================================================================

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

// =============================================================================
// 1. UNDERSTANDING PROCESSES VS THREADS
// =============================================================================

/**
 * PROCESS vs THREAD Demonstration
 * 
 * PROCESS:
 * - Independent execution unit with separate memory space
 * - Heavy resource consumption
 * - Inter-process communication is expensive
 * - Crash of one process doesn't affect others
 * 
 * THREAD:
 * - Lightweight sub-process within a process
 * - Shares memory space with other threads
 * - Low resource consumption
 * - Fast inter-thread communication
 * - Crash of one thread can affect entire process
 */
class ProcessVsThreadDemo {
    private static int sharedCounter = 0; // Shared memory between threads
    
    public static void demonstrateMemorySharing() {
        System.out.println("=== MEMORY SHARING DEMONSTRATION ===");
        
        // Create multiple threads that share the same memory
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                sharedCounter++;
                System.out.println("Thread 1 incremented counter to: " + sharedCounter);
                try { Thread.sleep(100); } catch (InterruptedException e) { }
            }
        });
        
        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                sharedCounter++;
                System.out.println("Thread 2 incremented counter to: " + sharedCounter);
                try { Thread.sleep(100); } catch (InterruptedException e) { }
            }
        });
        
        thread1.start();
        thread2.start();
        
        try {
            thread1.join(); // Wait for thread1 to complete
            thread2.join(); // Wait for thread2 to complete
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Final counter value: " + sharedCounter);
        System.out.println("Notice: Both threads accessed the same variable!\n");
    }
}

// =============================================================================
// 2. THREAD LIFECYCLE AND STATES
// =============================================================================

/**
 * Complete Thread Lifecycle Demonstration
 * States: NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED
 */
class ThreadLifecycleDemo {
    
    public static void demonstrateAllStates() {
        System.out.println("=== THREAD LIFECYCLE DEMONSTRATION ===");
        
        Thread thread = new Thread(() -> {
            try {
                System.out.println("Thread is RUNNABLE and executing");
                
                // Simulate some work
                Thread.sleep(2000); // TIMED_WAITING state
                
                // Demonstrate WAITING state
                synchronized(ThreadLifecycleDemo.class) {
                    System.out.println("Thread about to wait");
                    ThreadLifecycleDemo.class.wait(1000); // WAITING/TIMED_WAITING
                }
                
                System.out.println("Thread finishing execution");
                
            } catch (InterruptedException e) {
                System.out.println("Thread was interrupted");
                Thread.currentThread().interrupt();
            }
        });
        
        // NEW state
        System.out.println("Thread state after creation: " + thread.getState());
        
        // Start the thread - moves to RUNNABLE
        thread.start();
        System.out.println("Thread state after start(): " + thread.getState());
        
        // Monitor state changes
        new Thread(() -> {
            try {
                while (thread.isAlive()) {
                    Thread.State state = thread.getState();
                    System.out.println("Current thread state: " + state);
                    Thread.sleep(500);
                }
                System.out.println("Final thread state: " + thread.getState());
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
        
        try {
            Thread.sleep(1500);
            // Notify the waiting thread
            synchronized(ThreadLifecycleDemo.class) {
                ThreadLifecycleDemo.class.notify();
            }
            
            thread.join(); // Wait for completion
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Thread lifecycle demonstration complete\n");
    }
}

// =============================================================================
// 3. CREATING THREADS - ALL METHODS DEMONSTRATED
// =============================================================================

/**
 * Method 1: Extending Thread Class
 * - Direct inheritance from Thread
 * - Override run() method
 * - Less flexible (single inheritance limitation)
 */
class ThreadByExtension extends Thread {
    private final String taskName;
    
    public ThreadByExtension(String taskName) {
        this.taskName = taskName;
    }
    
    @Override
    public void run() {
        System.out.println("Executing task: " + taskName + " on thread: " + Thread.currentThread().getName());
        
        for (int i = 1; i <= 3; i++) {
            System.out.println(taskName + " - Step " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
        
        System.out.println(taskName + " completed!");
    }
}

/**
 * Method 2: Implementing Runnable Interface (PREFERRED)
 * - More flexible (can implement other interfaces)
 * - Separates task from thread management
 * - Better object-oriented design
 */
class TaskByRunnable implements Runnable {
    private final String taskName;
    
    public TaskByRunnable(String taskName) {
        this.taskName = taskName;
    }
    
    @Override
    public void run() {
        System.out.println("Runnable task: " + taskName + " on thread: " + Thread.currentThread().getName());
        
        for (int i = 1; i <= 3; i++) {
            System.out.println(taskName + " - Processing " + i);
            try {
                Thread.sleep(800);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }
        }
        
        System.out.println(taskName + " finished!");
    }
}

/**
 * Method 3: Using Lambda Expressions (Java 8+)
 * - Concise syntax
 * - Perfect for simple tasks
 * - Leverages functional programming
 */
class LambdaThreadDemo {
    public static void demonstrateLambdaThreads() {
        System.out.println("=== LAMBDA THREAD DEMONSTRATION ===");
        
        // Simple lambda thread
        Thread lambdaThread1 = new Thread(() -> {
            System.out.println("Lambda thread 1 executing on: " + Thread.currentThread().getName());
            
            for (int i = 0; i < 3; i++) {
                System.out.println("Lambda task - iteration " + (i + 1));
                try { Thread.sleep(500); } catch (InterruptedException e) { 
                    Thread.currentThread().interrupt(); 
                    return; 
                }
            }
        });
        
        // Lambda with parameters
        String message = "Hello from Lambda!";
        Thread lambdaThread2 = new Thread(() -> {
            System.out.println("Message: " + message);
            System.out.println("Lambda thread 2 on: " + Thread.currentThread().getName());
        });
        
        lambdaThread1.start();
        lambdaThread2.start();
        
        try {
            lambdaThread1.join();
            lambdaThread2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("Lambda demonstration complete\n");
    }
}

/**
 * Method 4: Using Callable Interface
 * - Can return a value
 * - Can throw checked exceptions
 * - Used with ExecutorService
 */
class CallableTaskDemo implements Callable<String> {
    private final String taskName;
    private final int processingTime;
    
    public CallableTaskDemo(String taskName, int processingTime) {
        this.taskName = taskName;
        this.processingTime = processingTime;
    }
    
    @Override
    public String call() throws Exception {
        System.out.println("Callable task: " + taskName + " started on: " + Thread.currentThread().getName());
        
        // Simulate processing
        Thread.sleep(processingTime);
        
        // Return result
        return "Task " + taskName + " completed successfully in " + processingTime + "ms";
    }
    
    public static void demonstrateCallable() {
        System.out.println("=== CALLABLE DEMONSTRATION ===");
        
        ExecutorService executor = Executors.newFixedThreadPool(2);
        
        try {
            // Submit callable tasks
            Future<String> future1 = executor.submit(new CallableTaskDemo("DataProcessing", 2000));
            Future<String> future2 = executor.submit(new CallableTaskDemo("FileIO", 1500));
            
            // Get results (this will block until tasks complete)
            System.out.println("Result 1: " + future1.get());
            System.out.println("Result 2: " + future2.get());
            
        } catch (Exception e) {
            System.err.println("Error executing callable tasks: " + e.getMessage());
        } finally {
            executor.shutdown();
        }
        
        System.out.println("Callable demonstration complete\n");
    }
}

// =============================================================================
// COMPREHENSIVE DEMONSTRATION CLASS
// =============================================================================

/**
 * Main class to demonstrate all Phase 1 concepts
 */
public class Phase1ThreadingFoundations {
    
    public static void main(String[] args) {
        System.out.println("=====================================");
        System.out.println("JAVA MULTITHREADING - PHASE 1 DEMO");
        System.out.println("=====================================\n");
        
        // 1. Process vs Thread Memory Sharing
        ProcessVsThreadDemo.demonstrateMemorySharing();
        
        waitForUser();
        
        // 2. Thread Lifecycle
        ThreadLifecycleDemo.demonstrateAllStates();
        
        waitForUser();
        
        // 3. Thread Creation Methods
        System.out.println("=== THREAD CREATION METHODS ===");
        
        // Method 1: Extending Thread
        System.out.println("1. Creating thread by extending Thread class:");
        ThreadByExtension task1 = new ThreadByExtension("DatabaseTask");
        task1.start();
        
        try { task1.join(); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        
        System.out.println("\n2. Creating thread by implementing Runnable:");
        Thread task2 = new Thread(new TaskByRunnable("NetworkTask"));
        task2.start();
        
        try { task2.join(); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        
        System.out.println();
        
        // Method 3: Lambda expressions
        LambdaThreadDemo.demonstrateLambdaThreads();
        
        waitForUser();
        
        // Method 4: Callable
        CallableTaskDemo.demonstrateCallable();
        
        // Summary
        printSummary();
    }
    
    private static void waitForUser() {
        System.out.println("Press Enter to continue to next demonstration...");
        try {
            System.in.read();
        } catch (Exception e) {
            // Ignore
        }
    }
    
    private static void printSummary() {
        System.out.println("=====================================");
        System.out.println("PHASE 1 SUMMARY");
        System.out.println("=====================================");
        System.out.println("✓ Understood Process vs Thread differences");
        System.out.println("✓ Learned Thread lifecycle states");
        System.out.println("✓ Mastered 4 ways to create threads:");
        System.out.println("  - Extending Thread class");
        System.out.println("  - Implementing Runnable interface (PREFERRED)");
        System.out.println("  - Using Lambda expressions");
        System.out.println("  - Using Callable interface");
        System.out.println();
        System.out.println("KEY TAKEAWAYS:");
        System.out.println("• Use Runnable over Thread extension");
        System.out.println("• Threads share memory space");
        System.out.println("• Always handle InterruptedException");
        System.out.println("• Use Callable when you need return values");
        System.out.println("• Lambda expressions are great for simple tasks");
        System.out.println();
        System.out.println("Ready for Phase 2: Core Threading Mechanisms!");
    }
}

// =============================================================================
// PRACTICE EXERCISES FOR PHASE 1
// =============================================================================

/**
 * EXERCISE 1: Create a thread that counts from 1 to 10
 * Requirements:
 * - Use Runnable interface
 * - Print current number and thread name
 * - Add 500ms delay between numbers
 */
class Exercise1_Counter implements Runnable {
    @Override
    public void run() {
        // TODO: Implement counter logic
    }
}

/**
 * EXERCISE 2: Create a Callable that calculates factorial
 * Requirements:
 * - Accept a number in constructor
 * - Return factorial as result
 * - Handle edge cases (negative numbers)
 */
class Exercise2_Factorial implements Callable<Long> {
    private final int number;
    
    public Exercise2_Factorial(int number) {
        this.number = number;
    }
    
    @Override
    public Long call() throws Exception {
        // TODO: Implement factorial calculation
        return 1L;
    }
}

/**
 * EXERCISE 3: Create multiple threads and observe their states
 * Requirements:
 * - Create 3 threads with different sleep times
 * - Monitor and print their states every 100ms
 * - Use lambda expressions
 */
class Exercise3_StateMonitor {
    public static void monitorThreadStates() {
        // TODO: Implement state monitoring
    }
}

// =============================================================================
// PHASE 1 PRACTICAL EXERCISES - COMPLETE SOLUTIONS
// =============================================================================

import java.util.concurrent.*;
import java.util.ArrayList;
import java.util.List;

// =============================================================================
// EXERCISE 1: Counter Thread (SOLUTION)
// =============================================================================

/**
 * Exercise 1: Create a thread that counts from 1 to 10
 * Learning objectives:
 * - Implement Runnable interface
 * - Use Thread.sleep() properly
 * - Handle InterruptedException
 * - Access thread information
 */
class Exercise1_Counter implements Runnable {
    private final String counterName;
    private final int startNum;
    private final int endNum;
    
    public Exercise1_Counter(String counterName, int startNum, int endNum) {
        this.counterName = counterName;
        this.startNum = startNum;
        this.endNum = endNum;
    }
    
    @Override
    public void run() {
        System.out.println("Starting counter: " + counterName + " on thread: " + Thread.currentThread().getName());
        
        for (int i = startNum; i <= endNum; i++) {
            System.out.println(counterName + " - Count: " + i + " [Thread: " + Thread.currentThread().getName() + "]");
            
            try {
                Thread.sleep(500); // 500ms delay
            } catch (InterruptedException e) {
                System.out.println(counterName + " was interrupted at count: " + i);
                Thread.currentThread().interrupt(); // Restore interrupt status
                return; // Exit gracefully
            }
        }
        
        System.out.println(counterName + " completed counting from " + startNum + " to " + endNum);
    }
}

// =============================================================================
// EXERCISE 2: Factorial Callable (SOLUTION)
// =============================================================================

/**
 * Exercise 2: Factorial calculator using Callable
 * Learning objectives:
 * - Implement Callable interface
 * - Return values from threads
 * - Handle edge cases
 * - Throw exceptions from call() method
 */
class Exercise2_Factorial implements Callable<Long> {
    private final int number;
    
    public Exercise2_Factorial(int number) {
        this.number = number;
    }
    
    @Override
    public Long call() throws Exception {
        System.out.println("Calculating factorial of " + number + " on thread: " + Thread.currentThread().getName());
        
        // Handle edge cases
        if (number < 0) {
            throw new IllegalArgumentException("Factorial is not defined for negative numbers: " + number);
        }
        
        if (number == 0 || number == 1) {
            return 1L;
        }
        
        // Calculate factorial
        long result = 1L;
        for (int i = 2; i <= number; i++) {
            result *= i;
            
            // Check for overflow
            if (result < 0) {
                throw new ArithmeticException("Factorial overflow for number: " + number);
            }
            
            // Simulate some processing time
            Thread.sleep(50);
        }
        
        System.out.println("Factorial of " + number + " = " + result);
        return result;
    }
}

// =============================================================================
// EXERCISE 3: Thread State Monitor (SOLUTION)
// =============================================================================

/**
 * Exercise 3: Monitor thread states
 * Learning objectives:
 * - Create multiple threads with lambda expressions
 * - Monitor thread states in real-time
 * - Understand thread lifecycle
 * - Coordinate multiple threads
 */
class Exercise3_StateMonitor {
    
    public static void monitorThreadStates() throws InterruptedException {
        System.out.println("\n=== THREAD STATE MONITORING EXERCISE ===");
        
        // Create threads with different behaviors
        Thread shortTask = new Thread(() -> {
            System.out.println("Short task starting");
            try { Thread.sleep(1000); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            System.out.println("Short task completed");
        }, "ShortTask");
        
        Thread mediumTask = new Thread(() -> {
            System.out.println("Medium task starting");
            try { Thread.sleep(3000); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            System.out.println("Medium task completed");
        }, "MediumTask");
        
        Thread longTask = new Thread(() -> {
            System.out.println("Long task starting");
            try { Thread.sleep(5000); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            System.out.println("Long task completed");
        }, "LongTask");
        
        // Store threads in list for monitoring
        List<Thread> threads = List.of(shortTask, mediumTask, longTask);
        
        // Display initial states
        System.out.println("\nInitial thread states:");
        threads.forEach(t -> System.out.println(t.getName() + ": " + t.getState()));
        
        // Start all threads
        System.out.println("\nStarting all threads...");
        threads.forEach(Thread::start);
        
        // Monitor states
        Thread monitor = new Thread(() -> {
            try {
                while (threads.stream().anyMatch(Thread::isAlive)) {
                    System.out.println("\n--- Thread States ---");
                    threads.forEach(t -> System.out.println(t.getName() + ": " + t.getState() + " (Alive: " + t.isAlive() + ")"));
                    Thread.sleep(500); // Check every 500ms
                }
                
                System.out.println("\n--- Final Thread States ---");
                threads.forEach(t -> System.out.println(t.getName() + ": " + t.getState()));
                
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "StateMonitor");
        
        monitor.start();
        
        // Wait for all tasks to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        monitor.join();
        System.out.println("State monitoring completed\n");
    }
}

// =============================================================================
// BONUS EXERCISE: Thread Communication Demo
// =============================================================================

/**
 * Bonus Exercise: Simple Producer-Consumer without synchronization
 * (We'll add proper synchronization in Phase 3)
 * Learning objective: See what happens without proper synchronization
 */
class BonusExercise_SimpleProducerConsumer {
    private static List<Integer> buffer = new ArrayList<>(); // Shared resource
    private static final int MAX_SIZE = 5;
    private static volatile boolean isProducing = true;
    
    public static void demonstrateWithoutSynchronization() throws InterruptedException {
        System.out.println("=== BONUS: PRODUCER-CONSUMER WITHOUT SYNCHRONIZATION ===");
        System.out.println("Note: This will show race conditions - we'll fix this in Phase 3!\n");
        
        // Producer thread
        Thread producer = new Thread(() -> {
            int item = 1;
            while (item <= 10) {
                if (buffer.size() < MAX_SIZE) {
                    buffer.add(item);
                    System.out.println("Produced: " + item + " | Buffer size: " + buffer.size());
                    item++;
                    
                    try { Thread.sleep(300); } catch (InterruptedException e) { 
                        Thread.currentThread().interrupt(); 
                        return; 
                    }
                }
            }
            isProducing = false;
            System.out.println("Producer finished");
        }, "Producer");
        
        // Consumer thread
        Thread consumer = new Thread(() -> {
            while (isProducing || !buffer.isEmpty()) {
                if (!buffer.isEmpty()) {
                    Integer item = buffer.remove(0); // This can cause issues!
                    System.out.println("Consumed: " + item + " | Buffer size: " + buffer.size());
                    
                    try { Thread.sleep(500); } catch (InterruptedException e) { 
                        Thread.currentThread().interrupt(); 
                        return; 
                    }
                }
            }
            System.out.println("Consumer finished");
        }, "Consumer");
        
        producer.start();
        consumer.start();
        
        producer.join();
        consumer.join();
        
        System.out.println("Producer-Consumer demo completed");
        System.out.println("Did you notice any issues? We'll fix them in Phase 3!\n");
    }
}

// =============================================================================
// COMPREHENSIVE TESTING AND DEMONSTRATION
// =============================================================================

public class Phase1ExercisesSolutions {
    
    public static void main(String[] args) {
        System.out.println("=====================================");
        System.out.println("PHASE 1 EXERCISES - SOLUTIONS");
        System.out.println("=====================================\n");
        
        try {
            // Exercise 1: Counter threads
            runExercise1();
            
            System.out.println("\nPress Enter to continue...");
            System.in.read();
            
            // Exercise 2: Factorial with Callable
            runExercise2();
            
            System.out.println("\nPress Enter to continue...");
            System.in.read();
            
            // Exercise 3: State monitoring
            Exercise3_StateMonitor.monitorThreadStates();
            
            System.out.println("Press Enter to continue...");
            System.in.read();
            
            // Bonus: Producer-Consumer
            BonusExercise_SimpleProducerConsumer.demonstrateWithoutSynchronization();
            
            printExerciseSummary();
            
        } catch (Exception e) {
            System.err.println("Error during exercises: " + e.getMessage());
            e.printStackTrace();
        }
    }
    
    private static void runExercise1() throws InterruptedException {
        System.out.println("=== EXERCISE 1: COUNTER THREADS ===");
        
        // Create multiple counters
        Thread counter1 = new Thread(new Exercise1_Counter("Counter-A", 1, 5));
        Thread counter2 = new Thread(new Exercise1_Counter("Counter-B", 10, 15));
        Thread counter3 = new Thread(new Exercise1_Counter("Counter-C", 100, 103), "CustomThreadName");
        
        counter1.start();
        counter2.start();
        counter3.start();
        
        // Wait for all to complete
        counter1.join();
        counter2.join();
        counter3.join();
        
        System.out.println("All counters completed!\n");
    }
    
    private static void runExercise2() {
        System.out.println("=== EXERCISE 2: FACTORIAL CALCULATION ===");
        
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        try {
            // Submit multiple factorial calculations
            Future<Long> future1 = executor.submit(new Exercise2_Factorial(5));
            Future<Long> future2 = executor.submit(new Exercise2_Factorial(10));
            Future<Long> future3 = executor.submit(new Exercise2_Factorial(0)); // Edge case
            Future<Long> future4 = executor.submit(new Exercise2_Factorial(-5)); // Error case
            
            // Get results
            System.out.println("Factorial 5! = " + future1.get());
            System.out.println("Factorial 10! = " + future2.get());
            System.out.println("Factorial 0! = " + future3.get());
            
            try {
                System.out.println("Factorial (-5)! = " + future4.get());
            } catch (ExecutionException e) {
                System.out.println("Error calculating factorial of -5: " + e.getCause().getMessage());
            }
            
        } catch (Exception e) {
            System.err.println("Error in factorial calculation: " + e.getMessage());
        } finally {
            executor.shutdown();
        }
        
        System.out.println("Factorial calculations completed!\n");
    }
    
    private static void printExerciseSummary() {
        System.out.println("\n=====================================");
        System.out.println("PHASE 1 EXERCISES SUMMARY");
        System.out.println("=====================================");
        System.out.println("✓ Exercise 1: Counter threads with Runnable");
        System.out.println("✓ Exercise 2: Factorial calculation with Callable");
        System.out.println("✓ Exercise 3: Thread state monitoring");
        System.out.println("✓ Bonus: Producer-Consumer (with race conditions)");
        System.out.println();
        System.out.println("SKILLS PRACTICED:");
        System.out.println("• Creating threads with different methods");
        System.out.println("• Handling InterruptedException properly");
        System.out.println("• Using Future to get results from Callable");
        System.out.println("• Monitoring thread states and lifecycle");
        System.out.println("• Understanding shared memory issues");
        System.out.println();
        System.out.println("NEXT STEPS:");
        System.out.println("• Review any concepts that weren't clear");
        System.out.println("• Try modifying the exercises");
        System.out.println("• Ready for Phase 2: Core Threading Mechanisms");
    }
}

// =============================================================================
// ADDITIONAL PRACTICE CHALLENGES
// =============================================================================

/**
 * CHALLENGE 1: Multi-threaded File Processor
 * Create threads that process different file types concurrently
 */
class Challenge1_FileProcessor implements Runnable {
    private final String fileName;
    private final String fileType;
    
    public Challenge1_FileProcessor(String fileName, String fileType) {
        this.fileName = fileName;
        this.fileType = fileType;
    }
    
    @Override
    public void run() {
        System.out.println("Processing " + fileType + " file: " + fileName);
        
        // Simulate different processing times based on file type
        int processingTime = switch (fileType.toLowerCase()) {
            case "txt" -> 1000;
            case "jpg" -> 2000;
            case "mp4" -> 5000;
            default -> 1500;
        };
        
        try {
            Thread.sleep(processingTime);
            System.out.println("Completed processing: " + fileName + " (" + fileType + ")");
        } catch (InterruptedException e) {
            System.out.println("Processing interrupted for: " + fileName);
            Thread.currentThread().interrupt();
        }
    }
    
    // TODO: Create main method to test this with multiple file types
}

/**
 * CHALLENGE 2: Thread Pool Simulation
 * Create a simple thread pool without using ExecutorService
 */
class Challenge2_SimpleThreadPool {
    // TODO: Implement a basic thread pool
    // - Fixed number of worker threads
    // - Queue for tasks
    // - Start/stop functionality
}

/**
 * CHALLENGE 3: Concurrent Counter
 * Create a counter that can be incremented by multiple threads
 * (Hint: This will have race conditions - we'll fix in Phase 3)
 */
class Challenge3_ConcurrentCounter {
    private int count = 0;
    
    public void increment() {
        count++; // Race condition here!
    }
    
    public int getCount() {
        return count;
    }
    
    // TODO: Create test with multiple threads incrementing
}