---
title: Concurrent Programming
type: docs
---

## Concurrency Models

- **Process-based concurrency**: Individual processes with private address space and logical control flow
  - Benefits: Isolation, stability, and security.
  - Drawbacks: High overhead in context switching, hard to share data. Inter-process communication: **OS mechanisms (pipes, message queues, shared memory)**
- **Thread-based concurrency**: Independent threads with shared code and memory - smallest unit of scheduling
  - Benefits: Lightweight, faster context switching, easier data sharing.
  - Drawbacks: Risk of race conditions. **Synchronization is required.**
- **Coroutine-based concurrency**: Program-level concurrency model not requiring OS scheduling
  - Benefits: Easier to read and maintain.
  - Drawbacks: Limited concurrency (no parallelism).
- **Event-based concurrency**: Using `select` to respond to descriptors with pending inputs (events)
  - Benefits: Efficient resource usage, simpler concurrency model.
  - Drawbacks: Complex system design, hard to debug.


## Multi-threading Programs

- Lifecycle of threads:
  - Thread creation: only a single argument can be passed to the function. Dynamically allocated memory may be used to pass additional arguments.
  - Detached threads will be automatically reaped on termination. Otherwise, the parent thread must use `join()` to reap the finished child. That is to say, **either `detach()` or `join()` must be called on all threads**.
- Shared data:
  - **Global variables and local static variables** are shared among peer threads.
  - `errno` is kept thread-local - each thread has its own copy.
- Synchronization and race condition solutions:
  - **Mutexes**: mutual exclusion on critical sections.
  - **Semaphores**: signaling/simple data sharing between threads.
  - **Condition variables**: allow threads to wait for certain conditions to be met - release the mutex, sleep until waken up by the condition variable, and re-acquire the mutex
  - **Read-write locks**: allow multiple readers or exclusive access for writers.
  - **Atomic operations**: make operations atomic to avoid unexpected behavior.
- Examples of multi-threading programs:
  - **Producer-consumer problem**: a mutex and two semaphores for a queue (buffer, empty slots, full slots)
    - This model can be used for pre-threading systems: threads consume tasks produced by producers
  - **Reader-writer problem**: a reader count and two mutexes (reader count, writing)
    - Other solutions: reader-writer locks
    - Starvation is possible: fair R/W locks may mitigate this

Example of using condition variable: print 1-100 alternatively with two threads

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

int turn = 1; // current number

void* print_numbers(void* arg) {
    int parity = *(int*)arg;  // 0 for even, 1 for odd

    while (1) {
        pthread_mutex_lock(&lock);

        // wait until it's this thread's turn or finished
        while (turn <= N && (turn % 2 != parity)) {
            pthread_cond_wait(&cond, &lock);
        }

        if (turn > N) {
            pthread_mutex_unlock(&lock);
            pthread_cond_broadcast(&cond);
            break;
        }

        printf("Thread-%d: %d\n", parity, turn);
        turn++;

        pthread_cond_broadcast(&cond);
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}
```
