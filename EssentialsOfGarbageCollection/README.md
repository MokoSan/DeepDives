# Notes from [Essentials of Garbage Collection](https://dmitrysoshnikov.teachable.com/courses/607252/)

## Memory Management

### 1. Allocation Types
1. 3 types of allocation types:
   1. __Static__
      1. All sizes are known at compile time.
      2. Most robust.
      3. Can't deal with:
         1. Recursion
         2. Variable sized data structures.
   2. __Stack__
      1. Allows recursion.
      2. Stack frames or activation records allow for users to pass in parameters to functions.
      3. Can't deal with:
         1. Runtime dynamic allocations.
         2. Stack overflow => robustness decreases.
   3. __Heap__
      1. Dynamic memory. 
      2. Any chunk of memory and pointer can be returned. 
      3. Variable sized data structures are allowed.
      4. As a developer, we need to deal with memory management:
         1. Memory Leaks
         2. Dangling pointers.

### 2. Manual Memory Management
1. __Memory Leak__
   1. Forgetting to free memory and as a result, the allocation is still on the heap but we have no way to free that memory up.
2. __Dangling Pointer__
   1. Freeing memory too early and as a result, a pointer pointing to invalid memory.
      1. ```free(ptr); *ptr = x;```
3. Reason for these issues with manual memory management is human error and as a result Automated Memory Management is done to reduce human factor of error.

### 3. Object Header
1. Stores metadata used by the allocator and/or collector.
2. Any automation comes with a price and the object header contains vital information that speeds up the allocation / collection aspect of GC. 
3. Typical allocation
   1. Object Header (N bytes)
   2. User Data (Requested bytes)
   3. Padding / Word Alignment
4. Object header can be co-located with the user data or be available elsewhere.

### 4. Virtual Memory and Memory Layout
1. OS allocates virtual memory space for a particular process. The amount allocated is limited by the size of the word. 
   1. e.g. 32 bit systems => 4 GB per process.
   2. For Linux:
      1. 1 GB reserved for the OS.
      2. 3 GB for the user space.
   3. For Windows:
      1. 2 GB for the OS (can be changed).
      2. 2 GB for the user space.
2. Layout
   1. ![Layout](Images/MemoryLayout.png)
   2. Regions
      1. __Code Segment__: Executable code containing instructions stored in the binary.
      2. __Data Segments__: Contains data e.g. static data.
         1. __BSS Segment__: Uninitialized data.
         2. __Data Segment__: Initialized data.
      3. __Stack__:
         1. Grows down from upper to lower addresses.
         2. Base Pointer (``%rbp``) to Stack Pointer (``%esp`` / ``%rsp``).
         3. Limit pointer denotes maximum size the stack can grow to before stack overflows.
         4. OS introduces random offset for security.
      4. __Heap__: 
         1. Grows up from lower to upper addresses.
         2. Top of the heap is denoted by the ``brk`` or Program Break.
         3. More memory can be requested by bumping up the ``brk``.
            1. In Linux, this can be done via the following system calls:
               1. ``brk(void *addr)``.
               2. ``sbrk(intptr_t increment)``.
               3. ``mmap(...)``.
            2. These system calls are manipulated by the ``malloc(..)`` and ``free(..)`` functions.
      5.  __Memory Mapped Segments: Space Between Stack and Heap__
          1.  Files
          2.  I/O Devices
          3.  Heap Memory is reclaimed from this space.
      6.  __Command Line Args / Environment Variables__
3. __Memory Mapping Process__
   1. Virtual Memory pages are mapped to physical memory via __MMU__: Memory Management Unit. 
         1. __TLB__: Translation Look Aside Buffer, the cache of the virtual -> physical memory.
   2. If there is no physical memory available, the OS can use disk space: Swap Files in Unix and is transparent to the user.

### 5. Mutator, Allocator, Collector.
1. 3 Main Modules involved in Automatic Memory Management.
2. __Mutator__
   1. User program that creates object through the allocator.
   2. The Mutator doesn't directly manipulate the Heap.
3. __Allocator__
   1. Directly manipulates the heap by allocating the memory of the needed size requested by the mutator and tracks the meta-information or Object Header of the allocated objects.
   2. Result of the allocator is a pointer (or reference) to allocated memory block.
4. __Collector__
   1. Collector reclaims memory.
   2. Preserves Mutator view by preserving heap invariants such as not reclaiming live objects.
   3. Collector closely communicates with the allocator
      1. Synchronizing on the Object Header structure to update the metadata associated the allocations.
   4. Mutator may not know that the collector exists as it interacts primarily with the allocator.
   5. Some languages expose APIs to directly interact with the collector.
      1. ``System.gc()`` in Java.
      2. ``gc.collect()`` in Python.
5. __Stop The World__: The state the mutator is put in when the collector starts its work. All the threads are put to sleep for the process of collection to take place.
   1. There are improvements to this via ``Background GC`` that don't stop the world.

### 6. Allocators: Free-List vs. Sequential
1. __Sequential Allocator__
   1. The existing free blocks aren't used immediately but the allocation pointer i.e. where the next allocation will take place is simply increased and the garbage is left behind.
   2. Advantage of this allocator is that allocations are fast.
   3. Example: Pool Allocator.
   4. Used for:
      1. Mark-Compact
      2. Copying GC
      3. Generational GC
   5. Implementation is trivial, we just track the allocation and the end of the heap pointer.
2. __Free-List Allocator__
   1. A list like structure of free blocks i.e. blocks of memory on the heap where subsequent allocations can be made. The free list allocator reuses the freed block of memory, tracking them in a list-like data structure by traversing to find the memory of the appropriate size to use.
   2. This approach is a bit slower because of free-block searching overhead. 
   3. Used for:
      1. Mark-Sweep 
      2. Reference Count GC
   4. __Free List Search Strategies__
      1. __First-Fit__
         1. Finds the first block that fits the size starting from the beginning of the allocated blocks.
         2. If the find block found is larger, the block can be split tracking only the size that's needed.
      2. __Next-Fit__
         1. Variation of the first-fit but starts search from the previous successful position. This allows for skipping small blocks at the beginning of the heap to get the free block faster.
         2. If we reach the end of the heap, we start over from the start => "Circular first fit allocation"
      3. __Best-Fit__
         1. Finding blocks for which the size fits the best for the requested allocation.
         2. Doesn't split the block wasting resources and time.
      4. __Segregated Fit__
         1. Most optimized search algorithm used in production allocators.
         2. Heap is partitioned and grouped based on size e.g. Segregated List of Size 8, 16, 32, 64 etc.
         3. Multiple free lists containing blocks of a certain size.

### 7. Semantic vs. Syntactic Garbage
1. __Semantic Garbage__
   1. Data that will not be reached.
   2. Strong references to unused but live data aka "Live Garbage"
   3. Bugs in the program logic.
   4. Example: Non-Invalidated Caches.
2. __Syntactic Garbage__
   1. Data that cannot be reached. 
   2. Unreachable objects with no references to them.
   3. GC works only with syntactic garbage.

## Garbage Collectors

### 1. Tracing vs. Direct Collectors
1. __Tracing Collectors__
   1. Tracing Collectors scan the heap for live objects and everything else by extension is treated as garbage.
   2. Analysis starts at the GC Root that could be held by a register, global or stack variable.
   3. Traversal path of the live objects on the heap from the roots is called the Trace.
   4. For __Stop the world__ where all mutator threads are blocked, the trace happens during the pause.
   5. For concurrent GC algorithms, the trace happens in parallel with the mutator threads. 
2. __Direct Collectors__
   1. Garbage is reclaimed exactly at the moment it is identified.
   2. There is no explicit GC Pause.
   3. Direct Collectors are deeply integrated into the mutator.
   4. No tracing is done via direct collectors.
   5. Reference counting is used.
3. __4 Most Used Collectors__
   1. __Mark-Sweep__
   2. __Mark-Compact__
   3. __Copying GC__
   4. __Reference Counting__
4. Mark-Sweep, Mark-Compact, Copying GC are all __Tracing Collectors__ while Reference Counting is a __Direct Collector__.

### 2. Mark-Sweep Collector
1. __Type__: Tracing Collector that searches for live objects and everything else is garbage.
2. __Phases__
   1. __Mark__: Trace for live objects.
      1. From the roots, goes to all children and sets the mark bit that's available in the object header to 1.
   2. __Sweep__: Reclaims the garbage.
      1. Traverses the entire heap to check if the mark bit is set.
      2. If the mark bit is set, it is unset for the subsequent GC.
      3. If the mark bit is unset, the object is a candidate for GC and it's added to the free-list.
3. __Moving__: Objects aren't moved.
   1. This is good for languages exposing pointer semantics such as C/C++
4. __Allocator__:
   1. The allocations are added to a free-list.
   2. These result in slower allocations as the next allocation block must be found.
5. __Issues__:
   1. __Fragmentation__: Next free block might be further away over the heap.
      1. Results in slower allocations.
      2. Bad cache locality.
6. __Pseudo-Code__

__Mark__:

```
to_scan = roots

while !to_scan.is_empty():
   pointer = to_scan.peak()

   # Check if the mark bit isn't set
   # Then subsequently set it for this live object.
   if pointer.get_markbit() == 0:
      pointer.set_markbit()

      # Add all the children that this live object is pointing to.
      for child in pointer.children():
         to_scan.add(child)
```

__Sweep__:

```
# Traverse the entire heap.
heap_runner = heap.start()

while heap_runner < heap.end():

   # If the current object has the mark bit set, reset it for the next GC.
   if heap_runner.is_set_markbit():
      heap_runner.reset_markbit()

   # If the current object doesn't have the mark bit set, add it to free list to be reused.
   else:
      add_to_freelist(heap_runner)

   # Increment the heap runner to the next object.
   heap_runner = heap_runner.next()
```

### 3. Mark-Compact Collector
1. Advantages because of compaction that results in less fragmentation:
   1. Better cache locality. 
   2. Faster allocation.
      1. We get to use the Bump Allocator after compaction.
2. Mark-Compact is an in-place collector.
3. __Type__: Tracing Collector.
   1. With GC Pauses.
4. __Phases__
   1. __Mark__: Marking phase is exactly the same as the Mark-Sweep collector. Trace of the live objects is done.
      1. Object header also stores the forwarding address or the new address where the object is moved.
   2. __Compact__: Relocation or moving live objects to the new forwarding address.
      1. To avoid fragmentation.
5. __Moving__
   1. Can't be used by languages exposing pointer semantics such as C/C++.
6. __Allocator__
   1. Sequential aka "Bump Allocator"
7. __Disadvantages__
   1. Up to 3x Heap Traversal.
8. __Several Mark-Compact Algorithms__
   1. Two Finger
   2. The LISP 2 <- Most Used In Practice
   3. Threaded
   4. One pass
9.  __LISP 2 Mark-Compact Algorithm__
   5. __Steps__: All require a full heap reversal. This is the price to pay for the fast compacted heap and allocation.
      1. __Compute Locations__
         1. 3 pointers used to get the locations:
            1. __Free Pointer__: Pointer of the position where the next scanned alive pointer will be relocated.
            2. __Scan Pointer__: Pointer used for analyzing the dead and alive objects.
            3. __End Pointer__: Pointer pointing to the end of the heap.
         2. The idea here is to populate the forwarding address of each of the blocks based on the traversal.
      2. __Update References__: Fixing the child references of alive objects to point to the new locations. 
         1. __Steps__
            1. Go through the roots.
            2. Go through all the children and set those to the new addresses.
      3. __Relocate__: Move alive objects to the new locations recorded in the forwarding addresses.
10. __Pseudo-Code of Compact__

__Compact__
```
compact():
   compute_locations()
   update_references()
   relocate()
```

__Compute Location__

```
compute_location():
   scan = start_of_heap()
   free = start_of_heap()

   # Traverse the entire heap.
   while scan < end:

      # If the object is marked i.e. alive. update the reference to the free pointer.
      if scan.get_markbit() == 1:
         *scan.forward = free
         # Move the free pointer to the next spot.
         free = free.next

      scan = scan.next
```

__Update References__

```
update_references():
   for root in roots:
      if root != null:
         root = root.forward
   
   scan = start
   while scan < end:
      if scan.get_markbit() == 1:
         for child in scan.get_children():
            if child != null:
               child = child.forward
      scan = scan.next
```

__Relocate__

```
relocate():
   scan = start_of_heap()
   while scan < end:
      if scan.get_markbit() == 1:
         scan.reset_markbit()
         move(scan, scan.forward)
      scan = scan.next
```

### 4. Copying Collector aka Semi Space Collector
1. Storage for speed is the trade off here.
2. Half the heap is reserved for collection purposes aka Semi-space collector.
3. Area used for allocation is called "From Space" or "Old Space".
   1. Only spot where allocations are allowed.
4. Area used for garbage collector is called "To Space" or "New Space".
5. We can run into OOM (Out of Memory) if we use up entire From Space even though we have the entire New Space available.
6. __Stages__
   1. __Copy__: Aka Evacuation - Evacuate live objects. 
      1. Copying the live data starting from the roots, from the From area to the To area. 
   2. __Forward__
      1. Keeping forward address



## Advanced Topics


### 4. Tri-color Abstraction


### 5. GC Barriers
1. __Barrier__: An extra fragment of code executed on certain mutator events. 
2. Two Types of Barriers
   1. Read
   2. Write