# Table-of-Contents

<!-- toc -->

- [8. Import Modules and Exploring The Standard Library](#8-import-modules-and-exploring-the-standard-library)
- [9. Variable Scope - Understanding the LEGB Rule and Global/Nonlocal Statements](#9-variable-scope---understanding-the-legb-rule-and-globalnonlocal-statements)
      - [Key Insights:](#key-insights)
      - [Examples:](#examples)
      - [Best Practices:](#best-practices)
- [10. Slicing Lists and Strings](#10-slicing-lists-and-strings)
  * [Key Insights](#key-insights)
  * [Code Snippets](#code-snippets)
    + [Slicing Lists](#slicing-lists)
    + [Slicing Strings](#slicing-strings)
- [11. Comprehensions](#11-comprehensions)
      - [Key Insights](#key-insights-1)
      - [Summary](#summary)
- [12. Sorting Lists, Tuples, and Objects](#12-sorting-lists-tuples-and-objects)
  * [Key Insights](#key-insights-2)
    + [Sorting Lists](#sorting-lists)
    + [Sorting Tuples](#sorting-tuples)
    + [Sorting Dictionaries](#sorting-dictionaries)
    + [Custom Sorting Criteria](#custom-sorting-criteria)
  * [Summary](#summary-1)
- [13. String Formatting - Advanced Operations for Dicts, Lists, Numbers, and Dates](#13-string-formatting---advanced-operations-for-dicts-lists-numbers-and-dates)
    + [Key Insights](#key-insights-3)
    + [Code Snippets](#code-snippets-1)

<!-- tocstop -->

---

# 8. Import Modules and Exploring The Standard Library

**Key Insights:**

- **Importing Custom Modules:**
  - Create a module (e.g., `my_module.py`) with functions and variables.
  - Import the module using `import my_module` if it's in the same directory.

- **Accessing Functions and Variables:**
  - Use `module_name.function_name()` to call a function.
  - Example:
    ```python
    import my_module
    index = my_module.find_index(courses, "Math")
    print(index)
    ```

- **Aliasing Modules:**
  - Simplify module name using `import module_name as alias`.
  - Example:
    ```python
    import my_module as mm
    index = mm.find_index(courses, "Math")
    ```

- **Importing Specific Functions/Variables:**
  - Import specific functions or variables using `from module_name import function_name`.
  - Example:
    ```python
    from my_module import find_index
    index = find_index(courses, "Math")
    ```

- **Importing Multiple Functions/Variables:**
  - Import multiple items using commas.
  - Example:
    ```python
    from my_module import find_index, test
    print(test)
    ```

- **Wildcard Imports (Not Recommended):**
  - Import all items using `from module_name import *`.
  - Example:
    ```python
    from my_module import *
    ```

- **Understanding `sys.path`:**
  - Python searches for modules in the directories listed in `sys.path`.
  - View `sys.path`:
    ```python
    import sys
    print(sys.path)
    ```

- **Adding Directories to `sys.path`:**
  - Manually add directories to `sys.path`.
  - Example:
    ```python
    import sys
    sys.path.append('/path/to/directory')
    ```

- **Setting Environment Variables for Module Paths:**
  - **Mac:** Add to `.bash_profile`:
    ```sh
    export PYTHONPATH="/path/to/directory"
    ```
  - **Windows:** Set environment variable in System Properties.

- **Using the Standard Library:**
  - **Random Module:**
    - Example:
      ```python
      import random
      random_course = random.choice(courses)
      print(random_course)
      ```
  - **Math Module:**
    - Example:
      ```python
      import math
      rads = math.radians(90)
      print(rads)
      print(math.sin(rads))
      ```
  - **Datetime Module:**
    - Example:
      ```python
      from datetime import datetime
      today = datetime.today()
      print(today)
      ```
  - **Calendar Module:**
    - Example:
      ```python
      import calendar
      print(calendar.isleap(2020))
      ```
  - **OS Module:**
    - Example:
      ```python
      import os
      print(os.getcwd())
      ```

- **Exploring the Standard Library:**
  - The standard library contains many useful modules for common tasks.
  - Example of viewing a module's file location:
    ```python
    import os
    print(os.__file__)
    ```

**Summary:**
- Importing and using custom modules and standard library modules in Python.
- Managing module imports, paths, and environment variables.
- Exploring the functionality provided by the standard library for various tasks.
___
# 9. Variable Scope - Understanding the LEGB Rule and Global/Nonlocal Statements

#### Key Insights:

- **Variable Scope**: Determines where variables can be accessed within a program.
- **LEGB Rule**: Stands for Local, Enclosing, Global, Built-in.
  - **Local**: Variables defined within a function.
  - **Enclosing**: Variables in the local scope of enclosing functions.
  - **Global**: Variables defined at the top level of a module or declared global using the `global` keyword.
  - **Built-in**: Names pre-assigned in Python.

#### Examples:

- **Global and Local Scope**:
  - Global variable:
    ```python
    x = "global x"
    ```
  - Local variable:
    ```python
    def test():
        y = "local y"
        print(y)
    test()  # Output: local y
    ```
  - Accessing global variable within a function:
    ```python
    def test():
        print(x)
    test()  # Output: global x
    ```

- **Overriding Global Variable within Function**:
  - Using `global` keyword:
    ```python
    def test():
        global x
        x = "local x"
    test()
    print(x)  # Output: local x
    ```

- **Function Arguments as Local Variables**:
  ```python
  def test(z):
      print(z)
  test("local z")  # Output: local z
  ```

- **Built-in Scope**:
  - Using built-in function `min`:
    ```python
    m = min([5, 1, 4, 2, 3])
    print(m)  # Output: 1
    ```
  - Importing and viewing built-ins:
    ```python
    import builtins
    print(dir(builtins))
    ```

- **Enclosing Scope**:
  - Nested functions example:
    ```python
    def outer():
        x = "outer x"
        def inner():
            print(x)
        inner()
        print(x)
    outer()
    ```
    - Output:
      ```
      outer x
      outer x
      ```

- **Nonlocal Keyword**:
  - Changing variable in enclosing scope:
    ```python
    def outer():
        x = "outer x"
        def inner():
            nonlocal x
            x = "inner x"
            print(x)
        inner()
        print(x)
    outer()
    ```
    - Output:
      ```
      inner x
      inner x
      ```

#### Best Practices:

- **Avoid Overusing `global`**: It can make code harder to understand and maintain.
- **Use Local Scope**: Makes code easier to understand and work with.
- **Nonlocal Usage**: Useful for changing state of closures and decorators.
___
# 10. Slicing Lists and Strings

## Key Insights

- **Slicing Basics**
  - Slicing extracts elements from lists and strings using `[start:end:step]`.
  - Positive and negative indexes can be used to reference elements.

- **Accessing Single Elements**
  - Positive index example: `my_list[5]` returns `5`.
  - Negative index example: `my_list[-1]` returns the last element.

- **Slicing Lists**
  - `my_list[start:end]`: Extracts elements from `start` to `end-1`.
    - Example: `my_list[0:5]` returns elements from index 0 to 4.
  - Leaving out `start` or `end` defaults to the beginning or end of the list.
    - Example: `my_list[5:]` returns elements from index 5 to the end.
  - Using negative indexes in slices.
    - Example: `my_list[3:-2]` returns elements from index 3 to the second-to-last element.

- **Step Value in Slices**
  - `my_list[start:end:step]`: Extracts elements with a specified step.
    - Example: `my_list[2:8:2]` returns every second element from index 2 to 7.
  - Negative step values reverse the direction.
    - Example: `my_list[::-1]` returns the list in reverse order.

- **Slicing Strings**
  - Same syntax as lists.
  - Example: `url[::-1]` returns the reversed string.
  - Extracting parts of a string using negative indexes.
    - Example: `url[-4:]` extracts the top-level domain.
  - Combining slicing techniques.
    - Example: `url[7:-4]` removes the "http://" prefix and top-level domain.

## Code Snippets

### Slicing Lists

```python
# List initialization
my_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# Accessing elements using positive and negative indexes
print(my_list[5])    # Output: 5
print(my_list[-1])   # Output: 9

# Slicing with start and end indexes
print(my_list[0:5])  # Output: [0, 1, 2, 3, 4]
print(my_list[5:])   # Output: [5, 6, 7, 8, 9]
print(my_list[:5])   # Output: [0, 1, 2, 3, 4]
print(my_list[3:-2]) # Output: [3, 4, 5, 6, 7]

# Slicing with step value
print(my_list[2:8:2])  # Output: [2, 4, 6]
print(my_list[::-1])   # Output: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

### Slicing Strings

```python
# String initialization
url = "https://example.com"

# Reversing the string
print(url[::-1])  # Output: "moc.elpmaxe//:sptth"

# Extracting top-level domain
print(url[-4:])  # Output: ".com"

# Removing "http://" prefix
print(url[7:])  # Output: "example.com"

# Removing both "http://" prefix and top-level domain
print(url[7:-4])  # Output: "example"
```
___

# 11. Comprehensions

#### Key Insights

- **Introduction to List Comprehensions**
  - Easier and more readable way to create lists in Python.
  - Simplifies the syntax compared to traditional for loops.

- **Basic Example: Copying a List**
  - **For Loop Method:**
    ```python
    nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    my_nums = []
    for n in nums:
        my_nums.append(n)
    print(my_nums)  # Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    ```
  - **List Comprehension Method:**
    ```python
    my_nums = [n for n in nums]
    print(my_nums)  # Output: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    ```

- **Example: Squaring Numbers**
  - **For Loop Method:**
    ```python
    my_nums = []
    for n in nums:
        my_nums.append(n * n)
    print(my_nums)  # Output: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
    ```
  - **List Comprehension Method:**
    ```python
    my_nums = [n * n for n in nums]
    print(my_nums)  # Output: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
    ```

- **Filtering with List Comprehensions**
  - **For Loop Method:**
    ```python
    my_nums = []
    for n in nums:
        if n % 2 == 0:
            my_nums.append(n)
    print(my_nums)  # Output: [2, 4, 6, 8, 10]
    ```
  - **List Comprehension Method:**
    ```python
    my_nums = [n for n in nums if n % 2 == 0]
    print(my_nums)  # Output: [2, 4, 6, 8, 10]
    ```

- **Nested Loops in List Comprehensions**
  - **For Loop Method:**
    ```python
    my_list = []
    for letter in 'ABCD':
        for num in range(4):
            my_list.append((letter, num))
    print(my_list)  # Output: [('A', 0), ('A', 1), ('A', 2), ('A', 3), ('B', 0), ...]
    ```
  - **List Comprehension Method:**
    ```python
    my_list = [(letter, num) for letter in 'ABCD' for num in range(4)]
    print(my_list)  # Output: [('A', 0), ('A', 1), ('A', 2), ('A', 3), ('B', 0), ...]
    ```

- **Dictionary Comprehensions**
  - **For Loop Method:**
    ```python
    names = ['Bruce', 'Clark', 'Peter']
    heroes = ['Batman', 'Superman', 'Spiderman']
    my_dict = {}
    for name, hero in zip(names, heroes):
        my_dict[name] = hero
    print(my_dict)  # Output: {'Bruce': 'Batman', 'Clark': 'Superman', 'Peter': 'Spiderman'}
    ```
  - **Dictionary Comprehension Method:**
    ```python
    my_dict = {name: hero for name, hero in zip(names, heroes)}
    print(my_dict)  # Output: {'Bruce': 'Batman', 'Clark': 'Superman', 'Peter': 'Spiderman'}
    ```

- **Set Comprehensions**
  - **For Loop Method:**
    ```python
    nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 1, 2, 3]
    my_set = set()
    for n in nums:
        my_set.add(n)
    print(my_set)  # Output: {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    ```
  - **Set Comprehension Method:**
    ```python
    my_set = {n for n in nums}
    print(my_set)  # Output: {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    ```

- **Generator Expressions**
  - Similar to list comprehensions but uses parentheses and generates items one at a time.
  - **Generator Expression Example:**
    ```python
    nums = [1, 2, 3, 4, 5]
    my_gen = (n * n for n in nums)
    for val in my_gen:
        print(val)  # Output: 1, 4, 9, 16, 25
    ```

#### Summary
- List comprehensions provide a more readable and concise way to create lists, apply operations, and filter items.
- Dictionary and set comprehensions extend these benefits to dictionaries and sets.
- Generator expressions offer a memory-efficient way to handle large data sets.
- Comprehensions improve code readability and maintainability, making them a valuable tool in Python programming.

___
# 12. Sorting Lists, Tuples, and Objects

## Key Insights

### Sorting Lists
- **Using `sorted()` Function**
  - Sorts and returns a new sorted list.
  - Example:
    ```python
    li = [4, 2, 9, 1]
    sorli = sorted(li)
    print(sorli)  # Output: [1, 2, 4, 9]
    print(li)     # Output: [4, 2, 9, 1]
    ```

- **Using `.sort()` Method**
  - Sorts the list in place, returns `None`.
  - Example:
    ```python
    li = [4, 2, 9, 1]
    li.sort()
    print(li)  # Output: [1, 2, 4, 9]
    ```

- **Sorting in Descending Order**
  - Add `reverse=True` parameter.
  - Example:
    ```python
    sorli = sorted(li, reverse=True)
    print(sorli)  # Output: [9, 4, 2, 1]
    li.sort(reverse=True)
    print(li)     # Output: [9, 4, 2, 1]
    ```

### Sorting Tuples
- **Using `sorted()` Function**
  - Tuples don’t have a `.sort()` method.
  - Example:
    ```python
    tup = (4, 2, 9, 1)
    sorted_tup = sorted(tup)
    print(sorted_tup)  # Output: [1, 2, 4, 9]
    ```

### Sorting Dictionaries
- **Using `sorted()` Function**
  - Sorts dictionary keys by default.
  - Example:
    ```python
    di = {'c': 3, 'a': 1, 'b': 2}
    sorted_di = sorted(di)
    print(sorted_di)  # Output: ['a', 'b', 'c']
    ```

### Custom Sorting Criteria
- **Sorting by Absolute Value**
  - Use `key` parameter.
  - Example:
    ```python
    li = [3, -6, 1, -2, 5]
    sorli = sorted(li, key=abs)
    print(sorli)  # Output: [1, -2, 3, 5, -6]
    ```

- **Sorting Objects by Attribute**
  - Define a custom function or use lambda.
  - Example with custom function:
    ```python
    class Employee:
        def __init__(self, name, age, salary):
            self.name = name
            self.age = age
            self.salary = salary

        def __repr__(self):
            return f"({self.name}, {self.age}, {self.salary})"

    employees = [
        Employee("John", 25, 50000),
        Employee("Jane", 22, 60000),
        Employee("Doe", 28, 70000)
    ]

    def e_sort(emp):
        return emp.age

    sorted_employees = sorted(employees, key=e_sort)
    print(sorted_employees)  # Output: [(Jane, 22, 60000), (John, 25, 50000), (Doe, 28, 70000)]
    ```

- **Using `lambda` Function**
  - Example:
    ```python
    sorted_employees = sorted(employees, key=lambda e: e.salary)
    print(sorted_employees)  # Output: [(John, 25, 50000), (Jane, 22, 60000), (Doe, 28, 70000)]
    ```

- **Using `operator.attrgetter`**
  - Import and use for attribute-based sorting.
  - Example:
    ```python
    from operator import attrgetter

    sorted_employees = sorted(employees, key=attrgetter('name'))
    print(sorted_employees)  # Output: [(Doe, 28, 70000), (Jane, 22, 60000), (John, 25, 50000)]
    ```

## Summary
- Use `sorted()` for returning new sorted iterables and `.sort()` for in-place list sorting.
- Custom sorting can be achieved with the `key` parameter using functions, lambda expressions, or `operator.attrgetter`.
- Flexibility in sorting allows handling of complex objects and various data types effectively.
___
# 13. String Formatting - Advanced Operations for Dicts, Lists, Numbers, and Dates

### Key Insights

- **Basic String Formatting**
  - String concatenation can be replaced with the `.format()` method for better readability and manageability.
  - Using placeholders `{}` in strings and passing values to `.format()` replaces these placeholders with the provided values.
  - Placeholders can be explicitly numbered for clarity and reuse.

    ```python
    name = "Alice"
    age = 30
    print("Name: {}, Age: {}".format(name, age))
    print("Name: {0}, Age: {1}".format(name, age))
    ```

- **Formatting with Dictionaries**
  - You can pass a dictionary to `.format()` and access its keys within placeholders.

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
  - Directly passing keyword arguments to `.format()` for readability.

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
  - Using the `datetime` module for advanced date formatting.

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