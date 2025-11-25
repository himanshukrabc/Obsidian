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
