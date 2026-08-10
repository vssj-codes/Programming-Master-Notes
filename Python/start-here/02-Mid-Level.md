# Table-of-Contents

<!-- toc -->

- [02-Mid-Level](#02-mid-level)
  * [Section 1 — Decorators (Depth)](#section-1--decorators-depth)
  * [Section 2 — Closures & Scope (Depth)](#section-2--closures--scope-depth)
  * [Section 3 — Generators (Depth)](#section-3--generators-depth)
  * [Section 4 — OOP (Depth)](#section-4--oop-depth)
  * [Section 5 — Functional Programming Patterns](#section-5--functional-programming-patterns)
  * [Section 6 — Itertools (Depth)](#section-6--itertools-depth)
  * [Section 7 — Data Structure Internals](#section-7--data-structure-internals)
  * [Section 8 — Comprehensions & Expressions (Depth)](#section-8--comprehensions--expressions-depth)
  * [Section 9 — Error Handling Patterns](#section-9--error-handling-patterns)
  * [Section 10 — String Internals & Patterns](#section-10--string-internals--patterns)
  * [Section 11 — Memory & Performance Basics](#section-11--memory--performance-basics)
  * [Section 12 — Pythonic Patterns](#section-12--pythonic-patterns)

<!-- tocstop -->

---

# 02-Mid-Level
## Section 1 — Decorators (Depth)

  

**Q1: Implement a basic decorator from scratch without using any library.**

A:

```python

def my_decorator(func):
	def wrapper(*args, **kwargs):
		print("before")
		result = func(*args, **kwargs)
		print("after")
		return result
	return wrapper

  
@my_decorator
def greet(name):
	print(f"Hello {name}")

```

`@my_decorator` is syntactic sugar for `greet = my_decorator(greet)`.

  

**Q2: Why do we need `functools.wraps`? What breaks without it?**

A: Without `functools.wraps`, the wrapper function replaces the original's metadata:

- `func.__name__` → `"wrapper"` instead of the original name

- `func.__doc__` → `None`

- `func.__module__`, `func.__annotations__` are also lost

- Breaks introspection, logging, pytest output, and `help()`

```python

import functools
def decorator(func):
    @functools.wraps(func)  # copies __name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

```

  

**Q3: What is `__wrapped__`? What sets it?**

A: `functools.wraps` sets `__wrapped__` on the wrapper to point to the original function.

- Allows introspection tools to unwrap decorators and inspect the real function

- `func.__wrapped__` → original unwrapped function

- `inspect.unwrap(func)` uses `__wrapped__` to fully unwrap a chain of decorators

  

**Q4: Implement a decorator that accepts arguments — explain the 3-layer pattern.**

A: Requires 3 layers: outer function takes config → returns decorator → returns wrapper.

```python

def repeat(n):              # layer 1: takes config
    def decorator(func):    # layer 2: takes function
        @functools.wraps(func)
        def wrapper(*args, **kwargs):  # layer 3: wraps call
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(): print("hi")


```

  

**Q5: What is the execution order when you stack multiple decorators?**

A: Applied bottom-up, executed top-down.

```python

@A

@B

@C

def f(): ...

# equivalent to: f = A(B(C(f)))

# call order: A's wrapper → B's wrapper → C's wrapper → f

```

  

**Q6: Implement a `timer` decorator.**

A:

```python

import time, functools

  

def timer(func):

@functools.wraps(func)

def wrapper(*args, **kwargs):

start = time.perf_counter()

result = func(*args, **kwargs)

print(f"{func.__name__} took {time.perf_counter() - start:.4f}s")

return result

return wrapper

```

  

**Q7: Implement a `retry(max_attempts, exceptions)` decorator.**

A:

```python

def retry(max_attempts=3, exceptions=(Exception,)):

def decorator(func):

@functools.wraps(func)

def wrapper(*args, **kwargs):

last_exc = None

for attempt in range(1, max_attempts + 1):

try:

return func(*args, **kwargs)

except exceptions as e:

last_exc = e

raise last_exc

return wrapper

return decorator

```

  

**Q8: Implement a `memoize` decorator without using `functools.lru_cache`.**

A:

```python

def memoize(func):

cache = {}

@functools.wraps(func)

def wrapper(*args):

if args not in cache:

cache[args] = func(*args)

return cache[args]

wrapper.cache = cache # expose for inspection

return wrapper

```

Arguments must be hashable. No cache eviction — grows unbounded.

  

**Q9: What is a class-based decorator? When would you prefer it over a function-based one?**

A: A class whose `__init__` takes the function and `__call__` implements the wrapper.

```python

class CountCalls:

def __init__(self, func):

functools.update_wrapper(self, func)

self.func = func

self.count = 0

def __call__(self, *args, **kwargs):

self.count += 1

return self.func(*args, **kwargs)

```

Prefer class-based when you need to maintain state (call count, cache) or expose additional methods/attributes on the decorator.

  

**Q10: Can you decorate a class? What would that do?**

A: Yes. A class decorator receives the class and returns a modified class (or a replacement).

```python

def singleton(cls):

instances = {}

def get_instance(*args, **kwargs):

if cls not in instances:

instances[cls] = cls(*args, **kwargs)

return instances[cls]

return get_instance

```

Common uses: singleton, registering classes, adding methods.

  

**Q11: Can you decorate a method inside a class? Are there any gotchas?**

A: Yes. Gotchas:

- Decorating with a plain function-based decorator works fine

- Class-based decorators break `self` — the instance is not passed correctly unless the decorator handles the descriptor protocol (`__get__`)

- `@staticmethod` and `@classmethod` must be the outermost decorator

  

---

  

## Section 2 — Closures & Scope (Depth)

  

**Q12: Explain the LEGB rule with an example.**

A: Python resolves names in this order: Local → Enclosing → Global → Built-in.

```python

x = "global"

def outer():

x = "enclosing"

def inner():

# x = "local" # uncomment to use local

print(x) # finds "enclosing" in E

inner()

outer()

```

  

**Q13: What is a free variable? How do you inspect it?**

A: A variable used in a function but defined in an enclosing scope — not local, not global.

```python

def make_adder(n):

def add(x):

return x + n # n is a free variable

return add

  

add5 = make_adder(5)

print(add5.__code__.co_freevars) # ('n',)

print(add5.__closure__[0].cell_contents) # 5

```

  

**Q14: What is the late binding gotcha in closures? Show the bug and two ways to fix it.**

A: Closures capture variables by reference, not by value. In a loop, all closures share the same variable.

```python

# Bug

funcs = [lambda x: x * i for i in range(5)]

print([f(2) for f in funcs]) # [8, 8, 8, 8, 8] — all use i=4

  

# Fix 1: default argument captures value at definition time

funcs = [lambda x, i=i: x * i for i in range(5)]

  

# Fix 2: closure factory

def make_fn(i):

return lambda x: x * i

funcs = [make_fn(i) for i in range(5)]

```

  

**Q15: What does `nonlocal` do? Why does it exist?**

A: Allows an inner function to rebind (not just read) a variable from an enclosing scope.

- Without `nonlocal`, assigning to the variable creates a new local variable instead

- `global` works for module-level. `nonlocal` works for enclosing function scope.

```python

def counter():

count = 0

def increment():

nonlocal count

count += 1

return count

return increment

```

  

**Q16: What is the difference between `nonlocal` and `global`?**

A:

- `global` — refers to the module-level variable

- `nonlocal` — refers to the nearest enclosing function scope (not global)

- `nonlocal` cannot refer to a global variable; it must find an enclosing function variable

  

**Q17: Can a closure modify an enclosing variable without `nonlocal`? What happens if you try?**

A: You can READ an enclosing variable without `nonlocal`. But assigning to it without `nonlocal` creates a new local variable, shadowing the enclosing one.

```python

def outer():

x = 10

def inner():

x += 1 # UnboundLocalError — Python sees assignment, treats x as local

inner()

```

Add `nonlocal x` to fix it.

  

**Q18: How does a closure differ from a class with `__call__`?**

A:

- Both encapsulate state + behavior

- Closure: lighter, less boilerplate, no explicit state attribute names

- Class: more explicit state, supports multiple methods, easier to inspect/test

- Use closure for simple single-purpose callables; class when you need more structure

  

**Q19: Implement a counter using a closure.**

A:

```python

def make_counter(start=0):

count = start

def increment():

nonlocal count

count += 1

return count

def reset():

nonlocal count

count = start

return increment, reset

```

  

**Q20: Implement a memoizer using a closure.**

A:

```python

def memoize(func):

cache = {} # lives in enclosing scope, shared across all calls

def wrapper(*args):

if args not in cache:

cache[args] = func(*args)

return cache[args]

return wrapper

```

  

---

  

## Section 3 — Generators (Depth)

  

**Q21: What is the difference between `return` and `yield` in a function?**

A:

- `return` — ends the function, returns a value once

- `yield` — suspends the function, returns a value, resumes on next `next()` call

- A function with `yield` becomes a generator function — calling it returns a generator object, not a value

  

**Q22: What does calling a generator function return? What does `next()` do to it?**

A: Calling a generator function returns a generator object — no code runs yet.

- `next(gen)` — resumes execution until the next `yield`, returns the yielded value

- When the function returns (or falls off the end), `StopIteration` is raised

  

**Q23: What are the four states a generator can be in?**

A:

- `GEN_CREATED` — created, not yet started

- `GEN_RUNNING` — currently executing (inside a `next()` call)

- `GEN_SUSPENDED` — paused at a `yield`

- `GEN_CLOSED` — finished or explicitly closed

Inspect with `inspect.getgeneratorstate(gen)`.

  

**Q24: What does `generator.send(value)` do? What must you do before calling `send()`?**

A: `send(value)` resumes the generator AND sends a value in — it becomes the result of the `yield` expression.

- Must call `next(gen)` or `gen.send(None)` first to advance to the first `yield`

- Sending to an unstarted generator raises `TypeError`

```python

def accumulator():

total = 0

while True:

value = yield total

total += value

gen = accumulator()

next(gen) # prime

gen.send(10) # total = 10

gen.send(20) # total = 30

```

  

**Q25: What does `generator.throw(exc)` do?**

A: Raises an exception at the point where the generator is suspended (at the `yield`).

- The generator can catch it with `try/except` and continue yielding

- If uncaught, the exception propagates to the caller

  

**Q26: What does `generator.close()` do? What exception does it throw inside the generator?**

A: Throws `GeneratorExit` at the suspension point.

- If the generator catches `GeneratorExit` and yields again → `RuntimeError`

- If it returns or doesn't catch it → generator closes cleanly

- Called automatically when a generator is garbage collected

  

**Q27: What is `yield from`? How is it different from `for x in iterable: yield x`?**

A: `yield from iterable` delegates to a sub-generator completely.

- Difference: also passes `send()` values and `throw()` calls through to the sub-generator

- `for x in it: yield x` only forwards values, not `send`/`throw`/`close`

- Also captures the `return` value of the sub-generator via `StopIteration.value`

  

**Q28: What is a generator expression? How does memory usage compare to a list comprehension?**

A: `(x for x in iterable)` — lazy, produces one item at a time.

- List comprehension: all items in memory at once — O(n) memory

- Generator expression: O(1) memory — only current item exists at any time

- Use when you only need to iterate once and don't need indexing

  

**Q29: Build a lazy pipeline using generators.**

A:

```python

def read_lines(data):

yield from data

  

def strip_lines(lines):

for line in lines:

yield line.strip()

  

def filter_lines(lines, keyword):

for line in lines:

if keyword not in line:

yield line

  

data = [" hello ", " skip this ", " world "]

pipeline = filter_lines(strip_lines(read_lines(data)), "skip")

print(list(pipeline)) # ["hello", "world"]

```

Each stage is lazy — data flows one item at a time.

  

**Q30: When would you choose a generator over a list? Give 3 real use cases.**

A:

1. Reading large files line by line — avoid loading entire file into memory

2. Infinite sequences — `fibonacci()`, `count()` — can't store in a list

3. Data pipelines — stream-process data through transformation stages without intermediate lists

  

---

  

## Section 4 — OOP (Depth)

  

**Q31: What is MRO? How does Python compute it (C3 linearization)?**

A: Method Resolution Order — the order Python searches classes for an attribute.

- Computed via C3 linearization algorithm

- Rules: subclass before parent, left before right, preserve local order

- `ClassName.__mro__` shows the full order

```python

class D(B, C): ...

# MRO: D → B → C → A → object

```

  

**Q32: Given a diamond inheritance, what is the MRO? Trace through it.**

A:

```python

class A: pass

class B(A): pass

class C(A): pass

class D(B, C): pass

# D.__mro__ = (D, B, C, A, object)

```

D first, then left branch B, then right branch C, then shared base A, then object.

  

**Q33: What does `super()` actually return? Is it just "call the parent class"?**

A: `super()` returns a proxy object that delegates method calls to the next class in the MRO — not necessarily the direct parent.

- In `B.method`, `super()` finds the next class after `B` in the MRO of the actual instance's class

- This enables cooperative multiple inheritance

  

**Q34: How does `super()` work in multiple inheritance? What is cooperative inheritance?**

A: Each class calls `super()` which passes control to the next in the MRO chain. All classes cooperate to ensure every `__init__` is called once.

```python

class A:

def __init__(self): super().__init__(); print("A")

class B(A):

def __init__(self): super().__init__(); print("B")

class C(A):

def __init__(self): super().__init__(); print("C")

class D(B, C):

def __init__(self): super().__init__(); print("D")

# D() prints: A, C, B, D — MRO order

```

  

**Q35: What is the difference between `@classmethod`, `@staticmethod`, and `@property`?**

A:

- `@classmethod` — first arg is the class (`cls`). Used for alternative constructors, factory methods.

- `@staticmethod` — no implicit first arg. Just a regular function in the class namespace.

- `@property` — turns a method into a computed attribute. Accessed without `()`.

  

**Q36: When would you use `@classmethod` as an alternative constructor? Give an example.**

A:

```python

class Date:

def __init__(self, year, month, day):

self.year, self.month, self.day = year, month, day

  

@classmethod

def from_string(cls, date_str):

y, m, d = map(int, date_str.split("-"))

return cls(y, m, d) # uses cls, not Date — works with subclasses too

  

d = Date.from_string("2026-08-10")

```

  

**Q37: What is a descriptor? What methods does the descriptor protocol define?**

A: A descriptor is an object that defines how attribute access is handled via:

- `__get__(self, obj, objtype)` — called on attribute access

- `__set__(self, obj, value)` — called on attribute assignment

- `__delete__(self, obj)` — called on `del`

A data descriptor defines `__set__` or `__delete__`. A non-data descriptor only defines `__get__`.

  

**Q38: How does `@property` work under the hood — is it a descriptor?**

A: Yes. `property` is a built-in data descriptor class.

- `@property` creates a `property` object with `__get__`, `__set__`, `__delete__`

- `obj.x` → calls `property.__get__`

- `obj.x = val` → calls `property.__set__` (if setter defined, else `AttributeError`)

  

**Q39: What is the difference between `__new__` and `__init__`?**

A:

- `__new__(cls)` — creates and returns the new instance. Called first.

- `__init__(self)` — initializes the already-created instance. Called second.

- Override `__new__` for: singletons, immutable types (str, tuple subclasses), controlling instance creation

  

**Q40: What is `__slots__`? What does it do and what are the tradeoffs?**

A: Replaces the per-instance `__dict__` with a fixed set of slots.

```python

class Point:

__slots__ = ("x", "y")

```

- Pro: less memory per instance, faster attribute access

- Con: no `__dict__`, can't add arbitrary attributes, harder to pickle, issues with multiple inheritance

  

**Q41: What are dataclasses? What do they auto-generate?**

A: `@dataclass` auto-generates boilerplate based on type-annotated fields:

- `__init__` — with all fields as parameters

- `__repr__` — readable string representation

- `__eq__` — value-based equality

- Optionally: `__lt__`, `__hash__`, `__post_init__`

  

**Q42: What is the difference between `dataclass(frozen=True)` and a regular dataclass?**

A:

- Regular: mutable, no `__hash__` (set to `None`)

- `frozen=True`: immutable (raises `FrozenInstanceError` on assignment), generates `__hash__` — can be used in sets/dicts

  

**Q43: Why must mutable default fields in dataclasses use `field(default_factory=...)`?**

A: Same reason as the mutable default argument bug — a single mutable object would be shared across all instances.

```python

# Wrong:

@dataclass

class Foo:

items: list = [] # TypeError — caught by dataclass

  

# Correct:

@dataclass

class Foo:

items: list = field(default_factory=list)

```

  

**Q44: What is `__call__`? Make a class instance callable.**

A: `__call__` lets an instance be called like a function.

```python

class Multiplier:

def __init__(self, factor):

self.factor = factor

def __call__(self, x):

return x * self.factor

  

double = Multiplier(2)

double(5) # 10

```

  

**Q45: What is `__getattr__` vs `__getattribute__`?**

A:

- `__getattribute__` — called on every attribute access. Override with extreme care.

- `__getattr__` — called only when normal lookup fails (attribute not found). Safer to override.

- Common use: proxy objects, lazy attributes, dynamic attribute generation

  

**Q46: What is `__repr__` vs `__str__` vs `__format__`?**

A:

- `__repr__` — unambiguous, for developers. `repr(obj)`. Should ideally recreate the object.

- `__str__` — readable, for users. `str(obj)`, `print(obj)`. Falls back to `__repr__`.

- `__format__` — called by f-strings and `format()`. Allows custom format specs: `f"{obj:.2f}"`

  

**Q47: What does `@functools.total_ordering` do?**

A: If you define `__eq__` and ONE of `__lt__`, `__le__`, `__gt__`, `__ge__`, it auto-generates the rest.

- Avoids writing all 6 comparison methods manually

- Small performance cost vs defining all manually

  

**Q48: What is an abstract base class? How do you define one?**

A: A class that cannot be instantiated and enforces that subclasses implement specific methods.

```python

from abc import ABC, abstractmethod

  

class Shape(ABC):

@abstractmethod

def area(self) -> float: ...

  

# Shape() → TypeError

# Subclass must implement area() or it also can't be instantiated

```

  

**Q49: What is the difference between nominal and structural subtyping?**

A:

- Nominal — a class must explicitly inherit from or register with a base to be considered a subtype. Python's default class system.

- Structural (duck typing) — a class satisfies a type if it has the required methods, regardless of inheritance. `Protocol` in `typing` enables this.

```python

class Drawable(Protocol):

def draw(self) -> str: ...

# Any class with draw() satisfies Drawable — no inheritance needed

```

  

---

  

## Section 5 — Functional Programming Patterns

  

**Q50: What does `functools.partial` do? Give a real use case.**

A: Creates a new function with some arguments pre-filled.

```python

from functools import partial

  

def power(base, exp): return base ** exp

square = partial(power, exp=2)

square(5) # 25

  

# Real use case: adapting a function for map()

add = lambda x, y: x + y

add5 = partial(add, y=5)

list(map(add5, [1, 2, 3])) # [6, 7, 8]

```

  

**Q51: What does `functools.reduce` do? Give an example.**

A: Applies a function cumulatively to items of an iterable, reducing it to a single value.

```python

from functools import reduce

reduce(lambda acc, x: acc * x, [1, 2, 3, 4, 5]) # 120

```

  

**Q52: What is the difference between `map()`, `filter()`, and a list comprehension? Which is more Pythonic?**

A:

- `map(func, it)` — applies func to each item, returns iterator

- `filter(pred, it)` — keeps items where pred is True, returns iterator

- List comprehension — does both in one readable expression

- List comprehensions are more Pythonic and readable for most cases. Use `map`/`filter` only when composing with other iterators.

  

**Q53: What is a higher-order function?**

A: A function that takes a function as argument OR returns a function.

- `map`, `filter`, `sorted(key=...)`, decorators are all higher-order functions

  

**Q54: What is function composition? Implement it in Python.**

A: Combining functions so the output of one is the input of the next.

```python

def compose(*funcs):

def composed(x):

for f in reversed(funcs):

x = f(x)

return x

return composed

  

double = lambda x: x * 2

inc = lambda x: x + 1

double_then_inc = compose(inc, double)

double_then_inc(3) # inc(double(3)) = 7

```

  

**Q55: What is `functools.lru_cache`? How does it work? What are its constraints?**

A: Memoizes function results using a Least Recently Used cache.

- Stores up to `maxsize` results. When full, evicts the least recently used.

- Arguments must be hashable (used as cache keys)

- Not thread-safe by default for the cached values (though cache operations are)

- `func.cache_info()` shows hits, misses, size. `func.cache_clear()` resets it.

  

**Q56: What is the difference between `functools.lru_cache` and `functools.cache`?**

A:

- `lru_cache(maxsize=N)` — bounded cache with LRU eviction

- `cache` (Python 3.9+) — equivalent to `lru_cache(maxsize=None)` — unbounded, no eviction, slightly faster

- Use `cache` when you want to memoize everything and memory is not a concern

  

**Q57: What does `functools.wraps` copy from the original function?**

A: Copies: `__module__`, `__name__`, `__qualname__`, `__annotations__`, `__doc__`, `__dict__`, `__wrapped__`.

- `__wrapped__` points to the original function for unwrapping

  

---

  

## Section 6 — Itertools (Depth)

  

**Q58: What does `itertools.chain()` do? How is it different from concatenating lists?**

A: Chains multiple iterables into one without creating a new list.

- `chain([1,2], [3,4])` → `1, 2, 3, 4` lazily

- List concat `[1,2] + [3,4]` creates a new list in memory — O(n)

- `chain` is O(1) memory, works on any iterable including generators

  

**Q59: What does `itertools.islice()` do? Why is it useful with infinite iterators?**

A: Takes a slice of an iterable without consuming it all.

- `islice(iterable, stop)` or `islice(iterable, start, stop, step)`

- Essential for infinite iterators — `islice(count(0), 5)` → first 5 integers

  

**Q60: What does `itertools.groupby()` do? What is the key requirement on the input?**

A: Groups consecutive elements by a key function. Returns `(key, group_iterator)` pairs.

- Input MUST be sorted by the key first — it only groups consecutive equal keys

```python

from itertools import groupby

data = sorted(["apple","avocado","banana"], key=lambda x: x[0])

for k, g in groupby(data, key=lambda x: x[0]):

print(k, list(g))

```

  

**Q61: What does `itertools.product()` do?**

A: Cartesian product of iterables — equivalent to nested for loops.

```python

list(product([1,2], ["a","b"]))

# [(1,'a'), (1,'b'), (2,'a'), (2,'b')]

```

  

**Q62: What is the difference between `itertools.zip_longest()` and `zip()`?**

A:

- `zip()` — stops at the shortest iterable

- `zip_longest()` — stops at the longest, fills missing values with `fillvalue` (default `None`)

  

**Q63: What does `itertools.takewhile()` vs `itertools.dropwhile()` do?**

A:

- `takewhile(pred, it)` — yields items while pred is True, stops at first False

- `dropwhile(pred, it)` — skips items while pred is True, yields rest once pred is False

  

**Q64: What does `itertools.accumulate()` do?**

A: Returns running accumulated results of a binary function (default: addition).

```python

list(accumulate([1, 2, 3, 4])) # [1, 3, 6, 10]

list(accumulate([1, 2, 3, 4], max)) # [1, 2, 3, 4]

list(accumulate([2, 3, 4], lambda a,b: a*b)) # [2, 6, 24]

```

  

---

  

## Section 7 — Data Structure Internals

  

**Q65: How does Python's `dict` work internally?**

A: Hash table. Each key is hashed to determine its slot index.

- `hash(key)` → index into an array of slots

- Stores `(hash, key, value)` per slot

- Load factor ~2/3: when 2/3 full, the table resizes (doubles)

- Since Python 3.7, maintains insertion order as an implementation detail (guaranteed in 3.7+)

  

**Q66: What is a hash collision? How does Python handle it?**

A: When two different keys produce the same hash value.

- Python uses open addressing with probing — finds the next available slot

- Average case O(1) lookup; worst case O(n) with many collisions (rare with good hash functions)

  

**Q67: Why must dict keys be hashable? What makes an object hashable?**

A: Dict uses `hash(key)` to find the slot. Unhashable objects can't be used.

- An object is hashable if it implements `__hash__` and `__eq__`

- Immutable built-ins are hashable: `int`, `str`, `tuple` (if elements hashable), `frozenset`

- Mutable types are not: `list`, `dict`, `set`

- Defining `__eq__` without `__hash__` sets `__hash__ = None` → unhashable

  

**Q68: How does Python's `set` differ from a `dict` internally?**

A: Same hash table mechanism, but stores only keys — no values.

- O(1) average add, remove, lookup

- No ordering guarantee (though CPython 3.7+ dicts are ordered, sets are not)

  

**Q69: What is the time complexity of list operations?**

A:

- Index lookup `lst[i]` — O(1)

- Append — O(1) amortized

- Pop from end — O(1)

- Insert at position / pop from front — O(n)

- Search `in` — O(n)

- `len()` — O(1)

  

**Q70: Why is `collections.deque` better than a list for a queue?**

A: `list.insert(0, x)` and `list.pop(0)` are O(n) — all elements shift.

- `deque.appendleft()` and `deque.popleft()` are O(1)

- Implemented as a doubly-linked list of fixed-size blocks

  

**Q71: What is `collections.defaultdict`? How does it differ from `dict.setdefault()`?**

A:

- `defaultdict(factory)` — auto-creates a default value on missing key access using the factory callable

- `dict.setdefault(key, default)` — inserts and returns default only if key missing, but requires calling it explicitly every time

- `defaultdict` is cleaner for repeated patterns like grouping

  

**Q72: What is `collections.Counter`? What are its most useful methods?**

A: A dict subclass for counting hashable objects.

- `most_common(n)` — top n elements by count

- `+`, `-`, `&`, `|` — arithmetic and set operations on counts

- `elements()` — iterator over elements repeated by count

- `update()` — add counts (not replace like dict.update)

  

**Q73: What is `collections.namedtuple`? When would you use it over a dataclass?**

A: Creates a tuple subclass with named fields.

- Lightweight, immutable, supports tuple unpacking, uses less memory than a class

- Use over dataclass when: you need tuple compatibility, immutability, very low memory overhead, or CSV/row unpacking

- Use dataclass when: you need mutability, methods, default values, or type annotations with IDE support

  

**Q74: What is `collections.OrderedDict` still useful for in Python 3.7+?**

A: Regular dicts maintain insertion order since 3.7, but `OrderedDict` still has:

- `move_to_end(key, last=True)` — move a key to front or end

- Order-sensitive equality — two `OrderedDict`s with same items in different order are not equal (plain dicts consider them equal)

- LRU cache implementations

  

---

  

## Section 8 — Comprehensions & Expressions (Depth)

  

**Q75: What happens under the hood when Python executes a list comprehension?**

A: Python compiles it to a nested function call internally.

- Creates a new scope (since Python 3), executes the iteration, collects results into a list

- Roughly equivalent to `list(map(...))` but often faster due to bytecode optimization

  

**Q76: Do list comprehensions have their own scope? What about generator expressions?**

A: Yes — since Python 3, both list comprehensions and generator expressions have their own scope.

- The iteration variable does NOT leak into the enclosing scope

- In Python 2, list comprehension variables leaked — this was a bug fixed in Python 3

  

**Q77: What is the walrus operator `:=`? Show a real use case inside a comprehension.**

A: Assigns a value and returns it in a single expression (Python 3.8+).

```python

# Without walrus — calls process() twice

results = [process(x) for x in data if process(x) > 0]

  

# With walrus — calls process() once

results = [y for x in data if (y := process(x)) > 0]

```

  

**Q78: Write a flat list from a nested list using a comprehension.**

A:

```python

matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

flat = [x for row in matrix for x in row]

# [1, 2, 3, 4, 5, 6, 7, 8, 9]

```

Read as: "for each row in matrix, for each x in row, yield x".

  

**Q79: What is the readability rule for when NOT to use a comprehension?**

A: If it doesn't read naturally in one line, use a regular loop.

- More than one condition + transformation → loop

- Nested more than 2 levels → loop

- Side effects inside → loop (comprehensions should be pure)

  

---

  

## Section 9 — Error Handling Patterns

  

**Q80: What is the difference between `raise X` and `raise X from Y`?**

A:

- `raise X` — raises X. If inside an except block, Python auto-sets `__context__` to the current exception (implicit chaining).

- `raise X from Y` — explicitly chains: sets `__cause__ = Y` and `__suppress_context__ = True`. Traceback shows "The above exception was the direct cause of..."

  

**Q81: What does `raise X from None` do?**

A: Suppresses exception chaining entirely. No previous exception is shown in the traceback.

```python

try:

int("abc")

except ValueError:

raise RuntimeError("conversion failed") from None

# Only RuntimeError is shown — ValueError is hidden

```

  

**Q82: What is `__cause__` vs `__context__` on an exception?**

A:

- `__context__` — set automatically when an exception is raised inside an except block (implicit chain)

- `__cause__` — set explicitly via `raise X from Y` (explicit chain)

- `__suppress_context__` — set to True by `from Y` to hide `__context__`

  

**Q83: When does the `else` clause of a `try` block run?**

A: Only when the `try` block completed without raising any exception.

- Useful for code that should only run on success but shouldn't be inside `try` (to avoid catching exceptions it raises)

  

**Q84: When does `finally` NOT run?**

A: Almost always runs, but exceptions:

- `os._exit()` — terminates the process immediately

- SIGKILL signal

- Power loss / OS crash

- `finally` does run even with `sys.exit()`, `return`, `break`, `continue`

  

**Q85: How do you design a custom exception hierarchy?**

A: Create a base exception for your domain, then specific subclasses.

```python

class AppError(Exception):

def __init__(self, message, code="APP_ERROR"):

super().__init__(message)

self.code = code

  

class ValidationError(AppError):

def __init__(self, field, reason):

super().__init__(f"{field}: {reason}", "VALIDATION_ERROR")

  

class NotFoundError(AppError): ...

```

Callers can catch `AppError` for all domain errors or specific subclasses.

  

**Q86: What is the difference between `Exception` and `BaseException`? Why does it matter?**

A:

- `BaseException` — root of all exceptions including `SystemExit`, `KeyboardInterrupt`, `GeneratorExit`

- `Exception` — base for all "normal" errors, excludes the above

- `except Exception` — safe, won't accidentally catch Ctrl+C or `sys.exit()`

- `except BaseException` — catches everything, almost always a mistake

  

**Q87: What is the correct way to log an exception with full traceback?**

A:

```python

import logging

logger = logging.getLogger(__name__)

  

try:

risky()

except Exception:

logger.exception("Something failed") # logs message + full traceback automatically

# OR:

logger.error("Failed", exc_info=True)

```

  

**Q88: When should you use `assert`? When should you NOT?**

A:

- Use: internal invariants, developer assumptions, test code

- Do NOT use: input validation, security checks, production error handling

- Reason: `assert` is disabled when Python runs with `-O` (optimize) flag — all asserts become no-ops

  

---

  

## Section 10 — String Internals & Patterns

  

**Q89: What is string interning? Which strings does Python intern automatically?**

A: Interning stores only one copy of a string and reuses it for all identical strings.

- Auto-interned: string literals that look like identifiers (no spaces, no special chars)

- `"hello"` is usually interned. `"hello world"` is not guaranteed.

- Force intern: `sys.intern("my string")`

- Interned strings allow `is` comparison instead of `==` — faster

  

**Q90: Why is string concatenation in a loop inefficient? What is the correct approach?**

A: Strings are immutable — each `+=` creates a new string object and copies all previous content. O(n²) total.

```python

# Bad — O(n²)

result = ""

for s in strings:

result += s

  

# Good — O(n)

result = "".join(strings)

```

  

**Q91: What is the difference between `str.find()` and `str.index()`?**

A:

- `find()` — returns -1 if substring not found

- `index()` — raises `ValueError` if substring not found

- Both return the index of the first occurrence if found

  

**Q92: What does `str.strip()` vs `str.lstrip()` vs `str.rstrip()` do?**

A:

- `strip(chars)` — removes leading and trailing characters (default: whitespace)

- `lstrip(chars)` — removes only from the left/start

- `rstrip(chars)` — removes only from the right/end

- `chars` is a set of characters to remove, not a prefix/suffix

  

**Q93: What is `str.encode()` and why does encoding matter?**

A: Converts a `str` to `bytes` using the specified encoding (default UTF-8).

- Strings are Unicode in Python 3. Bytes are raw binary data.

- Encoding matters when writing to files, sending over network, or interfacing with C libraries

- `"hello".encode("utf-8")` → `b"hello"`

- `b"hello".decode("utf-8")` → `"hello"`

  

---

  

## Section 11 — Memory & Performance Basics

  

**Q94: What is reference counting in CPython?**

A: Every object has a reference count. When it reaches 0, the object is immediately deallocated.

- `sys.getrefcount(obj)` returns count + 1 (the argument itself adds a reference)

- Fast and deterministic — objects are freed as soon as they're no longer referenced

- Limitation: cannot handle reference cycles

  

**Q95: What is the garbage collector in Python responsible for that reference counting cannot handle?**

A: Detecting and collecting reference cycles.

- Example: `a.ref = b; b.ref = a` — both have count > 0 but are unreachable

- The cyclic GC periodically scans for isolated reference cycles and frees them

- `gc` module: `gc.collect()`, `gc.disable()`, `gc.get_threshold()`

  

**Q96: What does `sys.getsizeof()` measure?**

A: The memory size of an object itself in bytes — NOT including referenced objects.

- `sys.getsizeof([1,2,3])` → size of the list container, not the integers inside

- For total deep size, you need a recursive traversal

  

**Q97: What is the difference between a generator and a list in terms of memory?**

A:

- List: all elements stored in memory at once — O(n) memory

- Generator: only the current state stored — O(1) memory

- A generator of 1 million items uses the same memory as a generator of 10 items

  

**Q98: What does `__slots__` do to memory usage?**

A: Eliminates the per-instance `__dict__` (typically 200-300 bytes overhead).

- Each instance only stores the slot values, not a hash table

- Significant savings when creating thousands/millions of instances

- `__dict__` overhead: ~200-400 bytes per instance without slots

  

---

  

## Section 12 — Pythonic Patterns

  

**Q99: What is the Pythonic way to check if a list is empty?**

A: `if not my_list:` — uses truthiness. Empty list is falsy.

- Not: `if len(my_list) == 0:` — verbose

- Not: `if my_list == []:` — slow, creates a new list

  

**Q100: What is the Pythonic way to iterate with an index?**

A: `for i, val in enumerate(iterable):`

- Not: `for i in range(len(lst)): val = lst[i]`

  

**Q101: What is the Pythonic way to iterate two lists together?**

A: `for a, b in zip(list1, list2):`

- Use `zip_longest` if lists may differ in length

  

**Q102: What is the Pythonic way to reverse a list?**

A:

- In-place: `lst.reverse()` — mutates, returns None

- New list: `lst[::-1]` or `list(reversed(lst))`

- Prefer `reversed(lst)` when you only need to iterate — it's a lazy iterator

  

**Q103: What is the Pythonic way to merge two dicts?**

A:

- Python 3.9+: `merged = d1 | d2`

- Python 3.5+: `merged = {**d1, **d2}`

- In-place: `d1.update(d2)` — modifies d1

  

**Q104: What does `zip(*matrix)` do? Why is this useful?**

A: Transposes a matrix — converts rows to columns.

```python

matrix = [[1,2,3],[4,5,6],[7,8,9]]

list(zip(*matrix))

# [(1,4,7), (2,5,8), (3,6,9)]

```

`*matrix` unpacks the rows as separate arguments to `zip`, which pairs up elements by position.

  

**Q105: What is the difference between `if x:` and `if x is not None:`? When does it matter?**

A:

- `if x:` — checks truthiness. Fails for `0`, `""`, `[]`, `False` — all falsy but not None

- `if x is not None:` — only checks for None specifically

- Matters when `0`, `False`, or `[]` are valid meaningful values that should not be treated as "missing"

```python

def process(value=None):

if value is not None: # correct — 0 is a valid value

use(value)

```