# Python MATRIX— Interview Guide (30 Questions)

## Introduction

This guide focuses on **Matrices** (2D Lists) in Python. While Python does not have a built-in primitive "matrix" type, they are universally implemented as **Lists of Lists**. This guide covers memory management, traversal optimization, standard algorithms, and common pitfalls expected of a developer with 4 years of experience.

---

# SECTION 1 — THEORY QUESTIONS (20 TOTAL)

## Part A: Easy Level (6 Questions)

### 1. How is a Matrix represented in standard Python, and what are its memory implications?

**Expected Answer:**
A matrix in standard Python is a "List of Lists". It is not a contiguous block of memory (like a C array or NumPy array). It is a list containing references (pointers) to other list objects, which in turn contain references to the actual data objects (integers, strings, etc.).

**Explanation:**
Python lists are dynamic arrays of pointers. A 2D list `matrix = [[1, 2], [3, 4]]` is essentially a list of pointers, where `matrix[0]` points to a distinct list object `[1, 2]`.

**Reasoning:**
Because the inner lists are separate objects scattered in the heap, accessing `matrix[i][j]` involves double pointer dereferencing. This structure allows for "jagged arrays" (rows of different lengths) but lacks the cache locality of C-style contiguous arrays.

**Time & Space Complexity:**

* Access: 
* Overhead: High (pointers + PyObject overhead).

**Common Mistakes:**
Assuming rows are stored next to each other in memory.

**Why interviewers ask this:**
To test understanding of Python's memory model vs. low-level languages.

**ASCII Diagram:**

```text
matrix (List Object)
+-------+       +-------------------+
| ptr 0 | ----> | Inner List (Row 0)| -> [1, 2]
+-------+       +-------------------+
| ptr 1 | ----> | Inner List (Row 1)| -> [3, 4]
+-------+       +-------------------+

```

**Sample Code:**

```python
# A 2x2 Matrix representation
matrix = [
    [1, 2],
    [3, 4]
]

# Accessing memory addresses to prove separation
print(f"Address of Row 0: {id(matrix[0])}")
print(f"Address of Row 1: {id(matrix[1])}")

```

**Line-by-line Explanation:**

1. `matrix` is defined as a list containing two lists.
2. `id(matrix[0])` prints the memory address of the first inner list, showing it is a distinct object.

---

### 2. What is the difference between `[[0]*3]*3` and `[[0]*3 for _ in range(3)]`?

**Expected Answer:**
`[[0]*3]*3` creates a list containing 3 references to the **same** inner list object. Modifying one element `matrix[0][0]` will change `matrix[1][0]` and `matrix[2][0]`. The list comprehension method creates 3 **distinct** inner lists.

**Explanation:**
The `*` operator on lists performs a shallow copy of references. The comprehension evaluates the expression 3 times, creating new objects each time.

**Reasoning:**
This is the "Aliasing Bug". It is critical for initialization.

**Common Mistakes:**
Using `[[0]*N]*N` for DP tables or grid initialization and encountering ghost updates.

**Why interviewers ask this:**
This is the #1 "Gotcha" in Python matrix manipulation.

**Sample Code:**

```python
# BAD Initialization
bad_matrix = [[0] * 3] * 3
bad_matrix[0][0] = 99
print("Bad:", bad_matrix) 
# Output: [[99, 0, 0], [99, 0, 0], [99, 0, 0]]

# GOOD Initialization
good_matrix = [[0] * 3 for _ in range(3)]
good_matrix[0][0] = 99
print("Good:", good_matrix)
# Output: [[99, 0, 0], [0, 0, 0], [0, 0, 0]]

```

**Line-by-line Explanation:**

1. `bad_matrix` repeats the reference to `[0, 0, 0]` three times.
2. Changing index `[0][0]` affects the single underlying list object, visible in all rows.
3. `good_matrix` runs the loop 3 times, creating fresh lists.

---

### 3. How do you efficiently iterate over a matrix: Row-major or Column-major?

**Expected Answer:**
In Python, Row-major iteration is slightly more natural and performant because `matrix[i]` retrieves a reference to a row, and we then iterate that row. Column-major requires accessing `matrix[0][j]`, `matrix[1][j]` jumping between different inner list objects.

**Explanation:**
Row-major matches the structure: "List of Rows".
Code: `for row in matrix: for item in row: ...`

**Reasoning:**
While Python doesn't have strict CPU cache locality benefits like C due to pointer indirection, accessing the *same* inner list object repeatedly (Row-major) is cheaper than resolving a different list object for every single element (Column-major).

**Common Mistakes:**
Iterating column-wise `for c in range(cols): for r in range(rows):` when order doesn't matter.

**Why interviewers ask this:**
Checks code style and understanding of the underlying data structure.

**Sample Code:**

```python
matrix = [[1, 2], [3, 4]]

# Row-major (Preferred)
for r in range(len(matrix)):
    for c in range(len(matrix[0])):
        val = matrix[r][c]

# Column-major (Less efficient logic)
for c in range(len(matrix[0])):
    for r in range(len(matrix)):
        val = matrix[r][c]

```

---

### 4. How do you find the dimensions of a potentially jagged matrix?

**Expected Answer:**
You can find the number of rows using `len(matrix)`. For columns, if the matrix is uniform, `len(matrix[0])`. If jagged, `len(matrix[i])` varies per row.

**Explanation:**
A jagged matrix is a list of lists where inner lists have different lengths. Python supports this natively.

**Reasoning:**
You must validate input before assuming a rectangular shape (MxN).

**Common Mistakes:**
Assuming `len(matrix[0])` applies to all rows without checking, leading to `IndexError`.

**Why interviewers ask this:**
To see if you consider edge cases (empty matrix, jagged rows).

**Sample Code:**

```python
jagged = [
    [1, 2, 3],
    [4, 5]
]

rows = len(jagged)
# Get cols for specific row
cols_row_1 = len(jagged[1]) # Returns 2

```

---

### 5. What is the time complexity of accessing an element `matrix[r][c]`?

**Expected Answer:**
.

**Explanation:**
It consists of two indexing operations. `matrix[r]` is  (finding the list reference at index `r` in the outer list). The result is a list, and accessing `[c]` on that list is also .

**Reasoning:**
Lists are implemented as dynamic arrays. Index lookup is arithmetic pointer addition.

**Common Mistakes:**
Thinking it is  or  because it's a list. It is not a Linked List.

**Why interviewers ask this:**
Basic complexity check.

---

### 6. How do you perform a "Deep Copy" of a matrix?

**Expected Answer:**
Using the `copy` module's `deepcopy`, or a nested list comprehension. `list.copy()` or slicing `[:]` only creates a shallow copy (copies the outer list but inner lists are still shared references).

**Explanation:**
Shallow copy: `new = old[:]`. `new[0]` points to the same object as `old[0]`.
Deep copy: Recursively copies all objects.

**Reasoning:**
Essential for algorithms (like backtracking) where you need to modify a state without affecting the previous state.

**Common Mistakes:**
Using `new = old[:]` and thinking modifying `new[0][0]` won't hurt `old`.

**Sample Code:**

```python
import copy

original = [[1, 2], [3, 4]]

# Method 1: Library
deep_1 = copy.deepcopy(original)

# Method 2: Comprehension (Faster for simple lists)
deep_2 = [row[:] for row in original]

```

---

## Part B: Medium Level (14 Questions)

### 7. How does Python's `zip` function assist in Matrix Transposition?

**Question:** Explain how `zip(*matrix)` works to transpose a matrix and its return type.

**Expected Answer:**
`zip(*matrix)` unpacks the rows and zips the -th elements of each row together, effectively creating columns. It returns an iterator of tuples.

**Explanation:**
If `matrix = [[1, 2], [3, 4]]`:
`*matrix` passes `[1, 2]` and `[3, 4]` as arguments to `zip`.
`zip` pairs `1` with `3` and `2` with `4`.

**Reasoning:**
This is the most "Pythonic" one-liner for transposition.

**Time Complexity:**
 to iterate and build new tuples.

**Common Mistakes:**
Forgetting that `zip` returns tuples, so you get `[(1, 3), (2, 4)]`. You usually need `map(list, ...)` or a comprehension to get lists back.

**Why interviewers ask this:**
Tests knowledge of unpacking operators and functional programming tools.

**Sample Code:**

```python
matrix = [[1, 2, 3], [4, 5, 6]]
transposed = list(map(list, zip(*matrix)))
# Result: [[1, 4], [2, 5], [3, 6]]

```

---

### 8. What is the complexity of inserting a column at index 0 vs appending a column at the end?

**Question:** Compare `for row in matrix: row.insert(0, val)` vs `row.append(val)`.

**Expected Answer:**
`insert(0, val)` is  per row, so  total (where M is rows, N is cols).
`append(val)` is amortized  per row, so  total.

**Reasoning:**
Python lists are dynamic arrays. Inserting at the front requires shifting all existing elements in memory for every single row.

**Common Mistakes:**
Using `insert(0)` inside loops for matrices, causing quadratic slowdowns.

**Why interviewers ask this:**
Performance awareness in data processing scenarios.

---

### 9. How do you "Flatten" a 2D matrix into a 1D list efficiently?

**Question:** What are the ways to flatten a matrix and which is preferred?

**Expected Answer:**

1. Nested List Comprehension: `[x for row in matrix for x in row]` (Preferred).
2. `itertools.chain`: `list(itertools.chain(*matrix))` (Memory efficient).
3. `sum(matrix, [])`: (Bad practice).

**Explanation:**
`sum(matrix, [])` is  because it creates a new list for every concatenation. Comprehension is .

**Reasoning:**
Readability vs Performance. Comprehension is standard.

**Sample Code:**

```python
import itertools
matrix = [[1, 2], [3, 4]]

# Best Pythonic Way
flat = [x for row in matrix for x in row]

# Itertools (Lazy evaluation)
flat_iter = list(itertools.chain.from_iterable(matrix))

```

---

### 10. Explain the "Sparse Matrix" problem and how to handle it in Python.

**Question:** If you have a  matrix with only 500 non-zero values, how should you store it?

**Expected Answer:**
Do not use a List of Lists (requires  integers). Use a Dictionary `{(row, col): value}` or a specialized class (like `scipy.sparse`, though in a pure Python interview, Dictionary is the answer).

**Reasoning:**
List of Lists allocates memory for all zeros. A hash map (dict) only stores meaningful data.

**Time Complexity:**
Access `dict[(r,c)]` is  average, same as matrix, but space is  where K is non-zero elements.

**Why interviewers ask this:**
System design scaling within a coding interview.

**Sample Code:**

```python
# Instead of matrix[100][500] = 5
sparse_matrix = {}
sparse_matrix[(100, 500)] = 5

# Accessing
val = sparse_matrix.get((100, 500), 0) # Returns 0 if not set

```

---

### 11. How does `enumerate` help in Matrix Grid problems?

**Question:** Why use `enumerate` in 2D traversal?

**Expected Answer:**
It provides both the index and value cleanly. `for r, row in enumerate(matrix): for c, val in enumerate(row):`.

**Explanation:**
Eliminates the need for `range(len(matrix))` and manual indexing `matrix[r][c]`.

**Comparisons:**
Cleaner than C-style loops.

**Sample Code:**

```python
grid = [["A", "B"], ["C", "D"]]
for r, row in enumerate(grid):
    for c, val in enumerate(row):
        print(f"At ({r}, {c}) is {val}")

```

---

### 12. Rotating a Matrix: Why is `matrix[:] = ...` used?

**Question:** In LeetCode "Rotate Image", you often see `matrix[:] = zip(*matrix[::-1])`. Why the `[:]`?

**Expected Answer:**
This is an **In-Place** assignment. `matrix = ...` creates a new local variable `matrix` and breaks the link to the caller's object. `matrix[:] = ...` replaces the contents of the *existing* list object in memory.

**Reasoning:**
Many interview questions require "O(1) Space / In-Place". Reassigning the variable name fails this constraint if the function expects the original object to be modified.

**Why interviewers ask this:**
Deep understanding of Python references and scope.

---

### 13. What happens if you sort a matrix directly? `matrix.sort()`

**Question:** How does Python compare two lists when sorting a list of lists?

**Expected Answer:**
Python compares lists lexicographically (element by element). It sorts the rows based on the first element. If first elements are equal, it checks the second, and so on.

**Reasoning:**
This is useful for interval merging problems (sorting by start time).

**Sample Code:**

```python
matrix = [[2, 5], [1, 9], [2, 1]]
matrix.sort()
# Result: [[1, 9], [2, 1], [2, 5]]
# [1,9] is first (1 < 2). 
# [2,1] vs [2,5]: 1 < 5, so [2,1] comes before [2,5].

```

---

### 14. Coordinate Deltas for Directional Traversal

**Question:** How do you standardly implement 4-way or 8-way movement in a matrix without 8 `if` statements?

**Expected Answer:**
Use a "Directions Array" (List of tuples).

**Explanation:**
Iterate over the deltas and add them to current coordinates.

**Reasoning:**
Reduces code duplication and cyclomatic complexity.

**Sample Code:**

```python
directions = [(-1, 0), (1, 0), (0, -1), (0, 1)] # Up, Down, Left, Right
for dr, dc in directions:
    nr, nc = r + dr, c + dc
    if 0 <= nr < rows and 0 <= nc < cols:
        # Process neighbor
        pass

```

---

### 15. How do you implement a 2D Prefix Sum (Immutable)?

**Question:** How do you preprocess a matrix to answer range sum queries in ?

**Expected Answer:**
Build a `prefix` matrix where `prefix[i][j]` contains the sum of the rectangle from `(0,0)` to `(i,j)`.
Formula: `dp[r][c] = matrix[r][c] + dp[r-1][c] + dp[r][c-1] - dp[r-1][c-1]`.

**Reasoning:**
Standard DP technique for image processing and grid queries.

**Why interviewers ask this:**
Tests dynamic programming on grids.

---

### 16. Swapping Rows vs Swapping Columns: Performance?

**Question:** Which is faster: `matrix[0], matrix[1] = matrix[1], matrix[0]` or swapping elements column-wise?

**Expected Answer:**
Swapping rows is . Swapping columns is .

**Explanation:**
Swapping rows just swaps two pointers in the outer list. Swapping columns requires iterating every row and swapping elements individually.

**Why interviewers ask this:**
Performance optimization. If you need to manipulate grids, row operations are always cheaper in Python's List of Lists structure.

---

### 17. The "Visited" Matrix: Boolean vs Set?

**Question:** In BFS/DFS, should you use a 2D boolean matrix `visited[r][c]` or a Set `visited_set.add((r,c))`?

**Expected Answer:**

* **Boolean Matrix:** Faster access , slightly higher memory upfront (allocates full grid), better if graph is dense.
* **Set:** Slower constant (hashing overhead), lower memory if visiting few nodes (sparse traversal).
* **Interview Preference:** Boolean matrix is usually preferred for fixed grids for clarity and consistent .

---

### 18. Boundary Traversal Logic

**Question:** What is the loop condition to traverse ONLY the border of a matrix?

**Expected Answer:**
You typically don't use a single loop. You break it into 4 loops (Top row, Right col, Bottom row, Left col) or iterate all and check `if r == 0 or r == rows-1 or c == 0 or c == cols-1`.

**Reasoning:**
Iterating all is  (bad if you only want border). 4 loops is  (optimal).

---

### 19. Search in a Row-wise and Column-wise Sorted Matrix

**Question:** If a matrix is sorted both row-wise and column-wise, what is the optimal search complexity?

**Expected Answer:**
.

**Explanation:**
Start at the Top-Right corner. If target < current, move Left. If target > current, move Down. This creates a staircase path.

**Why interviewers ask this:**
Classic algorithm question (Search a 2D Matrix II).

---

### 20. List Comprehension with Conditions

**Question:** How do you filter a matrix to keep only even numbers using a comprehension?

**Expected Answer:**
`[x for row in matrix for x in row if x % 2 == 0]` (Returns 1D list).
Or `[[x for x in row if x % 2 == 0] for row in matrix]` (Returns 2D jagged structure).

**Line-by-line Explanation:**
The structure depends on whether you want to flatten the result or preserve the row structure.

---

# SECTION 2 — CODING QUESTIONS (10 TOTAL)

## Part A: Easy Level (3 Questions)

### 21. Matrix Diagonal Sum

**Question:**
Given a square matrix, calculate the sum of the primary diagonal and the secondary diagonal. Do not double count the center element if the matrix size is odd.

**Solution:**

```python
def diagonalSum(mat: list[list[int]]) -> int:
    n = len(mat)
    total = 0
    
    for i in range(n):
        # Primary diagonal: row == col
        total += mat[i][i]
        
        # Secondary diagonal: row + col == n - 1
        # Avoid double counting the center (where i == n - 1 - i)
        if i != n - 1 - i:
            total += mat[i][n - 1 - i]
            
    return total

```

**Line-by-line Explanation:**

1. Loop `i` from 0 to `n-1`.
2. Add `mat[i][i]` (Primary).
3. Add `mat[i][n-1-i]` (Secondary) ONLY if it's not the same element as the primary (which happens at the exact center of an odd-sized matrix).

**Complexity:**
Time:  (One pass).
Space: .

**Real App Usage:**
Image processing filters, tic-tac-toe logic.

---

### 22. Transpose Matrix

**Question:**
Given a 2D integer array `matrix`, return the transpose of `matrix`. (Flip over main diagonal).

**Solution:**

```python
def transpose(matrix: list[list[int]]) -> list[list[int]]:
    rows = len(matrix)
    cols = len(matrix[0])
    
    # Create new matrix with swapped dimensions (cols x rows)
    res = [[0] * rows for _ in range(cols)]
    
    for r in range(rows):
        for c in range(cols):
            res[c][r] = matrix[r][c]
            
    return res

# Pythonic Alternate:
# return [list(row) for row in zip(*matrix)]

```

**Line-by-line Explanation:**

1. Identify new dimensions. If original is , new is .
2. Initialize `res` with zeroes.
3. Assign `res[c][r]` = `matrix[r][c]`.

**Complexity:**
Time: .
Space:  for output.

**Scenario:**
Data science (pandas DataFrame T), Linear Algebra computations.

---

### 23. Richest Customer Wealth (Row Sum)

**Question:**
You are given an  integer grid `accounts` where `accounts[i][j]` is the money the -th customer has in the -th bank. Return the wealth that the richest customer has.

**Solution:**

```python
def maximumWealth(accounts: list[list[int]]) -> int:
    max_wealth = 0
    
    for customer in accounts:
        current_wealth = sum(customer)
        if current_wealth > max_wealth:
            max_wealth = current_wealth
            
    return max_wealth

```

**Line-by-line Explanation:**

1. Iterate through each row (`customer`).
2. Calculate `sum(customer)` using Python's built-in.
3. Update global max.

**Complexity:**
Time: .
Space: .

---

## Part B: Medium Level (7 Questions)

### 24. Spiral Matrix

**Question:**
Given an  matrix, return all elements of the matrix in spiral order.

**Solution:**

```python
def spiralOrder(matrix: list[list[int]]) -> list[int]:
    result = []
    if not matrix: return result
    
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1
    
    while top <= bottom and left <= right:
        # 1. Traverse Right
        for i in range(left, right + 1):
            result.append(matrix[top][i])
        top += 1
        
        # 2. Traverse Down
        for i in range(top, bottom + 1):
            result.append(matrix[i][right])
        right -= 1
        
        # Check boundary conditions
        if top <= bottom:
            # 3. Traverse Left
            for i in range(right, left - 1, -1):
                result.append(matrix[bottom][i])
            bottom -= 1
        
        if left <= right:
            # 4. Traverse Up
            for i in range(bottom, top - 1, -1):
                result.append(matrix[i][left])
            left += 1
            
    return result

```

**Line-by-line Explanation:**

1. Define 4 boundaries: `top`, `bottom`, `left`, `right`.
2. Loop while boundaries don't cross.
3. Iterate top row, then shrink top (`top += 1`).
4. Iterate right col, shrink right.
5. Check `if top <= bottom` before going left (crucial for non-square matrices).
6. Iterate bottom row, shrink bottom.
7. Check `if left <= right` before going up.
8. Iterate left col, shrink left.

**Complexity:**
Time: .
Space:  (excluding output).

**Common Mistakes:**
Forgetting the checks before steps 3 and 4, resulting in duplicates.

**Real App Usage:**
Image processing (spiral pixel manipulation), Maps rendering.

---

### 25. Rotate Image (In-Place)

**Question:**
You are given an  2D matrix representing an image, rotate the image by 90 degrees (clockwise). You have to rotate the image **in-place**.

**Solution:**

```python
def rotate(matrix: list[list[int]]) -> None:
    n = len(matrix)
    
    # Step 1: Transpose (Swap matrix[i][j] with matrix[j][i])
    for i in range(n):
        for j in range(i + 1, n): # Start from i+1 to avoid re-swapping
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
            
    # Step 2: Reverse each row
    for i in range(n):
        matrix[i].reverse()

```

**Line-by-line Explanation:**

1. Transpose logic: The diagonal `(0,0), (1,1)...` stays fixed. We swap Upper Triangle elements with Lower Triangle.
2. `range(i + 1, n)` ensures we only visit each pair once.
3. After Transpose, the columns are correct but in reverse order for a 90-degree rotation.
4. Reversing each row completes the 90-degree clockwise turn.

**Complexity:**
Time: .
Space: .

**Why interviewers ask this:**
Tests ability to manipulate indices and perform in-place operations.

---

### 26. Set Matrix Zeroes

**Question:**
Given an  integer matrix, if an element is 0, set its entire row and column to 0's. You must do it in place.

**Solution:**

```python
def setZeroes(matrix: list[list[int]]) -> None:
    rows, cols = len(matrix), len(matrix[0])
    row_zero = False # Flag for the first row
    
    # 1. Mark zeros on the first row and first column
    for r in range(rows):
        for c in range(cols):
            if matrix[r][c] == 0:
                matrix[0][c] = 0          # Mark column header
                if r > 0:
                    matrix[r][0] = 0      # Mark row header
                else:
                    row_zero = True       # Handle first row separately
                    
    # 2. Use marks to set inner matrix elements
    for r in range(1, rows):
        for c in range(1, cols):
            if matrix[0][c] == 0 or matrix[r][0] == 0:
                matrix[r][c] = 0
                
    # 3. Handle first column (using matrix[0][0])
    if matrix[0][0] == 0:
        for r in range(rows):
            matrix[r][0] = 0
            
    # 4. Handle first row (using row_zero flag)
    if row_zero:
        for c in range(cols):
            matrix[0][c] = 0

```

**Explanation:**
We use the first row and first column as "storage" for our flags to avoid  space.
Because `matrix[0][0]` is the intersection of the first row and col, we need a separate flag (`row_zero`) to know if the first row itself needs to be zeroed.

**Complexity:**
Time: .
Space: .

**Scenario:**
Spreadsheet applications (dependency handling), Minesweeper logic.

---

### 27. Number of Islands (Flood Fill / DFS)

**Question:**
Given an  2D binary grid `grid` which represents a map of '1's (land) and '0's (water), return the number of islands (connected adjacent lands).

**Solution:**

```python
def numIslands(grid: list[list[str]]) -> int:
    if not grid: return 0
    
    rows, cols = len(grid), len(grid[0])
    count = 0
    
    def dfs(r, c):
        # Base case: Bounds check or water
        if r < 0 or c < 0 or r >= rows or c >= cols or grid[r][c] == '0':
            return
        
        # Mark as visited (sink the island)
        grid[r][c] = '0' 
        
        # Visit neighbors
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
        
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                dfs(r, c) # Explores entire island
                
    return count

```

**Line-by-line Explanation:**

1. Iterate every cell.
2. If we find a '1', it is a *new* island. Increment count.
3. Trigger DFS to visit all connected '1's.
4. Inside DFS, we set `grid[r][c] = '0'`. This acts as "visited" so we don't recount this land.

**Complexity:**
Time: .
Space:  (Recursion stack in worst case).

**Real App Usage:**
Clustering algorithms, Image segmentation, Game map logic.

---

### 28. Valid Sudoku

**Question:**
Determine if a  Sudoku board is valid. Only the filled cells need to be validated according to standard Sudoku rules.

**Solution:**

```python
import collections

def isValidSudoku(board: list[list[str]]) -> bool:
    rows = collections.defaultdict(set)
    cols = collections.defaultdict(set)
    squares = collections.defaultdict(set) # Key: (r//3, c//3)
    
    for r in range(9):
        for c in range(9):
            val = board[r][c]
            if val == ".":
                continue
                
            if (val in rows[r] or 
                val in cols[c] or 
                val in squares[(r // 3, c // 3)]):
                return False
                
            rows[r].add(val)
            cols[c].add(val)
            squares[(r // 3, c // 3)].add(val)
            
    return True

```

**Explanation:**
We use Hash Sets to track seen numbers.
The trick is calculating the  sub-box index: `(r // 3, c // 3)`.
0-2 maps to index 0. 3-5 maps to index 1. 6-8 maps to index 2.

**Complexity:**
Time:  (Fixed  board).
Space: .

---

### 29. Search a 2D Matrix (Fully Sorted)

**Question:**
Write an efficient algorithm that searches for a value in an  matrix. The matrix has the following properties:

1. Integers in each row are sorted from left to right.
2. The first integer of each row is greater than the last integer of the previous row.

**Solution:**

```python
def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    if not matrix: return False
    rows, cols = len(matrix), len(matrix[0])
    
    # Treat matrix as a single sorted array of length rows * cols
    left, right = 0, rows * cols - 1
    
    while left <= right:
        mid = (left + right) // 2
        
        # Map 1D index back to 2D coordinates
        mid_val = matrix[mid // cols][mid % cols]
        
        if mid_val == target:
            return True
        elif mid_val < target:
            left = mid + 1
        else:
            right = mid - 1
            
    return False

```

**Explanation:**
Since the matrix is strictly sorted row-over-row, it is virtually a flattened 1D sorted array.
We perform standard Binary Search.
Coordinate Mapping: `row = index // cols`, `col = index % cols`.

**Complexity:**
Time: .
Space: .

---

### 30. Game of Life

**Question:**
Implement the "Game of Life" next state in-place.
Rules:

1. Live(1) with <2 neighbors -> dies.
2. Live(1) with 2-3 neighbors -> lives.
3. Live(1) with >3 neighbors -> dies.
4. Dead(0) with 3 neighbors -> lives.

**Solution:**

```python
def gameOfLife(board: list[list[int]]) -> None:
    rows, cols = len(board), len(board[0])
    neighbors = [(-1,-1), (-1,0), (-1,1), (0,-1), 
                 (0,1), (1,-1), (1,0), (1,1)]
    
    for r in range(rows):
        for c in range(cols):
            # Count live neighbors
            live_neighbors = 0
            for dr, dc in neighbors:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols:
                    # Check abs() because we might have marked it as -1
                    if abs(board[nr][nc]) == 1:
                        live_neighbors += 1
            
            # Apply rules using temporary states
            # 1 -> 0 : Mark as -1
            # 0 -> 1 : Mark as 2
            if board[r][c] == 1 and (live_neighbors < 2 or live_neighbors > 3):
                board[r][c] = -1
            elif board[r][c] == 0 and live_neighbors == 3:
                board[r][c] = 2
                
    # Finalize state
    for r in range(rows):
        for c in range(cols):
            if board[r][c] > 0:
                board[r][c] = 1
            else:
                board[r][c] = 0

```

**Explanation:**
To solve In-Place, we need to store the *next* state without destroying the *current* state which is needed for neighbor calculations.
We use intermediate values:

* `-1`: Was Live, now Dead. (Current state is 1).
* `2`: Was Dead, now Live. (Current state is 0).
After computing all cells, we normalize `-1` to `0` and `2` to `1`.

**Complexity:**
Time: .
Space: .
