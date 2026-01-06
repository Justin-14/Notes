# Python Constructor, self and comparing objects— Interview Guide (30 Questions)

This guide is designed for a Python developer with approximately 4 years of experience. It moves beyond syntax basics into object lifecycle, memory management, identity, and best practices for class design in Python 3.12+.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (6 Questions)

#### 1. What is the precise role of `__init__` in Python? Is it the constructor?

* **Answer:** No, `__init__` is **not** the constructor; it is the **initializer**. The actual constructor in Python is `__new__`. When you create an instance `obj = MyClass()`, Python first calls `__new__` to allocate memory and create the object, and then calls `__init__` to initialize the attributes of that newly created object.
* **Explanation:** `__init__` receives the instance created by `__new__` as the first argument (`self`). It is responsible for setting up the initial state.
* **Reasoning:** Understanding this distinction is vital when subclassing immutable types (like `str` or `tuple`) or implementing patterns like Singletons, where you need control *before* initialization.
* **Mistakes:** Calling `__init__` a constructor is a common terminology error.
* **Interviewer Intent:** Checks if the candidate understands the object creation lifecycle vs. just object initialization.
* **Sample Code:**

```python
class Demo:
    def __new__(cls):
        print("1. __new__ called (Allocating memory)")
        return super(Demo, cls).__new__(cls)

    def __init__(self):
        print("2. __init__ called (Initializing state)")

d = Demo()

```

#### 2. What is `self` in Python? Is it a reserved keyword?

* **Answer:** `self` is a strong convention used to refer to the instance of the class upon which a method is being called. It is **not** a reserved keyword. You could technically use `this`, `me`, or `pizza`, but doing so is strongly discouraged as it breaks readability standards (PEP 8).
* **Explanation:** Python passes the instance object explicitly as the first argument to instance methods.
* **Reasoning:** Python chooses explicit is better than implicit. In C++/Java, `this` is implicit. In Python, explicit `self` makes it clear which scope (instance vs local) a variable belongs to.
* **Mistakes:** Thinking `self` is a keyword or magic variable. Omitting `self` in method definitions.
* **Interviewer Intent:** Verifies knowledge of Python's explicit method binding mechanism.
* **Sample Code:**

```python
class A:
    def method(self):
        return self  # Standard convention

class B:
    def method(pizza): # Valid syntax, but terrible practice
        return pizza 

```

#### 3. What is the difference between `==` and `is`?

* **Answer:** `==` checks for **value equality** (do these objects represent the same data?), whereas `is` checks for **reference equality** (are these two names pointing to the exact same object in memory?).
* **Explanation:** `==` calls the `__eq__` method. `is` compares the memory addresses (`id()`).
* **Reasoning:** Vital for bug avoidance. Two lists `[1,2]` are equal (`==`) but not the same object (`is`).
* **Mistakes:** Using `is` to compare strings or integers relying on interning (which is an implementation detail), or using `is` for value checks.
* **Interviewer Intent:** Basic sanity check on object identity vs. equivalence.
* **Sample Code:**

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b) # True (Values match)
print(a is b) # False (Different memory locations)

```

#### 4. If you don't define `__init__` in a class, what happens?

* **Answer:** Python will use the `__init__` method of the parent class. If the class inherits directly from `object` (which all classes do in Python 3), it calls `object.__init__`, which does nothing.
* **Explanation:** Method resolution order (MRO) looks up the hierarchy.
* **Reasoning:** Knowing this helps in understanding inheritance chains and Mixins.
* **Mistakes:** Thinking it throws an error or thinking valid attributes are auto-generated.
* **Interviewer Intent:** Understanding of default behavior and inheritance.

#### 5. Can you call `__init__` manually from an instance?

* **Answer:** Yes, you can call `obj.__init__()` like any other method, but it is rarely good practice outside of parent initialization (`super().__init__()`).
* **Explanation:** It will re-run the initialization logic, effectively "resetting" the attributes defined in `__init__`, but it does not create a new object.
* **Reasoning:** Useful in very specific reset scenarios, but usually indicates bad design.
* **Mistakes:** Thinking calling `__init__` returns a new object. It returns `None`.
* **Interviewer Intent:** probing boundaries of method invocation.

#### 6. What does `super().__init__()` do?

* **Answer:** It delegates the initialization task to the next class in the Method Resolution Order (MRO), typically the parent class.
* **Explanation:** It ensures that the parent class sets up its necessary state (attributes, resource allocations) before the child class adds its own logic.
* **Reasoning:** Essential for inheritance. Without it, the parent's attributes might not exist on the child instance.
* **Interviewer Intent:** Basic inheritance competence.

---

### Medium Level (14 Questions)

#### 7. Why is `self` passed explicitly in methods but you don't pass it when calling the method?

* **Answer:** When you access a method on an instance (e.g., `obj.method`), Python creates a **bound method**. This bound method wraps the function and automatically inserts the instance (`obj`) as the first argument when the function is actually called.
* **Explanation:** `obj.method()` is syntactic sugar for `Class.method(obj)`. The descriptor protocol handles this binding.
* **Reasoning:** This underlying mechanic explains why `TypeError: method() takes 1 positional argument but 2 were given` occurs if you forget `self` in the definition (Python passes `self`, plus your args).
* **Mistakes:** Confusion over why argument counts mismatch in tracebacks.
* **Interviewer Intent:** Deep understanding of Python's method binding and descriptors.
* **Diagram:**

```text
obj.method(arg)  -->  Class.method(obj, arg)
      ^                     ^
      |_____________________|
      Implicit 'self' injection

```

#### 8. How do you make a class "hashable" so it can be used in a `set` or as a `dict` key?

* **Answer:** You must implement both `__eq__` and `__hash__`.
* **Explanation:** `__eq__` determines if two objects are equal. `__hash__` returns an integer used for bucket lookup. The rule is: if `a == b`, then `hash(a)` **must** equal `hash(b)`.
* **Reasoning:** If you define `__eq__` but not `__hash__`, Python sets `__hash__` to `None`, making the object unhashable to prevent logical errors in sets/maps.
* **Mistakes:** Implementing `__eq__` without `__hash__` and trying to put objects in a set.
* **Interviewer Intent:** Ability to use custom objects in data structures efficiently.
* **Sample Code:**

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)
    def __hash__(self):
        return hash((self.x, self.y))

```

* **Line-by-line:** We tuple the state to use Python's native tuple hashing logic.

#### 9. What is the "Mutable Default Argument" trap in `__init__`, and how do you fix it?

* **Answer:** If you use a mutable object (like a list or dict) as a default argument value in `__init__`, that object is created **once** at class definition time, not every time the class is instantiated. All instances will share the *same* list.
* **Fix:** Set the default to `None` and initialize the list inside `__init__`.
* **Reasoning:** This is a classic Python "gotcha" related to how functions (and methods) are objects and defaults are stored in `__defaults__`.
* **Interviewer Intent:** Checks for 4 years of experience (you should have hit this bug by now).
* **Sample Code:**

```python
# BAD
class Student:
    def __init__(self, grades=[]): # Shared list!
        self.grades = grades

# GOOD
class Student:
    def __init__(self, grades=None):
        if grades is None:
            self.grades = [] # New list per instance
        else:
            self.grades = grades

```

#### 10. Explain `__new__` vs `__init__` in the context of Singleton Pattern or Immutable types.

* **Answer:** `__new__` is the only place to return an existing instance (Singleton) or return a modified instance of a built-in immutable type (like subclassing `float` or `str`). `__init__` cannot return anything (must return None).
* **Explanation:** Since immutable objects cannot be changed after creation, `__init__` is too late to alter their value. You must do it in `__new__`.
* **Reasoning:** Advanced control over object creation mechanisms.
* **Interviewer Intent:** Understanding metaclass-adjacent logic and immutability.

#### 11. What is the result of comparing objects of different types using `__eq__`?

* **Answer:** It depends on the implementation. By default, `object.__eq__` compares identity (`is`). If you implement `__eq__`, you should return `NotImplemented` (not `False`) if the type is not recognized.
* **Explanation:** Returning `NotImplemented` signals Python to try the *other* object's `__eq__` (or `__ne__`). If both return `NotImplemented`, Python falls back to object identity.
* **Reasoning:** This allows for symmetric equality logic (e.g., if `A == B` doesn't know how to compare, maybe `B == A` does).
* **Interviewer Intent:** Knowledge of Python's coercion and dispatch protocols.

#### 12. `__repr__` vs `__str__`: When is `__init__` arguments relevant here?

* **Answer:** `__str__` is for end-user readability. `__repr__` is for developer debugging and should ideally return a string that looks like a valid Python expression to recreate the object (calling the constructor).
* **Rule of Thumb:** `eval(repr(obj)) == obj`.
* **Reasoning:** Good `__repr__` makes logging and debugging significantly easier in backend systems.
* **Mistakes:** Implementing only `__str__` and getting ugly output in lists/console.
* **Interviewer Intent:** Professional coding standards.

#### 13. How does `copy.deepcopy` relate to `__init__`? Does it call it?

* **Answer:** `deepcopy` generally does **not** call `__init__`. It creates a new blank instance and copies the state (the `__dict__`) from the old object to the new one recursively.
* **Explanation:** Python restores the state directly to preserve the exact snapshot, rather than re-initializing via logic that might differ from the stored state.
* **Reasoning:** If your `__init__` has side effects (like connecting to a DB), deepcopying might produce an object that isn't connected, or it might prevent unwanted side effects.
* **Interviewer Intent:** Understanding serialization and object state persistence.

#### 14. How does `__init_subclass__` relate to class setup?

* **Answer:** `__init_subclass__` is a hook in a parent class that runs whenever a class inherits from it. It's an alternative to metaclasses for customizing class creation.
* **Reasoning:** It allows the parent to validate or modify the child class *at definition time*, not instantiation time.
* **Comparisons:** Simpler than using a custom `MetaClass`.
* **Interviewer Intent:** Knowledge of modern Python (3.6+) features for framework design.

#### 15. What is the `@dataclass` decorator's impact on `__init__` and equality?

* **Answer:** `@dataclass` automatically generates `__init__`, `__repr__`, and `__eq__` based on type annotations.
* **Explanation:** It reduces boilerplate. `__eq__` is generated comparing all defined fields in order.
* **Reasoning:** Dramatically speeds up development for data-holding objects.
* **Mistakes:** Trying to define a mutable default field without `field(default_factory=...)`.
* **Interviewer Intent:** Familiarity with modern standard library tools.

#### 16. Explain the logic of `self.__class__` vs `type(self)`.

* **Answer:** They are functionally equivalent for getting the class of an instance. However, `type(self)` is generally safer if an object shadows `__class__` attribute, though that is rare.
* **Reasoning:** Used in `__eq__` checks to ensure objects are of the exact same type before comparing attributes.
* **Sample Code:**

```python
def __eq__(self, other):
    if type(self) is not type(other):
        return NotImplemented
    return self.id == other.id

```

#### 17. Why might strict type checking in `__eq__` (`isinstance` vs `type(x) is type(y)`) be controversial?

* **Answer:** `isinstance(other, MyClass)` allows subclasses to be equal to the parent. `type(self) is type(other)` enforces strict type matching.
* **Reasoning:** According to the Liskov Substitution Principle, a subclass should be usable as a parent. However, in value semantics (like two distinct entities), a `ColoredPoint(1,1, 'red')` shouldn't equal a `Point(1,1)`.
* **Interviewer Intent:** Design philosophy and OOP theory.

#### 18. What is `__del__`? Why should you avoid relying on it for cleanup?

* **Answer:** `__del__` is the finalizer. It is called when the garbage collector is about to destroy the object.
* **Issue:** You cannot guarantee *when* (or if) it will be called (e.g., circular references, interpreter shutdown). Exceptions in `__del__` are ignored/printed to stderr.
* **Better Alternative:** Context managers (`__enter__`, `__exit__` with `with` statement).
* **Interviewer Intent:** Resource management best practices.

#### 19. How do slots (`__slots__`) affect `__init__` and object memory?

* **Answer:** `__slots__` restricts the valid attributes of a class to a fixed set, suppressing the creation of `__dict__`.
* **Impact:** `__init__` can only assign to attributes listed in slots. It significantly reduces memory usage and speeds up attribute access.
* **Reasoning:** Useful for objects created in massive quantities (millions of instances).
* **Interviewer Intent:** Performance optimization knowledge.

#### 20. When implementing `__eq__`, why is it risky to access `other.attribute` directly without checking?

* **Answer:** If `other` is `None` or a different type that lacks that attribute, it raises an `AttributeError`.
* **Fix:** Always check type/isinstance first, or use `getattr`, or return `NotImplemented` immediately if types don't match.
* **Code:**

```python
def __eq__(self, other):
    if not isinstance(other, MyClass):
        return NotImplemented
    # Safe to access other.attr now

```

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (3 Questions)

#### 21. Basic Class Setup with String Representation

**Question:** Create a class `Book` with `title` (str) and `author` (str).

1. Initialize attributes.
2. Implement `__str__` to return "Title by Author".
3. Implement `__eq__` so two books are equal if title and author match (case-insensitive).

**Solution:**

```python
class Book:
    def __init__(self, title: str, author: str):
        # Store common case for normalization
        self.title = title
        self.author = author

    def __str__(self) -> str:
        # User-friendly string representation
        return f"{self.title} by {self.author}"

    def __eq__(self, other):
        if not isinstance(other, Book):
            return NotImplemented
        # Case-insensitive comparison
        return (self.title.lower(), self.author.lower()) == \
               (other.title.lower(), other.author.lower())

# Usage
b1 = Book("Dune", "Frank Herbert")
b2 = Book("dune", "frank herbert")
print(b1 == b2)  # True
print(b1)        # Dune by Frank Herbert

```

**Explanation:**

* Uses `isinstance` for safety.
* Converts strings to lower case *during comparison* to preserve original casing in storage.

**Why ask this?** Checks basic syntax, case handling, and `__str__` vs `__repr__` concept.

#### 22. Safe Initialization with Lists

**Question:** Create a class `Classroom` that accepts a list of student names. If no list is provided, it should start empty. Ensure that modifying the external list afterwards does NOT modify the internal class list.

**Solution:**

```python
class Classroom:
    def __init__(self, students=None):
        if students is None:
            self.students = []
        else:
            # Create a shallow copy to prevent external side effects
            self.students = list(students) 

    def add_student(self, name):
        self.students.append(name)

# Usage
my_list = ["Alice", "Bob"]
c = Classroom(my_list)
my_list.append("Charlie") # Modifying external list

print(c.students) # Should be ['Alice', 'Bob']

```

**Common Mistakes:** `self.students = students` (stores a reference to the external mutable list).
**Real App Usage:** A configuration object in a backend service taking a list of allowed IPs.

#### 23. The "Color" Value Object

**Question:** Create a class `Color` accepting R, G, B values. Two colors are equal if their RGB values match.

**Solution:**

```python
class Color:
    def __init__(self, r, g, b):
        self.r = r
        self.g = g
        self.b = b

    def __eq__(self, other):
        if not isinstance(other, Color):
            return NotImplemented
        return (self.r, self.g, self.b) == (other.r, other.g, other.b)

```

**Developer Scenario:** Frontend UI component configuration in Python (e.g., PyQt or Tkinter wrappers).

---

### Medium Level (7 Questions)

#### 24. Singleton Pattern via `__new__`

**Question:** Implement a `DatabaseConnection` class using the Singleton pattern. Ensure only **one** instance is created, no matter how many times `DatabaseConnection()` is called.

**Solution:**

```python
class DatabaseConnection:
    _instance = None # Class-level variable to store the instance

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            print("Creating new instance...")
            # Call super().__new__ to actually allocate memory
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        # Warning: __init__ is called every time __new__ returns an instance!
        # We need to ensure we don't re-initialize connection logic.
        if not hasattr(self, "_initialized"):
            print("Initializing connection...")
            self.connected = True
            self._initialized = True

# Usage
db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(db1 is db2) # True

```

**Complexity:** Time O(1). Space O(1).
**Common Mistakes:** Putting logic in `__init__` without checking if it ran already. Python calls `__init__` on the returned object from `__new__` every time.
**Real App Usage:** Logging handlers, Database pools, Configuration managers.

#### 25. Custom Hashable Class (Employee)

**Question:** Create an `Employee` class with `id` and `name`. Employees are identified strictly by `id`. Allow this class to be used as a key in a dictionary.

**Solution:**

```python
class Employee:
    def __init__(self, emp_id: int, name: str):
        self.emp_id = emp_id
        self.name = name

    def __eq__(self, other):
        if not isinstance(other, Employee):
            return NotImplemented
        return self.emp_id == other.emp_id

    def __hash__(self):
        # Hash must depend ONLY on attributes used in __eq__
        return hash(self.emp_id) 

    def __repr__(self):
        return f"Employee({self.emp_id})"

# Usage
e1 = Employee(101, "John")
e2 = Employee(101, "Johnny") # Same ID, different name
staff_salary = {e1: 50000}

print(staff_salary[e2]) # Returns 50000 because e1 == e2 and hash(e1) == hash(e2)

```

**Why Interviewers Ask:** Tests understanding of the contract between `eq` and `hash`. If `hash` changed based on `name`, the dictionary lookup would fail.

#### 26. Sorting with `__lt__` (Rich Comparison)

**Question:** Create a class `Task` with `priority` (int) and `name` (str). Implement logic so that tasks can be sorted using `sorted()`. Tasks with higher priority number come first. If priorities are equal, sort alphabetically by name.

**Solution:**

```python
from functools import total_ordering

@total_ordering
class Task:
    def __init__(self, name, priority):
        self.name = name
        self.priority = priority

    def __eq__(self, other):
        if not isinstance(other, Task):
            return NotImplemented
        return (self.priority, self.name) == (other.priority, other.name)

    def __lt__(self, other):
        if not isinstance(other, Task):
            return NotImplemented
        # Invert priority for "High priority first" logic
        # If priorities equal, compare names normally
        if self.priority != other.priority:
            return self.priority > other.priority 
        return self.name < other.name

# Usage
tasks = [Task("Clean", 1), Task("Code", 10), Task("Sleep", 10)]
sorted_tasks = sorted(tasks) 
# Result: Code(10), Sleep(10), Clean(1)

```

**Explanation:** `@total_ordering` generates `le`, `gt`, `ge` automatically if you define `eq` and `lt`.
**Real App Usage:** Priority Queues, Job Scheduling systems (Celery).

#### 27. The `__post_init__` with Dataclasses

**Question:** Use `dataclasses`. Create a `Rectangle` class with `width` and `height`. It must automatically calculate an `area` attribute immediately after initialization.

**Solution:**

```python
from dataclasses import dataclass, field

@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False) # Not passed in constructor

    def __post_init__(self):
        # Runs automatically after __init__
        self.area = self.width * self.height

# Usage
r = Rectangle(10, 5)
print(r.area) # 50

```

**Developer Scenario:** Calculating derived fields (prices with tax, full names) automatically in Data Transfer Objects (DTOs).

#### 28. Factory Pattern Constructor (Returning Subclass)

**Question:** Create a class `FileOpener`. If the user passes a filename ending in `.txt`, return a `TextFile` instance. If `.json`, return a `JsonFile` instance. Do this inside the constructor logic.

**Solution:**

```python
class FileHandler:
    pass

class TextFile(FileHandler):
    pass

class JsonFile(FileHandler):
    pass

class FileOpener:
    def __new__(cls, filename):
        # Logic to decide WHICH class to instantiate
        if filename.endswith(".txt"):
            return super().__new__(TextFile)
        elif filename.endswith(".json"):
            return super().__new__(JsonFile)
        else:
            raise ValueError("Unsupported format")

    def __init__(self, filename):
        self.filename = filename

# Usage
f = FileOpener("data.json")
print(type(f)) # <class '__main__.JsonFile'>

```

**Explanation:** This is a rare but powerful use of `__new__` acting as a Factory.
**Note:** `__init__` will be called on the *returned* object. Both `TextFile` and `JsonFile` (or a common parent) should handle `filename` in `__init__`.

#### 29. Deep Equality Check with Recursion

**Question:** Implement a `Node` class for a binary tree. Implement `__eq__` to check if two trees are structurally identical and have identical values.

**Solution:**

```python
class Node:
    def __init__(self, val, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

    def __eq__(self, other):
        # 1. Identity check (optimization)
        if self is other:
            return True
        # 2. Type check
        if not isinstance(other, Node):
            return False
        # 3. Value check
        if self.val != other.val:
            return False
        # 4. Recursive Structure check
        return self.left == other.left and self.right == other.right

# Usage
tree1 = Node(1, Node(2), Node(3))
tree2 = Node(1, Node(2), Node(3))
print(tree1 == tree2) # True

```

**Time Complexity:** O(N) where N is the number of nodes (visits every node).
**Real App Usage:** DOM Tree comparisons, XML parsing, AST (Abstract Syntax Tree) equality checks in compilers.

#### 30. Preventing Attribute Creation (`__slots__`)

**Question:** Create a class `Point3D` that is memory optimized. It should generally *not* allow adding new attributes at runtime (e.g., `p.color = 'red'` should fail).

**Solution:**

```python
class Point3D:
    # Explicitly define allowed attributes
    __slots__ = ('x', 'y', 'z')

    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

    def __repr__(self):
        return f"Point3D({self.x}, {self.y}, {self.z})"

# Usage
p = Point3D(1, 2, 3)
try:
    p.color = "Red"
except AttributeError as e:
    print("Blocked:", e) # 'Point3D' object has no attribute 'color'

```

**Explanation:** `__slots__` prevents the creation of `__dict__`, saving significant RAM for millions of small objects.
**Why interviewers ask this:** It differentiates intermediate Python devs from advanced ones who understand Python's memory model.
