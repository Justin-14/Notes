# Python Types of argument— Interview Guide (30 Questions)

## Table of Contents

1. **Section 1 — Theory Questions** (20 Questions: 6 Easy, 14 Medium)
2. **Section 2 — Coding Questions** (10 Questions: 3 Easy, 7 Medium)

---

# SECTION 1 — THEORY QUESTIONS (20 TOTAL)

## Easy Level (6 Questions)

### 1. Positional vs. Keyword Arguments

**Question:** Explain the fundamental difference between positional arguments and keyword arguments in Python. How does the interpreter distinguish them during a function call?

**Expected Answer:**
Positional arguments are assigned to parameters based on their order (position) in the function call. Keyword arguments are assigned based on the parameter name explicitly specified by the caller (`name=value`). The interpreter first maps positional arguments to parameters in order, then matches keyword arguments.

**Explanation:**
Python functions define parameters in a specific sequence. When calling a function, if you provide values without names, Python assumes they correspond to the parameters in the order defined. If you provide `key=value`, Python looks for a parameter with that specific name, regardless of order (mostly).

**Reasoning:**
Understanding this distinction is crucial for readability and preventing bugs where arguments are swapped accidentally.

**Common Mistakes:**

* Placing positional arguments *after* keyword arguments (SyntaxError).
* Passing the same argument as both positional and keyword (TypeError).

**Why Interviewers Ask This:** To verify basic understanding of function invocation syntax.

**Sample Code:**

```python
def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type} named {pet_name}.")

# Positional
describe_pet('hamster', 'Harry') 

# Keyword
describe_pet(pet_name='Harry', animal_type='hamster')

# Mixed (Valid)
describe_pet('hamster', pet_name='Harry') 

# Mixed (Invalid - raises SyntaxError)
# describe_pet(animal_type='hamster', 'Harry')

```

**Line-by-Line Explanation:**

* `describe_pet` defines two parameters.
* The first call relies on order: 'hamster'  `animal_type`.
* The second call relies on names; order doesn't matter.
* The third call mixes them; positional must come first.

---

### 2. Default Arguments

**Question:** What are default arguments, and how does Python handle them if the caller does not provide a value?

**Expected Answer:**
Default arguments allow a function parameter to assume a preset value if the caller omits the corresponding argument. They are defined using the assignment operator (`=`) in the function signature.

**Explanation:**
If an argument is missing during the call, Python checks if a default was defined. If yes, it uses that value; otherwise, it raises a `TypeError`.

**Reasoning:**
Defaults make functions more flexible and reduce the need for multiple overloaded functions (common in languages like Java).

**Common Mistakes:**

* Defining non-default arguments *after* default arguments in the function definition (SyntaxError).

**Why Interviewers Ask This:** To ensure the candidate knows how to design flexible APIs.

**Sample Code:**

```python
def power(base, exponent=2):
    return base ** exponent

print(power(3))    # Uses default exponent=2 -> 9
print(power(3, 3)) # Overrides default -> 27

```

**Line-by-Line Explanation:**

* `exponent=2` sets the default.
* `power(3)` provides only `base`; `exponent` remains 2.
* `power(3, 3)` provides both; default is ignored.

---

### 3. Arbitrary Positional Arguments (`*args`)

**Question:** What does the syntax `*args` do in a function definition? What is the data type of `args` inside the function?

**Expected Answer:**
`*args` allows a function to accept an arbitrary number of positional arguments. Inside the function, `args` is a **tuple** containing all the extra positional arguments passed during the call.

**Explanation:**
The `*` acts as a gathering operator. It collects any remaining positional arguments that strictly defined parameters haven't caught.

**Reasoning:**
Essential for writing generic wrapper functions or functions where the number of inputs isn't known beforehand (e.g., `sum()`, `max()`).

**Common Mistakes:**

* Assuming `args` is a list (it is immutable).
* Trying to modify `args` in place.

**Why Interviewers Ask This:** To test knowledge of variable-length argument handling.

**Sample Code:**

```python
def sum_all(*args):
    # args is a tuple: (1, 2, 3)
    total = 0
    for num in args:
        total += num
    return total

print(sum_all(1, 2, 3))

```

**Line-by-Line Explanation:**

* `*args` collects `1, 2, 3` into a tuple `(1, 2, 3)`.
* The loop iterates over this tuple.

---

### 4. Arbitrary Keyword Arguments (`**kwargs`)

**Question:** What does `**kwargs` do? How does it differ from `*args`?

**Expected Answer:**
`**kwargs` allows a function to accept an arbitrary number of **keyword** arguments. Inside the function, `kwargs` is a **dictionary** where keys are argument names (strings) and values are the passed values.

**Explanation:**
The `**` operator gathers any keyword arguments that do not match existing parameter names.

**Reasoning:**
Crucial for configuration functions, ORMs, or passing parameters up to superclasses in inheritance (`super().__init__(**kwargs)`).

**Common Mistakes:**

* Using `*kwargs` (single asterisk) for keywords.
* Assuming the order of `kwargs` was preserved (historically not true, though preserved in Python 3.7+).

**Why Interviewers Ask This:** To check understanding of dictionary packing in function headers.

**Sample Code:**

```python
def build_profile(**kwargs):
    # kwargs is dict: {'name': 'John', 'age': 30}
    for key, value in kwargs.items():
        print(f"{key}: {value}")

build_profile(name="John", age=30)

```

**Line-by-Line Explanation:**

* `**kwargs` captures `name="John", age=30`.
* It converts them into `{'name': 'John', 'age': 30}`.

---

### 5. Argument Unpacking

**Question:** You have a list `numbers = [1, 2, 3]`. How do you pass these items as individual arguments to a function expecting three separate parameters?

**Expected Answer:**
Use the unpacking operator `*`. Calling `func(*numbers)` unpacks the list elements into individual positional arguments.

**Explanation:**
This is the inverse of `*args`. `*args` packs multiple arguments into a tuple; calling with `*list` unpacks a sequence into multiple arguments.

**Reasoning:**
Avoids manual extraction like `func(numbers[0], numbers[1], numbers[2])`.

**Common Mistakes:**

* Using `**` for lists (only works for dicts).
* Unpacking a list that doesn't match the required argument count (TypeError).

**Why Interviewers Ask This:** To test familiarity with Python's syntactic sugar for cleaner code.

**Sample Code:**

```python
def multiply(a, b, c):
    return a * b * c

nums = [2, 3, 4]
print(multiply(*nums)) # Equivalent to multiply(2, 3, 4)

```

**Line-by-Line Explanation:**

* `*nums` explodes the list `[2, 3, 4]`.
* `a` gets 2, `b` gets 3, `c` gets 4.

---

### 6. Passing by Object Reference

**Question:** Does Python pass arguments by value or by reference? Explain the implication for mutable types like lists.

**Expected Answer:**
Python uses "Call by Object Reference" (or "Pass by Assignment"). The function receives a reference to the same object in memory. If the object is mutable (like a list) and modified inside the function, the change is reflected outside. If the variable is reassigned inside, the external variable is unaffected.

**Explanation:**
Variables are just names bound to objects. Passing an argument binds the function parameter name to the existing object.

**Reasoning:**
This is the single most common source of bugs for beginners transitioning from C++ or Java.

**Common Mistakes:**

* Thinking Python is purely "Pass by Reference" (reassignment doesn't work like pointers).
* Thinking Python is "Pass by Value" (modifying a list inside changes it outside).

**Why Interviewers Ask This:** Critical for understanding side effects.

**Sample Code:**

```python
def modify_list(lst):
    lst.append(4) # Mutates original object

def reassign_list(lst):
    lst = [100] # Rebinds local name; original untouched

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list) # [1, 2, 3, 4]

reassign_list(my_list)
print(my_list) # [1, 2, 3, 4]

```

---

## Medium Level (14 Questions)

### 7. The Mutable Default Argument "Gotcha"

**Question:** Analyze the code below. What is the output of the second call and why?

```python
def append_item(item, list_arg=[]):
    list_arg.append(item)
    return list_arg

print(append_item(1))
print(append_item(2))

```

**Expected Answer:**
Output:
`[1]`
`[1, 2]`
Reason: Default argument values are evaluated **only once** at function definition time, not every time the function is called. The `list_arg` refers to the same list object across all calls where the argument is omitted.

**Explanation:**
When Python defines the function, it creates the list object `[]` and stores it. Subsequent calls reuse this exact object.

**Reasoning:**
This behavior is efficient but dangerous if unintended.

**Common Mistakes:**

* Expecting a new empty list every time.
* Fixing it by checking `if list_arg == []` (wrong logic). Correct fix is `if list_arg is None: list_arg = []`.

**Why Interviewers Ask This:** Classic Python interview question to weed out those who haven't encountered this specific quirk.

**Sample Code (Fix):**

```python
def append_item_safe(item, list_arg=None):
    if list_arg is None:
        list_arg = []
    list_arg.append(item)
    return list_arg

```

---

### 8. Keyword-Only Arguments

**Question:** How do you force a caller to use specific arguments *only* as keywords (preventing positional usage)?

**Expected Answer:**
Place an asterisk `*` in the parameter list. All parameters defined after the `*` must be passed as keyword arguments.

**Explanation:**
This explicitly separates positional arguments from configuration-style arguments that should be named for clarity.

**Reasoning:**
Improves readability for functions with many boolean flags or options (e.g., `sort(reverse=True)`).

**Common Mistakes:**

* Forgetting to pass the argument as a keyword (raises `TypeError`).

**Why Interviewers Ask This:** Shows knowledge of modern Python (3.x) API design features.

**Sample Code:**

```python
def transfer_money(amount, *, sender, receiver):
    print(f"Transfer {amount} from {sender} to {receiver}")

transfer_money(100, sender="Alice", receiver="Bob") # Valid
# transfer_money(100, "Alice", "Bob") # TypeError: takes 1 positional arg

```

**Line-by-Line Explanation:**

* `*` acts as a barrier.
* `amount` can be positional.
* `sender` and `receiver` MUST be keywords.

---

### 9. Positional-Only Arguments (`/`)

**Question:** Introduced in Python 3.8, what does the forward slash `/` in a function signature denote?

**Expected Answer:**
The `/` denotes that all parameters prior to it are **positional-only**. They cannot be passed as keyword arguments.

**Explanation:**
This is useful when parameter names don't matter or might change in the future, or to enforce strictly positional APIs (like C functions).

**Reasoning:**
It allows changing the internal parameter name without breaking client code that might have used `name=value`.

**Common Mistakes:**

* Trying to use `name=val` for a parameter before `/`.

**Why Interviewers Ask This:** Checks if the candidate keeps up with newer language features.

**Sample Code:**

```python
def strict_pos(x, y, /):
    return x + y

print(strict_pos(10, 20)) # Valid
# print(strict_pos(x=10, y=20)) # TypeError

```

---

### 10. Order of Parameters

**Question:** What is the mandatory order for defining parameters in a function signature if you use all types (positional, default, `*args`, keyword-only, `**kwargs`)?

**Expected Answer:**
The order is strictly:

1. Positional (standard)
2. Default arguments (standard)
3. `*args` (Variable positional)
4. Keyword-only arguments
5. `**kwargs` (Variable keywords)

**Diagram:**

```text
def func( pos,  def=1,  *args,  kw_only,  **kwargs )
          |      |       |        |          |
      Standard  Default  Tuple  Specific    Dict

```

**Explanation:**
If this order is violated, Python raises a `SyntaxError`.

**Reasoning:**
The parser needs a deterministic way to map values to names. `*args` consumes leftover positionals, so keyword-only args must follow it. `**kwargs` consumes everything else, so it must be last.

**Why Interviewers Ask This:** To test deep syntax knowledge.

---

### 11. Argument Binding & Scope (locals)

**Question:** How can you dynamically access all arguments passed to a function as a dictionary *without* defining `**kwargs`?

**Expected Answer:**
You can use `locals()` at the very beginning of the function call, or more robustly, use `inspect.signature` and `bind`.

**Explanation:**
`locals()` returns a dictionary of the current local symbol table. If called first thing, it contains arguments.

**Reasoning:**
Useful for logging or debugging wrappers.

**Common Mistakes:**

* Modifying the dict returned by `locals()` and expecting the local variables to update (behavior is undefined/implementation dependent).

**Sample Code:**

```python
def debug_me(a, b):
    args = locals()
    print(f"Called with: {args}")
    return a + b

debug_me(1, 2) # Called with: {'a': 1, 'b': 2}

```

---

### 12. Function Annotations (Type Hinting)

**Question:** How do you type-hint `*args` and `**kwargs`? If `*args` accepts integers, do you hint it as `list[int]` or `int`?

**Expected Answer:**
You hint the type of the **individual element**, not the container.

* For `*args` holding integers, use `*args: int`.
* For `**kwargs` holding integer values, use `**kwargs: int`.

**Explanation:**
Type checkers (like MyPy) infer that `args` is a `tuple[int, ...]` and `kwargs` is `dict[str, int]` automatically.

**Common Mistakes:**

* Hinting `*args: tuple[int]` (implies `args` is a tuple of tuples).

**Why Interviewers Ask This:** Verifies experience with modern production codebases using static analysis.

**Sample Code:**

```python
def sum_integers(*args: int) -> int:
    return sum(args)

```

---

### 13. Forwarding Arguments

**Question:** You are writing a decorator. How do you ensure the wrapper function accepts ANY combination of arguments that the decorated function might accept?

**Expected Answer:**
Define the wrapper with `(*args, **kwargs)` and call the original function with `(*args, **kwargs)`.

**Explanation:**
This captures all positional and keyword arguments and passes them through unchanged.

**Reasoning:**
This makes the decorator "transparent" regarding signature.

**Sample Code:**

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print("Log: Function called")
        return func(*args, **kwargs)
    return wrapper

```

---

### 14. Unpacking Dictionaries

**Question:** What happens if you pass a dictionary to a function using `**my_dict`, but the dictionary contains a key that does not match any parameter name in the function (and the function has no `**kwargs`)?

**Expected Answer:**
Python raises a `TypeError`: "got an unexpected keyword argument...".

**Explanation:**
The unpacking mechanism attempts to match keys to parameter names. Unmatched keys are fatal errors unless `**kwargs` is present to catch them.

**Reasoning:**
Ensures data integrity; you aren't passing data that gets silently ignored.

**Why Interviewers Ask This:** Tests knowledge of runtime error conditions in argument passing.

---

### 15. The `inspect` Module

**Question:** How can you programmatically determine which arguments a function requires at runtime?

**Expected Answer:**
Use `inspect.signature(func)`. It returns a `Signature` object containing parameters and their kinds (positional, keyword, etc.).

**Explanation:**
This is safer than parsing `func.__code__`.

**Scenario:**
Building a web framework where you map URL parameters to function arguments dynamically.

**Sample Code:**

```python
import inspect

def foo(a, b=1): pass

sig = inspect.signature(foo)
print(sig.parameters['a']) # a
print(sig.parameters['b'].default) # 1

```

---

### 16. Partial Functions

**Question:** What is `functools.partial`, and how does it relate to function arguments?

**Expected Answer:**
`partial` allows you to "freeze" some portion of a function's arguments and keywords, resulting in a new object with a simplified signature.

**Explanation:**
It creates a callable that pre-fills arguments.

**Reasoning:**
Useful in callbacks (e.g., UI event handlers) where you can't pass arguments directly at the moment of the call.

**Sample Code:**

```python
from functools import partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
print(square(10)) # 100

```

---

### 17. None as Default vs. Sentinel Objects

**Question:** Why might you use a custom sentinel object (like `object()`) instead of `None` as a default argument value?

**Expected Answer:**
If `None` is a valid value for the argument, you cannot use `if arg is None` to detect if the user omitted the argument. A unique object instance guarantees uniqueness.

**Explanation:**
`sentinel = object()` creates a unique reference in memory.

**Sample Code:**

```python
_MISSING = object()

def connect(timeout=_MISSING):
    if timeout is _MISSING:
        print("Using system default timeout")
    elif timeout is None:
        print("Timeout explicitly disabled (infinite)")
    else:
        print(f"Timeout: {timeout}")

```

---

### 18. Lambda Arguments

**Question:** Can lambda functions accept `*args` and `**kwargs`?

**Expected Answer:**
Yes, lambdas support the exact same argument syntax as standard `def` functions, including default values, `*args`, and `**kwargs`.

**Sample Code:**

```python
func = lambda *args: sum(args)
print(func(1, 2, 3)) # 6

```

---

### 19. Argument Limit

**Question:** Is there a limit to the number of arguments you can pass to a Python function?

**Expected Answer:**
Practically, it depends on the implementation (CPython) and stack size. Before Python 3.7, there was a limit of 255 arguments with explicit names. In modern Python 3.7+, the limit is much higher (INT_MAX), but passing thousands of arguments suggests a bad design (use a data structure instead).

**Why Interviewers Ask This:** Trivia/Edge case knowledge.

---

### 20. Side Effects in Default Arguments

**Question:** If a default argument is a function call (e.g., `def foo(x=calculate_now()):`), when is that function called?

**Expected Answer:**
It is called **once**, at the time the `def` statement is executed (module load time). It is NOT called every time `foo` runs.

**Explanation:**
This is a variation of the mutable default argument issue.

**Reasoning:**
If you want the time to be dynamic (e.g., `datetime.now()`), you must use `None` as default and calculate inside the body.

**Sample Code:**

```python
def log_time(t=print("Default Evaluated")): 
    pass
# "Default Evaluated" prints immediately when script loads, not when log_time is called.

```

---

# SECTION 2 — CODING QUESTIONS (10 TOTAL)

## Easy Level (3 Questions)

### 21. Sum of Arbitrary Arguments

**Question:**
Write a function `universal_sum` that accepts any number of numerical arguments and returns their sum. If no arguments are passed, return 0.

**Solution:**

```python
def universal_sum(*numbers: float) -> float:
    total = 0
    for num in numbers:
        total += num
    return total

# Usage
print(universal_sum(10, 20, 30)) # 60
print(universal_sum())           # 0

```

**Line-by-Line Explanation:**

1. `*numbers` gathers all inputs into a tuple.
2. We initialize `total` to 0.
3. Iterate through the tuple and accumulate values.
4. Return the result.
*(Note: Python's built-in `sum()` works on iterables, but `sum(numbers)` works here because `numbers` is a tuple).*

**Time Complexity:**  where  is number of args.
**Space Complexity:**  to store the tuple of args.
**Real App Usage:** A metrics aggregation function in a monitoring tool.

---

### 22. Formatting User Data

**Question:**
Write a function `format_user` that accepts mandatory `name` and `email`, and any number of extra details as keyword arguments. Return a formatted string.

**Solution:**

```python
def format_user(name: str, email: str, **details) -> str:
    base_info = f"User: {name} ({email})"
    extra_info = ", ".join(f"{k}={v}" for k, v in details.items())
    
    if extra_info:
        return f"{base_info} | Details: {extra_info}"
    return base_info

# Usage
print(format_user("Alice", "a@x.com", age=25, city="NY"))

```

**Line-by-Line Explanation:**

1. `name` and `email` are standard positional args.
2. `**details` captures `age` and `city` into a dict.
3. We loop over `details.items()` to create a string representation.
4. Join the base info with extra info.

**Why Interviewers Ask This:** Text processing + handling variable attributes is common in logging.

---

### 23. Safe List Appender

**Question:**
Write a function `add_to_store` that accepts a value and a list. If the list is not provided, create a new one. Ensure no "mutable default argument" bugs exist.

**Solution:**

```python
def add_to_store(value, store=None):
    if store is None:
        store = []
    store.append(value)
    return store

# Usage
list1 = add_to_store("apple")
list2 = add_to_store("banana") # Should be a NEW list, not containing apple

```

**Explanation:**
Using `None` as the sentinel ensures a new list is created for every call where `store` is omitted.

**Time Complexity:**  (amortized append).

---

## Medium Level (7 Questions)

### 24. Validating Keyword Arguments (Allow-list)

**Question:**
Write a function `connect_db` that accepts `host`, `port`, and `**options`.

* Allowed keys in `options` are only `timeout` and `ssl`.
* If a user passes any other keyword in `options`, raise a `ValueError` listing the invalid keys.

**Solution:**

```python
def connect_db(host, port, **options):
    ALLOWED_KEYS = {'timeout', 'ssl'}
    
    # Check for invalid keys
    invalid_keys = set(options.keys()) - ALLOWED_KEYS
    if invalid_keys:
        raise ValueError(f"Invalid arguments: {invalid_keys}")
    
    print(f"Connecting to {host}:{port} with {options}")

# Usage
# connect_db("localhost", 5432, timeout=5, user="admin") # Raises ValueError

```

**Line-by-Line Explanation:**

1. Define a set `ALLOWED_KEYS`.
2. Get keys from `options` (the `kwargs` dict).
3. Use set difference `-` to find keys present in `options` but not in allowed list.
4. Raise error if set is not empty.

**Scenario:** Strict API configuration (e.g., database drivers).

---

### 25. The "Print" Proxy

**Question:**
Implement a function `custom_print` that mimics Python's built-in `print` but adds a timestamp to the start of the line. It must support `*args` (for multiple items) and `**kwargs` (for `sep`, `end`, `file`).

**Solution:**

```python
import time
import builtins

def custom_print(*args, **kwargs):
    timestamp = time.strftime("[%H:%M:%S]")
    
    # Check if 'sep' is provided in kwargs, default to space
    sep = kwargs.get('sep', ' ')
    
    # Create the message string manually to prepend timestamp correctly
    message = sep.join(map(str, args))
    
    # We remove 'sep' from kwargs to avoid conflict if we passed it to message construction
    # But builtins.print needs it if we passed args directly.
    # Approach: Print timestamp + separator, then let built-in print handle the rest
    
    builtins.print(timestamp, *args, **kwargs)

# Usage
custom_print("Server started", "Port 80", sep=" - ")
# Output: [10:00:00] - Server started - Port 80

```

**Line-by-Line Explanation:**

1. Accepts `*args` (objects to print) and `**kwargs` (config).
2. Generates timestamp.
3. Calls `builtins.print` passing the timestamp as the first argument, followed by the unpacked `*args`.
4. Passes `**kwargs` forward so `end="\r"` or `file=sys.stderr` works.

**Why Interviewers Ask This:** Tests argument forwarding and wrapper logic.

---

### 26. Decorator to Enforce Types

**Question:**
Write a decorator `enforce_types` that checks if the passed arguments match the type hints defined in the function signature. If not, raise `TypeError`. (Handle simple positional args only for simplicity).

**Solution:**

```python
import inspect
import functools

def enforce_types(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        sig = inspect.signature(func)
        bound = sig.bind(*args, **kwargs)
        
        for name, value in bound.arguments.items():
            param = sig.parameters[name]
            expected_type = param.annotation
            
            if expected_type is inspect.Parameter.empty:
                continue
                
            if not isinstance(value, expected_type):
                raise TypeError(
                    f"Argument '{name}' expected {expected_type}, got {type(value)}"
                )
        
        return func(*args, **kwargs)
    return wrapper

@enforce_types
def add(x: int, y: int):
    return x + y

# add(1, "2") # Raises TypeError

```

**Line-by-Line Explanation:**

1. `inspect.signature` gets the type hints.
2. `sig.bind(*args, **kwargs)` maps the incoming values to parameter names.
3. Loop through bound arguments.
4. Check `isinstance`. Raise error on mismatch.

**Developer Scenario:** Runtime type checking in critical financial data processing.

---

### 27. Configuration Merger (Defaults < Config File < CLI Args)

**Question:**
Write a function `get_config` that takes:

1. `default_config` (dict)
2. `**overrides` (kwargs)
It should return a new dictionary where `overrides` update `default_config`. The original `default_config` must NOT be modified.

**Solution:**

```python
def get_config(default_config: dict, **overrides) -> dict:
    # Create a shallow copy to avoid side effects
    final_config = default_config.copy()
    
    # Update with keyword arguments
    final_config.update(overrides)
    
    return final_config

# Usage
defaults = {'theme': 'dark', 'notifications': True}
user_settings = get_config(defaults, theme='light', volume=50)
# user_settings -> {'theme': 'light', 'notifications': True, 'volume': 50}
# defaults -> Unchanged

```

**Time Complexity:**  where N is default size, M is overrides size.
**Why Interviewers Ask This:** Tests understanding of mutability and dictionary merging (`update`).

---

### 28. Map with Arguments

**Question:**
Python's `map(func, iterable)` applies `func` to items. Write `map_with_args(func, iterable, *args, **kwargs)` that applies `func(item, *args, **kwargs)` to every item in iterable.

**Solution:**

```python
def map_with_args(func, iterable, *args, **kwargs):
    results = []
    for item in iterable:
        # Apply func to item, plus the fixed extra args
        val = func(item, *args, **kwargs)
        results.append(val)
    return results

def scaler(x, factor, offset=0):
    return (x * factor) + offset

data = [1, 2, 3]
# Multiply by 10, add 5
processed = map_with_args(scaler, data, 10, offset=5)
print(processed) # [15, 25, 35]

```

**Scenario:** Data pipelines where a transformation function needs constant configuration parameters (like a scaling factor) applied to a stream of data.

---

### 29. Partial Application Implementation

**Question:**
Recreate `functools.partial` functionality manually as a function `my_partial(func, *fixed_args, **fixed_kwargs)`.

**Solution:**

```python
def my_partial(func, *fixed_args, **fixed_kwargs):
    def wrapper(*args, **kwargs):
        # Combine fixed args with new args
        # Note: fixed_args come FIRST in positionals
        all_args = fixed_args + args
        
        # Combine dictionaries (new kwargs override fixed ones usually, 
        # but here we merge them. Standard partial prefers new kwargs)
        all_kwargs = fixed_kwargs.copy()
        all_kwargs.update(kwargs)
        
        return func(*all_args, **all_kwargs)
    return wrapper

def greet(greeting, name):
    return f"{greeting}, {name}"

say_hello = my_partial(greet, "Hello")
print(say_hello("World")) # Hello, World

```

**Common Mistakes:**

* Getting the order of `all_args` wrong (`args + fixed_args` changes semantics).

---

### 30. Memoization with Varying Arguments

**Question:**
Write a memoization decorator that works for functions taking any combination of arguments. (Assume all arguments are hashable).

**Solution:**

```python
import functools

def memoize(func):
    cache = {}
    
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # Create a unique key for the arguments
        # kwargs must be sorted to ensure consistent key ('a':1, 'b':2) vs ('b':2, 'a':1)
        key = (args, tuple(sorted(kwargs.items())))
        
        if key not in cache:
            cache[key] = func(*args, **kwargs)
            
        return cache[key]
    return wrapper

@memoize
def slow_add(a, b):
    print("Calculating...")
    return a + b

print(slow_add(1, 2)) # Prints "Calculating...", returns 3
print(slow_add(1, 2)) # Returns 3 (from cache)

```

**Space Complexity:** Depends on input variety.
**Real App Usage:** caching API responses or recursive function results (Dynamic Programming).
