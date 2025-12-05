#### Threads
- A thread can be thought of as a process which shares its address space with other processes(threads).
- Each thread has its own set of registers, which need to be saved in **Thread Control Blocks**
- However the address space does not change, i.e, we can keep using the same page tale.
- Each thread has its own stack. There are multiple stacks on memory, so there is some difficulty in arranging them in address space. But since stacks do not grow a lot in size, it can be accomodated.
```C
#include <stdio.h>
#include <pthread.h>
#include "mythreads.h"

static volatile int counter = 0;
void *mythread(void *arg){
    printf("%s: begin\n", (char *) arg);
    int i;
    for (i = 0; i < 1e7; i++) counter = counter + 1;
    printf("%s: done\n", (char *) arg);
    return NULL;
}

int main(int argc, char *argv[]){
    pthread_t p1, p2;
    printf("main: begin (counter = %d)\n", counter);
    Pthread_create(&p1, NULL, mythread, "A");
    Pthread_create(&p2, NULL, mythread, "B");
    // join waits for the threads to finish
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("main: done with both (counter = %d)\n", counter);
    return 0;
}

```
#### Race Condition
- When two threads try to access the same piece of code and update the same variables, it may lead to indeterministic behaviour.
- This happens because while one thread is updating the values, it is switched out before it could save the value to memory.
In the above example,
add is a 3 step process. First you load up the variable in a register, then you modify it and then you save the updated value.
So if the thread is switched out before it can save the updated value and another thread updates the value, when the first thread resumes, the value update will trigger again. So essentially, 2 updates were performed but the value stored reflects just one.
- To avoid race condition -> Atomic Operations(None or all, i.e., completed in one go).
  -> Use Synchronisation primitives like locks.
#### Critical Section
- Piece of code that accesses a shared resource either a variable or a data structure.
#### Mutual Exclusion
- Ensuring mutual exclusion makes sure that only one thread accesses the critical section at a time and no race condition occurs
## Locks
- It is a variable of some kind. It is used to secure the critical section so that it can be executed atomically, i.e, only one thread executes it at a time.
- A lock can be in two states, **available** - If no thread holds the lock OR **acquired** - If a thread holds the lock
- **lock()** - If no thread has acquired the lock, the calling thread can enter the critical section and become owner of the lock. Else if a thread has acquired the lock, the calling thread will wait until the unlock() is called.
### Evaluating Locks
Locks are evaluated based on the following metrics -
- **Mutual Exclusion** - Locks must provide mutual exclusion
- **Fairness** - Locks must ensure each thread has an equal opportunity to access the lock once unlock is called.
- **Performance** - Overheads when acquiring locks.
### 1. Disabling Interrupts
- If a thread is executing the critical section, the OS will not process any interrupts which would stop the execution.
- Works for single processor systems.
```C
void lock() {
    DisableInterrupts();
}
void unlock() {
    EnableInterrupts();
}
```
##### Cons
- Allows threads to perform privileged instruction. A greedy program could monopolize the CPU be capturing the lock at execution or it could go into an endless loop where OS does not get to run at all.
- Works only for single processor systems.
- Turning of interrupts -> Interrupts will get lost. Eg- If a read is complete while interrupts are disabled, CPU will not know.
- Inefficient as enabling and disabling interrupts are heavy tasks.
### 2. Petersen's Algo
- This is a software based lock.
```C
int flag[2];
int turn;
void init() {
	flag[0] = flag[1] = 0; // 1->thread wants to grab lock
	turn = 0; // whose turn? (thread 0 or 1?)
}
void lock() {
	flag[self] = 1; // self: thread ID of caller
	turn = 1 - self; // make it other thread’s turn
	while ((flag[1-self] == 1) && (turn == 1 - self))
		; // spin-wait
	}
]
void unlock() {
	flag[self] = 0; // simply undo your intent
}
```
#### 3. Just Using a Flag -> **Does not work**
```C
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *mutex) {
    // 0 -> lock is available, 1 -> held
    mutex->flag = 0;
}

void lock(lock_t *mutex) {
    while (mutex->flag == 1) // TEST the flag
        ; // spin-wait (do nothing)
    mutex->flag = 1; // now SET it!
}

void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```
- There is no mutual exclusion here.
  mutex->flag = 0;
  T1 executes -> lock() -> checks whether (mutex->flag == 1) = false -> Lock acquired -> loop exited
  Before mutex->flag is set to 1 -> Timer Interrupt 
  T2 executes -> lock() -> Even though T1 acquired the lock, flag = 0 -> T2 acquires the lock, flag = 1.
  In middle of critical section execution -> Timer Interrupt
  T1 executes -> mutex.flag is set to one => 2 Threads have simultaneously acquired the lock.
### 3. Test and Set
- Test and set(atomic exchange) is an **atomic** hardware instruction, which takes in a pointer and a value. Updates the pointer with that value and then returns the old value.
```C
int TestAndSet(int *old_ptr, int new_val) {
    int old = *old_ptr; // fetch old value at old_ptr
    *old_ptr = new_val; // store ’new_val’ into old_ptr
    return old; // return the old value
}
typedef struct __lock_t {
    int flag;
} lock_t;
void init(lock_t *lock) {
    lock->flag = 0;    // 0 indicates that lock is available, 1 that it is held
}
void lock(lock_t *lock) {
    while (TestAndSet(&lock->flag, 1) == 1); // spin-wait (do nothing)
}
void unlock(lock_t *lock) {
    lock->flag = 0;
}
```
- When lock is unacquired, flag = 0; T1 executes -> lock)() -> Test&Set rets 0, flag=1 -> crit sec executes.
  T2 executes -> lock() -> Test&Set rets 1, flag=1 -> spin wait. => Mutex is ensured.
### 4. Compare and Swap
- Another atomic operation which checks if the value at a pointer is equal to expected. If it is it updates it. It returns the old value.
```C
int CompareAndSwap(int *ptr, int expected, int new_val) {
    int actual = *ptr;
    if (actual == expected)
        *ptr = new_val;
    return actual;
}
void lock(lock_t *lock) {
    while (CompareAndSwap(&lock->flag, 0, 1) == 1)
        ; // spin
}
```
- T1 executes -> lock() -> flag is 0, Comp&Swap rets 0, flag set to 1 -> Lock acquired. Crit section executes
  T2 executes -> lock() -> flag is 1, Comp&Swap rets 1, flag remains 1 -> spin wait.
### 5. Load Linked and Store Conditional
```C
int LoadLinked(int *ptr) {
    return *ptr;
}

int StoreConditional(int *ptr, int value) {
    if (no one has updated *ptr since the LoadLinked to this address) {
        *ptr = value;
        return 1; // success!
    } else {
        return 0; // failed to update
    }
}
void lock(lock_t *lock) {
    while (1) {
        while (LoadLinked(&lock->flag) == 1)
            ; // spin until it’s zero
        if (StoreConditional(&lock->flag, 1) == 1)
            return; // if set-it-to-1 was a success: all done
        // otherwise: try it all over again
    }
}

void unlock(lock_t *lock) {
    lock->flag = 0;
}
```
- Load Linked stores a reservation for the given address and returns the value.
- Store Conditional stores the value if and only if 
  -> No other core has modified the address 
  -> No context switch invalidated the reservation
- flag =0. T1 executes -> LoadLinked rets 0 -> StoreConditional sets flag = 1 -> Lock acquired.
  T2 executes -> LoadLinked rets 1 -> spin wait.
- flag=0. T1 executes -> LL is called and rets 0 -> Context switches.
  T2 executes -> LL is called and rets 0 -> SC is called but it fails as flag reservation is updated. -> Context switches
  T1 executes -> SC is called and lock acquired.
### 6. Fetch and Add
```C
int FetchAndAdd(int *ptr) {
    int old = *ptr;
    *ptr = old + 1;
    return old;
}

typedef struct __lock_t {
    int ticket;
    int turn;
} lock_t;

void lock_init(lock_t *lock) {
    lock->ticket = 0;
    lock->turn = 0;
}

void lock(lock_t *lock) {
    int myturn = FetchAndAdd(&lock->ticket);
    while (lock->turn != myturn)
        ; // spin
}

void unlock(lock_t *lock) {
    lock->turn = lock->turn + 1;
}
```
- ticket=0,turn=0 -> T1 executes -> lock() -> FetchAndAdd rets 0, ticket=1, turn=0, myTurn=0 -> Lock acquired -> Context Switch
  ticket=1,turn=0 -> T2 executes -> lock() -> FetchAndAdd rets 1, ticket = 2, turn = 1, myTurn = 1 -> Spin wait
  T1 calls unlock() -> turn = 1;
  T2 executes -> turn = 1 and myTurn = 1 -> Lock acquired.

## Evaluating Spin Locks
- **Mutex** - ensured
- **Fairness** - There no fairness guarantee as a thread holding the lock could go on forever.
- **Performance** - In a single processor system, it is very bad. If a thread acquires the lock, all other threads may be waiting and be run by the scheduler causing wasted CPU cycles. On multiprocessor systems it works reasonably well.
- If N threads are running, N-1 will wait while spinning.
### 1. Yielding
- The spinning thread instead of waiting can just call yield()
- **yield()** - moves the process from running to ready state.
- Still costly as the (N-1) threads will still call the lock and yield procedure.
### 2. Using Queues
```C
typedef struct __lock_t {
    int flag;
    int guard;
    queue_t *q;
} lock_t;

void lock_init(lock_t *m) {
    m->flag = 0;
    m->guard = 0;
    queue_init(m->q);
}

void lock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1)
        ; //acquire guard lock by spinning
    if (m->flag == 0) {
        m->flag = 1; // lock is acquired
        m->guard = 0;
    } else {
        queue_add(m->q, gettid());
        //setpark();
        m->guard = 0;
        park();
    }
}

void unlock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1)
        ; //acquire guard lock by spinning
    if (queue_empty(m->q))
        m->flag = 0; // let go of lock; no one wants it
    else
        unpark(queue_remove(m->q)); // hold lock (for next thread!)
    m->guard = 0;
}
```
- flag=0, guard=0.
  T1 executes -> Test&Set(guard) rets 1(guard=1) -> guardLock acquired -> flag=0 so lock is acquired -> flag=1, guard=0
  T2 executes -> Test&Set(guard) rets 1(guard=1) -> guardLock acquired -> flag=1 -> thread added to queue. -> guard=0 -> park();
- If T2 executes when guardLock is acquired by T1, all other threads will spin until T1 executes and releases the lock.
- *There is some spinning involved here but still its acceptable as number of operations within lock is very low.*
- park() -> Thread will sleep until the lock is no longer acquired.
- flag does not get set back to 0. This is because a thread that gets woken up will resume from park(). Thus we just pass the lock from the thread releasing the lock to the woken up thread.
##### setpark()
If we context switch just before park() to a thread holding the lock and it releases it. When the first thread executes, it would call park() and continue to sleep forever. => **wakeup/waiting race**
  -> setpark() -> indicates that a thread is about to park. If another thread calls the unpark(), park() would just return and not put the process to sleep.
### 2 Phase Locks
- Spinning can be useful if the lock is about to be released as park and unpark can be costly.
- So in these locks, threads spin for a while and then are put to sleep.
## Lock based Data Structures
### Sync Counter
- The simple approach would be to wrap each operation(inc, dec and get) with a lock. -> **Costly performance wise**
- To increment 1M times, 1 thread ->0.3s, 2threads ->5s.
#### Sloppy Counter
- Each core has its own copy of simple sync counter and there is a global sync counter.
- Each thread running on a core will increment the local counter. When the value of **counter > S(sloppiness)** -> acquire the global lock and update the global counter += S.
- low S -> behaves like sync counter. high S -> accuracy of get decreases.
```C
typedef struct __counter_t {
    int global;         // global count
    pthread_mutex_t glock; // global lock
    int local[NUMCPUS]; // local count (per cpu)
    pthread_mutex_t llock[NUMCPUS]; // ... and locks
    int threshold;      // update frequency
} counter_t;

void init(counter_t *c, int threshold) {
    c->threshold = threshold;
    c->global = 0;
    pthread_mutex_init(&c->glock, NULL);
    int i;
    for (i = 0; i < NUMCPUS; i++) {
        c->local[i] = 0;
        pthread_mutex_init(&c->llock[i], NULL);
    }
}

void update(counter_t *c, int threadID, int amt) {
    int cpu = threadID % NUMCPUS;
    pthread_mutex_lock(&c->llock[cpu]);
    c->local[cpu] += amt; 
    if (c->local[cpu] >= c->threshold) { // transfer to global
        pthread_mutex_lock(&c->glock);
        c->global += c->local[cpu];
        pthread_mutex_unlock(&c->glock);
        c->local[cpu] = 0;
    }
    pthread_mutex_unlock(&c->llock[cpu]);
}

int get(counter_t *c) {
    pthread_mutex_lock(&c->glock);
    int val = c->global;
    pthread_mutex_unlock(&c->glock);
    return val; // only approximate!
}
```
### Concurrent Linked Lists
- We have to cover the entire insert and read operation in locks.
- Just the malloc portion should be skipped as if it fails we will have to release the lock again. If wrapped inside the lock, it would lead to two ways to release the lock which might be more error prone. So we dont wrap malloc inside lock.
```C
typedef struct __node_t {
    int key;
    struct __node_t *next;
} node_t;
typedef struct __list_t {
    node_t *head;
    pthread_mutex_t lock;
} list_t;

void List_Init(list_t *L) {
    L->head = NULL;
    pthread_mutex_init(&L->lock, NULL);
}
int List_Insert(list_t *L, int key) {
    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        perror("malloc");
        return -1; // fail
    }
    pthread_mutex_lock(&L->lock);
    new->key = key;
    new->next = L->head;
    L->head = new;
    pthread_mutex_unlock(&L->lock);
    return 0; // success
}
int List_Lookup(list_t *L, int key) {
    pthread_mutex_lock(&L->lock);
    node_t *curr = L->head;
    while (curr) {
        if (curr->key == key) {
            pthread_mutex_unlock(&L->lock);
            return 0; // success
        }
        curr = curr->next;
    }
    pthread_mutex_unlock(&L->lock);
    return -1; // failure
}
```
#### Hand-over-hand locking/Lock coupling
- Instead of a single lock for entire list. There is one lock for each node. When traversing the list, the thread must acquire the next lock before releasing the current. 
- It ensures a higher degree of concurrency but is bad performance wise.
### Concurrent Queues
- There is one lock each for head and tail.
```C
typedef struct __node_t {
    int value;
    struct __node_t *next;
} node_t;

typedef struct __queue_t {
    node_t *head;
    node_t *tail;
    pthread_mutex_t headLock;
    pthread_mutex_t tailLock;
} queue_t;

void Queue_Init(queue_t *q) {
    node_t *tmp = malloc(sizeof(node_t));
    tmp->next = NULL;
    q->head = q->tail = tmp;
    pthread_mutex_init(&q->headLock, NULL);
    pthread_mutex_init(&q->tailLock, NULL);
}

void Queue_Enqueue(queue_t *q, int value) {
    node_t *tmp = malloc(sizeof(node_t));
    assert(tmp != NULL);
    tmp->value = value;
    tmp->next = NULL;

    pthread_mutex_lock(&q->tailLock);
    q->tail->next
}
```
### Concurrent Hash Tables
- Each bucket is implemented with its own concurrent linked list and has its own lock. Highly performant.
```C
#define BUCKETS (101)
typedef struct __hash_t {
    list_t lists[BUCKETS];
} hash_t;

void Hash_Init(hash_t *H) {
    int i;
    for (i = 0; i < BUCKETS; i++) {
        List_Init(&H->lists[i]);
    }
}

int Hash_Insert(hash_t *H, int key) {
    int bucket = key % BUCKETS;
    return List_Insert(&H->lists[bucket], key);
}

int Hash_Lookup(hash_t *H, int key) {
    int bucket = key % BUCKETS;
    return List_Lookup_
}
```

## Condition Variables
- A condition variable is an explicit queue that threads can put themselves on when some state of execution is not as desired or some condition is not fulfilled.
- When the state changes then it wakes up one of the waiting threads and allows them to continue.
- wait() -> puts the thread to sleep in the queue. Unlocks the mutex and puts thread to sleep atomically.
  signal() -> wakes up one of the sleeping threads. 
- Both of them take in an **acquired lock** as a parameter -> Endures that another thread using the same lock does not call signal()/wait() on it.
- *Hold the lock when signal() or wait()*
  *Always use while condition to check on the shared variable when using wait()*
#### Making child run before parent
```C
#include <stdio.h>
#include <pthread.h>

int done = 0;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t c = PTHREAD_COND_INITIALIZER;
void thr_exit() {
    pthread_mutex_lock(&m);
    done = 1;
    pthread_cond_signal(&c);
    pthread_mutex_unlock(&m);
}
void *child(void *arg) {
    printf("child\n");
    thr_exit();
    return NULL;
}
void thr_join() {
    pthread_mutex_lock(&m);
    while (done == 0) {
        pthread_cond_wait(&c, &m);
    }
    pthread_mutex_unlock(&m);
}
int main(int argc, char *argv[]) {
    printf("parent: begin\n");
    pthread_t p;
    pthread_create(&p, NULL, child, NULL);
    thr_join();
    printf("parent: end\n");
    return 0;
}
```
##### Why use while to check condition on done and not usual if ?
- If we use if, suppose the parent is running. It checks the condition and done = 0. 
  Now wait() is called -> P sleeps.
  C executes -> done = 1. -> signal()
  P executes -> **assumes everything is fine** and goes on without checking the condition.
- There are cases where CVs allow multiple threads to be awoken at the same time. That is why it is always advised **to use while when wait()** is called for a CV.
### Producer Consumer/ Bounded Buffer Problem
- There is a buffer of limited size(N). Multiple producers will try to add an entry to the buffer while the consumers will try to remove an entry.
#### Case 1 -> N=1
```C
int buffer;
int count = 0; // 0 = empty, 1 = full

void put(int value) {
    assert(count == 0); // must be empty
    count = 1;          // now full
    buffer = value;
}

int get() {
    assert(count == 1); // must be full
    count = 0;          // now empty
    return buffer;
}
```
##### Put a lock on the producer and consumer code.
```C
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        Pthread_mutex_lock(&mutex);      
        while (count == 1)
            Pthread_cond_wait(&cond, &mutex);
        put(i);
        Pthread_cond_signal(&cond);
        Pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < loops; i++) {
        Pthread_mutex_lock(&mutex);
        while (count == 0)
            Pthread_cond_wait(&cond, &mutex);
        int tmp = get();
        Pthread_cond_signal(&cond);
        Pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
    return NULL;
}
```
- Suppose there is 1P and 2Cs. C1 runs, count = 0 -> C1 sleeps.
  P runs, count = 0 -> count = 1, signal() -> C1 wakes up. 
  Now before C1 can consume the value, C2 runs and consumes the value. count = 0.
  C1 runs -> count = 0 even though it was signalled that the buffer is full.
- Problem is Producers wait for buffer emptiness and Consumers wait for buffer fullness.
##### Using 2 CVs
```C
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t full = PTHREAD_COND_INITIALIZER; // init to 0
pthread_cond_t empty = PTHREAD_COND_INITIALIZER; // init to 1

void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        Pthread_mutex_lock(&mutex);      
        while (count == 1)
            Pthread_cond_wait(&empty, &mutex);
        put(i);
        Pthread_cond_signal(&full);
        Pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < loops; i++) {
        Pthread_mutex_lock(&mutex);
        while (count == 0)
            Pthread_cond_wait(&full, &mutex);
        int tmp = get();
        Pthread_cond_signal(&empty);
        Pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
    return NULL;
}
```
#### N is not 1
```C
int buffer[MAX];
int fill_ptr = 0;
int use_ptr = 0;
int count = 0;

void put(int value) {
    buffer[fill_ptr] = value;
    fill_ptr = (fill_ptr + 1) % MAX;
    count++;
}
int get() {
    int tmp = buffer[use_ptr];
    use_ptr = (use_ptr + 1) % MAX;
    count--;
    return tmp;
}
pthread_cond_t empty;
pthread_cond_t fill;
pthread_mutex_t mutex;
void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        Pthread_mutex_lock(&mutex);          // p1
        while (count == MAX)                 // p2
            Pthread_cond_wait(&empty, &mutex); // p3
        put(i);                              // p4
        Pthread_cond_signal(&fill);          // p5
        Pthread_mutex_unlock(&mutex);        // p6
    }
    return NULL;
}
void *consumer(void *arg) {
    for (int i = 0; i < loops; i++) {
        Pthread_mutex_lock(&mutex);          // c1
        while (count == 0)                   // c2
            Pthread_cond_wait(&fill, &mutex);  // c3
        int tmp = get();                     // c4
        Pthread_cond_signal(&empty);         // c5
        Pthread_mutex_unlock(&mutex);        // c6
        printf("%d\n", tmp);
    }
    return NULL;
}
```

### Covering Conditions
- Suppose we are writing a program to allocate memory. 
- Now if 2 threads are requesting memory of size 100, 10. We have 5 units of memory available so both will go to sleep.
- Now if free(50) called. signal() -> might wake up 100 units thread, which not right.
- Instead we broadcast() and wake up all the threads. They check the condition of free memory and 10 units thread will get the memory in this case.
- This type of CVs are called Covering Conditions.
## Semaphores
- It is an object with an integer in it whose value can be manipulated with 2 methods.
- **sem_wait(s)** - s.value = s.value-1; If s.value > 0 -> return; else -> sleep()
- **sem_post(s)** - s.value = s.value+1; Wakes a sleeping thread if any.
### Binary Semaphores(Semaphores as Locks)
```C
sem_t m;
sem_init(&m, 0, 1);
sem_wait(&m);
  // critical section here
sem_post(&m);
```
- T1 executes calls sem_wait() -> m.val = 0. -> context switch
  T2 executes calls sem_wait(), m.val == 0 -> Goes to sleep.
  T1 executes -> calls sem_post() -> m.val = 1 -> Wakes T2. T2 executes.
### Semaphores as Condition Variables
```C
sem_t s;

void *child(void *arg) {
    printf("child\n");
    sem_post(&s);   // signal: child is done
    return NULL;
}

int main(int argc, char *argv[]) {
    sem_init(&s, 0, 0);   // X = 0
    printf("parent: begin\n");

    pthread_t c;
    pthread_create(&c, NULL, child, NULL);

    sem_wait(&s);   // parent waits for child
    printf("parent: end\n");
    return 0;
}
```
- If P executes first, s->val = 0 => Sleep.
  C executes -> sem_post() -> s.val=1, P executes.
#### Producer Consumer Problem with Semaphores
##### Approach 1 -> Use 2 Semaphores
```C
int buffer[MAX];
int fill = 0;
int use = 0;
void put(int value) {
    buffer[fill] = value;        // f1
    fill = (fill + 1) % MAX;     // f2
}
int get() {
    int tmp = buffer[use];       // g1
    use = (use + 1) % MAX;       // g2
    return tmp;
}
sem_t empty;
sem_t full;
void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        sem_wait(&empty);  // wait for space
        put(i);            // put item in buffer
        sem_post(&full);   // signal item available
    }
    return NULL;
}
void *consumer(void *arg) {
    int tmp = 0;
    while (tmp != -1) {
        sem_wait(&full);   // wait for item
        tmp = get();       // get item from buffer
        sem_post(&empty);  // signal space available
        printf("%d\n", tmp);
    }
    return NULL;
}
```
- C1 runs -> sem_wait(full to -1) -> puts to sleep
  P1 runs -> sem_wait(empty to 0) -> executes and puts value to buffer. -> sem_post(full) -> C1 wakes up.
  C1 runs -> sem_wait(full to 0) -> consumes data.
- But suppose there are 2 producers, and both try to produce at the same time.(MAX>1)
  P1 -> sem_wait(empty -> 1) -> put() -> puts to buffer[fill] -> before *f2* -> context switch.
  P2 -> sem_wait(empty -> 0) -> put() -> puts to buffer[fill]
  So two producers put at the same index.
- We forgot mutex locks.
##### Adding Mutex Locks
```C
void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        sem_wait(&mutex);  
        sem_wait(&empty);  // wait for space
        put(i);            // put item in buffer
        sem_post(&full);   // signal item available
        sem_post(&mutex);   
    }
    return NULL;
}
void *consumer(void *arg) {
    int tmp = 0;
    while (tmp != -1) {
        sem_wait(&mutex);  
        sem_wait(&full);   // wait for item
        tmp = get();       // get item from buffer
        sem_post(&empty);  // signal space available
        sem_post(&mutex);   
        printf("%d\n", tmp);
    }
    return NULL;
}
```

- **Deadlocks** - There may be a condition where a consumer is holding mutex and is waiting for full and a producer is holding full and is waiting on mutex. This is a problem as both of the processes cannot progress.
##### Avoiding Deadlock.
```C
void *producer(void *arg) {
    for (int i = 0; i < loops; i++) {
        sem_wait(&empty);  // wait for space
        sem_wait(&mutex);  
        put(i);            // put item in buffer
        sem_post(&mutex);   
        sem_post(&full);   // signal item available
    }
    return NULL;
}
void *consumer(void *arg) {
    int tmp = 0;
    while (tmp != -1) {
        sem_wait(&full);   // wait for item
        sem_wait(&mutex);  
        tmp = get();       // get item from buffer
        sem_post(&mutex);   
        sem_post(&empty);  // signal space available
        printf("%d\n", tmp);
    }
    return NULL;
}
```
- The critical section is just the get() and put() codes and not the acquiring of full and empty locks.
#### Reader-Writer Locks
- In the reader writer system we don't need to lock up the code of reader and writer. We can allow multiple readers to access data concurrently as long as the writer is not running.
```C
typedef struct _rwlock_t {
    sem_t lock;        // binary semaphore (basic lock)
    sem_t writelock;   // allows ONE writer or MANY readers
    int readers;       // number of active readers
} rwlock_t;

void rwlock_init(rwlock_t *rw) {
    rw->readers = 0;
    sem_init(&rw->lock, 0, 1);
    sem_init(&rw->writelock, 0, 1);
}

void rwlock_acquire_readlock(rwlock_t *rw) {
    sem_wait(&rw->lock);
    rw->readers++;
    if (rw->readers == 1)
        sem_wait(&rw->writelock); // first reader blocks writers
    sem_post(&rw->lock);
}

void rwlock_release_readlock(rwlock_t *rw) {
    sem_wait(&rw->lock);
    rw->readers--;
    if (rw->readers == 0)
        sem_post(&rw->writelock); // last reader allows writers
    sem_post(&rw->lock);
}

void rwlock_acquire_writelock(rwlock_t *rw) {
    sem_wait(&rw->writelock);
}

void rwlock_release_writelock(rwlock_t *rw) {
    sem_post(&rw->writelock);
}
```
- They often add more overhead (especially with more sophisticated implementations), and thus do not end up speeding up performance as compared to just using simple and fast locking primitives
  
## Dining Philosophers
- 5 philosophers sitting and have 1 fork between each of them.
- Each philosopher thinks and then eats so there will be times when the use no forks or use both forks to eat.
```C
while (1) {
	think();
	getforks();
	eat();
	putforks();
}
int left(int p) { return p; }
int right(int p) { return (p + 1) % 5; }

void getforks() {
    sem_wait(forks[left(p)]);
    sem_wait(forks[right(p)]);
}
void putforks() {
    sem_post(forks[left(p)]);
    sem_post(forks[right(p)]);
}
```
- Problem here is if all the philosophers acquire forks in the same way, it might be possible that all the philosophers acquire their left forks and are all stuck waiting for right forks. -> **Deadlock**
- **Solution** -> Change one philosopher's orientation. Make it acquire right then left.
## Zemaphores - Implementing Semaphores with CVs and Locks
```C
typedef struct __Zem_t {
    int value;
    pthread_cond_t cond;
    pthread_mutex_t lock;
} Zem_t;

// only one thread can call this
void Zem_init(Zem_t *s, int value) {
    s->value = value;
    pthread_cond_init(&s->cond, NULL);
    pthread_mutex_init(&s->lock, NULL);
}

void Zem_wait(Zem_t *s) {
    pthread_mutex_lock(&s->lock);
    while (s->value <= 0)
        pthread_cond_wait(&s->cond, &s->lock);
    s->value--;
    pthread_mutex_unlock(&s->lock);
}

void Zem_post(Zem_t *s) {
    pthread_mutex_lock(&s->lock);
    s->value++;
    pthread_cond_signal(&s->cond);
    pthread_mutex_unlock(&s->lock);
}
```
## Common Concurrency Problems
### Non Deadlock Bugs
#### Atomicity Violation Bugs
- Code sections which need to be atomic are not because of missing locks.
~~~C
// Thread 1
if (thd->proc_info) {
    ...
    fputs(thd->proc_info, ...);
    ...
}
// Thread 2
thd->proc_info = NULL;

-----------------------

pthread_mutex_t proc_info_lock = PTHREAD_MUTEX_INITIALIZER;

//Thread 1::
pthread_mutex_lock(&proc_info_lock);
if (thd->proc_info) {
    ...
    fputs(thd->proc_info, ...);
    ...
}
pthread_mutex_unlock(&proc_info_lock);

//Thread 2::
pthread_mutex_lock(&proc_info_lock);
thd->proc_info = NULL;
pthread_mutex_unlock(&proc_info_lock);

~~~
- T1 runs -> after checking proc_info is not null and before fputs -> context switch
  T2 runs -> makes proc_info NULL.
  T1 runs ->  calls fputs -> errors out.
#### Order Violation Bug
- Two sections which need to be executed in a fixed order are not executed in that order.
~~~C
// Thread 1
void init() {
    ...
    mThread = PR_CreateThread(mMain, ...);
    ...
}

// Thread 2
void mMain(...) {
    ...
    mState = mThread->State;
    ...
}

-------------------------------

pthread_mutex_t mtLock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t mtCond = PTHREAD_COND_INITIALIZER;
int mtInit = 0;

Thread 1::
void init() {
    ...
    mThread = PR_CreateThread(mMain, ...);

    // signal that the thread has been created...
    pthread_mutex_lock(&mtLock);
    mtInit = 1;
    pthread_cond_signal(&mtCond);
    pthread_mutex_unlock(&mtLock);
    ...
}

Thread 2::
void mMain(...) {
    ...
    // wait for the thread to be initialized...
    pthread_mutex_lock(&mtLock);
    while (mtInit == 0)
        pthread_cond_wait(&mtCond, &mtLock);
    pthread_mutex_unlock(&mtLock);

    mState = mThread->State;
    ...
}
~~~
### Deadlock Bugs
- They occur when T1 is holding lock L1 waiting for L2 while T2 is holding L2 waiting for L1.
#### Why do Deadlocks occur?
1. Large systems have a lot of complexity due to which there are circular dependencies and locking must be done strategically.
2. **Encapsulation** - As details of how locks are acquired is encapsulated, using encapsulated libraries may lead to circular dependencies as the dev does not know about the order of acquiring locks.
~~~C
Vector v1, v2;
v1.AddAll(v2);

Internally, because the method needs to be multi-thread safe, locks for both the vector being added to (v1) and the parameter (v2) need to be acquired. The routine acquires said locks in some arbitrary order (say v1 then v2) in order to add the contents of v2 to v1. If some other thread calls v2.AddAll(v1) at nearly the same time, we have the potential for deadlock, all in a way that is quite hidden from the calling application.
~~~
#### Conditions for Deadlocks
Four conditions need to hold for a deadlock to occur.
1. **Mutual exclusion**
2. **Hold-and-wait**: Threads hold resources allocated to them while waiting for additional resources
3. **No preemption**: Resources (e.g., locks) cannot be forcibly removed from threads that are holding them.
4. **Circular wait**: There exists a circular chain of threads such that each thread holds one or more resources (e.g., locks) that are being requested by the next thread in the chain.
#### Preventions of Deadlocks
##### Circular Wait
- **Total Ordering** - Provide an order in which the locks should be acquired for programmers writing code.
- **Partial Ordering** - For complex systems, partial orders also work as there may be locks which are totally unrelated.
- **Lock Address Ordering** - Acquire locks in the order of their addresses. This way you would never have deadlocks.
##### Hold and Wait
- Can be avoided by making lock grabbing atomic by using a prevention lock.
```C
lock(prevention);
lock(L1);
lock(L2);
...
unlock(prevention);
````
- By first grabbing the lock prevention, this code guarantees that no untimely thread switch can occur in the midst of lock acquisition and thus deadlock can once again be avoided.
##### No Preemption
- **trylock()** - It is a method which acquires a lock if available else returns -1;
```C
top:
lock(L1);
if (trylock(L2) == -1) {
    unlock(L1);
    goto top;
}
````
- **Livelock** - trylock() gives rise to Livelocks where even through resources are available, threads are not progressing.
- T1 has L1 and T2 has L2. As T1 cant acquire L2 it releases L1 and at the same time T2 cant acquire L1 so it releases L2.
- *Smiple Solution* - Add a time lag before trying to acquire again.
  But even then livelocks may occur due to different threads executing their code at different times.
- trylock() also has to deal with the issues arising due to encapsulation.
##### Mutual Exclusion
- to avoid the need for mutual exclusion at all. Using powerful hardware instructions, you can build data structures in a manner that does not require explicit locking.
##### Deadlock prevention through Scheduling
- T1 T2 T3 T4
L1 yes yes no no
L2 yes yes yes no
- Scheduler can ensure T1 and T2 never run together then deadlocks would never occur.
- However this considerably lengthens the amount of time required to run the processes as T1 and T2 need to execute sequentially
##### Detect and Recover
- Take action when deadlocks occur. A deadlock detector may be employed which runs periodically and checks if a deadlock cycle is formed. If formed, the system will be restarted.