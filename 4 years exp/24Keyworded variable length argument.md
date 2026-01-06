# Python Keyworded variable length argument— Interview Guide (30 Questions)

This guide focuses on mastering Python's variable length arguments, specifically **keyworded variable length arguments (`**kwargs`)**, alongside standard variable arguments (`*args`). It is tailored for a developer with **4 years of experience**, covering syntax, internal handling, unpacking, and advanced patterns like decorators and composition.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### 1. What does `**kwargs` stand for, and how does the Python interpreter handle it internally?

* **Difficulty:** Easy
* **Expected Answer:** `**kwargs` stands for "keyword arguments" (the name `kwargs` is a convention; the `**` is the operator). Internally, Python collects all extra keyword arguments passed to the function that do not match a formal parameter and packs them into a standard Python **dictionary**.
* **Explanation:** When a function is called, Python matches positional arguments first, then named keyword arguments. Any remaining keyword arguments are bundled into a new dictionary assigned to the local variable name defined with `**`.
* **Reasoning:** Tests understanding that `**kwargs` is just a dictionary inside the function scope.
* **Time & Space Complexity:**  to construct the dictionary, where N is the number of extra arguments. Space is  to store them.
* **Common Mistakes:** Thinking `kwargs` is a special object type; it is just a `dict`.
* **Why Interviewers Ask:** To verify foundational understanding of argument parsing.
* **Sample Code:**
```python
def inspect_internals(**kwargs):
    print(f"Type: {type(kwargs)}")
    print(f"Data: {kwargs}")

inspect_internals(status="active", retries=3)

```


* **Line-by-line:**
* `def inspect_internals(**kwargs):`: Defines a function accepting arbitrary keyword args.
* `type(kwargs)`: Confirms it is of class `<class 'dict'>`.



### 2. What is the mandatory order of parameters in a function definition when using positional, `*args`, default, and `**kwargs`?

* **Difficulty:** Easy
* **Expected Answer:** The order must be:
1. Standard (Positional) parameters
2. `*args` (Variable positional)
3. Default parameters (technically often mixed, but after `*args` usually become keyword-only)
4. `**kwargs` (Variable keyword)


* `**kwargs` **must** always be the last parameter.


* **Explanation:** Python's parser requires `**kwargs` to be last because it consumes *all* remaining keyword arguments. Placing anything after it would make those parameters unreachable.
* **Reasoning:** Syntax strictness is crucial for defining clean APIs.
* **Common Mistakes:** Placing `**kwargs` before `*args` or before default parameters.
* **Why Interviewers Ask:** Basic syntax filter question.
* **Sample Code:**
```python
# Correct
def func(a, b, *args, default_val=10, **kwargs):
    pass

# SyntaxError
# def func(a, **kwargs, *args): pass 

```



### 3. How do you unpack a dictionary into a function call?

* **Difficulty:** Easy
* **Expected Answer:** Use the double asterisk operator `**` before the dictionary object in the function call.
* **Explanation:** This "explodes" the dictionary keys and values into separate keyword arguments (`key=value`) passed to the function. Keys must be strings.
* **Reasoning:** This is the inverse of defining `**kwargs` in a function signature.
* **Common Mistakes:** Using `*` (single asterisk), which only unpacks keys as positional arguments.
* **Why Interviewers Ask:** Essential for wrapper functions and dynamic configurations.
* **Sample Code:**
```python
def connect(host, port):
    print(f"Connecting to {host}:{port}")

config = {'host': 'localhost', 'port': 8080}
connect(**config)

```



### 4. Can `**kwargs` keys be non-string types?

* **Difficulty:** Easy
* **Expected Answer:** No. When calling a function using the `func(key=value)` syntax, keys are identifiers (strings). When unpacking a dict using `**d`, Python explicitly checks that all keys in `d` are strings.
* **Explanation:** Python keywords must be valid identifiers. Therefore, you cannot pass an integer or object as a keyword argument name.
* **Reasoning:** Distinguishes between a general dictionary (which allows any hashable key) and `**kwargs` (which requires string keys).
* **Common Mistakes:** Assuming any dict can be unpacked into `**kwargs`.
* **Why Interviewers Ask:** Tests knowledge of language constraints regarding identifiers.
* **Sample Code:**
```python
def my_func(**kwargs): pass

data = {1: 'a', 'b': 2}
# my_func(**data)  # Raises TypeError: keywords must be strings

```



### 5. What happens if you pass a keyword argument that already exists as a positional argument while also using `**kwargs`?

* **Difficulty:** Easy
* **Expected Answer:** Python raises a `TypeError`: `got multiple values for argument 'x'`.
* **Explanation:** The interpreter assigns positional arguments first. If a named argument tries to overwrite a variable that was already assigned via position, it errors out before populating `**kwargs`.
* **Reasoning:** Checks understanding of the argument parsing precedence.
* **Common Mistakes:** Assuming the duplicate will just slide into `**kwargs`.
* **Why Interviewers Ask:** Debugging scenario readiness.
* **Sample Code:**
```python
def process(id, **kwargs): pass

# TypeError
# process(10, id=20) 

```



### 6. How do `*args` and `**kwargs` allow for function decorators to be "transparent"?

* **Difficulty:** Easy
* **Expected Answer:** They allow a wrapper function to accept *any* arguments and forward them exactly as received to the wrapped function.
* **Explanation:** A decorator defining `wrapper(*args, **kwargs)` captures all inputs. Calling `original_func(*args, **kwargs)` inside replays those inputs exactly.
* **Reasoning:** This is the primary use case for `**kwargs` in advanced Python programming.
* **Why Interviewers Ask:** Decorators are a staple of 4-year+ Python development.
* **Sample Code:**
```python
def log(func):
    def wrapper(*args, **kwargs):
        print("Call...")
        return func(*args, **kwargs)
    return wrapper

```



### 7. Explain the performance overhead of using `**kwargs` versus explicit named parameters.

* **Difficulty:** Medium
* **Expected Answer:** `**kwargs` involves the creation of a dictionary object and populating it with values at runtime. Explicit named parameters are optimized (often accessing slots/arrays in C struct locals).
* **Explanation:** While the difference is negligible for IO-bound apps, in tight loops, the overhead of allocating a dictionary and hashing keys makes `**kwargs` slower than fixed arguments.
* **Reasoning:** Performance awareness is key for a Senior/Mid-level role.
* **Time & Space Complexity:** `**kwargs` adds dict allocation overhead.
* **Why Interviewers Ask:** To see if you default to "magic" flexible args everywhere or use them judiciously.

### 8. How do "Keyword-Only Arguments" relate to `**kwargs`?

* **Difficulty:** Medium
* **Expected Answer:** Keyword-only arguments are defined after `*args` or a bare `*` in the signature. They *must* be passed by name. They are distinct from `**kwargs` because they are explicitly named variables, whereas `**kwargs` captures "everything else".
* **Explanation:** If you have `def f(*, a, **kwargs)`, `a` is a specific required keyword. `kwargs` catches any keywords *other than* `a`.
* **Reasoning:** Tests precision in API definition.
* **Sample Code:**
```python
def transfer(amount, *, param_check=False, **kwargs):
    # param_check MUST be named
    # kwargs catches extra metadata
    pass

```



### 9. How would you merge two dictionaries into `**kwargs` of a function call in Python 3.5 vs 3.12?

* **Difficulty:** Medium
* **Expected Answer:**
* **Pre-3.5:** You had to merge them beforehand: `d = d1.copy(); d.update(d2); func(**d)`.
* **3.5+:** You can unpack multiple times: `func(**d1, **d2)`.


* **Explanation:** Python allows multiple unpackings in a single call. If keys overlap, the later one (`d2`) overwrites the earlier one (`d1`).
* **Reasoning:** Checks knowledge of modern Python syntax evolution.
* **Sample Code:**
```python
default = {'timeout': 10}
custom = {'timeout': 30, 'scope': 'local'}

def request(**kwargs):
    print(kwargs)

request(**default, **custom)
# Output: {'timeout': 30, 'scope': 'local'}

```



### 10. In the context of Class Inheritance, why is `super().__init__(**kwargs)` commonly considered a best practice in Mixins?

* **Difficulty:** Medium
* **Expected Answer:** It allows for "Cooperative Multiple Inheritance". Each class in the MRO (Method Resolution Order) strips off the arguments it needs and passes the rest up the chain via `**kwargs`.
* **Explanation:** Without `**kwargs`, every class in the hierarchy would need to know the exact signature of the parent, which breaks the decoupling of Mixins.
* **Reasoning:** Essential for designing complex class hierarchies (e.g., Django views, DRF).
* **Common Mistakes:** Forgetting to pop arguments before calling super, leading to "unexpected argument" errors in the base `object` class.
* **Why Interviewers Ask:** OOP architecture knowledge.

### 11. Can you modify the `**kwargs` dictionary inside a function? Does it affect the caller?

* **Difficulty:** Medium
* **Expected Answer:** Yes, you can modify it inside because it is a local dictionary. No, it does not affect the caller's variables because `**kwargs` is created freshly for the function call (pass-by-value semantics for the reference to the new dict).
* **Explanation:** Even if the caller passed a dict via `**my_dict`, the function receives a shallow copy of the unpacking, not a reference to `my_dict` itself.
* **Reasoning:** Understanding scope and mutability.
* **ASCII Diagram:**
```text
Caller: {a:1} (Unpacked) --> [Value Copy] --> Function: kwargs = {a:1}
                                                  |
(Modifying kwargs here does NOT touch Caller's dict)

```



### 12. How do you type hint `**kwargs` in Python 3.12?

* **Difficulty:** Medium
* **Expected Answer:** You annotate the *values* contained in the dict. `def func(**kwargs: int)` means all values in `kwargs` are integers.
* **Explanation:** You do NOT write `Dict[str, int]`. The syntax implies the type of the value. If values are mixed, use `Any` or a `Union`. Modern Python (`Unpack` from `typing`) allows stricter TypedDict unpacking.
* **Reasoning:** Modern Python relies heavily on Type Hints/Mypy.
* **Sample Code:**
```python
from typing import Any
def configure(**kwargs: Any) -> None:
    pass

```



### 13. What is the security risk of passing `**kwargs` directly to an SQL query or shell command?

* **Difficulty:** Medium
* **Expected Answer:** SQL Injection or Command Injection.
* **Explanation:** If `kwargs` are used to dynamically construct strings (e.g., `f"params={kwargs['val']}"`), a malicious user can inject code.
* **Reasoning:** Security awareness. You should never unpack raw user input into sensitive execution contexts.
* **Why Interviewers Ask:** To see if you think beyond code syntax to system safety.

### 14. What is `Unpack[TypedDict]` in Python 3.12 `typing` and how does it improve `**kwargs`?

* **Difficulty:** Medium
* **Expected Answer:** It allows you to specify exactly which keys and value types are expected in `**kwargs`, providing static type checking for dynamic keyword arguments.
* **Explanation:** Historically, `**kwargs` was a "black box" to type checkers. `Unpack` allows defining a `TypedDict` schema that `**kwargs` must adhere to.
* **Reasoning:** Knowledge of very recent Python features (PEP 692).
* **Sample Code:**
```python
from typing import TypedDict, Unpack

class Options(TypedDict):
    verbose: bool
    retries: int

def connect(**kwargs: Unpack[Options]):
    pass

```



### 15. How does `functools.partial` interact with `**kwargs`?

* **Difficulty:** Medium
* **Expected Answer:** `partial` "freezes" specific keyword arguments. When the partial is called later, new `**kwargs` are merged. If keys collide, the new arguments provided at call time overwrite the frozen ones.
* **Explanation:** `partial` creates a new callable with pre-filled slots.
* **Reasoning:** Functional programming patterns in Python.
* **Sample Code:**
```python
from functools import partial
def log(msg, level="INFO"): print(level, msg)

error_log = partial(log, level="ERROR")
error_log("Failed") # Output: ERROR Failed
error_log("Fixed", level="DEBUG") # Output: DEBUG Fixed (Overwrites)

```



### 16. What is the difference between `def f(a=None)` and `def f(**kwargs)` when handling optional parameters?

* **Difficulty:** Medium
* **Expected Answer:**
* `a=None`: The parameter `a` is explicitly defined in the function signature (API contract). Tools (IDEs, introspection) see it.
* `**kwargs`: The parameter is hidden/implicit.


* **Explanation:** explicit arguments are self-documenting. `**kwargs` should only be used when the set of arguments is truly open-ended or forwarded elsewhere.
* **Reasoning:** API Design philosophy. "Explicit is better than implicit" (Zen of Python).

### 17. How can `**kwargs` facilitate "Data Transfer Objects" (DTO) or serialization from JSON?

* **Difficulty:** Medium
* **Expected Answer:** You can instantiate a class directly from a JSON object (dict) using `cls(**json_dict)`, provided the keys match the `__init__` arguments.
* **Explanation:** This is a common pattern in ORMs (like SQLAlchemy) or Pydantic (though Pydantic adds validation).
* **Reasoning:** Practical backend data handling.
* **Sample Code:**
```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

data = {"name": "Alice", "age": 30}
u = User(**data)

```



### 18. What happens if you define a function with same-named arguments in `**kwargs` unpacking? E.g. `func(**{'a':1}, **{'a':2})`?

* **Difficulty:** Medium
* **Expected Answer:** Python prohibits duplicate keyword arguments in the *call*. It raises `TypeError: type() got multiple values for keyword argument 'a'`.
* **Explanation:** While `func(**a, **b)` works by merging, if both `a` and `b` have the same key, Python 3.5+ allows the second to overwrite the first. However, if you mix explicit keywords `func(a=1, **{'a':2})`, it fails.
* **Correction/Nuance:**
* `func(**{'a':1}, **{'a':2})` -> **Valid** in Python 3.5+ (Last wins).
* `func(a=1, **{'a':2})` -> **Error** (Multiple values).


* **Reasoning:** Edge case syntax rules.

### 19. Can `**kwargs` capture arguments meant for positional-only parameters (Python 3.8+)?

* **Difficulty:** Medium
* **Expected Answer:** No. Positional-only parameters (defined before `/`) cannot be passed by name. If you try to pass them via `**kwargs`, Python treats them as "unexpected keyword arguments" unless the function *also* accepts `**kwargs`, in which case they end up inside the `kwargs` dict, failing to bind to the positional argument.
* **Explanation:** `def f(p, /, **kwargs)`: calling `f(p=1)` puts `p=1` into `kwargs` and leaves the positional `p` missing (TypeError).
* **Reasoning:** Interaction between modern `/` syntax and legacy `**kwargs`.

### 20. How do you forcefully prevent a function from accepting `**kwargs` (i.e., strict signature)?

* **Difficulty:** Medium
* **Expected Answer:** Simply do not include `**kwargs` in the definition. If extra arguments are passed, Python automatically raises `TypeError`.
* **Reasoning:** Sometimes the answer is doing *nothing*. This prevents "argument swallowing" where typos in variable names are ignored by the function because they silently fell into `**kwargs`.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### 21. Dynamic HTML Tag Generator

* **Difficulty:** Easy
* **Question:** Write a function `make_tag(tag_name, text_content, **attributes)` that returns an HTML string. The attributes should be formatted as `key="value"`. Handle the special case where `class_` should be rendered as `class`.
* **Scenario:** Generating simple HTML snippets in a utility script or templating engine.
* **Code:**
```python
def make_tag(tag_name, text_content, **attributes):
    attr_strings = []
    for key, value in attributes.items():
        # Handle Python reserved keyword collision (class_ -> class)
        clean_key = key[:-1] if key.endswith('_') else key
        attr_strings.append(f'{clean_key}="{value}"')

    attr_str = " ".join(attr_strings)
    if attr_str:
        return f"<{tag_name} {attr_str}>{text_content}</{tag_name}>"
    return f"<{tag_name}>{text_content}</{tag_name}>"

# Usage
print(make_tag("div", "Hello", id="main", class_="container"))
# Output: <div id="main" class="container">Hello</div>

```


* **Explanation:**
1. Iterates over `attributes.items()`.
2. Checks for trailing underscore to handle `class_`.
3. Formats into `key="value"`.
4. Assembles final string.


* **Time/Space:**  where K is number of attributes.

### 22. Configuration Merger with Defaults

* **Difficulty:** Easy
* **Question:** Write a function `connect_db(defaults, **overrides)` where `defaults` is a dictionary. The function should print the final connection string using merged settings. Overrides take precedence.
* **Scenario:** Database connection setup where environment variables might override defaults.
* **Code:**
```python
def connect_db(defaults, **overrides):
    # 1. Start with defaults
    final_config = defaults.copy()

    # 2. Update with kwargs
    final_config.update(overrides)

    print(f"Connecting to {final_config.get('host')}:{final_config.get('port')}")

# Usage
base_settings = {'host': 'localhost', 'port': 5432, 'db': 'test'}
connect_db(base_settings, port=9999, user="admin")

```


* **Common Mistakes:** Modifying `defaults` directly (side effect). Using `copy()` prevents this.
* **Time/Space:**  to copy and update.

### 23. The "Print All" Logger

* **Difficulty:** Easy
* **Question:** Create a function `logger(msg, **kwargs)` that prints a message, followed by all extra data formatted as `Key: Value` on new lines.
* **Scenario:** A generic logging utility for debugging API responses.
* **Code:**
```python
def logger(msg, **kwargs):
    print(f"[LOG] {msg}")
    for k, v in kwargs.items():
        print(f"  -> {k}: {v}")

logger("User login", user_id=101, status="Success", ip="127.0.0.1")

```


* **Why asked:** Tests basic iteration over `kwargs`.

### 24. Function Argument Proxy (Decorator)

* **Difficulty:** Medium
* **Question:** Write a decorator `time_it` that measures the execution time of *any* function, regardless of its signature (positional or keyword args).
* **Scenario:** Performance profiling middleware in a Flask/Django app.
* **Code:**
```python
import time
import functools

def time_it(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        # Forward arguments exactly as received
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@time_it
def heavy_process(n, step=1, **options):
    time.sleep(0.1)
    return n * step

heavy_process(10, step=2, mode="fast")

```


* **Explanation:** The `wrapper` accepts `*args` and `**kwargs` and unpacks them into `func`. This makes the decorator universal.
* **Real App Usage:** Used in Datadog/NewRelic APM agents to trace function calls.

### 25. Factory Pattern with Kwargs

* **Difficulty:** Medium
* **Question:** Implement a class `Car`. Add a static method `create(**kwargs)` that validates if 'model' and 'year' exist in `kwargs` before returning a `Car` instance. If missing, raise `ValueError`.
* **Scenario:** safely parsing JSON payloads from an API endpoint into Python Objects.
* **Code:**
```python
class Car:
    def __init__(self, model, year, color="white"):
        self.model = model
        self.year = year
        self.color = color

    @staticmethod
    def create(**kwargs):
        required = ['model', 'year']
        # Check for missing keys
        missing = [key for key in required if key not in kwargs]

        if missing:
            raise ValueError(f"Missing fields: {missing}")

        # Unpack remaining valid kwargs into constructor
        return Car(**kwargs)

try:
    c = Car.create(model="Tesla", year=2023, color="Red", extra="ignored")
    # Note: 'extra' might cause __init__ to fail if not handled. 
    # A safer create method would filter kwargs.
except TypeError as e:
    print(f"Error: {e}")

```


* **Common Mistakes:** Passing `extra` arguments that `__init__` doesn't accept. (The code above would actually fail if `__init__` doesn't accept `extra`. A robust version filters keys).

### 26. Safe Kwargs Filtering

* **Difficulty:** Medium
* **Question:** Write a function `filter_and_call(func, **kwargs)` that inspects `func`'s signature and only passes the keywords that `func` actually accepts, discarding the rest.
* **Scenario:** Passing a large configuration dict to multiple small functions that only need specific parts.
* **Code:**
```python
import inspect

def target(a, b):
    print(f"Target called with a={a}, b={b}")

def filter_and_call(func, **kwargs):
    # Get parameters of the function
    sig = inspect.signature(func)
    valid_keys = sig.parameters.keys()

    # Filter dictionary
    filtered_args = {k: v for k, v in kwargs.items() if k in valid_keys}

    return func(**filtered_args)

data = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
filter_and_call(target, **data) # 'c' and 'd' are safely ignored

```


* **Why asked:** Demonstrates advanced introspection capabilities relevant to framework development.

### 27. The "Double" Wrapper (Forwarding between classes)

* **Difficulty:** Medium
* **Question:** You have a `BasePlotter` class with a complex `draw(**settings)` method. Create a subclass `SpecialPlotter` that overrides `draw`. It should extract a specific argument `theme`, process it, and pass all *other* arguments to the parent.
* **Scenario:** Extending library classes (like Matplotlib wrappers) without breaking existing functionality.
* **Code:**
```python
class BasePlotter:
    def draw(self, **settings):
        print(f"Drawing with {settings}")

class SpecialPlotter(BasePlotter):
    def draw(self, **kwargs):
        # Extract specific arg, default to None if missing
        theme = kwargs.pop('theme', 'default')

        print(f"Applying theme: {theme}")

        # Forward the rest (theme is now removed from kwargs)
        super().draw(**kwargs)

sp = SpecialPlotter()
sp.draw(x=10, y=20, theme="dark")

```


* **Explanation:** Using `kwargs.pop('key')` removes the item so it doesn't cause an "unexpected argument" error in the parent class.

### 28. Recursive Kwargs (Nested Configuration)

* **Difficulty:** Medium
* **Question:** Write a recursive function `build_query(table, **conditions)` that constructs a mock SQL string. If a value in `conditions` is a dictionary, it should be treated as a nested JSON check (e.g., `data->>'key' = value`).
* **Scenario:** Building an ORM query builder.
* **Code:**
```python
def build_query(table, **conditions):
    clauses = []
    for col, val in conditions.items():
        if isinstance(val, dict):
            # Handle nested dict as JSON query
            for sub_k, sub_v in val.items():
                clauses.append(f"{col}->>'{sub_k}' = '{sub_v}'")
        else:
            clauses.append(f"{col} = '{val}'")

    where_clause = " AND ".join(clauses)
    return f"SELECT * FROM {table} WHERE {where_clause}"

print(build_query("users", id=1, meta={'role': 'admin', 'active': 'true'}))
# Output: SELECT * FROM users WHERE id = '1' AND meta->>'role' = 'admin' AND meta->>'active' = 'true'

```



### 29. Dependency Injection Container

* **Difficulty:** Medium
* **Question:** strict `**kwargs` usage. Create a function `register_service(name, **dependencies)`. Store services in a global dict. When a service is registered, check if its dependencies exist in the registry.
* **Scenario:** Simple Service Locator pattern.
* **Code:**
```python
registry = {}

def register_service(name, **deps):
    for dep_name, service_instance in deps.items():
        if not service_instance:
             raise ValueError(f"Dependency {dep_name} is None")

    registry[name] = {'deps': deps}
    print(f"Registered {name} with {list(deps.keys())}")

# Mock usage
class DB: pass
class Auth: pass

db = DB()
# Passing deps as kwargs
register_service("AuthService", database=db) 

```


* **Real Usage:** Similar to how Dependency Injection frameworks (like in FastAPI) wire components.

### 30. Memoization with kwargs

* **Difficulty:** Medium
* **Question:** Write a `memoize` decorator that works for functions taking `**kwargs`. Note that dictionaries are not hashable, so you cannot use them directly as cache keys. How do you solve this?
* **Scenario:** Caching API calls where arguments might be named parameters.
* **Code:**
```python
import functools

def memoize(func):
    cache = {}

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        # 1. Convert kwargs to a sorted tuple of items (hashable)
        kwargs_key = tuple(sorted(kwargs.items()))

        # 2. Create composite key
        key = (args, kwargs_key)

        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    return wrapper

@memoize
def expensive(a, b):
    print("Computing...")
    return a + b

expensive(a=1, b=2) # Computes
expensive(b=2, a=1) # Hits cache (sorted items make order irrelevant)

```


* **Explanation:** Dictionary keys in Python are not order-guaranteed (pre-3.7) or just generally not hashable. Sorting `items()` creates a consistent, hashable signature.
