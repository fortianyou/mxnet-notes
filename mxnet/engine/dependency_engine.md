DependencyEngine
====


Goals
-----

Here’s a summary of goals for the engine:

The engine should not be aware of what operations it performs, so that users can perform any operations they define.
It should not be restricted in what type of objects it can schedule.
We should be able to schedule dependencies on GPU and CPU memory.
We should be able to track dependencies on the random number generator, etc.
The engine should not allocate resources. It should only track dependencies. Users can allocate their own memory, PRNG, etc.

Separate Dependency Tracking with Running Policy
-----

If you’re reading carefully, you might have noticed that the preceding section shows only the algorithm for deciding when an operation can be executed. We didn’t show how to actually run an operation. In practice, there can be many different policies. For example, we can either use a global thread-pool to run all operations, or use a specific thread to run operations on each device.

This running policy is usually independent of dependency tracking, and can be separated out as either an independent module or a virtual interface of base-dependency tracking modules. Developing an elegant runtime policy that is fair to all operations and schedules is an interesting systems problem itself.

