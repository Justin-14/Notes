# Python Tuples — Interview Guide (30 Questions)

This comprehensive guide covers Python Tuples from the perspective of a Senior Technical Interviewer, designed for a developer with 4 years of experience. It moves beyond basic syntax into memory management, hashing mechanics, and architectural use cases.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. Tuple vs. List: What is the fundamental difference, and when should you use one over the other?

* **Expected Answer**: Tuples are immutable sequences, while lists are mutable. Tuples are generally used for heterogeneous data (records, distinct field types) or hashable keys, while lists are for homogeneous data (collections of similar items) meant to be modified.
* **Clear Explanation**: Once created, a tuple cannot be changed (no appending, removing, or re-assigning indices). This immutability allows Python to optimize memory allocation and makes tuples hashable.
* **Medium-Depth Reasoning**: Since tuples are immutable, the Python memory allocator can store them in a single block of memory (unlike lists, which need two blocks: one for the list object and one for the pointer array to allow resizing).
* **Time & Space Complexity**:
* Creation: O(n)
* Access: O(1)


* **Common Mistakes**: Using a tuple for a collection that needs to grow (e.g., collecting logs) or using a list for fixed configuration data.
* **Why Interviewers Ask This**: To verify if the candidate understands mutability and basic architectural intent.
* **Sample Code**:
```python
# List: Mutable, used for collections
users = ["Alice", "Bob"]
users.append("Charlie")

# Tuple: Immutable, used for records
# (Latitude, Longitude)
location = (40.7128, -74.0060)
# location[0] = 41.000  # Raises TypeError

```


* **Line-by-line Explanation**:
* `users`: A list that changes size dynamically.
* `location`: A fixed pairing of data where index 0 has a specific meaning (lat) and index 1 another (long).



#### 2. How do you create a tuple with a single element? What is the "trailing comma" trap?

* **Expected Answer**: You must include a trailing comma: `t = (1,)`. Without the comma, Python treats parentheses as mathematical grouping operators.
* **Clear Explanation**: `(1)` evaluates to the integer `1`. `(1,)` evaluates to a tuple containing the integer `1`. The comma is what triggers tuple construction, not the parentheses (except for the empty tuple `()`).
* **Medium-Depth Reasoning**: The parser sees `(expression)` as grouping. To disambiguate a tuple literal from an expression, syntax rules require the comma.
* **Common Mistakes**: Writing `singleton = ("hello")` and expecting a tuple, but getting a string. This crashes code later when iterating (iterating a string yields characters, not the string item).
* **Why Interviewers Ask This**: It is a classic Python "gotcha" that causes runtime bugs in production.
* **Sample Code**:
```python
x = (50)
y = (50,)
print(type(x))  # <class 'int'>
print(type(y))  # <class 'tuple'>

```



#### 3. Can a Tuple be used as a Dictionary Key? Why?

* **Expected Answer**: Yes, but only if all elements within the tuple are also immutable (hashable).
* **Clear Explanation**: Dictionaries require keys to be hashable. Tuples implement `__hash__`. However, if a tuple contains a mutable object (like a list), the tuple itself becomes unhashable.
* **Medium-Depth Reasoning**: The hash of a tuple is calculated based on the hashes of its items. If an item changes (like a list), its hash would change, breaking the dictionary lookup logic. Therefore, Python forbids it.
* **Common Mistakes**: Assuming *all* tuples are valid keys. `d = {(1, [2]): "val"}` raises `TypeError: unhashable type: 'list'`.
* **Why Interviewers Ask This**: To test understanding of hashability and recursive immutability.
* **Sample Code**:
```python
valid_key = (1, 2)
invalid_key = (1, [2])

d = {}
d[valid_key] = "safe"
# d[invalid_key] = "error" # Raises TypeError

```



#### 4. Explain Tuple Unpacking (Destructuring) syntax.

* **Expected Answer**: Unpacking allows assigning elements of an iterable (like a tuple) to multiple variables in a single statement.
* **Clear Explanation**: Python maps the values on the right to the variables on the left by position. The number of variables must match the number of elements (unless `*` is used).
* **Time & Space Complexity**: O(n) where n is the tuple length.
* **Why Interviewers Ask This**: It is idiomatic Python and heavily used in returning multiple values from functions.
* **Sample Code**:
```python
record = ("John", 30, "Developer")
name, age, job = record
print(job) # Developer

```



#### 5. What are the only two public methods available on a tuple object?

* **Expected Answer**: `count(value)` and `index(value)`.
* **Clear Explanation**: Since tuples are immutable, they lack methods like `append`, `extend`, `remove`, `pop`, or `sort`. They only support read-only inquiry.
* **Medium-Depth Reasoning**:
* `count(x)`: Returns number of occurrences of x.
* `index(x)`: Returns the first index of x (raises ValueError if missing).


* **Comparison**: Lists have ~11 methods; tuples have 2.
* **Why Interviewers Ask This**: To ensure you don't hallucinate methods like `find` or `get` on tuples.
* **Sample Code**:
```python
t = (10, 20, 10, 30)
print(t.count(10)) # 2
print(t.index(20)) # 1

```



#### 6. What is a `NamedTuple`?

* **Expected Answer**: A subclass of tuple that allows accessing elements by name (attribute lookup) as well as by index.
* **Clear Explanation**: Found in `collections` (and `typing` for generic version), it adds readability to tuples used as records without the memory overhead of a full class/dictionary.
* **Why Interviewers Ask This**: It distinguishes a "scripter" from a "developer" who cares about code readability and maintainability.
* **Sample Code**:
```python
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(10, 20)
print(p.x)  # 10 (Readable)
print(p[0]) # 10 (Backwards compatible)

```



---

### Medium Level (Questions 7–20)

#### 7. The "Immutable" Tuple Paradox: Can the contents of a tuple change?

* **Question**: If tuples are immutable, explain how `t = (1, [2, 3])` allows `t[1].append(4)`.
* **Expected Answer**: A tuple is immutable in that it stores fixed object references (pointers). It cannot be made to point to different objects. However, if one of those referenced objects is mutable (like a list), that object *can* change internally.
* **Clear Explanation**: The tuple guarantees "Object A is at index 0, Object B is at index 1". It does not guarantee the *state* of Object B.
* **ASCII Diagram**:
```text
Tuple Memory:  [ ref_to_int_1 ] [ ref_to_list_obj ]
                       |                 |
                       v                 v
                  (Integer 1)      (List Object)  <-- Mutable!
                                   [ 2, 3, ... ]

```


* **Common Mistakes**: Thinking immutability applies recursively to the entire data structure depth.
* **Why Interviewers Ask This**: This is the primary distinction between "shallow" immutability and deep immutability.
* **Sample Code**:
```python
t = (1, [])
t[1].append("change")
print(t) # (1, ['change'])
# t[1] = [] # This would raise TypeError (assigning new pointer)

```



#### 8. Why are Tuples more memory efficient than Lists?

* **Question**: Why does `sys.getsizeof(tuple(iter))` often return fewer bytes than `sys.getsizeof(list(iter))`?
* **Expected Answer**: Lists are over-allocated to optimize for `append` operations (amortized O(1)). Tuples are fixed-size; Python allocates the exact amount of memory needed for the items, with no "extra" capacity for growth.
* **Medium-Depth Reasoning**: A list maintains an internal dynamic array. When it fills up, it allocates a larger array (usually 1.125x or 2x size). A tuple knows its size at creation time, requiring only `sizeof(PyObject_VAR_HEAD) + n * sizeof(PyObject*)`.
* **Why Interviewers Ask This**: Backend optimization. If you are caching millions of records, using tuples instead of lists can save GBs of RAM.
* **Sample Code**:
```python
import sys
l = [1, 2, 3]
t = (1, 2, 3)
print(sys.getsizeof(l)) # Often 88 or 120 bytes (includes over-allocation)
print(sys.getsizeof(t)) # Often 48 or 64 bytes (exact fit)

```



#### 9. Explain the performance difference in creation: `list()` vs `tuple()`.

* **Question**: Which is faster to create: `[1, 2, 3]` or `(1, 2, 3)`? Why?
* **Expected Answer**: The tuple `(1, 2, 3)` is faster.
* **Medium-Depth Reasoning**:
1. **Constant Folding**: For small tuple literals containing constants, the Python bytecode compiler calculates the tuple *at compile time* and stores it as a constant in the code object.
2. **Lists**: Lists are always constructed *at runtime* because they are mutable and must be distinct objects every time.
3. **Memory**: Even for non-constant creation, avoiding the over-allocation logic of lists makes tuples slightly faster.


* **Time Complexity**: Both O(n), but tuple has a smaller constant factor.
* **Sample Code**:
```python
import dis
def make_list(): return [1, 2, 3]
def make_tuple(): return (1, 2, 3)

dis.dis(make_tuple) # Shows: LOAD_CONST
dis.dis(make_list)  # Shows: BUILD_LIST

```



#### 10. Advanced Unpacking: How does the `*` operator work in tuple assignment?

* **Question**: Given `t = (1, 2, 3, 4, 5)`, how do you get the first, last, and "middle" elements in one line?
* **Expected Answer**: Use the star (`*`) operator (extended iterable unpacking).
* **Clear Explanation**: `first, *middle, last = t`. The `*` variable captures all "leftover" items as a **list** (note: it becomes a list, not a tuple).
* **Edge Cases**:
* `head, *tail = (1,)` -> `head=1`, `tail=[]`.
* Only one starred expression is allowed in assignment.


* **Why Interviewers Ask This**: Essential for processing variable-length records (e.g., CSV rows where the first column is ID and rest are data).
* **Sample Code**:
```python
t = (10, 20, 30, 40, 50)
first, *mid, last = t
print(mid) # [20, 30, 40] -> It's a LIST!

```



#### 11. How does Tuple Hashing work internally?

* **Question**: How does Python calculate `hash((1, 2))`?
* **Expected Answer**: Python computes the hash of a tuple by combining the hashes of its individual elements using a specific algorithm (XOR-like mixing with multipliers).
* **Medium-Depth Reasoning**: The algorithm iterates over the tuple items. For each item, it calculates `hash(item)` and mixes it into the current accumulator. This ensures that `(1, 2)` hashes differently than `(2, 1)`.
* **Critical Detail**: If *any* element raises a `TypeError` during hashing (because it's mutable), the tuple hash fails.
* **Why Interviewers Ask This**: To verify knowledge of data structures. This explains why `(1, 2) == (1, 2)` is O(n) but looking it up in a set is O(1) (average).

#### 12. NamedTuple vs. DataClass vs. Dictionary: When to use which?

* **Question**: You have a data record. When do you choose a NamedTuple over a Dictionary or a DataClass?
* **Expected Answer**:
* **Dictionary**: When keys are dynamic/unknown until runtime, or data is messy/unstructured.
* **NamedTuple**: When the structure is fixed, you want immutability, memory efficiency (better than Dict/Class), and tuple-like unpacking.
* **DataClass**: When you want a mutable container, support for inheritance, defaults, or methods attached to the data.


* **Comparison**: NamedTuples are effectively "structs". DataClasses are "classes with boilerplate removed".
* **Why Interviewers Ask This**: Architectural decision making.

#### 13. Returning Multiple Values: What actually happens under the hood?

* **Question**: When you write `return a, b`, what is Python actually returning?
* **Expected Answer**: Python constructs and returns a single **Tuple** containing `(a, b)`.
* **Clear Explanation**: The comma creates the tuple. There is no such thing as returning multiple objects in Python; you return one container object.
* **Reasoning**: The caller then implicitly unpacks this tuple: `x, y = func()`.
* **Sample Code**:
```python
def func():
    return 1, 2

x = func()
print(type(x)) # <class 'tuple'>

```



#### 14. Explain Tuple Interning (Singleton optimization).

* **Question**: Why is `() is ()` True, and sometimes `(1, 2) is (1, 2)` True?
* **Expected Answer**: Python optimizes memory by "interning" (reusing) the empty tuple `()`. It is a singleton.
* **Medium-Depth Reasoning**: Python may also reuse small tuples (similar to small integers) during compilation (constant folding). However, this behavior is implementation-specific (CPython) and shouldn't be relied upon for logic, only performance.
* **Why Interviewers Ask This**: Deep internals knowledge.
* **Sample Code**:
```python
t1 = ()
t2 = ()
print(t1 is t2) # True (Always)

```



#### 15. Tuples as "Structural" vs. Lists as "Semantical".

* **Question**: In type theory usage within Python, what does `Tuple[int, str]` imply vs `List[int]`?
* **Expected Answer**:
* `Tuple[int, str]`: Structure is positional. Index 0 is strictly an int, Index 1 is strictly a string. Length is fixed.
* `List[int]`: A collection of indefinite length where every item is an int.


* **Why Interviewers Ask This**: To see if you understand how to type-hint complex data structures for large codebases.

#### 16. Multithreading: Are Tuples thread-safe?

* **Question**: Does using a tuple protect you from race conditions?
* **Expected Answer**: The *tuple itself* is thread-safe because it cannot be modified (no race condition on resizing or overwriting slots). However, the *objects inside* are not automatically protected.
* **Medium-Depth Reasoning**: Reading a tuple from multiple threads is safe without locks. Modifying a mutable object *inside* a shared tuple (e.g., `t[0].append(x)`) still requires locking.
* **Why Interviewers Ask This**: Concurrency safety is a critical senior-level topic.

#### 17. The `+=` Operator on Tuples.

* **Question**: `t = (1, 2); t += (3,)`. Explain what happens in memory.
* **Expected Answer**: Since tuples are immutable, `+=` does NOT modify the object in place (unlike lists).
* **Step-by-Step**:
1. Python calculates `t + (3,)` creating a **NEW** tuple object at a new memory address.
2. It rebinds the variable name `t` to this new object.


* **Comparison**: For lists, `+=` calls `extend()` (in-place). For tuples, it calls `__add__` (new object).
* **Sample Code**:
```python
t = (1,)
old_id = id(t)
t += (2,)
print(id(t) == old_id) # False

```



#### 18. Swapping Variables: The Pythonic Way.

* **Question**: How does `a, b = b, a` work internally?
* **Expected Answer**: It utilizes implicit tuple packing and unpacking.
* **Detailed Steps**:
1. The RHS `b, a` creates a temporary tuple `(b_val, a_val)`.
2. The LHS performs tuple unpacking, extracting the values into `a` and `b`.
3. This happens on the stack; optimizing compilers (CPython) often optimize the tuple creation away using `ROT_TWO` opcodes for simple swaps.


* **Why Interviewers Ask This**: It’s the signature Python syntax.

#### 19. Tuple Slicing and Memory.

* **Question**: If `t` is a huge tuple, does `sub = t[0:100]` copy the elements?
* **Expected Answer**: It creates a **shallow copy**.
* **Explanation**: `sub` is a new tuple object containing pointers to the first 100 objects of `t`. It does not deep-copy the objects themselves.
* **Memory Implication**: Creating slices of tuples creates new tuple shells, which costs memory, but the heavy objects inside are shared by reference.
* **Sample Code**:
```python
obj = {"data": "heavy"}
t = (obj, obj)
sub = t[0:1]
print(sub[0] is t[0]) # True (Same object reference)

```



#### 20. Using Tuples in Sets.

* **Question**: Can you have a set of tuples? A set of lists?
* **Expected Answer**: You can have a set of tuples (if items are hashable). You cannot have a set of lists.
* **Reasoning**: Sets rely on hashing to ensure uniqueness.
* **Scenario**: Removing duplicates from a list of coordinates `[(1,1), (2,2), (1,1)]`.
* Solution: `set(list_of_tuples)`.


* **Why Interviewers Ask This**: Deduplication logic is a common coding task.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Basic Coordinate Lookup

**Question**: Write a function that accepts a list of tuples representing `(x, y)` coordinates and a target `(x, y)`. Return `True` if the target exists, `False` otherwise. Do not use loops if possible.

* **Solution**:
```python
def has_coordinate(coords: list[tuple[int, int]], target: tuple[int, int]) -> bool:
    # Utilizing the 'in' operator which relies on tuple equality
    return target in coords

# Usage
points = [(1, 2), (3, 4), (5, 6)]
print(has_coordinate(points, (3, 4))) # True

```


* **Explanation**: The `in` operator on a list iterates effectively checking `item == target`. Since tuple equality checks elements recursively, this works out of the box.
* **Complexity**: Time O(N), Space O(1).
* **Mistake**: Manually iterating `for x, y in coords: if x == tx and y == ty...` (Valid, but unpythonic).

#### 2. Find Index of Element

**Question**: Given a tuple of strings, return the index of the string "admin". If not found, return -1.

* **Solution**:
```python
def find_admin(roles: tuple[str, ...]) -> int:
    try:
        return roles.index("admin")
    except ValueError:
        return -1

# Usage
data = ("guest", "editor", "admin")
print(find_admin(data)) # 2

```


* **Explanation**: Tuples have an `.index()` method. However, it raises `ValueError` if missing, so we must wrap it in try/except for safety.
* **Complexity**: Time O(N), Space O(1).
* **Real App Usage**: Checking user permission roles in a backend API middleware.

#### 3. Convert List of Tuples to Dictionary

**Question**: You have a list of `(key, value)` tuples from a database query. Convert it into a dictionary. If keys are duplicated, keep the *last* occurring value.

* **Solution**:
```python
def to_dict(data: list[tuple[str, int]]) -> dict[str, int]:
    # The dict() constructor accepts an iterable of (key, value) tuples
    return dict(data)

# Usage
query_result = [("a", 1), ("b", 2), ("a", 3)]
print(to_dict(query_result)) # {'a': 3, 'b': 2}

```


* **Explanation**: The `dict()` constructor iterates the list. When it encounters a duplicate key ("a"), it overwrites the previous entry, naturally satisfying the "keep last" requirement.
* **Complexity**: Time O(N), Space O(N).
* **Developer Scenario**: Processing configuration overrides where later configs override earlier ones.

---

### Medium Level (Questions 4–10)

#### 4. Sort Tuple List by Specific Element (Lambda)

**Question**: You have a list of tuples: `[('Item A', 15.50), ('Item B', 12.00), ('Item C', 15.50)]`. Sort them first by price (ascending), and then by name (descending) for ties.

* **Solution**:
```python
def custom_sort(items: list[tuple[str, float]]) -> list[tuple[str, float]]:
    # Sort key returns a tuple: (primary_sort, secondary_sort)
    # For descending string sort, we can't negate strings.
    # So we use standard sort logic or negate numbers.
    # Python sort is stable. We can chain sorts or use a complex key.

    # Strategy: Tuple comparison works element by element.
    # But we can't do -string.
    # Alternative: Sort by Name (desc) first, then Price (asc).
    items.sort(key=lambda x: x[0], reverse=True) # Secondary: Name Desc
    items.sort(key=lambda x: x[1])               # Primary: Price Asc (Stable sort preserves previous order)

    return items

# Usage
data = [('Item A', 15.50), ('Item B', 12.00), ('Item C', 15.50)]
print(custom_sort(data))
# Output: [('Item B', 12.0), ('Item C', 15.5), ('Item A', 15.5)]

```


* **Explanation**: Python's Timsort is stable. To sort by multiple criteria where directions (asc/desc) differ, it is often clearest to perform multiple sorts, starting from the *least* significant criteria (name) to the *most* significant (price).
* **Complexity**: Time O(N \log N).
* **Common Mistake**: Trying to write `key=lambda x: (x[1], -x[0])` (fails because you cannot negate a string).
* **Real App Usage**: E-commerce product listing (Amazon: Sort by Price, then Relevance).

#### 5. Flatten Tuple of Tuples

**Question**: Given a nested structure `((1, 2), (3, 4), (5,))`, write a generator to flatten it into a single sequence `1, 2, 3, 4, 5`.

* **Solution**:
```python
from typing import Generator, Any

def flatten_tuples(nested: tuple[tuple[Any, ...], ...]) -> Generator[Any, None, None]:
    for inner_tuple in nested:
        for item in inner_tuple:
            yield item
        # Alternatively: yield from inner_tuple

# Usage
data = ((1, 2), (3, 4), (5,))
print(list(flatten_tuples(data))) # [1, 2, 3, 4, 5]

```


* **Explanation**: Using `yield` (or `yield from` in Python 3.3+) creates a memory-efficient generator that handles the stream without creating a massive intermediate list.
* **Complexity**: Time O(N) (total elements), Space O(1) (iterator overhead).
* **Why Interviewers Ask This**: Testing generator knowledge vs. brute-force list concatenation.

#### 6. Group Data using Tuples (Dictionary Aggregation)

**Question**: Input: List of `(Category, Product)` tuples. Output: Dictionary `{'Category': [products]}`.
`data = [('Fruit', 'Apple'), ('Tech', 'Mouse'), ('Fruit', 'Banana')]`

* **Solution**:
```python
from collections import defaultdict

def group_by_category(items: list[tuple[str, str]]) -> dict[str, list[str]]:
    grouped = defaultdict(list)

    for category, product in items:
        grouped[category].append(product)

    return dict(grouped)

# Usage
data = [('Fruit', 'Apple'), ('Tech', 'Mouse'), ('Fruit', 'Banana')]
print(group_by_category(data))
# {'Fruit': ['Apple', 'Banana'], 'Tech': ['Mouse']}

```


* **Line-by-Line**:
* `defaultdict(list)`: Eliminates the need to check `if key in dict`.
* `category, product`: Unpacks the tuple immediately in the loop header.


* **Complexity**: Time O(N), Space O(N).
* **Real App Usage**: Grouping notifications by app name (Instagram: "3 notifications from Facebook").

#### 7. NamedTuple for CSV Parsing

**Question**: You are reading a CSV file without headers. Columns: `[ID, Username, Email]`. Map these to a clean object structure and filter out users with missing emails.

* **Solution**:
```python
from collections import namedtuple

# Define the schema
User = namedtuple('User', ['id', 'username', 'email'])

def parse_users(rows: list[list[str]]) -> list[User]:
    valid_users = []
    for row in rows:
        # Unpack list row into NamedTuple constructor arguments
        # Note: row must have exactly 3 elements
        user = User._make(row) # or User(*row)

        if user.email: # Check if email is not empty
            valid_users.append(user)

    return valid_users

# Usage
raw_data = [
    ["1", "alice", "alice@example.com"],
    ["2", "bob", ""]
]
results = parse_users(raw_data)
print(results[0].username) # "alice"

```


* **Explanation**: `namedtuple` makes the code self-documenting. `User._make(iterable)` is a factory method specifically for creating named tuples from existing iterables.
* **Why Interviewers Ask This**: Tests ability to write readable, maintainable data processing code.

#### 8. Memoization using Tuples as Keys

**Question**: Implement a simple cache for a function `add(a, b)` using a dictionary, where the arguments are the key.

* **Solution**:
```python
cache = {}

def cached_add(a: int, b: int) -> int:
    # Create a tuple key from arguments
    key = (a, b)

    if key in cache:
        print("Hit!")
        return cache[key]

    print("Miss - Calculating")
    result = a + b
    cache[key] = result
    return result

# Usage
cached_add(2, 3) # Miss
cached_add(2, 3) # Hit

```


* **Explanation**: This relies on the property that tuples are hashable. We bundle the function arguments `(a, b)` into a tuple to use as the dictionary key. A list `[a, b]` would fail here.
* **Developer Scenario**: Caching API responses based on `(endpoint, params)`.

#### 9. Merging Sorted Tuples

**Question**: You have two *sorted* tuples `t1 = (1, 3, 5)` and `t2 = (2, 4, 6)`. Merge them into a single sorted tuple without sorting from scratch (which would be O(N \log N)). Target O(N).

* **Solution**:
```python
def merge_sorted_tuples(t1: tuple[int, ...], t2: tuple[int, ...]) -> tuple[int, ...]:
    result = []
    i = j = 0

    # Two-pointer approach
    while i < len(t1) and j < len(t2):
        if t1[i] < t2[j]:
            result.append(t1[i])
            i += 1
        else:
            result.append(t2[j])
            j += 1

    # Add remaining elements
    result.extend(t1[i:])
    result.extend(t2[j:])

    return tuple(result)

# Usage
print(merge_sorted_tuples((1, 3, 5), (2, 4, 6))) # (1, 2, 3, 4, 5, 6)

```


* **Explanation**: Standard Merge Sort merge step. Since inputs are already sorted, we only need to iterate once.
* **Time Complexity**: O(N + M) where N and M are lengths of tuples.
* **Space Complexity**: O(N + M) for the result.
* **Real App Usage**: Merging timeline feeds from two different sources (e.g., local cache + server response) ordered by timestamp.

#### 10. Identifying "Mutable" Tuple Bugs

**Question**: The following code throws an error but *also* modifies the data. Explain why and fix it.

```python
t = (1, [2, 3])
try:
    t[1] += [4]
except TypeError:
    pass
print(t)

```

* **Solution**:
```python
# The Problem:
# t[1] += [4] is equivalent to:
# x = t[1]
# x.extend([4])  <-- This SUCCEEDS (List is modified)
# t[1] = x       <-- This FAILS (Tuple assignment raises TypeError)

# The Fix: Avoid augmented assignment (+=) on mutable items inside tuples.
# Use direct method calls on the object instead.

t = (1, [2, 3])
t[1].extend([4]) # Safe, atomic modification of the list
print(t) # (1, [2, 3, 4])

```


* **Explanation**: `+=` is not atomic for tuples containing mutable objects. It performs the operation (modifying the list in place) *then* tries to re-assign the result back to the tuple index, which is forbidden.
* **Why Interviewers Ask This**: An advanced edge case demonstrating deep knowledge of Python's bytecode execution order.
