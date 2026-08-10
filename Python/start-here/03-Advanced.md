# Table-of-Contents

<!-- toc -->

- [03-Advanced](#03-advanced)
  * [Section 1 — Python Data Model (Dunder Depth)](#section-1--python-data-model-dunder-depth)
  * [Section 2 — Descriptors (Deep)](#section-2--descriptors-deep)
  * [Section 3 — Metaclasses](#section-3--metaclasses)
  * [Section 4 — CPython Internals](#section-4--cpython-internals)
  * [Section 5 — Import System](#section-5--import-system)
  * [Section 6 — Advanced Async & Coroutine Internals](#section-6--advanced-async--coroutine-internals)
  * [Section 7 — Advanced Type System](#section-7--advanced-type-system)
  * [Section 8 — Memory Management (Advanced)](#section-8--memory-management-advanced)
  * [Section 9 — Thread Safety & Concurrency Primitives](#section-9--thread-safety--concurrency-primitives)
  * [Section 10 — Advanced OOP Patterns](#section-10--advanced-oop-patterns)
  * [Section 11 — Design Patterns in Python](#section-11--design-patterns-in-python)
  * [Section 12 — Performance & Profiling](#section-12--performance--profiling)
  * [Section 13 — Advanced Generators & Coroutines](#section-13--advanced-generators--coroutines)

<!-- tocstop -->

---

# 03-Advanced
## Section 1 — Python Data Model (Dunder Depth)

**Q1: What is the Python data model? Why does it matter?**
A: The data model is the set of interfaces (dunder methods) that define how objects behave with Python syntax and built-in functions.
- Implementing `__len__` makes `len(obj)` work
- Implementing `__iter__` makes `for x in obj` work
- Implementing `__add__` makes `obj + other` work
- It lets your custom classes integrate seamlessly with Python's syntax and standard library

**Q2: What is the difference between `__getattr__` and `__getattribute__`? When is each called?**
A:
- `__getattribute__` — called on EVERY attribute access, before checking `__dict__` or class. Override with extreme care — easy to cause infinite recursion.
- `__getattr__` — called ONLY when normal lookup fails (not in `__dict__`, not in class). Safe fallback.
```python
class Proxy:
    def __getattr__(self, name):
        return f"dynamic: {name}"  # only called if attr not found normally
```

**Q3: What is `__missing__`? Which class uses it natively?**
A: Called by `dict.__getitem__` when a key is not found. `defaultdict` uses it natively.
```python
class MyDict(dict):
    def __missing__(self, key):
        self[key] = []  # auto-create
        return self[key]
```
`defaultdict` calls its factory in `__missing__` to generate default values.

**Q4: How do you make a class support `len()`, `[]` indexing, and `in` checks?**
A:
```python
class MyContainer:
    def __init__(self, data):
        self._data = data

    def __len__(self):
        return len(self._data)

    def __getitem__(self, index):
        return self._data[index]

    def __contains__(self, item):
        return item in self._data
```
Implementing `__getitem__` also enables iteration and `in` as fallbacks.

**Q5: What is `__contains__`? What does Python fall back to if it's not defined?**
A: `__contains__` is called by the `in` operator.
- If not defined, Python falls back to iterating the object via `__iter__` and comparing each element
- If `__iter__` is also not defined, falls back to `__getitem__` starting from index 0

**Q6: What is `__bool__` vs `__len__` for truthiness? Which takes precedence?**
A:
- `__bool__` takes precedence — if defined, Python calls it first
- If `__bool__` is not defined, Python falls back to `__len__` — object is falsy if `len() == 0`
- If neither is defined, the object is always truthy

**Q7: What is `__call__`? How do you make an instance callable?**
A: Define `__call__` on the class. The instance can then be used like a function.
```python
class Adder:
    def __init__(self, n): self.n = n
    def __call__(self, x): return x + self.n

add5 = Adder(5)
add5(10)  # 15
callable(add5)  # True
```

**Q8: What is `__del__`? Why is it unreliable and when should you avoid it?**
A: Called when an object is about to be garbage collected. Unreliable because:
- Not guaranteed to be called at all (e.g. interpreter shutdown)
- Not called deterministically with cyclic references
- Can resurrect objects by storing `self` somewhere
- Use context managers (`__enter__`/`__exit__`) for deterministic cleanup instead

**Q9: What is `__init_subclass__`? Give a real use case.**
A: Called on the base class when a subclass is created. Lets the base class customize subclass creation without a metaclass.
```python
class Plugin:
    _registry = {}

    def __init_subclass__(cls, name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if name:
            Plugin._registry[name] = cls

class MyPlugin(Plugin, name="my"): pass

Plugin._registry  # {"my": MyPlugin}
```

**Q10: What is `__set_name__`? When is it called?**
A: Called on a descriptor when the class it belongs to is created. Receives the owner class and the attribute name.
```python
class Validator:
    def __set_name__(self, owner, name):
        self.name = name  # now knows its own attribute name

class User:
    age = Validator()  # __set_name__(User, "age") called here
```
Eliminates the need to pass the attribute name manually to the descriptor.

**Q11: What is `__class_getitem__`? How does it relate to generics?**
A: Called when a class is subscripted with `[]`, e.g. `list[int]` or `MyClass[T]`.
- Built-in types use it to support generic aliases: `list[int]`, `dict[str, int]`
- Custom classes can implement it for generic support with `typing`
```python
class MyContainer:
    def __class_getitem__(cls, item):
        return f"{cls.__name__}[{item}]"

MyContainer[int]  # "MyContainer[<class 'int'>]"
```

**Q12: What is the difference between `__eq__` and `__hash__`? What rule must they follow together?**
A:
- `__eq__` — defines value equality: `a == b`
- `__hash__` — returns an integer used as the hash key in dicts/sets
- Rule: objects that compare equal MUST have the same hash. `a == b` → `hash(a) == hash(b)`
- Violation breaks dict/set lookups silently

**Q13: If you define `__eq__`, what happens to `__hash__` by default? Why?**
A: `__hash__` is set to `None` — the object becomes unhashable.
- Reason: if you change equality semantics, the default `id()`-based hash would be inconsistent
- Fix: explicitly define `__hash__` alongside `__eq__`, or use `@dataclass(unsafe_hash=True)`

---

## Section 2 — Descriptors (Deep)

**Q14: What is the descriptor protocol? Name the three methods.**
A: A descriptor is an object that controls attribute access on another class via:
- `__get__(self, obj, objtype=None)` — called on attribute read
- `__set__(self, obj, value)` — called on attribute write
- `__delete__(self, obj)` — called on `del obj.attr`

**Q15: What is the difference between a data descriptor and a non-data descriptor?**
A:
- Data descriptor — defines `__set__` or `__delete__` (or both). Takes priority over instance `__dict__`.
- Non-data descriptor — defines only `__get__`. Instance `__dict__` takes priority over it.
- `property` is a data descriptor. Functions (methods) are non-data descriptors.

**Q16: Which takes priority — instance `__dict__` or a data descriptor?**
A: Data descriptor wins over instance `__dict__`. Instance `__dict__` wins over non-data descriptor.
Priority order: data descriptor > instance `__dict__` > non-data descriptor > `__getattr__`

**Q17: How does `@property` work under the hood as a descriptor?**
A: `property` is a built-in class that implements `__get__`, `__set__`, `__delete__`.
```python
# @property is equivalent to:
class property:
    def __get__(self, obj, objtype=None):
        if obj is None: return self
        return self.fget(obj)
    def __set__(self, obj, value):
        if self.fset is None: raise AttributeError
        self.fset(obj, value)
```
When you write `obj.x`, Python finds `property` in the class, calls `property.__get__(obj, type(obj))`.

**Q18: How does `@classmethod` work as a descriptor?**
A: `classmethod` implements `__get__` which returns a bound method with the class (not the instance) as the first argument.
```python
class classmethod:
    def __get__(self, obj, cls=None):
        return self.__func__.__get__(cls, type(cls))
```
So `obj.method()` and `Class.method()` both pass the class as first arg.

**Q19: How does `@staticmethod` work as a descriptor?**
A: `staticmethod` implements `__get__` which simply returns the raw function — no binding.
```python
class staticmethod:
    def __get__(self, obj, cls=None):
        return self.__func__  # no binding, no implicit argument
```

**Q20: Implement a type-validated descriptor from scratch.**
A:
```python
class TypedField:
    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __get__(self, obj, objtype=None):
        if obj is None: return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type}, got {type(value)}")
        setattr(obj, self.name, value)

class Person:
    name = TypedField(str)
    age = TypedField(int)
```

**Q21: What is `__set_name__` and how does it help descriptors?**
A: Called when the descriptor is assigned to a class attribute during class creation. Provides the descriptor with the attribute name automatically.
- Without it: you'd pass the name manually in `__init__`
- With it: descriptor self-discovers its name — cleaner API

**Q22: When would you use a descriptor over a `@property`?**
A: When the same validation/logic needs to apply to multiple attributes across multiple classes.
- `@property` is per-attribute, per-class — doesn't reuse
- Descriptor is a reusable class — define once, use on any attribute in any class
- Example: `TypedField`, `PositiveNumber`, `ValidatedString` reused across many models

---

## Section 3 — Metaclasses

**Q23: What is a metaclass? What is the metaclass of a regular class?**
A: A metaclass is the class of a class. Just as an object is an instance of a class, a class is an instance of its metaclass.
- Default metaclass: `type`
- `type(MyClass)` → `<class 'type'>`
- Metaclasses control class creation — `__new__`, `__init__`, `__prepare__`

**Q24: What does `type(name, bases, dict)` do?**
A: Dynamically creates a new class at runtime.
```python
MyClass = type("MyClass", (object,), {"x": 42, "greet": lambda self: "hi"})
obj = MyClass()
obj.greet()  # "hi"
obj.x        # 42
```
This is exactly what Python does internally when it processes a `class` statement.

**Q25: What is the difference between `__new__` and `__init__` in a metaclass?**
A:
- `__new__(mcs, name, bases, namespace)` — creates and returns the class object itself
- `__init__(cls, name, bases, namespace)` — initializes the already-created class
- Override `__new__` to modify the class before it's created; `__init__` to do post-creation setup

**Q26: How do you define a custom metaclass?**
A:
```python
class Meta(type):
    def __new__(mcs, name, bases, namespace):
        # modify namespace before class is created
        namespace["created_by"] = "Meta"
        return super().__new__(mcs, name, bases, namespace)

class MyClass(metaclass=Meta):
    pass

MyClass.created_by  # "Meta"
```

**Q27: What happens when Python creates a class — step by step?**
A:
1. Python reads the `class` body and calls `metaclass.__prepare__()` to get the namespace dict
2. Executes the class body in that namespace
3. Calls `metaclass(name, bases, namespace)` → triggers `metaclass.__new__()` then `metaclass.__init__()`
4. The resulting class object is bound to the name in the enclosing scope

**Q28: What is `__prepare__` in a metaclass? What does it return?**
A: Called before the class body is executed. Returns the namespace dict that the class body runs in.
- Default returns an empty `dict`
- Override to return an `OrderedDict` or custom mapping to control attribute ordering or intercept definitions
```python
class Meta(type):
    @classmethod
    def __prepare__(mcs, name, bases):
        return {}  # or a custom dict-like object
```

**Q29: When would you use a metaclass vs a class decorator vs `__init_subclass__`?**
A:
- `__init_subclass__` — simplest, use for subclass registration, validation, or light customization
- Class decorator — use for post-creation modification of a single class
- Metaclass — use when you need to intercept class creation deeply: modifying the namespace before body executes (`__prepare__`), enforcing interface across an entire hierarchy, or controlling `isinstance`/`issubclass` behavior

**Q30: What is `ABCMeta`? How does it enforce abstract methods?**
A: `ABCMeta` is the metaclass for `ABC`. It tracks which methods are decorated with `@abstractmethod`.
- When a class with `ABCMeta` is instantiated, it checks if all abstract methods are implemented
- If any remain unimplemented → `TypeError`
- `abc.ABC` is a convenience class: `class ABC(metaclass=ABCMeta): pass`

**Q31: What is the metaclass conflict problem? When does it occur?**
A: When a class inherits from two classes with different metaclasses that are not related by inheritance.
```python
class Meta1(type): pass
class Meta2(type): pass
class A(metaclass=Meta1): pass
class B(metaclass=Meta2): pass
class C(A, B): pass  # TypeError: metaclass conflict
```
Fix: create a new metaclass that inherits from both Meta1 and Meta2.

---

## Section 4 — CPython Internals

**Q32: What is CPython? How does it differ from PyPy or Jython?**
A:
- CPython — the reference implementation, written in C. Most widely used.
- PyPy — Python implemented in Python, with a JIT compiler. Faster for long-running CPU-bound code.
- Jython — Python on the JVM. True thread parallelism (no GIL), Java interop.
- IronPython — Python on .NET CLR.
- CPython compiles to bytecode and interprets it. PyPy JIT-compiles hot paths to machine code.

**Q33: What is bytecode? How do you inspect it?**
A: An intermediate representation — Python source is compiled to bytecode before execution. Platform-independent.
```python
import dis
def add(a, b): return a + b
dis.dis(add)
# Shows LOAD_FAST, BINARY_OP, RETURN_VALUE instructions
```
Also accessible via `add.__code__.co_code`.

**Q34: What does the `dis` module do?**
A: Disassembles Python bytecode into human-readable instructions.
- `dis.dis(func)` — shows bytecode for a function
- `dis.compile()` — compiles source to code object
- Useful for understanding performance, verifying optimizations, and debugging

**Q35: What is a `.pyc` file? Where is it stored?**
A: Compiled bytecode cache. Python compiles `.py` to `.pyc` to skip re-parsing on subsequent imports.
- Stored in `__pycache__/` directory alongside the source, named `module.cpython-3X.pyc`
- Python checks the source's modification time — regenerates if source is newer
- Speeds up import, not execution

**Q36: What is a frame object in CPython?**
A: A frame represents one level of the call stack — one function call in execution.
- Contains: local variables, the code object, instruction pointer, reference to the enclosing frame
- `inspect.currentframe()` returns the current frame
- Generators store their frame to resume execution at the `yield` point

**Q37: What is the GIL — how is it implemented and when is it released?**
A: A mutex in CPython that allows only one thread to execute Python bytecode at a time.
- Implemented as a simple lock in the C interpreter
- Released: during I/O operations, `time.sleep()`, C extension calls that explicitly release it (e.g. numpy), every `sys.getswitchinterval()` seconds (default 5ms)
- This is why threading helps I/O-bound but not CPU-bound tasks

**Q38: What operations are thread-safe in CPython due to the GIL?**
A: Operations that are atomic at the bytecode level:
- `list.append()`, `list.pop()`
- `dict.__setitem__`, `dict.__getitem__`
- Reading/writing simple variables
- But compound operations like `n += 1` (read-modify-write) are NOT atomic — three bytecode instructions

**Q39: What is reference counting? What is `sys.getrefcount()` and why does it return n+1?**
A: Every object has a count of references pointing to it. When it hits 0, the object is freed immediately.
- `sys.getrefcount(obj)` returns count + 1 because passing `obj` to the function creates a temporary reference

**Q40: What problem does the cyclic garbage collector solve that reference counting cannot?**
A: Reference cycles — two or more objects referencing each other with no external references.
```python
a = {}; b = {}
a["b"] = b; b["a"] = a
del a, b  # ref counts never reach 0 — cycle
```
CPython's cyclic GC periodically scans for isolated cycles and breaks them.

**Q41: What is `gc.collect()`? When would you call it manually?**
A: Triggers a full garbage collection cycle immediately.
- Call when: you deleted a large object graph with cycles and need memory freed immediately
- Generally not needed — the GC runs automatically
- Can disable with `gc.disable()` in performance-critical code that avoids cycles

**Q42: What is integer caching in CPython? What is the cached range?**
A: CPython pre-allocates integer objects for the range `[-5, 256]`. All references to these values share the same object.
- `a = 256; b = 256; a is b` → True
- `a = 257; b = 257; a is b` → False (not guaranteed)
- Implementation detail — do not rely on this in production code

**Q43: What is string interning? How do you force it with `sys.intern()`?**
A: Python automatically interns string literals that look like identifiers (no spaces, alphanumeric + underscore).
- Interned strings share the same object — allows `is` comparison (faster than `==`)
- `sys.intern("my string")` forces interning for any string
- Useful for: large dictionaries with repeated string keys, parsers, compilers

---

## Section 5 — Import System

**Q44: What happens step by step when Python executes `import foo`?**
A:
1. Check `sys.modules` — if already imported, return cached module
2. Find the module using `sys.meta_path` finders
3. Load the module (read file, compile to bytecode)
4. Execute the module's code in a new namespace
5. Store in `sys.modules[name]`
6. Bind the name in the importing namespace

**Q45: What is `sys.modules`? What happens if you delete an entry from it?**
A: A dict mapping module names to loaded module objects — the import cache.
- All imports check here first
- Deleting an entry forces Python to re-import the module on next `import`
- Used for: reloading modules, mocking in tests, hot-reloading

**Q46: What is the difference between a module and a package?**
A:
- Module — a single `.py` file
- Package — a directory containing an `__init__.py` file (regular package) or without it (namespace package)
- `__init__.py` runs when the package is imported; controls what `from pkg import *` exports

**Q47: What is `__all__`? How does it affect `from module import *`?**
A: A list of public names to export.
- `from module import *` only imports names listed in `__all__`
- Without `__all__`, imports all names not starting with `_`
- Best practice: always define `__all__` in modules meant to be imported with `*`

**Q48: What is the difference between relative and absolute imports?**
A:
- Absolute: `from package.module import foo` — full path from project root
- Relative: `from .module import foo` (same package), `from ..module import foo` (parent package)
- Relative imports only work inside packages
- PEP 8: prefer absolute imports for clarity

**Q49: What causes a circular import error? How do you fix it?**
A: Module A imports from B, and B imports from A — during A's execution, B tries to import A which isn't fully loaded yet.
Fixes:
1. Move the import inside the function that uses it (deferred import)
2. Restructure to extract shared code into a third module
3. Import the module itself instead of specific names: `import module` then use `module.name`

**Q50: What is an import hook? What is `sys.meta_path`?**
A: Import hooks customize how Python finds and loads modules.
- `sys.meta_path` — list of finder objects checked in order during import
- Each finder implements `find_spec(name, path, target)` — returns a `ModuleSpec` or None
- Use cases: import from zip files, databases, remote URLs, encrypted sources

**Q51: What does `importlib.import_module()` do?**
A: Programmatically imports a module by name string at runtime.
```python
import importlib
mod = importlib.import_module("os.path")
# equivalent to: import os.path; mod = os.path
```
Useful for plugin systems and dynamic loading.

**Q52: What is a namespace package (PEP 420)?**
A: A package without `__init__.py`. Allows a package to span multiple directories.
- Python merges all directories with the same package name into one virtual package
- Useful for: distributing sub-packages separately, corporate namespace packages like `com.company.project`

---

## Section 6 — Advanced Async & Coroutine Internals

**Q53: What is the difference between a coroutine and a generator at the protocol level?**
A:
- Generator protocol: `__iter__` + `__next__`, drives with `next()` and `send()`
- Coroutine protocol: `__await__`, drives with `send()` only, must be awaited
- A coroutine object has `send()`, `throw()`, `close()` — same as generator
- `async def` creates a coroutine function. `def` with `yield` creates a generator function.
- Before 3.5: coroutines were generators decorated with `@asyncio.coroutine`

**Q54: What does `await` actually do under the hood?**
A: `await expr` calls `expr.__await__()` to get an iterator, then drives it with `send()`/`throw()` until `StopIteration` — the value in `StopIteration` is the result.
- It yields control back to the event loop when the awaitable suspends
- The event loop resumes the coroutine by calling `send(result)` when the I/O is ready

**Q55: What is the `__await__` protocol?**
A: An object is awaitable if it implements `__await__()` returning an iterator.
- `asyncio.Future` implements `__await__`
- Coroutines are awaitable — their `__await__` returns `self`
- `yield` inside `__await__` suspends to the event loop; value sent back is the I/O result

**Q56: What is the difference between `asyncio.gather()` and `asyncio.TaskGroup` (3.11+)?**
A:
- `gather()` — runs coroutines concurrently, collects results. If one fails and `return_exceptions=False`, cancels others.
- `TaskGroup` — structured concurrency. If any task raises, ALL others are cancelled immediately. Cleaner error handling.
```python
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(coro1())
    t2 = tg.create_task(coro2())
# both done here, any exception propagates as ExceptionGroup
```
Prefer `TaskGroup` for structured concurrency in Python 3.11+.

**Q57: What is task cancellation? How do you handle `asyncio.CancelledError` correctly?**
A: `task.cancel()` schedules a `CancelledError` to be thrown into the coroutine at the next `await`.
```python
async def my_coro():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        # cleanup here
        raise  # MUST re-raise — do not swallow CancelledError
```
Swallowing `CancelledError` prevents proper task cancellation.

**Q58: What is `asyncio.shield()`? When would you use it?**
A: Protects a coroutine from being cancelled when the outer task is cancelled.
```python
result = await asyncio.shield(important_coro())
```
- If the surrounding task is cancelled, `shield` raises `CancelledError` to the caller but the inner coroutine continues
- Use when you have cleanup/commit operations that must complete even if the caller cancels

**Q59: What are asyncio synchronization primitives? Name them and their use cases.**
A:
- `asyncio.Lock` — mutual exclusion, only one coroutine at a time
- `asyncio.Event` — signal between coroutines; `wait()` blocks until `set()` is called
- `asyncio.Semaphore` — limit concurrency to N coroutines (e.g. max 10 DB connections)
- `asyncio.Condition` — like threading.Condition but async; wait for a condition
- `asyncio.Queue` — producer-consumer communication

**Q60: What is an async generator? How does `async for` work?**
A: A function with `async def` and `yield`. Produces values asynchronously.
```python
async def ticker(n):
    for i in range(n):
        await asyncio.sleep(0.1)
        yield i

async for value in ticker(5):
    print(value)
```
`async for` calls `__aiter__()` then repeatedly calls `__anext__()`, awaiting each result.

**Q61: What is `asyncio.Queue`? How does it help with producer-consumer patterns?**
A: An async-safe FIFO queue. `put()` and `get()` are coroutines that await when full/empty.
```python
queue = asyncio.Queue(maxsize=10)

async def producer():
    await queue.put(item)

async def consumer():
    item = await queue.get()
    queue.task_done()

await queue.join()  # wait until all items processed
```

**Q62: What is `loop.run_in_executor()`? When would you use `ProcessPoolExecutor` vs `ThreadPoolExecutor` here?**
A: Runs a blocking function in a thread or process pool without blocking the event loop.
```python
loop = asyncio.get_event_loop()
result = await loop.run_in_executor(None, blocking_func, arg)  # None = default ThreadPoolExecutor
```
- `ThreadPoolExecutor` — blocking I/O, C extensions that release the GIL
- `ProcessPoolExecutor` — CPU-bound work that needs true parallelism

**Q63: How do you run multiple event loops? Is it safe?**
A: Each thread can have its own event loop, but only one event loop per thread.
- `asyncio.get_event_loop()` gets/creates a loop for the current thread
- Running two loops in the same thread is not supported
- Use `loop.run_until_complete()` in a separate thread for nested event loop scenarios, or `asyncio.run()` which always creates a fresh loop

**Q64: What is `contextvars.ContextVar`? How does it differ from a global variable in async code?**
A: A context-local variable — each task/coroutine has its own isolated copy.
```python
from contextvars import ContextVar
request_id = ContextVar("request_id", default=None)

async def handler():
    request_id.set("abc-123")
    await process()  # process() sees "abc-123" even across awaits

async def process():
    print(request_id.get())  # "abc-123" — not affected by other concurrent handlers
```
A global variable would be shared and overwritten by concurrent tasks.

---

## Section 7 — Advanced Type System

**Q65: What is a `TypeVar`? What are `bound` and `constraints`?**
A: `TypeVar` defines a generic type placeholder that can be resolved to a specific type.
```python
T = TypeVar("T")                    # unconstrained
S = TypeVar("S", bound=Sequence)    # must be Sequence or subclass
N = TypeVar("N", int, float)        # must be exactly int or float
```

**Q66: What is the difference between `TypeVar(bound=X)` and `TypeVar(X, A, B)`?**
A:
- `bound=X` — T can be X or any subclass of X. More flexible.
- `TypeVar("T", A, B)` — T must be exactly A or B (or subclasses). More restrictive, acts like Union.
- `bound` is preferred when you want "any subtype of X"; constraints when you want specific choices.

**Q67: What is `ParamSpec`? When do you need it?**
A: Captures the parameter types of a callable — used when writing decorators that preserve the wrapped function's signature.
```python
from typing import ParamSpec, Callable, TypeVar
P = ParamSpec("P")
R = TypeVar("R")

def decorator(func: Callable[P, R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        return func(*args, **kwargs)
    return wrapper
```
Without `ParamSpec`, type checkers lose track of argument types through decorators.

**Q68: What is `Concatenate`? How does it work with `ParamSpec`?**
A: Used with `ParamSpec` to prepend parameters to a callable's signature.
```python
from typing import Concatenate
def with_user(func: Callable[Concatenate[User, P], R]) -> Callable[P, R]:
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        user = get_current_user()
        return func(user, *args, **kwargs)
    return wrapper
```
Expresses "this function takes a User as first arg plus whatever else".

**Q69: What is `TypeGuard`? Write a type guard function.**
A: A return type that tells the type checker a predicate narrows the type of its argument.
```python
from typing import TypeGuard

def is_str_list(val: list[object]) -> TypeGuard[list[str]]:
    return all(isinstance(x, str) for x in val)

items: list[object] = ["a", "b"]
if is_str_list(items):
    items  # type checker now knows this is list[str]
```

**Q70: What is `@overload`? Why can't you just use `Union` in its place?**
A: `@overload` lets you declare multiple signatures for a function with different input/output type relationships.
```python
@overload
def process(x: int) -> int: ...
@overload
def process(x: str) -> str: ...
def process(x):
    return x

result = process(42)   # type checker knows result is int, not int | str
```
`Union` can't express "if input is int then output is int" — it would say output is `int | str` always.

**Q71: What is `Protocol` with `runtime_checkable`? What does `isinstance()` check against it?**
A:
```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> str: ...

isinstance(Circle(), Drawable)  # True if Circle has draw()
```
Without `@runtime_checkable`, `isinstance` raises `TypeError`. With it, only checks for method presence — not signatures.

**Q72: What is `NewType`? How is it different from a type alias?**
A:
- Type alias: `UserId = int` — just another name, completely interchangeable with int
- `NewType`: `UserId = NewType("UserId", int)` — creates a distinct type for type checkers. Passing a plain `int` where `UserId` is expected is a type error.
- At runtime, `NewType` is the identity function — no overhead

**Q73: What is `TypeAlias` (PEP 613)?**
A: Explicitly marks a variable as a type alias to avoid ambiguity with regular assignments.
```python
from typing import TypeAlias
Vector: TypeAlias = list[float]  # unambiguous — this is a type alias
```
Without `TypeAlias`, type checkers may be confused about whether it's an alias or a variable.

**Q74: What is `Self` type (Python 3.11+)?**
A: Refers to the current class in type hints — useful for methods that return `self` or `cls`.
```python
from typing import Self

class Builder:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self  # type checker knows this returns the actual subclass type
```
Without `Self`, you'd use `TypeVar("T", bound="Builder")` — more verbose.

**Q75: What is `Unpack` and `TypeVarTuple`? What problem do they solve?**
A: Enable variadic generics — typed tuples with variable-length type parameters.
```python
from typing import TypeVarTuple, Unpack
Ts = TypeVarTuple("Ts")

def broadcast(func: Callable[[Unpack[Ts]], None], *args: Unpack[Ts]) -> None: ...
```
Solves typing of functions like `zip`, `map`, or neural network layer shapes where the number of types is variable.

---

## Section 8 — Memory Management (Advanced)

**Q76: What is `weakref`? When would you use a weak reference?**
A: A reference that doesn't increment the object's reference count — doesn't prevent garbage collection.
```python
import weakref
obj = SomeClass()
ref = weakref.ref(obj)
ref()  # returns obj if alive, None if collected
```
Use cases: caches (allow entries to be GC'd), observer patterns (avoid preventing cleanup), circular references.

**Q77: What is a `WeakValueDictionary`? Give a use case.**
A: A dict where values are weak references — entries are automatically removed when the value is garbage collected.
```python
cache = weakref.WeakValueDictionary()
cache["key"] = obj  # auto-removed when obj has no other references
```
Use case: object registry or cache that shouldn't prevent objects from being freed.

**Q78: What is `__slots__`'s effect on `__dict__` and memory layout?**
A: Eliminates `__dict__` entirely from instances. Instead, uses a fixed C-level array for the declared attributes.
- Each instance without `__slots__`: `__dict__` costs ~200-400 bytes
- With `__slots__`: only the slot values stored — typically 8-16 bytes per slot
- Cannot add arbitrary attributes at runtime

**Q79: How do you profile memory usage in Python?**
A:
- `tracemalloc` (stdlib) — traces memory allocations, find top allocators
- `memory_profiler` (third party) — line-by-line memory usage with `@profile`
- `objgraph` — visualize object references, find leaks
- `sys.getsizeof()` — size of a single object (shallow)
```python
import tracemalloc
tracemalloc.start()
# ... code ...
snapshot = tracemalloc.take_snapshot()
top = snapshot.statistics("lineno")
```

**Q80: What is copy-on-write? Does Python use it?**
A: COW delays copying data until a write occurs — multiple references share the same memory until one modifies it.
- CPython does NOT use COW for Python objects
- The OS uses COW when `fork()` is called — child process shares parent's memory pages until writing
- This is why multiprocessing with `fork` can appear memory-efficient initially

**Q81: What is the difference between `del x` and setting `x = None`?**
A:
- `del x` — removes the name binding from the namespace. The object's reference count decreases. If count hits 0, object is freed.
- `x = None` — rebinds the name to `None`. The old object's reference count decreases, but the name `x` still exists.
- `del x` after makes `x` raise `NameError` if accessed. `x = None` still allows access.

---

## Section 9 — Thread Safety & Concurrency Primitives

**Q82: What Python operations are atomic due to the GIL?**
A: Operations that are a single bytecode instruction:
- Reading/writing a variable reference
- `list.append()`, `list.pop()`
- `dict.__getitem__`, `dict.__setitem__`
- Fetching a list element by index
Not atomic: `n += 1` (three instructions: LOAD, ADD, STORE)

**Q83: Is `list.append()` thread-safe? What about `dict` updates?**
A:
- `list.append()` — yes, single bytecode op, GIL protects it
- `dict[key] = value` — yes for a single assignment
- But `d[k] = d.get(k, 0) + 1` — NOT atomic, read-modify-write race condition
- For thread-safe counters: use `threading.Lock` or `collections.Counter` with a lock

**Q84: What is `threading.Lock` vs `threading.RLock`?**
A:
- `Lock` — basic mutex. Only one thread at a time. If the same thread tries to acquire it twice → deadlock.
- `RLock` (reentrant lock) — the same thread can acquire it multiple times without deadlocking. Must release the same number of times.
- Use `RLock` when a function holding a lock might call another function that also tries to acquire the same lock.

**Q85: What is `threading.Event`? What is `threading.Condition`?**
A:
- `Event` — a simple flag. `set()` signals it, `wait()` blocks until set, `clear()` resets it. One-to-many notification.
- `Condition` — combines a lock with the ability to wait for a predicate. `wait()` releases lock and blocks; `notify()` wakes one waiter; `notify_all()` wakes all.
- Use Condition for producer-consumer where you need to wait for a specific state.

**Q86: What is `threading.local()`? What problem does it solve?**
A: Creates a thread-local storage object — each thread has its own independent copy of the attributes.
```python
local = threading.local()
local.value = 42  # each thread sees its own `value`
```
Solves: sharing state between functions in the same thread without passing it as arguments. Common in web frameworks for per-request context (DB sessions, user info).

**Q87: What is a race condition? Give a Python example.**
A: When two threads access shared state concurrently and the result depends on timing.
```python
counter = 0
def increment():
    global counter
    for _ in range(100000):
        counter += 1  # read-modify-write — not atomic

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)
t1.start(); t2.start()
t1.join(); t2.join()
print(counter)  # less than 200000 due to race
```
Fix: use a `Lock` around the `counter += 1`.

**Q88: What is a deadlock? How do you avoid it?**
A: Two or more threads each hold a lock the other needs — both wait forever.
```python
# Thread 1: acquire lock_a, then lock_b
# Thread 2: acquire lock_b, then lock_a → deadlock
```
Avoidance strategies:
- Always acquire locks in the same order across all threads
- Use `Lock.acquire(timeout=...)` with timeouts
- Use `threading.RLock` for reentrant cases
- Prefer higher-level abstractions (`queue.Queue`, `concurrent.futures`)

---

## Section 10 — Advanced OOP Patterns

**Q89: What is the Singleton pattern in Python? Implement it three ways.**
A:
```python
# Way 1: __new__
class Singleton:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Way 2: module-level instance (simplest, Pythonic)
# Just use a module — modules are singletons by nature

# Way 3: metaclass
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
```

**Q90: What is the Mixin pattern? Give a real use case.**
A: A class that provides methods to other classes via multiple inheritance, without being meant to stand alone.
```python
class TimestampMixin:
    def set_timestamps(self):
        self.created_at = datetime.now()
        self.updated_at = datetime.now()

class LogMixin:
    def log(self, msg):
        print(f"[{self.__class__.__name__}] {msg}")

class User(TimestampMixin, LogMixin, BaseModel):
    pass
```
Real use cases: Django's `LoginRequiredMixin`, serialization mixins, logging mixins.

**Q91: What is monkey patching? When is it acceptable?**
A: Replacing or extending a module, class, or function at runtime after it's been imported.
```python
import module
module.some_function = my_replacement
```
Acceptable: test mocking, hotfixes for third-party bugs, adding missing functionality to a library
Not acceptable: production code that alters library behavior for others, hard to debug

**Q92: What is the Registry pattern using `__init_subclass__`?**
A: Auto-register subclasses into a central registry on class creation.
```python
class Handler:
    _registry = {}

    def __init_subclass__(cls, event_type=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if event_type:
            Handler._registry[event_type] = cls

class ClickHandler(Handler, event_type="click"): pass
class HoverHandler(Handler, event_type="hover"): pass

Handler._registry  # {"click": ClickHandler, "hover": HoverHandler}
```

**Q93: What is multiple dispatch? How would you implement it in Python?**
A: Selecting a function implementation based on the types of multiple arguments (not just the first, like OOP dispatch).
```python
from functools import singledispatch

@singledispatch
def process(arg):
    raise NotImplementedError

@process.register(int)
def _(arg): return f"int: {arg}"

@process.register(str)
def _(arg): return f"str: {arg}"
```
For multiple arguments: use `multipledispatch` library or manual `isinstance` dispatch.

**Q94: What is the difference between composition and inheritance? When do you choose each?**
A:
- Inheritance — "is-a" relationship. Subclass shares and extends parent behavior. Tight coupling.
- Composition — "has-a" relationship. Object delegates to other objects. Looser coupling.
- Prefer composition: when the behavior is optional, may change, or comes from multiple sources
- Use inheritance: for true is-a relationships, framework extension points, polymorphism
- Rule: favor composition over inheritance

**Q95: What is the `__init_subclass__` hook? How does it differ from a metaclass?**
A:
- `__init_subclass__` — simpler, defined on the base class, called when any subclass is defined
- Metaclass — more powerful, intercepts the entire class creation process including `__prepare__`
- Use `__init_subclass__` for: validation, registration, setting class attributes on subclasses
- Use metaclass for: modifying the namespace before the class body runs, controlling `isinstance`

---

## Section 11 — Design Patterns in Python

**Q96: How do you implement the Observer pattern in Python?**
A:
```python
class EventEmitter:
    def __init__(self):
        self._listeners = {}

    def on(self, event, callback):
        self._listeners.setdefault(event, []).append(callback)

    def emit(self, event, *args, **kwargs):
        for cb in self._listeners.get(event, []):
            cb(*args, **kwargs)

emitter = EventEmitter()
emitter.on("data", lambda x: print(f"got: {x}"))
emitter.emit("data", 42)
```

**Q97: How do you implement the Factory pattern?**
A:
```python
class Animal: pass
class Dog(Animal): pass
class Cat(Animal): pass

def animal_factory(kind: str) -> Animal:
    registry = {"dog": Dog, "cat": Cat}
    cls = registry.get(kind)
    if cls is None:
        raise ValueError(f"Unknown animal: {kind}")
    return cls()

# Class method variant:
class Animal:
    @classmethod
    def create(cls, kind):
        subclasses = {c.__name__.lower(): c for c in cls.__subclasses__()}
        return subclasses[kind]()
```

**Q98: How do you implement the Strategy pattern using first-class functions?**
A:
```python
def sort_by_name(items): return sorted(items, key=lambda x: x.name)
def sort_by_price(items): return sorted(items, key=lambda x: x.price)

class ProductList:
    def __init__(self, sort_strategy=sort_by_name):
        self.sort_strategy = sort_strategy

    def get_sorted(self, items):
        return self.sort_strategy(items)

pl = ProductList(sort_strategy=sort_by_price)
```
In Python, functions are first-class — no need for a Strategy interface/class hierarchy.

**Q99: How do you implement the Decorator pattern (structural, not the `@` syntax)?**
A: Wrap an object to add behavior without changing its interface.
```python
class Coffee:
    def cost(self): return 5
    def description(self): return "Coffee"

class MilkDecorator:
    def __init__(self, coffee):
        self._coffee = coffee
    def cost(self): return self._coffee.cost() + 1
    def description(self): return self._coffee.description() + ", Milk"

c = MilkDecorator(Coffee())
c.cost()         # 6
c.description()  # "Coffee, Milk"
```

**Q100: What is the `contextlib.contextmanager` pattern and when is it a better fit than a class?**
A: A generator-based context manager — code before `yield` is `__enter__`, after is `__exit__`.
```python
@contextlib.contextmanager
def timer():
    start = time.perf_counter()
    yield
    print(f"elapsed: {time.perf_counter() - start:.3f}s")
```
Better than a class when: the context manager is simple, single-use, or you want less boilerplate. Use a class when you need multiple methods, state between enter/exit, or reuse across many places.

---

## Section 12 — Performance & Profiling

**Q101: How do you profile a Python program? Name two tools.**
A:
- `cProfile` (stdlib) — function-level call stats: calls, time, cumulative time
- `line_profiler` (third party) — line-by-line timing with `@profile`
- `tracemalloc` (stdlib) — memory allocation profiling
- `py-spy` — sampling profiler, zero overhead, attaches to running processes

**Q102: What is `cProfile`? How do you use it?**
A:
```python
import cProfile
cProfile.run("my_function()")

# Or from command line:
# python -m cProfile -s cumtime my_script.py
```
Output: ncalls, tottime (exclusive), cumtime (inclusive), per-call stats. Sort by `cumtime` to find bottlenecks.

**Q103: What is `timeit`? When would you use it over `time.perf_counter()`?**
A: Measures execution time of small code snippets precisely, with multiple runs to average out noise.
```python
import timeit
timeit.timeit("'-'.join(str(n) for n in range(100))", number=10000)
```
Use `timeit` for microbenchmarks of expressions. Use `time.perf_counter()` for profiling sections of real code.

**Q104: What is the cost of attribute lookup in Python? How does `__slots__` help?**
A: Normal attribute lookup: check `__dict__`, then class, then MRO — involves dict lookups at each step.
- `__slots__` replaces `__dict__` with a C-level array — direct offset access, no dict hashing
- Measurably faster for tight loops accessing attributes millions of times

**Q105: When would you reach for `numpy` instead of pure Python lists?**
A:
- Numeric arrays with vectorized operations — numpy is 10-100x faster
- Operations on entire arrays without Python loops: `arr * 2`, `arr.sum()`, `arr[arr > 0]`
- Memory: numpy arrays are contiguous typed memory vs Python lists of object pointers
- Use when: numerical computation, matrix operations, signal processing, ML preprocessing

**Q106: What is `functools.lru_cache` doing to speed things up? What is its memory tradeoff?**
A: Stores results in a dict keyed by arguments. On a cache hit, returns stored result in O(1) instead of recomputing.
- Memory grows with unique inputs — unbounded if `maxsize=None`
- LRU eviction with `maxsize=N` keeps memory bounded but adds eviction overhead
- Not suitable for: functions with mutable arguments, functions with side effects, very large result objects

**Q107: What is the fastest way to concatenate many strings in Python and why?**
A: `"".join(list_of_strings)` — O(n) total.
- `+=` in a loop: creates a new string each time, copies all previous content — O(n²)
- `join` pre-calculates total length, allocates once, copies each string once
- For building incrementally: use a list and join at the end

---

## Section 13 — Advanced Generators & Coroutines

**Q108: What is a coroutine-based generator (pre-async/await)? How does `send()` enable it?**
A: Before `async/await` (pre-3.5), coroutines were plain generators driven by `send()`.
```python
def coroutine():
    while True:
        value = yield          # receive via send()
        print(f"got: {value}")

c = coroutine()
next(c)           # prime
c.send("hello")   # "got: hello"
c.send("world")   # "got: world"
```
`yield` both suspends and receives — enabling two-way communication. `asyncio` used this internally before `async/await`.

**Q109: What is `yield from` doing at the protocol level when used with another generator?**
A: `yield from subgen` fully delegates to the subgenerator:
1. Forwards `next()` and `send()` calls into the subgenerator
2. Forwards `throw()` and `close()` into the subgenerator
3. Captures the `return` value from the subgenerator's `StopIteration` and returns it as the value of the `yield from` expression
- This is how asyncio worked before `await` — `await` is essentially `yield from` for coroutines

**Q110: What is an async generator? How do you write one?**
A: An `async def` function that contains `yield`. Produces values asynchronously.
```python
async def paginate(url, pages):
    for page in range(pages):
        data = await fetch(url, page=page)  # async I/O
        yield data                          # produce value

async for page in paginate("http://api.com", 5):
    process(page)
```
Cannot use `return value` (only bare `return`). Cannot use `yield` and `await` in same expression.

**Q111: What is the difference between `async for` and a regular `for` loop with an async iterator?**
A:
- `async for` — calls `__aiter__()` then awaits `__anext__()` each iteration. Must be inside `async def`.
- Regular `for` — calls `__iter__()` then `__next__()` — synchronous, blocks if I/O is involved.
- `async for` allows the event loop to handle other tasks between iterations.

**Q112: What is `contextlib.asynccontextmanager`?**
A: Creates an async context manager from an async generator function. Used with `async with`.
```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def db_transaction(conn):
    await conn.begin()
    try:
        yield conn
        await conn.commit()
    except Exception:
        await conn.rollback()
        raise

async with db_transaction(conn) as tx:
    await tx.execute("INSERT ...")
```
Code before `yield` = `__aenter__`. Code after `yield` = `__aexit__`.
