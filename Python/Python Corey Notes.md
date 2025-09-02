# Table-of-Contents

<!-- toc -->

- [1. Strings](#1-strings)
- [2. Integers and Floats](#2-integers-and-floats)
  * [Key Insights](#key-insights)
  * [Code Snippets](#code-snippets)
- [3. Lists, Tuples, and Sets](#3-lists-tuples-and-sets)
  * [Key Insights](#key-insights-1)
    + [Lists](#lists)
    + [Tuples](#tuples)
    + [Sets](#sets)
    + [Creating Empty Collections](#creating-empty-collections)
  * [Code Snippets](#code-snippets-1)
- [4. Dictionaries](#4-dictionaries)
    + [Key Interview Points](#key-interview-points)
- [5. Conditionals and Booleans](#5-conditionals-and-booleans)
      - [Key Insights](#key-insights-2)
      - [Code Snippets](#code-snippets-2)
- [6. Loops and Iterations](#6-loops-and-iterations)
    + [Key Insights](#key-insights-3)
    + [Code Snippets](#code-snippets-3)
- [7. Functions](#7-functions)
      - [Key Insights](#key-insights-4)
      - [Code Snippets](#code-snippets-4)
- [8. Import Modules and Exploring The Standard Library](#8-import-modules-and-exploring-the-standard-library)
- [9. Variable Scope (LEGB, Global/Nonlocal)](#9-variable-scope-legb-globalnonlocal)
- [10. Slicing Lists and Strings](#10-slicing-lists-and-strings)
- [11. Comprehensions](#11-comprehensions)
- [12. Sorting Lists, Tuples, and Objects](#12-sorting-lists-tuples-and-objects)
- [13. String Formatting](#13-string-formatting)
    + [Key Insights](#key-insights-5)
    + [Code Snippets](#code-snippets-5)
- [14. Datetime Module](#14-datetime-module)
      - [Key Concepts:](#key-concepts)
- [15. Context Managers](#15-context-managers)
- [16. OS Module](#16-os-module)
      - [Key Insights](#key-insights-6)
      - [Code Snippets](#code-snippets-6)
      - [Usage Tips](#usage-tips)
- [17. Closures](#17-closures)
    + [Key Insights](#key-insights-7)
    + [Python Example](#python-example)
    + [JavaScript Example](#javascript-example)
    + [Practical Python Example](#practical-python-example)
    + [Conclusion](#conclusion)
    + [Programming Terms: First-Class Functions](#programming-terms-first-class-functions)
      - [Key Insights:](#key-insights)
      - [Summary:](#summary)
  * [Programming Terms: Mutable vs Immutable](#programming-terms-mutable-vs-immutable)
    + [Key Insights](#key-insights-8)
    + [Examples and Explanation](#examples-and-explanation)
      - [Immutable Example](#immutable-example)
      - [Mutable Example](#mutable-example)
    + [Importance of Knowing the Difference](#importance-of-knowing-the-difference)
    + [Conclusion](#conclusion-1)
    + [Programming Terms: Memoization](#programming-terms-memoization)
- [18. Idempotence](#18-idempotence)
  * [Key Insights](#key-insights-9)
  * [HTTP Methods](#http-methods)
  * [Interview Preparation](#interview-preparation)
- [19. Generators](#19-generators)
    + [Key Insights](#key-insights-10)
    + [Code Snippets for Interviews](#code-snippets-for-interviews)
- [20. Decorators](#20-decorators)
  * [Key Insights](#key-insights-11)
    + [Introduction](#introduction)
    + [Pre-requisites](#pre-requisites)
    + [Example Recap: Closures](#example-recap-closures)
    + [Code Snippet: Closures Example](#code-snippet-closures-example)
    + [Understanding Decorators](#understanding-decorators)
    + [Basic Decorator Example](#basic-decorator-example)
    + [Code Snippet: Basic Decorator Example](#code-snippet-basic-decorator-example)
    + [Using Decorators with Arguments](#using-decorators-with-arguments)
    + [Code Snippet: Decorator with Arguments](#code-snippet-decorator-with-arguments)
    + [Real-world Usage](#real-world-usage)
  * [Summary](#summary)
- [21. Decorators With Arguments](#21-decorators-with-arguments)
      - [Key Insights:](#key-insights-1)
      - [Summary:](#summary-1)
- [22. Namedtuple](#22-namedtuple)
      - [Key Insights:](#key-insights-2)
      - [Code Snippets:](#code-snippets)
      - [Why Use Namedtuples?](#why-use-namedtuples)
      - [Conclusion:](#conclusion)

<!-- tocstop -->

---

# 1. Strings

**Key Insights:**

- **Introduction to Strings:**
  - Strings represent textual data in Python.
  - Strings can be created using single or double quotes.

- **Variable Assignment:**
  - Assign a string to a variable using `variable_name = "text"`.
  - Variables should be descriptive and follow naming conventions (e.g., `my_message`).

- **String Operations:**
  - Use single or double quotes to handle strings with embedded quotes.
  - Triple quotes (`"""` or `'''`) are used for multi-line strings.

- **String Length and Indexing:**
  - Use `len(variable)` to get the length of a string.
  - Access individual characters using indexing: `variable[index]`.

- **String Slicing:**
  - Slicing syntax: `variable[start:stop]` (stop index is exclusive).
  - Omitting `start` or `stop` assumes start of string or end of string respectively.

- **String Methods:**
  - **Convert Case:**
    ```python
    variable.lower()   # Converts to lowercase
    variable.upper()   # Converts to uppercase
    ```
  - **Count Occurrences:**
    ```python
    variable.count(substring)   # Counts occurrences of substring
    ```
  - **Find Index:**
    ```python
    variable.find(substring)   # Finds index of first occurrence of substring
    ```
  - **Replace Substring:**
    ```python
    variable.replace(old, new)   # Replaces occurrences of 'old' with 'new'
    ```

- **String Concatenation:**
  - Use `+` to concatenate strings:
    ```python
    greeting = "Hello"
    name = "Michael"
    message = greeting + ", " + name + "!"
    ```

- **Formatted Strings:**
  - Use `.format()` method:
    ```python
    message = "{}, {}!".format(greeting, name)
    ```
  - **f-strings (Python 3.6+):**
    ```python
    message = f"{greeting}, {name}!"
    ```

- **String Methods Overview:**
  - Use `dir(variable)` to list available methods.
  - Use `help(str)` or `help(str.method_name)` for method details.

**Code Snippets:**

```python
# Assigning strings to variables
message = "Hello, World!"

# Multi-line string
multiline_message = """This is a 
multi-line string."""

# String length and indexing
length = len(message)
first_char = message[0]
last_char = message[-1]

# String slicing
hello = message[0:5]
world = message[7:]

# String methods
lower_message = message.lower()
upper_message = message.upper()
count_l = message.count('l')
find_world = message.find('World')
replaced_message = message.replace('World', 'Universe')

# String concatenation
greeting = "Hello"
name = "Michael"
message = greeting + ", " + name + "!"

# Formatted strings
formatted_message = "{}, {}!".format(greeting, name)

# f-strings (Python 3.6+)
f_string_message = f"{greeting}, {name}!"
```
___

# 2. Integers and Floats 

## Key Insights

- **Integers and Floats:**
  - Integer: Whole number (e.g., `3`)
  - Float: Decimal number (e.g., `3.14`)
  - Use `type()` function to check data type.

- **Basic Arithmetic Operations:**
  - Addition (`+`): `3 + 2` equals `5`
  - Subtraction (`-`): `3 - 2` equals `1`
  - Multiplication (`*`): `3 * 2` equals `6`
  - Division (`/`): `3 / 2` equals `1.5` (Python 3 behavior)

- **Floor Division:**
  - Use `//` for floor division: `3 // 2` equals `1`

- **Exponentiation:**
  - Use `**` for power calculation: `3 ** 2` equals `9`

- **Modulo Operator:**
  - Use `%` to get the remainder: `3 % 2` equals `1`
  - Even or odd check: `number % 2 == 0` for even, `number % 2 == 1` for odd

- **Order of Operations:**
  - Use parentheses `()` to change the order: `3 * (2 + 1)` equals `9`

- **Incrementing Variables:**
  - Standard increment: `num = num + 1`
  - Shorthand increment: `num += 1`

- **Built-in Functions:**
  - Absolute value: `abs(-3)` equals `3`
  - Rounding: `round(3.75)` equals `4`
  - Rounding to specific decimal places: `round(3.75, 1)` equals `3.8`

- **Comparison Operators:**
  - Equality: `num1 == num2`
  - Inequality: `num1 != num2`
  - Greater than: `num1 > num2`
  - Less than: `num1 < num2`
  - Greater than or equal to: `num1 >= num2`
  - Less than or equal to: `num1 <= num2`

- **Casting Strings to Integers:**
  - Convert string to integer: `num1 = int('100')`

## Code Snippets

```python
# Integer and Float
num = 3
print(type(num))  # <class 'int'>

num = 3.14
print(type(num))  # <class 'float'>

# Arithmetic Operations
print(3 + 2)  # 5
print(3 - 2)  # 1
print(3 * 2)  # 6
print(3 / 2)  # 1.5

# Floor Division
print(3 // 2)  # 1

# Exponentiation
print(3 ** 2)  # 9

# Modulo Operator
print(3 % 2)  # 1

# Even or Odd Check
print(4 % 2 == 0)  # True (Even)
print(5 % 2 == 1)  # True (Odd)

# Order of Operations
print(3 * (2 + 1))  # 9

# Incrementing Variables
num = 1
num = num + 1
print(num)  # 2

num += 1
print(num)  # 2

# Built-in Functions
print(abs(-3))  # 3
print(round(3.75))  # 4
print(round(3.75, 1))  # 3.8

# Comparison Operators
num1 = 3
num2 = 2
print(num1 == num2)  # False
print(num1 != num2)  # True
print(num1 > num2)   # True
print(num1 < num2)   # False
print(num1 >= num2)  # True
print(num1 <= num2)  # False

# Casting Strings to Integers
num1 = int('100')
num2 = int('200')
print(num1 + num2)  # 300
```

These insights and code snippets will help you understand and work with numeric data in Python, which is essential for both basic programming tasks and technical interviews.

___

# 3. Lists, Tuples, and Sets

## Key Insights

### Lists
- **Creation and Basic Operations**
  - Create a list using square brackets: `courses = ["History", "Math", "Physics", "CompSci"]`
  - Print list: `print(courses)` 
  - Get length of list: `len(courses)`
  - Access elements by index: `courses[0]` (first element), `courses[-1]` (last element)
  - Slicing: `courses[0:2]` (first two elements)

- **Modifying Lists**
  - Append element: `courses.append("Art")`
  - Insert element at specific position: `courses.insert(0, "Art")`
  - Extend list with another list: `courses.extend(["Art", "Education"])`
  - Remove element: `courses.remove("Math")`
  - Pop element (last by default): `popped = courses.pop()`

- **Sorting and Reversing**
  - Reverse list: `courses.reverse()`
  - Sort list: `courses.sort()` (alphabetical for strings, ascending for numbers)
  - Sort in reverse order: `courses.sort(reverse=True)`
  - Get sorted list without altering original: `sorted_courses = sorted(courses)`

- **Built-in Functions**
  - Minimum: `min(courses)`
  - Maximum: `max(courses)`
  - Sum: `sum(numbers)`

- **Finding and Checking Elements**
  - Find index: `courses.index("CompSci")`
  - Check existence: `"Math" in courses`

- **Looping through List**
  - Simple loop: `for course in courses: print(course)`
  - Loop with index: `for index, course in enumerate(courses): print(index, course)`

- **Joining and Splitting**
  - Join list into string: `course_str = ", ".join(courses)`
  - Split string into list: `new_list = course_str.split(", ")`

### Tuples
- **Creation and Basic Operations**
  - Create tuple: `courses = ("History", "Math", "Physics", "CompSci")`
  - Tuples are immutable (cannot be modified after creation)

- **Difference from Lists**
  - Use tuples when you need a sequence of values that shouldn't change
  - Tuples have fewer methods due to immutability

### Sets
- **Creation and Basic Operations**
  - Create set: `courses = {"History", "Math", "Physics", "CompSci"}`
  - Sets are unordered and do not allow duplicates

- **Operations**
  - Add element: `courses.add("Art")`
  - Remove element: `courses.remove("Math")`
  - Membership test: `"Math" in courses`
  - Intersection: `common_courses = cs_courses.intersection(art_courses)`
  - Difference: `diff_courses = cs_courses.difference(art_courses)`
  - Union: `all_courses = cs_courses.union(art_courses)`

### Creating Empty Collections
- **Lists**: `empty_list = []` or `empty_list = list()`
- **Tuples**: `empty_tuple = ()` or `empty_tuple = tuple()`
- **Sets**: `empty_set = set()` (not `{}` which creates an empty dictionary)

## Code Snippets
```python
# List Operations
courses = ["History", "Math", "Physics", "CompSci"]
print(courses)
print(len(courses))
print(courses[0], courses[-1])

# Slicing
print(courses[0:2])

# Modifying Lists
courses.append("Art")
courses.insert(0, "Art")
courses.extend(["Art", "Education"])
courses.remove("Math")
popped = courses.pop()

# Sorting and Reversing
courses.reverse()
courses.sort()
courses.sort(reverse=True)
sorted_courses = sorted(courses)

# Built-in Functions
min_val = min(courses)
max_val = max(courses)
numbers = [1, 5, 4, 3]
total = sum(numbers)

# Finding and Checking Elements
index = courses.index("CompSci")
exists = "Math" in courses

# Looping through List
for course in courses:
    print(course)
for index, course in enumerate(courses):
    print(index, course)

# Joining and Splitting
course_str = ", ".join(courses)
new_list = course_str.split(", ")

# Tuples
courses = ("History", "Math", "Physics", "CompSci")

# Sets
courses = {"History", "Math", "Physics", "CompSci"}
courses.add("Art")
courses.remove("Math")
exists = "Math" in courses
common_courses = cs_courses.intersection(art_courses)
diff_courses = cs_courses.difference(art_courses)
all_courses = cs_courses.union(art_courses)

# Creating Empty Collections
empty_list = []
empty_tuple = ()
empty_set = set()
```
___
# 4. Dictionaries

- **Introduction to Dictionaries**
  - Dictionaries in Python store key-value pairs.
  - Keys are unique identifiers, and values are the data associated with those keys.
  - Comparable to hashmaps or associative arrays in other languages.

- **Creating a Dictionary**
  - Syntax: Use curly braces `{}` with key-value pairs separated by colons `:`.
  ```python
  student = {
      "name": "John",
      "age": 25,
      "courses": ["Math", "CompSci"]
  }
  ```

- **Accessing Values**
  - Access a value by its key using square brackets `[]`.
  ```python
  name = student["name"]  # Output: "John"
  courses = student["courses"]  # Output: ["Math", "CompSci"]
  ```
  - Using the `get` method to avoid errors if the key doesn't exist.
  ```python
  phone = student.get("phone", "Not Found")  # Output: "Not Found"
  ```

- **Adding and Updating Entries**
  - Add or update a key-value pair by assignment.
  ```python
  student["phone"] = "555-5555"
  student["name"] = "Jane"
  ```
  - Use the `update` method to update multiple entries at once.
  ```python
  student.update({"name": "Jane", "age": 26, "phone": "555-5555"})
  ```

- **Deleting Entries**
  - Use the `del` keyword to delete a key-value pair.
  ```python
  del student["age"]
  ```
  - Use the `pop` method to remove a key and return its value.
  ```python
  age = student.pop("age")  # Output: 25
  ```

- **Looping through Dictionary**
  - Loop through keys.
  ```python
  for key in student:
      print(key)
  ```
  - Loop through keys and values using the `items` method.
  ```python
  for key, value in student.items():
      print(f"{key}: {value}")
  ```

- **Additional Dictionary Methods**
  - Get the number of key-value pairs with `len`.
  ```python
  length = len(student)  # Output: 3
  ```
  - Get all keys using `keys`.
  ```python
  keys = student.keys()  # Output: dict_keys(['name', 'courses', 'phone'])
  ```
  - Get all values using `values`.
  ```python
  values = student.values()  # Output: dict_values(['Jane', ['Math', 'CompSci'], '555-5555'])
  ```

### Key Interview Points

- Understanding the syntax and structure of dictionaries in Python.
- Knowing how to access, add, update, and delete dictionary entries.
- Familiarity with common dictionary methods (`get`, `update`, `pop`, `keys`, `values`, `items`).
- Ability to iterate through dictionaries using loops.

___

# 5. Conditionals and Booleans

#### Key Insights

- **If Statement Basics**
  - `if` statements control which code blocks execute based on boolean conditions.
  - Syntax: `if condition:`
    ```python
    if True:
        print("Conditional was true")
    ```

- **Boolean Values**
  - True and False are the basic boolean values.
  - Conditional code only executes if the condition evaluates to True.

- **Comparisons**
  - Use `==` for equality, `!=` for inequality.
  - Other operators: `>`, `<`, `>=`, `<=`, `is` (checks if two variables point to the same object in memory).

- **Else Statement**
  - Executes an alternative block of code if the `if` condition is False.
    ```python
    language = "Python"
    if language == "Python":
        print("Language is Python")
    else:
        print("No match")
    ```

- **Elif Statement**
  - Used to check multiple conditions.
    ```python
    language = "Java"
    if language == "Python":
        print("Language is Python")
    elif language == "Java":
        print("Language is Java")
    else:
        print("No match")
    ```

- **Logical Operators**
  - `and`: Both conditions must be True.
  - `or`: At least one condition must be True.
  - `not`: Inverts the boolean value.
    ```python
    user = "admin"
    logged_in = True
    if user == "admin" and logged_in:
        print("Admin Page")
    else:
        print("Bad Creds")
    ```

- **Identity Operators**
  - `is`: Checks if two variables point to the same object.
    ```python
    a = [1, 2, 3]
    b = a
    print(a is b)  # True
    ```

- **Truthy and Falsy Values**
  - False, None, 0, empty sequences (e.g., `""`, `[]`, `{}`) are Falsy.
  - Everything else is Truthy.
    ```python
    condition = ""
    if condition:
        print("Evaluates to True")
    else:
        print("Evaluates to False")
    ```

#### Code Snippets

- **Basic If Statement**
  ```python
  if True:
      print("Conditional was true")
  ```

- **If-Else Statement**
  ```python
  language = "Python"
  if language == "Python":
      print("Language is Python")
  else:
      print("No match")
  ```

- **If-Elif-Else Statement**
  ```python
  language = "Java"
  if language == "Python":
      print("Language is Python")
  elif language == "Java":
      print("Language is Java")
  else:
      print("No match")
  ```

- **Logical Operators**
  ```python
  user = "admin"
  logged_in = True
  if user == "admin" and logged_in:
      print("Admin Page")
  else:
      print("Bad Creds")
  ```

- **Identity Operators**
  ```python
  a = [1, 2, 3]
  b = a
  print(a is b)  # True
  ```

- **Truthy and Falsy Values**
  ```python
  condition = ""
  if condition:
      print("Evaluates to True")
  else:
      print("Evaluates to False")
  ```
___
# 6. Loops and Iterations

### Key Insights

- **For Loops:**
  - Used to iterate over a sequence (list, tuple, string, etc.).
  - Syntax: `for variable in sequence`.
  - Example:
    ```python
    nums = [1, 2, 3, 4, 5]
    for num in nums:
        print(num)
    ```

- **Break Statement:**
  - Used to exit a loop prematurely when a condition is met.
  - Example:
    ```python
    nums = [1, 2, 3, 4, 5]
    for num in nums:
        if num == 3:
            print("Found:", num)
            break
        print(num)
    ```

- **Continue Statement:**
  - Skips the current iteration and moves to the next iteration of the loop.
  - Example:
    ```python
    nums = [1, 2, 3, 4, 5]
    for num in nums:
        if num == 3:
            print("Found:", num)
            continue
        print(num)
    ```

- **Nested Loops:**
  - A loop inside another loop, used to iterate over multi-dimensional data.
  - Example:
    ```python
    nums = [1, 2, 3]
    for num in nums:
        for letter in 'abc':
            print(num, letter)
    ```

- **Range Function:**
  - Generates a sequence of numbers, commonly used in for loops.
  - Syntax: `range(start, stop, step)`.
  - Examples:
    ```python
    for i in range(10):
        print(i)  # Prints 0 to 9

    for i in range(1, 11):
        print(i)  # Prints 1 to 10
    ```

- **While Loops:**
  - Repeatedly executes a block of code as long as a condition is true.
  - Syntax: `while condition`.
  - Example:
    ```python
    x = 0
    while x < 10:
        print(x)
        x += 1
    ```

- **Infinite Loops and Break:**
  - Infinite loops run forever unless stopped with a break statement.
  - Example:
    ```python
    x = 0
    while True:
        print(x)
        x += 1
        if x == 10:
            break
    ```

### Code Snippets

```python
# For Loop Example
nums = [1, 2, 3, 4, 5]
for num in nums:
    print(num)

# Break Statement Example
nums = [1, 2, 3, 4, 5]
for num in nums:
    if num == 3:
        print("Found:", num)
        break
    print(num)

# Continue Statement Example
nums = [1, 2, 3, 4, 5]
for num in nums:
    if num == 3:
        print("Found:", num)
        continue
    print(num)

# Nested Loops Example
nums = [1, 2, 3]
for num in nums:
    for letter in 'abc':
        print(num, letter)

# Range Function Examples
for i in range(10):
    print(i)  # Prints 0 to 9

for i in range(1, 11):
    print(i)  # Prints 1 to 10

# While Loop Example
x = 0
while x < 10:
    print(x)
    x += 1

# Infinite Loop with Break Example
x = 0
while True:
    print(x)
    x += 1
    if x == 10:
        break
```
___
# 7. Functions

#### Key Insights

- **Functions Overview**
  - Functions are reusable blocks of code that perform specific tasks.
  - Created using the `def` keyword.

- **Creating a Function**
  - Syntax: 
    ```python
    def function_name(parameters):
        # code block
    ```
  - Example:
    ```python
    def hello_func():
        print("Hello, Function!")
    ```

- **Calling a Function**
  - Execute a function by using its name followed by parentheses.
  - Example:
    ```python
    hello_func()
    ```

- **Function Parameters and Arguments**
  - Parameters are specified within the parentheses in the function definition.
  - Arguments are the actual values passed to the function.
  - Example:
    ```python
    def greet(greeting):
        return f"{greeting}, Function!"
    
    print(greet("Hi"))
    ```

- **Default Parameters**
  - Provide default values for parameters.
  - Example:
    ```python
    def greet(greeting="Hello"):
        return f"{greeting}, Function!"
    
    print(greet())
    print(greet("Hi"))
    ```

- **Return Statement**
  - Functions can return values using the `return` keyword.
  - Example:
    ```python
    def add(a, b):
        return a + b
    
    result = add(2, 3)
    print(result)
    ```

- **Function Scope**
  - Variables defined within a function are local to that function.
  - Example:
    ```python
    def greet():
        name = "Alice"
        print(f"Hello, {name}")
    
    greet()
    print(name)  # This will raise an error because 'name' is not defined outside the function.
    ```

- **Docstrings**
  - Use triple quotes to write docstrings that describe what the function does.
  - Example:
    ```python
    def is_leap(year):
        """
        Determine if the specified year is a leap year.
        """
        return year % 4 == 0 and (year % 100 != 0 or year % 400 == 0)
    ```

- ***args and **kwargs**
  - `*args` allows a function to accept any number of positional arguments.
  - `**kwargs` allows a function to accept any number of keyword arguments.
  - Example:
    ```python
    def student_info(*args, **kwargs):
        print(args)
        print(kwargs)
    
    student_info("Math", "Art", name="John", age=22)
    ```

#### Code Snippets

1. **Basic Function**
    ```python
    def hello_func():
        print("Hello, Function!")
    
    hello_func()
    ```

2. **Function with Parameters and Return Value**
    ```python
    def greet(greeting="Hello"):
        return f"{greeting}, Function!"
    
    print(greet())
    print(greet("Hi"))
    ```

3. **Using *args and **kwargs**
    ```python
    def student_info(*args, **kwargs):
        print(args)
        print(kwargs)
    
    student_info("Math", "Art", name="John", age=22)
    ```

4. **Leap Year Calculation**
    ```python
    def is_leap(year):
        """
        Determine if the specified year is a leap year.
        """
        return year % 4 == 0 and (year % 100 != 0 or year % 400 == 0)
    
    print(is_leap(2020))  # True
    print(is_leap(2019))  # False
    ```

5. **Days in Month Function**
    ```python
    month_days = [0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

    def days_in_month(year, month):
        if not 1 <= month <= 12:
            return "Invalid month"
        if month == 2 and is_leap(year):
            return 29
        return month_days[month]
    
    print(days_in_month(2020, 2))  # 29
    print(days_in_month(2019, 2))  # 28
    ```
___
# 8. Import Modules and Exploring The Standard Library

# 9. Variable Scope (LEGB, Global/Nonlocal)

# 10. Slicing Lists and Strings

# 11. Comprehensions

# 12. Sorting Lists, Tuples, and Objects

# 13. String Formatting

### Key Insights

- **Basic String Formatting**
    
    - String concatenation can be replaced with the `.format()` method for better readability and manageability.
        
    - Using placeholders `{}` in strings and passing values to `.format()` replaces these placeholders with the provided values.
        
    - Placeholders can be explicitly numbered for clarity and reuse.
        
        ```python
        name = "Alice"
        age = 30
        print("Name: {}, Age: {}".format(name, age))
        print("Name: {0}, Age: {1}".format(name, age))
        ```
        
- **Formatting with Dictionaries**
    
    - You can pass a dictionary to `.format()` and access its keys within placeholders.
        
        ```python
        person = {"name": "Alice", "age": 30}
        print("Name: {0[name]}, Age: {0[age]}".format(person))
        ```
        
- **Formatting with Lists**
    
    - Lists can be similarly accessed by their indices within placeholders.
        
        ```python
        values = ["Alice", 30]
        print("Name: {0[0]}, Age: {0[1]}".format(values))
        ```
        
- **Class Attributes Formatting**
    
    - Accessing class attributes using the dot notation in placeholders.
        
        ```python
        class Person:
            def __init__(self, name, age):
                self.name = name
                self.age = age
        
        p = Person("Alice", 30)
        print("Name: {0.name}, Age: {0.age}".format(p))
        ```
        
- **Keyword Arguments in Format**
    
    - Directly passing keyword arguments to `.format()` for readability.
        
        ```python
        print("Name: {name}, Age: {age}".format(name="Alice", age=30))
        ```
        
    - Unpacking dictionaries to pass as keyword arguments.
        
        ```python
        person = {"name": "Alice", "age": 30}
        print("Name: {name}, Age: {age}".format(**person))
        ```
        
- **Number Formatting**
    
    - Padding numbers with zeros using formatting options.
        
        ```python
        for i in range(1, 11):
            print("{:02}".format(i))
        ```
        
    - Formatting decimals to specific places.
        
        ```python
        pi = 3.141592653589793
        print("Pi to 2 decimal places: {:.2f}".format(pi))
        ```
        
    - Adding commas as thousand separators.
        
        ```python
        large_number = 1234567890
        print("Formatted number: {:,}".format(large_number))
        ```
        
- **Date Formatting**
    
    - Using the `datetime` module for advanced date formatting.
        
        ```python
        from datetime import datetime
        date = datetime(2016, 9, 24)
        print("Formatted date: {:%B %d, %Y}".format(date))
        ```
        
    - Combining multiple date format options.
        
        ```python
        print("Detailed date: {:%B %d, %Y fell on a %A and was the %j day of the year}".format(date))
        ```
        

### Code Snippets

1. **String Formatting with Placeholders:**
    
    ```python
    name = "Alice"
    age = 30
    print("Name: {}, Age: {}".format(name, age))
    ```
    
2. **Dictionary and List Access in Format:**
    
    ```python
    person = {"name": "Alice", "age": 30}
    print("Name: {0[name]}, Age: {0[age]}".format(person))
    
    values = ["Alice", 30]
    print("Name: {0[0]}, Age: {0[1]}".format(values))
    ```
    
3. **Class Attributes Formatting:**
    
    ```python
    class Person:
        def __init__(self, name, age):
            self.name = name
            self.age = age
    
    p = Person("Alice", 30)
    print("Name: {0.name}, Age: {0.age}".format(p))
    ```
    
4. **Number and Decimal Formatting:**
    
    ```python
    for i in range(1, 11):
        print("{:02}".format(i))
    
    pi = 3.141592653589793
    print("Pi to 2 decimal places: {:.2f}".format(pi))
    ```
    
5. **Date Formatting:**
    
    ```python
    from datetime import datetime
    date = datetime(2016, 9, 24)
    print("Formatted date: {:%B %d, %Y}".format(date))
    
    print("Detailed date: {:%B %d, %Y fell on a %A and was the %j day of the year}".format(date))
    ```

# 14. Datetime Module

#### Key Concepts:

- **Importing the Module**: 
  - `import datetime`
  
- **Naive vs. Aware DateTimes**:
  - **Naive DateTimes**: No timezone information, easier to work with if you don't need that level of detail.
  - **Aware DateTimes**: Include timezone info and daylight savings time details.
  
- **Creating Dates**:
  - Using `datetime.date`: 
    ```python
    import datetime
    d = datetime.date(2016, 7, 24)
    print(d)
    ```
  - Getting today’s date:
    ```python
    today = datetime.date.today()
    print(today)
    ```

- **Extracting Date Components**:
  - Year, Month, Day:
    ```python
    year = today.year
    month = today.month
    day = today.day
    ```

- **Day of the Week**:
  - Weekday (Monday is 0, Sunday is 6):
    ```python
    weekday = today.weekday()
    ```
  - ISO Weekday (Monday is 1, Sunday is 7):
    ```python
    iso_weekday = today.isoweekday()
    ```

- **Timedeltas**:
  - Creating a timedelta:
    ```python
    delta = datetime.timedelta(days=7)
    ```
  - Adding/Subtracting Timedeltas:
    ```python
    next_week = today + delta
    last_week = today - delta
    ```

- **Datetime Objects**:
  - Combining date and time:
    ```python
    dt = datetime.datetime(2016, 7, 26, 12, 30, 45)
    print(dt)
    ```
  - Extracting Components:
    ```python
    date_part = dt.date()
    time_part = dt.time()
    hour = dt.hour
    minute = dt.minute
    ```

- **Current DateTime**:
  - Current local date and time:
    ```python
    now = datetime.datetime.now()
    ```

- **Using Time Zones with `pytz`**:
  - Installing `pytz`:
    ```sh
    pip install pytz
    ```
  - Creating timezone aware datetime:
    ```python
    import pytz
    utc_dt = datetime.datetime(2016, 7, 26, 12, 30, 45, tzinfo=pytz.UTC)
    print(utc_dt)
    ```
  - Converting to different timezone:
    ```python
    mt = pytz.timezone('US/Mountain')
    mt_dt = utc_dt.astimezone(mt)
    print(mt_dt)
    ```

- **Formatting DateTime**:
  - ISO format:
    ```python
    iso_format = dt.isoformat()
    ```
  - Custom format using `strftime`:
    ```python
    formatted = dt.strftime('%B %d, %Y')
    print(formatted)
    ```

- **Parsing Strings to DateTime**:
  - Using `strptime`:
    ```python
    dt_str = 'July 26, 2016'
    dt = datetime.datetime.strptime(dt_str, '%B %d, %Y')
    print(dt)
    ```
___
# 15. Context Managers

**Key Insights:**

- **Introduction to Context Managers:**
  - Context managers help manage resources in Python, automating setup and teardown processes.
  - Commonly used with file handling, but applicable to databases, locks, etc.

- **Using Context Managers with File Handling:**
  - Recommended way to handle files: use `with` statement for automatic resource management.
  - Example:
    ```python
    with open('file.txt', 'w') as f:
        f.write('Hello, World!')
    # File is automatically closed after the block
    ```

- **Creating Custom Context Managers with Classes:**
  - Define `__enter__` and `__exit__` methods in a class to handle setup and teardown.
  - Example:
    ```python
    class OpenFile:
        def __init__(self, file_name, mode):
            self.file_name = file_name
            self.mode = mode
        
        def __enter__(self):
            self.file = open(self.file_name, self.mode)
            return self.file
        
        def __exit__(self, exc_type, exc_val, exc_tb):
            self.file.close()

    with OpenFile('file.txt', 'w') as f:
        f.write('Hello, World!')
    ```

- **Creating Custom Context Managers with Functions:**
  - Use the `contextlib` module and the `@contextmanager` decorator.
  - Example:
    ```python
    from contextlib import contextmanager
    
    @contextmanager
    def open_file(file_name, mode):
        file = open(file_name, mode)
        try:
            yield file
        finally:
            file.close()
    
    with open_file('file.txt', 'w') as f:
        f.write('Hello, World!')
    ```

- **Practical Example - Directory Management:**
  - Context manager to handle changing directories.
  - Example:
    ```python
    from contextlib import contextmanager
    import os
    
    @contextmanager
    def change_dir(destination):
        current_dir = os.getcwd()
        os.chdir(destination)
        try:
            yield
        finally:
            os.chdir(current_dir)
    
    with change_dir('sample_directory'):
        # Do something in sample_directory
        print(os.listdir('.'))
    ```

- **Handling Errors with Context Managers:**
  - Use `try` and `finally` blocks to ensure resources are always released, even if an error occurs.
  - Example integrated in function-based context manager.

**Summary:**
- Context managers provide a clean and efficient way to manage resources in Python.
- They automate setup and teardown processes, reducing the chance of resource leaks.
- Custom context managers can be created using either classes or functions with the `@contextmanager` decorator.
- Practical examples include file handling and directory management, demonstrating the flexibility and utility of context managers.
# 16. OS Module

#### Key Insights
- **OS Module Introduction**
  - Allows interaction with the underlying operating system.
  - Can navigate the file system, get file information, change environment variables, move files, etc.
  
- **Importing and Exploring the OS Module**
  - Import with `import os`.
  - Use `dir(os)` to list all attributes and methods.

- **Current Working Directory**
  - Get the current directory with `os.getcwd()`.
  ```python
  import os
  print(os.getcwd())  # Outputs the current working directory
  ```

- **Changing Directory**
  - Change the current directory with `os.chdir(path)`.
  ```python
  os.chdir('/path/to/directory')
  print(os.getcwd())  # Outputs the new current working directory
  ```

- **Listing Files and Directories**
  - List files and folders in the current directory with `os.listdir()`.
  ```python
  print(os.listdir())  # Lists all files and folders in the current directory
  ```

- **Creating Directories**
  - Create a single directory with `os.mkdir(path)`.
  - Create intermediate directories with `os.makedirs(path)`.
  ```python
  os.mkdir('new_folder')
  os.makedirs('new_folder/sub_folder')
  ```

- **Deleting Directories**
  - Remove a single directory with `os.rmdir(path)`.
  - Remove directories recursively with `os.removedirs(path)`.
  ```python
  os.rmdir('new_folder')
  os.removedirs('new_folder/sub_folder')
  ```

- **Renaming Files or Directories**
  - Rename using `os.rename(old_name, new_name)`.
  ```python
  os.rename('old_file.txt', 'new_file.txt')
  ```

- **Getting File Information**
  - Get file status with `os.stat(path)`.
  ```python
  file_info = os.stat('new_file.txt')
  print(file_info.st_size)  # Outputs file size in bytes
  print(file_info.st_mtime)  # Outputs last modification time
  ```

- **Traversing Directory Tree**
  - Use `os.walk(path)` to traverse directories.
  ```python
  for dirpath, dirnames, filenames in os.walk('/path/to/start'):
      print('Directory:', dirpath)
      print('Subdirectories:', dirnames)
      print('Files:', filenames)
  ```

- **Environment Variables**
  - Access environment variables with `os.environ`.
  ```python
  home_directory = os.environ.get('HOME')
  print(home_directory)  # Outputs the home directory path
  ```

- **Path Manipulation**
  - Join paths with `os.path.join()`.
  - Get base name and directory name with `os.path.basename()` and `os.path.dirname()`.
  - Check existence with `os.path.exists()`.
  - Check if directory or file with `os.path.isdir()` and `os.path.isfile()`.
  ```python
  file_path = os.path.join(home_directory, 'new_file.txt')
  print(os.path.basename(file_path))  # Outputs 'new_file.txt'
  print(os.path.dirname(file_path))  # Outputs the directory path
  print(os.path.exists(file_path))  # Checks if path exists
  print(os.path.isdir(file_path))  # Checks if it's a directory
  print(os.path.isfile(file_path))  # Checks if it's a file
  ```

#### Code Snippets

```python
import os
from datetime import datetime

# Get current working directory
print("Current Directory:", os.getcwd())

# Change directory
os.chdir('/path/to/new_directory')
print("Changed Directory:", os.getcwd())

# List files and directories
print("List Directory:", os.listdir())

# Create directories
os.mkdir('new_folder')
os.makedirs('new_folder/sub_folder')

# Delete directories
os.rmdir('new_folder')
os.removedirs('new_folder/sub_folder')

# Rename file
os.rename('old_file.txt', 'new_file.txt')

# Get file information
file_info = os.stat('new_file.txt')
print("File Size:", file_info.st_size)
print("Last Modified:", datetime.fromtimestamp(file_info.st_mtime))

# Traverse directory tree
for dirpath, dirnames, filenames in os.walk('/path/to/start'):
    print('Directory:', dirpath)
    print('Subdirectories:', dirnames)
    print('Files:', filenames)

# Get environment variable
home_directory = os.environ.get('HOME')
print("Home Directory:", home_directory)

# Path manipulation
file_path = os.path.join(home_directory, 'new_file.txt')
print("File Path:", file_path)
print("Base Name:", os.path.basename(file_path))
print("Directory Name:", os.path.dirname(file_path))
print("Path Exists:", os.path.exists(file_path))
print("Is Directory:", os.path.isdir(file_path))
print("Is File:", os.path.isfile(file_path))
```

#### Usage Tips
- Use `os.getcwd()` and `os.chdir()` to manage current working directory.
- Employ `os.listdir()` for listing directory contents.
- Create and delete directories with `os.mkdir()`/`os.makedirs()` and `os.rmdir()`/`os.removedirs()`.
- Rename files and directories using `os.rename()`.
- Retrieve file metadata with `os.stat()`.
- Traverse directories with `os.walk()`.
- Manage environment variables with `os.environ`.
- Use `os.path` for robust path manipulations.
___

# 17. Closures

### Key Insights

- **Closure Definition**:
  - A closure is a record storing a function together with an environment.
  - It maps each free variable of the function with the value or storage location to which the name was bound when the closure was created.
  - Closures allow functions to access captured variables through the closure's reference, even when invoked outside their scope.

### Python Example

- **Outer Function with Inner Function**:
  - Outer function defines a variable.
  - Inner function uses that variable, making it a free variable.
  - The inner function is returned, preserving the outer function's scope.
  
  ```python
  def outer_function():
      message = "Hi"
      def inner_function():
          print(message)  # Free variable
      return inner_function

  my_func = outer_function()
  my_func()  # Outputs: Hi
  ```

- **Returning Inner Function Without Executing**:
  - Outer function returns inner function without executing it.
  - The returned function can be executed later, preserving access to the outer function's scope.

  ```python
  def outer_function():
      message = "Hi"
      def inner_function():
          print(message)
      return inner_function

  my_func = outer_function()
  my_func()  # Outputs: Hi
  ```

- **Parameterized Outer Function**:
  - Outer function takes a parameter and sets the free variable.
  - Multiple instances of the inner function can be created with different scopes.

  ```python
  def outer_function(msg):
      message = msg
      def inner_function():
          print(message)
      return inner_function

  hi_func = outer_function("Hi")
  hello_func = outer_function("Hello")
  hi_func()  # Outputs: Hi
  hello_func()  # Outputs: Hello
  ```

### JavaScript Example

- **Function with Nested Function**:
  - Outer function takes a parameter.
  - Inner function uses the outer function's parameter as a free variable.
  - Inner function is returned, preserving access to the outer function's scope.

  ```javascript
  function htmlTag(tag) {
      function wrapText(msg) {
          console.log(`<${tag}>${msg}</${tag}>`);
      }
      return wrapText;
  }

  const printH1 = htmlTag('h1');
  printH1('Hello World');  // Outputs: <h1>Hello World</h1>
  ```

- **Parameterized Inner Function**:
  - Inner function takes its own parameter.
  - Both outer and inner function parameters are used within the inner function.

  ```javascript
  function htmlTag(tag) {
      return function wrapText(msg) {
          console.log(`<${tag}>${msg}</${tag}>`);
      };
  }

  const printH1 = htmlTag('h1');
  const printP = htmlTag('p');
  printH1('Hello World');  // Outputs: <h1>Hello World</h1>
  printP('Hello World');  // Outputs: <p>Hello World</p>
  ```

### Practical Python Example

- **Logger Example with Closures**:
  - Outer function takes another function as a parameter.
  - Inner function logs details and executes the passed function.
  - Useful for logging, but better implemented with Python decorators.

  ```python
  def logger(func):
      def log_func(*args):
          print(f'Running {func.__name__} with arguments {args}')
          result = func(*args)
          print(result)
          return result
      return log_func

  def add(x, y):
      return x + y

  def subtract(x, y):
      return x - y

  add_logger = logger(add)
  sub_logger = logger(subtract)

  add_logger(3, 3)  # Outputs: Running add with arguments (3, 3) 6
  sub_logger(10, 5)  # Outputs: Running subtract with arguments (10, 5) 5
  ```

### Conclusion

- **Closure Recap**:
  - Closures are inner functions that remember and have access to variables in their scope, even after the outer function has finished executing.
  - They are useful for various applications, such as function factories, and can replace more complex constructs like decorators in simpler scenarios.
  - Understanding closures in different languages helps solidify the concept beyond just syntax.

___
### Programming Terms: First-Class Functions

#### Key Insights:
- **First-Class Functions Definition**:
  - A programming language is said to have first-class functions if it treats functions as first-class citizens.
  - First-class citizens in programming are entities that support all operations generally available to other entities, such as being passed as arguments, returned from functions, and assigned to variables.

- **Assigning Functions to Variables**:
  - You can assign a function to a variable without executing it.
  - Example in Python:
    ```python
    def square(x):
        return x * x

    f = square  # Assigning function to variable
    print(f(5))  # Output: 25
    ```
  - Example in JavaScript:
    ```javascript
    function square(x) {
        return x * x;
    }

    let f = square;  // Assigning function to variable
    console.log(f(5));  // Output: 25
    ```

- **Passing Functions as Arguments (Higher-Order Functions)**:
  - A higher-order function is a function that accepts other functions as arguments or returns a function as a result.
  - Example using a custom map function in Python:
    ```python
    def my_map(func, arr):
        result = []
        for item in arr:
            result.append(func(item))
        return result

    def square(x):
        return x * x

    squares = my_map(square, [1, 2, 3, 4, 5])
    print(squares)  # Output: [1, 4, 9, 16, 25]
    ```
  - Example in JavaScript:
    ```javascript
    function myMap(func, arr) {
        let result = [];
        for (let i = 0; i < arr.length; i++) {
            result.push(func(arr[i]));
        }
        return result;
    }

    function square(x) {
        return x * x;
    }

    let squares = myMap(square, [1, 2, 3, 4, 5]);
    console.log(squares);  // Output: [1, 4, 9, 16, 25]
    ```

- **Returning Functions from Functions**:
  - Functions can return other functions, creating closures.
  - Example of a logger function in Python:
    ```python
    def logger(msg):
        def log_message():
            print(f"Log: {msg}")
        return log_message

    log_hi = logger("Hi")
    log_hi()  # Output: Log: Hi
    ```
  - Example in JavaScript:
    ```javascript
    function logger(msg) {
        function logMessage() {
            console.log(`Log: ${msg}`);
        }
        return logMessage;
    }

    let logHi = logger("Hi");
    logHi();  // Output: Log: Hi
    ```

- **Use Case: Custom HTML Tag Function**:
  - Demonstrates returning a function from another function.
  - Example in Python:
    ```python
    def html_tag(tag):
        def wrap_text(msg):
            print(f"<{tag}>{msg}</{tag}>")
        return wrap_text

    print_h1 = html_tag("h1")
    print_h1("Test Headline")  # Output: <h1>Test Headline</h1>
    ```
  - Example in JavaScript:
    ```javascript
    function htmlTag(tag) {
        function wrapText(msg) {
            console.log(`<${tag}>${msg}</${tag}>`);
        }
        return wrapText;
    }

    let printH1 = htmlTag("h1");
    printH1("Test Headline");  // Output: <h1>Test Headline</h1>
    ```

#### Summary:
Understanding first-class functions is crucial as it lays the foundation for understanding higher-order functions, currying, closures, and other advanced programming concepts. This knowledge allows you to treat functions like any other variable, making your code more flexible and reusable.
___
## Programming Terms: Mutable vs Immutable

### Key Insights

- **Definition**
  - **Immutable Object**: An object whose state cannot be modified after it is created (e.g., strings in Python).
  - **Mutable Object**: An object whose state can be modified after it is created (e.g., lists in Python).

### Examples and Explanation

#### Immutable Example
- **String in Python**
  ```python
  a = "Corey"
  print(a)  # Output: Corey
  a = "John"
  print(a)  # Output: John
  ```
  - Reassigning `a` to a new value creates a new string object.
  - Memory addresses before and after reassignment differ, indicating a new object is created.

- **Memory Address Check**
  ```python
  a = "Corey"
  print(id(a))  # Memory address of "Corey"
  a = "John"
  print(id(a))  # Memory address of "John"
  ```

- **Attempt to Modify String**
  ```python
  a = "Corey"
  a[0] = "C"  # Raises TypeError: 'str' object does not support item assignment
  ```

#### Mutable Example
- **List in Python**
  ```python
  a = [1, 2, 3, 4, 5]
  print(a)  # Output: [1, 2, 3, 4, 5]
  a[0] = 6
  print(a)  # Output: [6, 2, 3, 4, 5]
  ```
  - Modifying the list does not create a new object; the memory address remains the same.

- **Memory Address Check**
  ```python
  a = [1, 2, 3, 4, 5]
  print(id(a))  # Memory address of list
  a[0] = 6
  print(id(a))  # Same memory address after modification
  ```

### Importance of Knowing the Difference

- **Avoiding Errors**
  - Understanding mutable and immutable objects helps avoid errors related to unexpected modifications.
  
- **Memory Efficiency**
  - Immutable objects can lead to performance issues due to creation of multiple objects in memory.
  - Example: String concatenation in loops.
    ```python
    employees = ["John", "Jane", "Doe"]
    output = "<ul>"
    for employee in employees:
        output += f"<li>{employee}</li>"
    output += "</ul>"
    print(output)
    ```
    - This approach creates a new string object in each iteration, leading to performance overhead with large lists.

- **Alternative for Performance**
  - Use mutable objects like lists for efficient memory management in performance-critical applications.
    ```python
    employees = ["John", "Jane", "Doe"]
    output = ["<ul>"]
    for employee in employees:
        output.append(f"<li>{employee}</li>")
    output.append("</ul>")
    print("".join(output))
    ```

### Conclusion
- Understanding mutable and immutable objects is crucial for writing efficient and error-free code.
- It impacts both performance and memory usage, especially in large-scale applications.
- Examples from other languages (e.g., Java's `String` vs. `StringBuffer`) highlight the universal relevance of these concepts.
___
### Programming Terms: Memoization

**Key Insights:**
- **Definition:**
  - Memoization is an optimization technique used to speed up computer programs by storing results of expensive function calls and returning cached results when the same inputs occur again.

- **Purpose:**
  - Reduces redundant calculations by caching previously computed results.
  - Enhances performance by avoiding repetitive computations.

- **Concept:**
  - **Expensive Function:** A function that takes significant time/resources to compute.
  - **Cache:** Storage for previously computed results.

- **Implementation in Python:**
  - Use a dictionary to store computed results.
  - Check if the input is already in the cache before performing computations.
  - Store results in the cache after computation.

**Code Snippet:**
```python
import time

# Dictionary to cache results
EF_cache = {}

def expensive_func(num):
    if num in EF_cache:
        return EF_cache[num]
    
    # Simulate expensive computation
    print(f"Computing {num}")
    time.sleep(1)
    result = num * num
    
    # Store the result in the cache
    EF_cache[num] = result
    return result

# Test the function
print(expensive_func(4))  # Computed and cached
print(expensive_func(10)) # Computed and cached
print(expensive_func(4))  # Retrieved from cache
print(expensive_func(10)) # Retrieved from cache
```

**Explanation:**
1. **Function Definition:**
   - `expensive_func(num)` computes the square of `num` and caches the result.
   - It checks if `num` is in `EF_cache`. If yes, it returns the cached result.
   - If no, it computes the result, stores it in `EF_cache`, and then returns it.

2. **Output:**
   - The first call with `4` and `10` performs the computation and caches the results.
   - The subsequent calls with the same values retrieve the results from the cache, reducing computation time.

**Benefits:**
- Reduces computation time from four seconds to two seconds in the example.
- Particularly useful for functions with expensive calculations.

**Advanced Use:**
- Memoization can be automated using decorators in Python.
- Applicable across various programming languages.

**Conclusion:**
- Memoization is a valuable technique to optimize code performance.
- Essential for reducing redundancy in computationally intensive tasks.
- Useful to explore different techniques for implementing memoization in your preferred programming language.
___
# 18. Idempotence

## Key Insights

- **Definition**: Idempotence is a property of certain operations in mathematics and computer science that can be applied multiple times without changing the result beyond the initial application.
  - Example: `f(f(x)) = f(x)`

- **Function Example**: 
  - **Non-idempotent Function**: 
    ```python
    def add_10(num):
        return num + 10

    result = add_10(10)  # Output: 20
    result = add_10(result)  # Output: 30
    ```
    - Explanation: The result changes with each application, thus `add_10` is not idempotent.

  - **Idempotent Function**: 
    ```python
    abs_value = abs(-10)  # Output: 10
    abs_value = abs(abs_value)  # Output: 10
    ```
    - Explanation: The result remains the same regardless of repeated applications, thus `abs` is idempotent.

- **Assignment Example**: 
  ```python
  a = 10
  ```
  - Explanation: No matter how many times `a = 10` is executed, the result remains the same, making it an idempotent operation.

## HTTP Methods

- **Idempotent Methods**:
  - **GET**: 
    - Retrieves the same resource each time without changing any data.
  - **PUT**:
    - Updates a resource to the same state repeatedly without further changes.
  - **DELETE**:
    - Removes a resource; subsequent delete operations have no additional effect since the resource is already deleted.

- **Non-idempotent Method**:
  - **POST**:
    - Used for creating resources or performing actions that change the server's state each time it is called (e.g., casting a vote).

## Interview Preparation

- **Idempotence Concept**: 
  - Idempotence refers to an operation that produces the same result if executed multiple times.
  - In programming, idempotent operations help maintain consistency and predictability in system behavior.
  
- **HTTP Methods**:
  - Be familiar with which HTTP methods are idempotent (GET, PUT, DELETE) and which are not (POST).
  - Understand how idempotence applies to resource manipulation in RESTful APIs.

By understanding these concepts and examples, you can effectively discuss idempotence in technical interviews.### Programming Terms: Idempotence

**Key Insights:**
- **Definition:**
  - Idempotence refers to the property of certain operations in mathematics and computer science that can be applied multiple times without changing the result beyond the initial application.
  - Formally, if \( f(x) \) is idempotent, then \( f(f(x)) = f(x) \).

- **Example of Non-idempotent Function:**
  - A function `add_10` in Python:
    ```python
    def add_10(num):
        return num + 10

    result = add_10(10)  # Result is 20
    result = add_10(result)  # Result is 30
    ```
  - Since `add_10(10)` returns 20 and `add_10(20)` returns 30, this function is not idempotent.

- **Example of Idempotent Function:**
  - The built-in Python function `abs`:
    ```python
    result = abs(-10)  # Result is 10
    result = abs(result)  # Result is 10
    ```
  - `abs` function remains the same when applied multiple times.

- **Idempotent Statements:**
  - A simple assignment operation:
    ```python
    a = 10
    a = 10  # Running this multiple times will not change the value of 'a'
    ```
  - This statement is idempotent as it consistently assigns the same value to `a`.

- **HTTP Methods and Idempotence:**
  - **Idempotent Methods:**
    - **GET:** Fetches data without changing the state. Reloading the page gives the same result.
    - **PUT:** Updates a resource to the same value multiple times without changing the result.
    - **DELETE:** Removing a resource. Once deleted, further delete operations do not change the state (e.g., deleting an already deleted user).
  - **Non-idempotent Methods:**
    - **POST:** Often used for creating resources or performing actions that change the server state. Repeated calls can lead to different results (e.g., incrementing a vote count).

**Code Snippets:**
- **Non-idempotent Function Example:**
  ```python
  def add_10(num):
      return num + 10

  result = add_10(10)  # Result: 20
  result = add_10(result)  # Result: 30
  ```

- **Idempotent Function Example:**
  ```python
  result = abs(-10)  # Result: 10
  result = abs(result)  # Result: 10
  ```

- **Idempotent Assignment Statement:**
  ```python
  a = 10
  a = 10  # Idempotent: No change in value of 'a'
  ```

**Interview Insights:**
- Be prepared to explain the concept of idempotence with both theoretical and practical examples.
- Highlight the importance of idempotence in various HTTP methods and how it affects API design.
- Provide clear examples to differentiate between idempotent and non-idempotent operations.
# 19. Generators 

### Key Insights

- **Definition and Basics**:
  - **Function Example**: Demonstrates a function `square_numbers` that computes squares of numbers in a list.
  - **Traditional Approach**: Uses a list to store and return results.
  - **Generator Conversion**: Simplifies the function using `yield` instead of appending to a list, creating a generator.

- **Advantages of Generators**:
  - **Memory Efficiency**: Generators yield one item at a time, not storing the entire result in memory.
  - **Performance**: Generators have better performance with large data sets as they don't hold all values in memory.
  - **Code Readability**: More readable than list-based approaches, eliminating the need for intermediate lists.

- **Implementation Details**:
  - **Basic Generator Example**:
    ```python
    def square_numbers(nums):
        for num in nums:
            yield num * num
    ```
  - **Using `next()`**: Fetches the next value from the generator.
    ```python
    nums = square_numbers([1, 2, 3, 4, 5])
    print(next(nums))  # Output: 1
    print(next(nums))  # Output: 4
    ```

- **Generator Expressions**:
  - **List Comprehension vs. Generator**: Convert list comprehension to a generator by replacing square brackets with parentheses.
    ```python
    nums = (x * x for x in [1, 2, 3, 4, 5])
    print(list(nums))  # Output: [1, 4, 9, 16, 25]
    ```

- **Handling Exhaustion**:
  - **StopIteration Exception**: Raised when a generator has no more items to yield.
  - **For Loop Handling**: Automatically handles generator exhaustion without raising exceptions.
    ```python
    for num in nums:
        print(num)
    ```

- **Performance Comparison**:
  - **Memory Usage**: Generators use significantly less memory compared to lists.
  - **Timing Example**:
    ```python
    import time
    
    def list_function(num_people):
        result = []
        for _ in range(num_people):
            person = {
                'id': _,
                'name': 'Name',
                'major': 'Major'
            }
            result.append(person)
        return result
    
    def generator_function(num_people):
        for _ in range(num_people):
            person = {
                'id': _,
                'name': 'Name',
                'major': 'Major'
            }
            yield person
    
    start_time = time.time()
    people_list = list_function(1000000)
    print(f"List function took {time.time() - start_time} seconds")
    
    start_time = time.time()
    people_generator = generator_function(1000000)
    print(f"Generator function took {time.time() - start_time} seconds")
    ```

- **Converting Generators to Lists**:
  - **List Conversion**: Possible but negates performance benefits.
    ```python
    nums = (x * x for x in range(1000000))
    nums_list = list(nums)  # Converts generator to list
    ```

### Code Snippets for Interviews

- **Basic Generator**:
  ```python
  def square_numbers(nums):
      for num in nums:
          yield num * num
  ```

- **Using `next()` with Generators**:
  ```python
  nums = square_numbers([1, 2, 3, 4, 5])
  print(next(nums))  # Output: 1
  print(next(nums))  # Output: 4
  ```

- **Generator Expression**:
  ```python
  nums = (x * x for x in range(10))
  ```

- **Handling Exhaustion in a Loop**:
  ```python
  for num in nums:
      print(num)
  ```

- **Performance Comparison Example**:
  ```python
  import time
  
  def list_function(num_people):
      result = []
      for _ in range(num_people):
          person = {'id': _, 'name': 'Name', 'major': 'Major'}
          result.append(person)
      return result
  
  def generator_function(num_people):
      for _ in range(num_people):
          person = {'id': _, 'name': 'Name', 'major': 'Major'}
          yield person
  
  start_time = time.time()
  people_list = list_function(1000000)
  print(f"List function took {time.time() - start_time} seconds")
  
  start_time = time.time()
  people_generator = generator_function(1000000)
  print(f"Generator function took {time.time() - start_time} seconds")
  ```
  ___

#  20. Decorators 
## Key Insights

### Introduction
- Decorators are a slightly more advanced topic in Python.
- They allow you to dynamically alter the functionality of your functions.

### Pre-requisites
- Understanding of first-class functions and closures is necessary.
- First-class functions: Functions treated like any other object (can be passed as arguments, returned, and assigned to variables).
- Closures: Inner functions that remember and have access to variables local to the scope in which they were created.

### Example Recap: Closures
- **Outer Function**: Contains a local variable (`message`) and an inner function.
- **Inner Function**: Accesses and prints the `message` variable, which is a free variable (not defined within the inner function but accessible).
  
### Code Snippet: Closures Example
```python
def outer_function():
    message = "Hello, World!"
    
    def inner_function():
        print(message)
        
    return inner_function()

outer_function()
# Output: Hello, World!
```

### Understanding Decorators
- Decorators in Python are functions that modify the behavior of another function.
- They allow you to wrap another function to extend or alter its behavior.

### Basic Decorator Example
1. **Defining a Decorator**:
   - A decorator is a function that takes another function as an argument and returns a new function.
   - Example: A simple decorator that prints a message before and after a function execution.

### Code Snippet: Basic Decorator Example
```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Output:
# Something is happening before the function is called.
# Hello!
# Something is happening after the function is called.
```

### Using Decorators with Arguments
- Decorators can also be used with functions that accept arguments.
- Modify the wrapper function to accept any number of arguments and keyword arguments.

### Code Snippet: Decorator with Arguments
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before the function call")
        result = func(*args, **kwargs)
        print("After the function call")
        return result
    return wrapper

@my_decorator
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")
# Output:
# Before the function call
# Hello, Alice!
# After the function call
```

### Real-world Usage
- **Logging**: Automatically log function calls.
- **Authorization**: Check if a user is authorized to use an endpoint.
- **Caching**: Cache the results of expensive function calls.

## Summary
- Decorators enhance the functionality of your functions in a clean and readable way.
- They are widely used in Python frameworks, such as Flask and Django, for various purposes like route handling and middleware.
___

# 21. Decorators With Arguments

#### Key Insights:
- **Introduction to Decorators with Arguments**
  - Decorators are functions that modify the behavior of other functions.
  - This tutorial focuses on creating decorators that accept arguments.

- **Basic Decorator Recap**
  - A simple decorator function wraps another function to add pre- and post-execution code.
  - Example:
    ```python
    def my_decorator(func):
        def wrapper(*args, **kwargs):
            print("Before the function runs")
            result = func(*args, **kwargs)
            print("After the function runs")
            return result
        return wrapper
    ```

- **Decorator with Arguments**
  - To create a decorator that accepts arguments, an additional outer function is used.
  - This outer function accepts the argument and returns the actual decorator.
  - Example structure:
    ```python
    def decorator_with_args(arg):
        def actual_decorator(func):
            def wrapper(*args, **kwargs):
                print(f"Decorator argument: {arg}")
                result = func(*args, **kwargs)
                return result
            return wrapper
        return actual_decorator
    ```

- **Applying the Decorator with Arguments**
  - The outer function's argument can be used inside the wrapper function.
  - Example usage:
    ```python
    @decorator_with_args("Hello")
    def say_hello(name):
        print(f"Hello, {name}")
    
    say_hello("Alice")
    # Output:
    # Decorator argument: Hello
    # Hello, Alice
    ```

- **Real-World Example: Flask Route Decorators**
  - Flask uses decorators to define routes, passing the URL path as an argument.
  - Example:
    ```python
    from flask import Flask
    app = Flask(__name__)

    @app.route('/hello')
    def hello():
        return "Hello, World!"
    ```

- **Customizable Prefix Example**
  - A decorator that adds a customizable prefix to print statements.
  - Example:
    ```python
    def prefix_decorator(prefix):
        def decorator(func):
            def wrapper(*args, **kwargs):
                print(f"{prefix} Before function")
                result = func(*args, **kwargs)
                print(f"{prefix} After function")
                return result
            return wrapper
        return decorator

    @prefix_decorator("LOG:")
    def display_info(name, age):
        print(f"Name: {name}, Age: {age}")

    display_info("Alice", 30)
    # Output:
    # LOG: Before function
    # Name: Alice, Age: 30
    # LOG: After function
    ```

#### Summary:
- Understanding decorators with arguments involves creating an outer function to accept the argument.
- Decorators can enhance the functionality of existing functions by adding pre- and post-execution logic.
- This technique is useful in frameworks like Flask for routing and can be customized for various use cases.
___
# 22. Namedtuple 

#### Key Insights:

- **Introduction to Namedtuples**
  - Namedtuples are lightweight objects that behave like tuples but are more readable.
  - Useful for representing data where readability and immutability are important.

- **Comparison with Regular Tuples**
  - Regular tuples: `(55, 155, 255)`
  - Accessing elements by index: `color[0]` returns `55`
  - Lack of readability: future readers may not understand the meaning of the values.

- **Comparison with Dictionaries**
  - Dictionaries: `{'red': 55, 'green': 155, 'blue': 255}`
  - Accessing elements by key: `color['red']` returns `55`
  - More typing required; less concise for multiple instances.

- **Advantages of Namedtuples**
  - Combines the immutability of tuples and readability of dictionaries.
  - Easier to create multiple instances without repetitive typing.
  - Allows accessing elements by name using dot notation.

#### Code Snippets:

- **Creating a Namedtuple**
  ```python
  from collections import namedtuple

  Color = namedtuple('Color', ['red', 'green', 'blue'])
  ```

- **Using a Namedtuple**
  ```python
  # Creating an instance
  color = Color(55, 155, 255)

  # Accessing values
  print(color.red)   # Output: 55
  print(color[0])    # Output: 55

  # Creating another instance
  white = Color(255, 255, 255)
  print(white.blue)  # Output: 255
  ```

#### Why Use Namedtuples?
- **Readability**: More intuitive and easier to understand code.
- **Immutability**: Values cannot be changed, preserving data integrity.
- **Efficiency**: Less typing compared to dictionaries when creating multiple instances.

#### Conclusion:
Namedtuples are a valuable tool in Python for creating readable, immutable data structures with minimal overhead. They provide the best of both tuples and dictionaries, making your code cleaner and easier to maintain.
____