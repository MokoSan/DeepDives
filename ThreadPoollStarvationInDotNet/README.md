# Threadpool Starvation

## Main Takeaways
1. Threadpool starvation is when tasks scheduled aren't getting executed immediately as other tasks are busy executing.
2. The effects of threadpool starvation are unresponsive UI and low throughput.
3. Solutions to solve threadpool starvation:
   1. Prevent blocking by using ``async-await``
   2. Throttle spots where there are bursts of work
   3. Decrease lock contention 
4. Detection
   1. CPU is not saturated
   2. Sluggish UI

## Resources Referenced
1. [VS Threading and Threadpool Starvation](https://github.com/microsoft/vs-threading/blob/main/doc/threadpool_starvation.md) - [Notes](1.md)
2. [Diagnosing .NET Core ThreadPool Starvation with PerfView (Why my service is not saturating all cores or seems to stall).](https://docs.microsoft.com/en-us/archive/blogs/vancem/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall) - [Notes](2.md)