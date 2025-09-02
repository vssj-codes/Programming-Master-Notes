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
- [5. Conditionals and Booleans - If, Else, and Elif Statements](#5-conditionals-and-booleans---if-else-and-elif-statements)
      - [Key Insights](#key-insights-2)
      - [Code Snippets](#code-snippets-2)
- [Python Tutorial for Beginners 7: Loops and Iterations - For/While Loops](#python-tutorial-for-beginners-7-loops-and-iterations---forwhile-loops)
    + [Key Insights](#key-insights-3)
    + [Code Snippets](#code-snippets-3)
- [Python Tutorial for Beginners 8: Functions](#python-tutorial-for-beginners-8-functions)
      - [Key Insights](#key-insights-4)
      - [Code Snippets](#code-snippets-4)

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

# 5. Conditionals and Booleans - If, Else, and Elif Statements

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
# Python Tutorial for Beginners 7: Loops and Iterations - For/While Loops

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
# Python Tutorial for Beginners 8: Functions

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
