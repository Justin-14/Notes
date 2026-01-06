Here is the complete, professionally formatted interview guide for Python Functions and Function Arguments, tailored for a developer with 4 years of experience.

# Python Function, function argument— Interview Guide (30 Questions)

## Introduction

This guide focuses on mastering Python functions, argument passing mechanisms, scopes, and advanced features like closures and decorators. It is designed for a developer with ~4 years of experience, moving beyond syntax into behavior, performance, and best practices in Python 3.12+.

---

# SECTION 1 — THEORY QUESTIONS (20 TOTAL)

## Part A: Easy Level (6 Questions)

### 1. Function Arguments: Positional vs. Keyword

**Question:** Explain the difference between positional and keyword arguments in a function call. Can they be mixed?

* **Expected Answer:** Positional arguments are assigned to parameters based on their order. Keyword arguments are assigned based on the parameter name. They can be mixed, but positional arguments must always come *before* keyword arguments in the function call.
* **Clear Explanation:** Python matches arguments from left to right. Once the interpreter encounters a keyword argument, it expects all subsequent arguments to be keyword-based to avoid ambiguity.
* **Medium-depth Reasoning:** This flexibility allows APIs to change their internal parameter order (if keywords are forced) without breaking legacy code, provided the names remain consistent.
* **Common Mistakes:** Placing a positional argument after a keyword argument (`SyntaxError`).
* **Why Interviewers Ask This:** Checks basic syntax compliance and understanding of how Python parses function calls.
* **Sample Code:**
```python
def connect(host, port, secure=False):
    print(f"Connecting to {host}:{port} (Secure: {secure})")

# Mixed usage
connect("localhost", 8080, secure=True) 

# Error scenario
# connect(host="localhost", 8080) # SyntaxError: positional argument follows keyword argument

```



### 2. The `return` Statement & Implicit None

**Question:** What happens if a Python function does not have a `return` statement? How does this differ from `return None`?

* **Expected Answer:** If a function reaches the end of its block without hitting a `return`, it implicitly returns `None`. Functionally, `return` (empty), `return None`, and no return statement are identical in terms of the value received by the caller.
* **Clear Explanation:** Python functions are objects that *must* result in a value when called. `None` is the singleton object used to represent "no value".
* **Reasoning:** Explicit `return None` is often used for clarity (e.g., early exit in a search function), while implicit return is common in "void" functions that perform side effects (like logging).
* **Why Interviewers Ask This:** To ensure you understand Python's consistent object model—everything returns something.
* **Sample Code:**
```python
def implicit():
    pass

def explicit():
    return None

print(implicit() is explicit()) # True

```



### 3. Argument Unpacking (`*` and `**`)

**Question:** What do the `*` and `**` operators do when used in a function call (not definition)?

* **Expected Answer:** They unpack collections into individual arguments. `*` unpacks an iterable (list/tuple) into positional arguments. `**` unpacks a dictionary into keyword arguments.
* **Clear Explanation:** It allows dynamic construction of argument lists. The length of the iterable or keys of the dictionary must match the function signature expectations.
* **Why Interviewers Ask This:** Essential for writing wrappers or passing data dynamically between layers of an application.
* **Sample Code:**
```python
def point(x, y):
    print(f"X: {x}, Y: {y}")

coords = (10, 20)
data = {'y': 50, 'x': 100}

point(*coords)  # Equivalent to point(10, 20)
point(**data)   # Equivalent to point(x=100, y=50)

```



### 4. Docstrings vs. Comments

**Question:** How does a docstring differ from a regular comment in terms of access at runtime?

* **Expected Answer:** Comments (`#`) are ignored by the parser and discarded. Docstrings (string literals as the first statement) are retained and assigned to the function's `__doc__` attribute, accessible at runtime.
* **Clear Explanation:** Tools like Sphinx, IDEs, and the `help()` function rely on `__doc__` to generate documentation.
* **Why Interviewers Ask This:** Tests knowledge of introspection and professional coding standards.
* **Sample Code:**
```python
def my_func():
    """This is a docstring."""
    # This is a comment
    pass

print(my_func.__doc__) # Outputs: This is a docstring.

```



### 5. Lambda Functions

**Question:** What is a lambda function, and what is its primary limitation compared to a `def` function?

* **Expected Answer:** An anonymous, inline function defined with `lambda`. Its limitation is that it is restricted to a **single expression**. It cannot contain statements (like `assignment`, `while`, `try/except`) or multiple lines of logic.
* **Clear Explanation:** Lambdas return the result of the expression automatically. They are syntactic sugar usually used for short callbacks or keys in sorting.
* **Common Mistakes:** Trying to put complex logic or print statements inside a lambda.
* **Why Interviewers Ask This:** To check if you know when *not* to use them (readability).
* **Sample Code:**
```python
# Valid
adder = lambda x, y: x + y

# Invalid (SyntaxError)
# broken = lambda x: x = x + 1 

```



### 6. Scope Basics (Global vs. Local)

**Question:** If you assign a value to a variable inside a function that has the same name as a global variable, what happens?

* **Expected Answer:** Python creates a new *local* variable in the function's scope. It does not modify the global variable unless you explicitly use the `global` keyword.
* **Clear Explanation:** This is due to Python's scoping rules. Assignment defines the variable in the current scope.
* **Why Interviewers Ask This:** A fundamental source of bugs in beginner-to-intermediate code.
* **Sample Code:**
```python
x = 10
def change_x():
    x = 20 # New local variable
    print(x) # 20

change_x()
print(x) # 10 (Global remains untouched)

```



---

## Part B: Medium Level (14 Questions)

### 7. Mutable Default Arguments (The Trap)

**Question:** Explain the "mutable default argument" trap in Python. What is the output of the code below and why?

```python
def append_to(num, target=[]):
    target.append(num)
    return target

print(append_to(1))
print(append_to(2))

```

* **Expected Answer:** Output is `[1]` then `[1, 2]`. Default argument values are evaluated **only once** when the function is defined, not every time the function is called.
* **Clear Explanation:** The list `[]` is created at definition time and stored in `append_to.__defaults__`. Every call that uses the default references that *same* list object.
* **Medium-depth Reasoning:** Python functions are first-class objects; their default values are attributes of that object.
* **Common Mistakes:** Assuming `target=[]` creates a new list on every call.
* **Why Interviewers Ask This:** This is the #1 Python "gotcha" question.
* **Correct Pattern:** Use `None` as the default.
```python
def append_to(num, target=None):
    if target is None:
        target = []
    target.append(num)
    return target

```



### 8. `*args` and `**kwargs` Implementation

**Question:** How does Python handle `*args` and `**kwargs` internally? What data structures are they?

* **Expected Answer:** `*args` collects extra positional arguments into a **tuple**. `**kwargs` collects extra keyword arguments into a **dictionary**.
* **Clear Explanation:** These are catch-all parameters. They allow functions to accept an arbitrary number of inputs.
* **Reasoning:** Essential for decorators or subclassing where you want to pass arguments through to a wrapped function without knowing what they are.
* **Common Mistakes:** Using `*args` after `**kwargs` (SyntaxError).
* **ASCII Diagram:**
```text
Call: func(1, 2, a=3, b=4)
   |
   v
Def:  func(p1, *args, **kwargs)
   |      |      |        |
   |      v      v        v
Maps:    p1=1  args=(2,) kwargs={'a':3, 'b':4}

```



### 9. Keyword-Only Arguments

**Question:** How do you force a caller to use keyword arguments for specific parameters in Python 3?

* **Expected Answer:** By placing an asterisk `*` in the function signature. Arguments defined *after* the `*` are keyword-only.
* **Clear Explanation:** This ensures clarity for boolean flags or configuration options where positional arguments might be confusing.
* **Why Interviewers Ask This:** Shows knowledge of modern Python API design.
* **Sample Code:**
```python
# 'wait' must be a keyword
def process_data(data, *, wait=True): 
    pass

process_data("data", wait=False) # OK
# process_data("data", False)    # TypeError

```



### 10. Positional-Only Arguments (`/`)

**Question:** What does the forward slash `/` signify in a function signature (Python 3.8+)?

* **Expected Answer:** It marks parameters to its left as **positional-only**. They cannot be passed as keyword arguments.
* **Reasoning:** This is useful when parameter names are not semantic (e.g., `pow(x, y)` vs `pow(base=x, exp=y)`) or to enforce C-extension like behavior.
* **Comparison:**
* `def f(p1, p2, /, p_or_kw, *, kw):`
* `p1`, `p2`: Positional only.
* `p_or_kw`: Positional or keyword.
* `kw`: Keyword only.


* **Sample Code:**
```python
def calc(x, y, /):
    return x + y

calc(1, 2)      # OK
# calc(x=1, y=2) # TypeError

```



### 11. Pass-by-Assignment (Object Reference)

**Question:** Is Python "pass-by-value" or "pass-by-reference"?

* **Expected Answer:** Neither. It is **"pass-by-assignment"** (or "call-by-object-reference").
* **Clear Explanation:** The *reference* (memory address) to the object is passed by value.
1. If the object is **mutable** (list, dict), changes inside the function affect the original object.
2. If the object is **immutable** (int, string), attempting to change it creates a *new* object locally, leaving the original untouched.


* **Why Interviewers Ask This:** Crucial for understanding side effects.
* **Diagram:**
```text
[ Variable 'x' ] ----> [ List Object [1, 2] ] <---- [ Argument 'arg' ]
                              |
                       Function modifies object
                              v
                       [ List Object [1, 2, 3] ]
Both 'x' and 'arg' still point here.

```



### 12. LEGB Rule (Scope Resolution)

**Question:** Explain the LEGB rule for variable resolution in Python.

* **Expected Answer:** When Python looks up a variable name, it searches scopes in this order:
1. **L**ocal (Inside the current function)
2. **E**nclosing (Inside enclosing functions - closures)
3. **G**lobal (Module level)
4. **B**uilt-in (Python built-ins like `len`, `str`)


* **Reasoning:** Understanding this order explains why you can shadow built-in functions (e.g., defining a variable named `list` is bad practice but valid because Local is checked before Built-in).

### 13. Closures

**Question:** What is a closure? How does it relate to the `nonlocal` keyword?

* **Expected Answer:** A closure is a function object that remembers values in enclosing scopes even if they are not present in memory. It occurs when a nested function references a value in its outer function.
* **The `nonlocal` keyword:** Allows the nested function to *modify* a variable in the Enclosing (outer) scope. Without `nonlocal`, assignment would create a new Local variable.
* **Sample Code:**
```python
def outer():
    count = 0
    def inner():
        nonlocal count
        count += 1
        return count
    return inner

counter = outer()
print(counter()) # 1
print(counter()) # 2

```



### 14. Late Binding in Closures

**Question:** What is the output of this code and why? How do you fix it?

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])

```

* **Expected Answer:** Output is `[2, 2, 2]`. This is **late binding**.
* **Explanation:** The lambda functions look up `i` when they are *called*, not when they are *defined*. By the time they are called, the loop has finished and `i` is 2.
* **Fix:** Force early binding using a default argument: `lambda i=i: i`.
* **Why Interviewers Ask This:** Tests deep understanding of variable binding.

### 15. Decorators (Function Wrappers)

**Question:** What is a decorator technically? How does `@my_decorator` syntax translate to code?

* **Expected Answer:** A decorator is a function that takes another function as an argument and returns a new function (usually extending the behavior).
* **Translation:**
```python
@decorator
def func(): pass

# Is exactly equivalent to:
func = decorator(func)

```


* **Use Cases:** Logging, timing, authentication, caching.

### 16. `functools.wraps`

**Question:** Why should you use `functools.wraps` when writing a decorator?

* **Expected Answer:** When a function is decorated, it is replaced by the wrapper function. Without `wraps`, the original function's metadata (`__name__`, `__doc__`, annotations) is lost (it shows the wrapper's info). `wraps` copies this metadata back to the wrapper.
* **Impact:** Debugging becomes a nightmare without it because stack traces will show `wrapper` instead of the actual function name.
* **Sample Code:**
```python
from functools import wraps

def logger(func):
    @wraps(func) # Preserves identity
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

```



### 17. Generators vs. Functions

**Question:** How does a generator function differ from a standard function in execution flow?

* **Expected Answer:** A standard function runs until `return` and disposes of its local state. A generator uses `yield`. When `yield` is hit, the function **pauses**, saving its stack/local variables. It resumes from that exact point when `next()` is called on it.
* **Memory Impact:** Generators are memory efficient (lazy evaluation) as they produce one item at a time rather than storing a full list in memory.
* **Sample Code:**
```python
def count_up_to(n):
    count = 1
    while count <= n:
        yield count
        count += 1

```



### 18. Function Annotation & Introspection

**Question:** How can you access the type hints defined in a function at runtime?

* **Expected Answer:** Using the `__annotations__` attribute of the function object.
* **Reasoning:** Python type hints are not enforced by the interpreter (they are ignored at runtime mostly), but they are stored in this dictionary for third-party tools (mypy, Pydantic) or runtime validation libraries.
* **Sample Code:**
```python
def add(x: int, y: int) -> int:
    return x + y

print(add.__annotations__) 
# {'x': <class 'int'>, 'y': <class 'int'>, 'return': <class 'int'>}

```



### 19. The `__code__` Attribute

**Question:** What is stored in `func.__code__`?

* **Expected Answer:** It stores the compiled bytecode and low-level details of the function (number of local variables, stack size, variable names).
* **Why Interviewers Ask This:** Shows knowledge of Python internals. For example, `func.__code__.co_argcount` tells you how many positional arguments a function expects.

### 20. First Class Functions

**Question:** What does it mean that functions are "First Class Citizens" in Python?

* **Expected Answer:** It means functions can be:
1. Assigned to variables.
2. Passed as arguments to other functions.
3. Returned from other functions.
4. Stored in data structures (lists, dicts).


* **Developer Scenario:** This enables higher-order functions (like `map`, `filter`) and the strategy pattern (selecting a function to run from a dictionary based on user input).

---

# SECTION 2 — CODING QUESTIONS (10 TOTAL)

## Part A: Easy Level (3 Questions)

### 21. Flexible Summation

**Question:** Write a function `sum_all` that takes any number of arguments. If the arguments are numbers, sum them. If they are lists/tuples of numbers, flatten and sum them.

**Solution:**

```python
def sum_all(*args):
    total = 0
    for arg in args:
        if isinstance(arg, (list, tuple)):
            total += sum(arg)
        elif isinstance(arg, (int, float)):
            total += arg
    return total

# Usage
print(sum_all(1, 2, [3, 4], (5,))) # 15

```

* **Explanation:** Uses `*args` to accept variable inputs. `isinstance` checks the type to decide whether to add directly or sum the iterable.
* **Complexity:** O(N) where N is total number of elements.
* **Real App Usage:** A metrics aggregation function that might receive single data points or batches of data points.

### 22. Filter with Predicate

**Question:** Write a function `custom_filter(data, predicate)` that filters a list `data` based on a function `predicate`. Do not use the built-in `filter()`.

**Solution:**

```python
def custom_filter(data, predicate):
    result = []
    for item in data:
        if predicate(item):
            result.append(item)
    return result

# Usage
nums = [1, 2, 3, 4, 5]
is_even = lambda x: x % 2 == 0
print(custom_filter(nums, is_even)) # [2, 4]

```

* **Explanation:** Demonstrates passing a function (`is_even`) as an argument.
* **Why Interviewers Ask This:** Tests understanding of Higher-Order Functions.

### 23. Function Factory (Closure)

**Question:** Create a function `power_factory(n)` that returns a *function*. The returned function should take one argument `x` and return `x` raised to the power of `n`.

**Solution:**

```python
def power_factory(n):
    def nth_power(x):
        return x ** n
    return nth_power

# Usage
square = power_factory(2)
cube = power_factory(3)

print(square(4)) # 16
print(cube(2))   # 8

```

* **Explanation:** `nth_power` captures `n` from the parent scope (Closure).
* **Real App Usage:** Creating specific validators or formatters dynamically based on configuration.

---

## Part B: Medium Level (7 Questions)

### 24. Execution Timer Decorator

**Question:** Write a decorator `@time_it` that measures and prints the time a function takes to execute. It must preserve the metadata of the original function.

**Solution:**

```python
import time
from functools import wraps

def time_it(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@time_it
def heavy_process():
    time.sleep(0.5)
    return "Done"

heavy_process()

```

* **Line-by-line:**
* `@wraps(func)`: Copies `__name__` and docstring.
* `*args, **kwargs`: Ensures the wrapper can handle any function signature.
* `result = func(...)`: Executes the actual logic.
* `return result`: Ensures the return value isn't swallowed.


* **Scenario:** Performance profiling in backend endpoints.

### 25. Memoization (Caching)

**Question:** Implement a `memoize` decorator to cache results of expensive function calls. (Do not use `functools.lru_cache`).

**Solution:**

```python
from functools import wraps

def memoize(func):
    cache = {}
    
    @wraps(func)
    def wrapper(*args):
        if args in cache:
            return cache[args]
        
        result = func(*args)
        cache[args] = result
        return result
    return wrapper

@memoize
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

```

* **Key Detail:** `cache` is a local variable in `memoize`, kept alive by the `wrapper` closure.
* **Limitation:** This simple version only works if `args` are hashable (mutable lists cannot be keys).
* **Real App Usage:** Caching database query results or API responses.

### 26. Retry Decorator with Parameters

**Question:** Write a decorator factory `@retry(times=3)` that retries the decorated function if it raises an Exception, up to a specified number of times.

**Solution:**

```python
import time
from functools import wraps

def retry(times=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == times - 1: # Last attempt
                        raise e
                    print(f"Error: {e}. Retrying...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(times=3)
def unstable_api():
    import random
    if random.random() < 0.7:
        raise ConnectionError("Fail")
    return "Success"

```

* **Complexity:** This is a "Decorator with arguments", requiring three levels of nesting (`retry` -> `decorator` -> `wrapper`).
* **Scenario:** Essential for network requests in microservices (handling flaky connections).

### 27. Argument Validation Decorator

**Question:** Write a decorator that validates that all arguments passed to a function are positive integers. If not, raise a `ValueError`.

**Solution:**

```python
from functools import wraps

def validate_positive(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        for arg in args:
            if not isinstance(arg, int) or arg <= 0:
                raise ValueError(f"Invalid arg: {arg}. Must be +ve int")
        for arg in kwargs.values():
            if not isinstance(arg, int) or arg <= 0:
                raise ValueError(f"Invalid kwarg: {arg}. Must be +ve int")
        return func(*args, **kwargs)
    return wrapper

@validate_positive
def calculate_area(length, width):
    return length * width

```

* **Scenario:** ensuring data integrity in financial calculation functions.

### 28. Flattening JSON (Recursive Function)

**Question:** Write a recursive function to flatten a nested dictionary. Keys should be combined with underscores.
`{'a': 1, 'b': {'c': 2}}` -> `{'a': 1, 'b_c': 2}`

**Solution:**

```python
def flatten_dict(d, parent_key='', sep='_'):
    items = []
    for k, v in d.items():
        new_key = f"{parent_key}{sep}{k}" if parent_key else k
        if isinstance(v, dict):
            items.extend(flatten_dict(v, new_key, sep=sep).items())
        else:
            items.append((new_key, v))
    return dict(items)

data = {'user': {'name': 'John', 'address': {'city': 'NY'}}}
print(flatten_dict(data))
# {'user_name': 'John', 'user_address_city': 'NY'}

```

* **Complexity:** O(N) recursive steps.
* **Common Mistake:** Forgetting to pass the accumulated key down the recursion.
* **Scenario:** Preparing complex JSON logs for ingestion into flat databases like SQL tables or CSV exports.

### 29. Function Pipeline (Composition)

**Question:** Create a function `compose(*funcs)` that takes multiple functions and returns a new function that applies them sequentially (right to left or left to right).
`h(x) = f(g(x))`

**Solution:**

```python
def compose(*funcs):
    def applied(value):
        result = value
        for func in funcs:
            result = func(result)
        return result
    return applied

def add_one(x): return x + 1
def square(x): return x * x

# Pipeline: Input -> add_one -> square
pipeline = compose(add_one, square)
print(pipeline(5)) 
# (5+1) = 6 -> 6*6 = 36

```

* **Developer Scenario:** Data processing pipelines (e.g., Clean String -> Tokenize -> Vectorize).

### 30. Dictionary Dispatch (Switch Case Alternative)

**Question:** Instead of a long `if/elif/else` block, implement a simple calculator (`add`, `sub`, `mul`) using functions stored in a dictionary.

**Solution:**

```python
def add(x, y): return x + y
def sub(x, y): return x - y
def mul(x, y): return x * y

def calculate(operator, x, y):
    # Dispatch table
    ops = {
        'add': add,
        'sub': sub,
        'mul': mul
    }
    
    # Get function, default to a lambda for error
    func = ops.get(operator)
    
    if func:
        return func(x, y)
    else:
        raise ValueError("Invalid operation")

print(calculate('add', 10, 5)) # 15

```

* **Why Interviewers Ask This:** Checks if you know how to use functions as first-class objects to write cleaner, more maintainable code than massive `if/else` blocks.
* **Real App Usage:** Routing API requests based on URL paths or command patterns.
