# Async Programming
1. How ``async-await`` works
   1. First ``await``, control is yielded back to the caller whether that's a method or the message loop.
   2. The awaited Task is allocated on the heap.
   3. Once the awaited Task completes, it forces the message pump to resume control. 
   4. The underlying mechanism of ``async-await`` is a State Machine and an IAsyncMethodBuilder that the compiler automatically generates in the case it observes ``async`` in the method signature.