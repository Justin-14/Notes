# Python Operators — Interview Guide (30 Questions)

This guide covers the entire spectrum of Python Operators (Arithmetic, Logical, Comparison, Assignment, Identity, and Membership), excluding bitwise operators. It focuses on operator overloading, memory implications of in-place operations, and modern features like the Walrus operator.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. `is` vs `==`: The Fundamental Difference

* **Question**: Explain precisely the difference between `==` and `is`. When would `a == b` be True but `a is b` be False?
* **Expected Answer**:
* `==` checks for **value equality** (calls `__eq__`).
* `is` checks for **reference equality** (memory address identity, effectively `id(a) == id(b)`).


* **Scenario**: Two distinct lists with the same contents.
```python
l1 = [1, 2, 3]
l2 = [1, 2, 3]
print(l1 == l2) # True (Content matches)
print(l1 is l2) # False (Different memory objects)

```


* **Why Interviewers Ask This**: To determine if you understand the object memory model versus abstract data values.

#### 2. How does Chained Comparison work? (`a < b < c`)

* **Question**: In C or Java, `a < b < c` usually evaluates `(a < b)` first (0 or 1), then compares that result to `c`. How does Python handle it?
* **Expected Answer**: Python treats it as `(a < b) and (b < c)`, with the crucial optimization that `b` is evaluated **only once**.
* **Clear Explanation**: This prevents side effects if `b` is a function call.
* **Sample Code**:
```python
x = 5
if 1 < x < 10:
    print("x is between 1 and 10")

```



#### 3. True Division vs Floor Division (`/` vs `//`)

* **Question**: What is the return type of `10 / 2` versus `10 // 2` in Python 3?
* **Expected Answer**:
* `/` (True Division) *always* returns a `float` (`5.0`), even if the result is a whole number.
* `//` (Floor Division) returns an `int` (`5`) if operands are ints, but floors the result.


* **Why Interviewers Ask This**: Python 2 legacy confusion (`/` used to be integer division). In Python 3, this behavior is strict to prevent precision bugs.

#### 4. The `and` / `or` Short-Circuiting

* **Question**: What does the expression `a or b` actually return? Is it always a boolean?
* **Expected Answer**: It does **not** always return a boolean. It returns the **first truthy operand** or the last operand.
* `x or y`: If `x` is truthy, return `x`. Else return `y`.
* `x and y`: If `x` is falsy, return `x`. Else return `y`.


* **Sample Code**:
```python
print(0 or "default")  # "default" (0 is falsy)
print(5 and 10)        # 10 (5 is truthy, so return comparison)

```



#### 5. Modulo Operator with Negative Numbers

* **Question**: What is the result of `-5 % 2` in Python? Comparisons to C/Java?
* **Expected Answer**: `1`.
* **Reasoning**: Python's modulo follows the rule: `result` has the same sign as the **divisor** (the denominator).
* `-5 // 2` is `-3` (floored towards negative infinity).
* Math check: `-5 = 2 * (-3) + 1`.
* (In C/Java, `-5 % 2` is often `-1` because they truncate towards zero).


* **Why Interviewers Ask This**: Algorithms like "Cyclic Arrays" break if you don't understand this.

#### 6. Exponentiation Operator (`**`)

* **Question**: What is the difference between `pow(x, y)` and `x ** y`?
* **Expected Answer**: Functionally identical for two arguments. However, `pow()` supports a third argument for modular exponentiation: `pow(base, exp, mod)`.
* **Performance**: `pow(a, b, c)` is significantly faster than `(a ** b) % c` for large numbers because it computes modulo at every step (preventing massive number creation).

---

### Medium Level (Questions 7–20)

#### 7. The Walrus Operator (`:=`)

* **Question**: Introduced in Python 3.8, what does `:=` do and provide a valid use case?
* **Expected Answer**: It is the **Assignment Expression** operator. It assigns a value to a variable *inside* an expression.
* **Use Case**: Reducing code duplication in `while` loops or `if` conditions.
* **Sample Code**:
```python
# Without Walrus
data = f.read(1024)
while data:
    process(data)
    data = f.read(1024)

# With Walrus
while (data := f.read(1024)):
    process(data)

```



#### 8. `+=` (In-Place Add) on Mutable vs Immutable Types

* **Question**: Explain the memory difference between `x += y` and `x = x + y` for a List vs a Tuple.
* **Expected Answer**:
* **List (Mutable)**: `+=` calls `__iadd__`. It modifies the list **in-place** (extends it). The memory ID of `x` remains the same.
* **Tuple (Immutable)**: Tuples don't have `__iadd__`. Python falls back to `x = x + y`. It creates a **NEW** tuple object. The memory ID of `x` changes.


* **Why Interviewers Ask This**: Performance optimization and reference bugs.

#### 9. The "NotImplemented" Return Value

* **Question**: When implementing operator overloading (e.g., `__add__`), what should you return if your class doesn't know how to handle the other operand type? Should you raise `TypeError`?
* **Expected Answer**: No, return the singleton `NotImplemented`.
* **Reasoning**: This signals Python to try the *reflected* operation on the other operand (e.g., `other.__radd__(self)`). If both return `NotImplemented`, *then* Python raises `TypeError`.
* **Sample Code**:
```python
def __add__(self, other):
    if not isinstance(other, MyClass):
        return NotImplemented
    return MyClass(self.val + other.val)

```



#### 10. Operator Precedence: `not` vs `==`

* **Question**: How is `not x == y` evaluated?
* **Expected Answer**: It is evaluated as `not (x == y)`.
* **Reasoning**: Comparison operators (`==`, `>`, etc.) have higher precedence than logical `not`.
* **Why Interviewers Ask This**: Readability. It is often considered unpythonic. Prefer `x != y`.

#### 11. Matrix Multiplication Operator (`@`)

* **Question**: What is the `@` operator used for in Python?
* **Expected Answer**: It is reserved for **Matrix Multiplication**. It calls `__matmul__`.
* **Context**: It was introduced specifically for the Data Science community (NumPy, TensorFlow) to avoid the confusing syntax of `.dot()` chains.
* `A @ B` implies matrix multiplication.
* `A * B` usually implies element-wise multiplication in libraries like NumPy.



#### 12. Short-Circuiting Side Effects

* **Question**: Given `def f(): print("F runs"); return False`. What is printed by `True or f()`?
* **Expected Answer**: **Nothing** is printed.
* **Reasoning**: `True or ...` short-circuits. Since the first operand is True, the result is known (True). Python completely skips executing the second operand `f()`.
* **Why Interviewers Ask This**: To verify you understand control flow within boolean expressions.

#### 13. The `__radd__` (Reflected Add) Hook

* **Question**: If you have `class X` and `class Y`, and you run `y + x` where `y` does not define `__add__` (or returns NotImplemented), what happens?
* **Expected Answer**: Python attempts to call `x.__radd__(y)`.
* **Explanation**: "Reflected" (or Right-side) operators allow the object on the *right* to handle the logic if the object on the left cannot.
* **ASCII Diagram**:
```text
Expression: Y + X
   1. Try Y.__add__(X)  -> Returns NotImplemented
   2. Try X.__radd__(Y) -> Returns Result
   3. If both fail        -> TypeError

```



#### 14. Membership Operator `in` Complexity

* **Question**: What is the time complexity of `x in y`?
* **Expected Answer**: It depends entirely on the type of `y`.
* **List/Tuple**: O(N) (Linear scan).
* **Set/Dict**: O(1) (Hash lookup).
* **String**: O(N) (Substring search).


* **Why Interviewers Ask This**: Performance basics. Using `in` on a list inside a loop creates accidental quadratic O(N^2) complexity.

#### 15. Floor Division with Negative Floats

* **Question**: What is `-5.0 // 2`?
* **Expected Answer**: `-3.0`.
* **Nuance**: Even though it's "Floor" division (integer logic), if one operand is a float, the result is a float (`-3.0`, not `-3`). The rounding logic (toward negative infinity) stays the same.

#### 16. Unpacking Operators (`*` and `**`)

* **Question**: Apart from multiplication, what does `*` do?
* **Expected Answer**: It unpacks iterables.
* **Function Calls**: `func(*args)` unpacks a list into positional arguments.
* **List Literals**: `[1, *[2, 3]]` -> `[1, 2, 3]`.
* `**` does the same for Dictionaries (keyword arguments).



#### 17. The `__index__` method and slicing

* **Question**: You want to use your custom object as an index, e.g., `mylist[obj]`. Which operator method must `obj` implement?
* **Expected Answer**: `__index__`.
* **Explanation**: This method allows an object to be safely converted to an integer specifically for slicing or indexing purposes. (Older Python used `__int__`, but `__index__` is now required for this context).

#### 18. Comparison Chaining Internals

* **Question**: Can you chain `a < b > c`? What does it mean?
* **Expected Answer**: Yes, it is valid syntax.
* **Meaning**: `(a < b) and (b > c)`.
* **Example**: `1 < 5 > 2`.
* `1 < 5` is True.
* `5 > 2` is True.
* Result: True.


* **Note**: While syntactically valid, it is often confusing to read and usually avoided in style guides compared to monotonic chains like `a < b < c`.

#### 19. Operator Precedence: `and` vs `or`

* **Question**: How is `True or False and False` evaluated?
* **Expected Answer**: True.
* **Reasoning**: `and` has higher precedence than `or`.
1. `False and False` -> `False`.
2. `True or False` -> `True`.


* **Mnemonic**: In Algebra, Multiplication (`and`) comes before Addition (`or`).

#### 20. Identity Operator on Small Integers

* **Question**: Why is `256 is 256` True, but `300 is 300` sometimes False (in generic implementations)?
* **Expected Answer**: **Integer Interning**.
* **Explanation**: Python pre-allocates objects for integers in the range `-5` to `256`. Operators yielding results in this range return references to existing objects. Larger numbers *usually* create new objects (unless optimized by the compiler in the same scope).

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Custom Divisibility Check (Without `%`)

**Question**: Write a function `is_divisible(n, k)` that returns True if `n` is divisible by `k`, **without** using the modulo `%` operator.

* **Solution**:
```python
def is_divisible(n: int, k: int) -> bool:
    if k == 0: return False
    # Integer division * k should equal original number
    return (n // k) * k == n

# Usage
print(is_divisible(10, 2)) # True
print(is_divisible(10, 3)) # False (3 * 3 = 9 != 10)

```


* **Explanation**: `//` performs floor division. If `n` divides `k` perfectly, reconstructing `n` via multiplication works. If there was a remainder, the reconstruction would be smaller than `n`.
* **Complexity**: O(1).

#### 2. Safe Calculator (Operator Mapping)

**Question**: Create a function `calc(a, b, op)` where `op` is a string "+", "-", "*", "/". Use the `operator` module instead of `if/else` chains.

* **Solution**:
```python
import operator

def calc(a: int, b: int, op: str):
    ops = {
        "+": operator.add,
        "-": operator.sub,
        "*": operator.mul,
        "/": operator.truediv
    }
    if op not in ops:
        return None
    return ops[op](a, b)

# Usage
print(calc(10, 2, "/")) # 5.0

```


* **Why Interviewers Ask This**: Shows knowledge of the `operator` standard library, which maps symbols to functions (cleaner than `lambda a,b: a+b`).

#### 3. Walrus Operator in List Comprehension

**Question**: You have a list of strings. Create a list of lengths, but ONLY for strings longer than 3 characters. You must use the Walrus operator to avoid calculating `len(s)` twice.

* **Solution**:
```python
data = ["hi", "hello", "python", "no"]

# Goal: [5, 6] (Lengths of hello, python)
# Using := to capture len(s) inside the if condition
result = [n for s in data if (n := len(s)) > 3]

print(result)

```


* **Explanation**: `n` is assigned `len(s)` inside the filter part of the comprehension. If `n > 3`, the value `n` is then produced in the output list.
* **Comparison**: Without walrus: `[len(s) for s in data if len(s) > 3]` (Calculates length twice).

---

### Medium Level (Questions 4–10)

#### 4. Vector Class (Operator Overloading)

**Question**: Implement a `Vector` class that supports addition `v1 + v2` and scalar multiplication `v1 * 3`.

* **Solution**:
```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x + other.x, self.y + other.y)
        return NotImplemented

    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

# Usage
v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2) # Vector(4, 6)
print(v1 * 3)  # Vector(3, 6)

```


* **Key Concept**: Correct implementation of `__add__` and `__mul__`.
* **Note**: For `3 * v1` to work, you would also need `__rmul__`.

#### 5. Short-Circuiting Safe Navigation

**Question**: You have a nested dictionary `data`. Access `data['user']['profile']['name']` safely. If any key is missing or None, return "Unknown". Do not use try/except. Use Logical Operators.

* **Solution**:
```python
def get_name(data):
    # Using logical 'and' to guard access
    # 'or' provides the default
    return (data and data.get('user')
                 and data['user'].get('profile')
                 and data['user']['profile'].get('name')) or "Unknown"

# Usage
bad_data = {'user': None}
print(get_name(bad_data)) # Unknown

```


* **Explanation**: `a and b` returns `b` if `a` is Truthy. If `data` is None, it returns None (Falsy). If `data['user']` is None, the chain stops. Finally `... or "Unknown"` catches the resulting None and returns the string.
* **Modern Python**: Python 3.10+ doesn't have `?.` (safe nav) yet, so this pattern or `try/except` is standard.

#### 6. Custom Comparison Logic (Rich Comparison)

**Question**: Create a class `Box` with volume. Implement logic so `box1 > box2` compares volumes. Also allow sorting a list of boxes.

* **Solution**:
```python
import functools

@functools.total_ordering
class Box:
    def __init__(self, l, w, h):
        self.vol = l * w * h

    def __eq__(self, other):
        return self.vol == other.vol

    def __lt__(self, other):
        return self.vol < other.vol

# Usage
boxes = [Box(1,1,1), Box(2,2,2), Box(1,2,1)]
boxes.sort() # Uses __lt__
print([b.vol for b in boxes]) # [1, 2, 8]

```


* **Explanation**: `@total_ordering` is a decorator that generates missing comparison methods (`__le__`, `__gt__`, `__ge__`) if you define just `__eq__` and one other (like `__lt__`). This saves writing 6 methods.

#### 7. Modulo Arithmetic (Cyclic Iterator)

**Question**: Implement a function `get_cyclic_item(lst, index)` that allows accessing a list with any integer index (positive or negative), wrapping around endlessly.

* **Solution**:
```python
def get_cyclic_item(lst: list, index: int):
    if not lst: return None
    # The % operator handles negative numbers correctly in Python
    # -1 % 5 -> 4 (The last item)
    cyclic_index = index % len(lst)
    return lst[cyclic_index]

# Usage
items = ['a', 'b', 'c']
print(get_cyclic_item(items, 0))  # 'a'
print(get_cyclic_item(items, 3))  # 'a' (Wraps)
print(get_cyclic_item(items, -1)) # 'c' (Wraps backwards)

```


* **Why Interviewers Ask This**: To verify knowledge that `%` in Python is safe for negative indices (unlike C++ where it depends on implementation).

#### 8. In-Place Operator Optimization (`__iadd__`)

**Question**: Write a class `Accumulator` that holds a list of logs. Implement `+=` such that it modifies the list in place efficiently, but `+` creates a new instance.

* **Solution**:
```python
class Accumulator:
    def __init__(self, logs=None):
        self.logs = logs if logs else []

    def __add__(self, item):
        # New instance
        return Accumulator(self.logs + [item])

    def __iadd__(self, item):
        # Modify self.logs in place
        self.logs.append(item)
        # IMPORTANT: __iadd__ must return self
        return self

# Usage
acc = Accumulator()
acc += "Log 1" # Calls __iadd__, efficient append
acc2 = acc + "Log 2" # Calls __add__, new list creation

```


* **Common Mistake**: Forgetting `return self` in `__iadd__`. If you return `None`, `acc += item` will set `acc` to `None`.

#### 9. XOR Swap Logic (using Assignment)

**Question**: Though not using bitwise operators, simulate a variable swap using only assignment and arithmetic `+` and `-`.

* **Solution**:
```python
def arithmetic_swap(a, b):
    a = a + b # Sum
    b = a - b # (Sum - b) = Original a
    a = a - b # (Sum - a) = Original b
    return a, b

# Pythonic Way (Interviewers prefer this)
# a, b = b, a

```


* **Why Interviewers Ask This**: To test mathematical logic.
* **Caveat**: In languages with fixed integer sizes, this risks overflow. In Python, it is safe because integers are arbitrary precision.

#### 10. Matrix Multiplication Class

**Question**: Implement the `__matmul__` (`@`) operator for a simple 2x2 Matrix class.

* **Solution**:
```python
class Matrix2x2:
    def __init__(self, grid):
        # grid is [[a,b], [c,d]]
        self.grid = grid

    def __matmul__(self, other):
        # Simple Dot Product Logic for 2x2
        # [ a b ] @ [ e f ]
        # [ c d ]   [ g h ]
        # r00 = ae + bg, etc.
        A = self.grid
        B = other.grid
        result = [
            [A[0][0]*B[0][0] + A[0][1]*B[1][0], A[0][0]*B[0][1] + A[0][1]*B[1][1]],
            [A[1][0]*B[0][0] + A[1][1]*B[1][0], A[1][0]*B[0][1] + A[1][1]*B[1][1]]
        ]
        return Matrix2x2(result)

# Usage
m1 = Matrix2x2([[1, 2], [3, 4]])
m2 = Matrix2x2([[1, 0], [0, 1]]) # Identity
m3 = m1 @ m2
print(m3.grid) # [[1, 2], [3, 4]]

```


* **Developer Scenario**: Custom ML layers or geometric transformations.
