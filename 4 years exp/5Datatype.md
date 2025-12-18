# Python Datatypes — Interview Guide (30 Questions)

This guide covers the fundamental building blocks of Python: its Type System. It explores how Python handles memory, mutability, and object hierarchy for primitive and container types (beyond just Lists/Tuples).

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What is the difference between "Strong Typing" and "Dynamic Typing"? Which is Python?

* **Expected Answer**: Python is **Strongly Typed** and **Dynamically Typed**.
* **Dynamic**: Type checking happens at runtime. You don't declare variable types (e.g., `x = 10` then `x = "hello"` is valid).
* **Strong**: Python does not implicitly convert types during operations if logic forbids it. Adding `"123" + 456` raises a `TypeError`. (Unlike JavaScript, which is Weakly Typed and would result in `"123456"`).


* **Clear Explanation**: Dynamic refers to *when* types are checked (runtime). Strong refers to *how strict* the rules are when mixing types.
* **Why Interviewers Ask This**: To verify you understand the language's fundamental philosophy compared to C++ (Static/Strong) or JS (Dynamic/Weak).
* **Sample Code**:
```python
x = 10       # Dynamic: Type inferred as int
# print("Val: " + x) # Strong: TypeError (Won't implicitly cast int to str)
print("Val: " + str(x)) # Explicit casting required

```



#### 2. `type()` vs `isinstance()`: Which should you use and why?

* **Expected Answer**: Always prefer `isinstance(obj, Class)`.
* **Clear Explanation**:
* `type(obj) == Class`: Checks for an **exact** type match. It fails if `obj` is a subclass of `Class`.
* `isinstance(obj, Class)`: Checks if `obj` is an instance of `Class` **or any subclass**. It supports inheritance (Liskov Substitution Principle).


* **Medium-Depth Reasoning**: In object-oriented code, if you check `type(x) == Animal`, it returns False for a `Dog` instance. `isinstance` correctly identifies a Dog as an Animal.
* **Sample Code**:
```python
class Animal: pass
class Dog(Animal): pass

d = Dog()
print(type(d) == Animal)      # False (Bad check)
print(isinstance(d, Animal))  # True  (Good check)

```



#### 3. What is the relationship between `bool` and `int` in Python?

* **Expected Answer**: `bool` is a subclass of `int`.
* **Clear Explanation**: `True` is treated exactly as `1`, and `False` as `0`. They can be used in arithmetic operations (though it is not recommended for readability).
* **Reasoning**: This is a legacy feature from Python's early days to maintain compatibility. The `bool` class cannot be subclassed further.
* **Common Mistakes**: Thinking `bool` is a completely standalone type.
* **Sample Code**:
```python
print(True + True) # 2
print(isinstance(True, int)) # True

```



#### 4. How does Python handle the `None` type?

* **Expected Answer**: `None` is a singleton object of type `NoneType`.
* **Clear Explanation**: There is only one copy of `None` in the entire Python runtime.
* **Best Practice**: Always compare using `is None` rather than `== None`. `is` checks for object identity (memory address), which is faster and safer (as classes can override `__eq__` to return weird values).
* **Why Interviewers Ask This**: Checking for null/none is the most common operation in programming.
* **Sample Code**:
```python
x = None
if x is None:
    print("It is None")

```



#### 5. Are Strings Mutable or Immutable? Why?

* **Expected Answer**: Strings (`str`) are **Immutable**.
* **Clear Explanation**: Once created, the string content cannot be changed. Operations like `replace` or `upper` return a **new** string object.
* **Reasoning**:
1. **Hashing**: Strings must be immutable to be hashable (usable as dictionary keys).
2. **Safety**: Multiple variables can reference the same string without side effects.
3. **Interning**: Python optimizes memory by storing one copy of common string literals.


* **Sample Code**:
```python
s = "hello"
# s[0] = "H" # Raises TypeError
s = "H" + s[1:] # Creates NEW string object

```



#### 6. What defines a "Falsy" value in Python data types?

* **Expected Answer**: By default, an object is considered "Truthy" unless its class defines `__bool__` or `__len__` methods that return `False` or `0`.
* **Standard Falsy Values**:
* Constants: `None`, `False`.
* Zero numeric: `0`, `0.0`, `0j`.
* Empty sequences/collections: `''`, `()`, `[]`, `{}`, `set()`, `range(0)`.


* **Why Interviewers Ask This**: To see if you write Pythonic checks like `if my_list:` instead of `if len(my_list) > 0:`.

---

### Medium Level (Questions 7–20)

#### 7. `bytes` vs `str`: What is the fundamental difference?

* **Question**: When should you use `bytes` instead of `str`?
* **Expected Answer**:
* `str`: Unicode characters (text) intended for humans.
* `bytes`: Raw sequence of integers (0-255) intended for machines/network/disk.


* **Clear Explanation**: In Python 3, strict separation is enforced. You cannot mix them without explicit `.encode()` (str -> bytes) or `.decode()` (bytes -> str).
* **Scenario**: Reading an image file (`rb` mode) requires `bytes`. Parsing a JSON API response requires `str`.
* **Sample Code**:
```python
s = "Café"
b = s.encode('utf-8')
print(len(s)) # 4 chars
print(len(b)) # 5 bytes (é takes 2 bytes in UTF-8)

```



#### 8. Mutable Default Arguments Trap.

* **Question**: What happens if you define `def func(a=[]):`?
* **Expected Answer**: The default list `[]` is created **once** at function definition time, not every time the function is called.
* **Result**: If you modify `a` inside the function, the changes persist across subsequent calls.
* **Correct Pattern**: Use `None` as the default.
* **Why Interviewers Ask This**: The #1 most common Python interview question regarding datatypes/functions.
* **Sample Code**:
```python
def bad_func(val, l=[]):
    l.append(val)
    return l

print(bad_func(1)) # [1]
print(bad_func(2)) # [1, 2] !!! (Shared object)

```



#### 9. How does Python's `float` handle precision? (IEEE 754)

* **Question**: Why is `0.1 + 0.2 != 0.3` in Python?
* **Expected Answer**: Python floats are standard IEEE 754 double-precision binary floating-point numbers.
* **Reasoning**: Computers store numbers in binary. `0.1` (1/10) results in a repeating binary fraction (like 1/3 in decimal). It cannot be represented exactly. The minor error accumulates.
* **Comparison**: Use `decimal.Decimal` for financial calculations requiring exact precision.
* **Sample Code**:
```python
print(0.1 + 0.2) # 0.30000000000000004

```



#### 10. `set` vs `frozenset`.

* **Question**: What is a `frozenset` and why does it exist?
* **Expected Answer**: `frozenset` is an immutable version of a `set`.
* **Reasoning**: Since normal sets are mutable, they are **not hashable**. Therefore, you cannot use a `set` as a key in a dictionary or as an element inside another set. `frozenset` is hashable, solving this.
* **ASCII Diagram**:
```text
Set (Mutable)      -> Can change content -> Hash changes -> Invalid Key
Frozenset (Fixed)  -> Static content     -> Hash fixed   -> Valid Key

```


* **Sample Code**:
```python
s = {1, 2}
fs = frozenset([1, 2])
d = {fs: "valid"}
# d[s] = "error" # TypeError: unhashable type: 'set'

```



#### 11. Explain Small Integer Caching (Interning).

* **Question**: Why is `a = 256; b = 256; a is b` True, but `a = 300; b = 300; a is b` often False?
* **Expected Answer**: CPython pre-allocates and caches an array of integer objects for small integers (typically -5 to 256).
* **Reasoning**: References to integers in this range point to the *existing* singleton objects in the cache to save memory. Integers outside this range are created as new objects (though standard compilation may optimize this in script files).
* **Why Interviewers Ask This**: To test knowledge of CPython internals vs language specification.

#### 12. `__slots__` and Memory Optimization.

* **Question**: How does `__slots__` affect a class's datatype structure?
* **Expected Answer**: Normally, Python objects store attributes in a dynamic dictionary `__dict__`, which consumes RAM. Defining `__slots__` tells Python to statically allocate memory for a fixed set of attributes only.
* **Trade-off**: You save memory and gain faster attribute access, but you lose the ability to add new attributes dynamically at runtime.
* **Sample Code**:
```python
class Point:
    __slots__ = ['x', 'y'] # No __dict__ created
    def __init__(self, x, y):
        self.x = x
        self.y = y

```



#### 13. What is a "Hashable" object?

* **Question**: What requirements must an object satisfy to be usable as a Dict key?
* **Expected Answer**:
1. It must define a `__hash__()` method (returns an int).
2. It must define an `__eq__()` method.
3. **Stability**: If `a == b`, then `hash(a)` must equal `hash(b)`. The hash value should not change during the object's lifetime.


* **Implication**: Mutable types (list, dict, set) are unhashable because if they change, their hash would change, breaking the lookup bucket logic.

#### 14. `NaN` (Not a Number) peculiarities.

* **Question**: What does `float('nan') == float('nan')` return?
* **Expected Answer**: `False`.
* **Reasoning**: By IEEE 754 standard, NaN is not equal to anything, including itself.
* **How to check**: You must use `math.isnan(x)` or `pandas.isna(x)`. You cannot use `x == x` (a common trick to *detect* NaN is seeing if this returns False).
* **Sample Code**:
```python
import math
x = float('nan')
print(x == x)       # False
print(math.isnan(x)) # True

```



#### 15. The `complex` datatype.

* **Question**: How do you define a complex number in Python and access its parts?
* **Expected Answer**: Use the `j` suffix. e.g., `z = 3 + 4j`.
* **Access**: `z.real` gives the real part (float), `z.imag` gives the imaginary part (float).
* **Why Interviewers Ask This**: Scientific computing awareness.
* **Sample Code**:
```python
z = 1 + 2j
print(type(z)) # <class 'complex'>

```



#### 16. Shallow Copy vs Deep Copy.

* **Question**: Given a list of lists `l = [[1], [2]]`, what is the difference between `list(l)` and `copy.deepcopy(l)`?
* **Expected Answer**:
* **Shallow Copy** (`list(l)`, `l[:]`, `copy.copy`): Creates a new container, but inserts *references* to the original objects found in the original. (Modifying inner lists affects both).
* **Deep Copy**: Recursively copies objects found. Creates fully independent clones.


* **Sample Code**:
```python
import copy
l1 = [[1], [2]]
l2 = copy.deepcopy(l1)
l2[0][0] = 99
print(l1) # [[1], [2]] (Unaffected)

```



#### 17. Python's `id()` function.

* **Question**: What does `id(obj)` return specifically in CPython?
* **Expected Answer**: It returns the integer representing the **memory address** of the object.
* **Usage**: It is the basis for the `is` operator. `a is b` is equivalent to `id(a) == id(b)`.
* **Lifetime**: The ID is unique among simultaneously existing objects, but can be reused after an object is garbage collected.

#### 18. Type Hints: `List` vs `list`.

* **Question**: In Python 3.9+, why do we use `list[int]` instead of `List[int]` from `typing`?
* **Expected Answer**: Python 3.9 introduced **Generic Alias Types**.
* **Explanation**: Standard collections (`list`, `dict`, `tuple`) can now support subscripting `[]` for type hinting directly. The capitalized versions in `typing` (`List`, `Dict`) are now deprecated for this purpose, though they still exist for backward compatibility.
* **Why Interviewers Ask This**: Checks if your knowledge is up-to-date (Modern Python).

#### 19. The `range` object datatype.

* **Question**: Is `range(1000000)` a list?
* **Expected Answer**: No, it is a separate immutable sequence type (`class 'range'`).
* **Efficiency**: It uses O(1) memory. It stores only start, stop, and step values, calculating items on demand (lazy evaluation), rather than storing a million integers in RAM.
* **Sample Code**:
```python
r = range(5)
print(type(r)) # <class 'range'>

```



#### 20. `__class__` assignment (Type transmutation).

* **Question**: Can you change the type of an object at runtime?
* **Expected Answer**: Yes, by assigning to `obj.__class__`.
* **Caveat**: It is dangerous and rarely done. The new class should have a compatible layout (usually only works well for pure python classes without slots or C-extensions).
* **Why Interviewers Ask This**: Extremely advanced edge case to test dynamic language mastery.
* **Sample Code**:
```python
class A: pass
class B: pass
x = A()
x.__class__ = B
print(type(x)) # <class '__main__.B'>

```



---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Safe Integer Conversion

**Question**: Write a function that converts a list of mixed data types (strings, floats, None) into a list of integers. If conversion fails (e.g., "abc" or None), store `0`.

* **Solution**:
```python
def to_integers(data: list[any]) -> list[int]:
    result = []
    for item in data:
        try:
            # Convert float to int (truncates), str to int
            val = int(float(item)) # float cast handles "12.5" strings safely first
            result.append(val)
        except (ValueError, TypeError):
            result.append(0)
    return result

# Usage
raw = ["10", "5.5", "apple", None, 20]
print(to_integers(raw)) # [10, 5, 0, 0, 20]

```


* **Explanation**: We cast to `float` first to handle strings like `"5.5"` (which `int("5.5")` would crash on), then cast to `int`. Catching `ValueError` handles text, `TypeError` handles `None`.
* **Complexity**: Time O(N).

#### 2. Check for Specific Type (Ignoring Subclasses)

**Question**: Write a function `is_strictly_dict` that returns True ONLY if the input is a `dict`, not a subclass like `OrderedDict` or `defaultdict`.

* **Solution**:
```python
def is_strictly_dict(obj) -> bool:
    # type() checks exact type identity
    return type(obj) is dict

# Usage
from collections import OrderedDict
d = {}
od = OrderedDict()
print(is_strictly_dict(d))  # True
print(is_strictly_dict(od)) # False

```


* **Explanation**: This is one of the rare cases where `type()` is preferred over `isinstance()`.
* **Why Interviewers Ask This**: To verify you know how to bypass polymorphism when strictness is required.

#### 3. Count Datatypes in a List

**Question**: Given a list with mixed types, return a dictionary counting how many of each type are present. e.g., `[1, "a", 2]` -> `{'int': 2, 'str': 1}`.

* **Solution**:
```python
def count_types(data: list) -> dict[str, int]:
    counts = {}
    for item in data:
        type_name = type(item).__name__
        counts[type_name] = counts.get(type_name, 0) + 1
    return counts

# Usage
data = [1, 2.5, "hello", 3, "world"]
print(count_types(data)) # {'int': 2, 'float': 1, 'str': 2}

```


* **Line-by-Line**: `type(item).__name__` gets the string representation of the class (e.g., "int").

---

### Medium Level (Questions 4–10)

#### 4. Safe Float Comparison

**Question**: Write a function `floats_equal(a, b)` that returns True if two floats are "close enough" to be considered equal, handling the IEEE 754 precision error.

* **Solution**:
```python
import math

def floats_equal(a: float, b: float, epsilon=1e-9) -> bool:
    # Manual implementation: abs(a - b) < epsilon
    # Pythonic robust implementation:
    return math.isclose(a, b, rel_tol=epsilon)

# Usage
val1 = 0.1 + 0.2
val2 = 0.3
print(val1 == val2)          # False
print(floats_equal(val1, val2)) # True

```


* **Explanation**: `math.isclose` is the standard library solution. It checks relative tolerance (handling large numbers) and absolute tolerance (handling numbers near zero).
* **Common Mistake**: `a == b` for floats is a junior developer mistake.

#### 5. Recursive Deep Dictionary Merge

**Question**: You have two dictionaries representing configurations. `dict_a` is default, `dict_b` is user overrides. Merge them. If both have a dictionary at the same key, merge recursively. Otherwise `dict_b` wins.

* **Solution**:
```python
def deep_merge(target: dict, source: dict) -> dict:
    # Iterate over source keys
    for key, value in source.items():
        if key in target and isinstance(target[key], dict) and isinstance(value, dict):
            # Both are dicts, recurse
            deep_merge(target[key], value)
        else:
            # Overwrite or add
            target[key] = value
    return target

# Usage
defaults = {"db": {"host": "localhost", "port": 5432}}
overrides = {"db": {"port": 8000}, "debug": True}
print(deep_merge(defaults, overrides))
# {'db': {'host': 'localhost', 'port': 8000}, 'debug': True}

```


* **Explanation**: Standard dict `.update()` is shallow (it would replace the whole "db" dictionary). Recursion is needed to preserve nested keys like "host".
* **Complexity**: Time O(N) where N is total nodes in the dictionary tree.

#### 6. Custom Immutable Class (The Dataclass Way)

**Question**: Create a class `Coordinate` holding `x` and `y`. Make it immutable (read-only) and hashable so it can be used in a set. Use Python 3.7+ features.

* **Solution**:
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    x: int
    y: int

# Usage
p1 = Coordinate(10, 20)
# p1.x = 30 # Raises FrozenInstanceError
s = {p1} # Works because frozen=True adds __hash__
print(s)

```


* **Explanation**: The `@dataclass(frozen=True)` decorator automatically generates `__init__`, `__repr__`, `__eq__`, and `__hash__`. It also overrides `__setattr__` to prevent modification.
* **Developer Scenario**: Creating Value Objects (DDD pattern) in backend systems.

#### 7. Flattening Arbitrarily Nested Data

**Question**: Write a function that flattens a nested structure containing lists, tuples, and integers into a single list of integers. `[1, [2, (3, 4)], 5]` -> `[1, 2, 3, 4, 5]`.

* **Solution**:
```python
def flatten(data) -> list[int]:
    result = []
    for item in data:
        if isinstance(item, (list, tuple)):
            # Recurse if it's a container
            result.extend(flatten(item))
        else:
            # Append if it's a primitive
            result.append(item)
    return result

# Usage
nested = [1, [2, (3, 4)], 5]
print(flatten(nested))

```


* **Explanation**: `isinstance` checks if the item is a container. Recursion handles infinite depth.
* **Complexity**: Time O(N) (total elements).

#### 8. Converting Bytes to Hex and Back

**Question**: You receive a byte string `b'\x00\xFF'`. Convert it to a hex string `"00ff"` and back to integers.

* **Solution**:
```python
def process_bytes(data: bytes):
    # Bytes to Hex
    hex_str = data.hex()
    print(f"Hex: {hex_str}")

    # Hex to Bytes
    back_to_bytes = bytes.fromhex(hex_str)

    # Bytes to Int List
    int_values = list(data)
    return int_values

# Usage
data = b'\x00\xFF\x10'
print(process_bytes(data))
# Hex: 00ff10
# [0, 255, 16]

```


* **Explanation**: `bytes` objects iterate as integers (0-255). `.hex()` is a built-in helper for string representation.
* **Why Interviewers Ask This**: Cryptography and binary protocol handling.

#### 9. Dictionary Key Normalization

**Question**: A dictionary contains keys as strings, integers, and booleans. `{'1': 'a', 1: 'b', True: 'c'}`. Clean this up so all keys are converted to Strings. Handle collisions by keeping the *last* seen value.

* **Solution**:
```python
def normalize_keys(data: dict) -> dict[str, any]:
    clean_dict = {}
    for key, value in data.items():
        new_key = str(key)
        clean_dict[new_key] = value
    return clean_dict

# Usage
# Note: In Python, True == 1.
# So {1: 'b', True: 'c'} results in {1: 'c'} immediately upon creation!
# Let's assume input comes from a list of tuples to simulate raw data issue
raw_data = [('1', 'a'), (1, 'b'), (True, 'c')]

# Logic applied
result = {}
for k, v in raw_data:
    result[str(k)] = v

print(result)
# '1' -> 'a'
# 1 -> '1' -> Overwrites 'a' with 'b'
# True -> 'True' -> distinct key
# Result: {'1': 'b', 'True': 'c'}

```


* **Explanation**: This highlights a trick question. In Python `1` and `True` hash to the same value. If passed as a literal dict, data is already lost. If passed as tuples, we can recover it. The solution handles the conversion.

#### 10. Runtime Type Checking Decorator

**Question**: Write a decorator `@enforce_types` that checks if the arguments passed to a function match the type hints signature. If not, raise `TypeError`.

* **Solution**:
```python
import functools
import inspect

def enforce_types(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Get type hints
        hints = func.__annotations__
        # Get bound arguments (maps args to parameter names)
        sig = inspect.signature(func)
        bound = sig.bind(*args, **kwargs)

        for name, value in bound.arguments.items():
            if name in hints:
                expected_type = hints[name]
                if not isinstance(value, expected_type):
                    raise TypeError(f"Arg '{name}' must be {expected_type}, got {type(value)}")

        return func(*args, **kwargs)
    return wrapper

# Usage
@enforce_types
def add(a: int, b: int):
    return a + b

# add(1, 2) # Works
# add(1, "2") # Raises TypeError

```


* **Explanation**: Uses `inspect.signature` to map positional arguments (`1, 2`) to their names (`a, b`) so we can look up the correct annotation.
* **Complexity**: Adds reflection overhead to every call. Not for tight loops.
* **Real App Usage**: Data validation libraries like Pydantic use similar (but more optimized) logic.
