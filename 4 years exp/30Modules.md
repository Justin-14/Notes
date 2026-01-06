Here is the complete, professionally formatted interview guide for **Python Modules**, adapted to the constraints and depth requested.

```markdown
# Python Modules — Interview Guide (30 Questions)

This guide is designed for a developer with **4 years of experience**, focusing on **Python Modules, Packages, and Import Mechanics**. It covers internal behavior, best practices, and backend architectural patterns.

---

# SECTION 1 — THEORY QUESTIONS (20 TOTAL)

## Easy Level (6 Questions)

### 1. Module vs. Package vs. Script
**Question:** What is the technical difference between a Python module, a package, and a script?
**Expected Answer:**
- **Module:** Any file ending in `.py`. It allows organization of code into logical parts.
- **Package:** A directory containing a collection of modules and a special `__init__.py` file (though optional in Python 3.3+ namespace packages). It allows hierarchical structuring.
- **Script:** A Python file intended to be executed directly (top-level execution) rather than imported.
**Explanation:** Everything in Python is an object. A module is an object of type `<class 'module'>`. A package is simply a module that contains a `__path__` attribute.
**Time/Space Complexity:** N/A (Architectural concept).
**Common Mistakes:** Thinking a folder is a package without understanding `__init__.py` or namespace packages.
**Why Ask This:** To ensure the candidate uses correct terminology and understands project structure.
**Sample Code:**
```python
import os            # 'os' is a module
import json          # 'json' is a package (it's a directory with __init__.py)

print(type(os))      # <class 'module'>
print(type(json))    # <class 'module'>
print(json.__path__) # Packages have a __path__ attribute; modules usually don't.

```

### 2. The `__init__.py` File

**Question:** What is the purpose of `__init__.py` in a directory?
**Expected Answer:** It marks a directory as a Python package, allowing imports from it. It runs automatically when the package is imported, making it the perfect place for initialization logic or exporting specific sub-modules to the package namespace.
**Explanation:** In Python 3.3+, it is not strictly required for "Namespace Packages," but for standard "Regular Packages," it is essential for defining the package scope and simplifying API exposure.
**Common Mistakes:** Leaving it empty when it could be used to simplify imports (e.g., exposing `Item` from `package.sub.item` as `package.Item`).
**Why Ask This:** foundational knowledge for structuring large backend projects.
**Sample Code:**

```python
# structure:
# mypkg/
#   __init__.py
#   database.py

# Inside __init__.py:
from .database import connect
# Now user can do: `from mypkg import connect` 
# Instead of: `from mypkg.database import connect`

```

### 3. `import` vs `from ... import`

**Question:** What happens differently in memory when you do `import mymod` vs `from mymod import myfunc`?
**Expected Answer:** Both statements load the **entire** module `mymod` into `sys.modules`.

* `import mymod`: Binds the module object to the name `mymod` in the local namespace.
* `from mymod import myfunc`: Loads `mymod`, but only binds the name `myfunc` in the local namespace. `mymod` is still loaded in memory, just not accessible by name locally.
**Medium-Depth Reasoning:** Many developers think `from ... import` saves memory by only loading one function. This is false. The module initialization code runs in both cases.
**Common Mistakes:** Believing `from` imports are more memory efficient.
**Why Ask This:** To test understanding of the import cache and memory management.
**Diagram:**

```text
sys.modules (Global Cache)
   |
   +-- "mymod": <module object at 0x...>
           |
           +-- myfunc: <function>

Local Namespace (import mymod)      Local Namespace (from mymod import myfunc)
   |                                   |
   +-- mymod -> [Ref to Module]        +-- myfunc -> [Ref to Function]

```

### 4. The `if __name__ == "__main__":` block

**Question:** Explain `if __name__ == "__main__":` to a junior developer.
**Expected Answer:** It checks if the script is being run directly (like `python script.py`) or being imported (like `import script`). If imported, `__name__` is the module's filename; if run directly, `__name__` is set to `"__main__"`.
**Explanation:** This prevents execution of code (like starting a server or running tests) when a file is merely imported to access its functions.
**Why Ask This:** Crucial for writing testable code and reusable scripts.
**Sample Code:**

```python
def logic():
    print("Logic running")

if __name__ == "__main__":
    # Only runs if I execute 'python this_file.py'
    # Does NOT run if I do 'import this_file'
    logic()

```

### 5. `sys.path` and Search Order

**Question:** When you import a module, where does Python look for it?
**Expected Answer:** Python searches locations in `sys.path` in order:

1. The directory containing the input script (current directory).
2. `PYTHONPATH` (if set).
3. Installation-dependent default directories (Standard Library, site-packages).
**Explanation:** `sys.path` is just a list. You can modify it at runtime to import modules from non-standard locations.
**Common Mistakes:** Assuming Python looks in the current working directory (CWD) of the terminal, rather than the directory of the entry-point script.
**Why Ask This:** Debugging `ModuleNotFoundError` is a daily task.

### 6. Standard vs. Third-Party Modules

**Question:** Where are third-party modules (like `requests`) installed vs standard library modules (like `os`)?
**Expected Answer:**

* **Standard Lib:** Installed with the Python interpreter directory.
* **Third-Party:** Usually in a `site-packages` directory inside the Python installation or virtual environment.
**Why Ask This:** Validates understanding of virtual environments and Python installation layout.

---

## Medium Level (14 Questions)

### 7. How `sys.modules` Caching Works

**Question:** If I import a module twice in two different files, does the code inside the module execute twice?
**Expected Answer:** No. Python caches imported modules in `sys.modules` (a dictionary).

1. Python checks `sys.modules`.
2. If present, it returns the existing module object.
3. If not, it loads the file, executes it, adds it to `sys.modules`, and returns it.
**Reasoning:** This implements the **Singleton Pattern** by default for modules. It ensures state (global variables) is shared across the application.
**Time Complexity:** First import: O(Cost of module code). Subsequent imports: O(1) (dict lookup).
**Common Mistakes:** Thinking `import` always refreshes the code.
**Sample Code:**

```python
import sys
# sys.modules is a dict mapping names to module objects
print(sys.modules['os']) 
# <module 'os' from '...'>

```

### 8. Circular Imports

**Question:** What triggers a `ImportError: cannot import name X` (Circular Import), and how do you fix it?
**Expected Answer:** Occurs when Module A imports Module B, but Module B imports Module A before A has finished initializing.
**Fixes:**

1. **Refactor:** Move shared logic to a third Module C (Best practice).
2. **Deferred Import:** Move the import inside the function/method instead of the top level.
**Reasoning:** Python executes module code line-by-line. If B tries to access a name in A that hasn't been defined yet (because A is stuck waiting for B), it fails.
**Why Ask This:** Very common in complex backend architectures (e.g., Models importing Utils, Utils importing Models).

### 9. The `__all__` Variable

**Question:** What is the specific purpose of `__all__` in a module?
**Expected Answer:** It is a list of strings defining what symbols are exported when a client uses `from module import *`.
**Reasoning:** Without `__all__`, `import *` imports every name that doesn't start with `_`. `__all__` allows the module author to restrict the public API.
**Common Mistakes:** Thinking `__all__` prevents direct imports (e.g., `from module import private_func` still works even if not in `__all__`).
**Sample Code:**

```python
# mymodule.py
__all__ = ['public_func']

def public_func(): pass
def _private_func(): pass # Won't be imported by *
def hidden_func(): pass   # Won't be imported by * due to __all__

```

### 10. Absolute vs. Relative Imports

**Question:** Explain the syntax `from . import utils` versus `import utils`. When must you use relative imports?
**Expected Answer:**

* **Absolute (`import utils`):** Searches `sys.path`.
* **Relative (`from . import utils`):** Searches relative to the current package.
**Constraint:** Relative imports **only** work when the file is part of a package. You cannot use them in the entry-point script (the file passed to python executable).
**Why Ask This:** Testing knowledge of writing libraries vs writing applications.

### 11. Reloading Modules

**Question:** Since Python caches modules, how do you force a module to reload without restarting the interpreter?
**Expected Answer:** Use `importlib.reload(module_object)`.
**Reasoning:** This is useful in development (REPL/Jupyter) or hot-reloading servers. However, references to old objects from the module held by other modules are **not** updated automatically.
**Common Mistakes:** Thinking `del sys.modules['mod']` is the clean way to reload (it's risky).
**Sample Code:**

```python
import config
import importlib

# .. Change config.py file manually ..

importlib.reload(config) # Updates the config object in place

```

### 12. Namespace Packages (Python 3.3+)

**Question:** How can two different directories on your computer both contribute to the same package namespace (e.g., `foo.bar` and `foo.baz`)?
**Expected Answer:** Using **Implicit Namespace Packages**. If you omit `__init__.py`, Python treats directories with the same name found in different `sys.path` locations as part of the same package.
**Reasoning:** Essential for monorepos or plugin architectures where plugins are installed separately but share a root package name.
**Comparisons:** Regular package (has `__init__.py`) vs Namespace package (no `__init__.py`).

### 13. `import *` (Star Import) Dangers

**Question:** Why is `from module import *` generally banned in production code?
**Expected Answer:**

1. **Namespace Pollution:** It overwrites existing variables without warning.
2. **Readability:** It makes it impossible to tell where a function came from (e.g., `calculate()`—is it local or from the import?).
3. **Pylint/Linters:** Static analysis tools cannot detect undefined variables easily.
**Exception:** Acceptable in `__init__.py` to expose an API or in interactive shells.

### 14. Dynamic Importing

**Question:** How do you import a module if you only have its name as a string (e.g., user input or config file)?
**Expected Answer:** Use `importlib.import_module("module_name")`.
**Alternative:** `__import__` (built-in), but `importlib` is the modern, preferred Pythonic interface.
**Scenario:** A plugin system where plugins are listed in a JSON config.
**Sample Code:**

```python
import importlib
mod_name = "math"
math_mod = importlib.import_module(mod_name)
print(math_mod.sqrt(16)) # 4.0

```

### 15. The `__main__.py` File

**Question:** What does adding a `__main__.py` file to a package directory allow you to do?
**Expected Answer:** It allows the directory (package) to be executed directly as a script using the `-m` flag or by pointing python to the directory.
**Usage:** `python -m mypackage` or `python mypackage/`.
**Real App Example:** `pip` itself. You can run `python -m pip install ...`.
**Why Ask This:** Tests knowledge of creating CLI-friendly packages.

### 16. Module-level Singletons

**Question:** Python modules are often described as "Natural Singletons". How would you implement a database connection pool that is shared across the entire app using this fact?
**Expected Answer:** Instantiate the connection pool at the module level (global scope in that module). Any other module that imports this module will get the *same* instance because the module is only executed once.
**Reasoning:** This avoids the need for complex Singleton classes with `__new__`.
**Sample Code:**

```python
# db.py
class DB:
    def __init__(self): self.conn = "Connected"

# The instance
db_instance = DB() 
# Everyone who 'imports db' shares 'db.db_instance'

```

### 17. `sys.meta_path` and Custom Importers

**Question:** How can you intercept the import process to import Python code from a non-standard source (like a database or encrypted file)?
**Expected Answer:** By interacting with `sys.meta_path`. You write a **Finder** (locates the module) and a **Loader** (creates the module object).
**Depth:** This is how standard import logic works; you can append a custom finder to `sys.meta_path`.
**Why Ask This:** Advanced architecture scenarios (e.g., loading plugins from S3).

### 18. Monkey Patching via Modules

**Question:** What is monkey patching in the context of modules, and why is it dangerous?
**Expected Answer:** Changing a module's attributes/functions at runtime (e.g., `math.sqrt = lambda x: 0`).
**Use Case:** Testing (mocking).
**Danger:** It affects the **global** state. If Module A patches `os.path`, Module B (which relies on the real `os.path`) will break.
**Thread Safety:** Patching modules in a multi-threaded app can cause race conditions.

### 19. Optional Dependencies (Try/Except Import)

**Question:** You are writing a library that has a heavy feature dependent on `pandas`, but you don't want to force all users to install `pandas`. How do you handle this?
**Expected Answer:** Use a try/except block on the import.
**Reasoning:** This makes the dependency "optional".
**Sample Code:**

```python
try:
    import pandas as pd
    HAS_PANDAS = True
except ImportError:
    HAS_PANDAS = False

def process_data(data):
    if HAS_PANDAS and isinstance(data, pd.DataFrame):
        return data.to_dict()
    return data # fallback

```

### 20. `.pyc` and `__pycache__`

**Question:** What are `.pyc` files and when are they generated?
**Expected Answer:** They are compiled bytecode files stored in `__pycache__`. Python compiles source `.py` to bytecode for the Python Virtual Machine.
**Update Logic:** Python checks if the `.py` timestamp is newer than the `.pyc`. If yes, it recompiles. If no, it loads the `.pyc` (faster startup).
**Common Mistake:** Thinking `.pyc` makes the *execution* speed faster. It only makes the *startup/import* speed faster.

---

# SECTION 2 — CODING QUESTIONS (10 TOTAL)

## Easy Level (3 Questions)

### 21. Creating a Clean Package Structure

**Coding Question:** Create a file structure for a package named `payment_gateway` that exposes a single function `process_payment` from an internal file `core.py`, so the user can import it directly from the package.
**Solution Code:**

```text
File structure:
payment_gateway/
├── __init__.py
└── core.py

```

```python
# payment_gateway/core.py
def process_payment(amount):
    return f"Processed ${amount}"

# payment_gateway/__init__.py
# This exposes the function to the top level
from .core import process_payment

```

**Explanation:** The `__init__.py` uses a relative import to pull `process_payment` into the package namespace.
**Common Mistakes:** Forgetting the dot `.` in `from .core`.

### 22. Printing `sys.path` Debugging

**Coding Question:** Write a script that prints every directory Python is searching for modules in, nicely formatted.
**Solution Code:**

```python
import sys

def print_search_paths():
    print("Python Module Search Paths:")
    for i, path in enumerate(sys.path):
        # Handle empty string which means 'current directory'
        display_path = path if path else "(Current Directory)"
        print(f"{i}: {display_path}")

if __name__ == "__main__":
    print_search_paths()

```

**Why Interviewers Ask:** Shows you know how to debug `ImportError`.

### 23. Conditional JSON Import

**Coding Question:** Write a snippet that tries to import `ujson` (fast JSON). If not available, it falls back to the standard `json` library, but aliases it as `json_lib` in both cases.
**Solution Code:**

```python
try:
    import ujson as json_lib
except ImportError:
    import json as json_lib

def serialize(data):
    return json_lib.dumps(data)

```

**Developer Scenario:** optimizing for performance in high-throughput APIs (ujson is faster) while maintaining compatibility.

---

## Medium Level (7 Questions)

### 24. Dynamic Import from String

**Coding Question:** Write a function `load_service(service_path)` that takes a string like `"services.auth.AuthService"` (dot notation) and returns the class `AuthService`.
**Solution Code:**

```python
import importlib

def load_service(service_path: str):
    try:
        # Split module path and class name
        # e.g. "services.auth", "AuthService"
        module_path, class_name = service_path.rsplit('.', 1)
        
        # Dynamically import the module
        module = importlib.import_module(module_path)
        
        # Get the class from the module
        cls = getattr(module, class_name)
        return cls
        
    except (ValueError, ImportError, AttributeError) as e:
        raise ImportError(f"Could not load service {service_path}") from e

# Usage
# AuthClass = load_service("my_app.auth.Auth")
# instance = AuthClass()

```

**Line-by-Line:**

* `rsplit('.', 1)` splits from the right once, separating the module path from the object name.
* `import_module` loads the module into memory.
* `getattr` retrieves the class object from the module object.
**Why Interviewers Ask:** Basis for dependency injection frameworks and configuration-driven architectures.

### 25. Plugin Loader System

**Coding Question:** You have a `plugins/` directory. Write a function that iterates through all `.py` files in that directory (excluding `__init__.py`) and imports them dynamically.
**Solution Code:**

```python
import os
import importlib
import sys

def load_plugins(plugin_dir):
    # Ensure plugin_dir is in sys.path so we can import
    if plugin_dir not in sys.path:
        sys.path.append(plugin_dir)

    loaded_plugins = []
    
    # List files in directory
    for filename in os.listdir(plugin_dir):
        if filename.endswith(".py") and filename != "__init__.py":
            # Strip extension to get module name
            module_name = filename[:-3] 
            
            # Dynamic import
            module = importlib.import_module(module_name)
            loaded_plugins.append(module)
            print(f"Loaded plugin: {module_name}")
            
    return loaded_plugins

```

**Developer Scenario:** Building a tool like a Linter or a Test Runner that needs to auto-discover extensions.
**Common Mistakes:** Modifying `sys.path` inside a loop or forgetting to strip `.py`.

### 26. Fixing Circular Dependencies with Local Imports

**Coding Question:** Refactor the following crashing code without creating a new file.

```python
# user.py
from auth import check_permission
class User:
    def delete(self):
        check_permission(self)

# auth.py
from user import User # CIRCULAR DEPENDENCY
def check_permission(u: User):
    print("Checking...")

```

**Solution Code:**

```python
# user.py (No changes needed usually, but typically the dependency is mutual)
# Let's assume user.py is fine for this context, the fix is in auth.py

# auth.py
# REMOVE top-level import
# from user import User 

def check_permission(u):
    # Import INSIDE the function (Deferred Import)
    from user import User
    
    if isinstance(u, User):
        print("Checking permission...")

```

**Explanation:** By moving the import inside the function, the import only happens when `check_permission` is *called*, by which time `user.py` has likely finished initializing.
**Real App Usage:** Django models often use this when models refer to each other.

### 27. Restricting `from *` using `__all__`

**Coding Question:** Create a module `math_utils.py` with `fast_add`, `slow_add`, and `_helper`. Ensure that `from math_utils import *` ONLY imports `fast_add`.
**Solution Code:**

```python
# math_utils.py

__all__ = ['fast_add']

def fast_add(x, y):
    return x + y

def slow_add(x, y):
    return x + y

def _helper():
    pass

```

**Verification Script:**

```python
# main.py
from math_utils import *

print(globals().keys()) 
# Contains 'fast_add'
# Does NOT contain 'slow_add' or '_helper'

```

**Why Interviewers Ask:** API design cleanliness.

### 28. Measuring Import Time

**Coding Question:** Write a context manager that measures how long a specific import takes.
**Solution Code:**

```python
import time
import importlib
from contextlib import contextmanager

@contextmanager
def time_import(module_name):
    start = time.perf_counter()
    yield importlib.import_module(module_name)
    end = time.perf_counter()
    print(f"Importing '{module_name}' took {end - start:.4f} seconds")

# Usage
try:
    with time_import('pandas') as pd:
        print("Pandas loaded")
except ImportError:
    print("Pandas not installed")

```

**Developer Scenario:** Optimizing CLI tool startup time (e.g., `aws-cli` or `gcloud` CLI tools).

### 29. Mocking a Module for Testing

**Coding Question:** Manually inject a fake module into `sys.modules` so that subsequent imports of that name return the fake object.
**Solution Code:**

```python
import sys
from types import ModuleType

# 1. Create a fake module
fake_payment = ModuleType("stripe")
fake_payment.charge = lambda amt: f"Fake charged {amt}"

# 2. Inject into sys.modules
sys.modules["stripe"] = fake_payment

# 3. Now, anywhere in the app...
import stripe 
print(stripe.charge(100)) # Output: Fake charged 100

```

**Real App Usage:** Testing code that relies on external services (like Stripe or AWS) without actually installing their SDKs or making network calls.

### 30. Enforcing Environment-Based Config

**Coding Question:** Create a `config.py` that imports settings from `config_dev.py` by default, but overrides them with `config_prod.py` if an environment variable `ENV` is set to `PROD`.
**Solution Code:**

```python
import os
import importlib

# Default config
API_KEY = "dev_key"
DEBUG = True

env = os.getenv("ENV", "DEV")

if env == "PROD":
    try:
        # Dynamically import prod config
        prod = importlib.import_module("config_prod")
        
        # Override local variables with prod variables
        # Using vars() or getattr to merge
        if hasattr(prod, "API_KEY"):
            API_KEY = prod.API_KEY
        if hasattr(prod, "DEBUG"):
            DEBUG = prod.DEBUG
            
    except ImportError:
        print("Warning: config_prod.py not found, using defaults.")

print(f"Config loaded: Debug={DEBUG}")

```

**Why Interviewers Ask:** Standard pattern for managing secrets and environments in Flask/Django apps.

```

```
