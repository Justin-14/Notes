# Python Command Line Input — Interview Guide (30 Questions)

This guide covers interacting with the system via the Command Line Interface (CLI). It spans handling user prompts (`input`), parsing arguments (`argparse`, `sys.argv`), managing standard streams (`stdin`, `stdout`, `stderr`), and building robust CLI tools using the Standard Library.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What is the return type of the `input()` function in Python 3?

* **Expected Answer**: It always returns a **String** (`str`).
* **Clear Explanation**: Unlike Python 2’s `input()` (which evaluated code), Python 3’s `input()` reads a line from standard input and returns it as raw text. Even if the user types `123`, it returns `"123"`.
* **Common Mistakes**: Attempting math directly on the result without casting to `int()` or `float()`.
* **Sample Code**:
```python
age = input("Age: ")
print(type(age)) # <class 'str'>

```



#### 2. What is `sys.argv` and what does `sys.argv[0]` contain?

* **Expected Answer**: `sys.argv` is a list of strings representing the command-line arguments passed to the script. `sys.argv[0]` is the **name of the script** itself (or the path to it).
* **Clear Explanation**: The actual arguments provided by the user start at index `1`.
* **Comparison**: Similar to `argv` in C/C++.
* **Sample Code**:
```python
# Run: python script.py hello
import sys
print(sys.argv[0]) # script.py
print(sys.argv[1]) # hello

```



#### 3. Why use `argparse` instead of manually parsing `sys.argv`?

* **Expected Answer**: `argparse` handles **help generation** (`--help`), **type checking**, **default values**, and **error reporting** automatically.
* **Reasoning**: Parsing `sys.argv` manually requires writing complex logic to handle flags (like `-f` vs `--file`), optional arguments, and positional arguments. `argparse` standardizes this.
* **Why Interviewers Ask This**: To see if you reinvent the wheel or use standard libraries.

#### 4. What is the difference between `stdout` and `stderr`?

* **Expected Answer**:
* `stdout` (Standard Output): For the actual program output (data).
* `stderr` (Standard Error): For error messages, warnings, and logs.


* **Practical Use**: You can pipe `stdout` to a file (`> file.txt`) while still seeing errors on the screen, because errors are sent to `stderr`.
* **Sample Code**:
```python
import sys
print("Data", file=sys.stdout)
print("Error!", file=sys.stderr)

```



#### 5. How do you handle a "Silent" password input?

* **Expected Answer**: Use the `getpass` module.
* **Clear Explanation**: `getpass.getpass()` prompts the user without echoing characters to the screen, preventing shoulder-surfing or logging of passwords in terminal history.
* **Sample Code**:
```python
import getpass
pwd = getpass.getpass("Password: ")

```



#### 6. What is the Shebang line (`#!`)?

* **Expected Answer**: The first line of a script, starting with `#!`, which tells the Unix/Linux OS which interpreter to use to execute the file.
* **Common Example**: `#!/usr/bin/env python3`.
* **Reasoning**: It allows running `./script.py` directly without typing `python script.py`. It is ignored by Python on Windows (mostly) but critical on POSIX systems.

---

### Medium Level (Questions 7–20)

#### 7. `input()` vs `sys.stdin.read()`: Performance and Use Cases.

* **Question**: When would you use `sys.stdin.read()` instead of `input()`?
* **Expected Answer**:
* `input()`: Reads line-by-line, stripping the newline. Good for interactive prompts.
* `sys.stdin.read()`: Reads until EOF (End of File). Much faster for consuming **large piped data** (e.g., processing a 1GB log file passed via pipe).


* **Medium-Depth Reasoning**: `input()` has overhead because it decodes and handles prompt display for every line. `read()` or iterating `sys.stdin` directly is buffered and optimized for bulk data.

#### 8. How do you detect if data is being Piped to your script?

* **Question**: How does your script know if it's running interactively (`python script.py`) or receiving piped data (`echo "data" | python script.py`)?
* **Expected Answer**: Check `sys.stdin.isatty()`.
* Returns `True`: Interactive terminal.
* Returns `False`: Input is a pipe or file redirection.


* **Why Interviewers Ask This**: Essential for writing CLI tools that behave like standard Unix utilities (`ls`, `cat`, `grep`).
* **Sample Code**:
```python
import sys
if not sys.stdin.isatty():
    print("Reading from pipe...")
else:
    print("Interactive mode")

```



#### 9. Explain Exit Codes (`sys.exit`).

* **Question**: What is the significance of `sys.exit(0)` vs `sys.exit(1)`?
* **Expected Answer**:
* `0`: Success.
* Non-zero (`1-255`): Error / Failure.


* **Reasoning**: Shells and CI/CD pipelines check `$?` (the exit code). If your script crashes but returns 0, the pipeline thinks it succeeded.
* **Best Practice**: Always return non-zero on unrecoverable errors.

#### 10. `argparse`: `store_true` vs `store_const`.

* **Question**: What does `action='store_true'` do in `argparse`?
* **Expected Answer**: It creates a boolean flag.
* If the flag is present (`--verbose`), the variable is set to `True`.
* If absent, it defaults to `False`.


* **Comparison**: `store_const` sets the variable to a specific constant value provided in the code.
* **Sample Code**:
```python
parser.add_argument('--verbose', action='store_true')

```



#### 11. Handling `KeyboardInterrupt` (Ctrl+C).

* **Question**: When a user hits Ctrl+C, Python raises a generic traceback. How do you handle this gracefully?
* **Expected Answer**: Wrap the main logic in a `try/except KeyboardInterrupt` block.
* **Implementation**: Catch the exception and `sys.exit(0)` or print a clean "Aborted by user" message instead of showing a scary stack trace.
* **Why Interviewers Ask This**: User Experience (UX) for CLI tools.

#### 12. Positional vs Optional Arguments.

* **Question**: In `argparse`, how do you define a positional argument vs an optional flag?
* **Expected Answer**:
* **Positional**: Name starts *without* dashes. e.g., `add_argument("filename")`. (Required by default).
* **Optional**: Name starts with dashes. e.g., `add_argument("-f", "--file")`.


* **Nuance**: You can make positional arguments optional using `nargs='?'`.

#### 13. The `fileinput` module.

* **Question**: What is `fileinput` used for?
* **Expected Answer**: It iterates over lines from multiple input streams. It seamlessly handles the common pattern: "Read from files given as arguments, or read from stdin if no arguments."
* **Real Usage**: Recreating utilities like `grep` or `sed` in Python.
* **Sample Code**:
```python
import fileinput
for line in fileinput.input():
    print(line, end='')

```



#### 14. `shlex`: Parsing command strings safely.

* **Question**: You have a string `'run -f "my file.txt"'`. Splitting by space fails because of the quotes. How do you split it correctly?
* **Expected Answer**: Use `shlex.split()`.
* **Reasoning**: The `shlex` (shell lexical analysis) module handles quoted strings and escape characters exactly like a Unix shell does.
* **Sample Code**:
```python
import shlex
cmd = 'prog -f "foo bar"'
print(shlex.split(cmd)) # ['prog', '-f', 'foo bar']

```



#### 15. Sub-commands in CLI (Like `git commit`, `git push`).

* **Question**: How do you implement hierarchical commands using `argparse`?
* **Expected Answer**: Use the `add_subparsers()` method.
* **Mechanism**: It creates a special positional argument that selects which sub-parser (and specifically, which arguments) to use next.
* **Why Interviewers Ask This**: standard for building complex CLI applications.

#### 16. `sys.stderr.write` vs `print`.

* **Question**: Why use `sys.stderr.write("error\n")` instead of `print("error", file=sys.stderr)`?
* **Expected Answer**: Usually stylistic, but `write` implies raw output (no added newlines or spaces), giving precise control. `print` is higher-level.
* **Buffering**: `stderr` is usually unbuffered (appears immediately), whereas `stdout` is line-buffered.

#### 17. Environment Variables vs CLI Arguments Priority.

* **Question**: A script accepts a configuration via a flag `--host` or an env var `HOST`. Which should take precedence?
* **Expected Answer**: **CLI Arguments** generally override Environment Variables.
* **Hierarchy**:
1. CLI Arguments (Most specific, immediate user intent).
2. Environment Variables (Session context).
3. Config Files (Persistent defaults).
4. Hardcoded Defaults.


* **Why Interviewers Ask This**: Architectural best practices.

#### 18. What is `sys.executable`?

* **Question**: How do you find the exact path of the Python interpreter running your script?
* **Expected Answer**: `sys.executable`.
* **Use Case**: Useful when your script needs to spawn a subprocess using the *same* python environment (e.g., in a virtualenv).

#### 19. Type conversion in `argparse`.

* **Question**: `argparse` reads strings. How do you validate that an input is a valid integer or an existing file?
* **Expected Answer**: Use the `type=` parameter.
* `type=int`: Automatically converts or raises error.
* `type=argparse.FileType('r')`: Opens the file automatically.


* **Sample Code**:
```python
parser.add_argument('--count', type=int)

```



#### 20. Flushing Output (`flush=True`).

* **Question**: You print a progress bar `.` every second, but it only appears all at once at the end. Why?
* **Expected Answer**: Stdout is **buffered**. It waits for a newline `\n` or a buffer fill before flushing to the screen.
* **Fix**: Use `print(".", end="", flush=True)` or `sys.stdout.flush()`.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Basic Interactive Prompt

**Question**: Write a script that asks for a user's name. If the name is empty, keep asking. Once valid, print "Hello [Name]".

* **Solution**:
```python
def get_name():
    while True:
        try:
            name = input("Enter name: ").strip()
            if name:
                return name
            print("Name cannot be empty.")
        except EOFError:
            # Handle Ctrl+D gracefully
            return "Unknown"

name = get_name()
print(f"Hello {name}")

```


* **Explanation**: `strip()` removes accidental whitespace. The `while True` creates the retry loop. `EOFError` handles cases where user closes input stream.

#### 2. Sum Command Line Arguments

**Question**: Create a script that accepts any number of integers as arguments and prints their sum. `python sum.py 1 2 3` -> `6`.

* **Solution**:
```python
import sys

def main():
    # sys.argv[0] is the script name, slice from 1
    args = sys.argv[1:]

    total = 0
    for arg in args:
        try:
            total += int(arg)
        except ValueError:
            print(f"Skipping non-integer: {arg}", file=sys.stderr)

    print(total)

if __name__ == "__main__":
    main()

```


* **Complexity**: O(N) where N is number of args.
* **Mistake**: Forgetting to cast `int()`.

#### 3. Standard Input Filter

**Question**: Write a script that reads lines from stdin and prints only lines containing the word "ERROR". (A simplified `grep`).

* **Solution**:
```python
import sys

def grep_error():
    # Iterating sys.stdin reads line by line efficiently
    for line in sys.stdin:
        if "ERROR" in line:
            # line already has \n, so end='' prevents double spacing
            print(line, end='')

if __name__ == "__main__":
    grep_error()

```


* **Usage**: `cat logs.txt | python script.py`.

---

### Medium Level (Questions 4–10)

#### 4. Argparse with Types and Defaults

**Question**: Build a CLI tool that calculates the area of a rectangle. Arguments: `--width` (int, required), `--height` (int, default=10). Add a `--verbose` flag.

* **Solution**:
```python
import argparse

def calculate_area():
    parser = argparse.ArgumentParser(description="Calculate Rectangle Area")

    # Required argument (no --) vs Optional (with --)
    # Here we use Flags for everything as requested
    parser.add_argument("--width", type=int, required=True, help="Width of rectangle")
    parser.add_argument("--height", type=int, default=10, help="Height (default: 10)")
    parser.add_argument("-v", "--verbose", action="store_true", help="Detailed output")

    args = parser.parse_args()

    area = args.width * args.height

    if args.verbose:
        print(f"Calculating area for {args.width}x{args.height}...")

    print(area)

if __name__ == "__main__":
    calculate_area()

```


* **Why Interviewers Ask This**: Tests understanding of `required=True`, defaults, and boolean flags.

#### 5. Detecting Piped Input vs Arguments

**Question**: Write a script `process.py`.

1. If file paths are passed as args (`python process.py file.txt`), read them.
2. If data is piped (`echo "data" | python process.py`), read stdin.
3. If neither, print help.

* **Solution**:
```python
import sys

def process_data(source_name, data_stream):
    print(f"--- Processing {source_name} ---")
    content = data_stream.read()
    print(f"Size: {len(content)} bytes")

def main():
    # Check if arguments provided (excluding script name)
    if len(sys.argv) > 1:
        for filename in sys.argv[1:]:
            try:
                with open(filename, 'r') as f:
                    process_data(filename, f)
            except FileNotFoundError:
                print(f"Error: {filename} not found", file=sys.stderr)

    # Check if stdin has data (Piped)
    elif not sys.stdin.isatty():
        process_data("STDIN", sys.stdin)

    else:
        print("Usage: provide filenames or pipe data to stdin")
        sys.exit(1)

if __name__ == "__main__":
    main()

```


* **Key Concept**: `sys.stdin.isatty()` is the standard check for interactive vs piped mode.

#### 6. Simple REPL (Read-Eval-Print Loop)

**Question**: Create a mini-shell that accepts commands `greet [name]` and `exit`. It should loop forever until exit is typed.

* **Solution**:
```python
import sys

def repl():
    print("Welcome to MiniShell. Type 'exit' to quit.")

    while True:
        try:
            # Display prompt with flush to ensure it appears
            print("> ", end="", flush=True)
            command_line = sys.stdin.readline()

            # Handle EOF (Ctrl+D)
            if not command_line:
                break

            command_line = command_line.strip()
            parts = command_line.split()

            if not parts: continue

            cmd = parts[0].lower()

            if cmd == "exit":
                print("Goodbye!")
                break
            elif cmd == "greet":
                name = parts[1] if len(parts) > 1 else "User"
                print(f"Hello, {name}!")
            else:
                print(f"Unknown command: {cmd}")

        except KeyboardInterrupt:
            print("\nType 'exit' to quit.")

if __name__ == "__main__":
    repl()

```


* **Developer Scenario**: Building custom debug consoles for backend services.

#### 7. Password Logic with Confirmation

**Question**: Use `getpass` to accept a new password. Enforce:

1. Minimum length 6.
2. Must confirm password (type twice).
3. Max 3 attempts.

* **Solution**:
```python
import getpass
import sys

def set_password():
    attempts = 3
    while attempts > 0:
        p1 = getpass.getpass("New Password: ")
        if len(p1) < 6:
            print("Error: Too short (min 6 chars).")
            continue

        p2 = getpass.getpass("Confirm Password: ")
        if p1 == p2:
            print("Password set successfully.")
            return p1
        else:
            print("Error: Passwords do not match.")

        attempts -= 1

    print("Too many failed attempts.")
    sys.exit(1)

if __name__ == "__main__":
    set_password()

```



#### 8. Implementing Sub-commands (Git style)

**Question**: Implement a tool with structure:
`tool users add <name>`
`tool users list`

* **Solution**:
```python
import argparse

def handle_add(args):
    print(f"Adding user: {args.name}")

def handle_list(args):
    print("Listing all users...")

def main():
    parser = argparse.ArgumentParser()
    subparsers = parser.add_subparsers(dest="command", required=True)

    # Parent parser for 'users' (optional grouping, but let's do direct subcommands)
    # Level 1: Users
    users_parser = subparsers.add_parser("users", help="Manage users")
    users_subparsers = users_parser.add_subparsers(dest="action", required=True)

    # Level 2: add
    add_parser = users_subparsers.add_parser("add")
    add_parser.add_argument("name")
    add_parser.set_defaults(func=handle_add)

    # Level 2: list
    list_parser = users_subparsers.add_parser("list")
    list_parser.set_defaults(func=handle_list)

    args = parser.parse_args()
    # Execute the function assigned to the subcommand
    args.func(args)

if __name__ == "__main__":
    main()

```


* **Explanation**: `set_defaults(func=...)` is a clean pattern to map arguments to handler functions without massive if/else chains.

#### 9. The `fileinput` In-place Editing

**Question**: Write a script that replaces "foo" with "bar" in all files provided as arguments. Modify the files **in-place** (backup original).

* **Solution**:
```python
import fileinput
import sys

def replace_in_place():
    # fileinput.input with inplace=True redirects stdout back to the file
    # backup='.bak' saves the original
    with fileinput.input(files=sys.argv[1:], inplace=True, backup='.bak') as f:
        for line in f:
            # Print goes to the file, not the screen
            print(line.replace("foo", "bar"), end='')

if __name__ == "__main__":
    replace_in_place()

```


* **Real App Usage**: Bulk refactoring code or config files via CLI.

#### 10. Progress Bar with `\r` (Carriage Return)

**Question**: Simulate a download progress bar `[====>    ] 50%` that updates on the **same line**.

* **Solution**:
```python
import time
import sys

def progress_bar(total=20):
    print("Starting download...")
    for i in range(total + 1):
        time.sleep(0.1) # Simulate work

        percent = (i / total) * 100
        bar_length = 20
        filled = int(bar_length * i / total)
        bar = '=' * filled + '>' + ' ' * (bar_length - filled)

        # \r returns cursor to start of line
        # end='' prevents newline
        sys.stdout.write(f"\r[{bar}] {percent:.1f}%")
        sys.stdout.flush() # Force display update

    print("\nDone!")

if __name__ == "__main__":
    progress_bar()

```


* **Explanation**: `\r` (Carriage Return) moves the cursor back to the start of the line without moving down. Overwriting text creates the animation effect. `flush()` is mandatory here.
