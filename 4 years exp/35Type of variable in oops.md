# Python Types of Variables in OOP — Interview Guide (30 Questions)

This comprehensive guide focuses on variable scopes, lifecycles, and memory management within Python's Object-Oriented Programming paradigm. It is tailored for a developer with 4 years of experience, moving beyond syntax into behavior, performance, and best practices.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Question 1 (Easy): What are the three main types of variables in a Python class, and how do they differ in scope?

**Expected Answer:**
The three main types are:

1. **Instance Variables:** Defined inside methods (usually `__init__`) using `self`. Unique to each object instance.
2. **Class (Static) Variables:** Defined inside the class but outside methods. Shared across all instances of that class.
3. **Local Variables:** Defined inside a method without `self`. Accessible only within that specific method execution.

**Explanation:**
Python distinguishes variables based on where they are declared and how they are accessed. Instance variables store data specific to an object (like a User's ID). Class variables store data common to all objects (like a default configuration or connection string). Local variables are temporary scratchpads for logic within a function.

**Medium-depth Reasoning:**
Unlike Java or C++, Python does not enforce explicit keywords like `static` for class variables; it relies on the namespace (scope) where the variable is assigned. The distinction is crucial for memory management: class variables are allocated once, while instance variables are allocated per object.

**Time & Space Complexity:**

* Accessing variables involves dictionary lookups (`__dict__`), generally O(1).
* Space: Class variables save space by sharing data.

**Common Mistakes Candidates Make:**

* Thinking local variables persist between method calls.
* Confusing class variables with instance variables when modifying them via `self`.

**Why Interviewers Ask This:**
To verify the candidate understands the fundamental building blocks of state management in OOP.

**Sample Code:**

```python
class Server:
    # Class Variable
    default_port = 8080

    def __init__(self, ip):
        # Instance Variable
        self.ip = ip

    def connect(self):
        # Local Variable
        status = "Connected"
        print(f"{status} to {self.ip}:{self.default_port}")

s1 = Server("192.168.1.1")
s1.connect()

```

**Line-by-line Explanation:**

1. `default_port = 8080`: Created once for the class `Server`.
2. `self.ip = ip`: Created uniquely for every new `Server` object.
3. `status = "Connected"`: Created when `connect` runs, destroyed immediately after it finishes.

---

### Question 2 (Easy): How do you define a private variable in a Python class?

**Expected Answer:**
Python does not have strict `private` access modifiers like Java. Instead, it uses **Name Mangling**. By prefixing a variable with double underscores (e.g., `__password`), Python internally renames it to `_ClassName__password` to prevent accidental access from subclasses or outside.

**Explanation:**
Variables with a single underscore `_var` are "protected" by convention (developers should treat them as internal). Variables with double underscores `__var` trigger name mangling, making them harder (but not impossible) to access externally.

**Medium-depth Reasoning:**
This is a safety mechanism, not a security feature. It prevents namespace collisions in subclasses, ensuring that a subclass doesn't accidentally overwrite a parent class's internal state variable.

**Common Mistakes Candidates Make:**

* Believing `__var` makes data truly secure or inaccessible.
* Confusing `_var` (convention) with `__var` (mangling).

**Why Interviewers Ask This:**
To check if the candidate understands Python's "we are all consenting adults" philosophy regarding encapsulation vs. other languages.

**Sample Code:**

```python
class Account:
    def __init__(self, balance):
        self.__balance = balance  # Private

a = Account(1000)
# print(a.__balance) # Raises AttributeError
print(a._Account__balance) # Works (Name Mangling)

```

**Line-by-line Explanation:**

1. `self.__balance`: Python rewrites this attribute name in the object's `__dict__`.
2. `print(a.__balance)`: Fails because the attribute literally doesn't exist under that name.
3. `print(a._Account__balance)`: Accesses the mangled name directly (discouraged in production).

---

### Question 3 (Easy): What happens if you modify a class variable using an instance (e.g., `self.class_var = new_val`)?

**Expected Answer:**
It does **not** change the class variable for other instances. Instead, it creates a new **instance variable** with the same name on that specific object, "shadowing" the class variable.

**Explanation:**
Python looks up variables in this order: Instance `__dict__` -> Class `__dict__`. When you assign via `self`, you write to the Instance `__dict__`. The original class variable remains untouched in the Class `__dict__`.

**Medium-depth Reasoning:**
This behavior allows instances to override default values without affecting the global default for other objects. However, it is a frequent source of bugs if the developer intended to update the global state.

**Common Mistakes Candidates Make:**

* Assuming `self.var = x` updates the variable for all objects if `var` was defined as a class variable.

**Why Interviewers Ask This:**
This is the classic "Gotcha" question in Python OOP to test understanding of the lookup chain.

**ASCII Diagram:**

```text
Before Assignment:
Object A -> looks for 'var' -> Not found -> Class (var=10)

After `self.var = 20`:
Object A -> looks for 'var' -> Found in Instance (var=20) (Stops here)
Object B -> looks for 'var' -> Not found -> Class (var=10)

```

**Sample Code:**

```python
class Config:
    limit = 10

c1 = Config()
c2 = Config()

c1.limit = 20 # Shadows class variable

print(c1.limit) # 20 (Instance)
print(c2.limit) # 10 (Class)
print(Config.limit) # 10 (Class)

```

**Line-by-line Explanation:**

1. `c1.limit = 20`: Adds entry `'limit': 20` to `c1.__dict__`.
2. `c2.limit`: `c2` has no `limit`, so it fetches `Config.limit`.
3. `Config.limit`: remains 10.

---

### Question 4 (Easy): What is the purpose of the `__init__` method regarding variables?

**Expected Answer:**
`__init__` is the constructor method used to initialize the object's state. It is the standard place to define and assign values to **Instance Variables**.

**Explanation:**
While instance variables can be defined anywhere, defining them in `__init__` ensures the object is created in a valid, fully initialized state. It prevents `AttributeError` later in the code.

**Medium-depth Reasoning:**
In Python, `__init__` is not creating the object (that's `__new__`), but initializing it. It establishes the "shape" of the object's instance dictionary.

**Common Mistakes Candidates Make:**

* Defining instance variables outside `__init__` (e.g., inside random methods), leading to objects that have different attributes depending on which methods were called.

**Why Interviewers Ask This:**
To ensure code consistency and reliability practices.

**Sample Code:**

```python
class User:
    def __init__(self, name):
        self.name = name # Correct place
        self.active = True

    def activate(self):
        self.active = True # Modifying existing

```

**Line-by-line Explanation:**

1. `self.name = name`: Guarantees every User has a name immediately upon creation.
2. `self.active`: Sets a default instance state.

---

### Question 5 (Easy): Can you access an instance variable inside a `@classmethod`?

**Expected Answer:**
No, not directly.

**Explanation:**
A `@classmethod` receives `cls` (the class itself) as the first argument, not `self` (the instance). Instance variables belong to a specific object. Since a class method runs on the class level, it doesn't know which specific object instance you are referring to.

**Medium-depth Reasoning:**
You can technically access an instance variable in a class method **only if** you pass an instance of the class as an argument to that method, but that defeats the purpose of a class method.

**Common Mistakes Candidates Make:**

* Trying to use `self.variable` inside a method decorated with `@classmethod`.

**Why Interviewers Ask This:**
To distinguish between Class scope and Instance scope.

**Sample Code:**

```python
class Data:
    def __init__(self):
        self.value = 5

    @classmethod
    def info(cls):
        # return cls.value  # Error: Class has no attribute 'value'
        # return self.value # Error: 'self' is not defined
        return "Class Method"

```

**Line-by-line Explanation:**

1. `@classmethod`: Changes the first argument from `self` to `cls`.
2. Inside `info`, we only have access to Class variables (attached to `cls`) or static logic.

---

### Question 6 (Easy): What are "Magic Variables" or Dunder attributes like `__dict__`?

**Expected Answer:**
`__dict__` is a built-in dictionary that stores a writable object's attributes.

* `obj.__dict__` stores instance variables.
* `Class.__dict__` stores class variables and methods.

**Explanation:**
Python objects are dynamically structured. `__dict__` is the mapping object that holds the variable names and their values. This allows for dynamic introspection and modification of objects at runtime.

**Medium-depth Reasoning:**
Not all objects have `__dict__` (e.g., if `__slots__` is used, or for built-in types like `int`). Understanding `__dict__` is key to understanding Python's metaprogramming capabilities.

**Common Mistakes Candidates Make:**

* Assuming `__dict__` contains everything (it doesn't usually contain methods for instances; methods are looked up in the class).

**Why Interviewers Ask This:**
To test understanding of Python's internal object structure.

**Sample Code:**

```python
class A:
    c_var = 1
    def __init__(self):
        self.i_var = 2

a = A()
print(a.__dict__)

```

**Line-by-line Explanation:**

1. `print(a.__dict__)`: Outputs `{'i_var': 2}`. Note that `c_var` is not here; it lives in `A.__dict__`.

---

### Question 7 (Medium): Explain the behavior of Mutable objects (like lists) when used as Class Variables.

**Expected Answer:**
If a mutable object (list, dict) is a class variable, it is **shared** across all instances. Modifying it via one instance (e.g., appending to the list) will reflect in all other instances.

**Explanation:**
Class variables are initialized once when the class is defined. The memory reference to that list is stored in the class. All instances point to that same reference.

**Medium-depth Reasoning:**
This is a very common bug. If you want a unique list for every object, you **must** initialize it inside `__init__` using `self.list = []`.

**Common Mistakes Candidates Make:**

* Using `items = []` at the class level intending for each object to have its own empty list.

**Why Interviewers Ask This:**
This is a critical practical knowledge question to avoid data leakage between objects.

**ASCII Diagram:**

```text
Class A:
  shared_list = []  <-- Single memory address (0x123)

Instance 1 .shared_list points to -> 0x123
Instance 2 .shared_list points to -> 0x123

Inst1.shared_list.append(1) -> 0x123 is now [1]
Inst2 sees [1] immediately.

```

**Sample Code:**

```python
class Basket:
    items = [] # Dangerous if meant to be unique

    def add(self, item):
        self.items.append(item)

b1 = Basket()
b2 = Basket()
b1.add("Apple")

print(b2.items) # ['Apple'] - Unexpected sharing!

```

**Line-by-line Explanation:**

1. `items = []`: Created once at compile time.
2. `self.items.append`: Accesses the shared list via `self` and modifies it in place.
3. `b2.items`: Reads the same modified list.

---

### Question 8 (Medium): How does `__slots__` affect variable storage in a class?

**Expected Answer:**
`__slots__` explicitly declares data members (instance variables) and denies the creation of `__dict__`. This forces Python to allocate a fixed amount of space for the variables, saving memory and speeding up access.

**Explanation:**
Normally, Python uses a dictionary for variables, which has memory overhead. `__slots__` reserves space for specific variables only.

**Medium-depth Reasoning:**
The trade-off is flexibility. You cannot add new variables to an instance at runtime if they are not listed in `__slots__`. It is ideal for classes that will have millions of instances (e.g., a Point class in a geometry app).

**Time & Space Complexity:**

* Space: Significantly reduced (no dictionary overhead).
* Time: Slightly faster attribute access.

**Common Mistakes Candidates Make:**

* Using `__slots__` to prevent users from adding attributes (security), rather than for memory optimization.
* Forgetting that subclasses inherit `__slots__` behavior in complex ways.

**Why Interviewers Ask This:**
To test knowledge of memory optimization techniques in Python.

**Sample Code:**

```python
class Point:
    __slots__ = ['x', 'y']

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
# p.z = 3 # Raises AttributeError: 'Point' object has no attribute 'z'

```

**Line-by-line Explanation:**

1. `__slots__ = ['x', 'y']`: Tells Python "Only allocate memory for x and y".
2. `p.z = 3`: Fails because there is no dynamic `__dict__` to store `z`.

---

### Question 9 (Medium): What is the difference between `_variable` and `__variable` regarding inheritance?

**Expected Answer:**

* `_variable` (Protected): Inherited normally by subclasses. It is just a naming convention.
* `__variable` (Private): Not directly inherited by name due to Name Mangling. The subclass cannot access it using `self.__variable` because it is mangled to `_ParentClass__variable`.

**Explanation:**
If a parent class has `__id`, it becomes `_Parent__id`. If a subclass tries to access `self.__id`, Python looks for `_Subclass__id`, which doesn't exist (or is a different variable entirely).

**Medium-depth Reasoning:**
This ensures that a subclass implementation of a variable doesn't accidentally break the parent class's internal logic if they happen to choose the same variable name.

**Common Mistakes Candidates Make:**

* Thinking `__private` variables cannot be overridden. They can be redefined, but they are treated as distinct variables in memory.

**Why Interviewers Ask This:**
To test understanding of inheritance and namespace safety.

**Sample Code:**

```python
class Parent:
    def __init__(self):
        self.__val = "Parent"

    def get_val(self):
        return self.__val

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__val = "Child" # This is _Child__val

p = Child()
print(p.get_val()) # Returns "Parent", not "Child"!

```

**Line-by-line Explanation:**

1. Parent `__init__` sets `_Parent__val`.
2. Child `__init__` sets `_Child__val`.
3. `get_val` is defined in Parent, so it looks for `_Parent__val`. The parent's logic remains intact despite the child defining a variable with the same raw name.

---

### Question 10 (Medium): How do Global variables interact with Class scope?

**Expected Answer:**
Variables defined in the global scope are accessible inside class methods (read-only unless `global` is used). However, Class bodies (the space where methods are defined) are executed during definition, so they can read global variables immediately.

**Explanation:**
Python searches scopes in LEGB order (Local, Enclosing, Global, Built-in). If a variable isn't found in the method or class instance, Python looks at the Global scope.

**Medium-depth Reasoning:**
Relying on globals inside classes creates tight coupling and makes unit testing difficult. It is considered bad practice.

**Common Mistakes Candidates Make:**

* Shadowing a global variable with a local variable and getting confused why the global didn't change.

**Why Interviewers Ask This:**
To ensure the candidate writes clean, encapsulated code.

**Sample Code:**

```python
CONFIG_TIMEOUT = 5

class Connector:
    timeout = CONFIG_TIMEOUT # Reads global at definition time

    def set_custom(self):
        global CONFIG_TIMEOUT
        CONFIG_TIMEOUT = 10 # Modifies global for everyone!

```

**Line-by-line Explanation:**

1. `timeout = CONFIG_TIMEOUT`: Copies the value 5 into the class variable at the moment the script loads.
2. `global CONFIG_TIMEOUT`: Tells the method to affect the module-level variable, not create a local one.

---

### Question 11 (Medium): Explain variable scope inside a List Comprehension within a Class.

**Expected Answer:**
In Python 3, list comprehensions have their own scope (like a function). However, within a class body (not inside a method), a list comprehension **cannot** access other class variables defined in the same block directly by name.

**Explanation:**
Class bodies are special namespaces. While code inside the class body can see class variables, a scope created *inside* that body (like a comprehension) essentially "forgets" the class scope and looks straight to global.

**Medium-depth Reasoning:**
This is a quirky Python edge case. `[x for x in items]` works, but `[func(x) for x in items]` will fail if `func` is defined in the class body just above it, because the comprehension scope can't "see" `func`.

**Why Interviewers Ask This:**
To test deep knowledge of Python's scoping rules (LEGB) specifically in the context of class definitions.

**Sample Code:**

```python
class Demo:
    factor = 10
    # names = [x * factor for x in range(5)] # NameError: name 'factor' is not defined
    
    # Fix:
    names = [x * 10 for x in range(5)] 

```

**Line-by-line Explanation:**

1. The commented out line fails because `factor` is not visible inside the comprehension's private scope during class creation.

---

### Question 12 (Medium): What is the difference between `self.var` and `ClassName.var` inside a method?

**Expected Answer:**

* `self.var`: Accesses the variable via the instance. It respects inheritance and polymorphism (if `self` is a subclass instance). It reads from instance dict first, then class dict.
* `ClassName.var`: Hardcodes the access to that specific class's variable. It ignores inheritance if the object is a subclass attempting to override that property.

**Explanation:**
Using `self.var` is generally preferred for polymorphism. Using `ClassName.var` is preferred when you explicitly want the static data of that specific class regardless of the subclass.

**Medium-depth Reasoning:**
If you change `ClassName.var`, you change it for all instances. If you change `self.var`, you usually only change it for that object (unless `var` is a mutable object found on the class, as discussed in Q7).

**Why Interviewers Ask This:**
To see if you understand polymorphism vs static access.

**Sample Code:**

```python
class Base:
    limit = 10
    def get_limit(self):
        return self.limit

class Sub(Base):
    limit = 20

print(Sub().get_limit()) # Returns 20 (Polymorphic)

```

**Line-by-line Explanation:**

1. `self.limit`: Since `self` is an instance of `Sub`, it finds `Sub.limit` (20).
2. If we wrote `return Base.limit` in the parent, it would return 10 even for the subclass.

---

### Question 13 (Medium): What is the `@property` decorator and how does it relate to instance variables?

**Expected Answer:**
`@property` allows you to define a method that behaves like an instance variable. It provides "getter" functionality. It encapsulates instance variables, allowing validation or computation logic when accessing a variable.

**Explanation:**
It transforms `obj.method()` access into `obj.method`. It is the Pythonic way to implement getters and setters without changing the interface (no `get_variable()` calls needed).

**Medium-depth Reasoning:**
It promotes backward compatibility. You can start with a public variable `self.x`. Later, if you need logic (e.g., x cannot be negative), you can change it to a property without breaking code that uses `obj.x`.

**Common Mistakes Candidates Make:**

* Creating a property that performs expensive computations (users expect variable access to be O(1) or very fast).

**Why Interviewers Ask This:**
To check if the candidate knows how to manage variable access pythonically.

**Sample Code:**

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def area(self):
        return 3.14 * (self._radius ** 2)

c = Circle(5)
print(c.area) # Accessed like a variable, computed on fly

```

**Line-by-line Explanation:**

1. `@property`: Marks `area` as a descriptor.
2. `c.area`: Calls the method but looks like a variable access.

---

### Question 14 (Medium): What are "Weak References" regarding class/instance variables?

**Expected Answer:**
Weak references (`weakref` module) allow you to hold a reference to an object without increasing its reference count. This is useful for caching or mapping instance variables without preventing the Garbage Collector from deleting the object when it's no longer used elsewhere.

**Explanation:**
Standard variables create "strong" references. If `Class.registry` holds a list of all instances, those instances will never be deleted from memory because the list holds them. Using `weakref` solves this memory leak.

**Medium-depth Reasoning:**
This is essential for the "Observer" pattern or when keeping track of all live instances of a class.

**Why Interviewers Ask This:**
To assess knowledge of memory management and advanced lifecycle handling.

**Sample Code:**

```python
import weakref

class Node:
    pass

n = Node()
r = weakref.ref(n)

print(r()) # <__main__.Node object...>
del n
print(r()) # None (Object was garbage collected)

```

**Line-by-line Explanation:**

1. `weakref.ref(n)`: Creates a reference but doesn't "own" the object.
2. `del n`: Removes the only strong reference.
3. `r()`: Returns `None` because the object is destroyed.

---

### Question 15 (Medium): How are variables looked up in Multiple Inheritance (MRO)?

**Expected Answer:**
Python uses the C3 Linearization algorithm to determine the Method Resolution Order (MRO). When looking up a variable via `self.var`, Python checks the instance, then the class, then parent classes following the MRO list left-to-right.

**Explanation:**
If `class D(B, C): pass`, and both B and C have a variable `x`, Python follows the order: D -> B -> C. It stops at the first match.

**Medium-depth Reasoning:**
This resolves the "Diamond Problem". You can check the order using `ClassName.mro()`.

**Common Mistakes Candidates Make:**

* Assuming "Depth-first" search (old Python 2 style) rather than the C3 linearization.

**Why Interviewers Ask This:**
To ensure you can debug complex inheritance hierarchies.

**Sample Code:**

```python
class A: x = "A"
class B(A): x = "B"
class C(A): x = "C"
class D(B, C): pass

print(D.x) # "B"
print(D.mro())

```

**Line-by-line Explanation:**

1. `class D(B, C)`: B is listed first.
2. MRO for D is [D, B, C, A, object].
3. Python finds `x` in B first.

---

### Question 16 (Medium): What is the `nonlocal` keyword and does it apply to Class variables?

**Expected Answer:**
`nonlocal` is used in nested functions (closures) to modify a variable in the nearest enclosing scope (excluding global). It does **not** apply to Class variables and cannot be used to modify class attributes directly from a method.

**Explanation:**
`nonlocal` is strictly for function nesting. A method inside a class is a function, but the class body is not a function scope, it's a class scope. You cannot use `nonlocal` to bridge a method and a class variable.

**Medium-depth Reasoning:**
To modify a class variable from a method, you use `ClassName.var` or `type(self).var`. `nonlocal` is irrelevant in standard OOP attribute manipulation.

**Why Interviewers Ask This:**
To check if the candidate confuses functional scope tools with OOP scope tools.

**Sample Code:**

```python
def outer():
    x = 10
    def inner():
        nonlocal x
        x = 20
    inner()
    return x # Returns 20

```

**Line-by-line Explanation:**

1. `nonlocal x`: Binds `inner`'s `x` to `outer`'s `x`.
2. Without `nonlocal`, `inner` would create a new local `x`.

---

### Question 17 (Medium): When would you use a Static Method (`@staticmethod`) regarding variables?

**Expected Answer:**
You use `@staticmethod` when the method performs logic that belongs to the class namespace but **does not access any class variables (cls) or instance variables (self)**.

**Explanation:**
It is essentially a regular function that lives inside a class for organizational purposes. It cannot modify object state or class state.

**Medium-depth Reasoning:**
It signals intent to other developers: "This method does not touch the variables of this object." It is often used for utility functions related to the class.

**Why Interviewers Ask This:**
Code design and intent clarity.

**Sample Code:**

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

# MathUtils.add(5, 10)

```

**Line-by-line Explanation:**

1. `@staticmethod`: No implicit first argument (`self` or `cls`) is passed.
2. Pure function behavior inside a class namespace.

---

### Question 18 (Medium): What happens to instance variables when you `pickle` (serialize) an object?

**Expected Answer:**
By default, `pickle` saves the object's `__dict__` (instance variables). Class variables are **not** serialized; only the class name is saved. Upon deserialization, the object reconnects to the existing Class definition to access class variables.

**Explanation:**
If you change a class variable *after* pickling an object but *before* unpickling it, the unpickled object will see the *new* class variable value, but it will retain its old instance variable values.

**Medium-depth Reasoning:**
This is crucial for data persistence. You don't store code/logic in the DB/File, only the state (instance variables).

**Why Interviewers Ask This:**
To test knowledge of persistence and object reconstruction.

**Sample Code:**

```python
import pickle
class Data:
    shared = 1
    def __init__(self, val):
        self.val = val

d = Data(100)
serialized = pickle.dumps(d)
Data.shared = 2 # Change class var
d2 = pickle.loads(serialized)

print(d2.val)    # 100 (Restored)
print(d2.shared) # 2 (Uses current class definition)

```

**Line-by-line Explanation:**

1. `pickle.dumps(d)`: Saves `{val: 100}`.
2. `d2.shared`: Looks up `Data.shared` which is now 2.

---

### Question 19 (Medium): How can you dynamically add a variable to a Class (not instance) at runtime?

**Expected Answer:**
You can assign a value to `ClassName.new_var = value`. This immediately makes the variable available to all existing and future instances of that class.

**Explanation:**
Since classes are objects (instances of `type`), they are mutable. You can modify their `__dict__` at runtime.

**Medium-depth Reasoning:**
This is "Monkey Patching". While powerful, it is dangerous in large applications because it makes the flow of data hard to track. It is mostly used in testing (mocking).

**Common Mistakes Candidates Make:**

* Thinking you need to restart the application to change class structure.

**Why Interviewers Ask This:**
To understand the dynamic nature of Python.

**Sample Code:**

```python
class Hero:
    pass

h1 = Hero()
Hero.power = 100 # Added dynamically

print(h1.power) # 100

```

**Line-by-line Explanation:**

1. `Hero.power = 100`: Inject 'power' into `Hero.__dict__`.
2. `h1.power`: `h1` finds it in the class.

---

### Question 20 (Medium): What is the variable lifecycle in a recursive method within a class?

**Expected Answer:**
Each recursive call creates a **new stack frame** with its own set of **Local Variables**. Instance variables (`self.var`) persist and are shared across all recursion levels because `self` refers to the same heap object.

**Explanation:**
Local variables are isolated per call. Instance variables are shared state.

**Medium-depth Reasoning:**
If you need to accumulate a value during recursion, you should usually pass it as an argument or return it. Using an instance variable (e.g., `self.counter += 1`) works but makes the method not thread-safe and "dirty" (it leaves the object in a modified state after the calculation).

**Why Interviewers Ask This:**
To test algorithmic thinking combined with OOP state management.

**Sample Code:**

```python
class Factorial:
    def calc(self, n):
        result = 1 # Local, new every time (conceptually, though unused in recursion base)
        if n == 1: return 1
        return n * self.calc(n-1)

```

**Line-by-line Explanation:**

1. `result`: Created in every frame.
2. `self`: Passed to every frame, pointing to the same object.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Question 21 (Easy): Basic Instance Counter

**Coding Question:**
Create a class `Employee`. Every time a new `Employee` is created, increment a counter. Provide a method to get the current count.

**Solution Code:**

```python
class Employee:
    # Class Variable to track count
    count = 0

    def __init__(self):
        # Access via ClassName to update the shared variable
        Employee.count += 1

    @classmethod
    def get_count(cls):
        return cls.count

# Test
e1 = Employee()
e2 = Employee()
e3 = Employee()
print(Employee.get_count()) # Output: 3

```

**Line-by-line Explanation:**

1. `count = 0`: Initialized once.
2. `Employee.count += 1`: Inside `__init__`, we explicitly target the class variable. If we used `self.count += 1`, it would try to read `count`, add 1, and save it as an *instance* variable, failing to update the global count correctly.
3. `@classmethod`: Used because this method relates to the class as a whole, not a specific person.

**Time & Space:**
O(1) time for creation and access. O(1) space for the counter.

**Why Interviewers Ask This:**
Fundamental check of Class vs Instance variable usage.

**Developer Scenario:**
Tracking active database connections or game entities.

**Real App Usage:**
**Game Servers:** Counting total spawned enemies.

---

### Question 22 (Easy): Default Arguments Problem

**Coding Question:**
Identify the bug in this code and fix it. The goal is a class `Student` where each student has a list of grades.

```python
class Student:
    def __init__(self, name, grades=[]):
        self.name = name
        self.grades = grades

```

**Solution Code:**

```python
class Student:
    # Fix: Use None instead of mutable default argument
    def __init__(self, name, grades=None):
        self.name = name
        if grades is None:
            self.grades = [] # Create a NEW list for this instance
        else:
            self.grades = grades

s1 = Student("Alice")
s2 = Student("Bob")
s1.grades.append(95)

print(s1.grades) # [95]
print(s2.grades) # [] (Correctly empty)

```

**Line-by-line Explanation:**

1. `grades=None`: Prevents the "mutable default argument" trap where the list is created once at definition time and shared.
2. `self.grades = []`: allocates a fresh list in memory for this specific object.

**Time & Space:**
O(1).

**Common Mistakes:**
Using `grades=[]` in the signature.

**Why Interviewers Ask This:**
The #1 most common Python interview question regarding variables.

**Real App Usage:**
**Data Processing:** Initializing data containers for ETL jobs.

---

### Question 23 (Easy): Encapsulation with Properties

**Coding Question:**
Create a `Temperature` class that stores temperature in Celsius (instance variable). Allow the user to get and set the temperature in Fahrenheit using a property, automatically converting it.

**Solution Code:**

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def fahrenheit(self):
        return (self._celsius * 9/5) + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

t = Temperature(0)
print(t.fahrenheit) # 32.0
t.fahrenheit = 100
print(t._celsius)   # 37.77...

```

**Line-by-line Explanation:**

1. `_celsius`: Internal "protected" variable.
2. `@property`: exposes `fahrenheit` as a readable variable.
3. `@fahrenheit.setter`: Allows `t.fahrenheit = 100`, which calculates the backing `_celsius` value.

**Time & Space:**
O(1) for conversion.

**Developer Scenario:**
Legacy systems where DB stores Metric, but UI displays Imperial.

**Real App Usage:**
**Weather Apps:** Storing data in Kelvin/UTC, displaying in Local/Fahrenheit.

---

### Question 24 (Medium): Shared State Borg Pattern

**Coding Question:**
Implement the "Borg" pattern (Monostate). Create a class `Config` where all instances share the same state (variables), even if they are different objects.

**Solution Code:**

```python
class Config:
    _shared_state = {}

    def __init__(self):
        # Point the instance's dictionary to the shared class dictionary
        self.__dict__ = self._shared_state

c1 = Config()
c2 = Config()

c1.db_host = "localhost"
print(c2.db_host) # localhost
print(c1 is c2)   # False (Different objects, shared brain)

```

**Line-by-line Explanation:**

1. `_shared_state = {}`: A single dictionary living on the class.
2. `self.__dict__ = self._shared_state`: Hijacks the object's storage. Usually, every object has its own `__dict__`. Here, we force them to point to the same one.
3. Any variable set on `c1` goes into `_shared_state`, which `c2` also reads from.

**Time & Space:**
O(1).

**Why Interviewers Ask This:**
To test deep understanding of `__dict__`.

**Real App Usage:**
**Configuration Loaders:** Loading settings from a `.env` file that must be consistent across the app.

---

### Question 25 (Medium): Tracking Instances with WeakRefs

**Coding Question:**
Create a class `Widget` that keeps track of all its currently alive instances in a class variable list. Use `weakref` to ensure deleted widgets are automatically removed from the list.

**Solution Code:**

```python
import weakref

class Widget:
    _instances = []

    def __init__(self, name):
        self.name = name
        # Add a weak reference to self in the class list
        self._instances.append(weakref.ref(self))

    @classmethod
    def get_instances(cls):
        # Resolve weakrefs, ignore dead ones (None)
        alive = []
        for ref in cls._instances:
            obj = ref()
            if obj:
                alive.append(obj.name)
        return alive

w1 = Widget("A")
w2 = Widget("B")
del w1
print(Widget.get_instances()) # ['B']

```

**Line-by-line Explanation:**

1. `weakref.ref(self)`: Creates a pointer that doesn't count towards the object's "life support".
2. `obj = ref()`: Attempts to access the object. Returns `None` if it was garbage collected (like `w1` after `del`).

**Time & Space:**
O(N) to iterate instances.

**Why Interviewers Ask This:**
Memory management proficiency.

**Real App Usage:**
**GUI Frameworks:** Updating all open windows when the theme changes.

---

### Question 26 (Medium): Limiting Attributes with `__slots__`

**Coding Question:**
Create a memory-efficient class `Pixel` that stores `r, g, b` values. It must forbid adding any other attributes (like `alpha`) to save memory. Demonstrate the error.

**Solution Code:**

```python
class Pixel:
    __slots__ = ['r', 'g', 'b']

    def __init__(self, r, g, b):
        self.r = r
        self.g = g
        self.b = b

p = Pixel(255, 0, 0)
print(p.r)

try:
    p.alpha = 1.0
except AttributeError as e:
    print(f"Caught error: {e}")

```

**Line-by-line Explanation:**

1. `__slots__`: Defines the rigid structure. No `__dict__` is created.
2. `p.alpha = 1.0`: Python attempts to write to the object's slots. "alpha" is not a defined slot, so it raises `AttributeError`.

**Time & Space:**
Space: highly optimized compared to standard classes.

**Developer Scenario:**
Image processing where you have millions of pixel objects.

**Real App Usage:**
**Pandas/NumPy:** (Internally similar concepts) managing large arrays of structured data.

---

### Question 27 (Medium): Inheritance variable shadowing

**Coding Question:**
Create a class structure where:

1. Parent has a variable `setup_config`.
2. Child overrides `setup_config`.
3. Write a method in Parent that prints `setup_config`.
4. Show that calling this method on Child prints the Child's version (Polymorphism of variables).

**Solution Code:**

```python
class Handler:
    setup_config = "Basic"

    def process(self):
        # Accesses self.setup_config, utilizing polymorphism
        print(f"Processing with {self.setup_config}")

class SecureHandler(Handler):
    setup_config = "Encrypted"

h = Handler()
s = SecureHandler()

h.process() # Processing with Basic
s.process() # Processing with Encrypted

```

**Line-by-line Explanation:**

1. `process(self)`: Uses `self.setup_config`.
2. When `s.process()` runs, `self` is `s`.
3. Python looks for `setup_config` in `s` (instance). Not found.
4. Looks in `SecureHandler`. Found "Encrypted". Uses it.

**Common Mistakes:**
Using `Handler.setup_config` inside `process`, which would force "Basic" every time.

**Real App Usage:**
**Django Views:** Overriding `template_name` or `queryset` class variables in subclasses.

---

### Question 28 (Medium): Dictionary Wrapper (Dynamic Attributes)

**Coding Question:**
Create a class `DynamicConfig` that takes a dictionary in `__init__` and converts keys into instance variables so they can be accessed via dot notation (`obj.key`).

**Solution Code:**

```python
class DynamicConfig:
    def __init__(self, data_dict):
        for key, value in data_dict.items():
            # setattr(obj, name, value) is the dynamic equivalent of self.name = value
            setattr(self, key, value)

data = {"host": "127.0.0.1", "port": 80}
conf = DynamicConfig(data)

print(conf.host) # 127.0.0.1
print(conf.port) # 80

```

**Line-by-line Explanation:**

1. `setattr(self, key, value)`: Adds the variable to `self.__dict__` programmatically.
2. This allows converting JSON responses directly into Python Objects.

**Time & Space:**
O(N) where N is number of keys.

**Why Interviewers Ask This:**
To test meta-programming and dynamic attribute handling.

**Real App Usage:**
**API Clients:** Converting JSON responses into objects for easier usage.

---

### Question 29 (Medium): Variable Scope in Nested Classes

**Coding Question:**
Demonstrate that a nested class (Inner) **cannot** automatically see variables of the Outer class.

**Solution Code:**

```python
class Outer:
    outer_var = 10

    class Inner:
        def display(self):
            # return outer_var # NameError
            # return Outer.outer_var # Works (Explicit)
            return "Cannot see outer_var implicitly"

o = Outer()
i = o.Inner()
print(i.display())

```

**Line-by-line Explanation:**

1. Python classes do not form closures like functions do.
2. `Inner` is just a class defined in the namespace of `Outer`. It has no magical link to `Outer`'s variables.
3. You must explicitly access `Outer.outer_var`.

**Why Interviewers Ask This:**
To clarify the difference between lexical scope (functions) and class namespaces.

**Real App Usage:**
**ORM Models:** Defining `Meta` classes inside Models (Django).

---

### Question 30 (Medium): Class Variable Lazy Initialization

**Coding Question:**
Create a `Database` class. It should have a class variable `connection` that is initially `None`.
Write a method `get_connection`.

1. If `connection` is None, simulate connecting and assign it to the class variable.
2. If `connection` exists, return it.
Ensure all instances share this single connection.

**Solution Code:**

```python
class Database:
    _connection = None # Class variable

    def get_connection(self):
        if Database._connection is None:
            print("Connecting to DB...")
            Database._connection = "Active Connection 0x1"
        else:
            print("Using existing connection...")
        return Database._connection

db1 = Database()
db2 = Database()

db1.get_connection() # Connecting to DB...
db2.get_connection() # Using existing connection...

```

**Line-by-line Explanation:**

1. `Database._connection`: We access the variable explicitly on the Class to ensure we update the shared version.
2. First call: Sets the class variable.
3. Second call: Sees the variable is not None and reuses it.

**Why Interviewers Ask This:**
This implements the Singleton pattern logic using class variables.

**Real App Usage:**
**Database Pools:** Establishing expensive connections only once.
