Engine
=====

+ NaiveEngine(Single Thread)
   A very simple engine that uses the master thread to do the computation synchronously. Setting this engine disables multi-threading. You can use this type for debugging in case of any error. Backtrace will give you the series of calls that lead to the error. Remember to set MXNET_ENGINE_TYPE back to empty after debugging.

+ ThreadedEngine
   A threaded engine that uses a global thread pool to schedule jobs.

+ ThreadedEnginePerDevice
   A threaded engine that allocates thread per GPU and executes jobs asynchronously.


ThreadEngine
====

+ OprBlock
   Each OprBlock corresponds to an operation pushed to the engine.

+ VersionedVarBlock
   VersionedVarBlock that corresponding to a variable version.
   This is a basic unit of LinkedList in the ThreadedVar.

+ ThreadedVar
   Each ThreadedVar is a linked list(queue) of operations to be performed.