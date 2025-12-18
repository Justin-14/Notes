# Python Dictionaries — Interview Guide (30 Questions)

This guide covers **Python Dictionaries**, the most important data structure in the language. It dives deep into hash table internals, the evolution of dictionary memory layout in recent Python versions, and advanced manipulation techniques required for senior backend roles.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What is the Time Complexity of looking up a key in a Dictionary?

* **Expected Answer**: Average Case **O(1)** (Constant). Worst Case **O(N)** (Linear).
* **Clear Explanation**: Dictionaries are Hash Tables. Python computes the hash of the key to calculate the memory index directly.
* **Reasoning for Worst Case**: If **Hash Collisions** occur frequently (e.g., a poorly written custom object always returns `hash(x) = 1`), Python must resolve conflicts, degrading performance to a linear scan.
* **Why Interviewers Ask This**: To verify you don't assume dictionaries are magic; they have failure modes.
* **Sample Code**:
```python
d = {"a": 1}
# Operations are O(1)
val = d["a"]
d["b"] = 2

```



#### 2. `d[key]` vs `d.get(key)`: When to use which?

* **Expected Answer**:
* `d[key]`: Raises **`KeyError`** if the key is missing. Use when the key *must* exist.
* `d.get(key, default)`: Returns `None` (or a specified default) if missing. Use when the key is optional.


* **Common Mistakes**: Writing `if key in d: return d[key]` is redundant. Just use `d.get(key)`.
* **Sample Code**:
```python
d = {"id": 101}
print(d.get("name", "Unknown")) # "Unknown"
# print(d["name"]) # Crashes code

```



#### 3. What are the restrictions on Dictionary Keys?

* **Expected Answer**: Keys must be **Hashable**.
* **Clear Explanation**: This generally means the object must be **Immutable**.
* Allowed: Strings, Integers, Tuples (if they contain only hashables), Frozensets.
* Forbidden: Lists, Dictionaries, Sets.


* **Reasoning**: If a key changes (mutates), its hash value would change. The dictionary would then lose track of where it stored the value associated with that key.
* **Sample Code**:
```python
valid = {(1, 2): "coordinates"}
# invalid = {[1, 2]: "error"} # TypeError: unhashable type: 'list'

```



#### 4. Are Python Dictionaries ordered?

* **Expected Answer**: Yes, as of **Python 3.7+**.
* **Clear Explanation**: Dictionaries preserve **Insertion Order**. If you insert "a", then "b", iteration will always yield "a" then "b".
* **History**: Before 3.6/3.7, dictionaries were unordered (random-looking).
* **Why Interviewers Ask This**: To see if your knowledge is current. Relying on order in Python 3.5 or older codebases is a bug.

#### 5. How do you merge two dictionaries?

* **Expected Answer**:
* **Python 3.9+**: Use the Merge Operator `|`. `new_d = d1 | d2`.
* **Older**: `d1.update(d2)` (modifies in-place) or `{**d1, **d2}` (creates new).


* **Behavior**: If keys overlap, the right-hand dictionary overwrites the left-hand one.
* **Sample Code**:
```python
d1 = {"a": 1}
d2 = {"a": 2, "b": 3}
merged = d1 | d2
print(merged) # {'a': 2, 'b': 3}

```



#### 6. Dictionary Comprehension Syntax.

* **Question**: How do you transform a list `[('a', 1), ('b', 2)]` into a dict using comprehension?
* **Expected Answer**: `{k: v for k, v in data}`.
* **Why Interviewers Ask This**: It is the most Pythonic way to construct dictionaries from iterables.
* **Sample Code**:
```python
data = [('a', 1), ('b', 2)]
my_dict = {k: v * 10 for k, v in data}
print(my_dict) # {'a': 10, 'b': 20}

```



---

### Medium Level (Questions 7–20)

#### 7. How does Python implement Hash Tables? (Chaining vs Open Addressing)

* **Question**: Does Python use Linked Lists (Chaining) or Open Addressing for collision resolution?
* **Expected Answer**: Python uses **Open Addressing**.
* **Medium-Depth Reasoning**:
* **Chaining**: Each bucket holds a list of items.
* **Open Addressing**: If a bucket is full, Python looks for another empty bucket within the array itself using a specific probing sequence (Quadratic Probing / Pseudo-random).


* **Why**: Open Addressing is more cache-efficient (CPU cache) for the sizes of dictionaries typically used in Python.
* **ASCII Diagram**:
```text
Index | Key   | Value
---------------------
0     | None  |
1     | "A"   | 10   <- Hash("B") also maps to 1?
2     | "B"   | 20   <- Collision! Probe finds Index 2 empty. Insert here.

```



#### 8. The "Compact Dict" Optimization (Python 3.6+).

* **Question**: Why do dictionaries use significantly less memory in modern Python than in Python 2.7?
* **Expected Answer**: Python split the storage into two arrays: **Indices** and **Entries**.
* **Mechanism**:
1. **Indices**: A sparse array of integers (very small memory) mapping hash to entry index.
2. **Entries**: A dense array storing `[hash, key, value]` in insertion order.


* **Benefit**: This reduced memory footprint by ~20-25% and enabled insertion-order preservation naturally.
* **Why Interviewers Ask This**: Senior-level internals knowledge.

#### 9. How do `keys()`, `values()`, and `items()` behave? (View Objects)

* **Question**: What does `d.keys()` return? Is it a list?
* **Expected Answer**: It returns a **Dictionary View**. It is dynamic, not a static list.
* **Implication**: If the dictionary changes *after* you call `.keys()`, the view reflects those changes immediately.
* **Complexity**: Creating the view is O(1). Converting it to a list is O(N).
* **Sample Code**:
```python
d = {"a": 1}
keys = d.keys()
d["b"] = 2
print(list(keys)) # ['a', 'b'] (Updates automatically!)

```



#### 10. `defaultdict` vs standard `dict`.

* **Question**: When would you use `collections.defaultdict`?
* **Expected Answer**: When you want to avoid checking `if key in d` before appending or modifying a value.
* **Mechanism**: It accepts a factory function (e.g., `list`, `int`). If a key is missing, it calls the factory to create a default value, inserts it, and returns it.
* **Sample Code**:
```python
from collections import defaultdict
dd = defaultdict(list)
dd["users"].append("Alice") # No KeyError, creates list automatically

```



#### 11. What is the `__missing__` method?

* **Question**: How can you customize a dictionary to handle missing keys without using `defaultdict`?
* **Expected Answer**: Subclass `dict` and implement `__missing__(self, key)`.
* **Behavior**: When `d[key]` fails, Python calls `d.__missing__(key)`.
* **Real Usage**: Implementing a cache where missing keys trigger a database lookup.
* **Sample Code**:
```python
class LazyDict(dict):
    def __missing__(self, key):
        print(f"Fetching {key}...")
        self[key] = 0 # Default logic
        return 0

```



#### 12. Modifying a dictionary during iteration.

* **Question**: What happens if you run `for k in d: del d[k]`?
* **Expected Answer**: `RuntimeError: dictionary changed size during iteration`.
* **Solution**: Iterate over a copy of the keys. `for k in list(d): del d[k]`.
* **Why Interviewers Ask This**: A very common runtime bug.

#### 13. `OrderedDict` vs Modern `dict`.

* **Question**: Now that standard dicts are ordered, is `OrderedDict` obsolete?
* **Expected Answer**: Mostly, but not entirely.
1. **Equality**: `OrderedDict` checks order during equality (`od1 == od2` requires same order). Standard dicts only check content (`d1 == d2` ignores order).
2. **Methods**: `OrderedDict` has specific methods like `move_to_end()` which standard dicts lack.


* **Recommendation**: Use standard `dict` for simple ordering. Use `OrderedDict` for LRU Caches or re-orderable mappings.

#### 14. Dictionary Resizing and Amortized Complexity.

* **Question**: What happens when a dictionary gets full?
* **Expected Answer**: Python automatically resizes the hash table.
1. It allocates a new, larger block of memory (usually 2x).
2. It re-inserts all existing items into the new table (Rehashing).


* **Load Factor**: Python keeps the load factor (entries / table size) below **2/3**.
* **Complexity**: Insertion is amortized O(1), but the specific insertion that triggers a resize is O(N).

#### 15. The `fromkeys()` method trap.

* **Question**: What is the danger of `dict.fromkeys(['a', 'b'], [])`?
* **Expected Answer**: All keys share the **same** list object in memory.
* **Result**: Modifying one key's list modifies them all.
* **Correct Way**: Use a dict comprehension `{k: [] for k in keys}`.
* **Sample Code**:
```python
d = dict.fromkeys(['a', 'b'], [])
d['a'].append(1)
print(d['b']) # [1] (Shared reference!)

```



#### 16. Using custom objects as keys.

* **Question**: If you use a custom class `User` as a key, what two methods must you implement?
* **Expected Answer**: `__hash__` and `__eq__`.
* **Rule**:
1. If `a == b`, then `hash(a)` MUST equal `hash(b)`.
2. `__hash__` should rely on immutable attributes only.


* **Why Interviewers Ask This**: Essential for caching objects.

#### 17. `ChainMap`: Logical Merging.

* **Question**: How can you treat multiple dictionaries as a single mapping without copying data?
* **Expected Answer**: Use `collections.ChainMap`.
* **Mechanism**: It keeps a list of references to the underlying dicts. Lookups check the first dict, then the second, etc. Writes usually affect only the first dict.
* **Scenario**: Managing variable scopes (Local, Global, Built-in).

#### 18. Dictionary Memory Overhead.

* **Question**: Is a dictionary memory-efficient for storing millions of objects?
* **Expected Answer**: No. Dictionaries have significant overhead (hash table structure, pointers).
* **Alternative**:
* **`__slots__`** in classes (saves RAM by removing `__dict__`).
* **Pandas DataFrames** (columnar storage).
* **Tuples/NamedTuples** (if keys are static).



#### 19. Can `True`, `1`, and `1.0` coexist as keys?

* **Question**: If `d = {True: 'a', 1: 'b', 1.0: 'c'}`, what is `d`?
* **Expected Answer**: `d` is `{True: 'c'}`.
* **Reasoning**: `True == 1 == 1.0`. They all have the same hash `1`. Python considers them duplicates. The *key* remains the first one seen (`True`), but the *value* is overwritten by the last one seen (`'c'`).

#### 20. Removing items safely (`pop` vs `del` vs `popitem`).

* **Expected Answer**:
* `del d[k]`: Statement. Crashes if key missing.
* `d.pop(k, default)`: Method. Returns value. Safe if default provided.
* `d.popitem()`: Removes and returns the **last inserted** `(key, value)` pair (LIFO behavior in 3.7+).



---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Character Frequency Count

**Question**: Given a string, return a dictionary with the count of every character.

* **Solution**:
```python
from collections import Counter

def count_chars(s: str) -> dict:
    # Professional way:
    return dict(Counter(s))

def count_chars_manual(s: str) -> dict:
    # Algorithms way:
    counts = {}
    for char in s:
        counts[char] = counts.get(char, 0) + 1
    return counts

# Usage
print(count_chars("banana")) # {'b': 1, 'a': 3, 'n': 2}

```


* **Explanation**: `Counter` is optimized C-code. `get(char, 0)` is the standard pattern to handle initialization.
* **Complexity**: O(N).

#### 2. Invert a Dictionary

**Question**: Given `{'a': 1, 'b': 2, 'c': 1}`, return `{1: ['a', 'c'], 2: ['b']}`. Note that values are not unique, so the inverted values must be lists.

* **Solution**:
```python
from collections import defaultdict

def invert_dict(d: dict) -> dict:
    inverted = defaultdict(list)
    for key, value in d.items():
        inverted[value].append(key)
    return dict(inverted)

# Usage
d = {'a': 1, 'b': 2, 'c': 1}
print(invert_dict(d))

```


* **Real Usage**: Indexing database columns (looking up ID by Value).

#### 3. Dictionary Intersection

**Question**: Given two dictionaries, find keys present in both.

* **Solution**:
```python
def dict_intersection_keys(d1: dict, d2: dict) -> set:
    # Dict View Objects support set operations!
    return d1.keys() & d2.keys()

def dict_intersection_items(d1: dict, d2: dict) -> dict:
    # Keys and Values must match
    return {k: d1[k] for k in d1.keys() & d2.keys() if d1[k] == d2[k]}

# Usage
a = {"x": 1, "y": 2}
b = {"y": 2, "z": 3}
print(dict_intersection_keys(a, b)) # {'y'}

```


* **Why Interviewers Ask This**: Most candidates loop manually. Knowing that `.keys()` behaves like a Set is a "pro" move.

---

### Medium Level (Questions 4–10)

#### 4. Group Data by Key

**Question**: Input is a list of dictionaries: `[{'dept': 'IT', 'name': 'Alice'}, {'dept': 'HR', 'name': 'Bob'}, {'dept': 'IT', 'name': 'Charlie'}]`. Output: `{'IT': ['Alice', 'Charlie'], 'HR': ['Bob']}`.

* **Solution**:
```python
from collections import defaultdict

def group_by_dept(employees: list[dict]) -> dict:
    grouped = defaultdict(list)
    for emp in employees:
        dept = emp['dept']
        name = emp['name']
        grouped[dept].append(name)
    return dict(grouped)

```


* **Developer Scenario**: Aggregating SQL query results or API responses.

#### 5. Safe Nested Access (Deep Get)

**Question**: Implement `get_deep(d, "a.b.c")` that retrieves nested values using a dot-notation string. Return `None` if path fails.

* **Solution**:
```python
def get_deep(data: dict, path: str):
    keys = path.split('.')
    current = data
    for key in keys:
        if isinstance(current, dict) and key in current:
            current = current[key]
        else:
            return None
    return current

# Usage
config = {"db": {"host": {"primary": "127.0.0.1"}}}
print(get_deep(config, "db.host.primary")) # "127.0.0.1"
print(get_deep(config, "db.port")) # None

```


* **Complexity**: O(K) where K is depth of keys.

#### 6. Sort Dictionary by Value

**Question**: Sort a dictionary based on its values in descending order.

* **Solution**:
```python
def sort_by_value(d: dict) -> dict:
    # sorted() returns a list of tuples
    # We pass it to dict() constructor to rebuild (Py 3.7+ preserves order)
    sorted_items = sorted(d.items(), key=lambda item: item[1], reverse=True)
    return dict(sorted_items)

# Usage
scores = {"Alice": 50, "Bob": 80, "Charlie": 60}
print(sort_by_value(scores))
# {'Bob': 80, 'Charlie': 60, 'Alice': 50}

```


* **Why Interviewers Ask This**: Sorting is common, but you cannot "sort a dict" in-place (no `.sort()` method). You must rebuild it.

#### 7. First Non-Repeating Character

**Question**: Find the first character in a string that appears only once. O(N) Time.

* **Solution**:
```python
def first_unique_char(s: str) -> str | None:
    counts = {}
    # Pass 1: Count frequencies
    # Dict preserves insertion order (Crucial here!)
    for char in s:
        counts[char] = counts.get(char, 0) + 1

    # Pass 2: Find first count == 1
    for char, count in counts.items():
        if count == 1:
            return char

    return None

# Usage
print(first_unique_char("leetcode")) # "l"
print(first_unique_char("loveleetcode")) # "v"

```


* **Optimization**: Because Python dicts are ordered, iterating `counts.items()` guarantees we encounter characters in their original string order. We don't need a second index-based loop over the string.

#### 8. LRU Cache Implementation (Using OrderedDict)

**Question**: Implement a Least Recently Used (LRU) Cache with `get` and `put` in O(1). Max capacity N.

* **Solution**:
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        # Mark as used: move to end
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            # Remove first item (Least Recently Used)
            self.cache.popitem(last=False)

# Usage
lru = LRUCache(2)
lru.put(1, 1)
lru.put(2, 2)
lru.get(1)       # Moves 1 to end. Order: [2, 1]
lru.put(3, 3)    # 2 is LRU, evicted. Order: [1, 3]

```


* **Explanation**: `OrderedDict` is perfect here. `move_to_end` updates recency. `popitem(last=False)` acts like a queue `popleft`.

#### 9. Merge Dictionaries with Summation

**Question**: You have two dicts `d1` and `d2`. Merge them. If a key exists in both, sum their values.

* **Solution**:
```python
from collections import Counter

def merge_sum(d1: dict, d2: dict) -> dict:
    # Counter supports direct addition!
    c1 = Counter(d1)
    c2 = Counter(d2)
    total = c1 + c2
    return dict(total)

# Manual approach (if Counter forbidden):
# result = d1.copy()
# for k, v in d2.items():
#     result[k] = result.get(k, 0) + v

# Usage
a = {'apple': 1, 'orange': 2}
b = {'apple': 3, 'banana': 5}
print(merge_sum(a, b)) # {'apple': 4, 'orange': 2, 'banana': 5}

```



#### 10. Check if Dictionary is a Subset

**Question**: Write `contains_json(json_full, json_subset)`. Return True if all keys/values in `subset` exist in `full`. Handle nested dicts recursively.

* **Solution**:
```python
def contains_json(full: dict, sub: dict) -> bool:
    for key, sub_val in sub.items():
        if key not in full:
            return False

        full_val = full[key]

        if isinstance(sub_val, dict) and isinstance(full_val, dict):
            # Recurse for nested objects
            if not contains_json(full_val, sub_val):
                return False
        elif full_val != sub_val:
            # Compare primitives
            return False

    return True

# Usage
full = {"id": 1, "meta": {"valid": True, "count": 10}}
sub = {"meta": {"valid": True}}
print(contains_json(full, sub)) # True

```


* **Real Usage**: Testing API responses where the response contains extra metadata (timestamps) you want to ignore, validating only the core payload.
