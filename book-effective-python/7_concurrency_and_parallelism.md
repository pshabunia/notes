# Chapter 6. Concurrency and Parallelism

* Concurrency: tasks > executors. Tasks interleaved and interrupted.
* Parallelism: 1 < tasks <= executors. Tasks isolated and simultaneous.

## Item 52: Use `subprocess` to Manage Child Processes

```python
import subprocess
result = subprocess.run(
    ['echo', 'Hello from the child!'],
    capture_output=True,
    encoding='utf-8')
result.check_returncode() # No exception means clean exit
print(result.stdout) # >Hello from the child!
```

More advanced use cases involve I/O pipes, timeouts and inter-process communication.

* Use the subprocess module to run child processes and manage their input and output streams.
* Child processes run in parallel with the Python interpreter, enabling you to maximize your usage of CPU cores.
* Use the `run` convenience function for simple usage, and the `Popen` class for advanced usage like UNIX-style pipelines.
* Use the `timeout` parameter of the communicate method to avoid dead-locks and hanging child processes.

# Item 53: Use Threads for Blocking I/O, Avoid for Parallelism

GIL = Global Interpreter Lock

Essentially, the GIL is a mutual-exclusion lock (mutex) that prevents CPython from being affected 
by preemptive multithreading, where one thread takes control of a program by interrupting another thread.

Although Python supports multiple threads of execution, the GIL causes **only one of them** to ever make 
forward progress at a time. There are ways to get CPython to utilize multiple cores, but they don’t work with 
the standard `Thread` class. Must use `concurrent.futures` for true parallelism.

* Python threads can't run in parallel on multiple CPU cores because of the Global Interpreter Lock (GIL).
* Python threads are still useful despite the GIL because they provide an easy way to do multiple things seemingly at 
the same time.
* Use Python threads to make multiple system calls in parallel. This allows you to do blocking I/O at the same time as 
computation.

# Item 54: Use `Lock` to Prevent Data Races in Threads

https://docs.python.org/3/library/threading.html

The Python interpreter enforces fairness between all the threads
that are executing to ensure they get roughly equal processing time.
To do this, Python suspends a thread as it’s running and resumes
another thread in turn. The problem is that you don’t know exactly
when Python will suspend your threads. A thread can even be paused
seemingly halfway through what looks like an atomic operation.


Example of synchronized counter:
```python
from threading import Lock

class LockingCounter:
    def __init__(self):
        self.lock = Lock()
        self.count = 0
    
    def increment(self, offset):
        with self.lock:
            self.count += offset
```

* Even though Python has a GIL, developers are still responsible for protecting data in thread race conditions.
* The programs will corrupt their data structures if you allow multiple threads to modify the same objects 
without mutual-exclusion (mutex) locks.
* Use the `threading.Lock` class to enforce your program's invariants between multiple threads.

# Item 55: Use `Queue` to Coordinate Work Between Threads

https://docs.python.org/3/library/queue.html

Queue eliminates the busy waiting in the producer-consumer implementation by making the `get`
method to block until new data is available. To solve the pipeline backup issue, the `Queue` class allows
to specify the maximum amount of tasks queued. This buffer size causes calls to `put` to block when 
the queue is already full.

* Pipelines are a great way to organize sequences of work—especially
I/O-bound programs—that run concurrently using multiple Python threads.
* Be aware of the many problems in building concurrent pipelines:
busy waiting, how to tell workers to stop, and potential memory explosion.
* The `Queue` class has all the facilities you need to build robust
pipelines: blocking operations, buffer sizes, and joining.

# Item 56: Know How to Recognize When Concurrency Is Necessary

* A program often grows to require multiple concurrent lines of execution as its scope and complexity increases.
* The most common types of concurrency coordination are _fan-out_ (generating new units of concurrency) 
and _fan-in_ (waiting for existing units of concurrency to complete).
* Python has many different ways of achieving fan-out and fan-in.

# Item 57: Avoid Creating New Thread Instances for On-demand Fan-out

Threads are expensive (~8Mb). Doesn't scale well when you need 1000s of (lightweight) threads.
Threads entail a performance overhead when they run due to context switching between them.
Threads require extra effort for lock management and exception handling. Thread class will independently catch
any exceptions that are raised by the target function and then write their traceback to `sys.stderr`. 
Such exceptions are never re-raised to the caller that started the thread in the first place.

Therefore, threads are not the solution if you need to constantly create and finish new concurrent functions.

* Threads have many downsides: They're costly to start and run
if you need a lot of them, they each require a significant amount
of memory, and they require special tools like `Lock` instances for
coordination.
* Threads do not provide a built-in way to raise exceptions back in
the code that started a thread or that is waiting for one to finish,
which makes them difficult to debug.

# Item 58: Understand How Using `Queue` for Concurrency Requires Refactoring

* Using `Queue` instances with a fixed number of worker threads
improves the scalability of fan-out and fan-in using threads.
* It takes a significant amount of work to refactor existing code to use
`Queue`, especially when multiple stages of a pipeline are required.
* Using `Queue` fundamentally limits the total amount of I/O parallelism 
a program can leverage compared to alternative approaches provided by other 
built-in Python features and modules.

# Item 59: Consider `ThreadPoolExecutor` When Threads Are Necessary for Concurrency

https://docs.python.org/3/library/concurrent.futures.html

The best part about the `ThreadPoolExecutor` class is that it automatically propagates exceptions back to the caller
when the `result` method is called on the `Future` instance returned by the executor's `submit` method.

* ThreadPoolExecutor enables simple I/O parallelism with limited
refactoring, easily avoiding the cost of thread startup each time fan-out concurrency is required.
* Although ThreadPoolExecutor eliminates the potential memory blow-up issues of using threads directly, 
it also limits I/O parallelism by requiring max_workers to be specified upfront.

# Item 60: Achieve Highly Concurrent I/O with Coroutines

https://docs.python.org/3/library/asyncio.html

All previously discussed approaches fall short in their ability to handle thousands of simultaneously concurrent
functions. Python addresses the need for highly concurrent I/O with coroutines. Coroutines let you have a very large 
number of seemingly simultaneous functions in your Python programs. They're implemented using the `async` and `await`
keywords along with the same infrastructure that powers generators.

The cost of starting a coroutine is a function call. 
Once a coroutine is active, it uses less than 1 KB of memory until it’s exhausted.
Coroutines are running on the _event loop_, which can do highly concurrent I/O efficiently, 
while rapidly interleaving execution between appropriately written functions.

```python
import asyncio

async def do(input):
    ...

async def do_work(inputs):
    tasks = [do(input) for input in inputs]  # Fan out
    await asyncio.gather(*tasks)  # Fan in

asyncio.run(do_work(inputs)) # Run the event loop
```

* Calling `do` doesn't immediately run that function. Instead,
it returns a coroutine instance that can be used with an `await`
expression at a later time. This is similar to how generator functions 
that use `yield` return a generator instance when they're
called instead of executing immediately. Deferring execution like
this is the mechanism that causes fan-out.
* The `gather` function from the `asyncio` built-in library causes
fan-in. The `await` expression on `gather` instructs the event loop to
run the `do` coroutines concurrently and resume execution of the simulate 
coroutine when all of them have been completed.
* No locks are required for operations within low-level `async` functions
because execution occurs concurrently within a single thread 
using the event loop that’s provided by `asyncio.run`.

> The beauty of coroutines is that they decouple your code's instructions 
for the external environment (i.e., I/O) from the implementation
that carries out your wishes (i.e., the event loop). They let you focus
on the logic of what you're trying to do instead of wasting time trying
to figure out how you're going to accomplish your goals concurrently.

Summary:
* Functions that are defined using the `async` keyword are called **coroutines**. 
A caller can receive the result of a dependent coroutine by using the `await` keyword.
* Coroutines provide an efficient way to run tens of thousands of functions seemingly at the same time.
* Coroutines can use fan-out and fan-in in order to parallelize I/O,
while also overcoming all the problems associated with doing I/O in threads.

# Item 61: Know How to Port Threaded I/O to `asyncio`

* Python provides asynchronous versions of for loops, with statements, generators, comprehensions, 
and library helper functions that can be used as drop-in replacements in coroutines.
* The asyncio built-in module makes it straightforward to port existing code 
that uses threads and blocking I/O over to coroutines and asynchronous I/O.

# Item 62: Mix Threads and Coroutines to Ease the Transition to `asyncio`

> In order to do that, your codebase needs to be able to use threads
for blocking I/O (Item 53) and coroutines for asynchronous I/O (Item 60)
at the same time in a way that's mutually compatible. 
dIn other words, you need threads to be able to run coroutines, 
and you need coroutines to be able to start and wait on threads.
 
Top-down approach:
1. Change a top function to use `async def` instead of `def`.
2. Wrap all of its calls that do potentially blocking I/O to use `asyncio.run_in_executor` instead.
3. Ensure that the resources or callbacks used by `run_in_executor` invocations are properly synchronized 
(i.e., using `Lock` or the `asyncio.run_coroutine_threadsafe` function).
4. Try to eliminate `get_event_loop` and `run_in_executor` calls by moving downward through the call hierarchy 
and converting intermediate functions and methods to coroutines (following the first three steps).

(!!) beware thread allocation and even loop controls with `run_coroutine_threadsafe` and `run_in_executor`

Bottom-up approach:
1. Create a new asynchronous coroutine version of each leaf function that you're trying to port.
2. Change the existing synchronous functions so they call the coroutine versions and run the event loop 
instead of implementing any real behavior.
3. Move up a level of the call hierarchy, make another layer of coroutines, and replace existing calls 
to synchronous functions with calls to the coroutines defined in step 1.
4. Delete synchronous wrappers around coroutines created in step 2 as you stop requiring them 
to glue the pieces together.

Summary:
* The awaitable `run_in_executor` method of the asyncio event
loop enables coroutines to run synchronous functions in
`ThreadPoolExecutor` pools. This facilitates top-down migrations to asyncio.
* The `run_until_complete` method of the asyncio event loop enables
synchronous code to run a coroutine until it finishes. The
`asyncio.run_coroutine_threadsafe` function provides the same
functionality across thread boundaries. Together these help with
bottom-up migrations to asyncio.

# Item 63: Avoid Blocking the asyncio Event Loop to Maximize Responsiveness

* Making system calls in coroutines—including blocking I/O and
starting threads—can reduce program responsiveness and increase
the perception of latency.
* Pass the `debug=True` parameter to `asyncio.run` in order to detect
when certain coroutines are preventing the event loop from reacting
quickly.

# Item 64: Consider `concurrent.futures` for True Parallelism

References:
* https://docs.python.org/3/library/concurrent.futures.html#processpoolexecutor
* https://docs.python.org/3/library/multiprocessing.html
* https://swig.org/
* https://github.com/google/clif

Notes:
* Moving CPU bottlenecks to C-extension modules can be an effective
way to improve performance while maximizing your investment in
Python code. However, doing so has a high cost and may introduce
bugs.
* The multiprocessing module provides powerful tools that can parallelize 
certain types of Python computation with minimal effort.
* The power of multiprocessing is best accessed through the
`concurrent.futures` built-in module and its simple `ProcessPoolExecutor` class.
* Avoid the advanced (and complicated) parts of the `multiprocessing` module until you’ve exhausted all other options.