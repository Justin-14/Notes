# Python NumPy Array Creation — Interview Guide (30 Questions)

This guide covers the myriad ways to instantiate NumPy arrays (`ndarrays`). It moves beyond the basic `np.array()` into specialized constructors for mathematical grids, random sampling, IO operations, and memory mapping.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What is the difference between `np.array()` and `np.asarray()`?

* **Expected Answer**:
* `np.array(data)`: By default, it makes a **copy** of the data. If you pass a NumPy array to it, it creates a duplicate in memory.
* `np.asarray(data)`: It performs a **shallow check**. If `data` is *already* an array of the correct type, it returns the original object (no copy). If not, it converts it.


* **Why Interviewers Ask This**: To see if you understand how to avoid unnecessary memory duplication in functions that accept array-like inputs.
* **Sample Code**:
```python
a = np.ones(5)
b = np.array(a)   # Copies data (New memory)
c = np.asarray(a) # Points to 'a' (Same memory)
print(a is b, a is c) # False, True

```



#### 2. `np.arange()` vs Python `range()`: Return types and floats.

* **Expected Answer**:
1. **Return Type**: `range()` returns a generator-like object (integers only). `np.arange()` returns a real array in memory.
2. **Floats**: `range()` does not support floats. `np.arange(0, 1, 0.1)` works, but is often discouraged due to floating-point drift. (Use `linspace` for floats).


* **Sample Code**:
```python
# range(0, 0.5, 0.1) # TypeError
print(np.arange(0, 0.5, 0.1)) # [0.  0.1 0.2 0.3 0.4]

```



#### 3. How do you create an array filled with a specific value (e.g., 7)?

* **Expected Answer**: Use `np.full(shape, fill_value)`.
* **Alternatives**: `np.ones(shape) * 7` or `np.zeros(shape) + 7` (less efficient because they perform an extra arithmetic step).
* **Sample Code**:
```python
arr = np.full((2, 2), 7)
# [[7, 7], [7, 7]]

```



#### 4. `np.eye()` vs `np.identity()`: Is there a difference?

* **Expected Answer**:
* `np.identity(n)`: Strictly creates a square N \times N identity matrix.
* `np.eye(N, M, k)`: More flexible. Can create rectangular matrices (N \times M) and can shift the diagonal using `k` (e.g., `k=1` is the super-diagonal).


* **Usage**: Use `identity` for math purity, `eye` for manipulating diagonals.

#### 5. What does `np.linspace(0, 10, 5)` generate?

* **Expected Answer**: It generates **5** linearly spaced points between 0 and 10 (inclusive).
* **Result**: `[0. , 2.5, 5. , 7.5, 10. ]`.
* **Math**: Step size = \frac{stop - start}{num - 1}.
* **Why Interviewers Ask This**: Essential for generating X-axis data for plotting graphs.

#### 6. Random Integers vs Random Floats (`randint` vs `rand`).

* **Expected Answer**:
* `np.random.randint(low, high, size)`: Returns integers from uniform distribution.
* `np.random.rand(d0, d1...)`: Returns floats in range `[0, 1)`.


* **Modern API**: The new Generator API (`default_rng`) is preferred over the legacy `np.random` functions in Python 3.
* **Sample Code**:
```python
rng = np.random.default_rng()
print(rng.integers(0, 10, size=5))

```



---

### Medium Level (Questions 7–20)

#### 7. `np.empty_like()` vs `np.zeros_like()`.

* **Question**: When would you use `_like` functions?
* **Expected Answer**: When you want to create a new array with the **same shape and dtype** as an existing array `a`.
* **Distinction**: `empty_like` is faster (uninitialized garbage), `zeros_like` is safer (cleared memory).
* **Sample Code**:
```python
a = np.array([[1, 2], [3, 4]])
b = np.zeros_like(a) # Shape (2,2), dtype int

```



#### 8. Creating Arrays from Raw Bytes (`frombuffer`).

* **Question**: You have a binary packet from a socket. How do you interpret it as an array of integers without copying?
* **Expected Answer**: `np.frombuffer(bytes_obj, dtype=...)`.
* **Memory**: This creates a read-only array that points directly to the memory of the bytes object. Zero-copy.
* **Sample Code**:
```python
b = b'\x01\x00\x02\x00' # Bytes for 1 and 2 (short int)
arr = np.frombuffer(b, dtype='int16') # [1, 2]

```



#### 9. `np.meshgrid()`: What is it used for?

* **Question**: Explain the output of `np.meshgrid(x, y)`.
* **Expected Answer**: It takes two 1D coordinate vectors and creates two 2D matrices representing the grid of coordinates.
* Matrix 1 repeats `x` along rows.
* Matrix 2 repeats `y` along columns.


* **Use Case**: Evaluating functions Z = f(X, Y) on a grid (contour plots).

#### 10. `np.diag()`: Creation vs Extraction.

* **Question**: This function behaves differently based on input. Explain.
* **Expected Answer**:
1. **Input 1D**: Creates a 2D matrix with the input on the diagonal (rest zeros).
2. **Input 2D**: Extracts the diagonal elements into a 1D array.


* **Sample Code**:
```python
print(np.diag([1, 2])) # [[1, 0], [0, 2]]
print(np.diag([[1, 2], [3, 4]])) # [1, 4]

```



#### 11. `np.tile()` vs `np.repeat()`: How do they differ?

* **Question**: Both repeat data. What is the structural difference?
* **Expected Answer**:
* `np.repeat(arr, n)`: Repeats **elements** adjacently (1, 1, 2, 2).
* `np.tile(arr, n)`: Repeats the **entire array** as a block (1, 2, 1, 2).


* **ASCII Diagram**:
```text
Input: [1, 2]
Repeat: [1, 1, 2, 2]
Tile:   [1, 2, 1, 2]

```



#### 12. `np.logspace()`: When to use?

* **Question**: You need to test an algorithm on scales 10^1, 10^2, \dots, 10^5. How do you generate this?
* **Expected Answer**: `np.logspace(start_exp, stop_exp, num)`.
* **Example**: `np.logspace(1, 5, 5)` generates `[10., 100., 1000., 10000., 100000.]`.

#### 13. `np.fromfunction()`: Functional Creation.

* **Question**: How do you create a 5x5 matrix where every element's value is `row + col` without loops?
* **Expected Answer**: Use `np.fromfunction(func, shape)`.
* **Mechanism**: NumPy calls `func(x, y)` passing coordinate arrays (indices) as arguments.
* **Sample Code**:
```python
def f(r, c): return r + c
arr = np.fromfunction(f, (3, 3))
# [[0, 1, 2], [1, 2, 3], ...]

```



#### 14. `np.tri()`, `np.tril()`, `np.triu()`.

* **Question**: How do you quickly create a lower-triangular mask (useful for Transformer attention masks)?
* **Expected Answer**:
* `np.tri(N)`: Creates a matrix with 1s on and below diagonal, 0s above.
* `np.tril(arr)`: Returns the Lower Triangle of an existing array.
* `np.triu(arr)`: Returns the Upper Triangle.



#### 15. The `order` parameter ('C' vs 'F').

* **Question**: In constructors like `np.zeros((2, 2), order='F')`, what does 'F' mean?
* **Expected Answer**: **Fortran-contiguous**.
* 'C' (Default): Row-major order (last index changes fastest).
* 'F': Column-major order (first index changes fastest).


* **Impact**: Affects performance when interfacing with Fortran libraries (BLAS/LAPACK) or iterating columns.

#### 16. `np.pad()`: Extending arrays.

* **Question**: How do you add a border of zeros (padding) around an image array?
* **Expected Answer**: `np.pad(array, pad_width, mode='constant', constant_values=0)`.
* **Why**: Critical in Convolutional Neural Networks (CNNs) to maintain spatial dimensions.

#### 17. Random State and Seeding.

* **Question**: Why use `np.random.seed(42)`?
* **Expected Answer**: Determinism. It ensures that random array generation produces the exact same sequence every time the script runs. Essential for debugging and reproducible ML research.

#### 18. Creating Arrays from Text Files (`genfromtxt`).

* **Question**: How do you load a CSV into a NumPy array handling missing values?
* **Expected Answer**: `np.genfromtxt('file.csv', delimiter=',', filling_values=-999)`.
* **Comparison**: `np.loadtxt` is faster/simpler but crashes on missing values. `genfromtxt` handles complexity.

#### 19. Vandermonde Matrix (`np.vander`).

* **Question**: What is `np.vander`?
* **Expected Answer**: It generates a matrix where columns are powers of the input vector (x^0, x^1, x^2 \dots).
* **Use Case**: Polynomial regression / fitting.

#### 20. `np.copy()`: Deep vs Shallow.

* **Question**: If `b = np.array(a)`, is that a deep copy? What about `b = np.copy(a)`?
* **Expected Answer**: Both create a deep copy of the **data**.
* **Nuance**: If `a` contains Python objects (dtype=object), `np.copy(a)` does a shallow copy of the *objects* (new array, pointers to same Python objects). To deep copy object arrays, you need `copy.deepcopy`.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Basic Conversion with Type Cast

**Question**: Convert the list `[1.5, 2.6, 3.7]` into a NumPy array of integers.

* **Solution**:
```python
import numpy as np

data = [1.5, 2.6, 3.7]
# dtype='int' truncates decimals (floor towards zero for positive)
arr = np.array(data, dtype='int32')
print(arr) # [1, 2, 3]

```



#### 2. Checkerboard Pattern (Zeros/Ones logic)

**Question**: Create an 8x8 matrix of zeros and fill the center 4x4 area with ones.

* **Solution**:
```python
import numpy as np

def center_fill():
    board = np.zeros((8, 8), dtype=int)
    # Slicing: rows 2 to 6, cols 2 to 6
    board[2:6, 2:6] = 1
    return board

print(center_fill())

```



#### 3. Random Normal Distribution

**Question**: Generate a 1D array of 1000 samples from a Standard Normal Distribution (\mu=0, \sigma=1).

* **Solution**:
```python
import numpy as np

def generate_noise():
    # Using modern Generator API
    rng = np.random.default_rng(seed=42)
    return rng.standard_normal(1000)

noise = generate_noise()
print(f"Mean: {noise.mean():.4f}") # Should be close to 0

```



---

### Medium Level (Questions 4–10)

#### 4. The `linspace` Trap (Endpoint)

**Question**: Generate 10 points between 0 and 1. Do NOT include 1.0.

* **Solution**:
```python
import numpy as np

# Method 1: endpoint=False (Cleanest)
arr1 = np.linspace(0, 1, 10, endpoint=False)

# Method 2: Generate 11, drop last
arr2 = np.linspace(0, 1, 11)[:-1]

print(arr1)

```


* **Why**: Useful for generating cyclic signals (sine waves) where the last point overlaps the start of next cycle.

#### 5. Creating a Bordered Matrix (`pad`)

**Question**: Given an N \times N matrix of ones, add a border of zeros around it.

* **Solution**:
```python
import numpy as np

def add_border(n):
    ones = np.ones((n, n), dtype=int)
    # Pad width=1 on all axes
    return np.pad(ones, pad_width=1, mode='constant', constant_values=0)

print(add_border(3))
# 5x5 matrix with 1s in center 3x3

```



#### 6. Repeating Patterns (`tile`)

**Question**: Create the sequence `[1, 2, 3, 1, 2, 3, 1, 2, 3]` without writing the numbers multiple times.

* **Solution**:
```python
import numpy as np

pattern = np.array([1, 2, 3])
# Repeat the whole block 3 times
result = np.tile(pattern, 3)
print(result)

```



#### 7. Diagonal Matrix from Range

**Question**: Create a 5x5 matrix where the diagonal elements are `[1, 2, 3, 4, 5]` and specific off-diagonals (k=1) are `[1, 1, 1, 1]`.

* **Solution**:
```python
import numpy as np

def diag_construct():
    # Main diagonal
    main = np.diag(np.arange(1, 6))
    # Super diagonal (k=1)
    upper = np.diag(np.ones(4), k=1)

    return main + upper

print(diag_construct())
# [[1. 1. 0. 0. 0.]
#  [0. 2. 1. 0. 0.] ...

```


* **Real Usage**: Constructing Finite Difference matrices for solving Differential Equations.

#### 8. String to Array (Parsing Simulation)

**Question**: You have a string `"1,2;3,4;5,6"`. Convert this into a (3, 2) NumPy matrix.

* **Solution**:
```python
import numpy as np

def parse_string_matrix(s):
    # Replace delimiters to make it space-separated for simplicity
    # or use np.fromstring with sep (if delimiters were uniform)

    # Robust method:
    # Split into rows, then items
    rows = s.split(';')
    data = [list(map(int, r.split(','))) for r in rows]

    return np.array(data)

s = "1,2;3,4;5,6"
print(parse_string_matrix(s))

```



#### 9. Sliding Window View (`as_strided`) - Advanced

**Question**: **Advanced**: Create a sliding window view of window size 2 over `[1, 2, 3, 4]`. Result: `[[1, 2], [2, 3], [3, 4]]`. Use `stride_tricks` (Zero Copy).

* **Solution**:
```python
import numpy as np
from numpy.lib.stride_tricks import as_strided

def sliding_window(arr, window_size):
    arr = np.asarray(arr)
    # Calculate new shape and strides
    # Shape: (Num_Windows, Window_Size)
    # Num_Windows = len - window + 1
    n_windows = arr.shape[0] - window_size + 1

    new_shape = (n_windows, window_size)

    # Strides: (Step to next window, Step to next element in window)
    # Both are equal to element size (4 bytes for int32)
    new_strides = (arr.strides[0], arr.strides[0])

    return as_strided(arr, shape=new_shape, strides=new_strides)

arr = np.array([1, 2, 3, 4])
print(sliding_window(arr, 2))

```


* **Why**: Essential for highly efficient convolution or time-series analysis without duplicating data.

#### 10. Generating Coordinate Grids (`indices`)

**Question**: Create an array of shape `(2, 3, 3)` where `arr[0]` contains the row indices and `arr[1]` contains the column indices for a 3x3 grid.

* **Solution**:
```python
import numpy as np

def grid_indices(n):
    # np.indices returns an array representing the indices of a grid
    return np.indices((n, n))

res = grid_indices(3)
print("Row Indices:\n", res[0])
print("Col Indices:\n", res[1])

```


* **Output**:
```text
Row: [[0, 0, 0], [1, 1, 1], [2, 2, 2]]
Col: [[0, 1, 2], [0, 1, 2], [0, 1, 2]]

```
