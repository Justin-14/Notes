# Python Global keyword— Interview Guide (30 Questions)

## Table of Contents

* [Section 1: Theory Questions (20)](https://www.google.com/search?q=%23section-1--theory-questions-20-total)
* [Section 2: Coding Questions (10)](https://www.google.com/search?q=%23section-2--coding-questions-10-total)

---

## Section 1 — Theory Questions (20 Total)

### Easy Level (6 Questions)

#### 1. What is the precise purpose of the `global` keyword in Python?

**Answer:** The `global` keyword allows a function to modify a variable defined in the global scope (module level). By default, assigning a value to a variable inside a function creates a *local* variable, even if a variable with the same name exists globally. `global` declares that the identifier refers to the global namespace.

**Explanation:** Python uses lexical scoping. If you try to write to a variable inside a function, Python assumes it is local unless told otherwise. Reading a global variable does not require the keyword; only writing (assignment) does.

**Why interviewers ask this:** To test understanding of Python's default scoping rules regarding assignment vs. referencing.

**Sample Code:**

```python
x = 10

def modify_global():
    global x  # Declare x as the global 'x'
    x = 20    # Modifies the module-level variable

modify_global()
print(x)  # Output: 20

```

---

#### 2. Explain the `UnboundLocalError` in relation to global variables.

**Answer:** `UnboundLocalError` occurs when a variable is referenced before assignment within a function, but the compiler recognizes that the variable is assigned somewhere else in that same scope. This often happens when a developer tries to read a global variable and then assign to it later in the same function without using the `global` keyword.

**Reasoning:** Python's compiler scans the function body. If it sees an assignment (`x = ...`), it flags `x` as local for the *entire* function. If you try to print `x` before that assignment, Python looks for a local `x` (which isn't bound yet) rather than falling back to the global `x`.

**Common Mistakes:** Thinking code is executed line-by-line regarding scope; scope is determined at compile time for the block.

**Sample Code:**

```python
count = 0

def increment():
    # Throws UnboundLocalError because 'count' is assigned below, making it local
    print(count) 
    count += 1 

# increment() # Uncommenting this crashes

```

---

#### 3. Does modifying a mutable global object (like a list) inside a function require the `global` keyword?

**Answer:** No. You only need the `global` keyword if you are **rebinding** (reassigning) the variable name to a new object. Modifying the contents of a mutable object (e.g., `append`, `update`, slice assignment) works without `global` because you are just accessing the object referenced by the global name.

**Comparison:** - `x.append(1)` → **In-place modification** (No `global` needed).

* `x = [1]` → **Reassignment** (`global` needed).

**Why interviewers ask this:** To verify the candidate distinguishes between *name binding* and *object mutation*.

**Sample Code:**

```python
my_list = []

def add_item():
    my_list.append(1)  # Works fine without 'global'

def reset_list():
    global my_list
    my_list = []       # Reassignment requires 'global'

```

---

#### 4. What is the LEGB rule?

**Answer:** LEGB stands for **L**ocal, **E**nclosing, **G**lobal, **B**uilt-in. It describes the order in which Python resolves variable names.

1. **Local:** Inside the current function.
2. **Enclosing:** Inside enclosing functions (for nested functions/closures).
3. **Global:** At the top level of the module.
4. **Built-in:** Special names like `len`, `range`, `open`.

**ASCII Diagram:**

```text
   +------------------------------+
   |   Built-in (len, str)        |
   |  +------------------------+  |
   |  |  Global (Module)       |  |
   |  |  +------------------+  |  |
   |  |  | Enclosing (Outer)|  |  |
   |  |  |  +------------+  |  |  |
   |  |  |  | Local      |  |  |  |
   |  |  |  +------------+  |  |  |
   |  |  +------------------+  |  |
   |  +------------------------+  |
   +------------------------------+

```

---

#### 5. How can you access the dictionary of current global variables?

**Answer:** Using the built-in function `globals()`. It returns a dictionary representing the current global symbol table. You can read and write to this dictionary to dynamically affect global variables.

**Why interviewers ask this:** To see if the candidate knows about introspection and the underlying implementation of namespaces.

**Sample Code:**

```python
x = 100
def check_globals():
    g = globals()
    print(g['x']) # Access variable by string name
    g['y'] = 500  # Dynamically create a global variable

check_globals()
print(y) # Output: 500

```

---

#### 6. What happens if a global variable and a local variable have the same name?

**Answer:** The local variable **shadows** the global variable within that function's scope. Any reference to that name inside the function will target the local variable, leaving the global one untouched (unless `global` is explicitly used).

**Explanation:** This is a safety feature to prevent functions from accidentally corrupting global state.

---

### Medium Level (14 Questions)

#### 7. What is the difference between `global` and `nonlocal`?

**Answer:** - `global` binds a variable to the **module-level** (global) scope.

* `nonlocal` binds a variable to the **nearest enclosing** scope that is NOT global. It is used primarily in nested functions (closures).

**Reasoning:** If you are in a nested function and want to modify a variable in the outer function, `global` will skip the outer function and look at the module level. `nonlocal` stops at the outer function.

**Sample Code:**

```python
x = "global_x"

def outer():
    x = "outer_x"
    
    def inner_global():
        global x
        x = "modified_global"
        
    def inner_nonlocal():
        nonlocal x
        x = "modified_outer"
        
    inner_nonlocal()
    print(f"Outer after nonlocal: {x}") # modified_outer
    
    inner_global()
    print(f"Outer after global: {x}")   # modified_outer (unchanged)

outer()
print(f"Global: {x}") # modified_global

```

---

#### 8. Why is the heavy use of global variables generally discouraged in Python application design?

**Answer:** 1. **Implicit Coupling:** Functions depending on globals have hidden dependencies, making them hard to test and reuse.
2. **Concurrency Issues:** In multi-threaded apps, globals are shared state and require locking (Mutex) to prevent race conditions.
3. **Namespace Pollution:** Increases the risk of naming collisions.
4. **Debugging Difficulty:** It is hard to track where and when a global state was changed.

**Context:** In 4 years of experience, you are expected to prefer passing arguments and dependency injection over global state.

---

#### 9. How does the `global` keyword affect bytecode generation?

**Answer:** Using `global` changes the opcode used for variable access.

* Without `global` (and with assignment): Python generates `STORE_FAST` and `LOAD_FAST` (optimized for local array access).
* With `global`: Python generates `STORE_GLOBAL` and `LOAD_GLOBAL` (dictionary lookup).

**Performance Note:** `LOAD_FAST` is slightly faster than `LOAD_GLOBAL` because local variables are stored in a fixed-size array in the C stack frame, whereas globals require a hash map (dictionary) lookup.

---

#### 10. Can you use `global` to create a variable that didn't exist before?

**Answer:** Yes. If you declare `global x` inside a function and assign a value to `x`, it is created in the module scope even if it wasn't defined at the top level previously.

**Use Case:** This is often considered a "lazy initialization" of a global, though explicitly defining `x = None` at the top level is better for readability.

---

#### 11. How do global variables behave across different modules?

**Answer:** Global variables are **module-scoped**, not truly "application-global".
If `moduleA` has `x = 1` and `moduleB` imports `moduleA`, `moduleA.x` is the variable.
However, `from moduleA import x` creates a **local copy** of the reference in `moduleB`. Modifying `x` in `moduleB` (reassigning) will NOT affect `moduleA.x`.

**Common Mistake:** Users do `from config import DEBUG`, change `DEBUG = True`, and wonder why other modules don't see the change. Use `import config; config.DEBUG = True` instead.

---

#### 12. Is it thread-safe to modify a global variable in Python?

**Answer:** Not inherently. While the GIL (Global Interpreter Lock) protects atomic bytecode operations, many Python operations (like `count += 1`) are not atomic (Read value -> Add 1 -> Write value). A context switch can happen in between.
You must use `threading.Lock()` when modifying globals in a multi-threaded environment.

**Line-by-line explanation:**
`n += 1` translates to:

1. `LOAD_GLOBAL n`
2. `LOAD_CONST 1`
3. `INPLACE_ADD`
4. `STORE_GLOBAL n`
Threads can switch after step 1, leading to lost updates.

---

#### 13. How does `exec()` interact with global scope?

**Answer:** `exec(code, globals_dict, locals_dict)` allows dynamic execution. If `globals_dict` is not provided, it executes in the current scope. If you provide a dictionary, `exec` works within that "sandboxed" global scope.

**Relevance:** This is how metaprogramming and some plugin architectures work, isolating the execution context from the actual module globals.

---

#### 14. What is the "Module Singleton" pattern using globals?

**Answer:** Since modules are initialized only once (sys.modules cache), a global variable instantiated at the top level of a module acts as a Singleton. All other modules importing it share the exact same object instance.

**Scenario:** Database connections or configuration objects are often stored as module-level globals.

---

#### 15. Can a global variable be deleted from within a function?

**Answer:** Yes, but you must declare it global first, then use `del`.

**Sample Code:**

```python
x = 10
def delete_global():
    global x
    del x  # Removes x from the global namespace dictionary

delete_global()
# print(x) # Raises NameError

```

---

#### 16. How do List Comprehensions interact with Scope (Python 2 vs 3)?

**Answer:** - **Python 2:** Loop variables in list comprehensions leaked into the global/enclosing scope.

* **Python 3:** List comprehensions have their own internal scope. Variables defined inside the comprehension do **not** leak or overwrite global variables.

**Why interviewers ask this:** To check knowledge of Python version history and scoping safety improvements.

---

#### 17. How can you define "Constants" in Python global scope?

**Answer:** Python does not have a strict `const` keyword. By convention, global constants are named in `ALL_CAPS`.
From Python 3.8+, you can use `typing.Final` to indicate to static type checkers (like mypy) that a global variable should not be reassigned, but the runtime does not enforce it.

---

#### 18. What is the impact of `global` in recursive functions?

**Answer:** Using a global variable to maintain state in recursion (e.g., a counter) is a common pattern for beginners but is **not re-entrant**. If two calls to the recursive wrapper happen simultaneously (or in threads), the global state gets corrupted.
**Better approach:** Pass the state as an argument or use a class.

---

#### 19. If you import a module inside a function, is the module name global?

**Answer:** No. If you do `import math` inside `def func():`, the name `math` is **local** to `func`. You cannot access `math` outside that function. The module object itself is loaded into `sys.modules` globally, but the *name binding* is local.

---

#### 20. How do you share global variables across multiple processes (Multiprocessing)?

**Answer:** Standard global variables are **not shared** between processes because each process has its own memory space. Changing a global in Process A does not affect Process B.
**Solution:** Use `multiprocessing.Value`, `Manager`, or shared memory to share state across processes.

---

## Section 2 — Coding Questions (10 Total)

### Easy Level (3 Questions)

#### 1. Fix the UnboundLocalError

**Question:** The following code crashes. Fix it so it prints 0, then 1, and correctly updates the global counter without passing arguments.

```python
counter = 0

def step():
    print(counter)
    counter += 1

step()

```

**Solution:**

```python
counter = 0

def step():
    global counter  # Fix: Declare intent to write to global
    print(counter)
    counter += 1

step() # Prints 0
step() # Prints 1

```

**Why interviewers ask this:** The most basic check of `global` keyword usage.

---

#### 2. The Global Toggle Switch

**Question:** Write a function `toggle_state()` that switches a global boolean `IS_ACTIVE` between `True` and `False`.

**Solution:**

```python
IS_ACTIVE = False

def toggle_state():
    global IS_ACTIVE
    IS_ACTIVE = not IS_ACTIVE
    return IS_ACTIVE

# Usage
print(toggle_state()) # True
print(toggle_state()) # False

```

**Developer Scenario:** A feature flag toggle in a simple script or a pause button logic in a game loop.

---

#### 3. Resetting Global Configuration

**Question:** You have a global dictionary `CONFIG`. Write a function `reset_config()` that clears it and sets a default key `{'host': 'localhost'}`. Ensure the object identity of `CONFIG` remains the same if possible, or handle reassignment correctly.

**Solution:**

```python
CONFIG = {'host': '0.0.0.0', 'port': 8080}

def reset_config():
    # Option A: Modify in-place (No global needed)
    CONFIG.clear()
    CONFIG['host'] = 'localhost'

    # Option B: Reassign (Global needed)
    # global CONFIG
    # CONFIG = {'host': 'localhost'}

reset_config()
print(CONFIG)

```

**Explanation:** Option A is preferred as it preserves references held by other parts of the code.

---

### Medium Level (7 Questions)

#### 4. The Request Counter (Closures vs Globals)

**Question:** You are tracking request counts.

1. Implement it using a `global` variable.
2. Implement it using a closure (factory pattern) to avoid globals.
Explain why the closure is better.

**Solution:**

```python
# 1. Global Approach
req_count = 0
def request_handler_global():
    global req_count
    req_count += 1
    return req_count

# 2. Closure Approach (Better)
def counter_factory():
    count = 0  # Enclosed scope
    def handler():
        nonlocal count
        count += 1
        return count
    return handler

# Usage
api_a_counter = counter_factory()
api_b_counter = counter_factory()

print(api_a_counter()) # 1
print(api_a_counter()) # 2
print(api_b_counter()) # 1 (Independent state!)

```

**Why interviewers ask this:** Demonstrates how to replace globals with better state encapsulation patterns.

---

#### 5. Thread-Safe Global Increment

**Question:** You have a global variable `DATABASE_HITS = 0`. Write a function intended to be run by 100 threads concurrently that safely increments this variable without race conditions.

**Solution:**

```python
import threading

DATABASE_HITS = 0
db_lock = threading.Lock()

def record_hit():
    global DATABASE_HITS
    # Acquire lock before reading/writing global state
    with db_lock:
        local_copy = DATABASE_HITS
        local_copy += 1
        DATABASE_HITS = local_copy

# Simulation
threads = [threading.Thread(target=record_hit) for _ in range(100)]
for t in threads: t.start()
for t in threads: t.join()

print(DATABASE_HITS) # Output: 100

```

**Time Complexity:** O(1) per call (but serialized).

**Real App Usage:** Analytics counters in a multi-threaded web server (e.g., Flask without Gunicorn workers).

---

#### 6. Cross-Module Config Updater

**Question:** Simulate two modules.

* `config.py`: holds `SETTINGS = {'mode': 'prod'}`.
* `main.py`: imports `config`, modifies `SETTINGS['mode']` to 'dev', and proves the change is reflected globally.
Then show a mistake where `from config import SETTINGS` breaks updates if `SETTINGS` was an integer (immutable).

**Solution:**

```python
# --- config.py ---
# SETTINGS = {'mode': 'prod'}
# TIMEOUT = 10

# --- main.py ---
import config

def update_mode():
    # Modifying mutable global via module namespace
    config.SETTINGS['mode'] = 'dev' 

def update_timeout_wrong():
    from config import TIMEOUT
    # This creates a local variable TIMEOUT, distinct from config.TIMEOUT
    TIMEOUT = 999 

def update_timeout_correct():
    # Correct way to update immutable module global
    config.TIMEOUT = 999

update_mode()
print(config.SETTINGS) # {'mode': 'dev'}

update_timeout_correct()
print(config.TIMEOUT)  # 999

```

---

#### 7. Global Registry Pattern (Decorator)

**Question:** Create a decorator `@register_route` that adds the decorated function's name and reference to a global dictionary `ROUTE_MAP`.

**Solution:**

```python
ROUTE_MAP = {}

def register_route(path):
    def decorator(func):
        # We are modifying a mutable global dict (no 'global' needed)
        ROUTE_MAP[path] = func
        return func
    return decorator

@register_route("/home")
def home_view():
    return "Home Page"

@register_route("/about")
def about_view():
    return "About Page"

print(ROUTE_MAP)
# Output: {'/home': <function home_view...>, '/about': <function about_view...>}

```

**Real App Usage:** This is exactly how frameworks like **Flask** and **FastAPI** handle routing internally.

---

#### 8. Implementing a Singleton via Global Module State

**Question:** Create a simple `Logger` class. Ensure that no matter how many times `get_logger()` is called, it returns the *same* instance stored in a global variable. Initialize it lazily (only on the first call).

**Solution:**

```python
_LOGGER_INSTANCE = None

class Logger:
    def log(self, msg):
        print(f"LOG: {msg}")

def get_logger():
    global _LOGGER_INSTANCE
    if _LOGGER_INSTANCE is None:
        print("Initializing Logger...")
        _LOGGER_INSTANCE = Logger()
    return _LOGGER_INSTANCE

# Usage
l1 = get_logger() # Initializing Logger...
l2 = get_logger() # (No print)
print(l1 is l2)   # True

```

---

#### 9. Context Manager for Temporary Global Override

**Question:** Write a Context Manager `TempGlobal` that takes a global variable name and a temporary value. It should set the global to the temp value on enter, and restore the original value on exit.
*Hint: Use `globals()`.*

**Solution:**

```python
MODE = "PRODUCTION"

class TempGlobal:
    def __init__(self, var_name, temp_val):
        self.var_name = var_name
        self.temp_val = temp_val
        self.original_val = None
        
    def __enter__(self):
        self.original_val = globals()[self.var_name]
        globals()[self.var_name] = self.temp_val
        
    def __exit__(self, exc_type, exc_val, traceback):
        globals()[self.var_name] = self.original_val

print(f"Before: {MODE}")

with TempGlobal("MODE", "TESTING"):
    print(f"Inside: {MODE}")
    
print(f"After: {MODE}")

```

**Real App Usage:** Temporarily overriding configuration (like `DEBUG=True`) during unit tests.

---

#### 10. Dependency Injection Refactor

**Question:** (Refactoring) You are given a legacy function that relies on a global `API_KEY`.
Refactor it to accept the key as an argument, but maintain backward compatibility using a default argument pointing to the global.

**Solution:**

```python
API_KEY = "1234-secret"

# Legacy Code
# def fetch_data():
#     client = Client(API_KEY) ...

# Refactored Code
def fetch_data(api_key=None):
    # If no key passed, fallback to global
    # This allows testing with different keys without patching globals
    key_to_use = api_key if api_key is not None else API_KEY
    print(f"Using Key: {key_to_use}")

fetch_data()              # Uses global "1234-secret"
fetch_data("9999-test")   # Uses local "9999-test"

```

**Why interviewers ask this:** This shows you know how to modernize legacy codebases incrementally without breaking existing calls.
