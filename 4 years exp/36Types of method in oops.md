# Python Types of Methods in OOP — Interview Guide (30 Questions)

This guide is designed for a Python developer with 4 years of experience looking to master the three specific types of methods in Python Object-Oriented Programming: **Instance Methods**, **Class Methods**, and **Static Methods**.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### 1. What is the fundamental difference between an Instance Method, a Class Method, and a Static Method in Python? (Easy)

**Expected Answer:**

* **Instance Methods:** Take `self` as the first parameter. They can access and modify both instance state (`self.x`) and class state (`self.__class__.y`).
* **Class Methods:** Decorated with `@classmethod` and take `cls` as the first parameter. They can access and modify class state but generally cannot access specific instance state.
* **Static Methods:** Decorated with `@staticmethod` and take no implicit first parameter (neither `self` nor `cls`). They behave like regular functions belonging to a class namespace and cannot access instance or class state directly unless explicitly passed.

**Clear Explanation:**
Python methods are distinguished by how they are bound and what implicit arguments they receive. Instance methods are bound to the object. Class methods are bound to the class. Static methods are not bound to either (in terms of receiving an argument) but are structurally attached to the class.

**Medium-depth Reasoning:**
The distinction enforces design intent. Use instance methods for object behavior, class methods for factory patterns or class-wide configuration, and static methods for utility logic that is context-independent but logically relates to the class.

**Time & Space Complexity:**
N/A (Concept).

**Common Mistakes Candidates Make:**

* Thinking `@staticmethod` can access `cls` variables automatically.
* Forgetting to add `@classmethod` and trying to use `cls`.

**Why Interviewers Ask This:**
To verify the candidate understands the basic building blocks of Python OOP and scope.

**Relevant Comparisons:**

* *Java:* Static methods in Python are similar to static methods in Java, but Python's Class methods (with `cls`) have no direct equivalent in Java (Java uses static for both, but `cls` allows polymorphism in inheritance).

**Sample Code:**

```python
class Demo:
    def instance_m(self):
        return "Instance"

    @classmethod
    def class_m(cls):
        return "Class"

    @staticmethod
    def static_m():
        return "Static"

```

**Line-by-Line Explanation:**

* `def instance_m(self):`: Standard method, receives object instance.
* `@classmethod`: Decorator marks the next function as binding to the class.
* `def class_m(cls):`: Receives the class definition `Demo` as `cls`.
* `@staticmethod`: Decorator prevents implicit argument passing.

---

### 2. How does the `self` parameter actually work in an instance method? (Easy)

**Expected Answer:**
`self` is a reference to the specific object instance calling the method. It is not a keyword, just a convention. When you call `obj.method()`, Python internally translates this to `Class.method(obj)`.

**Clear Explanation:**
When an instance method is defined, it is a function. When accessed via an instance, it becomes a "bound method" which partially applies the instance as the first argument.

**Medium-depth Reasoning:**
This explicit `self` allows Python to handle method scope dynamically. Unlike C++ or Java (`this`), Python requires you to declare it to make the scope resolution explicit.

**Common Mistakes Candidates Make:**

* Omitting `self` in the method definition definition inside a class.
* Thinking `self` is a reserved keyword (you could technically use `this` or `me`, but don't).

**Why Interviewers Ask This:**
To check if the candidate understands that methods are just functions with syntactic sugar for argument passing.

**Sample Code:**

```python
class User:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello, {self.name}"

u = User("Alice")
print(u.greet())       # Syntactic sugar
print(User.greet(u))   # What actually happens

```

**Line-by-Line Explanation:**

* `u.greet()`: Python finds `greet` in `User`, sees it's a descriptor, and binds `u` to `self`.
* `User.greet(u)`: Calls the function explicitly passing `u` as `self`.

---

### 3. What happens if you define a method inside a class without `self`, `@classmethod`, or `@staticmethod`? (Easy)

**Expected Answer:**
In Python 3, it is treated as a regular function inside the class namespace. You can call it using `Class.method()`, but if you try to call it via an instance `obj.method()`, it will raise a `TypeError` because Python tries to pass the instance as the first argument, but the function accepts zero arguments.

**Clear Explanation:**
Python 3 relaxed the rules compared to Python 2 (where it would be an "unbound method"). Now it's just a function. However, the dot operator on an instance still attempts the binding protocol.

**Medium-depth Reasoning:**
This is often a source of bugs for beginners who forget decorators. It’s useful for internal helper functions that are strictly meant to be called by the class, not instances, though `@staticmethod` is preferred for clarity.

**Common Mistakes Candidates Make:**

* Thinking it's a syntax error (it is not).
* Thinking it works like a static method automatically.

**Why Interviewers Ask This:**
To test knowledge of argument binding mechanics and Python 3 changes.

**Sample Code:**

```python
class Test:
    def plain_function():
        return "I have no self"

# Test.plain_function()  # Works
# t = Test()
# t.plain_function()     # TypeError: takes 0 args but 1 was given

```

**Line-by-Line Explanation:**

* `def plain_function():`: Defined with 0 parameters.
* `t.plain_function()`: Python implicitly calls `plain_function(t)`. Logic mismatch causes crash.

---

### 4. When should you use `@staticmethod` over a standalone module-level function? (Easy)

**Expected Answer:**
Use `@staticmethod` when the function logic is tightly coupled to the class semantically, even if it doesn't use class data. It keeps the namespace clean and organizes code logically.

**Clear Explanation:**
If a function `validate_email(email)` is only ever used within the `User` class context, putting it inside `User` as a static method makes the code more organized than floating it in the global module scope.

**Medium-depth Reasoning:**
It improves code discovery. A developer using your class knows where to find helpers related to that class.

**Common Mistakes Candidates Make:**

* Overusing static methods when the function is generic utility (e.g., date formatting) that should be in a `utils.py`.

**Why Interviewers Ask This:**
To evaluate code organization and design choices.

**Sample Code:**

```python
class MathHelper:
    @staticmethod
    def add(a, b):
        return a + b

# vs module level
def add(a, b):
    return a + b

```

**Line-by-Line Explanation:**

* `MathHelper.add(1, 2)`: Namespaced clearly under `MathHelper`.

---

### 5. Can an Instance Method access Class Variables? (Easy)

**Expected Answer:**
Yes. An instance method can access class variables via `self.variable_name` (if not shadowed by an instance variable) or explicitly via `self.__class__.variable_name` or `ClassName.variable_name`.

**Clear Explanation:**
The lookup chain for `self.x` is: 1. Instance dictionary -> 2. Class dictionary -> 3. Base classes.

**Common Mistakes Candidates Make:**

* Thinking `self.var` only looks at instance attributes.
* Accidentally assigning `self.var = val` thinking it changes the class variable (it actually creates a new instance variable shadowing the class one).

**Why Interviewers Ask This:**
To test understanding of the attribute lookup hierarchy.

**Sample Code:**

```python
class Config:
    timeout = 10

    def get_timeout(self):
        return self.timeout

c = Config()
print(c.get_timeout()) # Returns 10

```

**Line-by-Line Explanation:**

* `self.timeout`: Python looks for `timeout` in `c`. Not found. Looks in `Config`. Found 10.

---

### 6. Can a Static Method modify the Class state? (Easy)

**Expected Answer:**
Not directly via a `cls` or `self` argument, because it receives neither. However, it *can* modify class state if you explicitly reference the class name inside the method (e.g., `ClassName.attr = value`).

**Clear Explanation:**
Static methods are blind to the class context unless hardcoded. This makes them rigid if you rename the class or use inheritance, as they won't automatically adapt to the subclass.

**Why Interviewers Ask This:**
To see if you understand the limitations of static methods regarding inheritance.

**Sample Code:**

```python
class Counter:
    count = 0

    @staticmethod
    def increment():
        Counter.count += 1  # Hardcoded dependency

```

**Line-by-Line Explanation:**

* `Counter.count += 1`: Explicitly accessing global namespace `Counter`.

---

### 7. Why is `@classmethod` preferred over `@staticmethod` for Factory patterns? (Medium)

**Expected Answer:**
`@classmethod` receives the class (`cls`) as the first argument. This supports inheritance. If you subclass the factory, a `@classmethod` factory will return an instance of the *Subclass*, whereas a `@staticmethod` (or hardcoded class name) would still return the *Parent* class.

**Clear Explanation:**
Polymorphism is the key. Factories need to be aware of *which* class is calling them to instantiate the correct object type.

**Medium-depth Reasoning:**
This is crucial in backend frameworks (like Django or Pydantic) where `Model.from_json()` must return a specific model instance, not a generic base object.

**Common Mistakes Candidates Make:**

* Using static methods for constructors/alternatives `__init__`.
* Hardcoding class names in factories.

**Why Interviewers Ask This:**
This is the single most important distinction for architectural design in Python.

**Sample Code:**

```python
class Base:
    @classmethod
    def factory(cls):
        return cls()

class Derived(Base):
    pass

obj = Derived.factory()
print(type(obj)) # <class '__main__.Derived'>

```

**Line-by-Line Explanation:**

* `Derived.factory()`: Calls `factory` passing `Derived` as `cls`.
* `return cls()`: Executes `Derived()`.

---

### 8. Explain the concept of "Bound" vs "Unbound" methods in Python 3. (Medium)

**Expected Answer:**
In Python 3, when you access a function on a class (`Class.fun`), it is just a **function**. When you access it on an instance (`instance.fun`), it becomes a **bound method**. The "bound method" is a wrapper that holds the instance and the function, automatically passing the instance as the first argument when called.

**Clear Explanation:**
Python 2 had "unbound methods". Python 3 removed that type. Now, `Class.fun` is strictly `function`. `instance.fun` is `method`.

**Medium-depth Reasoning:**
Understanding this helps when debugging why `method is function` returns False, or when passing methods as callbacks in event-driven programming (UI or Async).

**Diagram:**

```text
Class.method      --->  <function>
Instance.method   --->  <bound method> (wraps instance + function)

```

**Common Mistakes Candidates Make:**

* Confusing Python 2 "unbound method" errors with Python 3 behavior.

**Why Interviewers Ask This:**
To test deep knowledge of Python internals and introspection.

**Sample Code:**

```python
class A:
    def foo(self): pass

a = A()
print(A.foo) # <function ...>
print(a.foo) # <bound method ...>

```

---

### 9. Can you call an instance method from a static method? (Medium)

**Expected Answer:**
Not directly, because the static method does not have access to an instance (`self`). You would need to create a new instance inside the static method or pass an instance in as an argument.

**Clear Explanation:**
Static methods are isolated. They don't know "who" called them or if an instance exists.

**Medium-depth Reasoning:**
If you find yourself needing to call an instance method from a static method, your design is likely flawed. The method should probably be an instance method or class method.

**Why Interviewers Ask This:**
To detect code smells and design misunderstandings.

**Sample Code:**

```python
class A:
    def instance_m(self):
        print("Instance")

    @staticmethod
    def static_m(obj):
        # Must explicitly pass object
        obj.instance_m()

```

---

### 10. How does inheritance affect Class Methods vs Static Methods? (Medium)

**Expected Answer:**

* **Class Methods:** Are polymorphic. If `Child` inherits from `Parent`, calling `Child.class_method()` passes `Child` as `cls`.
* **Static Methods:** Are inherited but **not** polymorphic in the context of `cls`. They are just functions attached to the namespace. If the static method hardcodes a class name, it won't adapt.

**Clear Explanation:**
Class methods adapt to the context of the caller. Static methods are static (unchanging) regardless of who calls them, unless overridden/shadowed.

**Common Mistakes Candidates Make:**

* Expecting static methods to "know" they are being called from a subclass.

**Why Interviewers Ask This:**
To verify understanding of inheritance mechanics in Python's MRO.

**Sample Code:**

```python
class P:
    @classmethod
    def who_am_i(cls):
        print(cls.__name__)

    @staticmethod
    def plain():
        print("I am static")

class C(P):
    pass

C.who_am_i() # Prints "C" (Dynamic)
C.plain()    # Prints "I am static" (Fixed)

```

---

### 11. What is the behavior of `property` decorators regarding methods? (Medium)

**Expected Answer:**
The `@property` decorator transforms an instance method so that it can be accessed like an attribute (without parentheses). It effectively turns a method into a "getter".

**Clear Explanation:**
It implements the Descriptor Protocol. It allows computed attributes.

**Medium-depth Reasoning:**
This is vital for backward compatibility. If you start with a public variable `self.x` and later need validation, you can switch to `@property` without breaking the API (`obj.x` still works).

**Why Interviewers Ask This:**
To see if the candidate knows how to write clean, Pythonic APIs (Encapsulation).

**Sample Code:**

```python
class Circle:
    def __init__(self, r):
        self._r = r

    @property
    def area(self):
        return 3.14 * self._r ** 2

c = Circle(5)
print(c.area) # No () needed

```

---

### 12. Can you define abstract static methods or abstract class methods? (Medium)

**Expected Answer:**
Yes. Using the `abc` module.

* `@classmethod` + `@abstractmethod`
* `@staticmethod` + `@abstractmethod`

**Note:** In Python 3.3+, the decorators should be stacked with `@abstractmethod` as the innermost decorator.

**Medium-depth Reasoning:**
Useful when defining an interface where implementing classes *must* provide a specific factory or utility utility.

**Common Mistakes Candidates Make:**

* Wrong order of decorators (e.g., `@abstractmethod` on top of `@classmethod` often confuses older Python versions, though usually handled better now, standard is `@abstractmethod` inside).

**Why Interviewers Ask This:**
To test knowledge of Abstract Base Classes (ABCs) and interface enforcement.

**Sample Code:**

```python
from abc import ABC, abstractmethod

class Base(ABC):
    @classmethod
    @abstractmethod
    def create(cls):
        pass

```

---

### 13. How do you call a method of a Parent class from a Child class effectively? (Medium)

**Expected Answer:**
Using `super()`.
`super().method_name()` delegates the call to the next class in the Method Resolution Order (MRO).

**Clear Explanation:**
`super()` is safer than `Parent.method(self)` because it handles complex multiple inheritance structures correctly (the Diamond Problem).

**Medium-depth Reasoning:**
Directly calling `Parent.method(self)` hardcodes the hierarchy. `super()` is dynamic and follows the MRO computed at runtime.

**Why Interviewers Ask This:**
To check experience with inheritance and maintainability.

**Sample Code:**

```python
class Child(Parent):
    def save(self):
        print("Pre-processing")
        super().save()

```

---

### 14. Can a method be both a Class Method and an Instance Method? (Medium)

**Expected Answer:**
Not simultaneously in standard syntax. However, you can technically inspect the arguments to simulate this behavior (checking if first arg is a class or instance), or use descriptors to create hybrid methods (though this is advanced usage).

**Real-world context:**
Libraries like SQLAlchemy sometimes use hybrids that behave differently on Class vs Instance.

**Why Interviewers Ask This:**
To see if the candidate understands the boundaries of the language.

---

### 15. What is the performance difference between calling an Instance Method vs a Static Method? (Medium)

**Expected Answer:**
Static methods are marginally faster because they avoid the overhead of binding `self`.

**Clear Explanation:**
When you call `obj.instance_method()`, Python instantiates a bound method object. `obj.static_method()` returns the function directly (descriptor protocol returns function as is).

**Medium-depth Reasoning:**
The difference is negligible for 99% of applications. Optimizing for this is usually premature optimization. Readability (`@staticmethod` implies "no state access") is the primary reason to use them, not speed.

**Why Interviewers Ask This:**
To test if the candidate obsesses over micro-optimizations vs code clarity.

---

### 16. How do methods interact with Python's "Descriptor Protocol"? (Medium)

**Expected Answer:**
All Python methods are descriptors. They implement `__get__`.
When accessed via `instance.method`, `__get__` returns a bound method (partial with instance).
When accessed via `Class.method`, `__get__` returns the function (or unbound method in Py2).

**Clear Explanation:**
This is the "magic" that makes `self` appear automatically.

**Why Interviewers Ask This:**
This is a "deep dive" question to separate senior engineers who know *how* it works from mid-level engineers who just know *that* it works.

---

### 17. How can you dynamically add a method to a class at runtime? (Medium)

**Expected Answer:**
You can assign a function to `ClassName.method_name`.
If adding to a specific instance, you must bind it manually using `types.MethodType`.

**Sample Code:**

```python
import types

def new_method(self):
    return "Dynamic"

class A: pass

# Add to Class (affects all instances)
A.fun = new_method

# Add to Instance only
a = A()
a.only_mine = types.MethodType(new_method, a)

```

**Common Mistakes Candidates Make:**

* Assigning `a.fun = function` and expecting `self` to work. It won't; it will be treated as a standard function attribute, not a bound method.

**Why Interviewers Ask This:**
Monkey-patching scenarios in testing or dynamic plugin systems.

---

### 18. What is the role of `__new__` vs `__init__`? Are they instance or class methods? (Medium)

**Expected Answer:**

* `__new__` is effectively a **static method** (special case) that creates the instance. It takes `cls` as the first argument, but doesn't require the `@staticmethod` decorator. It returns the instance.
* `__init__` is an **instance method**. It initializes the instance returned by `__new__`.

**Clear Explanation:**
You rarely override `__new__` unless subclassing immutable types (like `str`, `int`) or implementing Singletons / Metaclasses.

**Why Interviewers Ask This:**
To test knowledge of object creation lifecycle.

---

### 19. Can `@classmethod` access `__init__`? (Medium)

**Expected Answer:**
Yes. A class method can call `cls()` which triggers `__init__`.

**Scenario:**
This is how alternative constructors work.

**Sample Code:**

```python
class Date:
    def __init__(self, year, month):
        self.year = year

    @classmethod
    def from_string(cls, date_str):
        y, m = map(int, date_str.split("-"))
        return cls(y, m) # Calls __init__

```

---

### 20. What happens if you use `self` in a `@classmethod`? (Medium)

**Expected Answer:**
It works, but it's misleading. The variable name `self` or `cls` is just a convention. Python passes the **Class** object as the first argument to a `@classmethod`. If you name it `self`, it will still contain the Class, not an instance.

**Common Mistakes Candidates Make:**

* Naming the first argument of a class method `self`, causing confusion for other developers who think it's an instance method.

**Why Interviewers Ask This:**
To check code readability standards and understanding that argument names are conventional, not syntactic constraints.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### 1. Simple Calculator with Static Methods (Easy)

**Question:**
Create a class `Calculator` that performs addition and subtraction. Since these operations do not require maintaining any state (history, memory), implement them as static methods.

**Full Working Solution:**

```python
class Calculator:
    @staticmethod
    def add(a: int, b: int) -> int:
        return a + b

    @staticmethod
    def subtract(a: int, b: int) -> int:
        return a - b

# Usage
print(Calculator.add(10, 5))      # 15
print(Calculator.subtract(10, 5)) # 5

```

**Line-by-Line Explanation:**

* `@staticmethod`: Marks methods as independent utility functions.
* `def add(a, b)`: No `self` or `cls` needed.
* `Calculator.add(...)`: Called directly on class without instantiation.

**Time/Space Complexity:** O(1) Time, O(1) Space.
**Why Ask This:** Tests basic syntax of `@staticmethod`.
**Real App Usage:** Helper math libraries in Game Development or Finance apps.

---

### 2. Employee Counter using Class Variables and Methods (Easy)

**Question:**
Create an `Employee` class. Every time a new `Employee` is created (`__init__`), increment a global counter. Add a class method `get_total_employees()` to return this count.

**Full Working Solution:**

```python
class Employee:
    count = 0  # Class Variable

    def __init__(self, name: str):
        self.name = name
        Employee.count += 1

    @classmethod
    def get_total_employees(cls) -> int:
        return cls.count

# Usage
e1 = Employee("John")
e2 = Employee("Doe")
print(Employee.get_total_employees()) # 2

```

**Line-by-Line Explanation:**

* `count = 0`: Defined in class scope.
* `Employee.count += 1`: Modifies the shared class variable.
* `@classmethod`: Used because we are accessing class state (`cls.count`).

**Time/Space Complexity:** O(1).
**Why Ask This:** Tests tracking global state across instances.

---

### 3. Instance Method for Data Update (Easy)

**Question:**
Create a class `Product` with `price` and `stock`. Write an instance method `sell(quantity)` that reduces stock if available and returns the total price. Raise ValueError if out of stock.

**Full Working Solution:**

```python
class Product:
    def __init__(self, price: float, stock: int):
        self.price = price
        self.stock = stock

    def sell(self, quantity: int) -> float:
        if quantity > self.stock:
            raise ValueError("Not enough stock")
        self.stock -= quantity
        return self.price * quantity

# Usage
p = Product(10.0, 5)
print(p.sell(2)) # 20.0
print(p.stock)   # 3

```

**Why Ask This:** Core OOP behavior—encapsulating state modification.
**Scenario:** E-commerce Inventory Management.

---

### 4. Alternative Constructor (Factory Pattern) (Medium)

**Question:**
Create a `User` class with `name` and `email`. Provide a standard constructor.
Add a `@classmethod` named `from_csv_string` that accepts a string like `"Alice,alice@example.com"` and returns a `User` instance.

**Full Working Solution:**

```python
class User:
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

    def __repr__(self):
        return f"User({self.name}, {self.email})"

    @classmethod
    def from_csv_string(cls, csv_str: str):
        # Validation logic could go here
        name, email = csv_str.split(',')
        return cls(name, email)

# Usage
u = User.from_csv_string("Alice,alice@example.com")
print(u)

```

**Line-by-Line Explanation:**

* `cls(name, email)`: Calls the constructor of the class (or subclass) dynamically.
* Allows creation of objects from different data formats without cluttering `__init__`.

**Time Complexity:** O(N) where N is string length (split).
**Why Ask This:** Standard pattern for parsing API responses or file data.
**Real App Usage:** `datetime.fromtimestamp()`, `pandas.read_csv()`.

---

### 5. Inheritance and Method Resolution (Medium)

**Question:**
Implement a `Logger` class with a static method `log(msg)`.
Create a subclass `FileLogger` that overrides `log`.
Create a `Service` class that has a class method `set_logger(cls)` and uses it. Demonstrate how passing different classes changes behavior.

**Full Working Solution:**

```python
class Logger:
    @staticmethod
    def log(message):
        print(f"[CONSOLE] {message}")

class FileLogger(Logger):
    @staticmethod
    def log(message):
        print(f"[FILE] Writing {message} to disk")

class Service:
    logger_cls = Logger

    @classmethod
    def set_logger(cls, logger_class):
        cls.logger_cls = logger_class

    @classmethod
    def do_work(cls):
        cls.logger_cls.log("Work started")

# Usage
Service.do_work() # [CONSOLE] Work started
Service.set_logger(FileLogger)
Service.do_work() # [FILE] Writing Work started to disk

```

**Explanation:**
This demonstrates **Dependency Injection** via class attributes. The `Service` class doesn't hardcode the logger; it uses a class variable that can be swapped using a class method.

**Why Ask This:** Tests architectural flexibility and configuration management.

---

### 6. Singleton Pattern using `__new__` (Medium)

**Question:**
Implement a `DatabaseConnection` class using the Singleton pattern (only one instance ever exists). Use `__new__` (a special static method behavior) to control this.

**Full Working Solution:**

```python
class DatabaseConnection:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            print("Creating new connection...")
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self.status = "Connected"

# Usage
d1 = DatabaseConnection()
d2 = DatabaseConnection()
print(d1 is d2) # True

```

**Line-by-Line Explanation:**

* `_instance`: Class variable to hold the single instance.
* `__new__`: Intercepts creation. checks if `_instance` exists. If not, creates it. If yes, returns existing.

**Common Mistakes:** Putting logic in `__init__` (which runs every time `__new__` returns an instance) instead of `__new__`.

**Scenario:** Database pools, Configuration loaders.

---

### 7. Validator Utility Mixin (Medium)

**Question:**
Create a Mixin class `ValidatorMixin` with a static method `is_valid_email(email)`.
Create a `User` class that inherits from this Mixin and uses the static method inside its `__init__` to validate the email.

**Full Working Solution:**

```python
import re

class ValidatorMixin:
    @staticmethod
    def is_valid_email(email: str) -> bool:
        # Simple regex for demonstration
        return bool(re.match(r"[^@]+@[^@]+\.[^@]+", email))

class User(ValidatorMixin):
    def __init__(self, email):
        if not self.is_valid_email(email):
            raise ValueError("Invalid Email Format")
        self.email = email

# Usage
try:
    u = User("bad-email")
except ValueError as e:
    print(e) # Invalid Email Format

```

**Explanation:**
Using inheritance to share static utility methods across different classes (e.g., `User`, `Admin`, `Customer` could all use `ValidatorMixin`).

**Why Ask This:** Tests Mixin usage and code reusability.

---

### 8. Handling "State" in Class Methods (Medium)

**Question:**
Implement a class `APIClient`. It has a class variable `base_url`.

1. A `@classmethod` `configure(url)` to set the URL.
2. An instance method `get(endpoint)` that makes a request to `{base_url}/{endpoint}`.
3. Show that changing configuration affects all instances.

**Full Working Solution:**

```python
class APIClient:
    base_url = "http://localhost"

    @classmethod
    def configure(cls, url):
        cls.base_url = url

    def get(self, endpoint):
        return f"Fetching {self.base_url}/{endpoint}"

# Usage
client1 = APIClient()
client2 = APIClient()

print(client1.get("users")) # Fetching http://localhost/users

APIClient.configure("https://production.com")

# Both instances update because they reference the class var
print(client1.get("users")) # Fetching https://production.com/users
print(client2.get("users")) # Fetching https://production.com/users

```

**Why Ask This:** Tests understanding of shared state vs instance state.
**Scenario:** Configuring global API endpoints or feature flags in a microservice.

---

### 9. Polymorphic Factory with Registration (Medium)

**Question:**
Create a `MediaLoader` class.
It should have a registry (dict) mapping file extensions to classes.
Implement `register_loader(cls, ext)` as a class method.
Implement `get_loader(cls, filename)` that looks at the file extension and returns an instance of the appropriate subclass.

**Full Working Solution:**

```python
class MediaLoader:
    _registry = {}

    @classmethod
    def register(cls, extension):
        def wrapper(subclass):
            cls._registry[extension] = subclass
            return subclass
        return wrapper

    @classmethod
    def get_loader(cls, filename):
        ext = filename.split('.')[-1]
        loader_cls = cls._registry.get(ext)
        if not loader_cls:
            raise ValueError("No loader found")
        return loader_cls()

    def load(self):
        raise NotImplementedError

@MediaLoader.register("jpg")
class JpgLoader(MediaLoader):
    def load(self): return "Loading JPG"

@MediaLoader.register("mp4")
class Mp4Loader(MediaLoader):
    def load(self): return "Loading MP4"

# Usage
loader = MediaLoader.get_loader("image.jpg")
print(loader.load()) # Loading JPG

```

**Explanation:**
Advanced usage of Class Methods + Decorators. The `register` class method acts as a decorator to register subclasses into the parent's `_registry`.

**Scenario:** Plugin systems (like Photoshop filters or VS Code extensions).

---

### 10. Abstract Method Enforcement (Medium)

**Question:**
Define an abstract class `Shape` using `ABC`.
Enforce that every subclass must implement a static method `get_type_description()` and an instance method `area()`.

**Full Working Solution:**

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @staticmethod
    @abstractmethod
    def get_type_description():
        pass

    @abstractmethod
    def area(self):
        pass

class Square(Shape):
    def __init__(self, side):
        self.side = side

    @staticmethod
    def get_type_description():
        return "A 4-sided polygon with equal sides."

    def area(self):
        return self.side * self.side

# Usage
# s = Shape() # Error: Can't instantiate abstract class
sq = Square(4)
print(sq.get_type_description())
print(sq.area())

```

**Why Ask This:** Tests enforcement of contracts in large codebases.
**Common Mistakes:** Forgetting to implement the static abstract method in the subclass (raises TypeError at instantiation).
