# Async Await Programming

## 3 Main Lessons 
1. How ``async-await`` works
   1. First ``await``, control is yielded back to the caller whether that's a method or the message loop.
   2. The awaited Task is allocated on the heap.
   3. Once the awaited Task completes, it forces the message pump to resume control. 
   4. The underlying mechanism of ``async-await`` is a State Machine and an AsyncTaskMethodBuilder that the compiler automatically generates in the case it observes ``async`` in the method signature.
2. ``Async`` code is Non-Blocking.
3. For I/O bound code, use ``async-await``; for CPU bound code, ``await Task.Run(...)``.

## Resources Referenced 

### Books
1. Ben Watson's Writing High Performance .NET Code: Chapter 5

### Links
1. [MS Docs: Async](https://docs.microsoft.com/en-us/dotnet/csharp/async)
2. [MS Docs: Asynchronous programming with async and await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
3. [MS Docs: Async in Depth](https://docs.microsoft.com/en-us/dotnet/standard/async-in-depth)
4. [Understanding async/await State Machine in .NET](https://mykkon.work/async-state-machine)
5. [David Fowler's Notes](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/0ba7625050f975f8a7df1df57c80ad08da250541/AsyncGuidance.md) 
6. [There is No Thread](https://blog.stephencleary.com/2013/11/there-is-no-thread.html)
7. [6 Essential Tips for Async Programming - Channel 9](https://channel9.msdn.com/Series/Three-Essential-Tips-for-Async)