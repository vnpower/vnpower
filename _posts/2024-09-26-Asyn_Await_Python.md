---
layout: post
author: Vincent Khuat
title: Simplest async/await example possible in Python
---

# Case 1: just normal Python
## Python code
The code below can be test using [W3School](https://www.w3schools.com/python/trypython.asp?filename=demo_ref_print3)
```python
import time
def sleep():
    print(f'Time: {time.time() - start:.2f}')
    time.sleep(1)

def sum(name, numbers):
    total = 0
    for number in numbers:
        print(f'Task {name}: Computing {total}+{number}')
        sleep()
        total += number
    print(f'Task {name}: Sum = {total}\n')

start = time.time()
tasks = [
    sum("A", [1, 2]),
    sum("B", [1, 2, 3]),
]
end = time.time()
print(f'Time: {end-start:.2f} sec')
```
## Output:

    Task A: Computing 0+1
    Time: 0.00
    Task A: Computing 1+2
    Time: 1.00
    Task A: Sum = 3

    Task B: Computing 0+1
    Time: 2.01
    Task B: Computing 1+2
    Time: 3.01
    Task B: Computing 3+3
    Time: 4.01
    Task B: Sum = 6

    Time: 5.02 sec

# Case 2: async/await done wrong
## Python code
```python
import asyncio
import time

async def sleep():
    print(f'Time: {time.time() - start:.2f}')
    time.sleep(1)

async def sum(name, numbers):
    total = 0
    for number in numbers:
        print(f'Task {name}: Computing {total}+{number}')
        await sleep()
        total += number
    print(f'Task {name}: Sum = {total}\n')

start = time.time()

loop = asyncio.get_event_loop()
tasks = [
    loop.create_task(sum("A", [1, 2])),
    loop.create_task(sum("B", [1, 2, 3])),
]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

end = time.time()
print(f'Time: {end-start:.2f} sec')
```
## Output:

    Task A: Computing 0+1
    Time: 0.00
    Task A: Computing 1+2
    Time: 1.00
    Task A: Sum = 3

    Task B: Computing 0+1
    Time: 2.01
    Task B: Computing 1+2
    Time: 3.01
    Task B: Computing 3+3
    Time: 4.01
    Task B: Sum = 6

    Time: 5.01 sec

# Case 3: async/await done right
## Python code:
```python
import asyncio
import time

async def sleep():
    print(f'Time: {time.time() - start:.2f}')
    await asyncio.sleep(1)

async def sum(name, numbers):
    total = 0
    for number in numbers:
        print(f'Task {name}: Computing {total}+{number}')
        await sleep()
        total += number
    print(f'Task {name}: Sum = {total}\n')

start = time.time()

loop = asyncio.get_event_loop()
tasks = [
    loop.create_task(sum("A", [1, 2])),
    loop.create_task(sum("B", [1, 2, 3])),
]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

end = time.time()
print(f'Time: {end-start:.2f} sec')
```

## Output:

    Task A: Computing 0+1
    Time: 0.00
    Task B: Computing 0+1
    Time: 0.00
    Task A: Computing 1+2
    Time: 1.00
    Task B: Computing 1+2
    Time: 1.00
    Task A: Sum = 3
    
    Task B: Computing 3+3
    Time: 2.00
    Task B: Sum = 6

    Time: 3.01 sec

# Explaination
Case 1 and case 2 give the same 5 seconds, whereas case 3 just 3 seconds. So the async/await done right is faster.

The reason for the difference is within the implementation of the sleep function.
```python
# Case 1
def sleep():
    ...
    time.sleep(1)

# Case 2
async def sleep():
    ...
    time.sleep(1)

# Case 3
async def sleep():
    ...
    await asyncio.sleep(1)
```
In case 1 and case 2, they are the "same": they "sleep" without allowing others to use the resources. Whereas in case 3, it allows access to the resources when it is asleep.

In case 2, we added async to the normal function. However the event loop will run it without interruption. Why? Because we didn't say where the loop is allowed to interrupt your function to run another task.

In case 3, we told the event loop exactly where to interrupt the function to run another task. Where exactly? Right here!
```python
await asyncio.sleep(1)
```
For more on this, read [here](https://djangostars.com/blog/asynchronous-programming-in-python-asyncio/).

Consider reading:

[A Hitchhikers Guide to Asynchronous Programming](https://github.com/crazyguitar/pysheeet/blob/master/docs/appendix/python-concurrent.rst)

[Asyncio Futures and Coroutines](https://autobahn.readthedocs.io/en/latest/asynchronous-programming.html#asyncio-futures-and-coroutines)
