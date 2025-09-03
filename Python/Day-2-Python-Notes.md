# Table-of-Contents

<!-- toc -->

- [Random-Module](#random-module)
      - [Important-Notes](#important-notes)
      - [Code-Snippet](#code-snippet)
      - [Key-Takeaway](#key-takeaway)
- [CSV-Module](#csv-module)
      - [Important-Notes](#important-notes-1)
      - [Code-Snippet](#code-snippet-1)
      - [Key-Takeaway](#key-takeaway-1)
- [RegEx-Module](#regex-module)
      - [Important-Notes](#important-notes-2)
      - [Code-Snippet](#code-snippet-2)
      - [Key-Takeaway](#key-takeaway-2)
- [Error-Handling](#error-handling)
      - [Important-Notes](#important-notes-3)
      - [Code-Snippet](#code-snippet-3)
      - [Key-Takeaway](#key-takeaway-3)
- [f-Strings](#f-strings)
      - [Important-Notes](#important-notes-4)
      - [Code-Snippet](#code-snippet-4)
      - [Key-Takeaway](#key-takeaway-4)
- [Generators](#generators)
      - [Important-Notes](#important-notes-5)
      - [Code-Snippet](#code-snippet-5)
      - [Key-Takeaway](#key-takeaway-5)

<!-- tocstop -->

---

# Random-Module
#### Important-Notes

* Provides random numbers, selections, and distributions.
* `random()` → float \[0.0, 1.0)
* `uniform(a, b)` → float in range \[a, b]
* `randint(a, b)` → int in range \[a, b]
* `randrange(start, stop, step)` → int with step
* `choice()` / `choices()` / `sample()` → pick items
* `shuffle()` → in-place shuffle of list
* `seed()` → reproducibility (fixed results)
* Has built-in probability distributions (`gauss`, `expovariate`, etc.)

#### Code-Snippet

```python
import random

# 1. Random Numbers
print(random.random())        # Output: 0.6394... (float between 0 and 1)
print(random.uniform(1, 5))   # Output: 3.74...  (float between 1 and 5)
print(random.randint(1, 10))  # Output: 7        (int between 1 and 10)
print(random.randrange(1, 10, 2))  # Output: 3  (odd number < 10)

# 2. Choice & Sampling
items = [1, 2, 3, 4, 5]
print(random.choice(items))        # Output: 4  (random element)
print(random.choices(items, k=3))  # Output: [2, 5, 5] (with replacement)
print(random.sample(items, 3))     # Output: [1, 4, 5] (unique elements)

# 3. Shuffle
random.shuffle(items)
print(items)  # Output: [3, 5, 1, 2, 4]

# 4. Seed
random.seed(42)
print(random.randint(1, 10))  # Output: 2 (fixed result)

# 5. Distributions
print(random.gauss(0, 1))        # Output: -0.1440... (Gaussian)
print(random.expovariate(1))     # Output: 0.234...   (Exponential)
print(random.betavariate(2, 5))  # Output: 0.145...   (Beta)
```

#### Key-Takeaway

* `random.random()` → float \[0.0, 1.0).
* `randint` / `uniform` → numbers in range.
* `choice` / `sample` → pick items.
* `shuffle` → reorder list.
* `seed` → reproducibility for debugging.

# CSV-Module

#### Important-Notes

* `csv.reader` → read rows from CSV.
* `csv.DictReader` → read rows as dictionaries (keys = headers).
* `csv.writer` → write rows to CSV.
* `csv.DictWriter` → write dictionaries (keys = headers).
* Always open files with `newline=''` when using `csv`.

#### Code-Snippet

```python
import csv

# 1. Writing CSV (rows)
rows = [
    ["name", "age", "city"],
    ["Alice", 25, "Delhi"],
    ["Bob", 30, "Mumbai"]
]

with open("people.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(rows)

# File 'people.csv' created with:
# name,age,city
# Alice,25,Delhi
# Bob,30,Mumbai


# 2. Reading CSV (rows)
with open("people.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

# Output:
# ['name', 'age', 'city']
# ['Alice', '25', 'Delhi']
# ['Bob', '30', 'Mumbai']


# 3. Reading CSV (as dicts)
with open("people.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row["name"], row["city"])

# Output:
# Alice Delhi
# Bob Mumbai


# 4. Writing CSV (dicts)
people = [
    {"name": "Charlie", "age": 28, "city": "Chennai"},
    {"name": "Diana", "age": 22, "city": "Hyderabad"}
]

with open("people_dict.csv", "w", newline="") as f:
    fieldnames = ["name", "age", "city"]
    writer = csv.DictWriter(f, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(people)

# File 'people_dict.csv' created with:
# name,age,city
# Charlie,28,Chennai
# Diana,22,Hyderabad
```

#### Key-Takeaway

* Use `reader`/`writer` for lists.
* Use `DictReader`/`DictWriter` for dictionaries with headers.
* Always open CSV files with `newline=''` to avoid blank lines.

# RegEx-Module

#### Important-Notes

* Import: `import re`
* **Functions**:

  * `match` → from start
  * `search` → first match
  * `findall` → all matches as list
  * `finditer` → iterator of match objects
  * `sub` → replace
  * `split` → split by regex
* **Character sets & rules**:

  * `.` → any char except newline
  * `\d` → digit, `\D` → non-digit
  * `\w` → word char (letters, digits, `_`), `\W` → non-word
  * `\s` → whitespace, `\S` → non-whitespace
  * `[abc]` → a or b or c
  * `[^abc]` → not a, b, c
  * `[0-9]` → any digit
  * `[a-zA-Z]` → any letter
* **Quantifiers**:

  * `+` = 1+ times
  * `*` = 0+ times
  * `?` = 0 or 1
  * `{n}` = exactly n
  * `{n,}` = n or more
  * `{n,m}` = between n and m

#### Code-Snippet

```python
import re

text = "My phone is 98765 and office is 12345"

# 1. Match from start
print(re.match(r"My", text).group())  
# Output: My

# 2. Search anywhere
print(re.search(r"\d+", text).group())  
# Output: 98765

# 3. Find all numbers
print(re.findall(r"\d+", text))  
# Output: ['98765', '12345']

# 4. Character sets
print(re.findall(r"[aeiou]", "hello world"))  
# Output: ['e', 'o', 'o']

print(re.findall(r"[^0-9]", "a1b2c3"))  
# Output: ['a', 'b', 'c']

# 5. Replace
print(re.sub(r"\d", "*", text))  
# Output: My phone is ***** and office is *****

# 6. Split by spaces
print(re.split(r"\s", text))  
# Output: ['My', 'phone', 'is', '98765', 'and', 'office', 'is', '12345']

# 7. Quantifiers
print(re.findall(r"a{2,3}", "caaabb aa aaaa"))  
# Output: ['aaa', 'aa', 'aaa']
```

#### Key-Takeaway

* `\d, \w, \s` cover most needs.
* Use `[ ]` for custom sets, `[^ ]` for negation.
* Quantifiers control repetition (`+`, `*`, `{n,m}`).

# Error-Handling

#### Important-Notes

* **Use `try`…`except` to catch errors.**
* **`else` block runs if no exception.**
* **`finally` always runs (cleanup).**
* **`raise` → manually raise exception.**
* **Custom exceptions → subclass `Exception`.**

#### Code-Snippet

```python
# 1. Basic try-except
try:
    x = 10 / 0
except ZeroDivisionError as e:
    print("Error:", e)
# Output: Error: division by zero

# 2. Multiple excepts
try:
    num = int("abc")
except ValueError:
    print("ValueError occurred")
except TypeError:
    print("TypeError occurred")
# Output: ValueError occurred

# 3. Else + Finally
try:
    print("Hello")
except:
    print("Something went wrong")
else:
    print("No errors!")
finally:
    print("Always runs")
# Output:
# Hello
# No errors!
# Always runs

# 4. Raise exception
def check_age(age):
    if age < 18:
        raise ValueError("Age must be 18+")
    return "Allowed"

print(check_age(20))  
# Output: Allowed

# 5. Custom Exception
class MyError(Exception):
    pass

try:
    raise MyError("Something custom went wrong")
except MyError as e:
    print("Caught:", e)
# Output: Caught: Something custom went wrong
```

#### Key-Takeaway

* **Use `try/except` for safe code execution.**
* **`else` runs if no error, `finally` always runs.**
* **`raise` + custom exceptions = explicit error handling.**

---
# f-Strings

#### Important-Notes

* Prefix string with `f` or `F`.
* Place expressions inside `{}`.
* Faster and cleaner than `format()` or `%`.
* Supports variables, expressions, and function calls.
* Format specifiers: `{value:.2f}`, `{value:>5}`, `{value:<5}`, etc.

#### Code-Snippet

```python
name = "Vamsi"
age = 31

# 1. Basic usage
print(f"My name is {name} and I am {age} years old")
# Output: My name is Vamsi and I am 31 years old

# 2. Expressions inside {}
print(f"Next year I will be {age + 1}")
# Output: Next year I will be 32

# 3. Function calls
print(f"Uppercase name: {name.upper()}")
# Output: Uppercase name: VAMSI

# 4. Numbers formatting
pi = 3.14159
print(f"Pi rounded: {pi:.2f}")
# Output: Pi rounded: 3.14

# 5. Alignment & width
num = 7
print(f"[{num:>5}]")  # right align
print(f"[{num:<5}]")  # left align
print(f"[{num:^5}]")  # center align
# Output:
# [    7]
# [7    ]
# [  7  ]

# 6. Debugging (Python 3.8+)
print(f"{age=}")
# Output: age=31
```

#### Key-Takeaway

* f-strings → concise & powerful formatting.
* Support **variables, expressions, functions, and formatting options** directly inside `{}`.

---
# Generators

#### Important-Notes

* Defined using `yield` instead of `return`.
* Produce values **one by one** → lazy evaluation.
* Save memory (don’t store entire sequence).
* Can be created with **generator functions** or **generator expressions**.

#### Code-Snippet

```python
# 1. Generator function with yield
def gen_nums():
    for i in range(3):
        yield i

g = gen_nums()
print(next(g))  # Output: 0
print(next(g))  # Output: 1
print(next(g))  # Output: 2
# print(next(g)) -> Raises StopIteration

# 2. Generator expression
squares = (x * x for x in range(4))
print(list(squares))
# Output: [0, 1, 4, 9]

# 3. Iterating generator
def countdown(n):
    while n > 0:
        yield n
        n -= 1

for num in countdown(3):
    print(num)
# Output:
# 3
# 2
# 1

# 4. Memory efficiency
import sys
nums_list = [x for x in range(1000)]
nums_gen = (x for x in range(1000))

print(sys.getsizeof(nums_list))  # Output: ~9000+
print(sys.getsizeof(nums_gen))   # Output: ~112
```

#### Key-Takeaway

* Use generators for **large datasets or infinite streams** → efficient in memory and performance.
