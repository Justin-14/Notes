# Python Class and Object — Interview Guide (30 Questions)

## Introduction

This guide is tailored for a Python backend developer with approximately 4 years of experience. It moves beyond the syntax of "how to define a class" into the architecture, memory management, design patterns, and "under the hood" mechanics of Python's object model.

---

# SECTION 1 — THEORY QUESTIONS (20 TOTAL)

## Part A: Easy Level (Concepts & Syntax)

### 1. The Role of `self`

**Question:** Explain the technical purpose of `self` in Python. Is it a keyword? What happens if you name it something else?

**Expected Answer:**
`self` represents the instance of the class. It binds the attributes with the given arguments. It is **not** a keyword in Python; it is a strong naming convention. You can technically name it `this` or `me`, but doing so is strongly discouraged as it breaks readability standards (PEP 8).

**Explanation:**
When you call `obj.method(arg)`, Python internally converts it to `Class.method(obj, arg)`. The instance `obj` is passed as the first argument automatically.

**Code:**

```python
class Demo:
    def __init__(myself, name):  # technically valid, but bad practice
        myself.name = name

    def display(myself):
        print(myself.name)

d = Demo("Python")
d.display()

```

**Why interviewers ask this:** To check if you understand explicit instance passing vs. implicit context (like `this` in Java/C++).

---

### 2. Class vs. Instance Variables

**Question:** What is the specific difference in memory allocation and scope between a class variable and an instance variable?

**Expected Answer:**

* **Class Variables:** Shared across all instances of the class. Defined directly in the class body. Modifying a mutable class variable affects all instances.
* **Instance Variables:** Unique to each instance. Defined inside methods (usually `__init__`) using `self`.

**Reasoning:**
Class variables are attributes of the Class Object itself. Instance variables are entries in the instance's `__dict__`.

**Diagram:**

```text
[ Class: Dog ]  <-- kind = "canine" (Shared)
      |
      +-- [ Instance: d1 ] <-- name = "Buddy" (Unique)
      +-- [ Instance: d2 ] <-- name = "Rex"   (Unique)

```

**Common Mistakes:**
Assigning a value to a class variable via an instance (`self.var = x`) creates a new *instance* variable that shadows the class variable, rather than updating the shared one.

---

### 3. `__str__` vs `__repr__`

**Question:** When should you implement `__str__` versus `__repr__`? What is the fallback behavior?

**Expected Answer:**

* `__str__`: Intended for the end-user. It should be readable. Called by `print()` and `str()`.
* `__repr__`: Intended for developers. It should be unambiguous and, if possible, a valid Python expression to recreate the object. Called by `repr()` and in the debugger.
* **Fallback:** If `__str__` is missing, Python calls `__repr__`.

**Sample Code:**

```python
import datetime

now = datetime.datetime.now()
print(str(now))  # 2026-01-04 10:30:00.123456 (Readable)
print(repr(now)) # datetime.datetime(2026, 1, 4, 10, 30, ...) (Exact)

```

---

### 4. Constructor: `__init__`

**Question:** Is `__init__` the constructor of a Python class? What does it strictly do?

**Expected Answer:**
Strictly speaking, `__init__` is the **initializer**, not the constructor. It creates no object. It receives the object (created by `__new__`) and populates initial attributes.

**Comparison:**
In C++ or Java, the constructor allocates memory. In Python, the allocation happens before `__init__` runs.

**Why interviewers ask this:** To separate senior engineers who know the object lifecycle from juniors who just know syntax.

---

### 5. Access Modifiers (Public, Protected, Private)

**Question:** Python does not have `private` or `protected` keywords. How do we implement encapsulation, and how strict is it?

**Expected Answer:**

* **Public:** No underscore (e.g., `name`). Accessible everywhere.
* **Protected:** Single underscore (e.g., `_name`). Convention only; signals "internal use."
* **Private:** Double underscore (e.g., `__name`). Triggers **Name Mangling** (`_ClassName__name`). It makes accidental access harder, but not impossible.

**Common Mistakes:**
Believing `__private` is truly secure. It can still be accessed via `obj._ClassName__private`.

---

### 6. Basics of Inheritance

**Question:** How do you verify if a class is a subclass of another, or if an instance belongs to a class?

**Expected Answer:**
Use `issubclass(Child, Parent)` for class relationships. Use `isinstance(obj, Class)` for object types.

**Reasoning:**
`isinstance()` is preferred over `type(obj) == Class` because `isinstance()` handles inheritance correctly (returns True for subclasses), whereas direct type comparison does not.

---

## Part B: Medium Level (Deep Reasoning & Internals)

### 7. The `__new__` Method

**Question:** We discussed `__init__`. When would you actually need to override `__new__`?

**Expected Answer:**
`__new__` is a static method responsible for creating and returning the instance. You override it when:

1. Subclassing **immutable types** (like `str`, `int`, `tuple`) because you cannot modify them in `__init__`.
2. Implementing the **Singleton Pattern** (controlling instance creation).
3. Metaclass programming.

**Sample Code:**

```python
class PositiveInt(int):
    def __new__(cls, value):
        return super().__new__(cls, abs(value))

x = PositiveInt(-5)
print(x)  # 5

```

**Line-by-line:** `__new__` intercepts the creation. We call `super().__new__` with the absolute value *before* the int is created.

---

### 8. MRO (Method Resolution Order)

**Question:** In multiple inheritance, how does Python decide which method to call? Explain the C3 Linearization algorithm briefly.

**Expected Answer:**
Python uses the C3 Linearization algorithm to determine the MRO. It ensures:

1. Subclasses appear before parents.
2. The order of inheritance in the class definition is preserved.
3. Monotonicity (the order doesn't change if you add a new parent).

You can view it via `ClassName.mro()`.

**Diagram:**

```text
    A
   / \
  B   C
   \ /
    D

```

MRO for D: `D -> B -> C -> A -> object` (Assuming `class D(B, C)`).

**Why interviewers ask this:** Diamond Problem resolution is a classic computer science problem.

---

### 9. `__slots__`

**Question:** What is the purpose of `__slots__`, and what is the trade-off when using it?

**Expected Answer:**
`__slots__` tells Python to not use a dynamic `__dict__` for instances, but to reserve space for a fixed set of attributes.

* **Benefit:** significantly reduced memory usage (40-50% less) and faster attribute access.
* **Trade-off:** You cannot add new attributes to instances at runtime. It breaks some multiple inheritance layouts if not careful.

**Sample Code:**

```python
class Pixel:
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y

```

---

### 10. @staticmethod vs @classmethod

**Question:** When should you use `@classmethod` over `@staticmethod`? Give a concrete scenario.

**Expected Answer:**

* **@classmethod:** Receives the class (`cls`) as the first argument. Use for **Factory methods** (alternative constructors) so that inheritance works correctly (returns an instance of the subclass, not the hardcoded parent).
* **@staticmethod:** Just a plain function inside a class namespace. Doesn't access `cls` or `self`. Use for utility functions logically related to the class.

**Scenario:**
Creating a `Date` class. `Date.from_string("2024-01-01")` should be a `@classmethod` so if I subclass `DateTime(Date)`, the method returns a `DateTime` object.

---

### 11. Property Decorators (Getters/Setters)

**Question:** Why do we use `@property` in Python instead of `get_variable()` and `set_variable()` methods like Java?

**Expected Answer:**
It allows backward compatibility. We can start with a public attribute `obj.x`. Later, if we need validation logic, we can change it to a `@property` without changing the interface (it's still accessed as `obj.x`). This is "Pythonic."

**Code:**

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0: raise ValueError("Positive only")
        self._radius = value

```

---

### 12. Metaclasses

**Question:** What is a Metaclass? If everything in Python is an object, what is the class of a Class?

**Expected Answer:**
A Metaclass is "the class of a class." It defines how a class behaves. The default metaclass is `type`.
`obj` is an instance of `Class`. `Class` is an instance of `Metaclass`.
You use them to intercept class creation (e.g., to enforce coding standards, auto-register plugins, or valid API schemas).

**Diagram:**

```text
Instance -> Class -> type (Metaclass)

```

---

### 13. Abstract Base Classes (ABC)

**Question:** How do you enforce that every subclass *must* implement a specific method?

**Expected Answer:**
Inherit from `abc.ABC` and use the `@abstractmethod` decorator. If a subclass fails to implement that method, Python will throw a `TypeError` at **instantiation time**, preventing the creation of an incomplete object.

**Why interviewers ask this:** To check knowledge of interface enforcement and API design.

---

### 14. Data Classes (Python 3.7+)

**Question:** Why were `dataclasses` introduced? How do they differ from `namedtuples`?

**Expected Answer:**
`dataclasses` automate the generation of boilerplate code (`__init__`, `__repr__`, `__eq__`).

* **vs namedtuple:** Data classes are mutable by default; namedtuples are immutable. Data classes support inheritance and type hinting much better.

**Sample Code:**

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    username: str
    active: bool = True

```

---

### 15. The `super()` Function

**Question:** In Python 3, why can we call `super()` without arguments? What does it actually do?

**Expected Answer:**
`super()` delegates method calls to a class in the instance's MRO.
In Python 3, the compiler automatically inserts `super(__class__, self)`. It acts as a proxy object that routes the call to the *next* class in the linearization list (MRO), not necessarily the direct parent (crucial in multiple inheritance).

---

### 16. Duck Typing

**Question:** "If it looks like a duck and quacks like a duck, it's a duck." How does this apply to Python classes?

**Expected Answer:**
Python does not enforce type checking at compile time. If an object implements the required methods (e.g., `__len__`, `__iter__`), it can be used in any context expecting those behaviors, regardless of its inheritance hierarchy. This is also known as EAFP (Easier to Ask for Forgiveness than Permission).

---

### 17. Magic Method: `__call__`

**Question:** How do you make an instance of a class callable like a function?

**Expected Answer:**
Implement the `__call__` magic method.
**Usage:** Useful for decorators, strategy patterns, or objects that maintain state between calls (like a counter or a neural network layer).

**Code:**

```python
class Adder:
    def __init__(self, n):
        self.n = n
    def __call__(self, x):
        return self.n + x

add5 = Adder(5)
print(add5(10)) # 15

```

---

### 18. Context Managers within Classes

**Question:** How do you make a class compatible with the `with` statement?

**Expected Answer:**
Implement `__enter__` and `__exit__`.

* `__enter__`: Sets up the resource and returns the object used in `as var`.
* `__exit__`: Handles teardown (closing files) and exception suppression (if it returns True).

**Relevance:** Database connections, File I/O, Lock management.

---

### 19. Dependency Injection

**Question:** How do you design a class to be testable? Explain Dependency Injection.

**Expected Answer:**
Instead of instantiating dependencies (like a database connector) *inside* the class, you pass them in via the constructor (`__init__`).
This allows you to pass a "Mock" object during testing without changing the class code.

**Code:**

```python
# Bad
class Service:
    def __init__(self):
        self.db = MySQLConnection() # Hard dependency

# Good (DI)
class Service:
    def __init__(self, db_conn):
        self.db = db_conn # Injectable

```

---

### 20. Garbage Collection & Circular References

**Question:** How does Python handle memory for objects? What happens if two objects reference each other?

**Expected Answer:**
Python uses **Reference Counting** as the primary mechanism. When ref count drops to 0, memory is freed.
However, for circular references (A -> B -> A), ref count never reaches 0. Python's **Garbage Collector (GC)** runs periodically to detect and clean up these isolated reference cycles. `__del__` allows finalization but is unreliable in circular references.

---

# SECTION 2 — CODING QUESTIONS (10 TOTAL)

## Part A: Easy Level

### 1. Bank Account Encapsulation

**Question:**
Create a `BankAccount` class.

1. Initialize with an owner name and a balance (default 0).
2. Methods: `deposit(amount)` and `withdraw(amount)`.
3. Prevent direct access to `balance` (use private variable).
4. Prevent overdrafts in `withdraw` (print "Insufficient funds").

**Solution:**

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.__balance = balance  # Private attribute

    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
            print(f"Deposited ${amount}")

    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            print(f"Withdrew ${amount}")
        else:
            print("Insufficient funds or invalid amount")

    def get_balance(self):
        return self.__balance

# Usage
acc = BankAccount("John", 100)
acc.withdraw(150) # Insufficient funds
acc.deposit(50)
print(acc.get_balance()) # 150

```

**Explanation:**

* `self.__balance` hides data from external modification.
* Logic is encapsulated within methods.

---

### 2. Geometry Class Inheritance

**Question:**
Create a base class `Shape` with a method `area()`. Create two subclasses `Rectangle` (l, w) and `Circle` (r) that override `area()`.
Use `math.pi` for Circle.

**Solution:**

```python
import math

class Shape:
    def area(self):
        raise NotImplementedError("Subclass must implement abstract method")

class Rectangle(Shape):
    def __init__(self, length, width):
        self.length = length
        self.width = width

    def area(self):
        return self.length * self.width

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * (self.radius ** 2)

```

**Why interviewers ask this:** Simple polymorphism check.

---

### 3. Student Grade Tracker

**Question:**
Create a `Student` class.

1. `__init__` takes name.
2. `add_grade(score)` adds a score to a list.
3. `average()` returns the average score.

**Solution:**

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.grades = []

    def add_grade(self, score):
        self.grades.append(score)

    def average(self):
        if not self.grades:
            return 0
        return sum(self.grades) / len(self.grades)

```

**Mistake to avoid:** Mutating a mutable default argument in `__init__` (e.g., `def __init__(self, grades=[])`).

---

## Part B: Medium Level (Real-world Scenarios)

### 4. Custom Iterator Class

**Question:**
Create a class `Countdown` that takes a number `start`. It should work in a loop: `for x in Countdown(5): ...` counting down to 1. Do not use a generator function; implement the Iterator Protocol (`__iter__` and `__next__`).

**Solution:**

```python
class Countdown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current < 1:
            raise StopIteration
        num = self.current
        self.current -= 1
        return num

# Usage
for i in Countdown(3):
    print(i)

```

**Line-by-line:**

* `__iter__`: Returns `self` because the object itself tracks state.
* `__next__`: Returns the value or raises `StopIteration` to signal the loop to end.
* **Relevance:** Used when building custom data streams (e.g., reading lines from a large log file).

---

### 5. LRU Cache (Least Recently Used)

**Question:**
Implement an LRU Cache using a Class.

1. Capacity `n`.
2. `get(key)`: Return value or -1. Move key to "recently used".
3. `put(key, value)`: Add key. If at capacity, remove least recently used item.
Hint: Use `OrderedDict`.

**Solution:**

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        # Move to end (show it was recently used)
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            # FIFO: pop the first item (least recently used)
            self.cache.popitem(last=False)

```

**Time Complexity:** O(1) for both get and put.
**Scenario:** Caching API responses or DB query results in backend services like Reddit or Instagram.

---

### 6. Thread-Safe Singleton

**Question:**
Implement a Singleton class that is safe to use in a multithreaded environment. Ensure only ONE instance exists ever.

**Solution:**

```python
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            with cls._lock:
                # Double-checked locking
                if not cls._instance:
                    cls._instance = super().__new__(cls)
        return cls._instance

# Usage
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True

```

**Explanation:**

* We use a `Lock` to prevent race conditions where two threads enter the creation block simultaneously.
* **Double-checked locking:** We check `_instance` inside the lock again to be sure.
* **Scenario:** Database connection pools, Logging configuration.

---

### 7. Custom Context Manager (Timer)

**Question:**
Create a class `Timer` that measures the execution time of a code block using `with`. Print the time taken upon exiting.

**Solution:**

```python
import time

class Timer:
    def __enter__(self):
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end = time.time()
        self.interval = self.end - self.start
        print(f"Code block took: {self.interval:.5f} seconds")
        # Return False to propagate exceptions, True to suppress
        return False

# Usage
with Timer():
    # Simulate work
    sum([x**2 for x in range(100000)])

```

**Developer Scenario:** Performance profiling specific functions in a Django/FastAPI app.

---

### 8. The Factory Pattern (Serializer Factory)

**Question:**
Create a system that processes data based on format ('JSON', 'XML').

1. Base class `Serializer`.
2. Subclasses `JSONSerializer`, `XMLSerializer`.
3. `SerializerFactory` class with a method `get_serializer(format)`.

**Solution:**

```python
from abc import ABC, abstractmethod

# Interfaces
class Serializer(ABC):
    @abstractmethod
    def serialize(self, data): pass

# Concrete Implementations
class JSONSerializer(Serializer):
    def serialize(self, data):
        return f"{{ 'data': '{data}' }}"

class XMLSerializer(Serializer):
    def serialize(self, data):
        return f"<data>{data}</data>"

# Factory
class SerializerFactory:
    @staticmethod
    def get_serializer(format_type):
        if format_type == 'JSON':
            return JSONSerializer()
        elif format_type == 'XML':
            return XMLSerializer()
        else:
            raise ValueError("Unknown format")

# Usage
factory = SerializerFactory()
serializer = factory.get_serializer('XML')
print(serializer.serialize("Hello"))

```

**Why:** Decouples client code from specific class implementations.

---

### 9. Custom Dictionary (Inheriting UserDict)

**Question:**
Create a dictionary class `CaseInsensitiveDict` where keys "Name" and "name" are treated as the same key.
(Use `collections.UserDict`, generally safer/easier than `dict` for subclassing).

**Solution:**

```python
from collections import UserDict

class CaseInsensitiveDict(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key.lower(), value)

    def __getitem__(self, key):
        return super().__getitem__(key.lower())

    def __contains__(self, key):
        return super().__contains__(key.lower())

# Usage
d = CaseInsensitiveDict()
d['Name'] = "Alice"
print(d['name'])  # Alice

```

**Real App Usage:** HTTP Headers processing (headers are case-insensitive).

---

### 10. Observer Pattern (Event System)

**Question:**
Implement a simple Event System.

1. `EventManager` class.
2. `subscribe(event_name, callback)`.
3. `notify(event_name, data)`.

**Solution:**

```python
class EventManager:
    def __init__(self):
        self.listeners = {}

    def subscribe(self, event_type, listener):
        if event_type not in self.listeners:
            self.listeners[event_type] = []
        self.listeners[event_type].append(listener)

    def notify(self, event_type, data):
        if event_type in self.listeners:
            for listener in self.listeners[event_type]:
                listener(data)

# Usage
def on_user_signup(user):
    print(f"Welcome email sent to {user}")

def on_user_signup_log(user):
    print(f"Log: User {user} created")

events = EventManager()
events.subscribe("signup", on_user_signup)
events.subscribe("signup", on_user_signup_log)

events.notify("signup", "user@example.com")

```

**Developer Scenario:** Decoupling logic. When a user buys an item (Amazon), notify Inventory Service, Email Service, and Analytics Service without the "Buy" function knowing about them.
