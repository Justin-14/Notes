# Python Tuples — Interview Guide (30 Questions)

**Target Audience:** Python Developer (4 Years Experience)

**Topic:** Python Tuples (Immutability, Hashing, Performance, Memory)

**Total Questions:** 30 (20 Theory, 10 Coding)

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Part A: Easy Level (6 Questions)

#### 1. Fundamental Difference: Tuple vs List

**Question:** What is the most significant difference between a list and a tuple, and why would you choose one over the other?

**Expected Answer:** The primary difference is **immutability**. Lists are mutable (can be changed), while tuples are immutable (cannot be changed after creation). You choose a tuple when the data should not be modified, for performance gains, or when you need a hashable object (like a dictionary key).

**Detailed Explanation:**
Because tuples are immutable, Python can optimize their memory allocation. Once defined, their size and contents are fixed.

**Time & Space Complexity:**

* Creation: O(N)
* Access: O(1)

**Common Mistakes:** Thinking that tuples are just "read-only lists." While similar, their semantic usage is different: lists are for homogeneous data (sequences), and tuples are often for heterogeneous data (records).

**Why Interviewers Ask This:** To ensure the candidate understands the basic architectural choices in Python's data structures.

---

#### 2. Tuple Immutability vs Member Mutability

**Question:** If a tuple contains a list, like `t = (1, 2, [3, 4])`, can you modify the list inside it? Does this violate immutability?

**Expected Answer:** Yes, you can modify the list (e.g., `t[2].append(5)`). This does **not** violate tuple immutability because the tuple only stores the **reference** to the list. The reference remains the same, even if the object it points to changes.

**ASCII Diagram:**
```text
Tuple (ID: 500)
Index 0: Value 1
Index 1: Value 2
Index 2: Reference (ID: 900) ----> List [3, 4, 5] (ID: 900)
```

**Sample Code:**
```python
t = (1, 2, [3, 4])
t[2].append(5)
print(t) # (1, 2, [3, 4, 5])
# t[2] = [10] # This WOULD raise a TypeError
```

---

#### 3. Defining a Single-Element Tuple

**Question:** How do you define a tuple with exactly one element? What happens if you write `x = (5)`?

**Expected Answer:** A single-element tuple requires a trailing comma: `x = (5,)`. If you write `x = (5)`, Python treats the parentheses as a grouping operator, making `x` an integer.

**Sample Code:**
```python
a = (5)   # Type: int
b = (5,)  # Type: tuple
print(type(a), type(b))
```

---

#### 4. Tuple Unpacking

**Question:** What is tuple unpacking, and how does the `*` (splat) operator work within it?

**Expected Answer:** Unpacking allows you to assign tuple elements to multiple variables in one line. The `*` operator captures "the rest" of the elements into a list.

**Sample Code:**
```python
data = (1, 2, 3, 4, 5)
a, *b, c = data
# a = 1, b = [2, 3, 4], c = 5
```

---

#### 5. Hashing and Tuples

**Question:** Can any tuple be used as a key in a dictionary?

**Expected Answer:** No. A tuple can only be used as a dictionary key if all of its elements are also **hashable** (immutable). If a tuple contains a list or another dictionary, it becomes unhashable.

**Why Interviewers Ask This:** Tests understanding of the requirements for dictionary keys and the `hash()` function.

---

#### 6. The `count()` and `index()` Methods

**Question:** Which built-in methods do tuples support for data analysis?

**Expected Answer:** Tuples only support two methods: `count(x)` (returns number of occurrences) and `index(x)` (returns the first index of x). They lack methods like `sort()`, `append()`, or `extend()` due to immutability.

---

### Part B: Medium Level (14 Questions)

#### 7. Memory Efficiency: List vs Tuple

**Question:** Why do tuples consume less memory than lists?

**Expected Answer:** Lists are over-allocated to allow for O(1) appends. They store extra "empty" slots. Tuples are fixed-size; Python allocates exactly the amount of memory needed for the data plus a small overhead.

**Sample Code:**
```python
import sys
t = (1, 2, 3)
l = [1, 2, 3]
print(sys.getsizeof(t)) # Smaller
print(sys.getsizeof(l)) # Larger
```

---

#### 8. "Named Tuples" in Python

**Question:** When would you use `collections.namedtuple` instead of a standard tuple?

**Expected Answer:** Use `namedtuple` when you want to access elements by field name (e.g., `point.x`) instead of index (`point[0]`). It makes code more readable while maintaining the performance and immutability of a tuple.

---

#### 9. Tuple Concatenation Performance

**Question:** If you run `t += (4, 5)` in a loop, what is the performance impact?

**Expected Answer:** It is O(N^2). Because tuples are immutable, `t += (4, 5)` creates a **completely new tuple** every iteration, copying all existing elements. For building sequences, lists are preferred.

---

#### 10. The `__slots__` Optimization

**Question:** How do tuples relate to the concept of `__slots__` in Python classes?

**Expected Answer:** `__slots__` allows a class to use a fixed, tuple-like structure for attributes instead of a dynamic dictionary (`__dict__`). This significantly reduces memory footprint for millions of objects.

---

#### 11. Built-in `hash()` Mechanism

**Question:** Explain how the `hash()` of a tuple is calculated.

**Expected Answer:** The hash of a tuple is derived from the hashes of its individual elements. If an element's hash changes (impossible for truly immutable objects) or if an element is unhashable, the tuple's hash cannot be computed.

---

#### 12. Tuple Interning

**Question:** Does Python perform "interning" on tuples like it does for small integers or short strings?

**Expected Answer:** Not generally. While Python interns some constants at compile-time, tuples created at runtime are usually distinct objects even if they contain the same data.

---

#### 13. Returning Multiple Values

**Question:** When a function returns `return a, b`, what is actually happening?

**Expected Answer:** Python implicitly wraps the variables into a single **tuple**. This is known as "tuple packing."

---

#### 14. Nested Tuples and Flattening

**Question:** How does Python handle deeply nested tuples in terms of recursion limits?

**Expected Answer:** Tuples themselves don't have a specific nesting limit, but processing them via recursion is bound by `sys.setrecursionlimit()`. Iterative approaches are safer for extreme nesting.

---

#### 15. The `tuple()` Constructor Behavior

**Question:** What happens when you pass a generator to the `tuple()` constructor?

**Expected Answer:** The constructor exhausts the generator, iterates through all elements, and stores them in a new tuple. This is an O(N) operation.

---

#### 16. Structural Pattern Matching (Python 3.10+)

**Question:** How can tuples be used with the `match` statement?

**Expected Answer:** Tuples are excellent for pattern matching. You can match against specific lengths or capture elements using the `case (x, y, *rest):` syntax.

---

#### 17. Slicing Tuples

**Question:** Does slicing a tuple (`t[1:3]`) return a new tuple or a view?

**Expected Answer:** It returns a **new tuple**. Unlike some other languages (or NumPy arrays), standard Python tuple slices are shallow copies.

---

#### 18. Comparing Tuples

**Question:** How does tuple comparison (`(1, 2) < (1, 3)`) work?

**Expected Answer:** It uses **lexicographical order**. It compares the first elements; if they are equal, it moves to the second, and so on.

---

#### 19. Literal Evaluation

**Question:** Is `(1, 2, 3)` faster to create than `[1, 2, 3]`?

**Expected Answer:** Yes. Tuple literals are often constant-folded by the Python compiler into the function's constant pool, making creation nearly instantaneous at runtime compared to lists.

---

#### 20. Type Hinting with Tuples

**Question:** How do you type hint a tuple of fixed size vs. variable size in Python 3.12?

**Expected Answer:** 
* Fixed: `tuple[int, str, float]`
* Variable: `tuple[int, ...]`

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Part A: Easy Level (3 Questions)

#### 21. Swap Variables using Tuples

**Question:** Swap two variables `a` and `b` without using a temporary variable.

**Solution:**
```python
a, b = b, a
```

**Explanation:** This uses tuple packing (right side) and unpacking (left side).
**Complexity:** O(1).

---

#### 22. Find the Most Frequent Element

**Question:** Given a tuple, find the element that appears most frequently.

**Solution:**
```python
from collections import Counter
def most_frequent(t):
    return Counter(t).most_common(1)[0][0]
```

**Real App Usage:** Finding the most common category in a fixed dataset.

---

#### 23. Sort a List of Tuples

**Question:** Sort a list of tuples `(name, age)` by age.

**Solution:**
```python
data = [("Alice", 25), ("Bob", 20), ("Charlie", 30)]
data.sort(key=lambda x: x[1])
```

---

### Part B: Medium Level (7 Questions)

#### 24. Record Deduping with Tuples

**Question:** You have a list of dictionaries. Some are duplicates. Use tuples to remove duplicates (dictionaries are not hashable).

**Solution:**
```python
def dedupe_dicts(list_of_dicts):
    # Convert dict items to a sorted tuple to make it hashable
    seen = set()
    unique = []
    for d in list_of_dicts:
        t = tuple(sorted(d.items()))
        if t not in seen:
            seen.add(t)
            unique.append(d)
    return unique
```

---

#### 25. Implementing a Priority Queue with Tuples

**Question:** Use `heapq` and tuples to implement a simple priority queue where tasks are strings.

**Solution:**
```python
import heapq
pq = []
heapq.heappush(pq, (2, "Medium Task"))
heapq.heappush(pq, (1, "High Priority Task"))
# Result: (1, "High Priority Task") is popped first
```

---

#### 26. Flatten a Deeply Nested Tuple

**Question:** Write a recursive function to flatten a tuple of arbitrary depth.

**Solution:**
```python
def flatten(t):
    for item in t:
        if isinstance(item, tuple):
            yield from flatten(item)
        else:
            yield item

# Usage: list(flatten((1, (2, (3, 4)), 5)))
```

---

#### 27. Chunking a Tuple

**Question:** Break a tuple into chunks of size `n`.

**Solution:**
```python
def chunk_tuple(t, n):
    return [t[i:i + n] for i in range(0, len(t), n)]
```

---

#### 28. Validating Record Schema

**Question:** Given a list of tuples representing database rows, return only those that match a specific schema (e.g., 3 elements, first is `int`).

**Solution:**
```python
def validate_rows(rows):
    return [r for r in rows if len(r) == 3 and isinstance(r[0], int)]
```

---

#### 29. Transpose a Grid (Matrix)

**Question:** Use `zip` and tuples to transpose a matrix (list of tuples).

**Solution:**
```python
grid = [(1, 2, 3), (4, 5, 6)]
transposed = list(zip(*grid))
# Output: [(1, 4), (2, 5), (3, 6)]
```

---

#### 30. Find Unique Combinations

**Question:** Given a tuple of numbers, find all unique pairs that sum to `k`.

**Solution:**
```python
def find_pairs(t, k):
    seen = set()
    pairs = set()
    for num in t:
        target = k - num
        if target in seen:
            pairs.add(tuple(sorted((num, target))))
        seen.add(num)
    return pairs
```

---

