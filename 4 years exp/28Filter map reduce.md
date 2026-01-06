# Python Filter map reduce— Interview Guide (30 Questions)

This guide covers Python's functional programming primitives: `filter()`, `map()`, and `functools.reduce()`. It is tailored for a developer with 4 years of experience, focusing on modern Python 3.12+ semantics, lazy evaluation, and backend data processing scenarios.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Question 1 (Easy): What is the fundamental difference between `map()` in Python 2 vs. Python 3?

**Expected Answer:**
In Python 2, `map()` returns a `list` immediately (eager evaluation). In Python 3, `map()` returns a `map` object, which is an **iterator** (lazy evaluation). The values are generated only when consumed (e.g., by looping or converting to a list).

**Explanation:**
This change improves memory efficiency. If you map a function over 1 million elements but only need the first 10, Python 3 won't process the other 999,990 elements.

**Time & Space Complexity:**

* **Python 2:** O(N) space (creates full list).
* **Python 3:** O(1) auxiliary space (returns an iterator object).

**Common Mistakes:**

* Expecting a list indexable object immediately in Python 3.
* Trying to print the map object directly and seeing `<map object at ...>` instead of values.

**Why Interviewers Ask:** To check if you understand lazy evaluation and Python version history.

**Sample Code:**

```python
# Python 3.12+
nums = [1, 2, 3, 4, 5]
mapped = map(lambda x: x * 2, nums)

print(mapped)        # <map object at 0x...>
print(list(mapped))  # [2, 4, 6, 8, 10]

```

---

### Question 2 (Easy): How does `filter()` handle `None` as the first argument?

**Expected Answer:**
If `None` is passed as the first argument to `filter()`, it defaults to the identity function. It filters out all elements that evaluate to `False` in a boolean context (falsy values like `0`, `""`, `[]`, `None`, `False`).

**Explanation:**
This is a shorthand for cleaning "empty" or "false" data from a sequence without writing a lambda like `lambda x: bool(x)`.

**Common Mistakes:**

* Thinking `filter(None, iterable)` returns an error.
* Thinking it filters *only* `None` values (it filters all falsy values).

**Why Interviewers Ask:** syntax familiarity and understanding of Python's "truthiness".

**Sample Code:**

```python
data = [0, 1, False, True, "hello", "", None]
cleaned = list(filter(None, data))

print(cleaned) 
# Output: [1, True, 'hello']
# Removed: 0, False, "", None

```

---

### Question 3 (Easy): What module does `reduce()` live in, and why was it moved there in Python 3?

**Expected Answer:**
`reduce()` resides in the `functools` module. It was moved from the global namespace (in Python 2) to force developers to think twice before using it, as Guido van Rossum (Python's creator) felt it often led to unreadable code compared to loops or recursion.

**Explanation:**
While `map` and `filter` are common, `reduce` collapses a sequence into a single value, which can be obscure.

**Common Mistakes:**

* Trying to call `reduce()` without importing it.
* Assuming it's still a built-in.

**Sample Code:**

```python
from functools import reduce

nums = [1, 2, 3, 4]
# Summing items
total = reduce(lambda acc, x: acc + x, nums) 
print(total) # 10

```

---

### Question 4 (Easy): Can `map()` accept multiple iterables? If so, how does it behave?

**Expected Answer:**
Yes, `map()` can accept multiple iterables. The function passed must accept as many arguments as there are iterables. The iteration stops when the *shortest* iterable is exhausted.

**Explanation:**
It parallels the behavior of `zip()`.

**ASCII Diagram:**

```text
Iterable A: [1, 2, 3]
Iterable B: [10, 20]
Function  : (x, y) -> x + y

Step 1: 1 + 10 -> 11
Step 2: 2 + 20 -> 22
Step 3: 3 + (No counterpart) -> Stop

```

**Common Mistakes:**

* Providing a function that takes 1 argument when providing 2 lists.
* Expecting it to error on length mismatch (it truncates silently).

**Sample Code:**

```python
a = [1, 2, 3]
b = [10, 20]
# lambda takes 2 args because 2 lists are passed
result = list(map(lambda x, y: x + y, a, b))
print(result) # [11, 22]

```

---

### Question 5 (Easy): What happens if the function passed to `filter()` returns a non-boolean value?

**Expected Answer:**
Python evaluates the return value in a **boolean context**. If the function returns a "truthy" value (e.g., non-zero integer, non-empty string), the element is kept. If "falsy", it is discarded.

**Explanation:**
You don't need to return explicit `True` or `False`.

**Common Mistakes:**

* Thinking you must return exactly `True` or `False`.

**Sample Code:**

```python
# Returns the number itself (int), not bool
def is_valid(n):
    return n % 2 # returns 1 (True) or 0 (False)

nums = [1, 2, 3, 4]
print(list(filter(is_valid, nums))) # [1, 3]

```

---

### Question 6 (Easy): What is the "initializer" argument in `reduce()` and why is it important?

**Expected Answer:**
The initializer is the optional third argument to `reduce()`. It serves as the starting value of the accumulator. If the iterable is empty, `reduce` returns this initializer. If omitted and the iterable is empty, `reduce` raises a `TypeError`.

**Explanation:**
Always using an initializer is a best practice to prevent crashes on empty lists and to define a clear starting type (e.g., `0` for sums, `[]` for lists).

**Common Mistakes:**

* Omitting initializer on data that might be empty.
* Assuming the first item of the list is always the starting accumulator (only true if initializer is missing).

**Sample Code:**

```python
from functools import reduce

# Empty list with initializer
print(reduce(lambda x, y: x + y, [], 0)) # Output: 0

# Empty list WITHOUT initializer
# reduce(lambda x, y: x + y, []) # Raises TypeError

```

---

### Question 7 (Medium): Compare `map()/filter()` vs. List Comprehensions regarding performance and memory.

**Expected Answer:**

* **Memory:** `map`/`filter` in Python 3 are iterators (O(1) memory), whereas list comprehensions create the full list in memory (O(N)), unless you use Generator Expressions `(...)`.
* **Speed:** For simple operations using built-ins (like `map(str, data)`), `map` can be slightly faster than comprehensions because the loop is pushed to C-level. For lambda functions, comprehensions are often faster due to the overhead of lambda function calls.

**Reasoning:**
Backend systems processing large streams (ETL jobs) prefer iterators (map/filter/generators) to avoid OOM (Out of Memory) errors.

**Common Mistakes:**

* Converting `map` to `list()` immediately, negating the memory benefit.
* Using `map` with a complex lambda where a comprehension would be more readable.

**Sample Code:**

```python
import sys

# Memory difference
r_map = map(lambda x: x**2, range(1000000))
r_list = [x**2 for x in range(1000000)]

print(sys.getsizeof(r_map))  # ~48 bytes (constant)
print(sys.getsizeof(r_list)) # ~8 MB (linear)

```

---

### Question 8 (Medium): How can you use `reduce()` to implement a "pipeline" of functions?

**Expected Answer:**
You can use `reduce()` to apply a list of functions sequentially to a single piece of data. The accumulator becomes the data flowing through the functions.

**Explanation:**
This is similar to the "Unix pipe" concept or functional composition .

**ASCII Diagram:**

```text
Data -> [Func1] -> Data' -> [Func2] -> Data'' -> Result

```

**Common Mistakes:**

* Confusing the arguments: `reduce(function, iterable, initializer)`. Here, the *iterable* is the list of functions, and the *initializer* is the input data.

**Sample Code:**

```python
from functools import reduce

def add_five(x): return x + 5
def double(x): return x * 2
def stringify(x): return f"Value: {x}"

funcs = [add_five, double, stringify]
data = 10

# ((10 + 5) * 2) -> stringify
result = reduce(lambda val, func: func(val), funcs, data)
print(result) # "Value: 30"

```

---

### Question 9 (Medium): Why can `map()` iterators only be consumed once?

**Expected Answer:**
`map()` returns a Python **generator-like iterator**. Iterators are stateful; they keep track of their current position. Once they yield `StopIteration`, they are exhausted. You cannot rewind a standard iterator.

**Reasoning:**
To reuse the data, you must either convert it to a sequence (list/tuple) or create a fresh iterator. This prevents hidden memory costs of storing history.

**Common Mistakes:**

* Assigning a map to a variable and trying to loop over it twice.

**Sample Code:**

```python
numbers = [1, 2, 3]
squares = map(lambda x: x**2, numbers)

print(list(squares)) # [1, 4, 9]
print(list(squares)) # [] -> Empty!

```

---

### Question 10 (Medium): How do you debug a complex `map()` or `filter()` chain without converting to list?

**Expected Answer:**
You cannot inspect items inside a map/filter object without consuming them. However, you can:

1. Wrap the iterator with a logging function that prints on access (side effects).
2. Use `itertools.islice` to peek at the first few items without consuming the whole stream.

**Reasoning:**
In production data pipelines, converting a 10GB stream to a list to debug is impossible.

**Sample Code:**

```python
def log_and_pass(x):
    print(f"Processing: {x}")
    return x

data = map(lambda x: x * 2, range(3))
# Inject logging helper
debugged_data = map(log_and_pass, data)

# Consumption triggers print
list(debugged_data)
# Output:
# Processing: 0
# Processing: 2
# Processing: 4

```

---

### Question 11 (Medium): Explain how `filter()` interacts with Exception handling inside the lambda.

**Expected Answer:**
It doesn't handle exceptions automatically. If the function passed to `filter` raises an exception for a specific element, the entire iteration crashes.

**Reasoning:**
You must handle exceptions *inside* the helper function if the data is dirty.

**Common Mistakes:**

* Expecting `filter` to just "skip" items that cause errors.

**Sample Code:**

```python
raw_data = ["1", "2", "bad", "3"]

def safe_int_check(x):
    try:
        int(x)
        return True
    except ValueError:
        return False

# filter keeps only valid integers
clean = filter(safe_int_check, raw_data)
print(list(clean)) # ['1', '2', '3']

```

---

### Question 12 (Medium): Is it possible to break out of a `map()` loop early?

**Expected Answer:**
No, not directly via `map` syntax because `map` has no "break" condition. However, since `map` is lazy, if the *consumer* (the loop iterating over the map) breaks, the `map` stops processing.

**Reasoning:**
The control flow belongs to the consumer, not the producer.

**Common Mistakes:**

* Trying to raise `StopIteration` inside the lambda to stop `map` (this just stops that specific value's generation, but standard behavior usually ends the iterator).

**Sample Code:**

```python
def expensive_op(x):
    print(f"Computing {x}")
    return x * x

m = map(expensive_op, range(10))

for val in m:
    if val > 5:
        break 
# 'Computing' prints stop after break. 
# Remaining items (3,4,...) are never computed.

```

---

### Question 13 (Medium): How would you use `reduce` to flatten a list of lists?

**Expected Answer:**
You can use `reduce` with list concatenation (`+` or `extend`).
`reduce(lambda acc, item: acc + item, list_of_lists, [])`

**Time Complexity:**
O(N^2) if using `+` operator because Python creates a new list for every concatenation.
Better approach: `itertools.chain` is O(N).

**Why Interviewers Ask:** To see if you recognize the performance trap of repeated list concatenation in `reduce`.

**Sample Code:**

```python
from functools import reduce
nested = [[1, 2], [3, 4], [5]]

# Functional flattening (Note: O(N^2) behavior)
flat = reduce(lambda acc, x: acc + x, nested, [])
print(flat) # [1, 2, 3, 4, 5]

```

---

### Question 14 (Medium): Can `map` functions modify the original list elements in place?

**Expected Answer:**
Yes, if the list contains **mutable objects** (like dicts or lists) and the map function modifies them as a side effect. This is generally considered bad practice (violation of functional purity).

**Reasoning:**
Functional programming discourages side effects. `map` should transform data into new data, not mutate old data.

**Sample Code:**

```python
users = [{'name': 'A'}, {'name': 'B'}]

def pollute(u):
    u['seen'] = True  # Side effect!
    return u

list(map(pollute, users))
print(users) # [{'name': 'A', 'seen': True}, ...] -> Original changed!

```

---

### Question 15 (Medium): How do `map/filter` behave with infinite iterators (like `itertools.count`)?

**Expected Answer:**
They return an infinite iterator. They do not hang immediately. They only hang if you try to consume them fully (e.g., `list(map(...))`).

**Reasoning:**
This allows defining infinite data streams. You must use `itertools.islice` or a `break` in the consumer loop to handle them.

**Sample Code:**

```python
from itertools import count, islice

# Infinite stream of evens
evens = filter(lambda x: x % 2 == 0, count())

# Take first 5 safely
print(list(islice(evens, 5))) # [0, 2, 4, 6, 8]

```

---

### Question 16 (Medium): What is the difference between `itertools.starmap` and `map`?

**Expected Answer:**
`map` applies the function `f(x)` to elements.
`starmap` applies the function `f(*x)` (unpacks arguments).
Used when the iterable yields tuples of arguments that need to be spread into the function.

**Why Interviewers Ask:** It shows deep knowledge of the standard library for data processing.

**Sample Code:**

```python
from itertools import starmap

data = [(2, 5), (3, 2), (10, 3)] # tuples (base, exponent)
# map(pow, data) -> Error: pow expects 2 args, got 1 tuple
# starmap(pow, data) -> pow(2, 5), pow(3, 2)...

results = list(starmap(pow, data))
print(results) # [32, 9, 1000]

```

---

### Question 17 (Medium): Rewrite a standard dictionary comprehension `{k: v for k, v in lst}` using `map` and `dict` constructor.

**Expected Answer:**
`dict(map(lambda x: (key_transform(x), val_transform(x)), lst))`

**Reasoning:**
The `dict()` constructor accepts an iterable of `(key, value)` tuples. `map` can generate these tuples.

**Sample Code:**

```python
users = ["alice", "bob", "charlie"]
# Dict comp: {u: len(u) for u in users}

# Functional approach:
res = dict(map(lambda u: (u, len(u)), users))
print(res) # {'alice': 5, 'bob': 3, 'charlie': 7}

```

---

### Question 18 (Medium): Why is `functools.reduce` often slower than a standard Python `for` loop for simple summation?

**Expected Answer:**

1. **Function Call Overhead:** `reduce` calls the lambda/function for every element. In Python, function calls are expensive.
2. **Optimization:** A simple `for` loop or built-in `sum()` is highly optimized in C.

**Comparison:**
`sum(list)` is significantly faster than `reduce(add, list)`.

**Sample Code:**

```python
import timeit

# reduce vs sum
# sum() is implemented in C and avoids python-level func calls

```

---

### Question 19 (Medium): How do you `filter` a dictionary?

**Expected Answer:**
You cannot pass a dictionary directly to `filter` and expect a dictionary back (iterating a dict yields keys). You must filter the `.items()`, then cast back to `dict()`.

**Sample Code:**

```python
scores = {'a': 10, 'b': 5, 'c': 20}

# Keep scores > 8
passed = dict(filter(lambda item: item[1] > 8, scores.items()))
print(passed) # {'a': 10, 'c': 20}

```

---

### Question 20 (Medium): How can you parallelize `map()` operations in Python?

**Expected Answer:**
The standard `map` is single-threaded. To parallelize, use `multiprocessing.Pool.map` or `concurrent.futures.ProcessPoolExecutor.map`.

**Reasoning:**
For CPU-bound tasks, standard `map` blocks. Multiprocessing bypasses the Global Interpreter Lock (GIL).

**Sample Code:**

```python
from multiprocessing import Pool

def heavy_computation(x):
    return x ** 100

if __name__ == '__main__':
    with Pool(4) as p:
        # Runs in parallel processes
        print(p.map(heavy_computation, [1, 2, 3]))

```

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Question 21 (Easy): Square and Filter

**Question:**
Write a one-liner using `map` and `filter` that takes a list of numbers `[1, 2, 3, 4, 5, 6]`, squares them, and returns a list containing only the squares that are greater than 10.

**Solution:**

```python
nums = [1, 2, 3, 4, 5, 6]

# Logic: Filter first? No, question says "squares that are > 10"
# Usually efficient to filter input first if possible, 
# but here we must check the result of the square.

result = list(filter(lambda x: x > 10, map(lambda x: x**2, nums)))

print(result) # [16, 25, 36]

```

**Line-by-line Explanation:**

1. `map(lambda x: x**2, nums)` creates an iterator yielding 1, 4, 9, 16...
2. `filter(lambda x: x > 10, ...)` consumes that map iterator and keeps items > 10.
3. `list(...)` resolves the iterator to a printable list.

**Time/Space:** O(N) Time, O(N) Space (for output list).

---

### Question 22 (Easy): Product of List

**Question:**
Implement a function `product(lst)` using `reduce` that multiplies all numbers in a list. Handle the edge case of an empty list by returning 1.

**Solution:**

```python
from functools import reduce
import operator

def product(lst):
    # operator.mul is faster than lambda x,y: x*y
    return reduce(operator.mul, lst, 1)

print(product([2, 3, 4])) # 24
print(product([]))        # 1

```

**Why Interviewers Ask:**
Checks if you know `operator` module (cleaner than lambdas) and if you remember the initializer for empty cases.

---

### Question 23 (Easy): Clean User Input

**Question:**
You have a list of user inputs: `["  alice ", "BOB", None, "   ", "charlie"]`.
Use `map` and `filter` to:

1. Remove None or empty/whitespace-only strings.
2. Strip whitespace and convert to title case (e.g., "Alice").

**Solution:**

```python
inputs = ["  alice ", "BOB", None, "   ", "charlie"]

# Step 1: Filter valid strings (not None, not just whitespace)
# Step 2: Map to clean format

# Note: We must filter None first to avoid AttributeError on strip()
cleaned = map(lambda s: s.strip().title(), 
              filter(lambda x: x and x.strip(), inputs))

print(list(cleaned)) 
# Output: ['Alice', 'Bob', 'Charlie']

```

**Real App Usage:**
Sanitizing form data in a Flask/Django backend before saving to DB.

---

### Question 24 (Medium): Grouping/Aggregation with Reduce

**Question:**
Given a list of transactions: `[{"cat": "food", "amt": 10}, {"cat": "travel", "amt": 50}, {"cat": "food", "amt": 20}]`.
Use `reduce` to create a summary dict summing amounts by category: `{'food': 30, 'travel': 50}`.

**Solution:**

```python
from functools import reduce

transactions = [
    {"cat": "food", "amt": 10}, 
    {"cat": "travel", "amt": 50}, 
    {"cat": "food", "amt": 20}
]

def reducer(acc, curr):
    category = curr['cat']
    amount = curr['amt']
    # .get() handles initialization of new keys
    acc[category] = acc.get(category, 0) + amount
    return acc

# Initializer is an empty dict {}
summary = reduce(reducer, transactions, {})

print(summary)

```

**Common Mistakes:**

* Forgetting to return `acc` inside the reducer function. (Python dictionaries are mutable, but `reduce` expects the return value to be passed to the next step).
* **Time Complexity:** O(N).

---

### Question 25 (Medium): Intersection of Multiple Lists

**Question:**
You have a list of lists: `[[1, 2, 3], [2, 3, 4], [3, 4, 5]]`.
Use `reduce` to find the intersection of all lists (elements present in ALL lists).

**Solution:**

```python
from functools import reduce

lists = [[1, 2, 3], [2, 3, 4], [3, 4, 5]]

# Logic: Set Intersection (&) of Set A and Set B, then result & Set C
intersection = reduce(lambda acc, curr: set(acc) & set(curr), lists)

print(list(intersection)) # [3]

```

**Developer Scenario:**
Finding common available time slots across multiple users' calendars.

---

### Question 26 (Medium): Custom Map Implementation

**Question:**
Implement your own version of `my_map(func, iterable)` using generators (yield). It should behave like Python 3's `map` (lazy).

**Solution:**

```python
def my_map(func, iterable):
    for item in iterable:
        yield func(item)

# Verification
data = [1, 2, 3]
gen = my_map(lambda x: x * 2, data)

print(gen)       # <generator object ...>
print(next(gen)) # 2
print(list(gen)) # [4, 6]

```

**Why Interviewers Ask:**
To ensure you understand that `map` is essentially a generator loop, not magic.

---

### Question 27 (Medium): Deep Flatten with Recursion + Reduce

**Question:**
Write a function to flatten a deeply nested list of arbitrary depth `[1, [2, [3, 4], 5]]` using `reduce`. (Note: This is a tricky recursive reduction).

**Solution:**

```python
from functools import reduce

def deep_flatten(lst):
    return reduce(
        lambda acc, x: acc + (deep_flatten(x) if isinstance(x, list) else [x]), 
        lst, 
        []
    )

nested = [1, [2, [3, 4], 5]]
print(deep_flatten(nested)) # [1, 2, 3, 4, 5]

```

**Line-by-line:**

1. The reducer checks `isinstance(x, list)`.
2. If list: Recurse `deep_flatten(x)`.
3. If not list: Wrap in list `[x]` and append to accumulator.
4. **Warning:** This is O(N^2) due to list concatenations. Good for interview logic, bad for production on huge lists.

---

### Question 28 (Medium): Parsing CSV-like Data

**Question:**
Given a string of raw CSV data:
`raw = "id,name,score\n1,Alice,80\n2,Bob,90\n3,Charlie,85"`
Use chaining of `map` and `filter` (and `str.split`) to return a list of dictionaries: `[{'id': '1', 'name': 'Alice', 'score': '80'}, ...]`.

**Solution:**

```python
raw = "id,name,score\n1,Alice,80\n2,Bob,90\n3,Charlie,85"

lines = raw.split('\n')
headers = lines[0].split(',')
data_lines = lines[1:]

# Map each line to a dict
# zip(headers, values) creates pairs
# dict() creates the object
parsed = map(lambda line: dict(zip(headers, line.split(','))), data_lines)

print(list(parsed))
# [{'id': '1', 'name': 'Alice', 'score': '80'}, ...]

```

**Real App Usage:**
Lightweight parsing of uploaded reports without using the `pandas` library.

---

### Question 29 (Medium): Rate Limiter / Window Processing (Chuncked Map)

**Question:**
You need to process 100 items, but your API only accepts batches (chunks) of 3.
Write a line using `map` (and `zip/iter`) to chunk a list `[1, 2, 3, 4, 5, 6, 7]` into `[(1, 2, 3), (4, 5, 6)]`. Ignore the remainder for this exercise or handle strictly equal chunks.

**Solution:**
The "zip(*[iter(s)]*n)" idiom.

```python
data = [1, 2, 3, 4, 5, 6]
n = 3

# This creates n references to the SAME iterator
# zip pulls one from each, effectively consuming n items per step.
chunks = zip(*[iter(data)] * n)

print(list(chunks)) # [(1, 2, 3), (4, 5, 6)]

```

**Explanation:**
This is a famous Python idiom.

1. `iter(data)` creates an iterator.
2. `[iter] * 3` creates a list with 3 references to that *same* iterator.
3. `zip(*...)` pulls from the 1st ref (gets 1), then 2nd ref (gets 2), then 3rd (gets 3).

---

### Question 30 (Medium): Conditional Transformation Pipeline

**Question:**
You have a list of products.

1. Apply a discount of 10% to "Electronics".
2. Apply a tax of 5% to ALL items.
3. Filter out items that cost less than $50 after calculations.
Use `map` for transformations and `filter` for selection.

**Solution:**

```python
products = [
    {"cat": "elec", "price": 100},
    {"cat": "book", "price": 40},
    {"cat": "elec", "price": 50}
]

def apply_discount(p):
    if p["cat"] == "elec":
        p["price"] *= 0.9
    return p

def apply_tax(p):
    p["price"] *= 1.05
    return p

# Pipeline
# Note: map is lazy. Order of wrapping matters.
# Inner map runs first (discount), then outer map (tax), then filter.

step1 = map(apply_discount, products)
step2 = map(apply_tax, step1)
final = filter(lambda p: p["price"] >= 50, step2)

# Rounding for display
print([{k: round(v, 2) if k=="price" else v for k,v in x.items()} 
       for x in final])

# Logic trace:
# Item 1: 100 -> 90 -> 94.5 (Keep)
# Item 2: 40 -> 40 -> 42.0 (Drop < 50)
# Item 3: 50 -> 45 -> 47.25 (Drop < 50)

```
