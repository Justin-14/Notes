# Python Arrays & User Input Parsing — Interview Guide (30 Questions)

This guide covers two distinct but related topics:

1. **The `array` module**: Python's efficient, typed array structure (standard library), often overlooked in favor of Lists.
2. **Parsing Arrays from Input**: Best practices for converting user input (strings) into numerical arrays, a staple of coding interviews and competitive programming.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What is the difference between a Python `list` and the `array` module?

* **Expected Answer**:
* **`list`**: Can hold items of **different types** (heterogeneous). It stores pointers to objects, adding memory overhead.
* **`array`**: Must hold items of the **same type** (homogeneous). It stores the actual values (packed bytes) continuously in memory, similar to C arrays.


* **Clear Explanation**: Use `array` when you need to store millions of integers/floats efficiently without the overhead of Python objects for every number.
* **Sample Code**:
```python
import array
# 'i' type code = signed int
arr = array.array('i', [1, 2, 3])
lst = [1, 2, 3]

```



#### 2. How do you convert a user input string `"1 2 3"` into a list of integers?

* **Expected Answer**: Use `split()` to break the string and `map()` (or list comprehension) to convert to `int`.
* **Idiom**: `list(map(int, input().split()))`.
* **Why Interviewers Ask This**: It is the standard boilerplate for almost every algorithmic problem input.
* **Sample Code**:
```python
raw = "1 2 3 4"
nums = list(map(int, raw.split()))
print(nums) # [1, 2, 3, 4]

```



#### 3. What are "Type Codes" in the `array` module?

* **Expected Answer**: Single characters that define the C-type of the array contents.
* **Common Codes**:
* `'b'`: signed char (1 byte)
* `'i'`: signed int (usually 2 or 4 bytes)
* `'f'`: float (4 bytes)
* `'d'`: double (8 bytes)
* `'u'`: unicode character (deprecated/changed in Py3)


* **Why**: Since `array` is typed, you must declare the type at creation.

#### 4. `input().split()` vs `input().split(' ')`.

* **Question**: Does `split()` behave differently than `split(' ')`?
* **Expected Answer**: **Yes**.
* `split()` (No args): Splits by **any whitespace** (space, tab, newline) and groups consecutive whitespace as a single separator. It removes empty strings from the result.
* `split(' ')`: Splits strictly by a single space. Consecutive spaces produce empty strings `''`.


* **Best Practice**: Always use `split()` for parsing user input to handle messy spacing safely.

#### 5. Can you resize a Python `array`?

* **Expected Answer**: Yes, similar to lists. You can `append()`, `extend()`, or slice assign.
* **Nuance**: While you *can*, if you need frequent resizing, a `list` is usually optimized for that (amortized growth). `array` is optimized for storage and data transfer.

#### 6. How do you handle multi-line array input (Matrix)?

* **Question**: How do you read a 3x3 matrix where each row is on a new line?
* **Expected Answer**: Use a loop or comprehension.
* **Sample Code**:
```python
rows = 3
matrix = [list(map(int, input().split())) for _ in range(rows)]

```



---

### Medium Level (Questions 7–20)

#### 7. Memory Efficiency: `array` vs `list`.

* **Question**: Approximately how much memory does `array('i', [1]*1000)` save compared to `[1]*1000`?
* **Expected Answer**:
* **List**: Stores 1000 pointers (8 bytes each) + 1000 Integer Objects (28 bytes each). Total ~36KB.
* **Array**: Stores 1000 integers (4 bytes each). Total ~4KB.


* **Ratio**: `array` is often 10x-20x more memory efficient for large sequences of numbers.

#### 8. Reading Binary Data directly into Array.

* **Question**: You have a binary file of integers. How do you load it fast?
* **Expected Answer**: Use `array.fromfile(f, n)`.
* **Explanation**: This reads `n` items directly from the file buffer into memory without parsing text or creating Python objects. It is extremely fast.
* **Sample Code**:
```python
import array
a = array.array('i')
with open('data.bin', 'rb') as f:
    # Reads 10 integers directly from bytes
    a.fromfile(f, 10)

```



#### 9. `sys.stdin.read` vs `input` for Massive Input.

* **Question**: You are solving a problem with 1,000,000 lines of input. `input()` is causing Time Limit Exceeded (TLE). Why?
* **Expected Answer**: `input()` does extra processing (stripping newlines, decoding prompts).
* **Solution**: Use `sys.stdin.read().split()`.
* It reads the entire input buffer into RAM at once (one syscall).
* `split()` tokenizes everything in one go.
* Iterate over the resulting list.



#### 10. `array` Endianness (Byte Swapping).

* **Question**: You received binary data from a machine with different Endianness. How does `array` help?
* **Expected Answer**: The `array` module has a `byteswap()` method.
* **Explanation**: It swaps the byte order of every element in the array in-place. Useful when networking or reading binary formats (Big Endian vs Little Endian).

#### 11. Why use `array` instead of NumPy?

* **Question**: NumPy is better. Why would you ever use the `array` module?
* **Expected Answer**:
1. **Standard Library**: `array` is built-in. NumPy requires installation (pip). Useful in restricted environments (embedded Python, strict AWS Lambda environments).
2. **Simplicity**: If you just need to store numbers and don't need matrix math (`dot`, `matmul`), `array` is lightweight.



#### 12. Input Parsing: `int` vs `float` pitfalls.

* **Question**: What happens if you use `map(int, input().split())` but the user types `1.0 2.0`?
* **Expected Answer**: `ValueError`.
* **Reasoning**: `int("1.0")` fails in Python. You must convert to float first: `int(float("1.0"))` or just use `map(float, ...)`.

#### 13. The `buffer_info()` method.

* **Question**: How do you pass a Python `array` to a C-extension?
* **Expected Answer**: Use `arr.buffer_info()`.
* **Return**: It returns a tuple `(memory_address, length)`.
* **Usage**: This allows C functions (via `ctypes` or `CPython` API) to access the raw memory pointer directly for zero-copy operations.

#### 14. Convert Array to Bytes and Back.

* **Question**: How do you serialize an array to send over a network socket?
* **Expected Answer**:
* **To Bytes**: `arr.tobytes()` returns a raw bytes object.
* **From Bytes**: `arr.frombytes(b_data)` appends raw bytes to the array.



#### 15. Slicing an `array`.

* **Question**: If you slice `a = array('i', [1, 2, 3]); b = a[:2]`, is `b` a view or a copy?
* **Expected Answer**: It is a **Copy**.
* **Comparison**: This matches List behavior. (NumPy slices are Views).
* **Implication**: Slicing huge arrays doubles memory usage momentarily. Use `memoryview(a)` for zero-copy slicing.

#### 16. Fast I/O Template (Competitive Programming).

* **Question**: What is the standard Python boilerplate for fastest I/O?
* **Expected Answer**:
```python
import sys
input = sys.stdin.readline
# Now input() is linked to the faster readline method

```


* **Why**: `sys.stdin.readline` is faster than built-in `input()` because it avoids the interactive prompt check logic.

#### 17. Safe Input Parsing (Generator).

* **Question**: How do you process an infinite stream of numbers separated by spaces without loading all into RAM?
* **Expected Answer**: Write a generator that reads chunks.
* **Code Snippet**:
```python
def read_ints():
    for line in sys.stdin:
        for token in line.split():
            yield int(token)

```



#### 18. Limitations of `array`.

* **Question**: Can you create an array of Arrays? `array('i', [[1], [2]])`?
* **Expected Answer**: **No**.
* **Reasoning**: The `array` module is strictly 1-Dimensional. For multi-dimensional data, you must use a List of Lists, a flattened 1D array with math logic, or NumPy.

#### 19. `array.typecode` vs `array.itemsize`.

* **Question**: How do you check what's inside an array and how much RAM one item takes?
* **Expected Answer**:
* `.typecode`: Returns the char (e.g., `'i'`).
* `.itemsize`: Returns the number of bytes per item (e.g., 4).


* **Total RAM**: `len(arr) * arr.itemsize`.

#### 20. Does `array` support `sort()`?

* **Question**: Can you run `arr.sort()`?
* **Expected Answer**: No, `array` objects do not have a `.sort()` method (unlike lists).
* **Workaround**: Use `sorted(arr)` which returns a new **List**, or standard library functions that work on iterables, but to sort in-place, you'd need to write your own swap logic or convert to list and back.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Basic User Input to List

**Question**: Read a single line of space-separated integers from the user and print the sum.
Input: `10 20 30`
Output: `60`

* **Solution**:
```python
def sum_input():
    try:
        # input() reads line, split() breaks it, map() converts types
        nums = list(map(int, input().split()))
        print(sum(nums))
    except ValueError:
        print("Invalid input")

if __name__ == "__main__":
    sum_input()

```


* **Complexity**: Time O(N).

#### 2. Fixed-Type Array Creation

**Question**: Create an `array` of integers from -5 to 5. Append the number 100. Print the memory address and length.

* **Solution**:
```python
import array

def array_ops():
    # 'i' is signed integer (usually 2 bytes minimum, often 4)
    arr = array.array('i', range(-5, 6))
    arr.append(100)

    # buffer_info returns (address, length)
    addr, length = arr.buffer_info()
    print(f"Address: {addr}, Count: {length}")
    print(f"Contents: {arr.tolist()}")

array_ops()

```



#### 3. Read N Lines of Arrays

**Question**: The first line contains `N`. The next `N` lines each contain a list of numbers. Store them in a list of lists.

* **Solution**:
```python
def read_matrix():
    try:
        n_line = input()
        if not n_line: return
        n = int(n_line)

        matrix = []
        for _ in range(n):
            # Comprehension for each row
            row = list(map(int, input().split()))
            matrix.append(row)

        print(f"Read {len(matrix)} rows.")
    except ValueError:
        pass

# read_matrix()

```



---

### Medium Level (Questions 4–10)

#### 4. Flatten User Matrix to 1D Array

**Question**: User inputs a 3x3 matrix. Store it in a **single 1D** `array` for cache efficiency. Access element `(r, c)` using math.

* **Solution**:
```python
import array

def matrix_to_1d():
    rows, cols = 3, 3
    # Initialize empty signed int array
    flat_matrix = array.array('i')

    print("Enter 3 lines of 3 numbers:")
    for _ in range(rows):
        line_data = list(map(int, input().split()))
        flat_matrix.extend(line_data)

    def get_element(r, c):
        # 2D to 1D mapping: index = r * width + c
        index = r * cols + c
        return flat_matrix[index]

    print(f"Element at (1,1): {get_element(1, 1)}")

# matrix_to_1d()

```


* **Why Interviewers Ask This**: Tests understanding of how 2D structures map to linear memory.

#### 5. Efficient File-to-Array Loader

**Question**: Write a function that creates a file containing 10,000 binary integers, then reads them back into an `array` instantaneously.

* **Solution**:
```python
import array
import os

def binary_io():
    data = array.array('i', range(10000))
    filename = 'temp_ints.bin'

    # 1. Write fast
    with open(filename, 'wb') as f:
        data.tofile(f)

    # 2. Read fast
    loaded = array.array('i')
    with open(filename, 'rb') as f:
        # We need to know file size to know how many items to read
        # or rely on file size / itemsize
        file_size = os.path.getsize(filename)
        count = file_size // loaded.itemsize
        loaded.fromfile(f, count)

    print(f"Loaded {len(loaded)} items. First: {loaded[0]}, Last: {loaded[-1]}")
    os.remove(filename)

binary_io()

```



#### 6. Safe Parsing with Error Skipping

**Question**: Parse a line of input "10 20 apple 30". It should result in `[10, 20, 30]`. Ignore non-integers.

* **Solution**:
```python
def parse_safe(raw_string):
    tokens = raw_string.split()
    valid_nums = []

    for t in tokens:
        try:
            valid_nums.append(int(t))
        except ValueError:
            continue # Skip invalid tokens

    return valid_nums

print(parse_safe("10 20 apple 30"))

```



#### 7. Fast I/O for Competitive Programming

**Question**: Implement a template that reads all integers from `stdin` (spanning multiple lines) into a single list using `sys.stdin.read`.

* **Solution**:
```python
import sys

def fast_read_all():
    # sys.stdin.read() grabs ALL text from input until EOF
    input_data = sys.stdin.read()

    if not input_data:
        return []

    # split() creates tokens from all lines at once
    iterator = map(int, input_data.split())

    return list(iterator)

# Usage: echo -e "1 2\n3 4" | python script.py

```


* **Efficiency**: This is the fastest method in Python to parse bulk numeric input.

#### 8. Bytes to Array (Network Simulation)

**Question**: You receive a packet of bytes `b'\x01\x00\x02\x00'`. Treat this as Little Endian short integers (2 bytes). Convert to array.

* **Solution**:
```python
import array
import sys

def parse_packet(byte_data):
    # 'h' is signed short (2 bytes)
    arr = array.array('h')
    arr.frombytes(byte_data)

    # Handle Endianness if system does not match network
    if sys.byteorder == 'big': 
        arr.byteswap()

    return arr.tolist()

data = b'\x01\x00\x02\x00' # Represents [1, 2] in Little Endian
print(parse_packet(data))

```



#### 9. Validating Array Constraints

**Question**: User inputs "10 20 30". Ensure all numbers are between 0 and 100. If any fail, return None.

* **Solution**:
```python
def get_valid_scores():
    try:
        scores = list(map(int, input("Enter scores: ").split()))

        # Use all() for cleaner validation logic
        if not all(0 <= s <= 100 for s in scores):
            print("Error: Scores must be 0-100")
            return None

        return scores
    except ValueError:
        print("Error: Non-numeric input")
        return None

```



#### 10. `memoryview` Slicing (Zero Copy)

**Question**: You have a huge array. You want to pass the second half of it to a function without copying the memory.

* **Solution**:
```python
import array

def process_chunk(view):
    # view behaves like an array/list but points to original memory
    # Modifying view modifies original!
    view[0] = 999 

data = array.array('i', range(1000000))

# Create a memoryview slice (Zero Copy)
mid = len(data) // 2
second_half_view = memoryview(data)[mid:]

process_chunk(second_half_view)

# Check original array
print(data[mid]) # 999

```


* **Why Interviewers Ask This**: Shows deep knowledge of Python memory management essential for high-performance data processing.
