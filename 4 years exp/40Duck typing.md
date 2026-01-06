# Python Duck Typing — Interview Guide (30 Questions)

## Introduction

This guide is tailored for a Python developer with approximately 4 years of experience. It focuses strictly on **Duck Typing**, a core concept in Python's dynamic typing system. The philosophy is: *"If it walks like a duck and quacks like a duck, then it must be a duck."*

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (6 Questions)

#### 1. What is Duck Typing in Python and how does it differ from Static Typing?

**Expected Answer:**
Duck typing is a dynamic typing concept where the *type* or the *class* of an object is less important than the methods it defines or the attributes it possesses. If an object implements the required methods (e.g., `quack()`), it can be used in any context that expects that behavior, regardless of inheritance. In contrast, Static Typing checks types at compile-time and usually requires explicit inheritance or interface implementation.

**Explanation:**
Python does not enforce type checking at compile time. Instead, the runtime checks if the object has the specific method being called. This allows for polymorphism without a strict class hierarchy.

**Reasoning:**
This question establishes the foundational mental model. A senior dev must understand that Python relies on interfaces/protocols (implicit) rather than types (explicit).

**Why interviewers ask this:**
To verify if the candidate understands the core philosophy of Python's type system versus languages like Java or C++.

**Sample Code:**

```python
class Duck:
    def quack(self): print("Quack!")

class Person:
    def quack(self): print("I'm quacking like a duck!")

def make_it_quack(obj):
    # Doesn't care if obj is a Duck or Person, only that it has .quack()
    obj.quack()

make_it_quack(Duck())   # Works
make_it_quack(Person()) # Works

```

---

#### 2. Explain the relationship between Duck Typing and EAFP (Easier to Ask for Forgiveness than Permission).

**Expected Answer:**
Duck typing often pairs with EAFP. Instead of checking `if isinstance(obj, Duck)` (LBYL - Look Before You Leap), Pythonic code simply tries to call `obj.quack()`. If the object doesn't support it, Python raises an `AttributeError`, which is caught and handled.

**Explanation:**
EAFP assumes the object is compatible. If it isn't, the error handling mechanism resolves the flow. This avoids expensive type checks and allows any object fitting the "duck" shape to pass through.

**Common Mistakes:**
Using `isinstance` checks extensively, which defeats the purpose of duck typing and makes code rigid.

**Why interviewers ask this:**
To see if the candidate writes "Pythonic" code or tries to write C-style code in Python.

**Sample Code:**

```python
def process_data(data):
    try:
        # EAFP: Assume data behaves like a duck (has .read)
        return data.read()
    except AttributeError:
        # Handle the case where it's not a duck
        return str(data)

```

---

#### 3. How does the `len()` function utilize Duck Typing?

**Expected Answer:**
The built-in `len(obj)` function does not check if `obj` is a `list` or `string`. It simply looks for the `__len__` dunder method on the object. Any object implementing `__len__` is valid for `len()`.

**Explanation:**
This is a classic example of a Python protocol. The "Length" protocol requires only the `__len__` method.

**Time & Space Complexity:**
Generally  if `__len__` just returns a stored counter, but it depends on the implementation of the custom object.

**Why interviewers ask this:**
To ensure the candidate understands that built-ins rely on dunder methods (protocols), not hardcoded type checks.

**Sample Code:**

```python
class Garage:
    def __init__(self):
        self.cars = ["BMW", "Audi"]
    
    def __len__(self):
        return len(self.cars)

g = Garage()
print(len(g)) # Output: 2

```

---

#### 4. What is a "File-like Object" in the context of Duck Typing?

**Expected Answer:**
A file-like object is any object that behaves like a file (implements methods like `read()`, `write()`, `close()`) but isn't necessarily a file on disk. Examples include `io.StringIO`, `io.BytesIO`, or `gzip.open`.

**Reasoning:**
Backend systems often swap real files for memory buffers during testing or processing. Duck typing makes this seamless—the consuming function doesn't know the difference.

**Why interviewers ask this:**
It is a practical, real-world application of duck typing used frequently in data processing and testing.

**Sample Code:**

```python
import io

def parse_log(file_obj):
    # Works with a real file OR a StringIO buffer
    return file_obj.read().splitlines()

fake_file = io.StringIO("error: fail\ninfo: ok")
print(parse_log(fake_file))

```

---

#### 5. Can Duck Typing lead to runtime errors? How do we mitigate them?

**Expected Answer:**
Yes. Since checks happen at runtime, passing an object missing the required method raises `AttributeError` during execution. Mitigation strategies include:

1. Unit testing (crucial in dynamic languages).
2. `try-except` blocks (EAFP).
3. Type Hinting with `Protocol` (from `typing`) for static analysis tools like `mypy`.

**Common Mistakes:**
Assuming Duck Typing means "no safety." It means "runtime safety" via error handling.

**Why interviewers ask this:**
To check if the candidate is aware of the trade-offs (flexibility vs. safety) and knows how to handle the risks.

---

#### 6. What is the role of Special (Magic) Methods in Duck Typing?

**Expected Answer:**
Magic methods (e.g., `__iter__`, `__getitem__`, `__call__`) define the "shape" of the duck. By implementing these, a custom object automatically works with Python's syntax (operators, loops, generic functions) just like built-in types.

**Explanation:**
Duck typing in Python is largely implemented via these magic methods. You don't implement an interface named `Iterable`; you implement `__iter__`.

**Why interviewers ask this:**
To verify knowledge of Python's data model.

**Sample Code:**

```python
class Adder:
    def __call__(self, x, y):
        return x + y

add = Adder()
print(add(5, 10)) # Object behaves like a function "duck"

```

---

### Medium Level (14 Questions)

#### 7. How does the `typing.Protocol` (introduced in Python 3.8) relate to Duck Typing?

**Question:**
Python introduced `typing.Protocol`. Is this a move towards static typing, and how does it interact with Duck Typing?

**Expected Answer:**
`typing.Protocol` formalizes Duck Typing for static analysis tools (like mypy). It is "Structural Subtyping." You define a Protocol class with methods; any object having those methods is considered a match by the type checker, *without* explicitly inheriting from the Protocol. At runtime, it behaves exactly like standard duck typing.

**Medium-depth Reasoning:**
It bridges the gap. It keeps the flexibility of duck typing (no inheritance required) but adds the safety of compile-time checking if you use a linter.

**Common Mistakes:**
Thinking one must inherit from `Protocol` for the code to run. It's purely for type checkers.

**Sample Code:**

```python
from typing import Protocol

class Flyer(Protocol):
    def fly(self) -> None: ...

class Bird:
    def fly(self): print("Flap flap")

def trigger_flight(entity: Flyer):
    entity.fly()

# Bird passes static check despite not inheriting from Flyer
trigger_flight(Bird()) 

```

---

#### 8. Compare Duck Typing with Abstract Base Classes (ABCs). When would you use `abc` over pure Duck Typing?

**Question:**
Python has `abc` module. If Duck Typing is preferred, why do ABCs exist?

**Expected Answer:**
ABCs (`collections.abc`) provide a middle ground.

1. **Verification:** `isinstance(obj, Sequence)` checks if `obj` inherits from `Sequence` or registers as one.
2. **Strictness:** ABCs can enforce method implementation via `@abstractmethod`.
3. **Usage:** Use ABCs when you need a contract guarantee or when building a framework where "implicit" behavior is too risky or ambiguous. Duck typing is for flexibility; ABCs are for structure.

**Why interviewers ask this:**
To see if the candidate knows when *not* to use pure duck typing. Complex systems often need the rigidity of ABCs.

---

#### 9. How does Duck Typing enable the "Iterator Protocol"?

**Question:**
Explain how a custom object becomes iterable using Duck Typing principles.

**Expected Answer:**
To be iterable (usable in `for` loops), an object acts like an "Iterable Duck." It must implement `__iter__` which returns an iterator. The iterator itself is another duck that must implement `__next__` (and `__iter__`).

**Explanation:**
The `for` loop doesn't check type. It calls `iter(obj)`. If `__iter__` is missing, Python attempts `__getitem__` starting at index 0 (legacy duck typing support).

**Sample Code:**

```python
class Countdown:
    def __init__(self, start):
        self.count = start
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count <= 0:
            raise StopIteration
        self.count -= 1
        return self.count + 1

# Works in loop because it quacks like an iterator
for x in Countdown(3):
    print(x)

```

---

#### 10. Explain Duck Typing in the context of Python's Context Managers (`with` statement).

**Question:**
How does the `with` statement rely on Duck Typing?

**Expected Answer:**
The `with` statement expects an object that implements the "Context Manager" protocol: `__enter__` and `__exit__`. Any class providing these two methods can be used in a `with` block, regardless of what the object actually is (database connection, file, lock, custom timer).

**Why interviewers ask this:**
Context managers are vital for resource management.

**Sample Code:**

```python
class HTMLTag:
    def __init__(self, tag):
        self.tag = tag
    def __enter__(self):
        print(f"<{self.tag}>")
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"</{self.tag}>")

with HTMLTag("h1"):
    print("Title")
# Output: 
# <h1>
# Title
# </h1>

```

---

#### 11. What is "Monkey Patching" and how does it rely on Duck Typing?

**Question:**
Developers often Monkey Patch libraries during testing. How is this possible due to Duck Typing?

**Expected Answer:**
Monkey patching is dynamically modifying a class or module at runtime (e.g., replacing a method). Since Python code calls methods by name lookup (Duck Typing), if we replace `Network.connect` with `Mock.connect`, the caller doesn't know the difference as long as the arguments match.

**Common Mistakes:**
Overusing monkey patching in production code, leading to "action at a distance" bugs.

**Developer Scenario:**
Used extensively in `unittest.mock` to replace external API calls with dummy functions.

---

#### 12. How does Duck Typing affect the performance of attribute lookup?

**Question:**
Does the dynamic nature of Duck Typing (resolving attributes at runtime) incur a performance cost?

**Expected Answer:**
Yes. Every time a method is called (e.g., `obj.quack()`), Python must look up "quack" in the object's instance dictionary, then the class dictionary, then base classes (MRO). This is slower than static binding (vtable lookups) in C++.

**Medium-depth Reasoning:**
However, optimizations like the specialized opcodes in newer Python versions (3.11+) and caching mechanisms reduce this overhead. The trade-off is development speed vs. execution speed.

**Why interviewers ask this:**
To verify understanding of Python internals and performance characteristics.

---

#### 13. How does `operator` overloading relate to Duck Typing?

**Question:**
If I want to use `+` on two custom objects, what do I need to implement?

**Expected Answer:**
You need to implement the `__add__` (or `__radd__`) method. Python's `+` operator delegates to these methods. If an object has `__add__`, it "quacks" like a number (or string/list).

**Sample Code:**

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

v1 = Vector(1, 2)
v2 = Vector(3, 4)
v3 = v1 + v2 # Works because v1 supports __add__

```

---

#### 14. What is the `__getitem__` fallback mechanism in Duck Typing?

**Question:**
If a class does not implement `__iter__`, can it still be iterated over?

**Expected Answer:**
Yes. If `__iter__` is missing, Python checks for `__getitem__`. It attempts to access index 0, 1, 2... until an `IndexError` is raised. This is a legacy duck typing feature that allows sequence-like objects to be iterable without explicitly being iterators.

**Why interviewers ask this:**
This is an edge case in iteration logic that usually surprises intermediate developers.

---

#### 15. How does `json.dump` use Duck Typing?

**Question:**
Does `json.dump` accept only dicts and lists?

**Expected Answer:**
By default, yes, but it relies on duck typing for introspection. More importantly, you can provide a `default` function or a custom `JSONEncoder`. If a custom object doesn't look like a standard JSON type, `json.dump` calls the `default` method (if provided) to let the object "quack" like a serialized dict.

**Real App Usage:**
Serializing complex SQLAlchemy models or custom API response objects into JSON for HTTP responses.

---

#### 16. Explain how `__bool__` and `__len__` interact in boolean contexts.

**Question:**
When we do `if obj:`, how does Python decide the result using Duck Typing?

**Expected Answer:**

1. Python looks for `__bool__`. If found, it uses it.
2. If `__bool__` is missing, it looks for `__len__`. If `__len__` returns non-zero, it's True.
3. If neither exists, the object is True by default.

**Explanation:**
This is a "Cascading Protocol." The object is tested for specific methods to determine its "truthiness."

**Sample Code:**

```python
class Cart:
    def __len__(self): return 0

c = Cart()
if not c:
    print("Cart is empty") # Prints because len is 0

```

---

#### 17. How does `__contains__` enable the `in` operator?

**Question:**
How do we make `if item in my_object:` work?

**Expected Answer:**
Implement `__contains__(self, item)`.
If `__contains__` is missing, Python falls back to iterating over the object (using `__iter__`) and checking equality.
If `__iter__` is missing, it uses `__getitem__`.

**Performance Note:**
`__contains__` is usually  or  (hashing/trees), whereas the fallback iteration is . Implementing the specific duck method is a performance optimization.

---

#### 18. What is the difference between "Nominal Typing" and "Structural Typing" (Duck Typing)?

**Question:**
Compare these two type systems.

**Expected Answer:**

* **Nominal Typing:** Compatibility is determined by explicit declarations (names). `class Dog(Animal)` is an Animal because it says so. (Java, C#).
* **Structural Typing (Duck Typing):** Compatibility is determined by structure (methods/attributes). `Dog` is an `Animal` because it has `eat()` and `sleep()`.

**Why interviewers ask this:**
Terminology check. Senior devs should know the academic terms for the concepts they use.

---

#### 19. How do you implement a "Function-like" object?

**Question:**
How can an instance of a class be called like `my_obj()`?

**Expected Answer:**
Implement the `__call__` magic method. This allows the object to store state (attributes) but behave syntactically like a function.

**Developer Scenario:**
Common in decorators (class-based decorators) or caching strategies where the cache needs to be callable but hold memory of previous results.

---

#### 20. Implicit Interfaces vs Explicit Interfaces.

**Question:**
Duck typing relies on implicit interfaces. What are the pros and cons in a large codebase?

**Expected Answer:**

* **Pros:** Low boilerplate, high flexibility, easy testing (mocking).
* **Cons:** Harder to discover what methods an argument *needs* to have without reading the code inside the function. Tooling support (IDEs) historically struggled with this, though Type Hints helped resolve this con.

---

# ============================================================ SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (3 Questions)

#### 21. Implement a Custom Length Container

**Question:**
Create a class `Basket` that holds items. Implement the necessary Duck Typing method so that `len(my_basket)` returns the count of items.

**Solution:**

```python
class Basket:
    def __init__(self, items=None):
        self.items = items if items else []
    
    def add(self, item):
        self.items.append(item)
        
    def __len__(self):
        return len(self.items)

# Usage
b = Basket(['Apple', 'Banana'])
print(f"Size: {len(b)}")  # Output: Size: 2

```

**Explanation:**
We implement `__len__`. Python's built-in `len()` delegates to this method.
**Complexity:**  (since list stores its size).

---

#### 22. Make an Object Iterable

**Question:**
Create a class `Sentence` that takes a string of words. Make it iterable so we can loop over the words directly.

**Solution:**

```python
class Sentence:
    def __init__(self, text):
        self.words = text.split()
        self.index = 0
    
    def __iter__(self):
        # Simplest way: return an iterator of the internal list
        return iter(self.words)

# Usage
s = Sentence("Duck typing is cool")
for word in s:
    print(word)

```

**Line-by-Line:**

* `__iter__`: Required for the iteration protocol.
* `return iter(self.words)`: We leverage the fact that Python lists are already iterable. We delegate the responsibility to the list's iterator.
**Why:** Demonstrates delegation, a common pattern in Duck Typing.

---

#### 23. Polymorphic Function

**Question:**
Write a function `double_it(x)` that accepts an int, a string, or a list, and returns the doubled value. Do not check types.

**Solution:**

```python
def double_it(thing):
    # Relies on the '*' operator being implemented by 'thing'
    return thing * 2

print(double_it(10))        # 20
print(double_it("ha"))      # haha
print(double_it([1, 2]))    # [1, 2, 1, 2]

```

**Explanation:**
The function relies on the object implementing `__mul__`.
**Developer Scenario:** Generic utility functions often use this property.

---

### Medium Level (7 Questions)

#### 24. Implement a Custom Context Manager for Locking

**Question:**
Create a class `SoftLock` that mimics a lock. It should print "Lock acquired" when entering a `with` block and "Lock released" when exiting.

**Solution:**

```python
class SoftLock:
    def __init__(self, lock_name):
        self.name = lock_name
        
    def __enter__(self):
        print(f"Lock '{self.name}' acquired")
        return self # Allows 'as' keyword
        
    def __exit__(self, exc_type, exc_value, traceback):
        # exc_type is not None if an error occurred
        if exc_type:
            print(f"Error occurred: {exc_value}")
        print(f"Lock '{self.name}' released")
        # Return False to propagate exception, True to suppress it
        return False 

# Usage
try:
    with SoftLock("DB_WRITE") as lock:
        print("Writing to DB...")
        raise ValueError("Connection failed")
except ValueError:
    print("Caught exception in main")

```

**Why interviewers ask this:**
Tests understanding of the Context Manager protocol (`__enter__`, `__exit__`), exception handling within context managers, and resource cleanup patterns.
**Real App Usage:** Managing database transactions or thread locks.

---

#### 25. Implement a "File-like" Logger

**Question:**
You have a function `write_data(output_stream)` that writes data to a stream. Create a class `Logger` that acts like a file (has a `write` method) but prepends a timestamp to every line, then prints to console.

**Solution:**

```python
import datetime

class TimestampLogger:
    def write(self, message):
        if message.strip(): # Ignore empty newlines often sent by print
            timestamp = datetime.datetime.now().strftime("%H:%M:%S")
            print(f"[{timestamp}] {message}")
            
    def flush(self):
        # File-like objects often need a flush method
        pass

def write_data(stream):
    stream.write("Processing started")
    stream.write("Processing finished")

# Usage
logger = TimestampLogger()
# write_data thinks it's writing to a file, but it's our custom object
write_data(logger) 

```

**Explanation:**
We mimic the interface of `sys.stdout` or a file object (`write`, `flush`).
**Developer Scenario:** Useful for redirecting `stdout` to a logging framework or a GUI widget without changing the core logic code.

---

#### 26. Custom Sequence with Indexing and Slicing

**Question:**
Create a class `Deck` representing a deck of cards. Implement Duck Typing methods so `Deck` supports indexing (`d[0]`) and slicing (`d[:5]`).

**Solution:**

```python
class Deck:
    def __init__(self):
        ranks = [str(n) for n in range(2, 11)] + list('JQKA')
        suits = "Spades Diamonds Clubs Hearts".split()
        self._cards = [f'{r} of {s}' for s in suits for r in ranks]
        
    def __len__(self):
        return len(self._cards)
    
    def __getitem__(self, position):
        # Delegates slicing and indexing to the internal list
        return self._cards[position]

# Usage
d = Deck()
print(d[0])       # 2 of Spades (Indexing)
print(d[-1])      # A of Hearts (Negative Indexing)
print(d[:3])      # List of top 3 cards (Slicing)

```

**Line-by-Line:**

* `__getitem__` handles both single integers (indexing) and `slice` objects (slicing) automatically because the underlying `_cards` list handles them.
**Why interviewers ask this:**
To ensure you know how to make objects behave like Lists (Sequence Protocol).

---

#### 27. Implement Arithmetic Support (Custom Money Class)

**Question:**
Create a `Money` class. It should support `+` (addition) between two Money instances. It should also support `sum()` built-in.

**Solution:**

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = amount
        self.currency = currency
        
    def __add__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        if self.currency != other.currency:
            raise ValueError("Currencies do not match")
        return Money(self.amount + other.amount, self.currency)
    
    def __radd__(self, other):
        # Required for sum() because sum starts with 0 (int)
        # 0 + Money(10) -> int.__add__ fails -> Money.__radd__ called
        if other == 0:
            return self
        return self.__add__(other)

    def __repr__(self):
        return f"{self.amount} {self.currency}"

# Usage
m1 = Money(100)
m2 = Money(50)
total = sum([m1, m2]) # Uses __radd__ for the initial 0 + m1
print(total) # 150 USD

```

**Common Mistakes:**
Forgetting `__radd__`. `sum([m1, m2])` effectively does `((0 + m1) + m2)`. Since `0` is an int, it doesn't know how to add `Money`. Python flips it to `m1.__radd__(0)`.

---

#### 28. Sorting Objects via Duck Typing

**Question:**
Create a class `Student` with `name` and `grade`. Make instances sortable by `grade` using the `sorted()` function.

**Solution:**

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade
        
    def __lt__(self, other):
        # "Less Than" protocol is used by Python's sort
        return self.grade < other.grade
    
    def __repr__(self):
        return f"{self.name}:{self.grade}"

# Usage
students = [Student("Bob", 88), Student("Alice", 95), Student("Charlie", 70)]
sorted_students = sorted(students)
print(sorted_students) 
# Output: [Charlie:70, Bob:88, Alice:95]

```

**Explanation:**
`sorted()` relies on the `<` operator. By implementing `__lt__` (less than), the object satisfies the sortable protocol.
**Complexity:** Python's Timsort uses these comparisons. Time complexity of sort is .

---

#### 29. Plugin System using Duck Typing

**Question:**
Design a simple plugin system. A function `run_plugins(plugins)` accepts a list of objects. Each object must have a `execute()` method. If it has a `setup()` method, run that first.

**Solution:**

```python
class DatabasePlugin:
    def setup(self):
        print("Connecting to DB...")
    def execute(self):
        print("Running DB Query")

class LoggerPlugin:
    # No setup method
    def execute(self):
        print("Logging info")

def run_plugins(plugins):
    for plugin in plugins:
        # Check for setup (Optional Protocol)
        if hasattr(plugin, 'setup'):
            plugin.setup()
        
        # Assume execute exists (Mandatory Protocol)
        try:
            plugin.execute()
        except AttributeError:
            print(f"Error: {plugin} is not a valid plugin (missing execute)")

# Usage
run_plugins([DatabasePlugin(), LoggerPlugin()])

```

**Developer Scenario:**
Common in frameworks (like Django Middleware or Pytest fixtures) where hooks are optional. `hasattr` checks for optional capabilities ("Can you setup?"), while direct calls enforce mandatory ones.

---

#### 30. Infinite Iterator (Custom Generator Logic)

**Question:**
Create an iterator class `Fibonacci` that generates infinite Fibonacci numbers. It must work with `next()`.

**Solution:**

```python
class Fibonacci:
    def __init__(self):
        self.a, self.b = 0, 1
        
    def __iter__(self):
        return self
    
    def __next__(self):
        current = self.a
        self.a, self.b = self.b, self.a + self.b
        return current

# Usage
fib = Fibonacci()
print(next(fib)) # 0
print(next(fib)) # 1
print(next(fib)) # 1
print(next(fib)) # 2

# Don't use 'for x in fib' without a break, or it loops forever!

```

**Why interviewers ask this:**
Tests the Iterator Protocol (`__next__`) and state management.
**Space Complexity:**  (Only stores two integers, not the whole history).
