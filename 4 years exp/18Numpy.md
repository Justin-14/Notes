# Python NumPy — Interview Guide (30 Questions)

This guide covers **NumPy**, the foundation of Python's scientific stack. It focuses on vectorization, memory management (`ndarrays`), broadcasting rules, and performance optimization compared to standard Python lists.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What is the main advantage of a NumPy array over a Python List?

* **Expected Answer**: **Performance** (Speed and Memory).
* **Clear Explanation**:
* **Memory**: NumPy arrays are densely packed contiguous blocks of memory (like C arrays). Python lists are arrays of pointers to scattered objects.
* **Speed**: NumPy operations are **Vectorized** (implemented in C), avoiding Python loops and overhead. They often utilize CPU SIMD instructions.


* **Why Interviewers Ask This**: To verify you understand *why* we introduce an external dependency instead of using built-ins.
* **Sample Code**:
```python
import numpy as np
import sys
l = [1] * 1000
a = np.ones(1000, dtype='int8')
print(sys.getsizeof(l)) # ~8000 bytes (pointers) + object overhead
print(a.nbytes)         # 1000 bytes (exact)

```



#### 2. What is `dtype` and why is it mandatory in NumPy?

* **Expected Answer**: `dtype` (Data Type) defines the type and size of the elements in the array. Since NumPy arrays are homogeneous (all elements match), `dtype` tells the CPU exactly how many bytes to read per step.
* **Common Types**: `int32`, `int64`, `float32`, `float64`, `bool`.
* **Sample Code**:
```python
arr = np.array([1, 2, 3], dtype='float32')
print(arr.dtype) # float32

```



#### 3. `np.zeros()` vs `np.empty()`: Which is faster?

* **Expected Answer**: `np.empty()` is marginally faster.
* **Reasoning**:
* `np.zeros()`: Allocates memory **and** initializes all values to 0.
* `np.empty()`: Allocates memory but **does not** initialize it. The array will contain garbage values (whatever bits happened to be at that memory address).


* **Risk**: Only use `empty` if you plan to overwrite every single element immediately, otherwise you risk processing garbage data.

#### 4. Explain the difference between `np.arange()` and `np.linspace()`.

* **Expected Answer**:
* `np.arange(start, stop, step)`: You define the **step size**. The number of elements is calculated automatically.
* `np.linspace(start, stop, num)`: You define the **number of elements**. The step size is calculated automatically.


* **Why Interviewers Ask This**: `linspace` is preferred for generating coordinate grids where you need precise endpoint inclusion.
* **Sample Code**:
```python
print(np.arange(0, 10, 2))  # [0, 2, 4, 6, 8]
print(np.linspace(0, 10, 3)) # [0., 5., 10.]

```



#### 5. How do you find the shape and dimension of an array?

* **Expected Answer**: Attributes `.shape` (tuple of dimensions) and `.ndim` (integer count of dimensions).
* **Context**: A vector has `ndim=1`, shape `(N,)`. A matrix has `ndim=2`, shape `(R, C)`.
* **Sample Code**:
```python
x = np.zeros((3, 4))
print(x.shape) # (3, 4)
print(x.ndim)  # 2

```



#### 6. What operator is used for Matrix Multiplication?

* **Expected Answer**: The `@` operator (introduced in Python 3.5 for this exact purpose).
* **Legacy**: `np.dot(a, b)` or `a.dot(b)`.
* **Distinction**: The `*` operator performs **element-wise** multiplication (Hadamard product), not matrix multiplication.
* **Sample Code**:
```python
A = np.array([[1, 2], [3, 4]])
B = np.eye(2)
print(A @ B) # Matrix Mul
print(A * B) # Element-wise Mul

```



---

### Medium Level (Questions 7–20)

#### 7. Explain "Broadcasting" in NumPy.

* **Expected Answer**: Broadcasting is the mechanism that allows NumPy to perform arithmetic operations on arrays of **different shapes**.
* **The Rule**: Two dimensions are compatible when:
1. They are equal, OR
2. One of them is 1.


* **Mechanism**: The dimension with size 1 is conceptually "stretched" or copied to match the other dimension without actual memory replication.
* **Why Interviewers Ask This**: This is the single most important concept in NumPy programming.

#### 8. `view()` vs `copy()`: Memory Implications.

* **Question**: If `b = a[0:5]`, does modifying `b` affect `a`?
* **Expected Answer**: **Yes**. Slicing returns a **View**, not a Copy.
* **Reasoning**: A view shares the same underlying data buffer but interprets it differently (different shape/strides). This is efficient (no data copying).
* **Forcing Copy**: Use `b = a[0:5].copy()`.
* **Contrast**: Python Lists always copy when sliced (`b = list_a[:]`). NumPy defaults to views.

#### 9. What are "Strides" in an array?

* **Question**: How does NumPy map multidimensional indices `(row, col)` to a 1D memory block?
* **Expected Answer**: Using `strides`. It is a tuple of bytes to step in each dimension when traversing memory.
* **Example**: For a 3x3 `int32` (4 bytes) array:
* To move to the next column: Step 4 bytes.
* To move to the next row: Step 12 bytes (3 * 4).
* Strides = `(12, 4)`.


* **Why Interviewers Ask This**: Deep internals. It explains why transposing `T` is O(1) (it just flips the strides, moving no data).

#### 10. Vectorization vs Loops.

* **Question**: Why is `np.sum(arr)` faster than `sum(arr)`?
* **Expected Answer**:
* `sum(arr)`: Python iterates elements one by one, checks type, unboxes the number, adds it, checks for overflow.
* `np.sum(arr)`: Calls optimized C code. No type checking per element. Can use **SIMD** (Single Instruction, Multiple Data) to add 4 or 8 numbers in a single CPU cycle.



#### 11. Advanced Indexing: Boolean vs Fancy Indexing.

* **Question**: How do `a[a > 5]` and `a[[1, 3, 5]]` work?
* **Expected Answer**:
* **Boolean Indexing** (`a > 5`): Creates a mask. Returns a **Copy** of elements where the mask is True.
* **Fancy Indexing** (`[1, 3]`): Using integer arrays as indices. Returns a **Copy** of elements at those specific positions.


* **Critical Note**: Unlike slicing, advanced indexing always returns a copy, never a view.

#### 12. Handling `NaN` (Not a Number).

* **Question**: What is the result of `np.sum([1, np.nan, 2])`? How do you fix it?
* **Expected Answer**: The result is `nan`. NaN is viral; any operation touching it becomes NaN.
* **Fix**: Use NaN-safe functions like `np.nansum()`, `np.nanmean()`, etc., which treat `nan` as missing values (ignoring them).

#### 13. Understanding the `axis` parameter.

* **Question**: In a 2D matrix, what does `sum(axis=0)` vs `sum(axis=1)` do?
* **Expected Answer**:
* `axis=0`: Collapses the rows. Performs operation **down the columns**. (Returns shape `(cols,)`).
* `axis=1`: Collapses the columns. Performs operation **across the rows**. (Returns shape `(rows,)`).


* **Mnemonic**: "The axis you specify is the one that goes away."

#### 14. `flatten()` vs `ravel()`.

* **Question**: Both convert multi-dimensional arrays to 1D. Difference?
* **Expected Answer**:
* `ravel()`: Returns a **View** if possible (faster).
* `flatten()`: Always returns a **Deep Copy**.


* **Usage**: Default to `ravel()` for memory efficiency unless you specifically need an independent copy.

#### 15. Stacking Arrays (`hstack`, `vstack`, `concatenate`).

* **Question**: How do you join two arrays?
* **Expected Answer**:
* `vstack`: Stacks vertically (adds rows).
* `hstack`: Stacks horizontally (adds columns).
* `concatenate((a, b), axis=...)`: General purpose.


* **Constraint**: Dimensions other than the concatenation axis must match exactly.

#### 16. What is a "Structured Array"?

* **Question**: Can NumPy handle heterogeneous data like a CSV with names (str) and ages (int)?
* **Expected Answer**: Yes, using Structured Arrays.
* **Mechanism**: You define a complex `dtype` like `[('name', 'U10'), ('age', 'i4')]`.
* **Memory**: It stores records contiguously (like a `struct` in C).
* **Sample Code**:
```python
data = np.array([('Alice', 25)], dtype=[('name', 'U10'), ('age', 'i4')])
print(data['age']) # [25]

```



#### 17. `np.newaxis` / `None` in slicing.

* **Question**: What does `arr[:, np.newaxis]` do?
* **Expected Answer**: It increases the dimension of the array by 1.
* **Usage**: Turns a 1D vector `(N,)` into a column vector `(N, 1)` or row vector `(1, N)` to facilitate broadcasting.
* **Alternative**: `arr.reshape(-1, 1)`.

#### 18. Performance: Row-Major (C) vs Column-Major (Fortran).

* **Question**: NumPy arrays are Row-Major by default. Why does this matter for performance?
* **Expected Answer**: CPU Caching.
* Iterating along rows (`axis=1`) accesses contiguous memory addresses, which is cache-friendly.
* Iterating along columns (`axis=0`) jumps across memory (stride step), causing cache misses.


* **Optimization**: Always try to structure operations to access memory sequentially.

#### 19. The `where` function.

* **Question**: Explain `np.where`.
* **Expected Answer**:
1. `np.where(condition, x, y)`: Vectorized "if-else". Returns elements from `x` where condition is True, `y` otherwise.
2. `np.where(condition)`: Returns indices where condition is True.


* **Sample Code**:
```python
arr = np.array([1, 10, 2])
# Replace outliers > 5 with 0
clean = np.where(arr > 5, 0, arr) # [1, 0, 2]

```



#### 20. `argmax` / `argmin`.

* **Question**: How do you find the index of the maximum value?
* **Expected Answer**: `np.argmax(arr)`.
* **Usage**: Essential in Machine Learning (e.g., finding the predicted class from a probability vector).

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Filter Elements by Condition

**Question**: Extract all odd numbers from `arr` and replace them with -1.

* **Solution**:
```python
import numpy as np

def replace_odds(arr):
    # 1. Create mask
    mask = arr % 2 == 1
    # 2. Use mask to assignment
    arr[mask] = -1
    return arr

a = np.array([1, 2, 3, 4, 5])
print(replace_odds(a)) # [-1, 2, -1, 4, -1]

```


* **Explanation**: Boolean indexing allows assigning values to the subset of the array where the condition is true.

#### 2. Create and Reshape

**Question**: Create a 1D array of numbers from 0 to 8. Reshape it into a 3x3 matrix.

* **Solution**:
```python
def make_matrix():
    # arange generates 0..8
    arr = np.arange(9)
    # Reshape to 3x3
    return arr.reshape(3, 3)

print(make_matrix())

```


* **Common Mistake**: `reshape` fails if total elements don't match (e.g., reshape 9 elements to 2x4).

#### 3. Normalize Vector (Min-Max)

**Question**: Normalize a random 5x5 matrix so that the minimum value is 0 and the maximum is 1.

* **Solution**:
```python
def normalize(matrix):
    min_val = matrix.min()
    max_val = matrix.max()

    if max_val == min_val: return matrix # Avoid div by zero

    return (matrix - min_val) / (max_val - min_val)

x = np.random.random((5, 5))
print(normalize(x))

```


* **Explanation**: This uses scalar broadcasting. `matrix - scalar` subtracts from every element.

---

### Medium Level (Questions 4–10)

#### 4. The "Nearest Value" Search

**Question**: Given a random array `arr` and a scalar `val`, find the value in `arr` that is closest to `val`.

* **Solution**:
```python
def find_nearest(array, value):
    array = np.asarray(array)
    # 1. Calculate absolute difference map
    diff = np.abs(array - value)
    # 2. Find index of the minimum difference
    idx = diff.argmin()
    # 3. Return element at that index
    return array[idx]

arr = np.array([1.2, 5.5, 9.9, 3.1])
print(find_nearest(arr, 3.0)) # 3.1

```


* **Complexity**: O(N) Time.

#### 5. One-Hot Encoding

**Question**: Convert an integer array `[1, 0, 3]` into a One-Hot encoded matrix.
Assume max classes = 4.

* **Solution**:
```python
def one_hot(labels, classes):
    # Create zero matrix of shape (N, classes)
    output = np.zeros((labels.size, classes), dtype=int)

    # Fancy Indexing:
    # Rows: 0 to N
    # Cols: The values in 'labels'
    output[np.arange(labels.size), labels] = 1

    return output

labels = np.array([1, 0, 3])
print(one_hot(labels, 4))
# [[0 1 0 0]
#  [1 0 0 0]
#  [0 0 0 1]]

```


* **Real App Usage**: Preparing categorical data for Neural Networks.

#### 6. Image Grayscale Conversion (Broadcasting)

**Question**: You have an image array of shape `(Height, Width, 3)` representing RGB. Convert it to Grayscale using formula: Y = 0.299R + 0.587G + 0.114B. Use `dot` product.

* **Solution**:
```python
def to_grayscale(img):
    # Weights vector
    weights = np.array([0.299, 0.587, 0.114])

    # Dot product along the last axis (Color channels)
    # img: (H, W, 3) @ weights: (3,) -> (H, W)
    return np.dot(img, weights)

img = np.random.random((100, 100, 3))
gray = to_grayscale(img)
print(gray.shape) # (100, 100)

```



#### 7. Moving Average (Convolution)

**Question**: Compute the moving average of an array over a window size `n`.

* **Solution**:
```python
def moving_average(a, n=3):
    # Using cumulative sum is an O(N) trick
    ret = np.cumsum(a, dtype=float)

    # Logic: (Sum[i] - Sum[i-n]) / n
    ret[n:] = ret[n:] - ret[:-n]
    return ret[n - 1:] / n

# Alternative: np.convolve(a, np.ones(n)/n, mode='valid')

arr = np.arange(10)
print(moving_average(arr, 3))

```


* **Why Interviewers Ask This**: Tests algorithmic thinking combined with NumPy tools (`cumsum`).

#### 8. Subtract Mean (Demean) - Broadcasting

**Question**: Given a matrix `(N, M)`, subtract the mean of each **row** from that row.

* **Solution**:
```python
def demean_rows(matrix):
    # 1. Calculate means of rows (axis=1) -> Shape (N,)
    row_means = matrix.mean(axis=1)

    # 2. Reshape to (N, 1) to enable broadcasting against (N, M)
    row_means = row_means[:, np.newaxis] 
    # or row_means.reshape(-1, 1)

    # 3. Broadcast subtract
    return matrix - row_means

m = np.array([[1, 2, 3], [4, 5, 6]])
# Means: [2, 5]
print(demean_rows(m)) 
# [[-1, 0, 1], [-1, 0, 1]]

```


* **Common Mistake**: Forgetting to reshape the mean vector. `(N, M) - (N,)` fails or broadcasts wrongly. It must be `(N, M) - (N, 1)`.

#### 9. Replace Max Value with 0

**Question**: Replace the maximum value in each row with 0.

* **Solution**:
```python
def zero_out_max(matrix):
    # Find indices of max in each row
    # argmax returns indices [0, 2, 1...]
    max_indices = np.argmax(matrix, axis=1)

    # Create row indices [0, 1, 2...]
    rows = np.arange(matrix.shape[0])

    # Assign 0 at (row, max_col)
    matrix[rows, max_indices] = 0
    return matrix

m = np.array([[10, 2, 3], [4, 50, 6]])
print(zero_out_max(m))
# [[0, 2, 3], [4, 0, 6]]

```



#### 10. Checkers Board Pattern

**Question**: Create an 8x8 matrix with a checkerboard pattern (0s and 1s) using slicing.

* **Solution**:
```python
def checkerboard(n=8):
    board = np.zeros((n, n), dtype=int)

    # Fill rows with offset slicing
    # board[row_start::step, col_start::step]

    # Rows 0, 2, 4... -> Set odd columns to 1
    board[0::2, 1::2] = 1

    # Rows 1, 3, 5... -> Set even columns to 1
    board[1::2, 0::2] = 1

    return board

print(checkerboard(4))
# [[0 1 0 1]
#  [1 0 1 0]
#  [0 1 0 1]
#  [1 0 1 0]]

```
