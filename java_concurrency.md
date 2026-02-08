# Java Concurrency - Deep Dive Guide

## Table of Contents

1. [Introduction to Concurrency](#1-introduction-to-concurrency)
2. [Concurrency Correctness](#2-concurrency-correctness)
3. [Thread Coordination](#3-thread-coordination)
4. [Resource Scarcity Problems](#4-resource-scarcity-problems)

---

## 1. Introduction to Concurrency

### What is Concurrency?

Concurrency is the ability of a program to execute multiple tasks simultaneously. In Java, this is achieved through threads - independent paths of execution within a program.

### Why Use Concurrency?

1. **Performance**: Utilize multiple CPU cores
2. **Responsiveness**: Keep UI responsive while processing
3. **Throughput**: Handle multiple requests simultaneously
4. **Resource Utilization**: Better CPU and I/O utilization

### Processes vs Threads

```java
// Process: Independent execution with own memory space
// Thread: Lightweight, shares memory within same process

public class ThreadBasics {
    public static void main(String[] args) {
        // Method 1: Extend Thread class
        Thread thread1 = new MyThread();
        thread1.start();

        // Method 2: Implement Runnable
        Thread thread2 = new Thread(new MyRunnable());
        thread2.start();

        // Method 3: Lambda expression
        Thread thread3 = new Thread(() -> {
            System.out.println("Running in thread: " + Thread.currentThread().getName());
        });
        thread3.start();

        // Method 4: ExecutorService (recommended)
        ExecutorService executor = Executors.newFixedThreadPool(3);
        executor.submit(() -> {
            System.out.println("Task executed by: " + Thread.currentThread().getName());
        });
        executor.shutdown();
    }
}

class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("MyThread running");
    }
}

class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("MyRunnable running");
    }
}
```

### Thread Lifecycle

```java
public class ThreadLifecycle {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                System.out.println("Thread RUNNING");
                Thread.sleep(1000); // TIMED_WAITING
                System.out.println("Thread finishing");
            } catch (InterruptedException e) {
                System.out.println("Thread INTERRUPTED");
            }
        });

        // NEW state
        System.out.println("State: " + thread.getState()); // NEW

        // RUNNABLE state
        thread.start();
        System.out.println("State: " + thread.getState()); // RUNNABLE

        // Wait for thread to finish (TERMINATED)
        thread.join();
        System.out.println("State: " + thread.getState()); // TERMINATED
    }
}
```

### ExecutorService Framework

```java
import java.util.concurrent.*;
import java.util.List;
import java.util.ArrayList;

public class ExecutorServiceExample {
    public static void main(String[] args) throws Exception {
        // Fixed Thread Pool
        ExecutorService fixedPool = Executors.newFixedThreadPool(3);

        // Cached Thread Pool (creates threads as needed)
        ExecutorService cachedPool = Executors.newCachedThreadPool();

        // Single Thread Executor
        ExecutorService singleExecutor = Executors.newSingleThreadExecutor();

        // Scheduled Executor
        ScheduledExecutorService scheduledExecutor = Executors.newScheduledThreadPool(2);

        // Submit tasks
        Future<Integer> future = fixedPool.submit(() -> {
            Thread.sleep(1000);
            return 42;
        });

        // Get result (blocking)
        Integer result = future.get(); // Waits for completion
        System.out.println("Result: " + result);

        // Check if done
        if (future.isDone()) {
            System.out.println("Task completed");
        }

        // Submit multiple tasks
        List<Callable<String>> tasks = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            tasks.add(() -> "Task " + taskId + " result");
        }

        // Execute all and wait for all to complete
        List<Future<String>> futures = fixedPool.invokeAll(tasks);

        for (Future<String> f : futures) {
            System.out.println(f.get());
        }

        // Scheduled tasks
        scheduledExecutor.schedule(() -> {
            System.out.println("Executed after 5 seconds");
        }, 5, TimeUnit.SECONDS);

        scheduledExecutor.scheduleAtFixedRate(() -> {
            System.out.println("Executed every 3 seconds");
        }, 0, 3, TimeUnit.SECONDS);

        // Proper shutdown
        fixedPool.shutdown();
        if (!fixedPool.awaitTermination(60, TimeUnit.SECONDS)) {
            fixedPool.shutdownNow();
        }
    }
}
```

### CompletableFuture (Modern Async Programming)

```java
import java.util.concurrent.CompletableFuture;

public class CompletableFutureExample {
    public static void main(String[] args) throws Exception {
        // Simple async task
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            sleep(1000);
            return "Hello";
        });

        // Chain operations
        CompletableFuture<String> result = future
            .thenApply(s -> s + " World")
            .thenApply(String::toUpperCase);

        System.out.println(result.get()); // HELLO WORLD

        // Combine multiple futures
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

        CompletableFuture<String> combined = future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2);
        System.out.println(combined.get());

        // Handle exceptions
        CompletableFuture<String> withError = CompletableFuture.supplyAsync(() -> {
            if (true) throw new RuntimeException("Error!");
            return "Success";
        }).exceptionally(ex -> "Recovered: " + ex.getMessage());

        System.out.println(withError.get());

        // All of / Any of
        CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2);
        allOf.join(); // Wait for all

        CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2);
        System.out.println(anyOf.get()); // First to complete
    }

    private static void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) {}
    }
}
```

---

## 2. Concurrency Correctness

### Race Conditions

A race condition occurs when multiple threads access shared data simultaneously, and the outcome depends on the timing of their execution.

```java
// INCORRECT: Race condition
class Counter {
    private int count = 0;

    public void increment() {
        count++; // NOT atomic! (read, increment, write)
    }

    public int getCount() {
        return count;
    }
}

public class RaceConditionDemo {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();

        // Create 1000 threads, each incrementing 1000 times
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }

        // Wait for all threads
        for (Thread t : threads) {
            t.join();
        }

        // Expected: 1,000,000
        // Actual: Less than 1,000,000 (race condition!)
        System.out.println("Count: " + counter.getCount());
    }
}
```

### Solution 1: Synchronized Keyword

```java
class SynchronizedCounter {
    private int count = 0;

    // Method-level synchronization
    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }

    // Block-level synchronization
    public void incrementBlock() {
        synchronized (this) {
            count++;
        }
    }

    // Static synchronization (class-level lock)
    private static int staticCount = 0;

    public static synchronized void incrementStatic() {
        staticCount++;
    }
}
```

### Solution 2: Atomic Classes

```java
import java.util.concurrent.atomic.*;

class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet(); // Atomic operation
    }

    public int getCount() {
        return count.get();
    }

    // Compare and Swap (CAS)
    public boolean incrementIfLessThan(int threshold) {
        int current;
        int next;
        do {
            current = count.get();
            if (current >= threshold) {
                return false;
            }
            next = current + 1;
        } while (!count.compareAndSet(current, next));
        return true;
    }
}

// Other atomic types
public class AtomicExamples {
    private AtomicLong atomicLong = new AtomicLong(0);
    private AtomicBoolean atomicBoolean = new AtomicBoolean(false);
    private AtomicReference<String> atomicRef = new AtomicReference<>("initial");

    // Atomic array
    private AtomicIntegerArray atomicArray = new AtomicIntegerArray(10);

    public void demonstrate() {
        // Atomic operations
        atomicLong.addAndGet(10);
        atomicLong.compareAndSet(10, 20);

        atomicBoolean.getAndSet(true);

        atomicRef.updateAndGet(s -> s.toUpperCase());

        atomicArray.incrementAndGet(0);
    }
}
```

### Solution 3: ReentrantLock

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class LockCounter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock(); // ALWAYS unlock in finally block
        }
    }

    // Try lock with timeout
    public boolean tryIncrement() {
        try {
            if (lock.tryLock(1, TimeUnit.SECONDS)) {
                try {
                    count++;
                    return true;
                } finally {
                    lock.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return false;
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}

// ReadWriteLock for read-heavy scenarios
class ReadWriteCounter {
    private int count = 0;
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();

    public void increment() {
        writeLock.lock();
        try {
            count++;
        } finally {
            writeLock.unlock();
        }
    }

    public int getCount() {
        readLock.lock(); // Multiple readers allowed
        try {
            return count;
        } finally {
            readLock.unlock();
        }
    }
}
```

### Volatile Keyword

```java
class VolatileExample {
    // Without volatile, threads may cache the value
    private volatile boolean running = true;

    public void stop() {
        running = false; // Immediately visible to all threads
    }

    public void run() {
        while (running) {
            // Do work
        }
        System.out.println("Stopped");
    }
}

// Double-checked locking with volatile
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) { // First check (no locking)
            synchronized (Singleton.class) {
                if (instance == null) { // Second check (with locking)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### Thread-Safe Collections

```java
import java.util.concurrent.*;
import java.util.*;

public class ThreadSafeCollections {
    public static void main(String[] args) {
        // ConcurrentHashMap - thread-safe, no locking for reads
        ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
        concurrentMap.put("key", 1);
        concurrentMap.putIfAbsent("key", 2);
        concurrentMap.computeIfPresent("key", (k, v) -> v + 1);

        // CopyOnWriteArrayList - good for read-heavy scenarios
        CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
        cowList.add("item");

        // BlockingQueue - producer-consumer pattern
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

        // Producer
        new Thread(() -> {
            try {
                queue.put("item"); // Blocks if queue is full
            } catch (InterruptedException e) {}
        }).start();

        // Consumer
        new Thread(() -> {
            try {
                String item = queue.take(); // Blocks if queue is empty
            } catch (InterruptedException e) {}
        }).start();

        // ConcurrentLinkedQueue - non-blocking queue
        ConcurrentLinkedQueue<String> nonBlockingQueue = new ConcurrentLinkedQueue<>();
        nonBlockingQueue.offer("item");
        String item = nonBlockingQueue.poll();

        // Synchronized collections (legacy)
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
    }
}
```

### ThreadLocal

```java
public class ThreadLocalExample {
    // Each thread has its own copy
    private static ThreadLocal<Integer> threadId = ThreadLocal.withInitial(() -> 0);

    public static void main(String[] args) {
        Runnable task = () -> {
            int id = (int) (Math.random() * 1000);
            threadId.set(id);

            System.out.println(Thread.currentThread().getName() + " ID: " + threadId.get());

            // Clean up to prevent memory leaks
            threadId.remove();
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
    }
}

// Common use case: Database connections
class DatabaseConnection {
    private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<>();

    public static Connection getConnection() {
        Connection conn = connectionHolder.get();
        if (conn == null) {
            conn = createNewConnection();
            connectionHolder.set(conn);
        }
        return conn;
    }

    public static void closeConnection() {
        Connection conn = connectionHolder.get();
        if (conn != null) {
            try {
                conn.close();
            } catch (Exception e) {}
            connectionHolder.remove();
        }
    }

    private static Connection createNewConnection() {
        // Create and return connection
        return null;
    }
}
```

---

## 3. Thread Coordination

### Wait/Notify Pattern

```java
class WaitNotifyExample {
    private final Object lock = new Object();
    private boolean dataReady = false;

    // Producer
    public void produce() {
        synchronized (lock) {
            System.out.println("Producing data...");
            dataReady = true;
            lock.notify(); // Wake up one waiting thread
            // lock.notifyAll(); // Wake up all waiting threads
        }
    }

    // Consumer
    public void consume() {
        synchronized (lock) {
            while (!dataReady) { // Always use while, not if!
                try {
                    lock.wait(); // Release lock and wait
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
            System.out.println("Consuming data...");
            dataReady = false;
        }
    }
}

// Producer-Consumer with Queue
class ProducerConsumer {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int MAX_SIZE = 10;
    private final Object lock = new Object();

    public void produce(int value) throws InterruptedException {
        synchronized (lock) {
            while (queue.size() == MAX_SIZE) {
                lock.wait(); // Queue full, wait
            }
            queue.add(value);
            System.out.println("Produced: " + value);
            lock.notifyAll(); // Notify consumers
        }
    }

    public int consume() throws InterruptedException {
        synchronized (lock) {
            while (queue.isEmpty()) {
                lock.wait(); // Queue empty, wait
            }
            int value = queue.poll();
            System.out.println("Consumed: " + value);
            lock.notifyAll(); // Notify producers
            return value;
        }
    }
}
```

### CountDownLatch

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        int numWorkers = 3;
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(numWorkers);

        // Create workers
        for (int i = 0; i < numWorkers; i++) {
            new Thread(new Worker(startSignal, doneSignal)).start();
        }

        System.out.println("Preparing...");
        Thread.sleep(1000);

        // Start all workers simultaneously
        startSignal.countDown();

        // Wait for all workers to finish
        doneSignal.await();
        System.out.println("All workers completed");
    }

    static class Worker implements Runnable {
        private final CountDownLatch startSignal;
        private final CountDownLatch doneSignal;

        Worker(CountDownLatch start, CountDownLatch done) {
            this.startSignal = start;
            this.doneSignal = done;
        }

        public void run() {
            try {
                startSignal.await(); // Wait for start signal
                doWork();
                doneSignal.countDown(); // Signal completion
            } catch (InterruptedException e) {}
        }

        void doWork() {
            System.out.println(Thread.currentThread().getName() + " working");
        }
    }
}
```

### CyclicBarrier

```java
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.BrokenBarrierException;

public class CyclicBarrierExample {
    public static void main(String[] args) {
        int numThreads = 3;

        // Barrier action runs when all threads reach barrier
        CyclicBarrier barrier = new CyclicBarrier(numThreads, () -> {
            System.out.println("All threads reached barrier, proceeding...");
        });

        for (int i = 0; i < numThreads; i++) {
            new Thread(new Task(barrier, i)).start();
        }
    }

    static class Task implements Runnable {
        private final CyclicBarrier barrier;
        private final int id;

        Task(CyclicBarrier barrier, int id) {
            this.barrier = barrier;
            this.id = id;
        }

        public void run() {
            try {
                System.out.println("Thread " + id + " doing phase 1");
                Thread.sleep((long) (Math.random() * 1000));

                barrier.await(); // Wait for all threads

                System.out.println("Thread " + id + " doing phase 2");
                Thread.sleep((long) (Math.random() * 1000));

                barrier.await(); // Can be reused!

                System.out.println("Thread " + id + " completed");
            } catch (InterruptedException | BrokenBarrierException e) {}
        }
    }
}
```

### Semaphore

```java
import java.util.concurrent.Semaphore;

// Connection pool using Semaphore
class ConnectionPool {
    private final Semaphore semaphore;
    private final List<Connection> connections;

    public ConnectionPool(int poolSize) {
        semaphore = new Semaphore(poolSize);
        connections = new ArrayList<>(poolSize);
        for (int i = 0; i < poolSize; i++) {
            connections.add(new Connection(i));
        }
    }

    public Connection acquire() throws InterruptedException {
        semaphore.acquire(); // Blocks if no permits available
        return getConnection();
    }

    public void release(Connection conn) {
        if (returnConnection(conn)) {
            semaphore.release(); // Release permit
        }
    }

    private synchronized Connection getConnection() {
        for (Connection conn : connections) {
            if (!conn.isInUse()) {
                conn.setInUse(true);
                return conn;
            }
        }
        return null;
    }

    private synchronized boolean returnConnection(Connection conn) {
        conn.setInUse(false);
        return true;
    }
}

// Binary semaphore (mutex)
class SemaphoreMutex {
    private final Semaphore mutex = new Semaphore(1);
    private int count = 0;

    public void increment() throws InterruptedException {
        mutex.acquire();
        try {
            count++;
        } finally {
            mutex.release();
        }
    }
}

// Rate limiter using Semaphore
class RateLimiter {
    private final Semaphore semaphore;

    public RateLimiter(int requestsPerSecond) {
        semaphore = new Semaphore(requestsPerSecond);

        // Refill permits every second
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        scheduler.scheduleAtFixedRate(() -> {
            int available = semaphore.availablePermits();
            if (available < requestsPerSecond) {
                semaphore.release(requestsPerSecond - available);
            }
        }, 1, 1, TimeUnit.SECONDS);
    }

    public boolean tryAcquire() {
        return semaphore.tryAcquire();
    }
}
```

### Phaser (Advanced Barrier)

```java
import java.util.concurrent.Phaser;

public class PhaserExample {
    public static void main(String[] args) {
        Phaser phaser = new Phaser(1); // Register main thread

        int numThreads = 3;
        for (int i = 0; i < numThreads; i++) {
            phaser.register(); // Register each thread
            new Thread(new PhaserTask(phaser, i)).start();
        }

        // Main thread arrives and deregisters
        phaser.arriveAndDeregister();

        // Wait for all to complete
        while (!phaser.isTerminated()) {
            Thread.yield();
        }
    }

    static class PhaserTask implements Runnable {
        private final Phaser phaser;
        private final int id;

        PhaserTask(Phaser phaser, int id) {
            this.phaser = phaser;
            this.id = id;
        }

        public void run() {
            System.out.println("Thread " + id + " phase 0");
            phaser.arriveAndAwaitAdvance(); // Phase 0

            System.out.println("Thread " + id + " phase 1");
            phaser.arriveAndAwaitAdvance(); // Phase 1

            System.out.println("Thread " + id + " completed");
            phaser.arriveAndDeregister(); // Done
        }
    }
}
```

### Exchanger

```java
import java.util.concurrent.Exchanger;

public class ExchangerExample {
    public static void main(String[] args) {
        Exchanger<String> exchanger = new Exchanger<>();

        // Thread 1
        new Thread(() -> {
            try {
                String data = "Data from thread 1";
                String received = exchanger.exchange(data);
                System.out.println("Thread 1 received: " + received);
            } catch (InterruptedException e) {}
        }).start();

        // Thread 2
        new Thread(() -> {
            try {
                String data = "Data from thread 2";
                String received = exchanger.exchange(data);
                System.out.println("Thread 2 received: " + received);
            } catch (InterruptedException e) {}
        }).start();
    }
}
```

---

## 4. Resource Scarcity Problems

### Deadlock

A deadlock occurs when two or more threads are blocked forever, waiting for each other to release resources.

```java
// DEADLOCK EXAMPLE
public class DeadlockDemo {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: Holding lock1...");
            sleep(100);

            synchronized (lock2) {
                System.out.println("Thread 1: Holding lock1 & lock2");
            }
        }
    }

    public void method2() {
        synchronized (lock2) {
            System.out.println("Thread 2: Holding lock2...");
            sleep(100);

            synchronized (lock1) {
                System.out.println("Thread 2: Holding lock1 & lock2");
            }
        }
    }

    public static void main(String[] args) {
        DeadlockDemo demo = new DeadlockDemo();

        new Thread(demo::method1).start();
        new Thread(demo::method2).start();
        // DEADLOCK! Both threads waiting forever
    }

    private static void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) {}
    }
}
```

### Deadlock Prevention Strategies

```java
// Strategy 1: Lock Ordering
public class LockOrdering {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void method1() {
        synchronized (lock1) { // Always acquire lock1 first
            synchronized (lock2) {
                System.out.println("Method 1 executing");
            }
        }
    }

    public void method2() {
        synchronized (lock1) { // Same order!
            synchronized (lock2) {
                System.out.println("Method 2 executing");
            }
        }
    }
}

// Strategy 2: Lock Timeout
public class LockTimeout {
    private final ReentrantLock lock1 = new ReentrantLock();
    private final ReentrantLock lock2 = new ReentrantLock();

    public boolean transfer() {
        try {
            if (lock1.tryLock(1, TimeUnit.SECONDS)) {
                try {
                    if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                        try {
                            // Perform transfer
                            return true;
                        } finally {
                            lock2.unlock();
                        }
                    }
                } finally {
                    lock1.unlock();
                }
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return false;
    }
}

// Strategy 3: Single Lock
public class SingleLock {
    private final Object globalLock = new Object();

    public void operation1() {
        synchronized (globalLock) {
            // All operations use same lock
        }
    }

    public void operation2() {
        synchronized (globalLock) {
            // No circular wait possible
        }
    }
}

// Strategy 4: Detect and Recover
public class DeadlockDetection {
    public static void detectDeadlock() {
        ThreadMXBean threadBean = ManagementFactory.getThreadMXBean();
        long[] deadlockedThreads = threadBean.findDeadlockedThreads();

        if (deadlockedThreads != null) {
            ThreadInfo[] threadInfos = threadBean.getThreadInfo(deadlockedThreads);
            System.out.println("Deadlock detected!");
            for (ThreadInfo info : threadInfos) {
                System.out.println(info.getThreadName() + " - " + info.getLockName());
            }
        }
    }
}
```

### Livelock

A livelock is similar to deadlock, but threads are actively trying to resolve the conflict and keep changing states without making progress.

```java
// LIVELOCK EXAMPLE
public class LivelockDemo {
    static class Spoon {
        private Diner owner;

        public synchronized void setOwner(Diner d) {
            owner = d;
        }

        public synchronized Diner getOwner() {
            return owner;
        }

        public synchronized void use() {
            System.out.println(owner.name + " is eating");
        }
    }

    static class Diner {
        private String name;
        private boolean isHungry;

        public Diner(String name) {
            this.name = name;
            this.isHungry = true;
        }

        public void eatWith(Spoon spoon, Diner spouse) {
            while (isHungry) {
                // Don't have spoon
                if (spoon.getOwner() != this) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {}
                    continue;
                }

                // Spouse is hungry, give them the spoon
                if (spouse.isHungry) {
                    System.out.println(name + ": You eat first, " + spouse.name);
                    spoon.setOwner(spouse);
                    continue; // LIVELOCK: constantly giving spoon back
                }

                // Eat
                spoon.use();
                isHungry = false;
                spoon.setOwner(spouse);
            }
        }
    }

    public static void main(String[] args) {
        Diner husband = new Diner("Husband");
        Diner wife = new Diner("Wife");
        Spoon spoon = new Spoon();
        spoon.setOwner(husband);

        new Thread(() -> husband.eatWith(spoon, wife)).start();
        new Thread(() -> wife.eatWith(spoon, husband)).start();
    }
}

// SOLUTION: Add randomness or priority
public class LivelockSolution {
    // Add random backoff
    private void eatWith(Spoon spoon, Diner spouse) {
        Random random = new Random();
        while (isHungry) {
            if (spoon.getOwner() != this) {
                Thread.sleep(random.nextInt(10)); // Random wait
                continue;
            }

            if (spouse.isHungry && random.nextBoolean()) { // 50% chance
                spoon.setOwner(spouse);
                continue;
            }

            spoon.use();
            isHungry = false;
        }
    }
}
```

### Starvation

Starvation occurs when a thread is perpetually denied access to resources.

```java
// STARVATION EXAMPLE
public class StarvationDemo {
    private final Object lock = new Object();

    // High priority task
    public void highPriorityTask() {
        while (true) {
            synchronized (lock) {
                System.out.println("High priority task running");
            }
        }
    }

    // Low priority task may never get the lock!
    public void lowPriorityTask() {
        while (true) {
            synchronized (lock) {
                System.out.println("Low priority task running");
            }
        }
    }
}

// SOLUTION: Use fair locks
public class StarvationSolution {
    // Fair lock ensures FIFO ordering
    private final ReentrantLock fairLock = new ReentrantLock(true);

    public void task() {
        fairLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " executing");
        } finally {
            fairLock.unlock();
        }
    }
}
```

### Dining Philosophers Problem

Classic concurrency problem demonstrating deadlock.

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class DiningPhilosophers {
    static class Fork {
        private final Lock lock = new ReentrantLock();

        public boolean pickUp() {
            return lock.tryLock();
        }

        public void putDown() {
            lock.unlock();
        }
    }

    static class Philosopher implements Runnable {
        private final int id;
        private final Fork leftFork;
        private final Fork rightFork;

        public Philosopher(int id, Fork left, Fork right) {
            this.id = id;
            this.leftFork = left;
            this.rightFork = right;
        }

        private void think() throws InterruptedException {
            System.out.println("Philosopher " + id + " is thinking");
            Thread.sleep((long) (Math.random() * 1000));
        }

        private void eat() throws InterruptedException {
            System.out.println("Philosopher " + id + " is eating");
            Thread.sleep((long) (Math.random() * 1000));
        }

        @Override
        public void run() {
            try {
                while (true) {
                    think();

                    // Try to pick up both forks
                    if (leftFork.pickUp()) {
                        try {
                            if (rightFork.pickUp()) {
                                try {
                                    eat();
                                } finally {
                                    rightFork.putDown();
                                }
                            }
                        } finally {
                            leftFork.putDown();
                        }
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    public static void main(String[] args) {
        int numPhilosophers = 5;
        Fork[] forks = new Fork[numPhilosophers];
        Philosopher[] philosophers = new Philosopher[numPhilosophers];

        for (int i = 0; i < numPhilosophers; i++) {
            forks[i] = new Fork();
        }

        for (int i = 0; i < numPhilosophers; i++) {
            Fork leftFork = forks[i];
            Fork rightFork = forks[(i + 1) % numPhilosophers];

            // Prevent deadlock: last philosopher picks right fork first
            if (i == numPhilosophers - 1) {
                philosophers[i] = new Philosopher(i, rightFork, leftFork);
            } else {
                philosophers[i] = new Philosopher(i, leftFork, rightFork);
            }

            new Thread(philosophers[i]).start();
        }
    }
}
```

### Resource Pool Pattern

```java
import java.util.concurrent.*;

public class ResourcePool<T> {
    private final BlockingQueue<T> pool;
    private final Semaphore semaphore;

    public ResourcePool(int size, Supplier<T> factory) {
        pool = new LinkedBlockingQueue<>(size);
        semaphore = new Semaphore(size, true); // Fair semaphore

        // Initialize pool
        for (int i = 0; i < size; i++) {
            pool.offer(factory.get());
        }
    }

    public T acquire() throws InterruptedException {
        semaphore.acquire();
        return pool.take();
    }

    public T tryAcquire(long timeout, TimeUnit unit)
            throws InterruptedException {
        if (semaphore.tryAcquire(timeout, unit)) {
            return pool.poll(timeout, unit);
        }
        return null;
    }

    public void release(T resource) {
        if (pool.offer(resource)) {
            semaphore.release();
        }
    }

    public int availableResources() {
        return semaphore.availablePermits();
    }
}

// Usage
class DatabaseConnectionPool {
    public static void main(String[] args) throws InterruptedException {
        ResourcePool<Connection> pool = new ResourcePool<>(10,
            () -> createConnection());

        // Acquire connection
        Connection conn = pool.acquire();
        try {
            // Use connection
            executeQuery(conn);
        } finally {
            // Always release back to pool
            pool.release(conn);
        }

        // With timeout
        Connection conn2 = pool.tryAcquire(5, TimeUnit.SECONDS);
        if (conn2 != null) {
            try {
                executeQuery(conn2);
            } finally {
                pool.release(conn2);
            }
        }
    }

    static Connection createConnection() {
        return new Connection();
    }

    static void executeQuery(Connection conn) {}

    static class Connection {}
}
```

---

## Best Practices Summary

### 1. Thread Safety

- **Prefer immutability**: Immutable objects are inherently thread-safe
- **Minimize shared mutable state**: Reduce what needs synchronization
- **Use thread-safe collections**: ConcurrentHashMap, CopyOnWriteArrayList
- **Encapsulate synchronization**: Don't expose locks to clients

### 2. Lock Management

- **Always release locks in finally blocks**
- **Use try-with-resources when possible**
- **Prefer ReentrantLock over synchronized for advanced features**
- **Use fair locks to prevent starvation**
- **Keep critical sections small**

### 3. Avoid Common Pitfalls

- **Don't lock on public objects**: Use private final locks
- **Avoid holding locks during I/O**: I/O operations can block
- **Don't use Thread.stop()**: It's deprecated and unsafe
- **Check interrupted status**: Handle InterruptedException properly
- **Avoid busy waiting**: Use wait/notify or blocking queues

### 4. Performance

- **Use ReadWriteLock for read-heavy scenarios**
- **Prefer atomic variables over synchronized for simple operations**
- **Use ConcurrentHashMap instead of synchronized HashMap**
- **Consider lock-free algorithms for high contention**
- **Profile before optimizing**: Measure actual contention

### 5. Testing

- **Use stress tests**: Run with many threads
- **Vary thread counts and timing**: Expose race conditions
- **Use tools**: ThreadSanitizer, FindBugs, SpotBugs
- **Test under load**: Simulate production conditions

---

## Common Interview Questions

### Q1: Implement a Thread-Safe Singleton

```java
// Best approach: Enum (recommended by Joshua Bloch)
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // Business logic
    }
}

// Alternative: Static inner class (lazy initialization)
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Q2: Implement BlockingQueue

```java
public class SimpleBlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    public SimpleBlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void put(T item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait();
        }
        queue.add(item);
        notifyAll();
    }

    public synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        T item = queue.poll();
        notifyAll();
        return item;
    }
}
```

### Q3: Print Numbers Alternatively

```java
// Print 1,2,3... alternately using two threads
public class AlternatePrinting {
    private int count = 1;
    private final int max = 10;
    private final Object lock = new Object();

    public void printOdd() {
        synchronized (lock) {
            while (count <= max) {
                while (count % 2 == 0) {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {}
                }
                if (count <= max) {
                    System.out.println("Odd: " + count);
                    count++;
                    lock.notify();
                }
            }
        }
    }

    public void printEven() {
        synchronized (lock) {
            while (count <= max) {
                while (count % 2 != 0) {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {}
                }
                if (count <= max) {
                    System.out.println("Even: " + count);
                    count++;
                    lock.notify();
                }
            }
        }
    }
}
```

---

This comprehensive guide covers the fundamental concepts of Java concurrency. Master these patterns and you'll be well-prepared for both interviews and production systems!
