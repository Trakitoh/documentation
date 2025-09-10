# Python Notes

_Any notes that may be useful for working with the project backend_

## Async Stuff

**Resources**
- [Concurrency Vs Parallelism! (4 min) ](https://www.youtube.com/watch?v=RlM9AfWf1WU)
- [Comparison of asyncio, threading, and multiprocessing (blogpost)](http://masnun.rocks/2016/10/06/async-python-the-different-forms-of-concurrency/)
- [Python Asyncio (24 min)](https://www.youtube.com/watch?v=Qb9s3UiMSTA)
- [Comparison of `async def` and `def` in FastAPI (5 min)](https://www.youtube.com/watch?v=tGD3653BrZ8)
- [Overview of `asyncio` concepts (docs)](https://docs.python.org/3/howto/a-conceptual-overview-of-asyncio.html#a-conceptual-overview-of-asyncio)
- [Coroutine and Tasks (docs)](https://docs.python.org/3/library/asyncio-task.html#)

**General Concepts**
Tasks
- unit of work to be performed by computer 
- CPU-Bound tasks
  - tasks that spend most of their time performing calculations
  - limited by CPU speed
  - Ex: image processing, ML model training, video rendering, etc
- I/O-Bound tasks
  - tasks that spend most of their time _waiting_ for input/output operations
  - CPU is idle during the waiting period
  - Ex: file reading, network requests, fetching data from database, etc
- Related
  - Process
    - Independent, isolated instance of a running program
    - Contains own dedicated memory space, resources, and threads
  - Thread
    - Smallest unit of execution within a process
    - Shares same memory space & system resources with other threads
    - Note: usually runs in parallel, but not in Python (see below)

Core
- Individual processing unit within CPU (aka processor) responsible for running tasks

Concurrency
- Managing multiple tasks at once
- Tasks are NOT executed at the SAME TIME
- Achieved by rapid switching between tasks (aka interleaving tasks)
- Usually done on single core of CPU
- Analogy: 
  - Single chef in a kitchen that multi-tasks between prepping, cooking, plating, etc. and does one task at a time
- Use case: 
  - I/O-bound tasks where CPU is idle due to waiting for tasks to complete

Cooperative Multitasking
- where running tasks yields control of CPU back 

Preemptive Multitasking
- where OS preempts (interrupts) a running task at anytime and takes back CPU control

Parallelism
- Executing multiple tasks at once
- Tasks are executed at same time, in parallel with other tasks
- Achieved through performing different tasks on different cores
- Requires multi-core CPU
- Analogy:
  - Multi-chef kitchen where multiple chefs work in the same kitchen, cooking meals
- Use case:
  - CPU-bound tasks where much CPU power is needed 

**Python Concepts**
`asyncio` (python module)
- concurrency on single processor using cooperative multitasking
- `async`, `await`
- involves async/await syntax & event loop
- best for handling many slow IO-bound tasks & connections

`threading` (python module)
- concurrency on single processor using preemptive multitasking
- similiar to `asyncio` but more "heavyweight" (takes more memory, more time to task-switch, etc)
- simpler syntax than `asyncio` (no need to structure code for async)
- involves GIL locking (not discussed here)
- best for handling fast, limited number of IO tasks/connections

`multiprocessing` (python module)
- parallelism using multiple cores to run each process on, at the same time
- best for CPU-intensive tasks

**`fastapi` Concepts**
Path operation function definition types
- use `async def` if
  - using code that returns `coroutine`
  - not using blocking code
    - or else you will end up with requests being processed sequentially
- use `def` if using blocking code that does not return `coroutine` (does not support `asyncio`)
  - will use [threading](https://fastapi.tiangolo.com/async/#very-technical-details) in the background to allow other requests to be handled parallely

**`asyncio` concepts**
Event loop
- Queue that manages collection of tasks to be run
- Once a task pauses, control returns to the event loop, and it selects another task to execute
- Continuously cycles through its collection of tasks until there are no more tasks pending execution
  - upon which, event loop will rest and not take up anymore CPU time

Awaitables
- an object that can be used in an `await` expression
- Three main types: coroutines, tasks, and futures

Coroutines
- represents function logic that can be paused or resumed
- Created from `async def` usage
  - even if a `async` function doesn't explicitly have a `return` statement, it will return a `Coroutine`
- Note: needs to be `await`ed or invoked via `asyncio.run()`
- Note: will be executed sequentially/synchronously if not wrapped as a `task`

Tasks
- coroutines that are tied to an event loop
- created via `asyncio.create_task(coroutine)`
- allows for asynchronous execution of coroutines

`asyncio.gather(seq, return_exception=False)`
- runs awaitables in `seq` concurrently
- if any awaitables are coroutines, it is automatically scheduled as a Tasks
- returns aggregate list of returned values
- if `return_exceptions` is False, then the first raised exception is propagated to the task that awaits on `gather()`
  - and other awaitables will continue to run/ won't be cancelled
- if `return_exceptions` is True, then exceptions are returned in the result list 

`asyncio.TaskGroup()`
- Alternative to `asyncio.gather()`
- if any task raises exception, then all remaining tasks will be cancelled