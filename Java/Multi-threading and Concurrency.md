- **Thread** → lightweight unit of execution inside a process.
- Threads share heap/resources; processes don’t.
- Each thread has:
    - its own **stack**
    - **instruction pointer**
- Too many threads → **thrashing** (CPU manages threads instead of doing work).
- **Thrashing** - Too many threads -> CPU spends more time managing threads than doing the work.
## Thread Creation - Thread vs Runnable
- **Thread Class** - creates thread by extending the tread class. Override run() of the Thread class.
- **Runnable Interface** - Implement the interface in a class. To create thread, you pass Runnable to it.
- **Checked Exception** - 
	- Declared in function description using *throws* and handled using *try...catch*. Checked at compile time.
	- Eg- IOException(Eg- file not found), SQLException(Eg- DB conn failed) etc.
	- Forces the calling code to implement try...catch
```Java
class MyThread extends Thread {
    public void run() {}
}

class MyRunnable implements Runnable {
    public void run() {}
}

Thread t1 = new MyThread();
Thread t2 = new Thread(new MyRunnable());

t1.start();
t2.start();

Thread.currentThread();
thread.sleep(1000);
thread.interrupt();
thread.join();

thread.setName("Worker");
thread.setPriority(Thread.MAX_PRIORITY);
```
## Thread Termination, Daemon and join
- If thread finishes but app is still running we need to terminate it.
- If thread is misbehaving or taking a long time, you might want to terminate it.
#### thread.interrupt(), isInterrupted()
- Can be used if the thread is executing a function which throws an InterruptedException
- Or Thread internally handles the interrupted exception.
#### Daemon Threads
- Background threads that stop when JVM exits. No need to handle interrupt exception in such a case.
- Eg - Thread which saves our code even after exit.
#### join()
- When you have to wait for a thread to complete.
- You can check isCompleted() in a while loop but that wastes compute cycles. This makes thread sleep.
```Java
class ThreadDemo {

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    System.out.println("Worker running : " + i);
                    Thread.sleep(1000);
                    if (Thread.currentThread().isInterrupted()) {
                        System.out.println("Worker detected interrupt");
                        return;
                    }
                }
            } catch (InterruptedException e) {
                System.out.println("Worker interrupted during sleep");
            }
            System.out.println("Worker finished");
        });
        Thread daemon = new Thread(() -> {
            while (true) {
                System.out.println("Daemon thread running...");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    System.out.println("Daemon interrupted");
                }
            }
        });
        daemon.setDaemon(true);
        worker.start();
        daemon.start();
        Thread.sleep(3000);
        System.out.println("\nMain thread interrupting worker...\n");
        worker.interrupt();
        worker.join();
        System.out.println("\nMain thread finished");
    }
}
```
## Thread Performance and Latency
- **Performance Criteria** - depends on the application. Eg- throughput for HFTs and latency for web apps.
- **Latency** - Time taken for one task.
- **Throughput** - Tasks completed per unit time.
### Thread Pooling - Executor Framework
- Multiple tasks to parallelize -> Can't create thread for each -> **Limited threads + thrashing.**
- A **thread pool** is -> A fixed number of reusable threads that execute tasks from a queue.
- **Executors** - Java implementation of a thread pool. Takes in Runnable/Callable and executes them.
- Extra compute to create threads every time is reduced.
```Java
import java.util.concurrent.*;
class ExecutorExample {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        for (int i = 1; i <= 5; i++) {
            int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " started by " + Thread.currentThread().getName());
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Task " + taskId + " finished by " + Thread.currentThread().getName());
            });
        }
        executor.shutdown();
    }
}
```
- Types of Thread Pools (important)
```Java
Executors.newFixedThreadPool(n); //-> n threads in the pool
Executors.newCachedThreadPool(); //-> Creates new threads as required and reuse the old one.
Executors.newSingleThreadExecutor(); //-> Uses a single thread to execute.
```
- Executor Lifecycle - Running, Shutting Down and Terminated
#### submit() vs execute()
- `execute()` → no return -> Use with Runnable
- `submit()` → returns `Future` -> Use with Callable
#### shutdown()
- use otherwise JVM will not exit.
## Data Sharing between Threads
- **Stack, IP** -> Not shared. **Heap** -> Shared.
- Obj, Class Members and static variables -> Heap
- Local variable obj references -> Stack, Obj references properties -> Heap. Obj -> Heap.
```Java
public class Counter{
    public static void main(String[] args) {
        InvCounter counter = new InvCounter();
        Thread incrementThread = new IncrementThread(counter);
        Thread decrementThread = new DecrementThread(counter);
        incrementThread.start();
        decrementThread.start();
        try {
            incrementThread.join();
            decrementThread.join();
        } catch (InterruptedException e) {
            System.out.println("Main thread was interrupted while waiting for child threads to finish.");
        }   
        System.out.println("Final count: " + counter.getCount());
    }
    public static class IncrementThread extends Thread {
        private InvCounter counter;
        IncrementThread(InvCounter counter) {
            this.counter = counter;
        }
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        }
    }
    public static class DecrementThread extends Thread {
        private InvCounter counter;
        DecrementThread(InvCounter counter) {
            this.counter = counter;
        }
        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                counter.decrement();
            }
        }
    }
    public static class InvCounter{
        private int count = 0;
        public void increment(){
            count++;
        }
        public void decrement(){
            count--;
        }
        public int getCount(){
            return count;
        }
    }
}
```
## Critical Sections and Locking
### Locking - synchronized
- **synchronized** keyword acts as a lock for the code section it covers.
- **synchronized** is object level lock 
- **Reentrant** - synchronized is reentrant -> If tread has one lock, it can also acquire another.
- Two ways to use -> Declare a method as sync or use sync block.
- **synchronized class methods** - Two methods are sync, ThreadA executes method1, ThreadB cannot execute method2.
	- Basically a synchronized block on *this*(current obj).
```Java
public class ClassWithCritSection{
	public synchronized void method1{...} // Thread A executes method1.
	public synchronized void method2{...} // Thread B cannot execute method2
}
```
- **synchronized block** -> You can make only portion of code sync. Multiple treads can execute different methods.
```Java
public class ClassWithCritSection{
	Object lock1 = new Object();
	Object lock2 = new Object();
	public void method1{ // Thread A executes here
		synchronized(lock1){
		}
	}
	public void method2{
		synchronized(lock2){ // Thread B can execute here simultaneously
		}
	}
}

public static class InvCounter{
	private int count = 0;
	private Object lock = new Object();
	public void increment(){
		synchronized(lock){
			count++;
		}
	}
	public void decrement(){
		synchronized(lock){
			count--;
		}
	}
	public int getCount(){
		return count;
	}
}
```
### Atomic Operations
- All reference assignments are atomic.
- All assignments to primitive types except double and long are atomic.
```Java
Object o1 = new Object();
Object o2 = o1; //Atomic
int x = 3;//atomic
int y;
y=x;//atomic

double x=123.4;//not atomic
```
#### Volatile
- Any volatile variable is ensured to be thread safe.
- Volatile variables guarantee -> Visibility + Ordering
- **Visibility** - Variable is not cached by CPU threads. Any read/write is made from main memory.
- **No Reordering** - CPU may execute two operations in random order if it does not violate the result.
```Java
x=2;
flag=false;
Thread t2 = new Thread(()->{x=524;flag=true;}); // Due to reordering flag=true is executed before.
Thread t1 = new Thread(()->{if(flag){System.out.println(x)}}); // Output becomes 2.
```
- **Happens before** - Visibility + No Reordering -> Every thing before a volatile value update is visibile to it.
## Race conditions and Data Race
### Race Condition
- Multiple threads try to access the same shared resource.
- If resource is not secured by a lock, it can lead to incorrect results. -> Make critical section atomic.
### Data Race
- CPU/Compiler reorders instruction for speed.
- If you try to read data being modified, due to reordering you might get incorrect data
```Java
x=2;
flag=false;
Thread t2 = new Thread(()->{x=524;flag=true;}); // Due to reordering flag=true is executed before.
Thread t1 = new Thread(()->{if(flag){System.out.println(x)}}); // Output becomes 2.
```
- This is called a data race -> Declare the shared code as synchronized so only one update happens.
-> All shared variable must be volatile.
```Java
volatile int shared;
public void method(){
... //All instructions will be executed before.
read/write(shared);
... //All instructions will be executed after.
}

```
## Reentrant Lock
- It means that the same thread can acquire the lock multiple times. 
-> If a thread acquires the lock and calls a method which again acquires the lock, it would not lead to errors
```Java
lock.lock();
lock.lock();
lock.unlock();
lock.unlock();
```
- It is a lock that provides explicit lock() and unlock() -> More flexible than synchronized.
```Java
Lock lock = new ReentrantLock();
lock.lock();
try{
	// Critical section.
}
finally{
	lock.unlock();
}
```
- Lock methods - 
	- **getQueuedThreads()** - Returns list of threads waiting to acquire lock.
	- **getOwner()** - Returns the current owner of the thread.
	- **isHeldByCurrentOwner()** - Returns if lock is held by the current thread.
	- **isLocked()** - Returns if any thread holds the lock.
- Used for testing and ensuring thread fairness.
### Fairness
- Use reentrant locks for fairness. -> All threads recieve nearly same time with the lock.
- May reduce throughput of the system.
```Java
Lock lock = new ReentrantLock(true);
```
### lockInterruptibly()
- If a thread is waiting for a lock, it cannot be interrupted.
- Helps create lock where threads are interruptible when waiting for the lock.
- This helps to get out of deadlock situations.
```Java
try{
	lockObj.lockInterruptibly();
}
catch(InterruptedException e){
	// Cleanup code.
}
```
### tryLock()
- If lock is free, it acquires the lock and proceeds to crit section.
- Used when suspending the thread on a lock is not acceptable -> Video Processing, HFT softwares etc.
```Java
if(lock.tryLock()){
	try{
	}
	finally{
		lock.unlock();
	}
}

lock.tryLock(long timeout,TimeUnit unit);
```
### RentrantReadWriteLock
- Other locks we have discussed lock the read and write threads alike.
- Reads do not need to be blocked from other reads.
- Low read latency -. No read locking.
```Java
private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
private Lock readLock = reentrantReadWriteLock.readLock();
private Lock writeLock = reentrantReadWriteLock.writeLock();
```
- **Read Locks** - can be acquired by many threads at once.
	- Can be acquired only when write lock is free.
- **Write Locks** - can be acquired by only one thread.
	- Can be acquired only when read lock is free.
## Inter Thread Communication
### Semaphores
- Restrict the number of users(threads) to shared resource.
- Binary Semaphore - Restricts to 1 user = Lock.
- Vs Locks - 
	- Semaphore does not have a notion of owner thread.
	- Same thread can acquire the thread multiple times.
	- Binary Semaphore is not reentrant -> If a binary semaphore is acquired, it cannot be acquired again.
- Semaphore can be released by any other thread despite not having acquired the semaphore.
```Java
TA ->
semaphore.acquire();
// Shared resource
semaphore.release();
TB ->
semaphore.release(); // If performed before TA calls release, it will release the semaphore.
```
#### Producer - Consumer
```Java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;
import java.util.LinkedList;
import java.util.Queue;
import java.util.List;
import java.util.ArrayList;

public class ProducerConsumer {
    private static final int MAX_PERMITS = 5;
    private static Semaphore empty = new Semaphore(MAX_PERMITS);
    private static Semaphore full = new Semaphore(0);
    private static Queue<Integer> queue = new LinkedList<>();
    private static final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        List<Thread> threads = new ArrayList<>();
        for(int i = 0; i < 3; i++){
            threads.add(new Thread(() -> {
                while(true){
                    try{
                        int item = (int)(Math.random() * 100);
                        empty.acquire();
                        lock.lock();
                        queue.add(item);
                        System.out.println("Produced: " + item);
                        lock.unlock();
                        full.release();
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }));
        }
        for(int i = 0; i < 1; i++){
            threads.add(new Thread(() -> {
                while(true){
                    try {
                        full.acquire();
                        lock.lock();
                        int item = queue.poll();
                        System.out.println("Consumed: " + item);
                        lock.unlock();
                        empty.release();
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }));
        }
        for (Thread thread : threads) {
            thread.start();
        }
    }
}
```

### Condition Variables
- *Puts the thread to sleep unitl a condition is met.* State changes-> thread notified, checks condition agian
- **always associated with a Lock** -> Lock ensures atomic check of shared variables in the condition.
- Semaphore is a special case of condition variables. Checks -> # permits available > 0 -> Proceed else sleep.
- **await()** - puts the thread to sleep.
	- **awaitNanos(nanosTimeout)** - sleep for certain nano secs.
	- **boolean await(long time, TimeUnit unit)** - wait for time in units.
	- **boolean awaitUntil(Date deadline)** - Wake up before deadline date.
- **signal()** - Wake up a single thread waiting on the condition. The thread waking up has to reacquire the lock.
- **signalAll()** - Wake up all the threads waiting.

- await() -> Release the lock and sleep -> signal() -> Moved to ready state, compete with others to get Lock.
-> Lock acquired -> check condition again(*IP is moved a instruction back when await()*) -> proceed else sleep
```Java
// DB Thread
lock.lock();
try{
	while(username == null || password == null) {condition.await();}
}
finally {
	lock.unlock();
｝
//Fetch data from DB.

// UI Thread
lock.lock();
try{
	username = userTextbox.getText();
	password = passwordTextbox.getText();
	condition.signal();
} finally {
	lock.unlock();
}
```
```Java
import java.util.concurrent.Semaphore;
import java.util.concurrent.locks.ReentrantLock;
import java.util.LinkedList;
import java.util.Queue;
import java.util.List;
import java.util.ArrayList;

public class ProducerConsumer {
    private static final int MAX_PERMITS = 5;
    private static Semaphore empty = new Semaphore(MAX_PERMITS);
    private static Semaphore full = new Semaphore(0);
    private static Queue<Integer> queue = new LinkedList<>();
    private static final ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) {
        List<Thread> threads = new ArrayList<>();
        for(int i = 0; i < 3; i++){
            threads.add(new Thread(() -> {
                while(true){
                    try{
                        int item = (int)(Math.random() * 100);
                        empty.acquire();
                        lock.lock();
                        queue.add(item);
                        System.out.println("Produced: " + item);
                        lock.unlock();
                        full.release();
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }));
        }
        for(int i = 0; i < 1; i++){
            threads.add(new Thread(() -> {
                while(true){
                    try {
                        full.acquire();
                        lock.lock();
                        int item = queue.poll();
                        System.out.println("Consumed: " + item);
                        lock.unlock();
                        empty.release();
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }));
        }
        for (Thread thread : threads) {
            thread.start();
        }
    }
}
```
### Objects as CVs
```Java
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.LinkedList;
import java.util.Queue;
import java.util.Scanner;
import java.util.StringJoiner;
import java.io.FileNotFoundException;

public class MatrixMultiplier {
    private static final String INPUT_FILE = "./Resources/matrices.txt";
    private static final String OUTPUT_FILE = "./Resources/matrix_product_result.txt";
    private static final int N = 10;

    public static void main(String[] args) {
        ThreadSafeQueue threadSafeQueue = new ThreadSafeQueue();
        File inputFile = new File(INPUT_FILE);
        File outputFile = new File(OUTPUT_FILE);
        try {
            ProducerThreaad matricesReader = new ProducerThreaad(new FileReader(inputFile), threadSafeQueue);
            ConsumerThread matricesConsumer = new ConsumerThread(new FileWriter(outputFile), threadSafeQueue);

            matricesConsumer.start();
            matricesReader.start();
        } catch (IOException e) {
            e.printStackTrace();
        } 
    }

    private static class ProducerThreaad extends Thread {
        private ThreadSafeQueue queue;
        private FileReader fileReader;
        private Scanner scanner;

        public ProducerThreaad(FileReader fileReader, ThreadSafeQueue queue) {
            this.fileReader = fileReader;
            this.queue = queue;
            this.scanner = new Scanner(fileReader);
        }

        @Override
        public void run() {
            while(true){
                float[][] matrixA = readMatrix();
                float[][] matrixB = readMatrix();
                if(matrixA == null || matrixB == null){
                    queue.terminate();
                    System.out.println("No more matrices to read from the file, producer is terminating");
                    break;
                }
                MatricesPair matricesPair = new MatricesPair(matrixA, matrixB);
                queue.add(matricesPair);
            }
        }

        private float[][] readMatrix() {
            float[][] matrix = new float[N][N];
            for (int r = 0; r < N; r++) {
                if (!scanner.hasNext()) {
                    return null;
                }
                String[] line = scanner.nextLine().split(",");
                for (int c = 0; c < N; c++) {
                    matrix[r][c] = Float.valueOf(line[c]);
                }
            }
            scanner.nextLine();
            return matrix;
        }
    }

    private static class ConsumerThread extends Thread {
        private ThreadSafeQueue queue;
        private FileWriter fileWriter;

        public ConsumerThread(FileWriter writer, ThreadSafeQueue queue) {
            this.queue = queue;
            this.fileWriter = writer;
        }

        @Override
        public void run() {
            while(true){
                MatricesPair matricesPair = queue.remove();
                if(matricesPair == null){
                    break;
                }

                float[][] result = multiplyMatrices(matricesPair);

                try{
                    saveMatrixToFile(fileWriter, result);
                }catch(IOException e){}
            }
            try {
                fileWriter.flush();
                fileWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        private float[][] multiplyMatrices(MatricesPair matricesPair){
            float[][] result = new float[N][N];
            for(int i = 0; i < N; i++){
                for(int j = 0; j < N; j++){
                    for(int k = 0; k < N; k++){
                        result[i][j] += matricesPair.matrixA[i][k] * matricesPair.matrixB[k][j];
                    }
                }
            }
            return result;
        }

        private static void saveMatrixToFile(FileWriter fileWriter, float[][] matrix) throws IOException {
            for (int r = 0; r < N; r++) {
                StringJoiner stringJoiner = new StringJoiner(", ");
                for (int c = 0; c < N; c++) {
                    stringJoiner.add(String.format("%.2f", matrix[r][c]));
                }
                fileWriter.write(stringJoiner.toString());
                fileWriter.write('\n');
            }
            fileWriter.write('\n');
        }
    }


    private static class ThreadSafeQueue{
        private Queue<MatricesPair> queue = new LinkedList<>();
        private boolean isTerminated = false;
        private final int MAX_SIZE = 5;

        public synchronized void add(MatricesPair matricesPair){
            while(queue.size() == MAX_SIZE){
                try{
                    wait();
                }catch(InterruptedException e){
                    System.out.println("Thread interrupted while waiting for space to be available in the queue to add matrices pair");
                }
            }
            queue.add(matricesPair);
            System.out.println("queue size " + queue.size());
            notify();
        }

        public synchronized MatricesPair remove(){
            while(queue.isEmpty()&&!isTerminated){
                try{
                    wait();
                }catch(InterruptedException e){
                    System.out.println("Thread interrupted while waiting for matrices pair to be added to the queue");
                }
            }
            if(queue.isEmpty()&&isTerminated){
                return null;
            }
            System.out.println("queue size " + queue.size());

            MatricesPair matricesPair = queue.remove();
            if(queue.size() == MAX_SIZE - 1){
                notifyAll();
            }
            return matricesPair;
        }

        public synchronized void terminate(){
            isTerminated = true;
        }
    }

    private static class MatricesPair{
        private float[][] matrixA;
        private float[][] matrixB;

        public MatricesPair(float[][] matrixA, float[][] matrixB) {
            this.matrixA = matrixA;
            this.matrixB = matrixB;
        }
    }
}

```
## Lock free DSA
### Problems with Locks
- **Deadlocks** are not recoverable and can bring the application to a halt.
- **Slow Crit Section** - One thread hold lock for long -> Other threads starve.
- **Priority Inversion** - Two threads share a lock but have different priority -> High Priority thread waits on Low Priority thread decreasing the priority of the high priority.
- **Missing Unlock** - All threads are hung.
- **Performance** - A has lock,B is scheduled tries to acquire then sleeps -> Extra context switch to B and then to another thread. Finally when lock is released, B needs to scheduled back -> *Latency overhead*
##### Lock Free Programming -> Use operation that are single hardware operations.
### Atomic Operations
- Read/Assignment of all primitive types(except long,double), references and all volatile primitives/references.
- Classes in the *java.util.concurrent.atomic*
#### Atomic Integer
```Java
AtomicInteger counter = new AtomicInteger(0);
counter.get();
counter.getAndIncrement();
counter.incrementAndGet();
counter.getAndDecrement();
counter.decrementAndGet();
counter.addAndGet(5);
counter.getAndAdd(-4);
```
##### Similarly we have AtomicBoolean and AtomicLong
### Atomic Reference
- Takes reference of any class and allows to perform atomic CAS updates on the object.
```Java
AtomicReference<String> ref = new AtomicReference<>("A");
ref.compareAndSet("A", "B"); // ref becomes B
ref.compareAnsSet("A", "C"); //does not update.
```
#### CompareAndSet
- **bool compareAndSet(V expectedVal, V newVal)** - 
	- returns true if expectedVal == curVal. updates the reference.
	- returns false if expectedVal ! = curVal. does not update the reference.
## Non Blocking IO - Futures
- When doing I/O thread is blocked and **CPU** deschedules it and **runs other threads.**
- When dealing with IO Bound applications, *# threads = # cores is not optimal for thread pool*
	- Say you have 4 cores, a request arrives and reads from DB -> **due to blocking one thread is blocked**
	  When similar requests arrive you will **run out of threads.**
	- This effects even requests which are not causing the blocking IO.
### Thread per task model
- We create a new thread for each task coming in -> Enough threads for each task.
- **Pros** - 
	- Increasing number of threads increases throughput and hardware utilization.
- **Cons** - 
	- Number of threads is limited. -> Limited memory.
	- Multiple blocking ops -> the thread gets switched out multiple times -> **Huge Overhead** + **Thrashing**(CPU is mostly managing threads instead of running actual code).
### Non Blocking IO
- You use futures here. You call a async operation and pass a method to it.
- When the operation is complete the method runs and produces result.
```Java
public void handleRequest(HttpExchange exchange) {
Request request = parseUserRequest(exchange);
readFromDatabaseAsync(request(data) → { 
	sendPageToUser(data, exchange);
});
```
- **Pros** - 
	- Here we only need # treads = # cores.
- **Cons** - 
	- **Callback Hell** - Multiple blocking operations have to chained in function calls.
![[Screenshot 2026-05-07 at 12.37.53 AM.png|500]]
#### Callable Interface
- Allows to run threads which return a value.
```Java
interface Callable<V>{
	public V call();
}
```
#### Future Interface
- represent the result of an asynchronous computation.
- ExecutorService.submit() returns a future object.
- **get()** - Blocking call. Waits on the callable method to finish and return a result.
- **cancel()** - Cancels the future task.
- **isDone()** - Checks if the task is done.
```Java
static ExecutorService threadPool = Executors.newFixedThreadPool(2);
Callable<Integer> sumTask = new Callable<Integer>() {
	public Integer call() throws Exception {
		int sum = 0;
		for (int i = 1; i <= n; i++) sum += i;
		return sum;
	}
};
Future<Integer> f = threadPool.submit(sumTask);
return f.get();
```
#### CompletableFuture
- Suppose you have dependent blocking tasks. And you have to run multiple of them.
```Java
Future<Order> f1 = service.submit(getOrderTask());
Order order = f1.get();
Future<Order> f2 = service.submit(getOrderTask(order));
order = f2.get();
Future<Order> f1 = service.submit(getOrderTask(order));
order = f3.get();

// Method - 1
for(int i=0;i<NUM_TASKS ; i++){
	Future<Order> f1 = service.submit(getOrderTask());
	Order order = f1.get();
	Future<Order> f2 = service.submit(enrich(order));
	order = f2.get();
	Future<Order> f1 = service.submit(processPayment(order));
	order = f3.get();
}
//--> Task 2 cannot start without Task 1 being completed.

// Method - 2
for(int i=0;i<NUM_TASKS ; i++){
	Future<Order> f1 = service.submit(getOrderTask());
	Order order = f1.get();
}
for(int i=0;i<NUM_TASKS ; i++){
	Future<Order> f2 = service.submit(enrich(order));
	order = f2.get();
}
for(int i=0;i<NUM_TASKS ; i++){
	Future<Order> f1 = service.submit(processPayment(order));
	order = f3.get();
}
//--> All tasks will have to wait for each one to complete their stages.
```
- Solution -> Completable Futures. Chains futures. 
- Once we get result of one future, run the next in chain. Dont bother the main thread.
- The same thread is tasked to do all the actions in chain.
```Java
for(int i=0;i<NUM_TASKS ; i++){
	CompletableFuture.supplyAsync(()->getOrderTask())
	.thenApply(order -> enrich(order))
	.thenApply(order -> sendEmail(order))
	.thenAccept(order -> processPayment(order));
}

Methods - 
supplyAsync(Callable);
supplyAsync(Callable, threadPool); // If you want to run in a specific pool.
thenApply(Callable); // Makes the same thread run the next task.
thenApplyAsync(Callable, threadPool); // Run the next task in a new thread.
thenAccept(Callable); // returns value.
exceptionally(e->{}) // Catches all the exceptions in the futures above it.

ExecutorService cpuBound = Executors.newFixedThreadPool(4);
ExecutorService ioBound = Executors.newCachedThreadPool();
CompletableFuture.supplyAsync(()->getOrderTask(), ioBound) // connects to DB hence use IOBound Thread pool.
	.thenApplyAsync(order -> enrich(order), cpuBound) // does not connect to DB so use cpu bound. blocking 
	.thenApply(order -> sendEmail(order)) // Subsequent are also not blocking so use cpu bound.
	.exceptionally(e-> { .... , new Exception()}) // Catch all exceptions.
	.thenAccept(order -> processPayment(order));

```
- 
## Virtual Threads
- When you create a thread, it is associated with a OS level thread which are limited in number.
- **Virtual Threads** - Is owned and managed by the JVM. Faster to create and manage.
	- Essentially Virtual Threads are objects.
	- JVM internally has a pool of OS threads and runs virtual threads on it.
	- Once a virtual thread is run on the OS thread, it is marked for garbage collection.
	- *When a virtual thread goes to sleep/performs blocking, JVM unmounts it from OS thread and runs other tasks.* -> **OS Threads are not blocked**
	- *Virtual Thread wakes up* -> **Mounted on a different OS Thread and run.**
- **Virtual threads are only useful in IO Bound Threads** - OS Threads are unblocked this way -> Non blocking IO.
- Instead of context switching, JVM manually switches the tasks it runs through the thread. 
- So within one run of OS thread, multiple Virtual threads are run.
```Java
ExecutorService service = Executors.newVirtualThreadPerTaskExecutor();
// Here you can submit as many threads as you want without crashing the application.
```
- setDaemon() and setPriority() are not applicable for virtual threads.
![[Screenshot 2026-05-07 at 7.37.32 PM.png|500]]

## Concurrent Collections
- Provide us with thread safe collections. Implemented using Locks, CAS and Copy-on-write.
	- *Copy-on-write* - Create a new copy on write -> Used for read heavy cases.
#### Concurrent HashMap
- Instead of locking the entire map, it locks specific buckets of keys -> better performance
``` Java
ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
map.put(1, "Java");
map.put(2, "Python");
map.put(3, "C++");
System.out.println(map.get(1)); // Java
map.putIfAbsent(2, "Go");
```
#### CopyOnWrite ArrayList
- Thread safe arrayList where new copy is created whenever array is modified.
```Java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("Apple");
list.add("Banana");
for (String fruit : list) {
	System.out.println(fruit);
	list.add("Grapes"); // Won't cause ConcurrentModificationException
}
```
#### BlockingQueue
- Used when one thread produces and other consume it. 
- It blocks the producer when the queue is full and blocks the consumer when it is empty.
- Supports blocking operations:
	- **put():** waits if the queue is full.
	- **take():** waits if the queue is empty.
```Java
ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(2);
queue.put(10);
queue.put(20);
// Will block until space is available
new Thread(() -> {
	try {
		queue.take();
	} catch (Exception e) {}
}).start();
queue.put(30); 
System.out.println(queue);
```
#### ConcurrentSkipListMap
- Thread-safe alternative to TreeMap which maintains elements in sorted order.
```Java
ConcurrentSkipListMap<Integer, String> map= new ConcurrentSkipListMap<>();
map.put(3, "Apple");
map.put(1, "Banana");
map.put(4, "Cherry");
map.put(2, "Mango");
System.out.println("ConcurrentSkipListMap: " + map);
System.out.println("Value for key 2: " + map.get(2));
map.remove(3);
System.out.println("After removing key 3: " + map);
System.out.println("First Entry: " + map.firstEntry());
System.out.println("Last Entry: " + map.lastEntry());
System.out.println("SubMap(2 to 4): " + map.subMap(2, true, 4, true));
```
#### ConcurrentLinkedQueue
- thread-safe, non-blocking queue that uses lock-free algorithms.
```Java
ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
queue.add("Task1");
queue.add("Task2");
System.out.println("Head: " + queue.peek());
System.out.println("Removed: " + queue.poll());
System.out.println("Queue after removal: " + queue);
```
## ThreadLocal\<V>
- Creates a local copy of a object property for each thread.
```Java
class Demonstration {
    public static void main( String args[] ) throws Exception{
        UnsafeCounter usc = new UnsafeCounter();
        Thread[] tasks = new Thread[100];
        for (int i = 0; i < 100; i++) {
            Thread t = new Thread(() -> {
                for (int j = 0; j < 100; j++) usc.increment();
                System.out.println(usc.counter.get());
            });
            tasks[i] = t;
            t.start();
        }
        for (int i = 0; i < 100; i++) {
            tasks[i].join();
        }
        System.out.println(usc.counter.get());
    }
}

class UnsafeCounter {
    ThreadLocal<Integer> counter = ThreadLocal.withInitial(() -> 0);
    void increment() {
        counter.set(counter.get() + 1);
    }
}
```