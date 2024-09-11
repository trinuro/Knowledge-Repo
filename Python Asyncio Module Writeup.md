1. Async is good for network requests and reading files (Things that wait a lot)
	1. Threads are good for applications that uses IO and share some data asynchronously but is not CPU intensive
	2. Processes are good for tasks that are CPU intensive (Maximum performance)
2. Terms:
	1. Event loop: Basically a manager that will handle one task at a time but when the task is in waiting state, it will release it and work on another task.
	2. Coroutine: An asynchronous function 
	3. Coroutine object: What an asynchronous function returns
	4. Futures: A promise of a future result
3. To import `asyncio`,
```python
import asyncio
```

# Simple Example
1. To define a coroutine function, use `async` keyword
```python
async def main():
    print('Start of main coroutine')
```
2. We cannot call main() straight away because it returns a Coroutine object. Since we cannot `await` outside of an asynchronous function, we will use `asyncio` to run the code.
```python
# coroutine
async def main():
    print('Start of main coroutine')

asyncio.run(main())
```
Output:
```
Start of main coroutine
```
3. To pause a coroutine but not pause the whole script, use `sleep`
```python
import asyncio

async def hello():
    print('Hello ...')
    await asyncio.sleep(1)
    print('... World!')

async def main():
    await asyncio.gather(hello(), hello())

asyncio.run(main())
```
Output:
```
Hello ...
Hello ...
... World!
... World!
```
In contrast, when we use `time.sleep(1)`,
```python
import asyncio
import time

async def hello():
    print('Hello ...')
    await time.sleep(1)
    print('... World!')

async def main():
    await asyncio.gather(hello(), hello())

asyncio.run(main())
```
Output:
```
Hello ...
... World!
Hello ...
... World!
```
- This is because `time.sleep` is blocking while `asyncio.sleep` is not.
 4. To execute an asynchronous function (in another asynchronous function), we need to use `await` on a Coroutine object.
```python
import asyncio

async def fetch_data():
    print('Fetching data')
    await asyncio.sleep(1)

async def main():
    print('Start of main coroutine')
    coroutine = fetch_data()
    print("function is not executed!")
    result = await coroutine
    print("Fetched data!")
    print(result)


asyncio.run(main())
```
Output:
```
Start of main coroutine
function is not executed!
Fetching data
Fetched data!
None
```
5. We must be careful not to use any blocking code between the `await`s that we have.
```python
import asyncio

async def fetch_data():
    print('Fetching data')
    await asyncio.sleep(1)
    return('Fetched data!')

async def main():
    coroutine1 = fetch_data()
    coroutine2 = fetch_data()

    res1 = await coroutine1
    print(res1)
    res2 = await coroutine2
    print(res2)

asyncio.run(main())
```
Output:
```
Fetching data
Fetched data!
Fetching data
Fetched data!
```
6. To create tasks, use `asyncio.create_task`
```python
import asyncio

async def fetch_data():
    print('Fetching data')
    await asyncio.sleep(1)
    return('Fetched data!')

async def main(): 
    coroutine1 = asyncio.create_task(fetch_data()) # pass in a coroutine object
    coroutine2 = asyncio.create_task(fetch_data())

    res1 = await coroutine1
    res2 = await coroutine2
    print(res2)
    print(res1)

asyncio.run(main())
```
- Both tasks will start at a much closer time with `asyncio.create_task`
Output:
```
Fetching data
Fetching data
Fetched data!
Fetched data!
```
7. Say we want to wait for certain async functions to execute, we can specify await on before the subroutine is created.
```python
import asyncio

async def fetch_data():
    print('Fetching data')
    await asyncio.sleep(1)
    return('Fetched data!')

async def main(): 
    coroutine1 = asyncio.create_task(fetch_data()) # pass in a coroutine object
    res1 = await coroutine1
    coroutine2 = asyncio.create_task(fetch_data())
    coroutine3 = asyncio.create_task(fetch_data())
    
    res2 = await coroutine2
    res3 = await coroutine3
    print((res1,res2,res3))

asyncio.run(main())
```
- Code will wait for coroutine1 to finish executing
# Advanced
1. To specify a list of Coroutines to execute together, use `asyncio.gather`
```python
import asyncio

async def fetch_data():
    print('Fetching data')
    await asyncio.sleep(1)
    return('Fetched data!')

async def main():
    outputs = await asyncio.gather(fetch_data(), fetch_data())
    for i in outputs:
        print(i)


asyncio.run(main())
```
 - The results is returned in an array
 - Equivalent to `create_task` and `await`
2. However, here's a better way to do it (supports iteration).
```python
async def main():
    tasks = []
    async with asyncio.TaskGroup() as tg:
        for _ in range(10):
            task = tg.create_task(fetch_data()) # Coroutines as input
            tasks.append(task) # Add task
    results = [task.result() for task in tasks] # task.result() will execute the async function
```
3. A lock is sort of a ticket for coroutines to access a shared resource. To provide a lock for coroutines to use,
```python
shared_resource = 0
lock = asyncio.Lock() # Create a lock

async def fetch_data():
    global shared_resource # inherit the global variable "shared_resource"
    async with lock: # check whether a coroutine is using a Lock object
        print(shared_resource)
        shared_resource+=1
        await asyncio.sleep(1)
```
- Prevents race conditions
- You can use the async functions as normal
4. To throttle our requests, use `semaphore`
```python
import asyncio

async def fetch_data(semaphore, id):
    async with semaphore: # Perform semaphore check
        await asyncio.sleep(1)
        print(id)
        return id

async def main():
    semaphore = asyncio.Semaphore(2) # Only a maximum of 2 tasks of the same kind at a time
    tasks = []
    async with asyncio.TaskGroup() as tg:
        for i in range(5):
            task = tg.create_task(fetch_data(semaphore, i)) # Coroutines as input
            tasks.append(task) # Add task
    results = [task.result() for task in tasks] # Execute tasks
    for result in results:
        print(result)

asyncio.run(main())
```
5. To create events (This forces another async function to wait for event to be set),
```python
async def waiter(event : asyncio.Event):
    print('Waiting for event to be set')
    await event.wait() # Await a "global" flag to be set 
    print('Event set and ready')


async def setter(event : asyncio.Event):
    await asyncio.sleep(2)
    event.set() # Set the global flag
    print('Event set')

async def main():
    event = asyncio.Event() # Create a global flag
    await asyncio.gather(waiter(event), setter(event)) # Both async functions will be passed in the same global flag

asyncio.run(main())
```
- `waiter` will need to wait for the flag to be set in the `setter` although `setter` comes before `waiter`.