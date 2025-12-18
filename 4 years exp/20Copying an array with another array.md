# Python Array/List Copying — Interview Guide (30 Questions)

This guide covers the critical distinction between **References**, **Shallow Copies**, and **Deep Copies** in Python. It addresses standard Python Lists as well as NumPy arrays, where the rules for copying (View vs Copy) differ significantly and often catch candidates off guard.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What happens when you do `b = a` for a list?

* **Expected Answer**: You create a **Reference**, not a copy.
* **Clear Explanation**: `a` and `b` point to the exact same object in memory. Modifying `b` will modify `a`.
* **Sample Code**:
```python
a = [1, 2, 3]
b = a
b.append(4)
print(a) # [1, 2, 3, 4] (Modified!)

```



#### 2. How do you create a Shallow Copy of a list?

* **Expected Answer**: Three common ways:
1. Slicing: `b = a[:]`
2. Constructor: `b = list(a)`
3. Method: `b = a.copy()` (Python 3.3+)


* **Behavior**: It creates a new list container, but the elements inside are references to the same objects as the original list.

#### 3. What is the difference between Slicing a List vs Slicing a NumPy Array?

* **Expected Answer**:
* **List Slicing**: Returns a **Copy**. Modifying the slice does not affect the original list.
* **NumPy Slicing**: Returns a **View**. Modifying the slice **does** affect the original array.


* **Why**: NumPy optimizes for large data; copying gigabytes of data by default would be too slow. Python lists optimize for safety/simplicity.
* **Sample Code**:
```python
# List
l = [1, 2, 3]; s = l[:2]; s[0] = 99
print(l[0]) # 1 (Unchanged)

# NumPy
import numpy as np
a = np.array([1, 2, 3]); v = a[:2]; v[0] = 99
print(a[0]) # 99 (Changed!)

```



#### 4. What is a "Deep Copy"?

* **Expected Answer**: A copy where the container **and** all objects recursively nested inside it are duplicated.
* **Usage**: Required when you have mutable objects inside your list (e.g., a list of lists) and you want complete independence.
* **Module**: `import copy; b = copy.deepcopy(a)`.

#### 5. Why is `copy.deepcopy()` slower than `copy.copy()`?

* **Expected Answer**: Deep copy must traverse the entire object graph recursively. It checks for circular references and duplicates every single mutable object it finds. Shallow copy only duplicates the outer container.
* **Complexity**: Deep copy is O(N) where N is the total count of all nested objects. Shallow copy is O(M) where M is the length of the top-level list.

#### 6. How do you force a copy of a NumPy array?

* **Expected Answer**: Use the `.copy()` method.
* **Sample Code**:
```python
import numpy as np
a = np.array([1, 2, 3])
b = a.view() # Reference/View
c = a.copy() # Independent Copy

```



---

### Medium Level (Questions 7–20)

#### 7. Explain the Memory Layout of a NumPy View.

* **Question**: If `v = a[::2]`, `v` is a view. How does it represent data without copying?
* **Expected Answer**: `v` points to the **same memory buffer** as `a`, but it has different **metadata** (strides and shape).
* **Details**: If `a` has a stride of 4 bytes (int32), `v` will have a stride of 8 bytes (skipping every other integer). No new data is allocated.

#### 8. Copying a List of Lists (The Trap).

* **Question**: `a = [[1], [2]]; b = list(a)`. If I do `b[0].append(3)`, does `a` change?
* **Expected Answer**: **Yes**.
* **Reasoning**: `list(a)` is a **Shallow Copy**.
* It created a new outer list `b`.
* But `b[0]` points to the *same inner list object* as `a[0]`.
* Modifying the inner list affects both.


* **Visual**:
```text
a -> [ * ] -> [ 1 ]
       |
b -> [ * ] (Both pointers point to same [1] object)

```



#### 9. The `id()` function proof.

* **Question**: How can you prove two variables are copies and not references?
* **Expected Answer**: Check `id(a) == id(b)`.
* If True: They are references to the same object.
* If False: They are distinct objects (one is a copy of the other, or distinct creation).
* **Note**: For deep equality, you must also check internal IDs if valid.



#### 10. Performance: `list(a)` vs `a[:]`.

* **Question**: Which is faster for copying a list?
* **Expected Answer**: `a[:]` is marginally faster in some CPython versions because it avoids the function call overhead of `list()`, implementing the copy directly in C via slicing logic. However, `list(a)` is more readable for type conversion.

#### 11. Copying Immutable objects (Tuples).

* **Question**: `t1 = (1, 2); t2 = tuple(t1)`. Is `t2` a copy?
* **Expected Answer**: **No**, it is usually the same object.
* **Reasoning**: Since tuples are immutable, Python optimizes memory by returning the original object (`t1 is t2` is True). Making a copy of an immutable object is logically redundant.

#### 12. Copying Dictionaries.

* **Question**: How do you copy a `dict`?
* **Expected Answer**: `d.copy()` or `dict(d)`.
* **Constraint**: Both are shallow copies. If values are lists, they are shared.

#### 13. What is `np.ascontiguousarray()`?

* **Question**: If you have a non-contiguous view (e.g., `a[::2]`), how do you turn it into a contiguous block of memory?
* **Expected Answer**: `np.ascontiguousarray(view)`.
* **Side Effect**: This forces a **Copy** of the data into a new, compact buffer.

#### 14. The `__copy__` and `__deepcopy__` hooks.

* **Question**: How do you customize how your custom class is copied?
* **Expected Answer**: Implement `__copy__(self)` and `__deepcopy__(self, memo)`.
* **Why**: Useful if your class handles file handles or database connections that cannot be simply duplicated.

#### 15. Circular References in Deep Copy.

* **Question**: `a = []; a.append(a); b = copy.deepcopy(a)`. Does this crash?
* **Expected Answer**: No.
* **Reasoning**: `deepcopy` maintains a `memo` dictionary of objects already copied during the current pass. When it encounters the recursive reference to `a`, it uses the reference to the *new* copy of `a` instead of recursing infinitely.

#### 16. JSON Serialization as a "Poor Man's Deep Copy".

* **Question**: Can you use `json.loads(json.dumps(a))` to deep copy?
* **Expected Answer**: Yes, but with limitations.
* **Pros**: Often faster than `copy.deepcopy` for pure data (lists/dicts of primitives).
* **Cons**:
1. Only works for JSON-serializable types (no custom objects, datetimes, sets).
2. Converts tuples to lists.
3. Keys in dicts become strings (e.g., `{1: 'a'}` becomes `{'1': 'a'}`).



#### 17. Slicing with `step` -Copying logic.

* **Question**: `b = a[::-1]`. Does this copy?
* **Expected Answer**: Yes. It creates a new list with elements in reverse order.
* **Complexity**: O(N) time and memory.

#### 18. NumPy: Assigning into a slice.

* **Question**: `a[:] = b`. Is this a copy?
* **Expected Answer**: No, this is **In-Place Modification**.
* **Explanation**: It copies elements *from* `b` *into* the existing memory buffer of `a`. The identity `id(a)` remains unchanged.

#### 19. Copying between types.

* **Question**: `a = [1, 2]; b = tuple(a)`. Is `b` a copy?
* **Expected Answer**: It is a conversion. `b` is a new object (Tuple) containing the same elements. It is effectively a shallow copy into a new container type.

#### 20. `copy.deepcopy` on NumPy arrays.

* **Question**: Does `deepcopy` work on NumPy arrays?
* **Expected Answer**: Yes, it calls the array's state serialization (pickle support) to recreate the array completely independent of the original.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Basic List Clone

**Question**: Write a function that accepts a list and returns a clone. Verify that modifying the clone does not affect the original.

* **Solution**:
```python
def clone_and_verify(original: list):
    # Method 1: Slicing
    clone = original[:]

    # Verify independence
    clone.append("New")

    print(f"Original: {original}")
    print(f"Clone:    {clone}")

    return clone

clone_and_verify([1, 2, 3])

```


* **Explanation**: Standard shallow copy pattern.

#### 2. Manual Deep Copy (Recursive)

**Question**: Implement `deep_copy(lst)` for a nested list of integers (no circular refs) without using `copy` module.

* **Solution**:
```python
def manual_deep_copy(data):
    if isinstance(data, list):
        # Recursively copy elements
        return [manual_deep_copy(item) for item in data]
    else:
        # Primitive (int/str) - return as is (immutable)
        return data

# Usage
orig = [[1, 2], [3, 4]]
cp = manual_deep_copy(orig)
cp[0][0] = 99
print(orig) # [[1, 2], [3, 4]] (Unaffected)

```


* **Complexity**: Time O(N) where N is total elements.

#### 3. Fixing the Default Arg Bug

**Question**: This function is buggy because it shares the list. Fix it by making a copy.
`def add_item(item, l=[]): l.append(item); return l`

* **Solution**:
```python
def add_item(item, l=None):
    if l is None:
        l = [] # Create NEW list every call
    else:
        # Optional: If you don't want to modify the passed list input:
        l = l[:] 
        pass

    l.append(item)
    return l

```



---

### Medium Level (Questions 4–10)

#### 4. NumPy: Copy vs View Trap

**Question**: Given a 2D array, create a variable `sub` representing the top-left 2x2 corner. Ensure that modifying `sub` does **NOT** modify the original array.

* **Solution**:
```python
import numpy as np

def safe_subgrid():
    arr = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

    # Slicing returns a View
    # .copy() forces new memory allocation
    sub = arr[0:2, 0:2].copy()

    sub[0, 0] = 999

    print("Sub:\n", sub)
    print("Original (Should be 1 at top left):\n", arr)

safe_subgrid()

```



#### 5. Cloning a Dictionary of Lists

**Question**: You have `d = {'a': [1, 2], 'b': [3]}`. Create a copy such that appending to the lists in the copy doesn't affect the original. Do not use `deepcopy`.

* **Solution**:
```python
def copy_dict_of_lists(d: dict) -> dict:
    # Dictionary comprehension
    # k: v[:] creates a slice copy of the list value
    return {k: v[:] for k, v in d.items()}

orig = {'a': [1, 2]}
cp = copy_dict_of_lists(orig)
cp['a'].append(3)

print(orig['a']) # [1, 2]

```



#### 6. Copying 2D List (List Comprehension)

**Question**: Use list comprehension to deep copy a 2D matrix (List of Lists). `[[1,2], [3,4]]`.

* **Solution**:
```python
matrix = [[1, 2], [3, 4]]

# Outer list comp creates new rows
# row[:] creates copy of row content
copy_matrix = [row[:] for row in matrix]

# Or purely:
# copy_matrix = [[item for item in row] for row in matrix]

```



#### 7. Filter and Copy

**Question**: Given a list of numbers, create a new list containing only even numbers multiplied by 2. Ensure the original list is untouched.

* **Solution**:
```python
data = [1, 2, 3, 4]

# List comprehension creates a new list naturally
result = [x * 2 for x in data if x % 2 == 0]

print(result) # [4, 8]

```



#### 8. Circular Reference Handling (Advanced)

**Question**: Detect if a list contains a reference to itself.

* **Solution**:
```python
def has_cycle(lst):
    seen = set()

    def check(obj):
        if id(obj) in seen:
            return True
        if isinstance(obj, list):
            seen.add(id(obj))
            for item in obj:
                if check(item): return True
            seen.remove(id(obj)) # Backtrack
        return False

    return check(lst)

a = [1]
a.append(a)
print(has_cycle(a)) # True

```


* **Context**: Deep copy logic needs this detection to prevent infinite recursion.

#### 9. NumPy `as_strided` (Copy-Free Tricks)

**Question**: **Advanced**: Create a sliding window view of `[1, 2, 3, 4]` with window size 2 -> `[[1, 2], [2, 3], [3, 4]]` without copying data.

* **Solution**:
```python
import numpy as np
from numpy.lib.stride_tricks import as_strided

arr = np.array([1, 2, 3, 4], dtype='int32')

# Itemsize is 4 bytes
# To move row: step 4 bytes (1 element)
# To move col: step 4 bytes (1 element)
new_strides = (4, 4) 
new_shape = (3, 2)

view = as_strided(arr, shape=new_shape, strides=new_strides)
print(view)

```


* **Why**: Demonstrates mastery of NumPy memory layout (Views).

#### 10. Copying via Serialization (Pickle)

**Question**: Copy an arbitrary object `obj` by serializing to bytes and back.

* **Solution**:
```python
import pickle

def pickle_copy(obj):
    # Dumps to bytes, loads to new object
    return pickle.loads(pickle.dumps(obj))

data = {'a': {1, 2, 3}} # Sets are supported by pickle (unlike JSON)
cp = pickle_copy(data)
print(cp)

```
