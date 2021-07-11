[Csharp](/languages/csharp)
# Concurrency in CSharp

Summarizing concurrency in C# from recent lectures on *C# in a Nutshell*, *Concurrency in C#* and self experience on using it more and more in server-side code for CPU and I/O bound work for both web services and worker hosts.


#### Concurrency, paralellism, multithreading...

TODO: Concurrency vs parallelism vs multithreading vs asynchrony vs reactive

#### Task, ThreadPool, Thread

- **`Thread`** - virtual (managed) thread on CLR, closely (but not neccessary 1-to-1) related to OS-level threads. Share (heap) memory of parent process but have separate stack. High cost of creation and tear down. Consumes 1MB of memory upfront. Usage: `new Thread(someDelegate).Start();`
- **`ThreadPool`** - since creating threads is expensive, pooling threads for reuse (recycling?) has been introduced. ThreadPool uses throttling to not overburden CPU (protects from thread/resource starvation). There are two "types" of threads reserved by the pool: worker threads (for CPU-bound user code) and I/O threads (for monitoring I/O Completion Ports). ASP.NET or WCF use `ThreadPool` for handling requests. Usage: `ThreadPool.QueueUserWorkItem(_ => DoStuff());` or simply via Tasks: `Task.Run(() => DoStuff());`
- **`Task`** - higher-level (higher than Thread) abstraction of concurrent operation. Delegates creation or recycling of threads to `ThreadPool` (so might or might not run on new thread) or can even be threadless via `TaskCompletionSource`. Tasks can emit result on completion via `Task<TResult>`. Tasks support composition, continuation, cancellation and are core of asynchronous programming (with async-await) in C#. In Task-based Asynchronous Pattern, Tasks are returned 'hot', which means they are already running.

#### lock, SemaphoreSlim, ReadWriterLockSlim

TODO

#### Atomic operations - Interlocked, Exchange, CompareExchange

TODO

#### Parallel & PLINQ

TODO

#### System.Collections.Concurrent & System.Collections.Immutable

TODO

#### TaskCompletionSource

TODO

#### ConfigureAwait

TODO

#### CancellationToken

TODO

#### Lazy<T>

TODO

#### Thread-local storage

TODO

#### References
- Joseph Albahari, Ben Albahari, *C# in a Nutshell, 7th edition*
- Stephen Cleary, *Concurrency in C#, Cookbook*
- Joseph Albahari, *Threading in C#* - http://www.albahari.com/threading/
- https://blog.slaks.net/2013-10-11/threads-vs-tasks/
- https://github.com/dotnet/coreclr/blob/master/Documentation/botr/threading.md