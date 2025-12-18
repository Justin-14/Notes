# Python Loops: While & For — Interview Guide (30 Questions)

This guide covers the core iteration mechanisms in Python: the `while` loop (indefinite iteration) and the `for` loop (definite iteration). It explores internal protocols (`__iter__`), modern features like `zip(strict=True)`, and avoiding common mutability traps.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. Fundamental Difference: `for` vs `while` in Python.

* **Expected Answer**:
* **`for`**: Designed for **Definite Iteration**. It iterates over a collection (iterable) of known items (list, range, string).
* **`while`**: Designed for **Indefinite Iteration**. It repeats as long as a boolean condition remains True.


* **Clear Explanation**: Unlike C-style `for` loops (which act like `while` loops with counters), Python's `for` is strictly a "for-each" loop.
* **Sample Code**:
```python
# Definite: Run 3 times
for i in range(3): pass

# Indefinite: Run until user says stop
while input() != "stop": pass

```



#### 2. How does `range()` work in Python 3? Is it a list?

* **Expected Answer**: `range()` returns an immutable sequence type (a **range object**), NOT a list.
* **Reasoning**: It generates numbers on demand (lazy evaluation).
* **Memory Implication**: `range(1000000)` takes the same small amount of memory as `range(1)`. It only stores start, stop, and step.
* **Python 2 Note**: Python 2's `range()` returned a list (memory heavy); `xrange()` was the lazy version. In Python 3, `range` is `xrange`.
* **Sample Code**:
```python
r = range(10)
print(type(r)) # <class 'range'>

```



#### 3. How do you iterate with both Index and Value?

* **Expected Answer**: Use `enumerate(iterable, start=0)`.
* **Why Interviewers Ask This**: To verify you don't write unpythonic code like `for i in range(len(items)): val = items[i]`.
* **Sample Code**:
```python
items = ["a", "b"]
for i, val in enumerate(items):
    print(f"Index: {i}, Val: {val}")

```



#### 4. Iterating backwards: What is the Pythonic way?

* **Expected Answer**: Use `reversed(iterable)` or slice `[::-1]`.
* **Comparison**:
* `reversed()`: Memory efficient (returns iterator).
* `[::-1]`: Creates a copy (memory inefficient for large lists).
* `range(N, -1, -1)`: Works but less readable for existing lists.


* **Sample Code**:
```python
for i in reversed(range(5)):
    print(i) # 4, 3, 2, 1, 0

```



#### 5. How do you iterate over a Dictionary?

* **Expected Answer**: Iterating a dict directly yields its **Keys**.
* **Methods**:
* `.keys()`: Keys only (default).
* `.values()`: Values only.
* `.items()`: `(Key, Value)` tuples.


* **Sample Code**:
```python
d = {'a': 1, 'b': 2}
for k, v in d.items():
    print(k, v)

```



#### 6. What defines an "Infinite Loop"?

* **Expected Answer**: A loop where the termination condition is never met.
* **Syntax**: `while True:` is the standard idiom for intentionally infinite loops (e.g., server listeners), usually paired with a `break`.
* **Common Mistake**: Forgetting to update the loop variable in a `while` block.
```python
i = 0
while i < 5:
    print(i)
    # i += 1 (Missing this causes infinite loop)

```



---

### Medium Level (Questions 7–20)

#### 7. Modifying a List while Iterating.

* **Question**: What happens if you remove items from a list you are currently iterating over?
* **Expected Answer**: It causes **unexpected behavior** (skips elements).
* **Reasoning**: The iterator maintains an internal index. If you remove item at index 0, item at index 1 shifts to index 0. The iterator then advances to index 1, effectively skipping the item that shifted.
* **Solution**: Iterate over a copy (`list[:]`) or iterate backwards.
* **Sample Code**:
```python
nums = [1, 2, 3, 4]
for n in nums:
    if n == 2:
        nums.remove(n)
print(nums) # [1, 3, 4] (Correctly removed? Often skips 3 depending on logic)

```



#### 8. The `for...else` Construct.

* **Question**: When does the `else` block execute in a `for` loop?
* **Expected Answer**: It runs **only if the loop completes without hitting a `break**`.
* **Use Case**: Search operations. "Run through the haystack. If I didn't break (didn't find the needle), run the `else` (report not found)."
* **Why Interviewers Ask This**: It is unique to Python and often misunderstood.

#### 9. Loop Variable Scope Leak.

* **Question**: Is the variable `i` in `for i in range(10): pass` available after the loop?
* **Expected Answer**: **Yes**.
* **Explanation**: Python loops do not create a new scope. The variable `i` retains the last value assigned to it (likely 9).
* **Comparison**: This differs from C++/Java loop-scoped variables.

#### 10. `zip()` and the `strict` argument (Python 3.10+).

* **Question**: What happens if you `zip` two lists of different lengths? How do you enforce equal length?
* **Expected Answer**:
* Default `zip()`: Stops at the **shortest** list. The rest of the longer list is ignored silently.
* `zip(..., strict=True)`: Raises `ValueError` if lengths differ.


* **Why Interviewers Ask This**: To see if you are aware of modern Python safety features.

#### 11. Emulating `do-while` loops.

* **Question**: Python has no `do-while`. How do you write code that executes the body *at least once* before checking the condition?
* **Expected Answer**: Use `while True` with a `break` at the end.
* **Sample Code**:
```python
while True:
    data = input("Enter valid data: ")
    if is_valid(data):
        break

```



#### 12. Internal Iteration Protocol.

* **Question**: How does a `for` loop actually work under the hood?
* **Expected Answer**:
1. Calls `iter(obj)` to get an **Iterator**.
2. Repeatedly calls `next(iterator)`.
3. Catches `StopIteration` exception to finish the loop.


* **Diagram**:
```text
Object -> iter() -> Iterator
                      |
next() <--------------+
  |
Value or StopIteration -> End Loop

```



#### 13. Walrus Operator in `while` loops.

* **Question**: How can `:=` simplify a `while` loop reading a file/socket?
* **Expected Answer**: It combines the read and the check in one line.
* **Comparison**:
* Old: `data = f.read(); while data: ... data = f.read()`
* New: `while (data := f.read()): ...`


* **Benefit**: Eliminates code duplication (DRY principle).

#### 14. Nested Loops and Big-O.

* **Question**: What is the Time Complexity of a loop `for i in range(N)` containing `for j in range(i, N)`?
* **Expected Answer**: O(N^2).
* **Reasoning**: The inner loop runs N, N-1, N-2... times. Sum of 1..N is N(N+1)/2, which simplifies to quadratic time.

#### 15. The `_` (Underscore) Variable.

* **Question**: What does `for _ in range(10):` signify?
* **Expected Answer**: It indicates that the **loop variable is ignored/unused**. We just want to repeat an action 10 times.
* **Why**: Code readability. Linters (like Pylint) won't complain about an unused variable named `_`.

#### 16. Breaking out of Multiple Nested Loops.

* **Question**: A `break` only exits the inner loop. How do you exit 3 levels of nested loops instantly?
* **Expected Answer**:
1. **Return**: Wrap loops in a function and `return`. (Best).
2. **Exception**: Raise a custom exception.
3. **Flag**: Check a `stop` flag in every loop layer.



#### 17. `iter(callable, sentinel)`.

* **Question**: There is a two-argument form of `iter()`. What is it for?
* **Expected Answer**: `iter(func, sentinel)` calls `func()` repeatedly until it returns `sentinel`.
* **Use Case**: Reading binary chunks until EOF.
* **Sample Code**:
```python
# Read 64 bytes at a time until empty bytes b''
for chunk in iter(lambda: f.read(64), b''):
    process(chunk)

```



#### 18. List Comprehension vs For Loop speed.

* **Question**: Which is faster: `[i*2 for i in nums]` or a manual `for` loop with `append`?
* **Expected Answer**: **List Comprehension** is generally faster.
* **Reasoning**: The comprehension iteration is performed in C-speed optimization within the interpreter. The `for` loop involves Python-level method lookups (`append`) in every iteration.

#### 19. Can you modify the iterator variable?

* **Question**: If you do `for i in range(3): i += 10`, does it affect the next iteration?
* **Expected Answer**: **No**.
* **Reasoning**: At the start of the next iteration, `i` is overwritten by the next value from the iterator. The modification is lost/overwritten immediately.

#### 20. `itertools`: Why use it?

* **Question**: When should you switch from standard `for` loops to `itertools`?
* **Expected Answer**: For complex iteration patterns (permutations, combinations, infinite counters, chaining iterables) or memory efficiency.
* **Example**: `itertools.chain(list1, list2)` iterates both without creating a generic concatenated list O(N+M) memory.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Factorial Calculation (While Loop)

**Question**: Calculate N! using a `while` loop.

* **Solution**:
```python
def factorial(n: int) -> int:
    if n < 0: raise ValueError("Negative input")
    result = 1
    while n > 0:
        result *= n
        n -= 1
    return result

print(factorial(5)) # 120

```


* **Complexity**: Time O(N).

#### 2. Sum of List (Manual For)

**Question**: Write a loop to sum a list of numbers without using `sum()`.

* **Solution**:
```python
def manual_sum(nums: list[int]) -> int:
    total = 0
    for n in nums:
        total += n
    return total

print(manual_sum([1, 2, 3])) # 6

```


* **Note**: Simple accumulator pattern.

#### 3. Reverse String (Loop)

**Question**: Reverse a string using a `for` loop (not slicing).

* **Solution**:
```python
def reverse_str(s: str) -> str:
    reversed_s = ""
    for char in s:
        reversed_s = char + reversed_s # Prepend
    return reversed_s

print(reverse_str("hello")) # "olleh"

```


* **Note**: String concatenation in a loop is O(N^2) due to immutability. Using a list and `.join()` is O(N).

---

### Medium Level (Questions 4–10)

#### 4. Flatten Nested List (Recursive Loop)

**Question**: Flatten `[1, [2, [3, 4], 5], 6]` into `[1, 2, 3, 4, 5, 6]`. Handle arbitrary depth.

* **Solution**:
```python
def flatten(nested_list):
    flat = []
    for item in nested_list:
        if isinstance(item, list):
            # Recursively extend results
            flat.extend(flatten(item))
        else:
            flat.append(item)
    return flat

data = [1, [2, [3, 4], 5], 6]
print(flatten(data))

```


* **Complexity**: O(N) where N is total integers.

#### 5. Detect Cycle in Linked List (While Loop)

**Question**: Implement Floyd’s Cycle-Finding Algorithm (Tortoise and Hare) using a `while` loop. (Assume a `Node` class exists).

* **Solution**:
```python
class Node:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def has_cycle(head: Node) -> bool:
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next          # Move 1 step
        fast = fast.next.next     # Move 2 steps

        if slow == fast:
            return True # Cycle detected

    return False

```


* **Why Interviewers Ask This**: Classic use of `while` with pointer arithmetic logic.

#### 6. Chunking a List

**Question**: Split a list into chunks of size N using `range` with step. `[1,2,3,4,5]`, size=2 -> `[[1,2], [3,4], [5]]`.

* **Solution**:
```python
def chunk_list(data, size):
    # Range with step creates indices: 0, 2, 4...
    return [data[i : i + size] for i in range(0, len(data), size)]

# Alternative explicit loop version:
def chunk_loop(data, size):
    chunks = []
    for i in range(0, len(data), size):
        chunks.append(data[i : i + size])
    return chunks

print(chunk_list([1, 2, 3, 4, 5], 2))

```



#### 7. Merge Two Sorted Lists (Two Pointer While)

**Question**: Merge two sorted lists `L1` and `L2` into one sorted list. Do not use `.sort()`.

* **Solution**:
```python
def merge_lists(l1, l2):
    result = []
    i, j = 0, 0

    # Compare and append smaller
    while i < len(l1) and j < len(l2):
        if l1[i] < l2[j]:
            result.append(l1[i])
            i += 1
        else:
            result.append(l2[j])
            j += 1

    # Append remaining
    result.extend(l1[i:])
    result.extend(l2[j:])
    return result

print(merge_lists([1, 4, 5], [2, 3, 6])) # [1, 2, 3, 4, 5, 6]

```


* **Complexity**: O(N + M).

#### 8. Matrix Transpose

**Question**: Convert rows to columns in a matrix `[[1, 2], [3, 4]]` -> `[[1, 3], [2, 4]]`. Use nested loops.

* **Solution**:
```python
def transpose(matrix):
    if not matrix: return []
    rows = len(matrix)
    cols = len(matrix[0])

    # Pre-allocate result with zeros
    result = [[0] * rows for _ in range(cols)]

    for r in range(rows):
        for c in range(cols):
            result[c][r] = matrix[r][c]

    return result

# Pythonic one-liner for bonus points:
# return list(zip(*matrix))

print(transpose([[1, 2], [3, 4]]))

```



#### 9. Finding Common Items (3 Lists)

**Question**: Given 3 lists, find common elements using loops (assume no Sets allowed for this exercise).

* **Solution**:
```python
def common_three(a, b, c):
    # Optimization: Sort first to allow binary search or pointer logic
    # Brute force with loops (O(N*M*K) - slow but demonstrates logic)
    res = []
    for x in a:
        if x in b:
            if x in c:
                if x not in res: # Avoid duplicates
                    res.append(x)
    return res

```


* **Discussion**: Interviewer will ask to optimize. Best approach is converting to Sets (Intersection), but looping approach tests basic flow control.

#### 10. Custom Range Iterator

**Question**: Implement a class `MyRange(start, end)` that works in a `for` loop by implementing `__iter__` and `__next__`.

* **Solution**:
```python
class MyRange:
    def __init__(self, start, end):
        self.current = start
        self.end = end

    def __iter__(self):
        # Returns self as the iterator object
        return self

    def __next__(self):
        if self.current >= self.end:
            raise StopIteration

        val = self.current
        self.current += 1
        return val

for i in MyRange(0, 3):
    print(i) # 0, 1, 2

```


* **Why Interviewers Ask This**: This proves you understand the "magic" that makes `for` loops work with objects.
