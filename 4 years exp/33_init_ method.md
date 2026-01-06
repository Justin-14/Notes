# Python `__init__` method — Interview Guide (30 Questions)

This guide focuses exclusively on the `__init__` method in Python, tailored for a developer with 4 years of experience. It moves beyond basic syntax into object initialization mechanics, design patterns, and best practices relevant to backend development.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (6 Questions)

#### 1. What is the primary purpose of the `__init__` method in a Python class?

* **Expected Answer:** `__init__` is the initializer method. It is automatically called after an instance of the class has been created (by `__new__`). Its primary purpose is to initialize the object's state by assigning values to instance attributes.
* **Clear Explanation:** It acts as a constructor in other languages (though technically `__new__` is the constructor). It sets up the initial state of the object.
* **Medium-Depth Reasoning:** Python separates creation (`__new__`) and initialization (`__init__`). This distinction allows for immutable types to be created correctly.
* **Time & Space Complexity:** O(1) generally, dependent on the assignments made.
* **Common Mistakes:** Returning a value from `__init__`. It must return `None`.
* **Why Interviewers Ask This:** To ensure basic understanding of the object lifecycle.
* **Sample Code:**
```python
class User:
    def __init__(self, username):
        self.username = username # State initialization

u = User("dev_user")

```



#### 2. Why is the first argument of `__init__` typically named `self`? Is it a keyword?

* **Expected Answer:** `self` is a convention, not a keyword. It represents the specific instance of the class being initialized.
* **Clear Explanation:** When you call `User()`, Python passes the newly created object instance as the first argument to `__init__` automatically.
* **Medium-Depth Reasoning:** Explicit `self` is consistent with Python's philosophy "Explicit is better than implicit." It clarifies scope (instance vs. local).
* **Common Mistakes:** Omitting `self` in the definition or using a different name (valid but confusing).
* **Why Interviewers Ask This:** To check if the candidate understands Python's explicit instance passing mechanism.
* **Sample Code:**
```python
class Dog:
    def __init__(this_dog, name): # 'this_dog' works, but 'self' is standard
        this_dog.name = name

```



#### 3. What happens if you try to return a value (e.g., `return True`) from `__init__`?

* **Expected Answer:** It raises a `TypeError`. `__init__` is only allowed to return `None`.
* **Clear Explanation:** The instantiation process expects the *object instance* to be the result. Returning something else breaks the instantiation contract.
* **Common Mistakes:** Returning `self` at the end of `__init__` (common in other languages).
* **Why Interviewers Ask This:** To test knowledge of language constraints.
* **Sample Code:**
```python
class Broken:
    def __init__(self):
        return 10 # Raises TypeError: __init__() should return None

```



#### 4. Can you overload `__init__` in Python like in Java or C++?

* **Expected Answer:** No, Python does not support method overloading by signature. You cannot have multiple `__init__` methods with different arguments.
* **Clear Explanation:** Defining a second `__init__` overwrites the first one.
* **Medium-Depth Reasoning:** To achieve similar behavior, Python uses default arguments, `*args`/`**kwargs`, or class methods acting as alternative constructors.
* **Why Interviewers Ask This:** To see if the candidate tries to apply idioms from statically typed languages to Python.
* **Sample Code:**
```python
class Data:
    # Last definition wins
    def __init__(self, x): ...
    def __init__(self, x, y): ... # This is the only active init

```



#### 5. What happens if a parent class has an `__init__` but the child class does not define one?

* **Expected Answer:** The child class inherits the parent's `__init__`. When initializing the child, the parent's `__init__` is called automatically.
* **Clear Explanation:** MRO (Method Resolution Order) looks for `__init__` in the child; if not found, it goes up the chain.
* **Common Mistakes:** Assuming you must always define `__init__` in every subclass.
* **Why Interviewers Ask This:** To test inheritance mechanics.
* **Sample Code:**
```python
class Base:
    def __init__(self):
        print("Base Init")
class Child(Base):
    pass

c = Child() # Prints "Base Init"

```



#### 6. How do you handle optional arguments in `__init__`?

* **Expected Answer:** Use default parameter values in the method signature or use `**kwargs`.
* **Clear Explanation:** Default values allow instantiation without providing every argument.
* **Medium-Depth Reasoning:** This reduces code duplication (no need for multiple constructors) and simplifies API usage.
* **Common Mistakes:** Using mutable default arguments (e.g., `list=[]`)—a classic pitfall.
* **Why Interviewers Ask This:** To check for cleaner, Pythonic API design skills.
* **Sample Code:**
```python
class Server:
    def __init__(self, host, port=8080):
        self.host = host
        self.port = port

```



---

### Medium Level (14 Questions)

#### 7. Explain the "Mutable Default Argument" trap in `__init__` and how to fix it.

* **Expected Answer:** Default arguments are evaluated only once at function definition time, not at call time. If a mutable object (like a list) is the default, all instances share that same list.
* **Clear Explanation:** Modifying the list in one instance affects all new instances that used the default.
* **Medium-Depth Reasoning:** Fix this by setting the default to `None` and initializing the mutable object inside the method body.
* **Common Mistakes:** `def __init__(self, items=[]): self.items = items`.
* **Why Interviewers Ask This:** This is the #1 "gotcha" in Python. A 4-year dev must know this.
* **Sample Code:**
```python
# BAD
def __init__(self, tags=[]):
    self.tags = tags

# GOOD
def __init__(self, tags=None):
    self.tags = tags if tags is not None else []

```


* **Line-by-line:** The "Good" version creates a *new* list every time `__init__` is called.



#### 8. What is the difference between `__new__` and `__init__`?

* **Expected Answer:** `__new__` creates the instance and returns it. `__init__` initializes the instance after it's created.
* **Clear Explanation:** `__new__` is a static method (implicitly) that takes `cls`. `__init__` is an instance method that takes `self`.
* **Medium-Depth Reasoning:** You rarely need `__new__` unless subclassing immutable types (like `str` or `int`) or implementing strict Singletons/Meta-classes.
* **Why Interviewers Ask This:** To distinguish between object allocation and initialization.
* **ASCII Diagram:**
```
Class() Call -> __new__(cls) -> Returns Instance -> __init__(self) -> Returns None

```



#### 9. How does `super().__init__()` work in multiple inheritance?

* **Expected Answer:** `super()` delegates the call to the next class in the Method Resolution Order (MRO). It ensures that all parent classes are initialized correctly, adhering to the linearization of the inheritance graph.
* **Clear Explanation:** It prevents the "Diamond Problem" where a common base class might be initialized twice if explicit calls (e.g., `Parent.__init__(self)`) were used.
* **Medium-Depth Reasoning:** `super()` does not just call the "parent"; it calls the next in line. In complex multiple inheritance, "next" might be a sibling class, not a direct parent.
* **Why Interviewers Ask This:** Essential for building complex frameworks or mixins.
* **Sample Code:**
```python
class A:
    def __init__(self):
        print("A init")
        super().__init__() # Propagates to object or next mixin

```



#### 10. Why might you define a class *without* an `__init__` method?

* **Expected Answer:** If the class is a Mixin, a pure interface, a static utility class, or a Dataclass (where `__init__` is auto-generated).
* **Medium-Depth Reasoning:** Often, simple data containers or logic grouping classes don't need explicit initialization logic.
* **Why Interviewers Ask This:** To see if the candidate over-engineers code by adding empty `__init__` methods.

#### 11. How does Python's `dataclasses` module change the role of `__init__`?

* **Expected Answer:** The `@dataclass` decorator automatically generates an `__init__` method based on type hints.
* **Clear Explanation:** You declare variables as class fields with type hints, and the decorator injects the boilerplate `__init__`.
* **Medium-Depth Reasoning:** If you need custom logic *after* auto-initialization, you use `__post_init__`.
* **Why Interviewers Ask This:** Dataclasses are standard in modern Python (3.7+).
* **Sample Code:**
```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str
    price: float
    # __init__ is auto-generated here

```



#### 12. Can you make arguments "keyword-only" in `__init__`? How?

* **Expected Answer:** Yes, by placing an asterisk `*` in the argument list or putting arguments after `*args`.
* **Medium-Depth Reasoning:** This forces the caller to be explicit, improving readability for classes with many configuration options (e.g., `Config(host="...", port=...)` vs `Config("...", ...)`).
* **Why Interviewers Ask This:** Shows attention to API design and code clarity.
* **Sample Code:**
```python
class Database:
    def __init__(self, db_name, *, connection_timeout=5, strict=True):
        ...
# Usage: Database("users", connection_timeout=10) # Valid
# Usage: Database("users", 10) # TypeError

```



#### 13. What is the implication of raising an exception inside `__init__`?

* **Expected Answer:** The instantiation fails. The object is not successfully created, and the memory allocated for the instance is eventually reclaimed (if no references leak).
* **Medium-Depth Reasoning:** It is a valid way to enforce invariants (validation). If the inputs are invalid, the object should not exist.
* **Common Mistakes:** Leaving the object in a "half-initialized" state (though usually, the variable assignment to the instance fails entirely).
* **Why Interviewers Ask This:** To check error handling strategies during object construction.

#### 14. How do you handle expensive operations (like API calls or DB connections) in `__init__`?

* **Expected Answer:** **Avoid it.** `__init__` should be fast.
* **Clear Explanation:** Doing I/O in constructors makes testing difficult and slows down startup.
* **Medium-Depth Reasoning:** Use a `connect()` method or a class method factory (e.g., `await Class.create()`) for async operations. `__init__` is synchronous and cannot await coroutines.
* **Why Interviewers Ask This:** Architectural awareness. Blocking in `__init__` is a bad practice.

#### 15. Explain how `**kwargs` in `__init__` supports forward compatibility.

* **Expected Answer:** It allows the class to accept new arguments in the future without breaking existing calls, or to pass arguments up to a superclass without knowing what they are.
* **Clear Explanation:** Essential for Mixins or wrapper classes.
* **Sample Code:**
```python
class BaseWrapper:
    def __init__(self, **kwargs):
        self.meta = kwargs.get('meta')
        super().__init__(**kwargs) # Forward unknown args up

```



#### 16. How does `__init__` interact with `__slots__`?

* **Expected Answer:** `__init__` must assign only to attributes defined in `__slots__`.
* **Medium-Depth Reasoning:** `__slots__` prevents the creation of `__dict__`, saving memory. Attempting to assign a variable in `__init__` that isn't in `__slots__` raises an `AttributeError`.
* **Why Interviewers Ask This:** Optimization knowledge.

#### 17. Can you assign attributes to `self` in `__init__` that were not passed as arguments?

* **Expected Answer:** Yes, absolutely.
* **Medium-Depth Reasoning:** This is common for internal state tracking (e.g., `self._is_connected = False`, `self._cache = {}`).
* **Why Interviewers Ask This:** To verify understanding that `__init__` isn't just a parameter mapper; it sets up the *whole* initial state.

#### 18. What is the pattern for "Private" variables in `__init__`?

* **Expected Answer:** Prefix with `_` (protected convention) or `__` (name mangling).
* **Clear Explanation:** `self._variable` implies internal use. `self.__variable` triggers name mangling to prevent subclass collision.
* **Why Interviewers Ask This:** Encapsulation in Python.
* **Sample Code:**
```python
class Account:
    def __init__(self, id):
        self.__id = id # Becomes _Account__id

```



#### 19. How do you implement the Singleton pattern using `__init__` limitations?

* **Expected Answer:** You cannot enforce Singleton purely in `__init__` because `__init__` is called *after* a new object is created.
* **Medium-Depth Reasoning:** You must override `__new__` to return the existing instance. If you use `__init__`, it will be called every time `__new__` returns the singleton instance, re-initializing it repeatedly unless guarded.
* **Why Interviewers Ask This:** A classic design pattern question that exposes deep knowledge of Python's instantiation steps.

#### 20. If `__init__` calls a method that is overridden in a subclass, which version runs?

* **Expected Answer:** The **subclass** version runs.
* **Medium-Depth Reasoning:** In C++, the constructor calls the base class method. In Python, methods are virtual by default. If the subclass overrides a method, `self.method()` in the parent's `__init__` calls the child's implementation.
* **Common Mistakes:** Relying on the parent method logic during initialization when it might be overridden destructively by a child.
* **Why Interviewers Ask This:** Deep understanding of polymorphism during initialization.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (3 Questions)

#### 21. Basic Class Initialization with Validation

**Question:** Create a class `Rectangle` that accepts `width` and `height`. Ensure both are positive numbers. If not, raise a `ValueError`.
**Scenario:** Basic data modeling.

**Solution:**

```python
class Rectangle:
    def __init__(self, width: float, height: float):
        if width <= 0 or height <= 0:
            raise ValueError("Dimensions must be positive")
        self.width = width
        self.height = height

# Usage
try:
    r = Rectangle(10, -5)
except ValueError as e:
    print(e) # Dimensions must be positive

```

* **Explanation:** Checks inputs before assigning state.
* **Time/Space:** O(1) / O(1).
* **Common Mistakes:** Assigning `self.width` *before* validation.

#### 22. Handling Default List correctly

**Question:** Create a `Student` class with a `name` and a list of `grades`. If no grades are provided, it should start empty.
**Scenario:** Managing mutable state.

**Solution:**

```python
from typing import List, Optional

class Student:
    def __init__(self, name: str, grades: Optional[List[int]] = None):
        self.name = name
        # Fix mutable default argument trap
        self.grades = grades if grades is not None else []

s1 = Student("Alice")
s2 = Student("Bob")
s1.grades.append(90)
# s2.grades remains []

```

* **Explanation:** Uses `None` as the default sentinel value.
* **Time/Space:** O(1) / O(1).

#### 23. Simple Composition

**Question:** Create a `Car` class that takes a `Engine` object in its `__init__`.
**Scenario:** Dependency Injection basics.

**Solution:**

```python
class Engine:
    pass

class Car:
    def __init__(self, engine: Engine):
        self.engine = engine # Dependency Injection

e = Engine()
c = Car(e)

```

* **Explanation:** Assigning a complex object to an attribute.
* **Time/Space:** O(1) / O(1).

---

### Medium Level (7 Questions)

#### 24. Flexible Configuration Loader

**Question:** Create a `ServiceConfig` class that can be initialized with explicitly named arguments, OR a dictionary of settings using `**kwargs`. Prioritize explicit arguments.
**Scenario:** Loading configuration for a backend service (e.g., from env vars or a JSON file).

**Solution:**

```python
class ServiceConfig:
    def __init__(self, host="localhost", port=80, **kwargs):
        # Explicit args take precedence (defaults provided)
        # kwargs act as a fallback or extra config storage
        self.host = host
        self.port = port
        
        # Capture extra config safely
        self.debug = kwargs.get('debug', False)
        self.extra_settings = kwargs

# Usage 1: Explicit
conf1 = ServiceConfig(host="127.0.0.1")

# Usage 2: Dictionary unpacking
data = {"host": "0.0.0.0", "port": 443, "debug": True}
conf2 = ServiceConfig(**data)

```

* **Explanation:** Using `**kwargs` allows the class to absorb dict-based data easily while keeping the signature readable.
* **Time Complexity:** O(N) where N is the number of keys in kwargs.
* **Why Ask:** Very common when parsing JSON configs in microservices.

#### 25. The "Super" Hierarchy (Diamond Problem)

**Question:** Create a Diamond inheritance structure (`A` -> `B`, `A` -> `C`, `D` inherits `B` and `C`). Use `super()` in `__init__` to ensure `A` is initialized exactly once when creating `D`.
**Scenario:** Complex framework inheritance (e.g., Django mixins).

**Solution:**

```python
class A:
    def __init__(self):
        print("Init A")

class B(A):
    def __init__(self):
        print("Init B")
        super().__init__()

class C(A):
    def __init__(self):
        print("Init C")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("Init D")
        super().__init__()

# Usage
d = D()

```

* **Explanation:**
1. `D` calls `B`'s init.
2. `B` calls `super()`. In D's MRO (`D, B, C, A`), next is `C`.
3. `C` calls `super()`. Next is `A`.
4. `A` initializes.


* Output: Init D -> Init B -> Init C -> Init A.


* **Why Ask:** Tests deep understanding of MRO and why `super()` is necessary over direct `Parent.__init__` calls.

#### 26. Read-Only Attributes via Init

**Question:** Create a class `ImmutablePoint` where `x` and `y` are set in `__init__` but cannot be changed afterwards (simulate immutability).
**Scenario:** Creating value objects or data transfer objects (DTOs).

**Solution:**

```python
class ImmutablePoint:
    def __init__(self, x, y):
        # Bypass __setattr__ if necessary, or use private vars + property
        super().__setattr__("_x", x)
        super().__setattr__("_y", y)

    @property
    def x(self):
        return self._x

    @property
    def y(self):
        return self._y

    def __setattr__(self, name, value):
        # Prevent modification after init
        raise AttributeError(f"Cannot modify immutable attribute {name}")

# Usage
p = ImmutablePoint(1, 2)
# p.x = 5  # Raises AttributeError

```

* **Explanation:** Overriding `__setattr__` generally blocks assignment. Inside `__init__`, we use `super().__setattr__` to bypass our own block to set initial values.
* **Real App Usage:** Internal configuration objects in libraries like `SQLAlchemy` or `Pydantic`.

#### 27. Dependency Injection with Type Checking

**Question:** Create a `PaymentProcessor` that requires a `DatabaseConnection` object. In `__init__`, strictly validate that the passed object is an instance of `DatabaseConnection`.
**Scenario:** Ensuring type safety in a large application without static analysis.

**Solution:**

```python
class DatabaseConnection:
    def connect(self): pass

class MockDB(DatabaseConnection):
    pass

class PaymentProcessor:
    def __init__(self, db: DatabaseConnection):
        if not isinstance(db, DatabaseConnection):
            type_name = type(db).__name__
            raise TypeError(f"Expected DatabaseConnection, got {type_name}")
        
        self.db = db

# Usage
db = MockDB()
p = PaymentProcessor(db) # Works
# p = PaymentProcessor("string") # Raises TypeError

```

* **Explanation:** Explicit `isinstance` check guards against improper usage at runtime (fail fast).
* **Why Ask:** Demonstrates defensive programming.

#### 28. Alternative Constructors (Class Methods vs Init)

**Question:** Create a `Date` class with standard `__init__(year, month, day)`. Add an alternative constructor `from_string("YYYY-MM-DD")`.
**Scenario:** Parsing input data from different formats (API responses vs DB rows).

**Solution:**

```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, date_str):
        # Logic to parse string
        y, m, d = map(int, date_str.split('-'))
        # Return an instance of cls (calls __init__)
        return cls(y, m, d)

# Usage
d1 = Date(2023, 10, 1)
d2 = Date.from_string("2023-10-01")

```

* **Explanation:** Keeps `__init__` simple and focused. Complex parsing logic moves to class methods.
* **Real App Usage:** `datetime.datetime.fromtimestamp()`.

#### 29. Counting Instances via Init

**Question:** Create a class `Widget` that tracks how many instances of itself have been created and are currently alive. Increment in `__init__`.
**Scenario:** Resource pooling or metrics tracking.

**Solution:**

```python
class Widget:
    _count = 0 # Class variable

    def __init__(self):
        Widget._count += 1
        self.id = Widget._count

    @classmethod
    def get_count(cls):
        return cls._count
    
    def __del__(self):
        # Decrement on garbage collection (note: __del__ is tricky)
        Widget._count -= 1

w1 = Widget()
w2 = Widget()
print(Widget.get_count()) # 2

```

* **Explanation:** Modifies class-level state within the instance initializer.
* **Common Mistakes:** Using `self._count` instead of `Widget._count`.

#### 30. Dynamic Attribute Assignment (The "Kwargs Dump")

**Question:** Create a class `APIResponse` where `__init__` accepts any number of keyword arguments and turns them into instance attributes dynamically.
**Scenario:** Converting raw JSON responses from an API into Python objects.

**Solution:**

```python
class APIResponse:
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            # Dynamically set attributes
            setattr(self, key, value)

# Usage
data = {"status": 200, "body": "OK", "timestamp": 123456}
resp = APIResponse(**data)

print(resp.status) # 200
print(resp.body)   # OK

```

* **Explanation:** Iterates over the dictionary and uses `setattr` to inject attributes.
* **Common Mistakes:** Security risk if `kwargs` contains internal keys (like `__class__`). A production version should filter keys.
* **Real App Usage:** Simple ORMs or JSON parsers.
