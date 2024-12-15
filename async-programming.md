# Asynchronous Programming
*Last Updated on 16 December 2024, by Wilson Xu*

## 📚 Why Asynchronous Programming is Needed

- In traditional **Synchronous programming**, code executes sequentially, with each operation blocking the program's execution until it completely finishes.
    - Some operations inherently require longer execution times, especially when dependent on external services.
    - Execution time is predominantly spent **waiting**, during which the program remains entirely idle.
    - These time-consuming operations are classified as **Input and Output operations (I/O-bound Operations)**.
        - Examples include:
            - Performing database operations (queries and commits)
            - Sending API requests (HTTP, Websockets, AI Model inference)
            - File operations (reading and writing)
    - The main thread halts and waits for each operation to complete before proceeding, creating potential performance bottlenecks.

- To optimize resource utilization and reduce idle time, **Asynchronous programming** enables operations to execute independently and concurrently.
    - A program can initiate a potentially long-running operation and immediately proceed with other tasks while waiting for the operation's response.
    - This approach maintains a single thread but allows for more efficient task management (distinct from **parallelism** / **multithreading**).
    - Async operations can start, pause, and resume execution, maintaining their state during intervals, thereby maximizing computational efficiency.

### Illustration of flow
```
┌─────────────────────────┐   ┌─────────────────────────┐
│    SYNCHRONOUS FLOW     │   │   ASYNCHRONOUS FLOW     │
├─────────────────────────┤   ├─────────────────────────┤
│                         │   │                         │
│  ╔════════════════╗     │   │  ╔════════════════╗     │
│  ║   TASK 1       ║     │   │  ║   TASK 1       ║     │
│  ╚════════════════╝     │   │  ╚════════════════╝     │
│          ↓              │   │    ┌ ─ ─ ─ ─ ─ ─ ┐      │
│                         │   │    │  WAITING... │      │
│  ╔════════════════╗     │   │  ╔════════════════╗     │
│  ║   TASK 2       ║     │   │  ║   TASK 2       ║     │
│  ╚════════════════╝     │   │  ╚════════════════╝     │
│          ↓              │   │    ┌ ─ ─ ─ ─ ─ ─ ┐      │
│                         │   │    │  WAITING... │      │
│  ╔════════════════╗     │   │  ╔════════════════╗     │
│  ║   TASK 3       ║     │   │  ║   TASK 3       ║     │
│  ╚════════════════╝     │   │  ╚════════════════╝     │
│                         │   │                         │
└─────────────────────────┘   └─────────────────────────┘
        TIME ➡️                      TIME ➡️

SYNCHRONOUS:                 ASYNCHRONOUS:
• Tasks execute sequentially  • Tasks can start concurrently
• Each task blocks execution  • Non-blocking task execution
• Total time = Sum of all     • Total time ≈ Longest task 
task times                    duration
```

## 💡 Asyncio: Python library for Asychronous Programming
`asyncio` is a library that enables writing **concurrent** code using the **async/await** syntax, allowing efficient I/O-bound and high-level structured network code.

### Key Async Programming Constructs
1. `async` Keyword
    - Defines a coroutine (a special type of function that can be paused and resumed)
    - Indicates that the function will work asychronously (or is expected to eventually)
    - Cannot be called directly like a normal function; must be awaited or scheduled
2. `await` Keyword
    - Pauses the execution of the current coroutine (the function itself)
    - Allows the event loop to run other tasks while waiting for an operation to complete
    - Only used inside async functions
    - Waits for an "awaitable" object to complete
        - Awaitable objects include: coroutines, tasks, and futures
        - Example: `await asyncio.sleep(2) vs time.sleep(2)
            - `asyncio.sleep()` releases control back to the event loop
            - `time.sleep()` blocks the entire thread
3. `asyncio.create_task(func(*args))`
    - Schedules a coroutine to run as a Task
    - Allows multiple coroutines to run concurrently
    - Returns a Task object that can be awaited
4. `asyncio.gather(*tasks)`
    - Schedules all tasks
    - returns a Futures object that represents the eventual results of asychronous operation
5. `asyncio.run(main())`
    - Entry point for async programs
    - Manages the async event loop
    - Runs the main coroutine and handles its completion
    - Replaces traditional synchronous program entry points

### Example Implementation

 ```python
import asyncio

async def fetch_data(delay):
    print(f"Start fetching data with {delay}s delay")
    await asyncio.sleep(delay)  # Non-blocking sleep
    print(f"Finished fetching data after {delay}s")
    return f"Data from {delay}s task"

async def main():
    # Create multiple tasks to run concurrently
    # Python 3.11: asyncio.TaskGroup class is a more modern alternative to create and running tasks, await is implicit when context manager exits
    # It also implements internal cancellation, if one task fails with an unhandled exception, all remaining tasks will be cancelled
    async with asyncio.TaskGroup() as tg:
        task_one = tg.create_task(fetch_data(2))
        task_two = tg.create_task(fetch_data(1))        
        results = [task_one.result(), task_two.result()] 

    # Older method: using asyncio.gather
    # task_one = asyncio.create_task(fetch_data(2))
    # task_two = asyncio.create_task(fetch_data(1))
    # futures = asyncio.gather(task_one, task_two)
    # results = await futures  # returns an iterator of results; you can also await each task individually, e.g. result_one = await task_one
    print("All tasks completed:", results)

# Run the main coroutine
asyncio.run(main())
```

## 💻 Asyncio with FastAPI 

### Example 1: FastAPI HTTP Endpoint
<details><summary>Show code</summary>

```python
app = FastAPI()

# Simulated async database or service functions
async def get_user_data(user_id):
    await asyncio.sleep(1)  # Simulate async database query
    return {"id": user_id, "name": "John Doe"}

@app.get("/users/{user_id}")
async def read_user(user_id: int):
    # Await an async function to fetch user data
    user_data = await get_user_data(user_id)
    return user_data

```
</details>

### Example 2: FastAPI WebSocket Endpoint
<details><summary>Show code</summary>

```python
app = FastAPI()

async def save_message(user_id, message):
    await asyncio.sleep(0.5)  # Simulate async database save
    return True

@app.websocket("/ws/{user_id}")
async def websocket_endpoint(websocket: WebSocket, user_id: int):
    await websocket.accept()
    try:
        while True:  # The loops ensures the server is always receiving messages from the client
            # Await receiving a message
            data = await websocket.receive_text()
            # Await saving the message
            saved = await save_message(user_id, data)
            if saved:
                # Await sending a response
                await websocket.send_text(f"Message saved: {data}")
    except WebSocketDisconnect:
        print(f"User {user_id} disconnected")
```
</details>

### Example 3: FastAPI startup_events
<details><summary>Show code</summary>

```python
app = FastAPI()

async def some_async_startup_task():
    await asyncio.sleep(2)
    print("Startup task completed")

@app.on_event("startup")
async def startup_event():
    # Perform async operations on startup
    await some_async_startup_task()
```
</details>

### Example 4: FastAPI WebSocket Connection with Concurrency
<details><summary>Show code</summary>

```python
app = FastAPI()

# Simulated async services
async def process_sensor_data(sensor_id):
    await asyncio.sleep(2)  # Simulate processing delay
    return f"Processed data for sensor {sensor_id}"

async def save_sensor_log(sensor_id, data):
    await asyncio.sleep(1)  # Simulate database save
    return True

class ConnectionManager:
    def __init__(self):
        # Maintains a list of all connected clients
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def broadcast(self, message: str):
        # Broadcasting to all connected clients
        for connection in self.active_connections:
            await connection.send_text(message)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        # Send a message to a specific client
        await websocket.send_text(message)

    async def disconnect_client(self, websocket: WebSocket):
        # Properly close and remove a specific client
        await websocket.close()
        self.disconnect(websocket)

connection_manager = ConnectionManager()

@app.websocket("/ws/sensors")
async def websocket_sensor_endpoint(websocket: WebSocket):
    await connection_manager.connect(websocket)
    
    try:
        while True:
            # Receive sensor data
            data = await websocket.receive_text()
            # Create concurrent tasks for processing and logging
            process_task = asyncio.create_task(process_sensor_data(data))
            log_task = asyncio.create_task(save_sensor_log(data, "raw_data"))
            # Wait for both tasks concurrently
            processed_data, log_result = await asyncio.gather(
                process_task, 
                log_task
            )
            # Send results back to client
            await websocket.send_text(f"Result: {processed_data}, Logged: {log_result}")
    
    except WebSocketDisconnect:
        connection_manager.disconnect(websocket)
        await websocket.close()

# Client-side example for demonstration
async def websocket_client():
    import websockets
    
    uri = "ws://localhost:8000/ws/sensors"
    async with websockets.connect(uri) as websocket:
        # Send multiple sensor IDs
        for sensor_id in range(5):
            await websocket.send(str(sensor_id))
            response = await websocket.recv()
            print(f"Received: {response}")
```
</details>


## ✅ Best Practices in Asychronous Programming
- timeout
- queues: queues can be used to distribute workload between several concurrent tasks, https://docs.python.org/3/library/asyncio-queue.html
- to_thread
- mutual lock
- event


## ❌ Bad Practices in Asynchronous Programming







- To cancel all tasks if one task in the group fail is automatic; previously this was done manually. [Asyncio gather() cancel all tasks if one task fails](https://superfastpython.com/asyncio-gather-cancel-all-if-one-fails/#How_to_Cancel_All_Tasks_if_One_Task_Fails_the_wrong_way)

















## 🤯 Other Useful Primitives

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





##  Issues with async programming











Online Tutorials:
- [https://superfastpython.com/category/asyncio/]
- [https://www.youtube.com/watch?v=Qb9s3UiMSTA]
- [https://fastapi.tiangolo.com/async/]

Last Updated: 9 December 2024
Author: Wilson Xu