# Table-of-Contents

<!-- toc -->

- [Python Threading Notes](#python-threading-notes)
  * [When to Use Threading](#when-to-use-threading)
  * [Manual Threading (The "Older" Way)](#manual-threading-the-older-way)
    + [1. Create and Start Threads](#1-create-and-start-threads)
    + [2. Wait for Threads to Complete](#2-wait-for-threads-to-complete)
    + [3. Manage Multiple Threads with a Loop](#3-manage-multiple-threads-with-a-loop)
    + [4. Passing Arguments to a Thread](#4-passing-arguments-to-a-thread)
  * [Modern Threading with `concurrent.futures`](#modern-threading-with-concurrentfutures)
    + [Using `executor.submit()`](#using-executorsubmit)
    + [Processing Results with `as_completed()`](#processing-results-with-as_completed)
    + [Using `executor.map()`](#using-executormap)
- [Python Threading Examples](#python-threading-examples)
  * [1. The Initial Synchronous Script](#1-the-initial-synchronous-script)
    + [Code](#code)
    + [Explanation](#explanation)
  * [2. Manual Threading (The "Older" Way)](#2-manual-threading-the-older-way)
    + [Creating, Starting, and Joining Threads](#creating-starting-and-joining-threads)
    + [Handling Multiple Threads with a Loop](#handling-multiple-threads-with-a-loop)
    + [Passing Arguments to Threads](#passing-arguments-to-threads)
  * [3. Modern Threading with `concurrent.futures`](#3-modern-threading-with-concurrentfutures)
    + [Using `executor.submit()` and Futures](#using-executorsubmit-and-futures)
    + [Processing Results with `as_completed`](#processing-results-with-as_completed)
    + [Using `executor.map()`](#using-executormap-1)
  * [4. Real-World Example: Downloading Images](#4-real-world-example-downloading-images)
    + [Synchronous Version](#synchronous-version)
    + [Threaded Version with `.map()`](#threaded-version-with-map)
  * [Takeaways](#takeaways)
- [Python Multiprocessing – Micro Notes](#python-multiprocessing-%E2%80%93-micro-notes)
  * [1. Why Use Multiprocessing?](#1-why-use-multiprocessing)
  * [2. Synchronous vs. Parallel Execution](#2-synchronous-vs-parallel-execution)
    + [Synchronous Example](#synchronous-example)
    + [Parallel Execution (Multiprocessing)](#parallel-execution-multiprocessing)
  * [3. Manual Multiprocessing (Older Way)](#3-manual-multiprocessing-older-way)
    + [Create, Start, Join Processes](#create-start-join-processes)
    + [Multiple Processes with a Loop](#multiple-processes-with-a-loop)
    + [Passing Arguments](#passing-arguments)
  * [4. Modern Multiprocessing with `concurrent.futures`](#4-modern-multiprocessing-with-concurrentfutures)
    + [Using `submit` + `as_completed`](#using-submit--as_completed)
    + [Using `executor.map`](#using-executormap)
  * [5. Real-World Example: Image Processing](#5-real-world-example-image-processing)
    + [Original (Synchronous)](#original-synchronous)
    + [Multiprocessing Refactor](#multiprocessing-refactor)
  * [6. Switching Between Threads and Processes](#6-switching-between-threads-and-processes)
  * [Quick Takeaways](#quick-takeaways)

<!-- tocstop -->

---
# Python Threading Notes

## When to Use Threading
- **Use for I/O-Bound Tasks**  
  Threading is best for **I/O-bound** operations (waiting on external resources).  
  Examples: downloading files, reading/writing to disk.

- **Avoid for CPU-Bound Tasks**  
  For heavy computation, threading adds overhead and slows down. Use **multiprocessing** instead.

- **Concurrency vs. Parallelism**  
  Threading provides **concurrency**, not true parallelism. It just switches tasks while waiting on I/O.

---

## Manual Threading (The "Older" Way)

### 1. Create and Start Threads
```python
import threading
import time

def do_something():
    print('Sleeping 1 second...')
    time.sleep(1)
    print('Done sleeping.')

# Create two thread objects
t1 = threading.Thread(target=do_something)
t2 = threading.Thread(target=do_something)

# Start the threads
t1.start()
t2.start()
````

### 2. Wait for Threads to Complete

```python
t1.join()
t2.join()
```

### 3. Manage Multiple Threads with a Loop

```python
threads = []
for _ in range(10):
    t = threading.Thread(target=do_something)
    t.start()
    threads.append(t)

for thread in threads:
    thread.join()
```

### 4. Passing Arguments to a Thread

```python
def do_something(seconds):
    print(f'Sleeping {seconds} second(s)...')
    time.sleep(seconds)
    print('Done sleeping.')

t = threading.Thread(target=do_something, args=[1.5])
```

---

## Modern Threading with `concurrent.futures`

### Using `executor.submit()`

* `.submit()` schedules a function and returns a **future object**.
* `.result()` waits for the task and gets the return value.

```python
import concurrent.futures

def do_something(seconds):
    print(f'Sleeping {seconds} second(s)...')
    time.sleep(seconds)
    return f'Done sleeping {seconds}'

with concurrent.futures.ThreadPoolExecutor() as executor:
    f1 = executor.submit(do_something, 1)
    print(f1.result())
```

---

### Processing Results with `as_completed()`

* Handles results **as soon as tasks finish**, not in submission order.

```python
secs = [1, 2, 3, 4, 5]
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = [executor.submit(do_something, sec) for sec in secs]

    for f in concurrent.futures.as_completed(results):
        print(f.result())
```

---

### Using `executor.map()`

* Applies a function to each item in iterable.
* Returns results in **submission order**, not finish order.

```python
secs = [5, 4, 3, 2, 1]
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = executor.map(do_something, secs)

    for result in results:
        print(result)
```

# Python Threading Examples

## 1. The Initial Synchronous Script

### Code
```python
import time

# Function that simulates a 1-second task
def do_something():
    print('Sleeping 1 second...')
    time.sleep(1)
    print('Done sleeping.')

# Start a timer
start = time.perf_counter()

# Run the function twice, one after the other
do_something()
do_something()

# Calculate and print the total time taken
finish = time.perf_counter()
print(f'Finished in {round(finish-start, 2)} second(s)')
````

### Explanation

* Runs **synchronously**: one call finishes before the next starts.
* Each call sleeps for 1s → total \~2s runtime.
* Demonstrates an **I/O-bound** task → good candidate for threading.

---

## 2. Manual Threading (The "Older" Way)

### Creating, Starting, and Joining Threads

```python
import threading
import time

def do_something():
    print('Sleeping 1 second...')
    time.sleep(1)
    print('Done sleeping.')

start = time.perf_counter()

t1 = threading.Thread(target=do_something)
t2 = threading.Thread(target=do_something)

t1.start()
t2.start()

t1.join()
t2.join()

finish = time.perf_counter()
print(f'Finished in {round(finish-start, 2)} second(s)')
```

**Key Points**

* `target=do_something` (no parentheses!) → pass function reference.
* `.start()` begins execution.
* `.join()` ensures main thread waits for worker threads.
* Both tasks run concurrently → \~1s total instead of 2s.

---

### Handling Multiple Threads with a Loop

```python
threads = []
for _ in range(10):
    t = threading.Thread(target=do_something)
    t.start()
    threads.append(t)

for thread in threads:
    thread.join()
```

**Key Points**

* Use a list to keep track of threads.
* Two loops:

  1. Create/start all threads.
  2. Join all threads (wait for completion).
* Avoid calling `.join()` in the first loop → that would make it synchronous again.

---

### Passing Arguments to Threads

```python
def do_something(seconds):
    print(f'Sleeping {seconds} second(s)...')
    time.sleep(seconds)
    print('Done sleeping.')

t = threading.Thread(target=do_something, args=[1.5])
t.start()
```

**Key Points**

* Modify function to accept parameters.
* Pass arguments as a list/tuple with `args`.

---

## 3. Modern Threading with `concurrent.futures`

### Using `executor.submit()` and Futures

```python
import concurrent.futures
import time

def do_something(seconds):
    print(f'Sleeping {seconds} second(s)...')
    time.sleep(seconds)
    return f'Done Sleeping...{seconds}'

with concurrent.futures.ThreadPoolExecutor() as executor:
    f1 = executor.submit(do_something, 1)
    print(f1.result())
```

**Key Points**

* `.submit()` schedules function and returns a **future object**.
* `.result()` waits and retrieves the function’s return value.

---

### Processing Results with `as_completed`

```python
secs = [1, 2, 3, 4, 5]
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = [executor.submit(do_something, sec) for sec in secs]

    for f in concurrent.futures.as_completed(results):
        print(f.result())
```

**Key Points**

* `as_completed` yields futures in order of **completion**, not submission.
* Shorter tasks finish and print earlier.

---

### Using `executor.map()`

```python
secs = [5, 4, 3, 2, 1]
with concurrent.futures.ThreadPoolExecutor() as executor:
    results = executor.map(do_something, secs)

    for result in results:
        print(result)
```

**Key Points**

* `.map()` works like Python’s built-in `map`.
* Results are returned in **submission order**, not completion order.
* Context manager automatically waits for all threads.

---

## 4. Real-World Example: Downloading Images

### Synchronous Version

```python
# img_urls is a list of image URLs...
for img_url in img_urls:
    img_bytes = requests.get(img_url).content
    img_name = img_url.split('/')[-1]
    # save image to disk...
```

**Observation**

* Downloads sequentially.
* I/O-bound → wastes time waiting for network.
* Took \~23s for 15 images.

---

### Threaded Version with `.map()`

```python
def download_image(img_url):
    img_bytes = requests.get(img_url).content
    img_name = img_url.split('/')[-1]
    # save image to disk...

with concurrent.futures.ThreadPoolExecutor() as executor:
    executor.map(download_image, img_urls)
```

**Observation**

* `download_image` runs concurrently for each URL.
* Finished in \~5s instead of 23s.

---

## Takeaways

* Use **threading for I/O-bound tasks**, not CPU-heavy work.
* `threading.Thread` gives manual control but is verbose.
* `concurrent.futures.ThreadPoolExecutor` is cleaner and more powerful.
* `.submit()` + futures → granular control.
* `as_completed` → process tasks as they finish.
* `.map()` → simple, ordered results.
* Perfect for network, file, and I/O tasks.
Perfect. Here’s your content neatly formatted into **Obsidian-ready micro notes** with clear headings, code blocks, and explanations, just like you asked:

# Python Multiprocessing – Micro Notes

## 1. Why Use Multiprocessing?
- **Goal**: Speed up programs by running tasks truly in parallel.  
- **Mechanism**: Spreads work across multiple CPU cores.  
- **Use Cases**:
  - **CPU-bound tasks** → heavy computation (e.g., image processing).  
  - **I/O-bound tasks** → file/network wait times.  
- Unlike threading (best for I/O), multiprocessing helps with both CPU-bound and I/O-bound tasks.

---

## 2. Synchronous vs. Parallel Execution

### Synchronous Example
```python
import time

def do_something():
    print('Sleeping 1 second...')
    time.sleep(1)
    print('Done sleeping.')

start = time.perf_counter()
do_something()
do_something()
finish = time.perf_counter()

print(f'Finished in {round(finish-start, 2)} second(s)')
# Output: ~2.0 seconds
````

**Observation**
Runs tasks one after another. Two calls → \~2 seconds.

### Parallel Execution (Multiprocessing)

Same two calls, but run on separate processes → \~1 second.

---

## 3. Manual Multiprocessing (Older Way)

### Create, Start, Join Processes

```python
import multiprocessing
import time

def do_something():
    print('Sleeping 1 second...')
    time.sleep(1)
    print('Done sleeping.')

p1 = multiprocessing.Process(target=do_something)
p2 = multiprocessing.Process(target=do_something)

p1.start()
p2.start()

p1.join()
p2.join()
```

**Key Points**

* `multiprocessing.Process(target=...)` → creates process.
* `.start()` → runs process.
* `.join()` → waits for process to finish.

---

### Multiple Processes with a Loop

```python
processes = []
for _ in range(10):
    p = multiprocessing.Process(target=do_something)
    p.start()
    processes.append(p)

for process in processes:
    process.join()
```

* Two loops:

  1. Start all processes.
  2. Join them all.
* Avoid `.join()` inside the start loop (would make it synchronous).

---

### Passing Arguments

```python
def do_something(seconds):
    print(f'Sleeping {seconds} second(s)...')
    time.sleep(seconds)
    print('Done sleeping.')

p = multiprocessing.Process(target=do_something, args=[1.5])
p.start()
```

* Use `args=[...]` (must be **pickleable**).

---

## 4. Modern Multiprocessing with `concurrent.futures`

### Using `submit` + `as_completed`

```python
import concurrent.futures
import time

def do_something(seconds):
    print(f'Sleeping {seconds} second(s)...')
    time.sleep(seconds)
    return f'Done Sleeping {seconds}'

secs = [1, 2, 3, 4, 5]

with concurrent.futures.ProcessPoolExecutor() as executor:
    results = [executor.submit(do_something, sec) for sec in secs]
    for f in concurrent.futures.as_completed(results):
        print(f.result())
```

* `.submit()` → schedules function, returns future.
* `.result()` → waits + returns value.
* `as_completed` → yields results as soon as finished.

---

### Using `executor.map`

```python
secs = [1, 2, 3, 4, 5]

with concurrent.futures.ProcessPoolExecutor() as executor:
    results = executor.map(do_something, secs)
    for result in results:
        print(result)
```

* Applies function to iterable.
* Returns results in **submission order**, not completion order.
* Context manager auto-joins processes.

---

## 5. Real-World Example: Image Processing

### Original (Synchronous)

```python
for img_name in img_names:
    # ... process one image ...
```

Took **22 seconds** for 15 images.

### Multiprocessing Refactor

```python
def process_image(img_name):
    # ... process one image ...
    print(f'{img_name} processed...')

with concurrent.futures.ProcessPoolExecutor() as executor:
    executor.map(process_image, img_names)
```

* Finished in **7 seconds** instead of 22.

---

## 6. Switching Between Threads and Processes

* Just change `ProcessPoolExecutor` ↔ `ThreadPoolExecutor`.
* **Rule of Thumb**:

  * CPU-bound → use processes.
  * I/O-bound → use threads.
* **Benchmark Always**: In tests, threading sometimes outperforms processes if tasks involve more I/O than CPU.

---

## Quick Takeaways

* **Processes** → true parallelism (good for CPU-heavy work).
* **Threads** → concurrency (good for I/O waits).
* Manual API (`multiprocessing.Process`) works but verbose.
* Modern API (`concurrent.futures`) is cleaner and more flexible.
* `.map()` = simple ordered results, `.as_completed()` = results as they finish.
* Switching between threads and processes is as simple as swapping the executor class.

