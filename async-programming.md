# Asynchronous Programming
*Last Updated on 21 December 2024, by Wilson Xu*

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
    - Creates and manages the async event loop
    - Runs the main coroutine and handles its completion
    - Replaces traditional synchronous program entry points

### Example Implementation

Python version >= 3.11: using `asyncio.TaskGroup()`to create and run tasks concurrently
 ```python
import asyncio

async def fetch_data(delay):
    print(f"Start fetching data with {delay}s delay")
    await asyncio.sleep(delay)  # Non-blocking sleep
    print(f"Finished fetching data after {delay}s")
    return f"Data from {delay}s task"

async def main():
    # await is implicit when context manager exits
    # context manager ensures resources are closed at the end
    # it also implements internal cancellation; all remaining tasks will be cancelled if one tasks encounters unhandled exception
    async with asyncio.TaskGroup() as tg:
        task_one = tg.create_task(fetch_data(2))
        task_two = tg.create_task(fetch_data(1))        
        results = [task_one.result(), task_two.result()] 

# Run the main coroutine
asyncio.run(main())
```

Python version < 3.11: using `asyncio.gather()` to create and run tasks concurrently
<details><summary>Show code</summary>


```python
async def main():
    task_one = asyncio.create_task(fetch_data(2))
    task_two = asyncio.create_task(fetch_data(1))
    futures = asyncio.gather(task_one, task_two)
    results = await futures  # returns an iterator of results; you can also await each task individually, e.g. result_one = await task_one
    print("All tasks completed:", results)
```
</details>

## ❌ Common mistakes when implementing `asyncio`

### Blocking code in async function
Blocking operations halt the event loop, defeating the purpose of asynchronous programming. Use async library instead to run operations.
```python
# Wrong
async def main():
    print("Start sleeping...")
    time.sleep(3)  # This blocks the event loop!
    print("Done sleeping!")

# Correct
async def main():
    print("Start sleeping...")
    await asyncio.sleep(3)  # Non-blocking sleep
    print("Done sleeping!")
```
```python
# Wrong
async def fetch_url():
    print("Fetching URL...")
    response = requests.get("https://example.com")  # This blocks the event loop!
    print(f"Response: {response.status_code}")

# Correct
async def fetch_url():
    async with aiohttp.ClientSession() as session:
        print("Fetching URL...")
        async with session.get("https://example.com") as response:
            print(f"Response: {response.status}")

```
```python
# Wrong
async def read_file():
    print("Reading file...")
    with open("example.txt", "r") as f:  # Blocking operation
        data = f.read()
    print(f"File content: {data}")

# Correct
async def read_file():
    print("Reading file...")
    async with aiofiles.open("example.txt", "r") as f:
        data = await f.read()  # Non-blocking file read
    print(f"File content: {data}")
```

### Blocking the event loop with synchronous code
Sync code (especially CPU-bound tasks) can block the event loop, reducing the efficiency of async programs.
Avoid long-running sync operations in async code or run sync code in a separate thread or process.

```python
def compute():
    # Simulate a long-running computation
    total = 0
    for i in range(10**7):
        total += i
    return total

# Wrong
async def main():
    print("Starting computation...")
    result = compute()  # Blocking the event loop!
    print(f"Computation result: {result}")

# Correct
async def main():
    print("Starting computation...")
    result = await asyncio.to_thread(compute)  # Offloading to a thread
    print(f"Computation result: {result}")
```
```python
# Example where there's both async and sync code mixing

# Synchronous function (CPU-bound task)
def cpu_bound_task(n):
    print(f"Starting CPU-bound task for {n} iterations...")
    total = sum(i ** 2 for i in range(n))  # Simulate intensive computation
    print(f"CPU-bound task completed with result: {total}")
    return total

# Asynchronous function (I/O-bound task)
async def async_io_task(url):
    print(f"Fetching data from {url}...")
    await asyncio.sleep(2)  # Simulating network delay
    print(f"Data fetched from {url}.")

# Combined main function
async def main():
    url = "https://example.com"
    iterations = 10**6

    # Offload the synchronous task to a thread
    cpu_task = asyncio.to_thread(cpu_bound_task, iterations)  # For heavy CPU-bound tasks, consider ProcessPoolExecutor (Parallelism)

    # Run the async I/O task concurrently
    io_task = async_io_task(url)

    # Await both tasks
    await asyncio.gather(cpu_task, io_task)
```
Visual timeline:
```
Time → 
Task 1 (cpu_task):  ──> Offloaded to Thread ──> Running ─> Done
Task 2 (io_task):   → Yield → Waiting for I/O → Resume → Done
Event Loop:         → Schedule → Manage Tasks → Handle Async Work

```
### Not handling task cancellation
Cancelled tasks might leave resources in an inconsistent state. Tasks are cancelled automatically due to timeout, when another task in the group fails, or when the event loop is terminated. Raise exception handling to handle cancellation and also catch it from the main coroutine.
```python
asyncio.TimeoutError  # exception for timeout
asyncio.CancelledError  # exception for task fail or event loop termination
```

```python
async def task():
    try:
        print("Task started...")
        await asyncio.sleep(5)
        print("Task completed.")
    except asyncio.CancelledError:
        print("Task was cancelled! Cleaning up resources...")
        raise  # Re-raise to propagate the cancellation

async def main():
    t = asyncio.create_task(task())
    await asyncio.sleep(1)
    t.cancel()
    try:
        await t
    except asyncio.CancelledError:
        print("Handled cancellation in main.")
```

### Neglecting the event loop policy for Windows OS
Windows uses a different default event loop policy (SelectorEventLoop) which may not support certain features like subprocesses. Explicitly set the event loop policy on Windows if needed, particularly for subprocess support.
```python
import platform

if platform.system() == "Windows":
    asyncio.set_event_loop_policy(asyncio.WindowsProactorEventLoopPolicy())
```

### Overlooking debugging tools
Ignoring debugging tools for async programs, making errors harder to trace. Debugging async code can be more complex than sync code.
Use asyncio’s debugging features (e.g., `asyncio.run(debug=True))`, log exceptions using `asyncio.Task`, and leverage tools like aiomonitor).

### Forgetting to handle timeout
Failing to set timeouts for long-running operations, unresponsive tasks can hang indefinitely. Use asyncio.wait_for() or timeouts in libraries like aiohttp.
```python
# wrong
async def long_running_task():
    await asyncio.sleep(10)

async def main():
    await long_running_task()  # Hangs indefinitely
# right
async def main():
    try:
        await asyncio.wait_for(long_running_task(), timeout=5)
    except asyncio.TimeoutError:
        print("Task timed out!")
```

### Overloading the event loop
Spawning too many tasks, overwhelming the event loop. The event loop might become sluggish or run out of resources. Limit concurrent tasks with mechanisms like asyncio.Semaphore.
```python
# wrong
async def task():
    await asyncio.sleep(1)

async def main():
    tasks = [task() for _ in range(100000)]  # Too many tasks
    await asyncio.gather(*tasks)

# right
sem = asyncio.Semaphore(10)

async def task():
    async with sem:
        await asyncio.sleep(1)

async def main():
    tasks = [task() for _ in range(1000)]
    await asyncio.gather(*tasks)
```

## 🗃️ Other useful asyncio functions:
| Primitive         | Use Case                                   | Example Use Case                                      |
|--------------------|-------------------------------------------|-----------------------------------------------------|
| `asyncio.Lock`     | Prevent concurrent access to resources    | Updating a shared variable                          |
| `asyncio.Event`    | Notify tasks of a condition               | Signal completion of a task                        |
| `asyncio.Condition`| Notify multiple tasks of a condition      | Coordinate threads waiting for a specific state    |
| `asyncio.Semaphore`| Limit the number of concurrent tasks      | Restrict HTTP connections to an API                |
| `asyncio.Queue`    | FIFO task communication                   | Producer-consumer model                            |
| `asyncio.PriorityQueue` | Priority-based task communication    | Task scheduling based on priority                  |
| `asyncio.LifoQueue`| LIFO task communication                   | Reverse-order processing (e.g., undo operations)   |


## 💻 Asyncio with FastAPI 
FastAPI is commonly used as a backend framework for web applications. Here are some useful code snippets on implementing async programming to FastAPI
### REST APIs Endpoint
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

### WebSocket Endpoint
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

### Lifespan Events
```python
# Example shared resource
class SharedResource:
    def __init__(self):
        self.data = None

    async def initialize(self):
        print("Initializing resource...")
        await asyncio.sleep(2)  # Simulate async initialization
        self.data = "Resource Ready"
        print("Resource initialized.")

    async def cleanup(self):
        print("Cleaning up resource...")
        await asyncio.sleep(1)  # Simulate async cleanup
        self.data = None
        print("Resource cleaned up.")

# Create the shared resource instance
shared_resource = SharedResource()

# Define the FastAPI app
app = FastAPI()

# Define lifespan events
@app.on_event("startup")
async def on_startup():
    print("Application is starting...")
    await shared_resource.initialize()

@app.on_event("shutdown")
async def on_shutdown():
    print("Application is shutting down...")
    await shared_resource.cleanup()

@app.get("/resource")
async def get_resource():
    if shared_resource.data:
        return {"resource": shared_resource.data}
    return {"error": "Resource not initialized"}
```

### Example 4: FastAPI WebSocket Connection with Concurrency

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



















