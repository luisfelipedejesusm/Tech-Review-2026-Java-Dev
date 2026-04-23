# Java Essentials for 2026

## 1. Core Language Fundamentals

### Data Types
- Primitives and Reference types
    - Primitive types store values directly in memory whereas reference types store the address to objects in memory. There are 8 primitive types: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`

- Autoboxing and Unboxing
    - This process used by Java converts primitives to their wrapper class and viceversa.
        ```java
        Integer x = 10;     // int -> Integer
        int y = x;          // Integer -> int
        ```
- Immutability
    - An object is immutable if it can not change after creation. One example in Java are Strings and other wrapper classes.
        ```java
        String obj1 = "object 1";   // this creates a new object
        obj1.concat("new");         // this creates a new object

        Integer x = 1;              // New object
        x = x + 1;                  // this creates a new object
        ```

### Collections Framework
- Core interfaces:
    - **List**  : ordered, allow duplicates
    - **Set**   : no duplicates
    - **Map**   : key-value pairs
    - **Queue** : FIFO / Processing Order

- Implementations
    - **ArrayList vs LinkedList**: ArrayLists are backed by dynamic arrays whereas LinkedLists are double linked nodes. That said, random access for ArrayLists are `O(1)` and for LinkedLists `O(n)`. Insertion in the other hand is `O(n)` for ArrayLists and `O(1)` for LinkedLists. 

    - **HashMaps vs TreeMap vs ConcurrentHashMap**: Use HashMaps as the default option unless you need ordering or multi-thread support. For ordering, better use TreeMap (red-black tree) and for multi-threading use ConcurrentHashMap

    - **HashSet vs TreeSet**: A hashSet uses a HashMap and has `O(1)` operations. On the other hand, a TreeSet uses a TreeMap and has `O(log n)` complexity

- Key Concepts
    - **Time Complexity**: efficiency of an algorithm based on the amount of time it takes to run as a function of the input size (denoted as `n`)

    - **Hash Collisions & Resizing**: Java creates a hashcode using a key and the size of the collection to store the object in the bucket. When these keyx collide, a resize is triggered. This doubles the capacity of the collection and all entries need to be rehashed (which is expensive).

    - **Fail-Fast vs Fail-Safe**: Fail-Fast iterators (most collections) throw exeption when a concurrent modification is being taken place. Fail-Safe (concurrent collections) iterate over a copy of the collection, ensuring therefore thread-safety at the cost of extra memory.

### Memory and JVM
- Memory areas
    - **Heap**: Memory allocated to the application. It can be shared across threads and is managed by the Garbage Collector.

    - **Stack**: Memory utilized by threads (one stack per thread) that stores method calls, local variables, object references, etc.. It is managed by the programmer, has very fast allocation / deallocation and it's automatically cleaned when method ends.

    - **Metaspace**: Stores class metadata. This lives in native memory and not the heap.

- Garbage Collection
    - The GC is an automated memory management process that identifies and reclaims memory occupied by objects that are no longer reachable by the application. 

    - Minor GC cleans memory from short-lived objects such as local variables. It runs faster and more frequent than the Major GC. Major Gc reclaims memory from long-lived objects that survived many Minor GC. It is slower and can create pauses in the application.

    - Some common collectors are G1 and ZGC. G1 Garbage Collector is region-based, splitting the heap into multiple regions and focusing on collecting those that are mostly garbage in order to maximize efficiency. ZGC is a low-latency collector, good for large-scale systems there pause times should be only miliseconds.

- GC Tuning
    ```bash
    -Xms2g      # initial heap
    -Xmx2g      # max heap
    -Xlog:gc    # enable logging
    ```

### Concurrency and Multithreading
- Basics
    - **Threads and Processes**: A process is an independent program (it has its own memory space). A thread is a lightweight unit inside a process and has shared memory.
        ```java
        // example of a thread in Java
        Thread t = new Thread(() -> System.out.println("Hi"));
        t.start();
        ```
    
    - **Runnable vs Callable**: Callable returns a result and can handle exceptions. 
        ```java
        // Example in java
        Runnable r = () -> System.out.println("no return");

        Callable<Integer> c = () -> 42; // returns a value
        ```
    
    - **Thread Lifecycle**: NEW → RUNNABLE → RUNNING → BLOCKED/WAITING → TERMINATED

- Synchronization
    - **Synchronized**: ensures that only one thread can execute / access this section of the code at a time.
        ```java
        public synchronized void increment(){
            count++;
        }
        ```
    
    - **ReentrantLock**: more control than synchronized, can timeone and skip access.
        ```java
        Lock lock new ReentrantLock();

        lock.lock();
        try{
            count++;
        } finally {
            lock.unlock();
        }
        ```
        This is better because we can use `.tryLock()` to set up skip or fallback logic
        ```java
        Lock lock = new ReentrantLock();

        if(lock.tryLock()) {
            try {
                // do work
            }finally {
                lock.unlock();
            }
        } else {
            // fallback logic
        }
        ```
    - **volatile**: Guarantees visibility, NOT atomicity. This is to ensure that threads read the most up-to-date value of a variable.
        ```java
        volatile boolean running = true; 
        running = false; // running is visible to other threads and its false

        // this is NOT safe
        volatile int count = 0;
        count++; // not atomic -> race condition

        // instead use
        AtomicInteger count = new AtomicInteger(0);
        count.incrementAndGet();
        ```

- Advanced
    - **Executors**: Manages a pool of reusable threads instead of creating new ones manually. Threads creating is expensive and the use of executors allow reusing them as well as limiting concurrency to avoid overload.
        ```java
        ExecutorService executor = Executors.newFixedThreadPool(4);
        executor.submit(() -> doWork());
        ```
    
    - **Fork/Join Framework**: This framework allows you to parallelize tasks by splitting them into smaller subtasks and combining the results

    - **CompletableFuture**: A class for writting async, non-blocking code using chained operations.
        ```java
        CompletableFuture.supplyAsync(() -> fetchData())
            .thenApply(data -> process(data))
            .thenAccept(System.out::println);
        ```
        We can always make it blocking to wait the result by using the method `.get()`

- Problems
    - **Race Condition**: Mulitple threads access and modify data concurrently, leading to unpredictable results. *Solution*: locking, synchronized, using Atomic properties.

    - **Deadlock**: When two or more threads are waiting on eachother, freezig the execution completely. Solution: use timeouts.

    - **Starvation**: This happens when a thread never gets access to resources because others dominate. Solution: Use faire locks (`ReentrantLock`) and avoid priority abuse.