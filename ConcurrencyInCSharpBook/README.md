# Concurrency In C# By Stephen Cleary

## Main Takeaways
1. Definitions
   1. Concurrency is doing many things at the same time.
   2. Multithreading is a type of concurrency making use of many threads.
   3. Parallel programming is concurrent multithreading.
   4. Async programming is non-blocking and makes use of Futures and callbacks.
2. Based on requirements, assess the type of concurrent data structure needed.
3. Synchronization is important for program correctness.
   1. Use:
      1. locks 
      2. signals
      3. throttling 

## Resources Referenced
1. Chapter 1: Concurrency: An Overview - [Notes](1.md)
2. Chapter 9: Collections - [Notes](2.md)
3. Chapter 12: Synchronization - [Notes](3.md)
4. Chapter 13: Scheduling - [Notes](4.md)
   1. [``TaskFactory.StartNew`` vs. ``Task.Run``](https://devblogs.microsoft.com/pfxteam/task-run-vs-task-factory-startnew/)