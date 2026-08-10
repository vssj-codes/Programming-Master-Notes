# Table-of-Contents

<!-- toc -->

- [Python Basics](#python-basics)
  * [Section 1 — Data Types](#section-1--data-types)
  * [Section 2 — Variables & Reference Model](#section-2--variables--reference-model)
  * [Section 3 — Numeric Operations](#section-3--numeric-operations)
  * [Section 4 — Operators](#section-4--operators)
  * [Section 5 — Strings](#section-5--strings)
  * [Section 6 — Control Flow](#section-6--control-flow)
  * [Section 7 — Functions & Arguments](#section-7--functions--arguments)
  * [Section 8 — Unpacking](#section-8--unpacking)
  * [Section 9 — Data Structures](#section-9--data-structures)
  * [Section 10 — Comprehensions](#section-10--comprehensions)
  * [Section 11 — Built-in Functions](#section-11--built-in-functions)
  * [Section 12 — OOP Basics](#section-12--oop-basics)
  * [Section 13 — Mutability](#section-13--mutability)
  * [Section 14 — Scope](#section-14--scope)
  * [Section 15 — Error Handling](#section-15--error-handling)
  * [Section 16 — Type Conversion](#section-16--type-conversion)
  * [Section 17 — Iterables & Iterators](#section-17--iterables--iterators)
  * [Section 18 — Recursion](#section-18--recursion)
  * [Section 19 — Modules & Imports](#section-19--modules--imports)
  * [Section 20 — File I/O](#section-20--file-io)

<!-- tocstop -->

---

# Python Basics 
## Section 1 — Data Types

  

**Q1: How many built-in data types does Python have? Name them all.**

A: 12 built-in types across 6 categories:

- Numeric: `int`, `float`, `complex`, `bool`

- Sequence: `str`, `list`, `tuple`, `range`

- Mapping: `dict`

- Set: `set`, `frozenset`

- Binary: `bytes`, `bytearray`, `memoryview`

- None: `NoneType`

  

**Q2: What are the numeric types in Python?**

A: 4 numeric types:

- `int` — whole numbers, unbounded size

- `float` — decimal, 64-bit IEEE 754

- `complex` — real + imaginary, e.g. `3 + 4j`

- `bool` — subclass of `int`, `True == 1`, `False == 0`

  

**Q3: What is `bool` in Python — is it its own type or a subtype of something?**

A: `bool` is a subtype of `int`.

- `issubclass(bool, int)` → `True`

- `True == 1`, `False == 0`

- `True + True` → `2`

- Only two instances: `True` and `False`

  

**Q4: What is `None`? What is its type?**

A: `None` is a singleton representing the absence of a value.

- Its type is `NoneType`

- Only one instance exists in Python

- Always check with `is None`, not `== None`

- Functions without a `return` statement return `None`

  

**Q5: What is the difference between `set` and `frozenset`?**

A:

- `set` — mutable, unordered, no duplicates. Can add/remove elements.

- `frozenset` — immutable version of set. Hashable, can be used as dict key or in another set.

- Both have O(1) average lookup.

  

**Q6: What are the sequence types in Python?**

A: `str`, `list`, `tuple`, `range`

- All support indexing, slicing, iteration, and `len()`

- `range` is lazy — does not store all values in memory

  

**Q7: What are binary types in Python?**

A: `bytes`, `bytearray`, `memoryview`

- `bytes` — immutable sequence of bytes

- `bytearray` — mutable version of bytes

- `memoryview` — zero-copy view into another binary object's memory

  

---

  

## Section 2 — Variables & Reference Model

  

**Q8: Are Python variables boxes that store values, or something else?**

A: Variables are labels/references pointing to objects in memory, not boxes.

- `a = 5` means: create object `5`, point label `a` at it

- Multiple variables can point to the same object

- Assigning `a = b` makes both labels point to the same object

  

**Q9: What does `id()` return?**

A: The memory address of an object (unique identity for its lifetime).

- `id(a) == id(b)` means `a is b` — same object in memory

- `is` operator compares `id()` values under the hood

  

**Q10: What actually happens in memory when you do `a = b` then `a = 5`?**

A:

- `a = b` — both `a` and `b` point to the same object

- `a = 5` — `a` is rebound to a new object `5`. `b` is unchanged.

- Python variables are independent labels, not aliases to each other

  

**Q11: Python is dynamically typed — what does that mean?**

A: Types are checked at runtime, not at compile time.

- A variable can hold any type and can be rebound to a different type

- `x = 5` then `x = "hello"` is valid

- No type declarations needed

  

**Q12: What is duck typing?**

A: If an object has the methods/attributes needed, it can be used — regardless of its type.

- "If it walks like a duck and quacks like a duck, it's a duck"

- Python doesn't check the type of an object, only whether it supports the required operation

- Example: any object with `__iter__` can be used in a `for` loop

  

---

  

## Section 3 — Numeric Operations

  

**Q13: What is the difference between `/` and `//`?**

A:

- `/` — true division, always returns `float`. `7 / 2` → `3.5`

- `//` — floor division, rounds down to nearest integer. `7 // 2` → `3`, `-7 // 2` → `-4`

  

**Q14: What does `%` return? What is `divmod()`?**

A:

- `%` — modulo, returns the remainder. `7 % 3` → `1`

- `divmod(a, b)` — returns `(a // b, a % b)` as a tuple. `divmod(7, 3)` → `(2, 1)`

  

**Q15: Why does `0.1 + 0.2 != 0.3` in Python?**

A: Floating point numbers are stored in binary (IEEE 754). `0.1` and `0.2` cannot be represented exactly in binary, so small rounding errors accumulate.

- `0.1 + 0.2` → `0.30000000000000004`

- Fix: use `math.isclose()` or the `decimal` module for precision

  

**Q16: How large can a Python `int` get?**

A: Unlimited. Python `int` is arbitrary precision — it grows as needed, limited only by available memory. No overflow like in C/Java.

  

---

  

## Section 4 — Operators

  

**Q17: What is short-circuit evaluation in `and` / `or`?**

A:

- `and` — stops and returns the first falsy value. If all truthy, returns the last value.

- `or` — stops and returns the first truthy value. If all falsy, returns the last value.

- `False and f()` — `f()` is never called

- `True or f()` — `f()` is never called

  

**Q18: What does `not` return?**

A: Always returns a `bool` — `True` or `False`.

- `not 0` → `True`

- `not "hello"` → `False`

- `not None` → `True`

  

**Q19: What is the ternary / conditional expression syntax in Python?**

A: `value_if_true if condition else value_if_false`

- Example: `x = "even" if n % 2 == 0 else "odd"`

- Single line, no need for a full if/else block

  

**Q20: What is the walrus operator `:=`? Give an example.**

A: Assigns a value AND returns it in a single expression. Added in Python 3.8.

```python

if (n := len(data)) > 10:

print(f"Too long: {n}")

```

Avoids calling `len(data)` twice.

  

**Q21: What is the difference between `is` / `is not` and `==` / `!=`?**

A:

- `==` — value equality, calls `__eq__`

- `is` — identity equality, checks if both refer to the same object in memory (`id()`)

- Use `is` only for singletons: `None`, `True`, `False`

- `[] == []` → `True`, but `[] is []` → `False`

  

**Q22: What do `in` and `not in` do?**

A: Membership testing.

- `in` — returns `True` if item is found in the sequence/collection

- `not in` — opposite of `in`

- Calls `__contains__` if defined, otherwise iterates

- O(1) for `set`/`dict`, O(n) for `list`/`tuple`

  

**Q23: What is a chained comparison? e.g. `1 < x < 10`**

A: Python allows chaining comparison operators.

- `1 < x < 10` is equivalent to `1 < x and x < 10`

- `x` is evaluated only once

- More readable and Pythonic than the explicit `and` version

  

---

  

## Section 5 — Strings

  

**Q24: Are strings mutable in Python?**

A: No. Strings are immutable.

- Any operation that "modifies" a string creates a new string object

- `s = "hello"; s[0] = "H"` → `TypeError`

- This is why string concatenation in a loop is inefficient

  

**Q25: What is the difference between `str.split()` and `str.partition()`?**

A:

- `split(sep)` — splits on all occurrences of `sep`, returns a list

- `partition(sep)` — splits on the FIRST occurrence only, always returns a tuple of 3: `(before, sep, after)`

- `"a-b-c".split("-")` → `["a", "b", "c"]`

- `"a-b-c".partition("-")` → `("a", "-", "b-c")`

  

**Q26: Why is `join()` called on the separator rather than the list?**

A: Because `join()` works on any iterable of strings, not just lists. The separator is a string, so it's a string method.

- `", ".join(["a", "b", "c"])` → `"a, b, c"`

- More efficient than `+` concatenation — builds the string in one pass

  

**Q27: What is the difference between f-strings, `.format()`, and `%` formatting?**

A:

- `%` — oldest style, C-style. `"Hello %s" % name`

- `.format()` — more flexible. `"Hello {}".format(name)`

- f-strings — fastest, most readable. `f"Hello {name}"`. Evaluated at runtime. Python 3.6+.

- Prefer f-strings in all modern code.

  

**Q28: How does string slicing work? What does `s[::-1]` do?**

A: `s[start:stop:step]`

- `start` — inclusive, defaults to 0

- `stop` — exclusive, defaults to end

- `step` — increment, defaults to 1

- `s[::-1]` — reverses the string (step of -1, from end to start)

- `s[1:4]` → characters at index 1, 2, 3

  

---

  

## Section 6 — Control Flow

  

**Q29: What is the difference between `break` and `continue`?**

A:

- `break` — exits the loop entirely

- `continue` — skips the rest of the current iteration, moves to the next one

  

**Q30: Can a `for` loop have an `else`? When does it run?**

A: Yes. The `else` block runs only if the loop completed without hitting a `break`.

```python

for i in range(5):

if i == 3:

break

else:

print("no break") # not printed

```

Use case: searching — if item not found (no break), run else.

  

**Q31: What does `pass` do? When would you use it?**

A: `pass` is a no-op — does nothing. Used as a placeholder where syntax requires a statement.

- Empty class body: `class Foo: pass`

- Empty function: `def todo(): pass`

- Empty except: `except Exception: pass` (avoid this in production)

  

**Q32: What is the difference between `while True` and a `for` loop?**

A:

- `for` — iterates over a known sequence or iterable. Finite.

- `while True` — runs indefinitely until a `break`. Use when you don't know how many iterations are needed.

- Use `for` when you have an iterable. Use `while True` for event loops, polling, retry logic.

  

---

  

## Section 7 — Functions & Arguments

  

**Q33: What is the difference between `*args` and `**kwargs`?**

A:

- `*args` — captures extra positional arguments as a tuple

- `**kwargs` — captures extra keyword arguments as a dict

- Both names are convention — `*a` and `**kw` work too

  

**Q34: What is the correct order of parameters in a function signature?**

A: `def f(positional, /, normal, *args, keyword_only, **kwargs)`

1. Positional-only (before `/`)

2. Regular (positional or keyword)

3. `*args`

4. Keyword-only (after `*args` or bare `*`)

5. `**kwargs`

  

**Q35: What does `*` alone in a signature do? e.g. `def f(a, *, b)`**

A: Everything after `*` must be passed as a keyword argument.

- `f(1, b=2)` → works

- `f(1, 2)` → `TypeError`

- Forces callers to be explicit. Improves readability.

  

**Q36: What is the mutable default argument bug?**

A: Default argument objects are created once when the function is defined, not on each call. A mutable default is shared across all calls.

```python

def f(x, lst=[]): # bug: lst is shared

lst.append(x)

return lst

  

def f(x, lst=None): # fix

if lst is None:

lst = []

lst.append(x)

return lst

```

  

**Q37: What is the difference between `print()` and `return`?**

A:

- `print()` — outputs to stdout. Returns `None`. Side effect only.

- `return` — sends a value back to the caller. Ends function execution.

- A function that only prints cannot be used in an expression.

  

**Q38: What do `sep` and `end` do in `print()`?**

A:

- `sep` — separator between multiple arguments. Default is `" "`.

- `end` — what to print at the end. Default is `"\n"`.

- `print("a", "b", sep="-", end="!")` → `a-b!`

  

**Q39: What does a function return if there is no `return` statement?**

A: `None`. Every Python function returns a value. Without an explicit `return`, it returns `None` implicitly.

  

---

  

## Section 8 — Unpacking

  

**Q40: What is tuple unpacking?**

A: Assigning multiple variables from an iterable in one line.

```python

a, b, c = (1, 2, 3)

x, y = "hi"

```

Left side must match the number of elements on the right.

  

**Q41: What is extended unpacking? e.g. `a, *rest, c = [1,2,3,4,5]`**

A: `*` captures multiple values into a list.

- `a = 1`, `rest = [2, 3, 4]`, `c = 5`

- `*` can appear anywhere: `*head, last = [1,2,3]`

- Only one `*` per unpacking expression allowed

  

**Q42: How do you unpack a list/dict into a function call?**

A:

- `*list` — unpacks as positional arguments: `f(*[1, 2, 3])` → `f(1, 2, 3)`

- `**dict` — unpacks as keyword arguments: `f(**{"a": 1})` → `f(a=1)`

  

**Q43: How do you swap two variables in one line in Python?**

A: `a, b = b, a`

- Python evaluates the right side first as a tuple, then unpacks

- No temporary variable needed

  

---

  

## Section 9 — Data Structures

  

**Q44: What is the difference between a `list` and a `tuple`?**

A:

- `list` — mutable, `[]`

- `tuple` — immutable, `()`

- Tuples are faster, use less memory, and are hashable (if elements are hashable)

- Use tuples for fixed data, lists for collections that change

  

**Q45: What is the time complexity of `in` for a list vs a set?**

A:

- `list` — O(n), scans each element

- `set` — O(1) average, uses hash table

  

**Q46: When would you use a `set` over a `list`?**

A:

- When you need fast membership testing — O(1) vs O(n)

- When you need to eliminate duplicates

- When order doesn't matter

- When you need set operations: union, intersection, difference

  

**Q47: How do you merge two dicts in Python 3.9+? And before 3.9?**

A:

- 3.9+: `merged = d1 | d2` or `d1 |= d2` (in-place)

- Before 3.9: `merged = {**d1, **d2}` or `d1.update(d2)` (modifies d1)

  

**Q48: What does `dict.get(key, default)` do differently from `dict[key]`?**

A:

- `dict[key]` — raises `KeyError` if key not found

- `dict.get(key, default)` — returns `default` (or `None`) if key not found, no exception

  

**Q49: What does `dict.setdefault()` do?**

A: Returns the value if key exists. If not, inserts the key with the given default and returns it.

```python

d.setdefault("key", []).append(1)

```

Useful for initializing missing keys without overwriting existing ones.

  

**Q50: What is the difference between `list.append()`, `list.extend()`, and `+`?**

A:

- `append(x)` — adds `x` as a single element. `[1].append([2])` → `[1, [2]]`

- `extend(iterable)` — adds each element of iterable. `[1].extend([2])` → `[1, 2]`

- `+` — creates a new list. Does not modify original.

  

**Q51: What do `dict.keys()`, `.values()`, `.items()` return?**

A: They return view objects, not lists.

- `.keys()` — view of keys

- `.values()` — view of values

- `.items()` — view of `(key, value)` tuples

- Views reflect changes to the dict in real time. Convert with `list()` if you need a snapshot.

  

---

  

## Section 10 — Comprehensions

  

**Q52: What is the difference between a list comprehension and a generator expression?**

A:

- List comprehension `[x for x in it]` — evaluates immediately, stores all in memory

- Generator expression `(x for x in it)` — lazy, produces one item at a time, lower memory

- Use generator when you only need to iterate once and don't need random access

  

**Q53: Write a dict comprehension that squares only even numbers from a list.**

A:

```python

nums = [1, 2, 3, 4, 5]

result = {x: x**2 for x in nums if x % 2 == 0}

# {2: 4, 4: 16}

```

  

**Q54: Can you nest comprehensions? What is the readability tradeoff?**

A: Yes.

```python

flat = [x for row in matrix for x in row]

```

- More than one level of nesting hurts readability

- Rule: if it doesn't read like plain English, use a regular loop instead

  

---

  

## Section 11 — Built-in Functions

  

**Q55: What is the difference between `sorted()` and `list.sort()`?**

A:

- `sorted()` — returns a new sorted list, works on any iterable

- `list.sort()` — sorts in-place, returns `None`, only works on lists

- Both accept `key=` and `reverse=` arguments

  

**Q56: What do `any()` and `all()` return on an empty iterable?**

A:

- `any([])` → `False` — no element is truthy

- `all([])` → `True` — vacuous truth, no element is falsy

- Both short-circuit: stop as soon as the result is determined

  

**Q57: What is the difference between `isinstance()` and `type()`?**

A:

- `type(x) == int` — exact type match only

- `isinstance(x, int)` — returns `True` for subclasses too

- `isinstance(True, int)` → `True` (bool is subclass of int)

- Prefer `isinstance()` in production code

  

**Q58: What do `enumerate()` and `zip()` do?**

A:

- `enumerate(iterable, start=0)` — yields `(index, value)` pairs

- `zip(a, b)` — yields tuples pairing elements from each iterable. Stops at shortest.

- Both return lazy iterators

  

**Q59: What does `map()` return in Python 3?**

A: A lazy iterator, not a list. Wrap with `list()` to get all results.

- `map(func, iterable)` — applies `func` to each element lazily

- Prefer list comprehensions for readability in most cases

  

**Q60: What do `min()` and `max()` do with a `key=` argument?**

A: Apply the key function to each element and compare by the result.

```python

min(["apple", "fig", "banana"], key=len) # "fig"

max([(1,3), (2,1)], key=lambda x: x[1]) # (1,3)

```

  

---

  

## Section 12 — OOP Basics

  

**Q61: What is `self`? Why is it explicit in Python?**

A: `self` is a reference to the current instance. It's the first parameter of instance methods.

- Python passes the instance automatically when you call a method

- It's explicit by design — "explicit is better than implicit" (Zen of Python)

- The name `self` is a convention, not a keyword

  

**Q62: What is `__init__`? Is it a constructor?**

A: `__init__` is an initializer, not a constructor. The actual constructor is `__new__`.

- `__new__` creates the object

- `__init__` initializes it after creation

- In practice, `__init__` is what you override for setup

  

**Q63: What is the difference between a class variable and an instance variable?**

A:

- Class variable — defined in the class body, shared across all instances

- Instance variable — defined in `__init__` via `self.x`, unique per instance

- If a class variable is mutable and modified on the class, all instances see the change

  

**Q64: What does it mean to override a method?**

A: Defining a method in a subclass with the same name as in the parent class. The subclass version takes precedence.

```python

class Animal:

def speak(self): return "..."

  

class Dog(Animal):

def speak(self): return "Woof" # overrides Animal.speak

```

  

**Q65: What is `super()` used for at a basic level?**

A: Calls a method from the parent class without hardcoding the parent's name.

```python

class Dog(Animal):

def __init__(self, name):

super().__init__(name) # calls Animal.__init__

```

  

**Q66: What is the difference between `__str__` and `__repr__`?**

A:

- `__repr__` — unambiguous, for developers. Called by `repr()` and in containers.

- `__str__` — readable, for users. Called by `str()` and `print()`.

- If only one is defined, `__repr__` is used as fallback for both.

- Rule: `__repr__` should ideally be valid Python to recreate the object.

  

---

  

## Section 13 — Mutability

  

**Q67: Which built-in types are mutable? Which are immutable?**

A:

- Mutable: `list`, `dict`, `set`, `bytearray`

- Immutable: `int`, `float`, `complex`, `bool`, `str`, `tuple`, `range`, `frozenset`, `bytes`

  

**Q68: What happens when you pass a mutable object to a function and modify it?**

A: The original object is modified — Python passes references, not copies.

```python

def f(lst):

lst.append(1)

  

x = []

f(x)

print(x) # [1] — original is modified

```

  

**Q69: What is the difference between shallow copy and deep copy?**

A:

- Shallow copy — new container, but inner objects are still shared references. `copy.copy()` or `list[:]`

- Deep copy — fully recursive clone, all nested objects are new. `copy.deepcopy()`

- Modifying a nested object in a shallow copy affects the original.

  

---

  

## Section 14 — Scope

  

**Q70: What is the difference between a local and a global variable?**

A:

- Local — defined inside a function, only accessible within it

- Global — defined at module level, accessible everywhere in the module

- A function can read a global variable without any keyword, but needs `global` to rebind it

  

**Q71: What does the `global` keyword do?**

A: Declares that a variable inside a function refers to the module-level variable, allowing it to be rebound.

```python

x = 0

def f():

global x

x = 10 # modifies module-level x

```

  

**Q72: What does `__name__ == "__main__"` mean and why is it used?**

A: `__name__` is set to `"__main__"` when the file is run directly. When imported as a module, `__name__` is the module name.

- Guards code that should only run when the file is executed directly, not when imported.

  

---

  

## Section 15 — Error Handling

  

**Q73: What is the structure of `try / except / else / finally`? What does each part do?**

A:

- `try` — code that might raise an exception

- `except` — runs if an exception is raised in `try`

- `else` — runs only if NO exception was raised in `try`

- `finally` — always runs, exception or not. Used for cleanup.

  

**Q74: How do you catch multiple exception types?**

A:

```python

except (ValueError, TypeError) as e: # tuple of types

...

except ValueError:

...

except TypeError:

...

```

Most specific exceptions first.

  

**Q75: How do you raise an exception?**

A:

```python

raise ValueError("message") # raise new exception

raise ValueError("msg") from original # chain exceptions

raise # re-raise current exception

```

  

---

  

## Section 16 — Type Conversion

  

**Q76: What is the difference between implicit and explicit type conversion?**

A:

- Implicit — Python converts automatically. `True + 1` → `2` (bool → int)

- Explicit — you convert manually. `int("42")`, `str(3.14)`, `list((1,2))`

  

**Q77: What happens when you do `int("abc")`?**

A: Raises `ValueError: invalid literal for int() with base 10: 'abc'`

- `int()` only works on strings that represent valid integers

  

**Q78: What does `bool()` return for various types?**

A: Returns `False` for falsy values, `True` for everything else.

- Falsy: `None`, `False`, `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `()`

- Everything else is truthy

  

---

  

## Section 17 — Iterables & Iterators

  

**Q79: What is the difference between an iterable and an iterator?**

A:

- Iterable — has `__iter__()`, returns an iterator. e.g. `list`, `str`, `dict`

- Iterator — has both `__iter__()` and `__next__()`. Maintains state, produces one value at a time.

- All iterators are iterables. Not all iterables are iterators.

  

**Q80: What do `iter()` and `next()` do?**

A:

- `iter(obj)` — calls `obj.__iter__()`, returns an iterator

- `next(iterator)` — calls `iterator.__next__()`, returns next value. Raises `StopIteration` when exhausted.

  

**Q81: What protocol makes an object iterable?**

A: Implement `__iter__()` returning an iterator, or `__getitem__()` starting from index 0.

- The iterator itself must implement `__next__()` and `__iter__()` (returning `self`)

  

---

  

## Section 18 — Recursion

  

**Q82: What are the two essential parts of a recursive function?**

A:

- Base case — condition that stops recursion, returns a value directly

- Recursive case — calls itself with a smaller/simpler input moving toward the base case

- Without a base case → infinite recursion → `RecursionError`

  

**Q83: What is Python's default recursion limit and how do you check it?**

A: Default is 1000.

```python

import sys

sys.getrecursionlimit() # 1000

sys.setrecursionlimit(2000) # change it

```

  

---

  

## Section 19 — Modules & Imports

  

**Q84: What is the difference between `import module` and `from module import x`?**

A:

- `import module` — imports the module object. Access with `module.x`

- `from module import x` — imports `x` directly into current namespace

- `from module import *` — imports everything in `__all__` (or all public names)

  

**Q85: What does `__name__ == "__main__"` guard against?**

A: Prevents code from running when the file is imported as a module.

- Without the guard, any top-level code runs on import, causing unintended side effects

  

---

  

## Section 20 — File I/O

  

**Q86: What are the common file modes in `open()`?**

A:

- `r` — read (default)

- `w` — write, truncates file

- `a` — append

- `rb`, `wb` — binary read/write

- `r+` — read and write

- `x` — create, fails if file exists

  

**Q87: Why should you use `with open(...) as f` instead of just `open()`?**

A: `with` guarantees the file is closed when the block exits — even if an exception occurs.

- Without `with`, you must call `f.close()` manually

- If an exception occurs before `f.close()`, the file stays open (resource leak)