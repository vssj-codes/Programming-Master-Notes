# Python — Round 2 Prep (Uber Client Round)

---

## 1. Threading & Async Programming

### Process vs Thread
- **Process** = a running program with its own memory, file handles, resources. Isolated.
- **Thread** = a worker inside a process. All threads share the same memory.

**Mental model:**
```
Process = restaurant building
  ├── shared kitchen (memory)
  ├── Thread 1 = Waiter 1
  ├── Thread 2 = Waiter 2
  └── Thread 3 = Waiter 3
```

| | Process | Thread |
|---|---|---|
| Memory | Separate | Shared |
| Speed to create | Slow | Fast |
| Communication | Hard (IPC) | Easy (shared memory) |
| Crash impact | Isolated | One thread can crash all |

---

### I/O-bound vs CPU-bound
- **I/O-bound** — slow because it waits (API calls, DB queries, file reads). Threading/async helps.
- **CPU-bound** — slow because of heavy computation (ML, image processing). Threading doesn't help due to GIL. Use Multiprocessing.

---

### Threading
Run multiple threads inside one process. Good for I/O-bound tasks.

```python
import time
import threading

def fetch_data(source):
    print(f"Fetching {source}...")
    time.sleep(2)
    print(f"Done: {source}")

threads = []
for source in ["API 1", "API 2", "API 3"]:
    t = threading.Thread(target=fetch_data, args=(source,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
# Total: ~2s instead of 6s
```

- `t.start()` — starts the thread
- `t.join()` — main thread waits for this thread to finish

---

### Race Condition & Lock
`counter += 1` is 3 steps: read → add → write. Two threads can interleave and overwrite each other.

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1  # only one thread at a time

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)
t1.start(); t2.start()
t1.join(); t2.join()

print(counter)  # 200000 — correct
```

---

### Async Programming (asyncio)
Alternative to threading for I/O-bound tasks. Uses a single thread with an **event loop**.

**Key idea:** Instead of blocking and waiting, `await` suspends the current task and lets the event loop run other tasks.

```python
import asyncio

async def fetch_data(source):
    print(f"Fetching {source}...")
    await asyncio.sleep(2)  # non-blocking wait
    print(f"Done: {source}")

async def main():
    await asyncio.gather(
        fetch_data("API 1"),
        fetch_data("API 2"),
        fetch_data("API 3"),
    )
# Total: ~2s

asyncio.run(main())
```

- `async def` — defines a coroutine
- `await` — suspends current coroutine, gives control back to event loop
- `asyncio.gather()` — runs multiple coroutines concurrently

---

### Threading vs Async

| | Threading | Async |
|---|---|---|
| Model | Multiple threads | Single thread, event loop |
| Best for | I/O-bound | I/O-bound |
| Race conditions | Yes (shared memory) | Rare (single thread) |
| Overhead | Higher (OS manages threads) | Lower |
| Code complexity | Simpler | Requires async/await everywhere |

**Rule of thumb:**
- Use `async` for high-concurrency network I/O (FastAPI does this)
- Use `threading` when working with libraries that don't support async

---

## 2. Multiprocessing

### Why Threading Fails for CPU-bound Tasks (GIL)
Python has a **Global Interpreter Lock (GIL)** — only one thread can execute Python bytecode at a time, even on multi-core machines.

For I/O-bound tasks, threads release the GIL while waiting → still useful.
For CPU-bound tasks, threads fight over the GIL → no real parallelism.

**Fix:** Use `multiprocessing` — each process has its own Python interpreter and GIL.

---

### multiprocessing.Process
```python
from multiprocessing import Process

def compute(n):
    result = sum(i * i for i in range(n))
    print(f"Result: {result}")

p1 = Process(target=compute, args=(10000000,))
p2 = Process(target=compute, args=(10000000,))

p1.start(); p2.start()
p1.join(); p2.join()
# Runs on 2 CPU cores simultaneously
```

---

### multiprocessing.Pool
For distributing work across multiple processes easily.

```python
from multiprocessing import Pool

def square(n):
    return n * n

with Pool(processes=4) as pool:
    results = pool.map(square, [1, 2, 3, 4, 5])
    print(results)  # [1, 4, 9, 16, 25]
```

`pool.map()` splits the list across 4 processes and collects results.

---

### Sharing Data Between Processes
Processes have separate memory. To share data:

```python
from multiprocessing import Value, Array

counter = Value('i', 0)   # shared integer
arr = Array('d', [1.0, 2.0, 3.0])  # shared array
```

Or use `Queue` for passing messages between processes:

```python
from multiprocessing import Process, Queue

def worker(q):
    q.put("result from worker")

q = Queue()
p = Process(target=worker, args=(q,))
p.start()
print(q.get())  # "result from worker"
p.join()
```

---

### Threading vs Multiprocessing vs Async

| | Threading | Multiprocessing | Async |
|---|---|---|---|
| Best for | I/O-bound | CPU-bound | I/O-bound (high concurrency) |
| Parallelism | No (GIL) | Yes (multiple cores) | No (single thread) |
| Memory | Shared | Separate | Shared |
| Overhead | Low | High | Lowest |

---

## 3. Object Model — Why Everything in Python is an Object

### The Core Idea
In Python, **everything** is an object — integers, strings, functions, classes, modules.

An object is an instance of a class with:
- **Identity** — memory address (`id()`)
- **Type** — what kind of object it is (`type()`)
- **Value** — the data it holds

```python
x = 42
print(type(x))   # <class 'int'>
print(id(x))     # memory address
print(x.__class__)  # <class 'int'>
```

---

### Even Functions are Objects
```python
def greet():
    return "hello"

print(type(greet))      # <class 'function'>
print(id(greet))        # has a memory address
greet.custom = "yes"    # you can add attributes to it
```

Because functions are objects, you can:
- Pass them as arguments
- Return them from functions
- Store them in lists/dicts

This is called **first-class functions**.

---

### Even Classes are Objects
Classes are instances of `type` (the metaclass).

```python
class Dog:
    pass

print(type(Dog))    # <class 'type'>
print(type(int))    # <class 'type'>
```

`type` is the class of all classes.

---

### `__dict__` — Where Attributes Live
Every object stores its attributes in a dictionary.

```python
class Person:
    def __init__(self, name):
        self.name = name

p = Person("Alice")
print(p.__dict__)  # {'name': 'Alice'}
```

---

### Why This Matters in Interviews
- Explains decorators (functions wrapping functions — possible because functions are objects)
- Explains how Python passes arguments (everything passed by object reference)
- Explains duck typing — Python checks what an object can do, not what type it is

---

## 4. `is` vs `==`

### `==` — Value Equality
Checks if two objects have the **same value**.

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a == b)  # True — same values
```

---

### `is` — Identity Equality
Checks if two variables point to the **same object in memory**.

```python
a = [1, 2, 3]
b = [1, 2, 3]

print(a is b)  # False — different objects in memory

c = a
print(a is c)  # True — same object
```

---

### Integer Caching (Common Gotcha)
Python caches small integers from **-5 to 256**. They always return the same object.

```python
a = 100
b = 100
print(a is b)  # True — cached, same object

a = 1000
b = 1000
print(a is b)  # False — not cached, different objects
```

This is an implementation detail of CPython. Don't rely on it.

---

### String Interning (Another Gotcha)
Python interns short strings that look like identifiers.

```python
a = "hello"
b = "hello"
print(a is b)  # True — interned

a = "hello world"
b = "hello world"
print(a is b)  # False (may vary)
```

---

### Rule of Thumb
- Use `==` to compare values (almost always what you want)
- Use `is` only to check against `None`: `if x is None`
- Never use `is` to compare integers, strings, or lists

---

## 5. Decorator

### What is a Decorator?
A decorator is a function that **wraps another function** to add behavior before/after it runs — without modifying the original function.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def greet(name):
    print(f"Hello {name}")

greet("Alice")
# Before
# Hello Alice
# After
```

`@my_decorator` is syntactic sugar for `greet = my_decorator(greet)`.

---

### Preserving Function Metadata
Without `functools.wraps`, the wrapped function loses its name and docstring.

```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # preserves original function's metadata
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper
```

---

### Decorator with Arguments
```python
def repeat(n):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(n):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def say_hi():
    print("Hi")

say_hi()
# Hi
# Hi
# Hi
```

---

### Real-World Use Cases

**Logging:**
```python
def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}")
        return func(*args, **kwargs)
    return wrapper
```

**Auth check (FastAPI style):**
```python
def require_auth(func):
    @functools.wraps(func)
    def wrapper(request, *args, **kwargs):
        if not request.user.is_authenticated:
            raise Exception("Unauthorized")
        return func(request, *args, **kwargs)
    return wrapper
```

**Timing:**
```python
import time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper
```

---

### Class-Based Decorator
```python
class Timer:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        start = time.time()
        result = self.func(*args, **kwargs)
        print(f"Took {time.time() - start:.2f}s")
        return result

@Timer
def slow_function():
    time.sleep(1)
```

---

## 6. Generator

### What is a Generator?
A generator is a function that **yields values one at a time** instead of returning all at once. It produces values lazily — only when asked.

```python
def count_up(n):
    for i in range(n):
        yield i  # pauses here, sends value, resumes on next()

gen = count_up(5)
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2
```

---

### `yield` vs `return`

| | `return` | `yield` |
|---|---|---|
| Returns | Once, then function ends | One value at a time |
| State | Lost after return | Preserved between yields |
| Memory | All at once | One item at a time |

---

### Why Use Generators? (Memory Efficiency)

```python
# This loads ALL 1 million numbers into memory
def get_numbers_list(n):
    return [i * i for i in range(n)]

# This generates one at a time — almost no memory used
def get_numbers_gen(n):
    for i in range(n):
        yield i * i

for num in get_numbers_gen(1000000):
    print(num)
```

---

### Generator Expression
Like list comprehension but lazy:

```python
# List comprehension — all in memory
squares = [x * x for x in range(1000000)]

# Generator expression — lazy
squares = (x * x for x in range(1000000))

print(next(squares))  # 0
print(next(squares))  # 1
```

---

### Real-World Use Case
Reading a huge file line by line without loading it all into memory:

```python
def read_large_file(filepath):
    with open(filepath) as f:
        for line in f:
            yield line.strip()

for line in read_large_file("huge_log.txt"):
    process(line)
```

---

### `StopIteration`
When a generator runs out of values, it raises `StopIteration`. A `for` loop handles this automatically.

```python
gen = count_up(2)
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # StopIteration
```

---

## Quick Reference — When to Use What

| Scenario | Use |
|---|---|
| Multiple API calls at once | `threading` or `asyncio` |
| CPU-heavy computation | `multiprocessing` |
| High-concurrency web server | `asyncio` (FastAPI) |
| Large data processing | Generator |
| Add behavior to functions | Decorator |
| Compare values | `==` |
| Check if same object / None check | `is` |
