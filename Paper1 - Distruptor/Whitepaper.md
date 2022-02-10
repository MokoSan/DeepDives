# Paper 1 - Disruptor: High Performance Alternative To Bounded Queues For Exchanging Data Between Concurrent Threads

## Notes

### 1. Abstract

1. __Problem__: Latency costs of traditional queuing logic is extremely high - at the level of IO operations i.e. dramatically slow.
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
   1. Modern processors perform out-of-order execution of instructions, loads and stores for performance reasons.
      1. Correctness is guaranteed but order isn't.
   2. Important implications when multiple threads share state => ordering is more important!
   3. Memory barriers are used by processors to indicate sections of code where ordering is important and visibility of change is achieved amongst threads.
   4. __Details about Modern CPUs__
      1. Faster than ever in comparison to memory systems.
      2. To bridge this gap between speed of computation and memory, complex cache systems are used.
         1. Essentially hardware based hash-tables without chaining.
      3. Caches are kept __coherent__ via message passing protocols.
         1. __Coherence__: Consistency / uniformity of shared resource data that ends up stored in multiple local caches.
      4. Processors have:
         1. __Store Buffers__: Offload writes to caches.
            1. A store buffer is a speculative structure that exists in the CPU, just like the load queue and is for allowing the CPU to speculate on stores.
         2. __Invalidate Queues__: Cache coherency protocols can acknowledge invalidation messages quickly for efficiency.
            1. A queue that keeps track of invalidations and ensures that they complete properly so that a cache can take ownership of a cache line so it can then write that line. 
      5. Latest version of any value could, at any stage after being written, be in a register, a store buffer, one of the many layers of cache, or in main memory.
         1. If threads share this value, it needs to be visible in an ordered fashion and is achieved through the coordinated exchange of cache coherency messages.
         2. The timely generation of these is controlled by memory barriers.
   5. __Types of Memory Barriers__
      1. __Read (Load) Memory Barrier__: Orders load instructions on the CPU that executes it marking a point in the invalidate queue for changes coming into the cache.
         1. Consistent view of the world for write operations __before__ the read barrier.
      2. __Write (Store) Memory Barrier__: Orders store instructions on the CPU that executes it by marking a point in the store buffer, thus flushing writes out via its cache. This barrier gives an ordered view of the world of what store operations before the write barrier.
      3. __Full Memory Barrier__: Orders both loads and stores only on the CPU that executes it.
   6. __Memory Barriers / Fences from #2__
      1. Memory Barriers provide 2 properties:
         1. Preservation of external visible program order by ensuring all instructions either side of the barrier appear in the correct program order if observed from another CPU.
         2. They make memory visible by ensuring the data is propagated to the cache sub-system.
      2. __Store / Write Barrier__
         1. ``sfence`` in x86.
         2. Waits for all store instructions prior to the barrier to be written from the store buffer to the L1 cache for the CPU on which it is issued.
         3. This will make program state visible to other CPUs so that they can act on it if necessary.
      3. __Load / Read Barrier__
         1. ``lfence`` in x86.
         2. Ensures all load instructions after the barrier happen after the barrier and then wait on the load buffer to drain for the issuing CPU.
         3. This makes program state exposed from other CPUs visible to this CPU before making further progress.
      4. __Full Barrier__
         1. ``mfence`` in x86.
         2. Composite of both load and store barriers happening on a CPU.
      5. __Impact__
         1. Prevention of reordering.
         2. There is an advantage to grouping necessary memory barriers in that buffers flushed after the first one will be less costly because no work will be under way to refill them.
   7. .NET / Java -> read and write of a ``volatile`` field implements read and write barriers.