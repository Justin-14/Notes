# Python Special Variable `__name__` — Interview Guide (30 Questions)

## Table of Contents

1. [Section 1: Theory Questions (20)](https://www.google.com/search?q=%23section-1--theory-questions-20-total)
2. [Section 2: Coding Questions (10)](https://www.google.com/search?q=%23section-2--coding-questions-10-total)

---

# Section 1 — Theory Questions (20 Total)

## Easy Level (6 Questions)

### 1. What is the fundamental purpose of the `__name__` variable in Python?

**Answer:**
`__name__` is a built-in special variable (attribute) that evaluates to the name of the current module. It is automatically set by the Python interpreter when a file is read.

* If the module is being run directly (e.g., via command line), `__name__` is set to the string `"__main__"`.
* If the module is imported, `__name__` is set to the actual name of the module (usually the filename without `.py`).

**Explanation:**
This mechanism allows a Python file to act as both a reusable script (library) and a standalone program.

**Sample Code:**

```python
# script.py
print(f"The value of __name__ is: {__name__}")

# Execution: python script.py
# Output: The value of __name__ is: __main__

```

---

### 2. What is the standard idiom `if __name__ == "__main__":` used for?

**Answer:**
This idiom (often called the "main guard") is used to control code execution. It ensures that a block of code runs **only** when the file is executed directly as a script, and **not** when it is imported as a module in another script.

**Why interviewers ask this:**
To verify you understand basic script reusability and how to prevent side effects during imports.

**Sample Code:**

```python
def main():
    print("Doing work...")

if __name__ == "__main__":
    main()

```

---

### 3. What is the data type of `__name__`?

**Answer:**
It is a strictly built-in **string** (`str`).

**Common Mistakes:**
Candidates sometimes guess it might be a special object or a symbol. It is just a standard string.

**Sample Code:**

```python
print(type(__name__))
# Output: <class 'str'>

```

---

### 4. Does `__name__` exist in the Python Interactive Shell (REPL)?

**Answer:**
Yes. Inside the interactive shell, the scope is considered the "top-level" environment, so `__name__` is set to `"__main__"`.

**Reasoning:**
The REPL acts as the entry point for execution, similar to running a script directly.

**ASCII Diagram:**

```text
[ User ] -> [ Opens REPL ] -> [ Python Interpreter ]
                                     |
                                  Sets global scope:
                                  __name__ = "__main__"

```

---

### 5. If I have a file `utils.py` and I import it into `app.py`, what is the value of `__name__` inside `utils.py`?

**Answer:**
Inside `utils.py`, `__name__` will be the string `"utils"`.

**Explanation:**
When Python imports a file, it creates a module object. The `__name__` attribute of that object is set to the module's name (derived from the filename).

**Sample Code:**

```python
# utils.py
print(__name__)

# app.py
import utils
# Output when running app.py: "utils"

```

---

### 6. Is `__name__` a read-only variable?

**Answer:**
No, strictly speaking, it is mutable within the module's global scope. You *can* assign a new value to it, though doing so is bad practice and can break import logic.

**Common Mistakes:**
Assuming "special" dunder variables are immutable constants.

**Sample Code:**

```python
print(__name__)  # __main__
__name__ = "CustomName"
print(__name__)  # CustomName

```

---

## Medium Level (14 Questions)

### 7. How does `__name__` behave differently when running a directory using `python -m <directory>` vs running the script directly?

**Answer:**
If you run a directory or package, Python looks for a `__main__.py` file inside that directory.

* Inside `__main__.py`, the `__name__` variable will be set to `"__main__"`.
* This allows a package to be executable.

**Reasoning:**
This is the entry point mechanism for packages. It tells Python, "Treat this directory as an executable program."

**Relevant Comparison:**

* `python script.py` -> `script.py` is `__main__`.
* `python -m mypkg` -> `mypkg/__main__.py` is executed, and its `__name__` is `__main__`.

---

### 8. Explain the relationship between `__name__` and `sys.modules`.

**Answer:**
`sys.modules` is a dictionary mapping module names to module objects. The key used in `sys.modules` usually corresponds to the module's `__name__`.
When you run a script, it is stored in `sys.modules` under the key `"__main__"`. When you import it, it is stored under its filename (e.g., `"utils"`).

**Deep Reasoning:**
This is why importing a script that is currently running as main causes it to run twice: once as `__main__` and once as `utils` (two different entries in `sys.modules`).

---

### 9. How does `__name__` affect Relative Imports?

**Answer:**
Relative imports (e.g., `from . import utils`) rely on `__name__` (and `__package__`) to determine the current module's position in the package hierarchy.
If you run a script directly, `__name__` is `"__main__"`, which contains no dot-separated hierarchy information. Therefore, **relative imports fail** in a script executed directly.

**Common Mistakes:**
Trying to use `from . import x` inside a file run as `python my_script.py` results in `ImportError: attempted relative import with no known parent package`.

---

### 10. How does the `multiprocessing` module rely on `__name__` on Windows?

**Answer:**
On Windows (which uses `spawn` instead of `fork`), `multiprocessing` must import the main module in the child process to rebuild the environment.
If the main code (creating processes) is not guarded by `if __name__ == "__main__":`, the child process will re-execute the process creation line, leading to an infinite loop of process spawns (a "fork bomb").

**Why interviewers ask this:**
To test knowledge of OS-level differences and the critical safety role of `__name__`.

**Sample Code:**

```python
import multiprocessing

def worker():
    print("Worker")

# On Windows, this MUST be inside the guard
if __name__ == "__main__":
    p = multiprocessing.Process(target=worker)
    p.start()

```

---

### 11. What is the value of `__name__` for a sub-package module `mypkg.sub.utils`?

**Answer:**
The value will be the full dot-separated path: `"mypkg.sub.utils"`.

**Explanation:**
Python sets `__name__` to reflect the full import path, ensuring uniqueness in `sys.modules`.

---

### 12. If `__name__` is modified at runtime, does it affect the module name in `sys.modules`?

**Answer:**
No. Changing `__name__` inside the script only changes the local variable within that module's scope. It does not update the key in the `sys.modules` dictionary.

**Sample Code:**

```python
# In a module
import sys
__name__ = "FakeName"
# sys.modules key remains the original import name

```

---

### 13. How does `runpy.run_module` interact with `__name__`?

**Answer:**
The `runpy` standard library module allows you to execute modules without importing them in the standard way. It allows you to specify what you want `__name__` to be (defaulting to `"__main__"`).
This is what `python -m <module_name>` uses internally.

---

### 14. Can `__name__` ever be None?

**Answer:**
In standard Python execution, no. Even in edge cases, it defaults to a string. However, if you manually create a dummy module object type, it might not have the attribute until set. But in standard flow: No.

---

### 15. What is the difference between `__name__` and `__file__`?

**Answer:**

* `__name__`: Logical name of the module (e.g., `"json"` or `"__main__"`).
* `__file__`: The filesystem path to the script (e.g., `"/usr/lib/python3.12/json/__init__.py"`).

**Use Case:**
Use `__name__` for logic control; use `__file__` for resource loading (locating files relative to the script).

---

### 16. Why does `from module import *` not import `__name__`?

**Answer:**
`__name__` is a global variable in the module, but it is considered system-defined metadata. However, `import *` technically imports all public names (those not starting with `_`).
Since `__name__` starts with `_`, it is excluded by default from wildcard imports.

---

### 17. How is `__name__` handled in a compiled `.pyc` file execution?

**Answer:**
The behavior is identical. The `.pyc` file contains the bytecode, but the interpreter sets `__name__` dynamically at runtime based on *how* the bytecode is invoked, not based on the static file content.

---

### 18. What happens to `__name__` if I execute a script using `exec()`?

**Answer:**
It depends on the dictionary passed to `exec()`.
If you pass a dictionary as `globals`, `exec` uses it. If that dictionary doesn't contain `__name__`, it isn't automatically added unless you add it. If you run `exec(open('file.py').read())` in the global scope, the code in `file.py` runs with the *current* `__name__` (e.g., `"__main__"`).

**Sample Code:**

```python
code = "print(__name__)"
exec(code) # Output: __main__ (inherits current scope)

```

---

### 19. How does `__name__` relate to `__package__`?

**Answer:**
`__package__` explicitly stores the name of the package the module belongs to.

* If running as main script: `__name__` = `"__main__"`, `__package__` = `None`.
* If imported: `__name__` = `"pkg.mod"`, `__package__` = `"pkg"`.
Python uses `__name__` to derive `__package__` if `__package__` is not set, to facilitate relative imports.

---

### 20. Scenario: You see `if __name__ == "my_module":`. What is the developer doing?

**Answer:**
This is extremely rare and usually indicates a testing block meant to run only when the module is imported under a specific alias, or executed via `runpy` with a forced name. It is generally considered a "code smell" or a very specific hook for a custom test runner.

---

# Section 2 — Coding Questions (10 Total)

## Easy Level (3 Questions)

### 21. Create a "Dual-Mode" Script

**Question:** Write a script that calculates the area of a circle.

1. If imported, it provides a function `calculate_area(radius)`.
2. If run directly, it prompts the user for a radius input and prints the result.

**Solution:**

```python
import math

def calculate_area(radius):
    """Returns area of a circle given radius."""
    if radius < 0:
        raise ValueError("Radius cannot be negative")
    return math.pi * (radius ** 2)

# Line-by-line:
# 1. Define the core logic function.
# 2. Check for __name__ to decide if we are in 'Script Mode'.
if __name__ == "__main__":
    try:
        user_input = input("Enter radius: ")
        r = float(user_input)
        print(f"Area: {calculate_area(r):.2f}")
    except ValueError as e:
        print(f"Error: {e}")

```

**Common Mistakes:**
Putting the `input()` call outside the `if __name__ == "__main__":` block. This causes the import to hang waiting for user input.

**Real App Usage:**
Utility scripts in backend projects (e.g., a database cleaner that can be imported to be tested, or run manually by a dev).

---

### 22. Debugging Helper

**Question:** Create a function that prints the current module's name. Demonstrate how this output changes when run directly vs imported.

**Solution:**

```python
# file: debug_demo.py

def who_am_i():
    return f"I am running inside: {__name__}"

if __name__ == "__main__":
    print("Direct Execution Report:")
    print(who_am_i())
    # Output: I am running inside: __main__

```

*To verify, creating a second file:*

```python
# file: runner.py
import debug_demo
print("Import Report:")
print(debug_demo.who_am_i())
# Output: I am running inside: debug_demo

```

**Why interviewers ask this:**
Verifies you understand that `__name__` is context-dependent.

---

### 23. Package Entry Check

**Question:** Write a snippet that detects if a file is being run as part of a package or as a standalone script using `__package__` and `__name__`.

**Solution:**

```python
def check_context():
    if __name__ == "__main__" and __package__ is None:
        print("Running as a standalone script.")
    elif __package__:
        print(f"Running inside package: {__package__}")
    else:
        print("Imported as a top-level module.")

if __name__ == "__main__":
    check_context()

```

**Developer Scenario:**
Used in CLI tools to warn users if they are running a file incorrectly (e.g., `python file.py` instead of `python -m package.file`).

---

## Medium Level (7 Questions)

### 24. Test Suite Selector

**Question:** You have a file `math_ops.py`.

1. It contains heavy computation functions.
2. It contains a `unittest` TestSuite class.
3. Ensure that running the file executes the tests, but importing it only gives access to the functions without running tests.

**Solution:**

```python
import unittest

def heavy_computation(x):
    return x * x

class TestMathOps(unittest.TestCase):
    def test_computation(self):
        self.assertEqual(heavy_computation(5), 25)

if __name__ == "__main__":
    # This block is only executed when run directly
    print("Running Unit Tests...")
    unittest.main()

```

**Explanation:**
`unittest.main()` scans the module for `TestCase` classes and runs them. By placing it inside the guard, we prevent tests from running during a production import in a web server (e.g., Django/Flask).

**Time Complexity:** N/A (Architecture pattern).

---

### 25. The `multiprocessing` Guard Pattern

**Question:** Write a script that uses `multiprocessing.Pool` to square numbers. Specifically, demonstrate why the `if __name__` block is mandatory for this code to run on Windows without crashing.

**Solution:**

```python
from multiprocessing import Pool
import time

def square(x):
    return x * x

def run_processing():
    # If this logic was outside a function and outside the main block,
    # Windows would recursively spawn processes infinitely.
    with Pool(4) as p:
        print(p.map(square, [1, 2, 3]))

if __name__ == "__main__":
    print("Starting Main Process")
    run_processing()

```

**Line-by-line Explanation:**

1. On Windows, a new process imports the script to initialize.
2. If `run_processing()` was called at the top level, the new process would call it again.
3. This creates a new process, which calls it again... (infinite loop).
4. The `__name__` check breaks this cycle because in the child process, `__name__` is NOT `"__main__"`.

**Real App Usage:**
Data processing pipelines (ETL) using Python on Windows servers.

---

### 26. Custom CLI Entry Point

**Question:** Build a CLI tool `cli.py` that accepts arguments.

1. If run as `python cli.py --help`, it uses `argparse`.
2. If imported, it exposes a `run()` function that accepts arguments programmatically.

**Solution:**

```python
import argparse
import sys

def process_data(filename, verbose):
    if verbose:
        print(f"Processing {filename}...")
    # Simulation of work
    return {"status": "success", "file": filename}

def main(args=None):
    # If args are not passed (Command Line), parse sys.argv
    parser = argparse.ArgumentParser()
    parser.add_argument("filename")
    parser.add_argument("--verbose", action="store_true")
    
    parsed_args = parser.parse_args(args)
    process_data(parsed_args.filename, parsed_args.verbose)

if __name__ == "__main__":
    # Standard entry point
    main()

```

**Developer Scenario:**
This allows other scripts to reuse the logic:
`import cli; cli.main(['myfile.txt', '--verbose'])` without spawning a subprocess.

---

### 27. Relative Import Simulation

**Question:** Demonstrate a script that fails due to relative imports when run directly, and fix it using a "hack" to manipulate `sys.path` or `__package__` (for educational demonstration of why `__name__` matters).

**Solution:**

```python
# structure:
# /app
#   __init__.py
#   main.py
#   helper.py

# In main.py
import sys
import os

# The Problem: 'from . import helper' fails if run as 'python main.py'
# because __name__ is '__main__' and __package__ is None.

# The Fix (Simulating package run):
if __name__ == "__main__" and __package__ is None:
    # Add parent dir to sys.path
    current_dir = os.path.dirname(os.path.abspath(__file__))
    parent_dir = os.path.dirname(current_dir)
    sys.path.append(parent_dir)
    
    # Manually import helper without relative syntax
    import app.helper as helper
    print(f"Loaded helper via hack. Name: {__name__}")

# Standard relative import for when running as 'python -m app.main'
try:
    from . import helper
except ImportError:
    pass # Already handled above or genuine error

```

**Why interviewers ask this:**
Tests deep understanding of how Python resolves imports based on `__name__`.

---

### 28. Preventing Expensive Initialization

**Question:** You have a `database.py` that connects to a DB. This takes 5 seconds.
Ensure the connection happens *only* if the function is called, not when the file is imported. However, if run as a script, perform a connection test immediately.

**Solution:**

```python
import time

class DBConnection:
    def __init__(self):
        print("Connecting to DB (Expensive)...")
        time.sleep(1) # Simulating delay
        print("Connected.")

# Global variable is None initially
_db_instance = None

def get_db():
    global _db_instance
    if _db_instance is None:
        _db_instance = DBConnection()
    return _db_instance

if __name__ == "__main__":
    print("Script execution: Testing connection immediately.")
    db = get_db() # Forces connection
    print("Test Complete.")
else:
    print(f"Module {__name__} imported. Connection deferred.")

```

**Logic:**
This pattern (Lazy Loading) prevents `import database` from slowing down the startup time of the main application.

---

### 29. Dynamic Module Runner

**Question:** Write a function `run_by_name(module_str)` that imports a module dynamically and checks if it has a `main()` function. If so, run it. Print the `__name__` of the loaded module.

**Solution:**

```python
import importlib

def run_by_name(module_str):
    try:
        # Dynamically import
        mod = importlib.import_module(module_str)
        
        print(f"Loaded module: {mod.__name__}")
        
        if hasattr(mod, "main") and callable(mod.main):
            print(f"Executing main() of {module_str}...")
            mod.main()
        else:
            print(f"No main() function found in {module_str}")
            
    except ImportError:
        print(f"Could not import {module_str}")

if __name__ == "__main__":
    # Test on standard library 'json' (no main) or 'http.server' (has main logic usually in if block)
    run_by_name("os") 

```

**Developer Scenario:**
Used in plugin architectures where the main app loads plugins defined in a config file.

---

### 30. Quine-like Behavior

**Question:** Write a script that reads its own source code using `__file__`, but only prints it if `__name__` indicates it is running as a standalone script.

**Solution:**

```python
import sys

def print_source():
    try:
        with open(__file__, 'r') as f:
            print(f.read())
    except NameError:
        print("Cannot determine file path (REPL context?)")

if __name__ == "__main__":
    print(f"--- Source Code of {sys.argv[0]} ---")
    print_source()
else:
    print(f"I am imported as {__name__}. Source hidden.")

```

**Explanation:**
This combines filesystem access (`__file__`) with execution context (`__name__`).
**Real App Usage:**
Self-diagnostic scripts or installers that dump logs/config when run directly.
