# Paper 1 - Disruptor: High Performance Alternative To Bounded Queues For Exchanging Data Between Concurrent Threads

## Notes

### 1. Abstract

1. __Problem__: Latency costs of traditional queuing logic is extremely high - at the cost of IO operations i.e. dramatically slow.
2. __Mechanical Sympathy__: Hardware and software working together in harmony.
   1. Design practices with strong focus on tearing apart concerns.
3. Disruptor is a general purpose mechanism to help with concurrent exchange of data using a queuing mechanism.
4. The goal is:
   1. Less write contention.
   2. Lower concurrency overhead.
   3. More cache friendly.
   4. Lower latency, jitter and higher throughput.

### 3. Complexities of Concurrency
1. Concurrency
   1. 2 or more tasks happening in parallel.
   2. Contended access to resources.
2. Concurrent Execution
   1. __Mutual Exclusion__: Contended updates.
   2. __Visibility of Change__: Change is made visible to other threads.
3. Most costly concurrent operations are those with write contention.
4. __Cost of Locks__
   1. Locks provide mutual exclusion and ensure that visibility of change occurs in an ordered manner.
   2. Locks are expensive because of the need of arbitration by the kernel.
   3. Context Switch -> Kernel for Arbitration -> if lock is taken, the thread is suspended and housekeeping tasks are resumed; during this time we could lose the previously cached data and instructions.
5. __Cost of Compare and Swap__
   1. Target update is a single word.
   2. Atomic operations are used that are provided by the instruction set architecture.
   3. Much faster than locks usually because there is no arbitration by the kernel.
   4. There is still a cost of CAS
      1. The instruction pipeline is locked to ensure atomicity.
      2. A memory barrier is employed so that the changes are visible by other threads.
   5. Ideal: Single thread owning the writes but multiple threads for reading.
6. __Memory Barriers__ 