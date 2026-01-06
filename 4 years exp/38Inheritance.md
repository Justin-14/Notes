# Python Inheritance — Interview Guide (30 Questions)

This comprehensive guide is designed for a Python developer with approximately 4 years of experience. It moves beyond syntax basics into architectural patterns, the Method Resolution Order (MRO), cooperative multiple inheritance, and design principles like Liskov Substitution.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Q1. What is the difference between checking types with `type()` versus `isinstance()`?

* **Difficulty:** Easy
* **Expected Answer:** `type(obj) == Class` checks for an exact type match and ignores inheritance. `isinstance(obj, Class)` checks if an object is an instance of that class OR any of its subclasses.
* **Clear Explanation:** In an inheritance hierarchy, you usually want to treat a subclass as its parent (polymorphism). `type()` breaks this behavior because it demands the types be identical. `isinstance()` supports the Liskov Substitution Principle.
* **Medium-depth Reasoning:** Using `type()` is generally considered un-Pythonic for flow control because it prevents your code from working with valid subclasses (like a Mock object in testing or a specialized version of a class).
* **Complexity:** O(1) for `type()`. `isinstance()` is O(N) where N is the depth of the inheritance chain (though highly optimized in C).
* **Common Mistakes:** Using `type(x) == list` instead of `isinstance(x, list)` to check for a list, which fails if the user passes a custom `MyList` subclass.
* **Why Interviewers Ask This:** To see if you understand polymorphism.
* **ASCII Diagram:**
```text
Animal
  ^
  |
 Dog

isinstance(dog, Animal) -> True
type(dog) == Animal     -> False

```


* **Sample Code:**
```python
class Animal: pass
class Dog(Animal): pass

d = Dog()

print(isinstance(d, Animal)) # True (Correct for polymorphism)
print(type(d) == Animal)     # False (Too restrictive)

```


* **Line-by-line Explanation:**
* `class Dog(Animal)` defines a subclass.
* `isinstance` traverses the MRO to find `Animal`.
* `type(d)` returns `<class '__main__.Dog'>`, which is not equal to `<class '__main__.Animal'>`.



---

### Q2. Explain the syntax and purpose of `super()`.

* **Difficulty:** Easy
* **Expected Answer:** `super()` returns a proxy object that delegates method calls to a parent or sibling class of type based on the Method Resolution Order (MRO). It is primarily used to call methods (often `__init__`) defined in a superclass to ensure proper initialization or behavior extension.
* **Clear Explanation:** It avoids hardcoding parent class names (e.g., `Parent.__init__(self)`), which makes code maintainable and supports multiple inheritance correctly.
* **Medium-depth Reasoning:** Without `super()`, if you change the parent class, you must manually update all hardcoded references.
* **Complexity:** O(N) traversal of MRO.
* **Common Mistakes:** Forgetting to pass arguments to `super().__init__(args)` or confusing it with Java's `super`.
* **Why Interviewers Ask This:** Essential for writing maintainable OOP code.
* **Sample Code:**
```python
class Base:
    def greet(self):
        print("Hello from Base")

class Derived(Base):
    def greet(self):
        super().greet()  # Delegates to Base
        print("Hello from Derived")

```


* **Line-by-line Explanation:**
* `super().greet()` finds the next class in the MRO after `Derived` (which is `Base`) and calls its `greet`.



---

### Q3. What is the Method Resolution Order (MRO) in Python?

* **Difficulty:** Medium
* **Expected Answer:** MRO is the order in which Python searches for a base class when executing a method. Python 3 uses the C3 Linearization algorithm.
* **Clear Explanation:** It ensures that a class always appears before its parents, and if there are multiple parents, they are checked in the order listed, keeping the order of parents of parents consistent.
* **Medium-depth Reasoning:** This is crucial in multiple inheritance to prevent the "Diamond Problem" (ambiguity) and ensures that every class in the hierarchy is initialized exactly once if `super()` is used correctly.
* **Complexity:** The MRO is calculated at class creation time.
* **Common Mistakes:** Assuming MRO is just "Depth First Search" (it was in old Python 2 Classic classes, but not in Python 3).
* **Why Interviewers Ask This:** To check deep understanding of multiple inheritance internals.
* **Sample Code:**
```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.mro())
# Output: [D, B, C, A, object]

```


* **Line-by-line Explanation:**
* Python linearizes the hierarchy. It looks at `D`, then `B`, then `C`, then `A`, then `object`.



---

### Q4. How does Python handle the "Diamond Problem"?

* **Difficulty:** Medium
* **Expected Answer:** Python handles it via the C3 Linearization algorithm. It merges the inheritance lists of all parent classes to create a consistent linear order (the MRO).
* **Clear Explanation:** In a Diamond (D inherits B and C, both inherit A), Python ensures `A` is called only after both `B` and `C` have been visited (if `super()` is used), or allows specific selection via MRO.
* **Medium-depth Reasoning:** In languages like C++, this can cause ambiguity or duplicate objects. Python resolves this by strictly ordering the classes.
* **Common Mistakes:** Thinking Python throws an error (it doesn't, unless the MRO cannot be resolved consistently).
* **Why Interviewers Ask This:** Classic OOP theory question applied to Python.
* **ASCII Diagram:**
```text
  A
 / \
B   C
 \ /
  D

```


* **Sample Code:**
```python
class A:
    def call(self): print("A")
class B(A):
    def call(self):
        print("B")
        super().call()
class C(A):
    def call(self):
        print("C")
        super().call()
class D(B, C):
    def call(self):
        print("D")
        super().call()

D().call()

```


* **Line-by-line Explanation:**
* Output will be D -> B -> C -> A.
* Note that `B`'s super calls `C`, not `A`. This is cooperative multiple inheritance.



---

### Q5. What are Mixins and why use them?

* **Difficulty:** Medium
* **Expected Answer:** A Mixin is a class designed to provide specific functionality to other classes through inheritance but is not intended to stand alone or imply an "is-a" relationship strictly.
* **Clear Explanation:** They allow horizontal composition of behavior. For example, a `JsonSerializableMixin` adds a `.to_json()` method to any class.
* **Medium-depth Reasoning:** They solve the limitation of single inheritance (in languages that have it) and allow modular code reuse in Python without deep hierarchies.
* **Common Mistakes:** Adding state (`__init__`) to Mixins which can conflict with the main class.
* **Why Interviewers Ask This:** Tests architectural patterns beyond simple inheritance.
* **Sample Code:**
```python
class LogMixin:
    def log(self, msg):
        print(f"[LOG]: {msg}")

class User(LogMixin):
    def save(self):
        self.log("User saved")

User().save()

```


* **Line-by-line Explanation:**
* `User` inherits from `LogMixin`. It gains the `log` method without `LogMixin` needing to know anything about `User`.



---

### Q6. Difference between Inheritance and Composition?

* **Difficulty:** Easy
* **Expected Answer:** Inheritance implies an "is-a" relationship (Dog is an Animal). Composition implies a "has-a" relationship (Car has an Engine).
* **Clear Explanation:** Inheritance creates a tight coupling; changes in the parent affect the child. Composition delegates work to an internal object, offering more flexibility.
* **Medium-depth Reasoning:** The general advice is "Favor Composition over Inheritance" because it reduces fragility and makes testing easier (dependency injection).
* **Common Mistakes:** Overusing inheritance to share code between unrelated concepts (e.g., inheriting `User` from `UtilityFunctions`).
* **Why Interviewers Ask This:** Standard design pattern question.
* **Sample Code:**
```python
# Composition
class Engine:
    def start(self): pass

class Car:
    def __init__(self):
        self.engine = Engine() # Has-a

```



---

### Q7. What is an Abstract Base Class (ABC)?

* **Difficulty:** Medium
* **Expected Answer:** An ABC is a class that cannot be instantiated and is used to define a common interface for subclasses. In Python, this is done using the `abc` module.
* **Clear Explanation:** It enforces that derived classes implement specific methods decorated with `@abstractmethod`.
* **Medium-depth Reasoning:** Python is dynamically typed, so interfaces aren't strict. ABCs provide a formal way to define interfaces, allowing tools (and runtime) to catch missing implementations.
* **Common Mistakes:** Forgetting to inherit from `ABC` or `ABCMeta`, or thinking just defining a method that raises `NotImplementedError` makes it an ABC (it doesn't prevent instantiation).
* **Why Interviewers Ask This:** To see if you know how to build contracts/interfaces.
* **Sample Code:**
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self): pass

# s = Shape() # Raises TypeError

```



---

### Q8. How do `__slots__` interact with inheritance?

* **Difficulty:** Medium
* **Expected Answer:** `__slots__` are used to save memory by preventing the creation of `__dict__`. If a parent has slots and a child does not, the child will generate a `__dict__`, negating the memory benefits.
* **Clear Explanation:** To maintain memory optimization, the child class must also define `__slots__` (even if empty).
* **Medium-depth Reasoning:** Slots fundamentally change the object layout. Inheritance merges slots from parents.
* **Complexity:** Reduces memory overhead significantly for millions of small objects.
* **Common Mistakes:** Assuming slots in the parent automatically restrict the child.
* **Why Interviewers Ask This:** Performance optimization knowledge.
* **Sample Code:**
```python
class Parent:
    __slots__ = ('x',)

class Child(Parent):
    # Without this, Child has __dict__
    __slots__ = ('y',)

```


* **Line-by-line Explanation:**
* `Child` instances will have storage for `x` (from Parent) and `y`, and no `__dict__`.



---

### Q9. What happens if you inherit from `dict` vs `UserDict`?

* **Difficulty:** Medium
* **Expected Answer:** Inheriting from built-in types (like `dict`, `list`) in CPython often ignores overridden methods when called internally by the C code of the class itself. `collections.UserDict` is a pure Python wrapper designed to be inherited safely.
* **Clear Explanation:** If you override `__setitem__` in a subclass of `dict`, the `update()` method (which is C optimized) might ignore your override. `UserDict` solves this.
* **Medium-depth Reasoning:** This is a "gotcha" in Python. Built-ins maximize performance; `User*` classes maximize extensibility.
* **Common Mistakes:** Subclassing `dict` directly to create a custom mapping and wondering why `update` doesn't trigger custom logic.
* **Why Interviewers Ask This:** Deep knowledge of Python internals vs standard library tools.
* **Sample Code:**
```python
class MyDict(dict):
    def __setitem__(self, k, v):
        print("Setting")
        super().__setitem__(k, v)

d = MyDict()
d.update({1: 2}) # Might NOT print "Setting" depending on Python version/impl

```



---

### Q10. What is "Cooperative Multiple Inheritance"?

* **Difficulty:** Medium
* **Expected Answer:** It is a pattern where classes are designed to be used in a multiple inheritance scenario by consistently calling `super()` in their methods.
* **Clear Explanation:** "Cooperative" means every class in the chain agrees to forward the call to the next class in the MRO, even if they don't know who that class is.
* **Medium-depth Reasoning:** If one class in the middle of the MRO fails to call `super()`, the chain breaks, and subsequent parent classes are not initialized/called.
* **Common Mistakes:** Thinking `super()` calls the parent. It calls the *next in line*.
* **Why Interviewers Ask This:** To ensure you can write mixins that play nicely with others.

---

### Q11. Can private attributes (starting with `__`) be accessed in a subclass?

* **Difficulty:** Easy
* **Expected Answer:** No, not directly by name.
* **Clear Explanation:** Double underscore attributes undergo "Name Mangling". `__attr` in class `A` becomes `_A__attr`. The subclass `B` cannot access `self.__attr` because it looks for `_B__attr`.
* **Medium-depth Reasoning:** This is intended to prevent accidental variable overwriting in subclasses, not strictly for security "privacy".
* **Common Mistakes:** Trying to access `self.__private` in a child class and getting `AttributeError`.
* **Why Interviewers Ask This:** Python specific scoping rules.
* **Sample Code:**
```python
class A:
    def __init__(self): self.__secret = 1

class B(A):
    def get_secret(self):
        # return self.__secret  # AttributeError
        return self._A__secret # Works (Mangling)

```



---

### Q12. Explain the Liskov Substitution Principle (LSP) in the context of Python inheritance.

* **Difficulty:** Medium
* **Expected Answer:** LSP states that objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.
* **Clear Explanation:** If `Bird` has a `fly()` method, and `Penguin` inherits from `Bird` but cannot fly, `Penguin` violates LSP.
* **Medium-depth Reasoning:** Violating LSP leads to fragile code littered with `if isinstance(obj, Penguin): don't_fly()`.
* **Common Mistakes:** Forcing inheritance where it doesn't fit semantically.
* **Why Interviewers Ask This:** Solid principles distinguish seniors from juniors.

---

### Q13. How does `__new__` interact with inheritance compared to `__init__`?

* **Difficulty:** Medium
* **Expected Answer:** `__new__` is the actual constructor (creates the instance), while `__init__` is the initializer (sets up attributes). `__new__` is static.
* **Clear Explanation:** In inheritance, if you are subclassing an immutable type (like `str`, `int`, `tuple`), you MUST override `__new__` to customize instance creation, because `__init__` is too late (the object already exists and is immutable).
* **Medium-depth Reasoning:** `__new__` must return the instance. If it doesn't return an instance of the class, `__init__` is not called.
* **Why Interviewers Ask This:** Advanced object lifecycle knowledge.
* **Sample Code:**
```python
class UpperStr(str):
    def __new__(cls, value):
        return super().__new__(cls, value.upper())

print(UpperStr("hello")) # HELLO

```



---

### Q14. What are the arguments passed to `super()`?

* **Difficulty:** Medium
* **Expected Answer:** `super([type, [object-or-type]])`.
* **Clear Explanation:** In Python 3 inside a class, `super()` (no args) is syntactic sugar for `super(__class__, self)`.
* **Medium-depth Reasoning:** You can pass arguments manually to skip parts of the MRO. `super(MyClass, self)` starts searching *after* `MyClass`.
* **Common Mistakes:** Using `super(self.__class__, self)` which is buggy if the class is further subclassed (recursion error).
* **Why Interviewers Ask This:** Understanding the mechanics behind the magic.

---

### Q15. Is it possible to inherit from a function?

* **Difficulty:** Easy (Trick Question)
* **Expected Answer:** No. You can only inherit from types (classes).
* **Clear Explanation:** However, you can make a class *callable* like a function by implementing `__call__`, and you can inherit from that class.
* **Why Interviewers Ask This:** To check basic terminology.

---

### Q16. What is the difference between `@classmethod` and `@staticmethod` in inheritance?

* **Difficulty:** Easy
* **Expected Answer:** `@classmethod` takes `cls` as the first arg; `@staticmethod` takes no implicit first arg.
* **Clear Explanation:** If you call a class method on a subclass, `cls` will be the *subclass*, not the parent. This allows "Factory methods" to create instances of the correct subclass. Static methods behave the same regardless of inheritance context (like a plain function in a namespace).
* **Medium-depth Reasoning:** Use class methods when the logic relies on the class itself (e.g., alternative constructors).
* **Sample Code:**
```python
class Base:
    @classmethod
    def create(cls): return cls()

class Derived(Base): pass

print(type(Derived.create())) # <class 'Derived'>

```



---

### Q17. How do you implement a "Final" class (cannot be inherited) in Python?

* **Difficulty:** Medium
* **Expected Answer:** Historically via conventions or metaclasses. Since Python 3.8, you can use `typing.final` decorator.
* **Clear Explanation:** `@final` indicates to static type checkers (mypy) that the class should not be subclassed. It does not enforce it at runtime (unless you use a metaclass hack).
* **Medium-depth Reasoning:** Python is "consenting adults"; runtime enforcement is rare.
* **Why Interviewers Ask This:** Awareness of modern Python typing features.
* **Sample Code:**
```python
from typing import final

@final
class Base: pass

# Class Derived(Base): pass # Flagged by MyPy

```



---

### Q18. What is the "fragile base class" problem?

* **Difficulty:** Medium
* **Expected Answer:** A fundamental architectural problem where seemingly safe modifications to a base class cause derived classes to malfunction.
* **Clear Explanation:** Because the child implementation often depends heavily on the internal details of the parent.
* **Medium-depth Reasoning:** This is an argument for Composition over Inheritance or keeping inheritance hierarchies shallow.
* **Why Interviewers Ask This:** System design considerations.

---

### Q19. How do you call a method from a specific parent class, bypassing MRO?

* **Difficulty:** Medium
* **Expected Answer:** Call the method directly on the class and pass `self` manually.
* **Clear Explanation:** `ParentClass.method(self, args)`.
* **Medium-depth Reasoning:** This is "unbound method call". It's dangerous in multiple inheritance because it ignores the cooperative chain, but useful when you strictly need a specific parent's behavior.
* **Sample Code:**
```python
class A:
    def save(self): print("A save")

class B(A):
    def save(self):
        A.save(self) # Direct call

```



---

### Q20. Can you modify the MRO dynamically?

* **Difficulty:** Medium
* **Expected Answer:** Technically yes, but you shouldn't.
* **Clear Explanation:** You can modify `__bases__`. Python will re-calculate the MRO.
* **Medium-depth Reasoning:** This is extremely dangerous and confusing. It's mostly done in dynamic class generation or metaprogramming.
* **Why Interviewers Ask This:** To see if you know the boundaries of the language.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Q1. Basic Inheritance & Overriding (Easy)

**Question:** Create a class `Shape` with a method `area()` that returns 0. Create two subclasses `Rectangle` (takes width, height) and `Circle` (takes radius). Implement `area()` for both. Ensure `Circle` uses `math.pi`.

**Solution Code:**

```python
import math

class Shape:
    def area(self):
        return 0

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * (self.radius ** 2)

# Usage
shapes = [Rectangle(10, 5), Circle(7)]
for s in shapes:
    print(f"{type(s).__name__} Area: {s.area():.2f}")

```

**Line-by-line Explanation:**

1. `Shape` defines the interface (default implementation).
2. `Rectangle` overrides `__init__` to store dimensions and `area` to calculate logic.
3. `Circle` imports `math` and uses the formula .
4. Usage demonstrates polymorphism—iterating a list of generic shapes.

**Complexity:** O(1) for calculations.
**Common Mistakes:** Forgetting `self` in method definitions.
**Why asked:** Sanity check for basic OOP syntax.
**Scenario:** Graphics rendering, CAD software.
**Real App Usage:** A drawing app (Canva) handling different elements.

---

### Q2. The Diamond Problem & `super()` (Medium)

**Question:** Implement a Diamond hierarchy: `Device` (root), `Computer` (inherits Device), `Printer` (inherits Device), and `SmartPrinter` (inherits Computer, Printer). Add a method `start()` to all. Ensure that when `SmartPrinter` starts, every parent's start method is called exactly once.

**Solution Code:**

```python
class Device:
    def start(self):
        print("Device starting...")

class Computer(Device):
    def start(self):
        print("Computer starting...")
        super().start()

class Printer(Device):
    def start(self):
        print("Printer starting...")
        super().start()

class SmartPrinter(Computer, Printer):
    def start(self):
        print("SmartPrinter starting...")
        super().start()

# Usage
sp = SmartPrinter()
sp.start()
# print(SmartPrinter.mro())

```

**Line-by-line Explanation:**

1. Each class calls `super().start()`.
2. `SmartPrinter` calls `Computer`.
3. `Computer`'s `super()` (in this context) refers to `Printer` (the next sibling in MRO), NOT `Device`. This is the key.
4. `Printer`'s `super()` calls `Device`.
5. `Device` is the end of the chain.
6. This ensures `Device` starts only once.

**Complexity:** MRO resolution is computed once at class definition.
**Common Mistakes:** `Computer` calling `Device.start(self)` directly—this would cause `Device` to start twice (once via Computer, once via Printer).
**Why asked:** To test understanding of Cooperative Multiple Inheritance.
**Scenario:** Initialization of complex hardware drivers.

---

### Q3. Custom Exception Hierarchy (Easy)

**Question:** Build a custom exception hierarchy for an API. Base class `ApiException`, subclass `NotFoundException` (404), and `NotAuthorizedException` (401). Both subclasses should accept a message, but store a specific default status code.

**Solution Code:**

```python
class ApiException(Exception):
    status_code = 500

    def __init__(self, message="API Error"):
        super().__init__(message)
        self.message = message

class NotFoundException(ApiException):
    status_code = 404

class NotAuthorizedException(ApiException):
    status_code = 401

# Usage
try:
    raise NotFoundException("User ID 123 missing")
except ApiException as e:
    print(f"Error: {e.message}, Code: {e.status_code}")

```

**Line-by-line Explanation:**

1. Inherit from Python's built-in `Exception`.
2. `ApiException` sets a default behavior.
3. Subclasses override the class attribute `status_code`.
4. `try/except` catches the base `ApiException`, handling all specific API errors generically.

**Complexity:** O(1).
**Common Mistakes:** Inheriting from `BaseException` (too broad) instead of `Exception`.
**Why asked:** Exception handling is a daily task.
**Scenario:** REST API development (FastAPI/Django).
**Real App Usage:** Stripe API error handling.

---

### Q4. Mixins for Logging (Medium)

**Question:** create a `JsonMixin` that adds a `to_json()` method. Apply it to a `Product` class. The `Product` class has `name` and `price`. `to_json` should return a JSON string of the object's `__dict__`.

**Solution Code:**

```python
import json

class JsonMixin:
    def to_json(self):
        # Dump the instance dictionary to a JSON string
        return json.dumps(self.__dict__)

class Product(JsonMixin):
    def __init__(self, name, price):
        self.name = name
        self.price = price

# Usage
p = Product("Laptop", 999.99)
print(p.to_json())

```

**Line-by-line Explanation:**

1. `JsonMixin` is a standalone class providing utility.
2. It accesses `self.__dict__` generically.
3. `Product` inherits `JsonMixin`.
4. This keeps the JSON logic separate from the Product business logic.

**Complexity:** JSON serialization is O(N) relative to object size.
**Common Mistakes:** Hardcoding fields in the mixin instead of using `__dict__`.
**Why asked:** Tests code modularity and mixin usage.
**Scenario:** Adding serialization to multiple ORM models.
**Real App Usage:** Django DRF Serializers.

---

### Q5. Abstract Base Class for Payment (Medium)

**Question:** Define an abstract class `PaymentProcessor` with an abstract method `process_payment(amount)`. Implement two subclasses: `StripeProcessor` and `PayPalProcessor`.

**Solution Code:**

```python
from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float):
        """Process the payment of given amount."""
        pass

class StripeProcessor(PaymentProcessor):
    def process_payment(self, amount: float):
        print(f"Processing ${amount} via Stripe API...")
        return True

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount: float):
        print(f"Processing ${amount} via PayPal API...")
        return True

# Usage
def checkout(processor: PaymentProcessor, cost: float):
    processor.process_payment(cost)

checkout(StripeProcessor(), 100.0)

```

**Line-by-line Explanation:**

1. Inherit from `ABC`.
2. Use `@abstractmethod` to enforce the contract.
3. If `StripeProcessor` fails to implement `process_payment`, Python raises `TypeError` at instantiation.
4. `checkout` function type-hints the base class (Dependency Inversion).

**Complexity:** O(1).
**Common Mistakes:** Not inheriting from ABC.
**Why asked:** Interface design is crucial for scalable systems.
**Scenario:** E-commerce checkout systems.
**Real App Usage:** Shopify payment adapters.

---

### Q6. Cooperative `__init__` with Arguments (Medium)

**Question:** Create classes `A` and `B` that accept `**kwargs` in `__init__`. Class `C` inherits from both. Show how to pass arguments up the chain so `A` gets its needed args and `B` gets its needed args, without them knowing about each other.

**Solution Code:**

```python
class A:
    def __init__(self, a_val, **kwargs):
        super().__init__(**kwargs)
        self.a_val = a_val
        print(f"A initialized with {a_val}")

class B:
    def __init__(self, b_val, **kwargs):
        super().__init__(**kwargs)
        self.b_val = b_val
        print(f"B initialized with {b_val}")

class C(A, B):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        print("C initialized")

# Usage
# Note: A is first in MRO, so it consumes a_val and passes rest to B
c = C(a_val=10, b_val=20)

```

**Line-by-line Explanation:**

1. Every `__init__` accepts `**kwargs`.
2. They pop their specific argument or define it in signature (`a_val`) and pass `**kwargs` to `super()`.
3. `object` (the final class) has an `__init__` that takes no args, so the kwargs must be empty by the time they reach the end, or `object` will complain. (In this simple example, `object` is lenient or we assume correct usage).
4. Allows `C` to aggregate parameters for all parents.

**Complexity:** O(N) MRO traversal.
**Common Mistakes:** Not using `**kwargs` in `super().__init__`, breaking the chain.
**Why asked:** This is the correct way to initialize multiple inheritance hierarchies.
**Scenario:** Configurable UI widgets inheriting multiple behaviors.

---

### Q7. Extending Built-in `list` (Medium)

**Question:** Create a class `IndexList` that inherits from `collections.UserList`. It should behave like a normal list but if you access an index that is out of bounds, it returns `None` instead of raising `IndexError`.

**Solution Code:**

```python
from collections import UserList

class IndexList(UserList):
    def __getitem__(self, index):
        try:
            return super().__getitem__(index)
        except IndexError:
            return None

# Usage
il = IndexList([1, 2, 3])
print(il[0])  # 1
print(il[10]) # None (No crash)

```

**Line-by-line Explanation:**

1. Use `UserList` (wrapper) instead of `list` for safer overriding.
2. Override `__getitem__` (syntax `obj[i]`).
3. Call `super().__getitem__` to attempt standard retrieval.
4. Catch `IndexError` and return `None`.

**Complexity:** O(1).
**Common Mistakes:** Inheriting from `list` and forgetting that internal C methods might bypass `__getitem__`.
**Why asked:** Understanding magic methods and safe subclassing.
**Scenario:** Data processing pipelines where missing data is acceptable.
**Real App Usage:** Pandas DataFrames (loose indexing logic).

---

### Q8. Factory Pattern via Inheritance (Medium)

**Question:** Implement a parser system. `BaseParser` has a method `parse()`. Create `JsonParser` and `XmlParser`. Use a class method `get_parser(filename)` on `BaseParser` that returns the correct subclass instance based on file extension.

**Solution Code:**

```python
class BaseParser:
    @classmethod
    def get_parser(cls, filename):
        if filename.endswith(".json"):
            return JsonParser()
        elif filename.endswith(".xml"):
            return XmlParser()
        else:
            raise ValueError("Unknown format")

    def parse(self):
        raise NotImplementedError

class JsonParser(BaseParser):
    def parse(self):
        return "Parsing JSON"

class XmlParser(BaseParser):
    def parse(self):
        return "Parsing XML"

# Usage
p = BaseParser.get_parser("data.json")
print(p.parse()) # Parsing JSON
print(type(p))   # <class 'JsonParser'>

```

**Line-by-line Explanation:**

1. `BaseParser` acts as a Factory.
2. `get_parser` logic decides which child to instantiate.
3. Returns an *instance* of the subclass.
4. Client code doesn't need to import specific parsers, just the Base.

**Complexity:** O(1) factory logic.
**Common Mistakes:** Circular imports if subclasses are in different files (solve by registering classes dynamically).
**Why asked:** Design patterns.
**Scenario:** File importers.
**Real App Usage:** `Pillow` (PIL) library `Image.open()`.

---

### Q9. Immutable Child of Mutable Parent? (Medium)

**Question:** You have a mutable class `Point(x, y)`. You want to create an `ImmutablePoint` subclass. How do you prevent attribute modification in the subclass?

**Solution Code:**

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class ImmutablePoint(Point):
    def __setattr__(self, name, value):
        # Allow setting only during initialization logic if strictly needed,
        # otherwise block all.
        # Here we block all modifications after creation.
        if hasattr(self, name):
            raise AttributeError(f"Cannot modify immutable instance attribute '{name}'")
        super().__setattr__(name, value)

# Usage
ip = ImmutablePoint(1, 2)
print(ip.x)
# ip.x = 5 # Raises AttributeError

```

**Line-by-line Explanation:**

1. Override `__setattr__`.
2. Check if attribute already exists. If yes, it's a modification -> Raise Error.
3. If no (first time set in `__init__`), allow `super().__setattr__`.
4. *Note:* A better approach for new code is `dataclasses(frozen=True)`, but this tests inheritance mechanics manually.

**Complexity:** O(1) overhead on attribute set.
**Why asked:** Metaprogramming/Attribute control.
**Scenario:** Passing configuration objects that shouldn't be changed.

---

### Q10. Interface Segregation (Medium)

**Question:** You have a `MultiFunctionDevice` that prints and scans. Create interfaces `Printer` and `Scanner`. Create a class `OldPrinter` that *only* inherits `Printer`. Create `ModernMachine` that inherits both.

**Solution Code:**

```python
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print_doc(self): pass

class Scanner(ABC):
    @abstractmethod
    def scan_doc(self): pass

class OldPrinter(Printer):
    def print_doc(self):
        print("Printing...")

class ModernMachine(Printer, Scanner):
    def print_doc(self):
        print("Printing fast...")
    
    def scan_doc(self):
        print("Scanning...")

# Usage
def do_print(device: Printer):
    device.print_doc()

do_print(OldPrinter())
do_print(ModernMachine())

```

**Line-by-line Explanation:**

1. Defines small, focused interfaces (IS of SOLID).
2. `OldPrinter` isn't forced to implement `scan_doc` (which it can't do).
3. `ModernMachine` combines capabilities via multiple inheritance.

**Complexity:** O(1).
**Common Mistakes:** Creating one massive `Machine` interface with methods `print` and `scan`, forcing `OldPrinter` to implement a dummy `scan`.
**Why asked:** SOLID principles application.
**Scenario:** Hardware drivers or Role Based Access Control (RBAC).
