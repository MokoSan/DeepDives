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