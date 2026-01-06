# Python Constructor in Inheritance — Interview Guide (30 Questions)

This comprehensive guide covers Python constructors (`__init__` and `__new__`) specifically within the context of inheritance. It is tailored for a Python developer with 4 years of experience, moving from basic syntax to the nuances of Method Resolution Order (MRO), cooperative multiple inheritance, and design patterns.

# ============================================================ SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### 1. Basic `__init__` Overriding (Easy)

**Question:**
When you define a class `Child` that inherits from `Parent`, what happens if you define an `__init__` method in `Child` but do not explicitly call `Parent.__init__`?

**Expected Answer:**
If `Child` defines its own `__init__`, it overrides the `Parent`'s `__init__`. The `Parent`'s initialization logic will **not** execute automatically. The attributes defined in the `Parent` class will not be initialized in the `Child` instance unless explicitly called.

**Explanation:**
Python does not automatically chain constructors like Java or C++. If the child class provides an `__init__`, it is the sole responsibility of that method to ensure the object is fully initialized, which usually involves calling `super().__init__()`.

**Common Mistakes:**
Assuming Python automatically calls the parent constructor.

**Why Interviewers Ask This:**
To verify basic understanding of Python's explicit nature versus implicit behaviors in other languages.

**Sample Code:**

```python
class Parent:
    def __init__(self):
        print("Parent init")
        self.parent_attr = "parent"

class Child(Parent):
    def __init__(self):
        print("Child init")
        # Parent.__init__(self) is missing!

c = Child()
# Output: "Child init"
# accessing c.parent_attr would raise AttributeError

```

---

### 2. The Purpose of `super()` (Easy)

**Question:**
What is the primary purpose of `super()` in a class constructor, and how does it differ from calling `Parent.__init__(self)` directly?

**Expected Answer:**
`super()` returns a proxy object that delegates method calls to a parent or sibling class of a type. Its primary purpose is to support cooperative multiple inheritance and ensure the Method Resolution Order (MRO) is respected. Calling `Parent.__init__(self)` hardcodes the parent class, which can break if the class hierarchy changes or in diamond inheritance scenarios.

**Explanation:**
`super()` makes the code more maintainable and is essential for complex inheritance structures to ensure every class in the hierarchy is initialized exactly once.

**Time/Space Complexity:**
O(N) where N is the length of the MRO list (for calculating the next method).

**Why Interviewers Ask This:**
To ensure you understand modern Python inheritance best practices.

**Sample Code:**

```python
class Base:
    def __init__(self):
        print("Base init")

class Derived(Base):
    def __init__(self):
        # Recommended way
        super().__init__() 
        # Hardcoded way (Avoid in multiple inheritance)
        # Base.__init__(self) 

```

---

### 3. `self` in Constructors (Easy)

**Question:**
In the context of inheritance, when `super().__init__()` is called, which object does `self` refer to in the parent class's `__init__` method?

**Expected Answer:**
`self` always refers to the **instance currently being created**, specifically the instance of the most derived class (the class instantiated by the user). Even inside the Parent's `__init__`, `self` is the Child instance.

**Explanation:**
Inheritance allows the parent to operate on the child's instance memory. This is why attributes set in `Parent.__init__` end up attached to the `Child` object.

**Why Interviewers Ask This:**
To confuse candidates about scope and instance identity.

**ASCII Diagram:**

```text
Instance creation: c = Child()
   |
   +-- Child.__init__(self=c)
          |
          +-- super().__init__() calls Parent
                 |
                 +-- Parent.__init__(self=c)  <-- self is still 'c'

```

---

### 4. Passing Arguments Upwards (Easy)

**Question:**
If a Parent class constructor requires arguments, how do you handle them in the Child class constructor?

**Expected Answer:**
The Child class constructor should accept those arguments (along with its own) and pass the relevant ones to `super().__init__()`.

**Explanation:**
The signature of the Child's init usually extends the Parent's. You capture the required data and forward it.

**Sample Code:**

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

class Square(Rectangle):
    def __init__(self, side_length):
        super().__init__(width=side_length, height=side_length)

```

---

### 5. Accessing Parent Attributes (Easy)

**Question:**
Can a Child class access attributes defined in the Parent's `__init__` immediately after calling `super().__init__()`?

**Expected Answer:**
Yes. Once `super().__init__()` returns, the parent initialization is complete, and the `self` instance has been populated with the parent's attributes.

**Common Mistakes:**
Trying to access parent attributes *before* calling `super().__init__()`.

**Sample Code:**

```python
class Parent:
    def __init__(self):
        self.secret = 42

class Child(Parent):
    def __init__(self):
        super().__init__()
        print(self.secret) # Works: Prints 42

```

---

### 6. Default Arguments in Inheritance (Easy)

**Question:**
How do mutable default arguments in a constructor affect inheritance if both Parent and Child use them?

**Expected Answer:**
The "mutable default argument" trap applies regardless of inheritance. However, in inheritance, if a Child relies on a Parent's default mutable arg (like a list), all instances of Child (and Parent) might end up sharing the same list object if not handled carefully using `None`.

**Why Interviewers Ask This:**
It combines a classic Python "gotcha" with inheritance.

**Sample Code:**

```python
class Parent:
    def __init__(self, items=None):
        self.items = [] if items is None else items

```

---

### 7. The Diamond Problem & MRO (Medium)

**Question:**
Explain the Diamond Problem in Python. How does Python 3 solve the issue of the base class constructor being called multiple times?

**Expected Answer:**
The Diamond Problem occurs when Class D inherits from B and C, and both B and C inherit from A. If B and C call `A.__init__` explicitly, A is initialized twice.
Python 3 solves this using the C3 Linearization algorithm to generate a Method Resolution Order (MRO). If everyone uses `super()`, Python ensures A is called only once, after B and C.

**Medium-Depth Reasoning:**
`super()` doesn't just call the parent; it calls the *next* class in the MRO. In a diamond shape, B's "next" might be C, not A, effectively chaining siblings before grandparents.

**ASCII Diagram:**

```text
      A
     / \
    B   C
     \ /
      D

```

**MRO for D:** D -> B -> C -> A -> object

---

### 8. Cooperative Multiple Inheritance (Medium)

**Question:**
What is "Cooperative Multiple Inheritance," and why must `**kwargs` be used in constructors to support it?

**Expected Answer:**
Cooperative Multiple Inheritance relies on all classes in the hierarchy calling `super().__init__`. Since classes in the MRO might accept different arguments, using `*args` and `**kwargs` is crucial. Each class pops the arguments it needs from `kwargs` and passes the rest to the next class via `super().__init__(**kwargs)`.

**Why Interviewers Ask This:**
This is the standard pattern for writing mixins and complex frameworks (like Django or Django REST Framework).

**Sample Code:**

```python
class A:
    def __init__(self, **kwargs):
        print("A init")
        super().__init__(**kwargs) # Passes empty kwargs to object

class B(A):
    def __init__(self, b_val, **kwargs):
        print(f"B init: {b_val}")
        super().__init__(**kwargs)

# Usage implies carefully stripping args

```

---

### 9. `__new__` vs `__init__` in Inheritance (Medium)

**Question:**
When inheriting from an immutable type (like `str` or `tuple`), why must you override `__new__` instead of `__init__`?

**Expected Answer:**
`__new__` is the actual constructor that creates the instance. `__init__` is an initializer called *after* creation. For immutable types, the object is created and its value set in `__new__`. By the time `__init__` runs, the object is already created and cannot be modified.

**Common Mistakes:**
Trying to modify `self` in `__init__` for a subclass of `str`.

**Sample Code:**

```python
class UpperString(str):
    def __new__(cls, content):
        return super().__new__(cls, content.upper())

```

---

### 10. `super()` without Arguments (Medium)

**Question:**
In Python 3, you can call `super()` without arguments. How does Python know which class and instance to use?

**Expected Answer:**
It is "compiler magic" (specifically, the implementation involves adding a generic cell `__class__` and a variable `self` to the function scope). At runtime, `super()` looks for the variable named `self` (the instance) and `__class__` (the class where the method is defined) in the local scope to calculate the MRO correctly.

**Why Interviewers Ask This:**
To see if you understand that `super()` is context-dependent and not just a keyword alias.

---

### 11. Abstract Base Classes (ABC) and Init (Medium)

**Question:**
If an Abstract Base Class (ABC) has an `__init__`, does a concrete subclass have to call it? What happens if it doesn't?

**Expected Answer:**
Technically, Python does not enforce calling `super().__init__` even for ABCs. However, if the ABC sets up internal state (like private variables) required for its concrete methods to work, failing to call it will lead to runtime errors when those methods are used.

**Sample Code:**

```python
from abc import ABC, abstractmethod

class Base(ABC):
    def __init__(self):
        self._cache = {} # Essential state

class Concrete(Base):
    def __init__(self):
        # If we forget super().__init__(), self._cache is missing
        pass 

```

---

### 12. Mixins and Initialization (Medium)

**Question:**
When designing a Mixin class that requires initialization, how do you ensure it doesn't break the inheritance chain of the class it is mixed into?

**Expected Answer:**
The Mixin should accept `*args` and `**kwargs` and pass them to `super().__init__`. It should ideally be designed to be agnostic of where it sits in the hierarchy. It should not hardcode a parent class.

**Why Interviewers Ask This:**
Mixins are ubiquitous in Python (e.g., `LoginRequiredMixin`), and improper `init` chains are a common source of bugs.

---

### 13. Parameter Name Collisions (Medium)

**Question:**
In a multiple inheritance scenario where two parent classes expect a parameter with the same name (e.g., `id`) but distinct meanings, how do you resolve the conflict in the child's constructor?

**Expected Answer:**
You must explicitly accept the conflicting arguments in the child's `__init__` with distinct names (e.g., `user_id` and `order_id`), and then pass them explicitly to the specific parent classes (by name) or modify `kwargs` carefully before calling `super()`. Sometimes, bypassing `super()` for specific parents and calling `Parent.__init__` directly is the only clean solution, though it breaks MRO cooperativeness.

---

### 14. `__init_subclass__` (Medium)

**Question:**
What is `__init_subclass__` and how does it relate to class construction and inheritance?

**Expected Answer:**
`__init_subclass__` is a class method called whenever a class inheriting from the defining class is created. It allows the parent class to customize the *class setup* of its children without using a metaclass. It is often used to validate that subclasses define specific attributes or to register plugins.

**Sample Code:**

```python
class PluginBase:
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        if not hasattr(cls, 'name'):
            raise TypeError("Plugins must have a name")

```

---

### 15. Slots and Inheritance (Medium)

**Question:**
If a Parent class defines `__slots__` and a Child class inherits from it, does the Child class automatically inherit the slot restrictions? How does `__init__` behave?

**Expected Answer:**
The Child inherits the slots of the Parent, but if the Child doesn't define `__slots__` itself, it will have a `__dict__` and allow dynamic attribute assignment. To maintain memory efficiency, the Child must also define `__slots__` (even if empty). `__init__` works normally, but it can only assign to attributes defined in the union of all `__slots__` in the hierarchy.

---

### 16. Dependency Injection via Init (Medium)

**Question:**
How does Constructor Injection facilitate testing in a class hierarchy compared to hardcoding dependencies in `__init__`?

**Expected Answer:**
By passing dependencies (like database connections or API clients) as arguments to `__init__`, subclasses or tests can easily swap them for mocks. If a Parent hardcodes `self.db = PostgresDB()`, a Child cannot easily override this for testing without patching.

**Developer Scenario:**
Writing unit tests for a Service class that inherits from a BaseService.

---

### 17. Calling `__init__` multiple times (Medium)

**Question:**
Is it legal to call `__init__` on an object that has already been initialized? What are the risks?

**Expected Answer:**
Yes, it is legal (it's just a method). However, it resets the object's state to what is defined in `__init__`. This is generally bad practice unless you are implementing a specific "reset" functionality (which is better named `reset()`). In inheritance, accidentally calling a parent's `__init__` twice (via poor MRO management) causes performance loss and state reset bugs.

---

### 18. Returning values from `__init__` (Medium)

**Question:**
What happens if you return a value (like `True` or a string) from `__init__` in a Python class?

**Expected Answer:**
It raises a `TypeError`. `__init__` must return `None`. This rule is strictly enforced by the Python interpreter during object instantiation.

**Why Interviewers Ask This:**
To catch candidates who might confuse `__init__` (initializer) with a factory method.

---

### 19. Private Variables in Inheritance (Medium)

**Question:**
If a Parent defines `self.__private = 1` in its `__init__`, can the Child class access or override this variable in its own `__init__`?

**Expected Answer:**
No, not directly by name. Python performs "name mangling" on double-underscore attributes (changing `__private` to `_Parent__private`). The Child would effectively be creating a *new* variable `_Child__private` if it tried to set `self.__private`.

**Explanation:**
This protects the Parent's internal state from being accidentally broken by subclasses.

---

### 20. Properties in Init (Medium)

**Question:**
If a Parent class has a property with a setter, and `__init__` assigns to that property, what happens if the Child class overrides that property?

**Expected Answer:**
The Parent's `__init__` will invoke the *Child's* implementation of the property setter. This is a powerful feature of polymorphism but can be dangerous if the Child's property relies on state that hasn't been initialized yet (since the Child's `__init__` hasn't finished running).

# ============================================================ SECTION 2 — CODING QUESTIONS (10 TOTAL)

### 21. Basic Hierarchy Setup (Easy)

**Question:**
Create a class `Employee` with `name` and `salary`. Create a subclass `Manager` that adds a `department` attribute. Ensure `Manager` properly initializes the parent `Employee` class.

**Solution:**

```python
class Employee:
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary

class Manager(Employee):
    def __init__(self, name: str, salary: float, department: str):
        # Call parent init using super()
        super().__init__(name, salary)
        self.department = department

# Usage
mgr = Manager("Alice", 90000, "HR")
print(mgr.name, mgr.department)

```

**Line-by-Line Explanation:**

1. `class Employee`: Defines the base class.
2. `super().__init__(name, salary)`: The Manager forwards the relevant data to Employee logic.
3. `self.department`: Manager sets its specific data.

**Why Interviewers Ask:** To verify basic syntax comfort.

---

### 22. Extending a List (Easy)

**Question:**
Create a custom list class `LoggedList` that inherits from Python's built-in `list`. Its `__init__` should accept a standard list of items, print a log message "List initialized with X items", and then store the items.

**Solution:**

```python
class LoggedList(list):
    def __init__(self, items=None):
        if items is None:
            items = []
        # Print log before or after super call
        print(f"List initialized with {len(items)} items")
        # Initialize the actual list behavior
        super().__init__(items)

# Usage
l = LoggedList([1, 2, 3])
# Output: List initialized with 3 items
print(l) # [1, 2, 3]

```

**Common Mistakes:**
Assigning `self.items = items` instead of calling `super().__init__(items)`. Since we inherit from `list`, `self` *is* the list.

**Real App Usage:**
Custom data structures in backend processing pipelines that need auditing.

---

### 23. Simple Role-Based Access (Easy)

**Question:**
Create a `User` class (username) and an `Admin` class inheriting from `User`. The `Admin` class sets a `permissions` list to `['ALL']` in its constructor.

**Solution:**

```python
class User:
    def __init__(self, username):
        self.username = username
        self.permissions = []

class Admin(User):
    def __init__(self, username):
        super().__init__(username)
        self.permissions = ['ALL']

# Usage
a = Admin("root")
print(a.permissions) # ['ALL']

```

---

### 24. The Diamond Problem Implementation (Medium)

**Question:**
Implement a Diamond inheritance structure (`Device` -> `Phone`, `Camera` -> `SmartPhone`).

* `Device` init prints "Device Init".
* `Phone` init prints "Phone Init".
* `Camera` init prints "Camera Init".
* `SmartPhone` init prints "SmartPhone Init".
Ensure every constructor is called exactly once using `super()`.

**Solution:**

```python
class Device:
    def __init__(self):
        print("Device Init")

class Phone(Device):
    def __init__(self):
        print("Phone Init")
        super().__init__()

class Camera(Device):
    def __init__(self):
        print("Camera Init")
        super().__init__()

class SmartPhone(Phone, Camera):
    def __init__(self):
        print("SmartPhone Init")
        super().__init__()

# Usage
s = SmartPhone()
print(SmartPhone.mro())

```

**Output:**

```text
SmartPhone Init
Phone Init
Camera Init
Device Init

```

*Note: The order depends on the MRO. `Phone` comes before `Camera` because it is listed first in `class SmartPhone(Phone, Camera)`.*

**Common Mistakes:**
Not using `super()` in `Device` (stops the chain if Device were in the middle of a deeper hierarchy, though here it is top).

---

### 25. Cooperative Kwargs Propagation (Medium)

**Question:**
Create a system with a `BaseTask`.

* `NetworkMixin` expects `ip_address` in init.
* `FileMixin` expects `filename` in init.
* `DownloadTask` inherits from both.
Use `**kwargs` so `DownloadTask` can accept both arguments and pass them up correctly.

**Solution:**

```python
class BaseTask:
    def __init__(self, **kwargs):
        # Base of chain, consumes remaining kwargs or ignores them
        pass

class NetworkMixin(BaseTask):
    def __init__(self, ip_address, **kwargs):
        self.ip_address = ip_address
        print(f"Network setup: {ip_address}")
        super().__init__(**kwargs)

class FileMixin(BaseTask):
    def __init__(self, filename, **kwargs):
        self.filename = filename
        print(f"File setup: {filename}")
        super().__init__(**kwargs)

class DownloadTask(NetworkMixin, FileMixin):
    def __init__(self, **kwargs):
        print("Starting Download Task")
        super().__init__(**kwargs)

# Usage
dt = DownloadTask(ip_address="192.168.1.1", filename="data.json")

```

**Complexity:**
Time: O(N) where N is classes in MRO.
Space: O(1) extra space.

**Developer Scenario:**
Configuring Request/Response handlers in a web framework where different plugins need different config keys.

---

### 26. Custom Exception Hierarchy (Medium)

**Question:**
Create a custom exception `AppError` that accepts `message` and `error_code`. Create a subclass `DatabaseError` that sets `error_code` to 500 automatically and prefixes the message with "[DB]".

**Solution:**

```python
class AppError(Exception):
    def __init__(self, message, error_code):
        self.error_code = error_code
        super().__init__(message) # Initialize standard Exception with message

class DatabaseError(AppError):
    def __init__(self, message):
        full_message = f"[DB] {message}"
        super().__init__(full_message, error_code=500)

# Usage
try:
    raise DatabaseError("Connection failed")
except DatabaseError as e:
    print(f"{e} (Code: {e.error_code})")
# Output: [DB] Connection failed (Code: 500)

```

**Line-by-Line:**

1. `super().__init__(message)`: We call the built-in `Exception` constructor so `str(e)` works.
2. `error_code=500`: Hardcoded default for this specific subclass.

---

### 27. Immutable Data Class Inheritance (Medium)

**Question:**
Create a class `Point` inheriting from `tuple`. It should accept `x` and `y` and store them. Use `__new__`.

**Solution:**

```python
class Point(tuple):
    def __new__(cls, x, y):
        # Create the tuple object with the data
        return super().__new__(cls, (x, y))
    
    # Optional: properties for access
    @property
    def x(self): return self[0]
    @property
    def y(self): return self[1]

# Usage
p = Point(10, 20)
print(p) # (10, 20)

```

**Why Interviewers Ask:**
Tests knowledge of `__new__` vs `__init__`. If you used `__init__` here, the tuple would already be empty by the time `__init__` ran.

---

### 28. Flask-Style Config Inheritance (Medium)

**Question:**
Simulate a configuration system.

* `Config`: `DEBUG = False`
* `DevConfig(Config)`: `DEBUG = True`
* `TestConfig(Config)`: `TESTING = True`
Write a class `App` that takes a config class in `__init__`, instantiates it, and loads settings into `self.config`.

**Solution:**

```python
class Config:
    DEBUG = False
    TESTING = False

class DevConfig(Config):
    DEBUG = True

class App:
    def __init__(self, config_class):
        # Instantiate the config class
        self.config_obj = config_class()
        # Load attributes that don't start with __
        self.config = {k: v for k, v in self.config_obj.__class__.__dict__.items() 
                       if not k.startswith('__')}
        # Also grab parent config values (simplified)
        for k in dir(self.config_obj):
             if not k.startswith('__'):
                 self.config[k] = getattr(self.config_obj, k)

# Usage
app = App(DevConfig)
print(app.config['DEBUG']) # True

```

**Real App Usage:**
Flask and Django configuration management works exactly like this (loading classes as config objects).

---

### 29. Abstract Factory Pattern Init (Medium)

**Question:**
Define an abstract class `Vehicle` with an abstract method `engine_type`. Create `Car` and `Bike` that inherit from `Vehicle`. `Vehicle.__init__` should log "Vehicle Created". Ensure children call it.

**Solution:**

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    def __init__(self, brand):
        self.brand = brand
        print(f"Vehicle Created: {brand}")

    @abstractmethod
    def engine_type(self):
        pass

class Car(Vehicle):
    def __init__(self, brand, model):
        super().__init__(brand)
        self.model = model

    def engine_type(self):
        return "V8"

# Usage
c = Car("Toyota", "Camry")
# Output: Vehicle Created: Toyota

```

**Why Interviewers Ask:**
Combines `ABC`, inheritance, and constructors.

---

### 30. Threading Inheritance (Medium)

**Question:**
Create a class `Worker` that inherits from `threading.Thread`. The `__init__` must accept a `task_id` and initialize the thread properly so `start()` works.

**Solution:**

```python
import threading
import time

class Worker(threading.Thread):
    def __init__(self, task_id):
        # CRITICAL: Must initialize the internal Thread logic
        super().__init__()
        self.task_id = task_id

    def run(self):
        print(f"Processing task {self.task_id}")
        time.sleep(0.1)

# Usage
w = Worker(101)
w.start() # This would fail if super().__init__() was missed
w.join()

```

**Common Mistakes:**
Forgetting `super().__init__()` in a `Thread` subclass causes `RuntimeError: thread.__init__() not called` when `start()` is invoked.

**Real App Usage:**
Background processing agents in Python services.
