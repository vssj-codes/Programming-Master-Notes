# Table-of-Contents

<!-- toc -->

- [OOP](#oop)
  * [OOP-1:-Classes-and-Instances](#oop-1-classes-and-instances)
      - [Key-Insights:](#key-insights)
      - [Example Code:](#example-code)
  * [OOP-2:-Class-Variables](#oop-2-class-variables)
    + [Key Insights](#key-insights)
    + [Code Snippets](#code-snippets)
      - [Employee Class with Class Variables](#employee-class-with-class-variables)
    + [Summary](#summary)
  * [OOP-3:-classmethods-and-staticmethods](#oop-3-classmethods-and-staticmethods)
    + [Regular Methods](#regular-methods)
    + [Class Methods](#class-methods)
    + [Static Methods](#static-methods)
    + [Key Points](#key-points)
    + [Key Insights](#key-insights-1)
    + [Code Snippets](#code-snippets-1)
      - [Class Method Example](#class-method-example)
      - [Alternative Constructor Using Class Method](#alternative-constructor-using-class-method)
      - [Static Method Example](#static-method-example)
    + [Additional Notes](#additional-notes)
  * [OOP-4:-Inheritance-Creating-Subclasses](#oop-4-inheritance-creating-subclasses)
    + [Key Insights](#key-insights-2)
    + [Code Snippets](#code-snippets-2)
  * [OOP-5: `str()`-vs-`repr()`](#oop-5-str-vs-repr)
      - [Important-Notes](#important-notes)
      - [Code-Snippet](#code-snippet)
      - [Key-Takeaway](#key-takeaway)
  * [OOP-6:-Special-(Magic/Dunder)-Methods](#oop-6-special-magicdunder-methods)
      - [Key Insights:](#key-insights)
  * [OOP-7: Property Decorators - Getters, Setters, and Deleters](#oop-7-property-decorators---getters-setters-and-deleters)

<!-- tocstop -->

---

# OOP
## OOP-1:-Classes-and-Instances

#### Key-Insights:

- **Purpose of Classes:**
  - Logical grouping of data (attributes) and functions (methods).
  - Facilitates reuse and expansion.

- **Basic Class Creation:**
  - Define a class using `class ClassName:`.
  - Use `pass` to create an empty class temporarily.

    ```python
    class Employee:
        pass
    ```

- **Class vs. Instance:**
  - A class is a blueprint.
  - Instances are unique objects created from the class.
  
    ```python
    emp1 = Employee()
    emp2 = Employee()
    ```

- **Instance Variables:**
  - Unique to each instance.
  - Can be manually assigned:

    ```python
    emp1.first = 'John'
    emp1.last = 'Doe'
    emp1.email = 'john.doe@company.com'
    ```

- **Initialization with `__init__`:**
  - Automates setting instance variables upon creation.
  - `self` represents the instance.

    ```python
    class Employee:
        def __init__(self, first, last, pay):
            self.first = first
            self.last = last
            self.pay = pay
            self.email = f"{first}.{last}@company.com"
    ```

- **Creating Instances with `__init__`:**
  - Pass parameters to initialize instance variables.

    ```python
    emp1 = Employee('John', 'Doe', 50000)
    emp2 = Employee('Jane', 'Doe', 60000)
    ```

- **Instance Methods:**
  - Define functions within a class.
  - Automatically take `self` as the first parameter.

    ```python
    class Employee:
        def __init__(self, first, last, pay):
            self.first = first
            self.last = last
            self.pay = pay
            self.email = f"{first}.{last}@company.com"

        def fullname(self):
            return f"{self.first} {self.last}"
    ```
- If we forget to write self in method signature `def fullname()`, we will get error saying fullname accepts 0 positional arguments, but 1 was given => instance is passed to instance method
- **Using Instance Methods:**
  - Call methods on instances.

    ```python
    print(emp1.fullname())
    print(emp2.fullname())
    ```

- **Class Methods vs. Instance Methods:**
  - Instance methods automatically receive the instance (`self`) as the first parameter.
  - Can also call instance methods using the class name, passing the instance explicitly.

    ```python
    print(Employee.fullname(emp1))
    ```

#### Example Code:

```python
class Employee:
    def __init__(self, first, last, pay):
        self.first = first
        self.last = last
        self.pay = pay
        self.email = f"{first}.{last}@company.com"

    def fullname(self):
        return f"{self.first} {self.last}"

# Creating instances
emp1 = Employee('John', 'Doe', 50000)
emp2 = Employee('Jane', 'Doe', 60000)

# Accessing attributes
print(emp1.email)
print(emp2.email)

# Using methods
print(emp1.fullname())
print(emp2.fullname())

# Calling method using class name
print(Employee.fullname(emp1))
```
___
## OOP-2:-Class-Variables

### Key Insights

- **Instance Variables**
  - Unique to each instance.
  - Defined within the `__init__` method using `self`.
  - Example: `self.name`, `self.email`, `self.pay`.

- **Class Variables**
  - Shared among all instances of a class.
  - Defined directly within the class, outside any methods.
  - Example: `raise_amount` in the Employee class.
  
- **Creating and Using Class Variables**
  - Define class variable: `raise_amount = 1.04`
  - Access via class: `Employee.raise_amount`
  - Access via instance: `self.raise_amount` or `employee1.raise_amount`
  
- **Applying Class Variables in Methods**
  - Method definition: `def apply_raise(self):`
  - Access class variable within method using `self.raise_amount` or `Employee.raise_amount`
  
- **Namespace and Attribute Lookup**
  - Instance checks its own namespace first for an attribute.
  - If not found, checks the class namespace.
  - Example: 
    ```python
    print(employee1.__dict__)  # Shows instance attributes
    print(Employee.__dict__)   # Shows class attributes, including class variables
    ```

- **Modifying Class Variables**
  - Changing via class affects all instances: `Employee.raise_amount = 1.05`
  - Changing via instance creates a new attribute for that instance: `employee1.raise_amount = 1.05` and we can see the attribute in instance namespace now.  
  
- **Class Variable Use Case: Tracking Instances**
  - Example: Keeping count of created instances.
  - Define counter: `num_of_employees = 0`
  - Increment in `__init__`: `Employee.num_of_employees += 1`
  
### Code Snippets

#### Employee Class with Class Variables
```python
class Employee:
    # Class variables
    raise_amount = 1.04
    num_of_employees = 0

    def __init__(self, name, email, pay):
        self.name = name
        self.email = email
        self.pay = pay
        Employee.num_of_employees += 1

    def apply_raise(self):
        self.pay = int(self.pay * self.raise_amount)

# Creating instances
emp1 = Employee('John Doe', 'john@example.com', 50000)
emp2 = Employee('Jane Doe', 'jane@example.com', 60000)

# Applying raise
print(emp1.pay)         # Output: 50000
emp1.apply_raise()
print(emp1.pay)         # Output: 52000

# Accessing class variable
print(Employee.raise_amount)  # Output: 1.04
print(emp1.raise_amount)      # Output: 1.04

# Modifying class variable via instance
emp1.raise_amount = 1.05
print(emp1.raise_amount)      # Output: 1.05
print(emp2.raise_amount)      # Output: 1.04

# Tracking number of employees
print(Employee.num_of_employees)  # Output: 2
```

### Summary
- **Instance Variables:** Unique per instance, set in `__init__`.
- **Class Variables:** Shared among all instances, set directly in class.
- **Namespace Lookup:** Instance attributes checked first, then class attributes.
- **Modifying Class Variables:** Class-level changes affect all instances; instance-level changes create new attributes.
- **Use Case:** Track number of instances with class variables.
- 
## OOP-3:-classmethods-and-staticmethods

### Regular Methods
- Regular methods in a class take the instance (`self`) as the first argument.
- Example:
  ```python
  class Employee:
      def __init__(self, first, last):
          self.first = first
          self.last = last
      def full_name(self):
          return f"{self.first} {self.last}"
  ```

### Class Methods
- Class methods take the class (`cls`) as the first argument.
- Use the `@classmethod` decorator.
- Can be used as alternative constructors.
- Example:
  ```python
  class Employee:
      raise_amount = 1.04
      
      @classmethod
      def set_raise_amount(cls, amount):
          cls.raise_amount = amount
      
      #used as constructor    
      @classmethod
      def from_string(cls, emp_str):
          first, last, pay = emp_str.split('-')
          return cls(first, last, float(pay))
  ```
- Usage:
  ```python
  Employee.set_raise_amount(1.05)
  emp1 = Employee.from_string('John-Doe-70000')
  ```

### Static Methods
- Static methods don't take the instance (`self`) or the class (`cls`) as the first argument.
- Use the `@staticmethod` decorator.
- Behave like regular functions but have a logical connection to the class.
- Example:
  ```python
  class Employee:
      @staticmethod
      def is_workday(day):
          return day.weekday() != 5 and day.weekday() != 6
  ```
- Usage:
  ```python
  import datetime
  my_date = datetime.date(2022, 11, 24)
  print(Employee.is_workday(my_date))
  ```

### Key Points
- **Regular Methods:** Operate on the instance of the class.
- **Class Methods:** Operate on the class itself, used for factory methods.
- **Static Methods:** Perform utility functions related to the class but don't operate on an instance or class variables.## Python OOP Tutorial 3: classmethods and staticmethods

### Key Insights

- **Instance Methods**
  - Automatically take the instance (`self`) as the first argument.
  - Used for operations that use instance variables.

- **Class Methods**
  - Automatically take the class (`cls`) as the first argument.
  - Created using the `@classmethod` decorator.
  - Can be used to change class variables.
  - Can be used as alternative constructors.

- **Static Methods**
  - Do not take any automatic first argument (neither `self` nor `cls`).
  - Created using the `@staticmethod` decorator.
  - Behave like regular functions but are logically related to the class.
  - Used for utility functions that don’t modify class or instance data.

### Code Snippets

#### Class Method Example
```python
class Employee:
    raise_amount = 1.04  # Class variable
    
    @classmethod
    def set_raise_amount(cls, amount):
        cls.raise_amount = amount

# Usage
Employee.set_raise_amount(1.05)
```

#### Alternative Constructor Using Class Method
```python
class Employee:
    def __init__(self, first, last, pay):
        self.first = first
        self.last = last
        self.pay = pay
        self.email = f'{first.lower()}.{last.lower()}@company.com'
    
    @classmethod
    def from_string(cls, emp_str):
        first, last, pay = emp_str.split('-')
        return cls(first, last, int(pay))

# Usage
emp_str = 'John-Doe-70000'
new_emp = Employee.from_string(emp_str)
```

#### Static Method Example
```python
class Employee:
    @staticmethod
    def is_workday(day):
        return day.weekday() < 5

# Usage
import datetime
my_date = datetime.date(2024, 5, 17)
print(Employee.is_workday(my_date))  # Output: True (if it's a weekday)
```

### Additional Notes

- **Class Method Use Cases**
  - Often used for factory methods that return class instances.
  - Examples: Parsing strings, working with different input formats.

- **Static Method Use Cases**
  - Utility functions that are related to the class but don’t modify its state.
  - Examples: Date manipulation, validation functions.
___
## OOP-4:-Inheritance-Creating-Subclasses

### Key Insights
- **Inheritance in Python**:
  - Allows subclasses to inherit attributes and methods from a parent class.
  - Useful for creating subclasses (e.g., `Developer`, `Manager`) that reuse code from the parent class (`Employee`).

- **Creating Subclasses**:
  - Define a subclass using the `class SubclassName(ParentClass):` syntax.
  - Example:
    ```python
    class Developer(Employee):
        pass
    ```
  - Subclasses inherit all attributes and methods from the parent class.
  - We can see method resolution order in `print(help(Developer))` 

- **Overriding Methods**:
  - Subclasses can override methods from the parent class to provide specific functionality.
  - Example:
    ```python
    class Developer(Employee):
        raise_amount = 1.10  # Override the raise amount for developers
    ```

- **Custom Initialization in Subclasses**:
  - Subclasses can have their own `__init__` method.
  - Use `super()` to call the parent class's `__init__` method.
  - Example:
    ```python
    class Developer(Employee):
        def __init__(self, first, last, pay, prog_lang):
            super().__init__(first, last, pay)
            # Employee.__init__(self, first,last,pay)
            self.prog_lang = prog_lang
    ```

- **Method Resolution Order (MRO)**:
  - Python uses MRO to determine which method to execute when called.
  - Use the `help()` function to visualize the MRO.

- **Built-in Functions `isinstance` and `issubclass`**:
  - `isinstance(object, class)`: Checks if an object is an instance of a class.
  - `issubclass(class, parent_class)`: Checks if a class is a subclass of another class.
  - Examples:
    ```python
    isinstance(dev_1, Developer)  # True
    issubclass(Developer, Employee)  # True
    ```

### Code Snippets
- **Creating Developer and Manager Subclasses**:
  ```python
  class Developer(Employee):
      raise_amount = 1.10
      
      def __init__(self, first, last, pay, prog_lang):
          super().__init__(first, last, pay)
          self.prog_lang = prog_lang

  class Manager(Employee):
      def __init__(self, first, last, pay, employees=None): #None instead of [], use immutable datatypes as default
          super().__init__(first, last, pay)
          if employees is None:
              self.employees = []
          else:
              self.employees = employees

      def add_employee(self, emp):
          if emp not in self.employees:
              self.employees.append(emp)

      def remove_employee(self, emp):
          if emp in self.employees:
              self.employees.remove(emp)

      def print_employees(self):
          for emp in self.employees:
              print('-->', emp.fullname())
  ```

- **Testing Subclasses**:
  ```python
  dev_1 = Developer('Corey', 'Schafer', 50000, 'Python')
  dev_2 = Developer('Test', 'Employee', 60000, 'Java')

  mgr_1 = Manager('Sue', 'Smith', 90000, [dev_1])

  print(mgr_1.email)
  mgr_1.print_employees()
  mgr_1.add_employee(dev_2)
  mgr_1.print_employees()
  mgr_1.remove_employee(dev_1)
  mgr_1.print_employees()
  ```
___
## OOP-5: `str()`-vs-`repr()`

#### Important-Notes

* Both return **string representations** of objects.
* `str(obj)` → user-friendly, readable, meant for display.
* `repr(obj)` → unambiguous, meant for debugging, ideally valid Python code to recreate the object.
* Custom classes can override `__str__` and `__repr__`.

#### Code-Snippet

```python
# Built-in types
x = 3.14159
print(str(x))   # Output: 3.14159
print(repr(x))  # Output: 3.14159

s = "hello\nworld"
print(str(s))   # Output: hello
                 #         world   (prints newline)
print(repr(s))  # Output: 'hello\nworld'


# Custom class
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __str__(self):   # user-friendly
        return f"{self.name}, Age {self.age}"
    
    def __repr__(self):  # unambiguous
        return f"Person(name={self.name!r}, age={self.age!r})"

p = Person("Vamsi", 31)
print(str(p))   # Output: Vamsi, Age 31
print(repr(p))  # Output: Person(name='Vamsi', age=31)
```

#### Key-Takeaway

* Use `str()` for **readable output** (end-users).
* Use `repr()` for **debugging/developers** (unambiguous, recreate object).
## OOP-6:-Special-(Magic/Dunder)-Methods

#### Key Insights:
  - Special methods in Python allow us to emulate built-in behavior and implement operator overloading.
  - These methods are surrounded by double underscores (e.g., `__init__`).
  - Commonly referred to as "dunder" methods.

- **__init__ Method**
  - Already familiar and commonly used for initializing objects.
  - Automatically called when creating an instance of a class.

- **__repr__ and __str__ Methods**
  - `__repr__`: Unambiguous representation of the object, useful for debugging and logging. 
  - `__str__`: Readable representation of the object, meant for end-users.
  - Important to implement both for better readability and usability of custom objects.

- **Example Implementation of __repr__ and __str__**
  ```python
  class Employee:
      def __init__(self, first, last, pay):
          self.first = first
          self.last = last
          self.pay = pay

      def __repr__(self):
          return f"Employee('{self.first}', '{self.last}', {self.pay})"

      def __str__(self):
          return f'{self.first} {self.last} - {self.email}'
      
      @property
      def email(self):
          return f'{self.first.lower()}.{self.last.lower()}@company.com'
  ```

- **Arithmetic Special Methods**
  - Special methods like `__add__` allow customization of arithmetic operations.
  - Example of adding salaries of two `Employee` objects:
  ```python
  class Employee:
      # ... other methods

      def __add__(self, other):
          return self.pay + other.pay
  ```

- **Special Method for Length**
  - `__len__` can be implemented to use the `len()` function on custom objects.
  - Example returning the total number of characters in an employee’s full name:
  ```python
  class Employee:
      # ... other methods

      def __len__(self):
          return len(self.fullname())
      
      def fullname(self):
          return f'{self.first} {self.last}'
  ```

- **Real-World Examples**
  - `datetime` module uses special methods extensively (e.g., `__add__` in `timedelta` class).
  - Date class uses `__repr__` for an unambiguous string and `__str__` linked to `isoformat` for readability.

- **Useful Links**
  - Python documentation for special methods: [Python Data Model](https://docs.python.org/3/reference/datamodel.html#special-method-names)
___
## OOP-7: Property Decorators - Getters, Setters, and Deleters

**Key Insights:**

- **Introduction to Property Decorators:**
  - Property decorators allow adding getter, setter, and deleter functionality to class attributes.
  - Useful to maintain attribute consistency without changing existing code.

- **Basic Employee Class Example:**
  - The `email` attribute depends on `first_name` and `last_name`.
  - Changing `first_name` or `last_name` directly does not update `email`.

- **Using Property Decorators:**
  - Define a method for `email` and use `@property` to access it like an attribute.
  - Example:
    ```python
    class Employee:
        def __init__(self, first, last):
            self.first = first
            self.last = last
        
        @property
        def email(self):
            return f'{self.first}.{self.last}@email.com'
        
        @property
        def full_name(self):
            return f'{self.first} {self.last}'
    ```

- **Adding Setter for `full_name`:**
  - Use `@<property_name>.setter` decorator.
  - Allows setting `full_name` and updating `first_name` and `last_name` accordingly.
  - Example:
    ```python
    class Employee:
        # previous code...
        
        @full_name.setter
        def full_name(self, name):
            first, last = name.split(' ')
            self.first = first
            self.last = last
    ```

- **Adding Deleter for `full_name`:**
  - Use `@<property_name>.deleter` decorator.
  - Example:
    ```python
    class Employee:
        # previous code...
        
        @full_name.deleter
        def full_name(self):
            print('Deleting name...')
            self.first = None
            self.last = None
    ```

- **Usage Example:**
  ```python
  emp1 = Employee('John', 'Smith')
  print(emp1.email)  # John.Smith@email.com
  print(emp1.full_name)  # John Smith
  
  emp1.full_name = 'Jim Halpert'
  print(emp1.email)  # Jim.Halpert@email.com
  
  del emp1.full_name
  print(emp1.first)  # None
  print(emp1.last)  # None
  ```

**Benefits of Property Decorators:**
- Maintain consistent attribute access.
- Avoid breaking existing code when adding new functionality.
- Cleaner and more readable code.

**Considerations:**
- Be cautious when converting methods to properties if the class is widely used, as it might require changes in the consuming code.