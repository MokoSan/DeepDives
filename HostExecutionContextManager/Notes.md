# HostExecutionContextManager

1. CLR calls the HostExecutionContextManager every time a call to ``ExecutionContext.Run`` takes place to allow hooks into the thread execution.
   1. Conditional on if the AppDomainManager has a reference to the ``HostExecutionContextManager``. 
2. ``ExecutionContext`` provides a container for all information relevant to a thread.
   1. If you have to know it's there, either you’re doing something super advanced, or something’s gone wrong.
   2. ExecutionContext is about ambient information i.e. it stores data about the relevant to the current context.
      1. In the single threaded world, Thread Local Storage will contain all the context needed for the secure and correct execution of the thread.
      2. In the now more asynchronous world, Thread Local Storage will not flow amongst the different threads that are borrowed to finish some asynchronous task.
         1. Thread Local Storage is specific to a certain thread while asynchronous operations aren't tied to a specific thread.
      3. ExecutionContext enables ambient data to flow from one thread to another in the case of asynchronous workflows.
   3. ExecutionContext is really just a state bag that can be used to capture all of this state from one thread and then restore it onto another thread while the logical flow of control continues.
   4. ``ExecutionContext.Capture()``: Ambient State to retain for the next thread is captured from this method.
      1. This ambient state is restored during the invocation of a delegate in ``ExecutionContext.Run``.
         1. All of the methods in the .NET Framework that fork asynchronous work capture and restore ExecutionContext in a manner like this.
      2. All of the methods in the .NET Framework that fork asynchronous work capture and restore ExecutionContext. E.g. with ``Task.Run()``.
   5. “flowing ExecutionContext" => process of capturing state that was ambient on thread and restoring the state onto a thread at some later point while that thread executes a supplied delegate.
      1. When the async method is about to suspend while awaiting, the infrastructure captures an ExecutionContext.  
      2. The delegate that gets passed to the awaiter has a reference to this ExecutionContext instance and will use it when resuming the method.  This is what enables the important “ambient” information represented by ExecutionContext to flow across awaits.
3. ``HostExecutionContextManager.Capture()``: Captures the host execution context from the current thread.
4. ``HostExecutionContextManager.SetHostExecutionContext(HostExecutionContext)``: Sets the current host execution context to the specified host execution context.
5. ``HostExecutionContextManager.Revert(Object)``: Restores the host execution context to its prior state.