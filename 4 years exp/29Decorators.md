# Python Decorators — Interview Guide (30 Questions)

This comprehensive guide is designed for a Python developer with 4 years of experience looking to master **Python Decorators**. It covers theory, internal mechanics, and practical backend implementations.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Question 1: What exactly is a Python decorator, and what concept does it rely on? (Easy)

**Expected Answer:**
A decorator is a design pattern in Python that allows a user to add new functionality to an existing object (usually a function or a class) without modifying its structure. It relies on **First-Class Functions**—the ability to pass functions as arguments, return them from other functions, and assign them to variables.

**Explanation:**
Decorators are essentially "wrappers." They take a callable (function), create a closure (inner function) that extends the behavior of the original callable, and return that new callable.

**Medium-depth reasoning:**
Because Python functions are objects, we can manipulate them at runtime. A decorator captures the original function in a closure scope, executing code before or after the original function runs.

**Time & Space Complexity:**

* **Time:**  overhead per call (negligible).
* **Space:**  auxiliary space for the wrapper closure.

**Common Mistakes:**

* Thinking decorators change the function name (without `wraps`).
* Forgetting to return the inner `wrapper` function.

**Why interviewers ask this:**
To verify understanding of High-Order Functions and Closures.

**ASCII Diagram:**

```text
  [Input Function] --> [Decorator Box] --> [Enhanced Function]
       func             wrapper(func)        wrapper

```

**Sample Code:**

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

def say_hello():
    print("Hello!")

# Manual application
say_hello = my_decorator(say_hello)

```

**Line-by-line explanation:**

* `def my_decorator(func):`: Defines a higher-order function receiving `func`.
* `def wrapper():`: Defines the inner function (closure).
* `func()`: Calls the original function captured in scope.
* `return wrapper`: Returns the *new* function logic.
* `say_hello = my_decorator(say_hello)`: Replaces the original variable with the decorated version.

---

### Question 2: Explain the `@` syntactic sugar. (Easy)

**Expected Answer:**
The `@decorator_name` syntax, placed immediately above a function definition, is syntactic sugar for `function = decorator_name(function)`. It applies the decorator at **definition time**, not execution time.

**Explanation:**
Python reads the definition, compiles the function object, passes it to the decorator, and binds the result to the function name in the local scope.

**Medium-depth reasoning:**
It prevents the verbose `func = dec(func)` pattern. Importantly, if multiple decorators are used, they are applied from bottom to top (closest to the function first).

**Common Mistakes:**

* Believing the decorator runs every time the function is called. (The *wrapper* runs on call; the *decorator setup* runs on import/definition).

**Why interviewers ask this:**
To ensure you know how the syntax maps to actual Python execution.

**Sample Code:**

```python
@my_decorator
def say_bye():
    print("Bye!")

# Equivalent to:
# def say_bye(): print("Bye")
# say_bye = my_decorator(say_bye)

```

---

### Question 3: Why do we usually need `*args` and `**kwargs` in the wrapper function? (Easy)

**Expected Answer:**
To ensure the decorator can handle any function signature. The wrapper must accept whatever arguments the original function expects and pass them through.

**Explanation:**
If the wrapper is defined as `def wrapper():` but the decorated function is `def add(a, b):`, calling `add(1, 2)` will raise a `TypeError`. `*args` collects positional arguments into a tuple, and `**kwargs` collects keyword arguments into a dictionary.

**Common Mistakes:**

* Hardcoding arguments in the wrapper.
* Forgetting to pass `*args, **kwargs` to the original `func()`.

**Why interviewers ask this:**
Checks understanding of argument unpacking and generic programming.

**Sample Code:**

```python
def universal_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"Arguments: {args}, {kwargs}")
        return func(*args, **kwargs)
    return wrapper

```

---

### Question 4: What happens to the return value of the decorated function? (Easy)

**Expected Answer:**
The wrapper function must explicitly capture and return the value from the original function. If it doesn't, the decorated function will return `None`.

**Explanation:**
The wrapper replaces the original function. If the original function calculates data, the wrapper is now responsible for handing that data back to the caller.

**Medium-depth reasoning:**
Decorators often perform actions "around" a function (logging, timing). If the `return` statement is omitted in the wrapper, the data flow is broken.

**Common Mistakes:**

* Calling `func(*args)` but forgetting `return func(*args)`.

**Why interviewers ask this:**
A common bug in production is a decorator that swallows return values, breaking business logic.

**Sample Code:**

```python
def return_check(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs) # Capture
        return result # Pass back
    return wrapper

```

---

### Question 5: Can a decorator accept arguments? (Medium)

**Expected Answer:**
Yes, but it requires a three-level function structure. The outer function accepts the arguments, the middle function accepts the target function, and the inner function is the wrapper.

**Explanation:**
When you write `@decorator(arg)`, Python evaluates `decorator(arg)` first, expecting it to return the actual decorator function.

**Medium-depth reasoning:**
Structure:

1. `outer(arg)` -> returns `decorator`
2. `decorator(func)` -> returns `wrapper`
3. `wrapper(*args)` -> calls `func`

**Time & Space Complexity:**

* Complexity is unchanged, but the call stack depth increases during definition.

**Common Mistakes:**

* Confusing the arguments of the decorator with the arguments of the function.
* Forgetting one layer of nesting.

**Why interviewers ask this:**
This is the standard threshold for "intermediate" knowledge of decorators.

**ASCII Diagram:**

```text
@repeat(3)
   |
   +--> calls repeat(3) -> returns 'actual_decorator'
           |
           +--> applies 'actual_decorator(func)'

```

**Sample Code:**

```python
def repeat(num_times):
    def decorator_repeat(func):
        def wrapper(*args, **kwargs):
            for _ in range(num_times):
                func(*args, **kwargs)
        return wrapper
    return decorator_repeat

@repeat(num_times=3)
def greet(name):
    print(f"Hello {name}")

```

---

### Question 6: What is `functools.wraps` and why is it essential? (Easy)

**Expected Answer:**
`functools.wraps` is a helper decorator used on the wrapper function to copy the metadata (name, docstring, annotations) of the original function to the wrapper.

**Explanation:**
Without it, the decorated function takes the name `wrapper` and loses its original docstring. This breaks introspection tools (like `help()` or debuggers) and documentation generators.

**Medium-depth reasoning:**
It sets attributes like `__name__`, `__doc__`, and `__module__` on the wrapper to match the wrapped function.

**Common Mistakes:**

* Omitting it, leading to confusing logs where every function is named `wrapper`.

**Why interviewers ask this:**
It distinguishes between a "hacker" implementation and a "production-grade" implementation.

**Sample Code:**

```python
import functools

def my_dec(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper

@my_dec
def original():
    """Original docstring"""
    pass

print(original.__name__) # Prints "original", not "wrapper"

```

---

### Question 7: Explain the order of execution when multiple decorators are applied. (Medium)

**Expected Answer:**
Decorators are applied from **bottom to top** (closest to the function first). However, the wrapper code *before* the function call runs **top to bottom**, and code *after* the function call runs **bottom to top**.

**Explanation:**
Math analogy: `@f @g def x()` is equivalent to `x = f(g(x))`.

1. `g(x)` is called.
2. The result is passed to `f()`.

**Medium-depth reasoning:**
Think of it like an onion. The top decorator is the outer skin, the bottom decorator is the layer around the core.

**Common Mistakes:**

* Thinking they execute in parallel or top-down completely.
* Order matters significantly for decorators like authentication vs. data serialization.

**Why interviewers ask this:**
Critical for middleware chains in frameworks like Flask or Django.

**ASCII Diagram:**

```text
@dec1 (Top)     | Enter dec1 -> Enter dec2 -> FUNCTION -> Exit dec2 -> Exit dec1
@dec2 (Bottom)  V
def func():

```

**Sample Code:**

```python
def make_bold(func):
    def wrapper():
        return "<b>" + func() + "</b>"
    return wrapper

def make_italic(func):
    def wrapper():
        return "<i>" + func() + "</i>"
    return wrapper

@make_bold
@make_italic
def hello():
    return "hello"

# hello() returns "<b><i>hello</i></b>"

```

---

### Question 8: How do class-based decorators differ from function-based ones? (Medium)

**Expected Answer:**
A class-based decorator uses a class with a `__call__` method. It is often used when the decorator needs to maintain internal state (like a count or cache).

**Explanation:**
The class is instantiated with the function as the argument (`__init__`), and the instance is called when the decorated function is invoked (`__call__`).

**Medium-depth reasoning:**
Using classes can be cleaner for complex decorators, but handling `self` when decorating methods in other classes can be tricky (requires descriptors or `__get__`).

**Common Mistakes:**

* Forgetting `__call__`.
* Using a class decorator on a class method without handling the binding of `self` properly.

**Why interviewers ask this:**
Tests Object-Oriented Programming (OOP) integration with functional concepts.

**Sample Code:**

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.num_calls = 0

    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"Call {self.num_calls}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hi():
    pass

```

---

### Question 9: How would you debug a decorated function? (Medium)

**Expected Answer:**
If `functools.wraps` was used, basic debugging works normally. However, stepping through code will take you into the wrapper logic first. Alternatively, you can access the original function using the `__wrapped__` attribute (added by `wraps`).

**Explanation:**
The `__wrapped__` attribute points to the original, undecorated function object.

**Medium-depth reasoning:**
This allows unit testing the core logic of the function independently of the decorator (e.g., testing the logic without waiting for the `retry` decorator to finish).

**Common Mistakes:**

* Trying to test the decorated version when you only want to verify the business logic.

**Why interviewers ask this:**
Practical experience check. Do you know how to write unit tests for decorated code?

**Sample Code:**

```python
# Calling original function avoiding decorator logic
decoded_result = decorated_function.__wrapped__(*args)

```

---

### Question 10: Can you decorate a Class? What does that do? (Medium)

**Expected Answer:**
Yes. A class decorator receives the class object, modifies it (or returns a new class), and rebinds it. It is commonly used to add methods, register classes to a registry, or enforce singleton patterns.

**Explanation:**
It works exactly like function decorators but targets the class definition.

**Medium-depth reasoning:**
Often cleaner than metaclasses for simple modifications. Commonly used in `dataclasses` (`@dataclass`).

**Common Mistakes:**

* Assuming it decorates every method inside the class automatically (it does not; you have to write logic to do that if desired).

**Why interviewers ask this:**
To see if the candidate understands that classes are also objects in Python.

**Sample Code:**

```python
def add_id(cls):
    cls.class_id = "12345"
    return cls

@add_id
class MyClass:
    pass

print(MyClass.class_id) # 12345

```

---

### Question 11: What is the difference between a decorator and a Context Manager? (Medium)

**Expected Answer:**

* **Decorator:** Wraps a function definition. Affects the function for its *entire lifetime*. Used for "always-on" behavior (logging, auth).
* **Context Manager (`with`):** Wraps a specific block of execution. Used for temporary resources setup/teardown (files, locks, DB connections).

**Explanation:**
Decorators work at the function level; Context Managers work at the block level.

**Medium-depth reasoning:**
However, `contextlib.ContextDecorator` allows a class to act as both.

**Relevant comparisons:**
Use decorator for "How this function behaves". Use context manager for "What context this code runs in".

**Why interviewers ask this:**
Design pattern selection capability.

---

### Question 12: How do you handle side effects in decorators (e.g., Global State)? (Medium)

**Expected Answer:**
Decorators can modify global state or a registry (like a list of routes in Flask). This usually happens at **import time** (when the decorator runs), not call time.

**Explanation:**
The decorator body (outside the wrapper) runs immediately when the module is loaded.

**Medium-depth reasoning:**
This is the mechanism behind web frameworks routing: `@app.route('/')`. The function is added to a global routing table.

**Common Mistakes:**

* Doing heavy computation (DB calls) in the decorator body (slows down app startup).

**Why interviewers ask this:**
Understanding the lifecycle of a Python application (Import vs. Runtime).

**Sample Code:**

```python
registry = []

def register(func):
    registry.append(func) # Side effect at import time
    return func

@register
def a(): pass

```

---

### Question 13: What happens if you apply a decorator to a static method (`@staticmethod`)? (Medium)

**Expected Answer:**
Order matters. `@staticmethod` returns a descriptor, not a callable function. Custom decorators should usually be applied **below** `@staticmethod` (inner) if they expect a function, or handle descriptors if applied above.

**Explanation:**
In Python 3.10+, `@staticmethod` wraps the callable. If your decorator expects a standard function, put it under `@staticmethod`.

**Medium-depth reasoning:**
If you put your decorator on top, it receives a `staticmethod` object, not a function, which often crashes simple wrappers.

**Common Mistakes:**

```python
@staticmethod
@my_decorator  # Works (usually)
def func(): ...

@my_decorator
@staticmethod  # Fails if my_decorator expects a callable immediately
def func(): ...

```

---

### Question 14: How to implement a decorator that validates argument types? (Medium)

**Expected Answer:**
The wrapper inspects `*args` and `**kwargs`, compares them against `func.__annotations__`, and raises `TypeError` if they don't match.

**Explanation:**
It utilizes Python's introspection capabilities.

**Medium-depth reasoning:**
This adds runtime overhead but enforces type safety in critical paths.

**Why interviewers ask this:**
Combines decorators with the typing module and introspection.

---

### Question 15: What is "Memoization" in the context of decorators? (Medium)

**Expected Answer:**
Memoization is an optimization technique where the results of expensive function calls are cached based on their input arguments. A decorator checks a cache (dict) before calling the function.

**Explanation:**
If `args` are in `cache`, return `cache[args]`. Else, run function, store result, return it.

**Medium-depth reasoning:**
Arguments must be hashable (immutable) to be dictionary keys.

**Relevant comparisons:**
`functools.lru_cache` is the standard library implementation.

**Why interviewers ask this:**
Classic algorithm optimization question wrapped in Python features.

---

### Question 16: How do you create a decorator that is optional (can be used with or without brackets)? (Medium)

**Expected Answer:**
You check the arguments passed to the decorator. If the first argument is a callable (the function), it was used without brackets. If not, it was used with brackets/args.

**Explanation:**
This supports both `@log` and `@log(level='DEBUG')`.

**Medium-depth reasoning:**
This requires complex boilerplate involving `functools.partial` or type checking the first argument.

**Sample Code:**

```python
def log(func=None, *, level="INFO"):
    if func is None:
        return partial(log, level=level)
    # ... wrapper logic ...

```

---

### Question 17: Explain `__new__` vs `__init__` in Class Decorators. (Medium)

**Expected Answer:**
When decorating a class, the decorator receives the class object. If you return a *new* class, you might use `__new__`. If you modify the existing class, you just manipulate attributes.

**Why interviewers ask this:**
Deep understanding of Python's object model meta-programming.

---

### Question 18: Can a decorator change the function signature? (Medium)

**Expected Answer:**
Yes. The wrapper can accept different arguments than the original function.

**Explanation:**
For example, a decorator might inject a `user` object based on a generic `user_id` argument, changing the signature seen by the inner logic.

**Medium-depth reasoning:**
However, you should update `__signature__` if you want tools to recognize the change.

---

### Question 19: Performance: What is the overhead of a decorator? (Medium)

**Expected Answer:**
The overhead involves one extra function call (the wrapper) and argument unpacking/repacking.

**Medium-depth reasoning:**
In high-frequency loops (millions of calls), this can accumulate. For standard web requests (latency ~20ms), the microsecond overhead is irrelevant.

**Why interviewers ask this:**
To see if you are a "premature optimizer" or understand pragmatic tradeoffs.

---

### Question 20: How does `property` work as a decorator? (Medium)

**Expected Answer:**
`@property` is a built-in decorator that transforms a method into a read-only attribute getter. It utilizes the Descriptor Protocol (`__get__`, `__set__`).

**Explanation:**
It allows accessing `obj.method` instead of `obj.method()`.

**Why interviewers ask this:**
It is the most common built-in decorator alongside `classmethod` and `staticmethod`.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Coding Question 1: The Execution Timer (Easy)

**Question:**
Write a decorator `@timer` that measures and prints the time a function takes to execute.

**Working Solution:**

```python
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter() # High precision
        result = func(*args, **kwargs)
        end_time = time.perf_counter()
        execution_time = end_time - start_time
        print(f"Function '{func.__name__}' took {execution_time:.4f} seconds")
        return result
    return wrapper

@timer
def heavy_process():
    time.sleep(0.5)

heavy_process()

```

**Explanation:**

* `time.perf_counter()` is used for precision (better than `time.time()`).
* The logic wraps the function call.
* Returns the original result.

**Complexity:**

* Time:  overhead.
* Space: .

**Why interviewers ask this:**
The "Hello World" of decorators.

**Real App Usage:**
Profiling backend endpoints in Django/FastAPI to find bottlenecks.

---

### Coding Question 2: Simple Input Logger (Easy)

**Question:**
Create a decorator that prints the arguments and keyword arguments passed to a function every time it is called.

**Working Solution:**

```python
import functools

def log_args(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"Calling {func.__name__}({signature})")
        return func(*args, **kwargs)
    return wrapper

@log_args
def add(a, b):
    return a + b

add(3, b=4)

```

**Explanation:**

* Converts all args to string representation (`repr`).
* Joins them to mimic a function signature.

**Common Mistakes:**

* Failing to handle both positional and keyword args.

**Real App Usage:**
Debugging production issues by tracing input values in logs.

---

### Coding Question 3: Ensure Admin Access (Easy)

**Question:**
Write a decorator `@requires_admin` that checks a global dictionary `CURRENT_USER`. If `role` is not 'admin', raise a `PermissionError`.

**Working Solution:**

```python
import functools

CURRENT_USER = {"username": "john", "role": "guest"}

def requires_admin(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if CURRENT_USER.get("role") != "admin":
            raise PermissionError("User is not an admin")
        return func(*args, **kwargs)
    return wrapper

@requires_admin
def delete_database():
    print("Database deleted")

try:
    delete_database()
except PermissionError as e:
    print(e)

```

**Explanation:**

* A simple pre-check guard clause.
* Stops execution before the sensitive function runs.

**Developer Scenario:**
Protecting sensitive API endpoints.

---

### Coding Question 4: Retry Mechanism (Medium)

**Question:**
Write a decorator `@retry(times=3, delay=1)` that retries the function if it raises an exception. It should wait `delay` seconds between attempts.

**Working Solution:**

```python
import time
import functools

def retry(times=3, delay=1):
    def decorator_retry(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    print(f"Attempt {attempt + 1} failed: {e}")
                    last_exception = e
                    time.sleep(delay)
            print("All attempts failed.")
            raise last_exception
        return wrapper
    return decorator_retry

@retry(times=2, delay=0.1)
def unstable_network():
    import random
    if random.random() < 0.8:
        raise ConnectionError("Network fail")
    return "Success"

```

**Explanation:**

* **3-level nesting** required because the decorator takes arguments.
* Loop `times` amount of times.
* `try/except` captures errors; `raise` re-throws the last one if all fail.

**Common Mistakes:**

* Catching `Exception` without re-raising it at the end (swallowing errors).

**Real App Usage:**
Retrying failed HTTP requests to external APIs (Stripe, Twilio) or DB connections.

---

### Coding Question 5: In-Memory Cache (Memoization) (Medium)

**Question:**
Implement a `@memoize` decorator from scratch (do not use `functools.lru_cache`). It should cache results based on arguments.

**Working Solution:**

```python
import functools

def memoize(func):
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Create a key. Note: kwargs must be sorted to be consistent
        # For simplicity here, we assume only args are used or kwargs are hashable
        key = (args, frozenset(kwargs.items()))
        
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
        
    return wrapper

@memoize
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

print(fib(30)) # Runs instantly

```

**Explanation:**

* `cache` is a closure variable (persists across calls).
* `key` construction uses `frozenset` because dicts (kwargs) are not hashable.

**Time Complexity:**

*  lookup for cached items.

**Common Mistakes:**

* Trying to use a list or dict as a dictionary key (TypeError: unhashable type).

**Real App Usage:**
Caching expensive configuration loads or heavy mathematical computations.

---

### Coding Question 6: Rate Limiting (Medium)

**Question:**
Write a decorator that limits a function to be called at most once every `X` seconds. If called too soon, return `None` (or raise error).

**Working Solution:**

```python
import time
import functools

def rate_limit(seconds):
    def decorator(func):
        last_called = [0] # Mutable closure hack or use nonlocal
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            elapsed = now - last_called[0]
            if elapsed < seconds:
                print(f"Rate limited. Wait {seconds - elapsed:.2f}s")
                return None
            
            last_called[0] = now
            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(seconds=2)
def api_call():
    print("API Call executed")

api_call() # Runs
api_call() # Printed: Rate limited

```

**Explanation:**

* We store `last_called` timestamp.
* Note: Integers are immutable in Python, so inside `wrapper`, `last_called = now` would create a local variable. We use a list `[0]` or `nonlocal` keyword to modify the closure variable.

**Real App Usage:**
Preventing button spamming in UI or API flooding.

---

### Coding Question 7: Type Enforcement (Medium)

**Question:**
Create `@enforce_types`. If the function signature is `def foo(a: int)`, and `foo("string")` is called, raise `TypeError`.

**Working Solution:**

```python
import functools
import inspect

def enforce_types(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Get signature and bind arguments
        sig = inspect.signature(func)
        bound = sig.bind(*args, **kwargs)
        bound.apply_defaults()
        
        for name, value in bound.arguments.items():
            expected = func.__annotations__.get(name)
            if expected and not isinstance(value, expected):
                raise TypeError(
                    f"Argument '{name}' expected {expected}, got {type(value)}"
                )
        return func(*args, **kwargs)
    return wrapper

@enforce_types
def divide(a: int, b: int):
    return a / b

# divide(10, "2") # Raises TypeError

```

**Explanation:**

* Uses `inspect.signature` to map `*args` to their parameter names correctly.
* Checks `isinstance` against the annotation.

**Common Mistakes:**

* Iterating `zip(args, types)` which fails if kwargs are used. `sig.bind` solves this.

**Real App Usage:**
Data validation layers in strictly typed frameworks.

---

### Coding Question 8: Singleton Class Decorator (Medium)

**Question:**
Write a decorator `@singleton` that ensures a class only has one instance.

**Working Solution:**

```python
def singleton(cls):
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class Database:
    def __init__(self):
        print("Loading DB...")

d1 = Database()
d2 = Database()
print(d1 is d2) # True, "Loading DB..." prints only once

```

**Explanation:**

* The decorator replaces the Class with a function `get_instance`.
* The first time it's called, it creates the object and stores it.
* Subsequent calls return the stored object.

**Real App Usage:**
Database connection pools, Configuration managers.

---

### Coding Question 9: Deprecation Warning (Medium)

**Question:**
Create a decorator `@deprecated` that prints a warning to stderr when the function is called, but still runs the function.

**Working Solution:**

```python
import warnings
import functools

def deprecated(reason=""):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            warnings.warn(
                f"{func.__name__} is deprecated. {reason}",
                category=DeprecationWarning,
                stacklevel=2
            )
            return func(*args, **kwargs)
        return wrapper
    return decorator

@deprecated(reason="Use new_api() instead")
def old_api():
    pass

# old_api()

```

**Explanation:**

* Uses the standard `warnings` module.
* `stacklevel=2` ensures the warning points to the caller, not the wrapper line.

**Real App Usage:**
Library maintenance (Pandas, NumPy versions).

---

### Coding Question 10: Role-Based Access Control (RBAC) (Medium)

**Question:**
Create a decorator factory `@has_permission(permission_name)`. Assume a `User` object is passed as the first argument to the function. Check `User.permissions` list.

**Working Solution:**

```python
import functools

class User:
    def __init__(self, name, perms):
        self.name = name
        self.permissions = perms

def has_permission(required_perm):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # Assume first arg is 'self' or 'user'
            user = args[0] 
            if required_perm not in user.permissions:
                raise PermissionError(f"User misses {required_perm}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@has_permission("write_data")
def save_record(user, data):
    print(f"Saved {data} by {user.name}")

u = User("Alice", ["read_only"])
# save_record(u, "test") # Raises PermissionError

```

**Explanation:**

* Introspects arguments to find the user.
* Typical in Class methods (`self` is args[0]) or simple functions where User is explicitly passed.

**Why interviewers ask this:**
Directly applicable to backend logic (e.g., "Can this user delete this post?").

**Real App Usage:**
Enterprise SaaS applications managing user roles.
