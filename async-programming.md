# Asynchronous Programming


## ❗️ Issue: Execution time of I/O operations in synchronous programming

- **Synchronous programming** is often used for simple applications; tasks are performed one at a time in the order they are called, and must finish before the next one can begin.

- **Input and Output operations** are relatively slower processes. Examples of these processes include:
    - Database operations (queries and commits)
    - API Requests (HTTP, Websockets, AI Model inference)
    - File operations (reading and writing)

- Execution time of the operation is **mainly due to waiting**; the program is doing nothing while waiting for the exact moment that the task finishes before proceeding with other operations.


## 💡 Approach: Asynchronous Programming

- **Asynchronous programming** allows a program to start a potentially long-running task and still be able to be responsive to run other events while that task runs, instead of having to wait until that task has finished to proceed.

- Also known as **Coroutines**, these functions can start, process and finish in overlapping periods of time because they can pause and resume executions; their states are maintained during pauses. 

- This approach achieves **concurrency**; overall execution time of all operations within a program is reduced, as other tasks can now run while waiting for another task's operation to complete. 


## 📚 Asyncio: Python library for Asychronous Programming

### Basic asyncio
Here is a sample code of how to use asyncio to implement asychronous programming in Python:

```python
import asyncio

async def fetch_data(name):  # Define coroutine with 'async' keyword at function definition >> returns a Coroutine object
    await asyncio.sleep(1)  # Simulate I/O bound operation, use 'await' keyword to specify where the coroutine can pause and yield control to other coroutines; you can only put in front of 'awaitable' commands (e.g. asyncio.sleep() and not time.sleep())
    return f"Data fetched for {name}"

async def main():  # Define main coroutine, needs 'async' because of the 'await asyncio.gather'
    
    # Create tasks from Coroutine objects
    tasks = [
        asyncio.create_task(fetch_data("Task 1")),
        asyncio.create_task(fetch_data("Task 2"))
    ]
    
    # Wait for all tasks to complete
    results = await asyncio.gather(*tasks)  # Execution here

    # You can also get the results individually
    # task_one = asyncio.create_task(fetch_data("Task 1"))
    # task_two = asyncio.create_task(fetch_data("Task 1"))
    # result_one = await task_one 
    # result_two = await task_two
    
    # Get individual results
    for result in results:
        print(result)

asyncio.run(main())  # To run the main coroutine; Cannot just use main()
```

### TaskGroup
- TaskGroup is used to create and manage a collection of tasks. 
- It is intended as replacement for the *create_task()* and *gather()* function for waitng on a gtoup of tasks.
- To cancel all tasks if one task in the group fail is automatic; previously this was done manually. [Asyncio gather() cancel all tasks if one task fails](https://superfastpython.com/asyncio-gather-cancel-all-if-one-fails/#How_to_Cancel_All_Tasks_if_One_Task_Fails_the_wrong_way)

```python
async def main():
    tasks = []

    # wait on tasks in group by exiting the asynchronous context manager block

    # if one task in the group fail, all non-done tasks remaining in the group will be cancelled.

    async with asyncio.TaskGroup() as tg:
        tasks = [
            asyncio.create_task(fetch_data("Task 1")),
            asyncio.create_task(fetch_data("Task 2"))
        ]

    # at the end of the async with block, the tasks have already been executed        
    results = [task.results() for task in tasks]

    for result in results:
        print(f"Received results")
```

### Mutual exclusion Lock
- When two or more coroutines operate upon the same variables, it is possible to suffer race conditions
- Mutex locks can be used to protect critical sections of code from concurrent execution; atomic blocks of codes can be made sequential
- It is coroutine-safe, but not thread-safe or process-safe.

```python
lock = asyncio.Lock()

async def modify_shared_resource():
    global shared_resource
    async with lock:  # Using context manager is recommended over "lock.acquire()" with "lock.release()"
        # critical section starts
        print(f"Resource before modification: {shared_resource}")
        shared_resource += 1
        print(f"Resource after modification: {shared_reousrce}")
        # critical section ends

async def main():
    await asyncio.gather(*(modify_shared_resource() for _ in range(5)))

asyncio.run(main())
```

### Event
- An event allows communication between coroutines.
- The event objects can be either "set" or "not set"
- When first created, it is implicitly "not set" by default.

```python
# task coroutine
async def task(event, number):
    # wait for the event to be set
    await event.wait()
    # generate a random value between 0 and 1
    value = random()
    # block for a moment
    await asyncio.sleep(value)
    # report a message
    print(f'Task {number} got {value}')
 
# main coroutine
async def main():
    # create a shared event object
    event = asyncio.Event()
    # create and run the tasks
    tasks = [asyncio.create_task(task(event, i)) for i in range(5)]
    # allow the tasks to start
    print('Main blocking...')
    await asyncio.sleep(0)
    # start processing in all tasks
    print('Main setting the event')
    event.set()
    # await for all tasks  to terminate
    _ = await asyncio.wait(tasks)
```

### to_thread()
- Using a blocking call (synch function) suspends the thread in which the coroutine is running (thread blocking call), instead of just suspending the current coroutine.
- The "to_thread()" function call allows the asyncio event loop to treat a blocking function call as a coroutine and execute asynchronously using thread-based concurrency instead of coroutine-based concurrency.

```python
# blocking function
# blocking function
def blocking_task():
    # report a message
    print('task is running')
    # block
    time.sleep(2)
    # report a message
    print('task is done')
 
# background coroutine task
async def background():
    # loop forever
    while True:
        # report a message
        print('>background task running')
        # sleep for a moment
        await asyncio.sleep(0.5)
 
# main coroutine
async def main():
    # run the background task
    _= asyncio.create_task(background())
    # create a coroutine for the blocking function call
    coro = asyncio.to_thread(blocking_task)
    # execute the call in a new thread and await the result
    await coro
```


Online Tutorials:
- [https://superfastpython.com/category/asyncio/]
- [https://www.youtube.com/watch?v=Qb9s3UiMSTA]
- [https://fastapi.tiangolo.com/async/]

Last Updated: 9 December 2024
Author: Wilson Xu