# Python Math Functions — Interview Guide (30 Questions)

This guide covers Python's mathematical capabilities, focusing on the built-in functions, the standard `math` library, floating-point arithmetic nuances, and algorithmic implementations. It distinguishes between standard Python math and scientific libraries like NumPy.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. How does Python's `round()` function handle ties (e.g., 0.5)?

* **Expected Answer**: Python 3 uses **"Banker's Rounding"** (Round Half to Even).
* **Clear Explanation**: If the value is exactly halfway between two integers, it rounds to the nearest **even** number. This minimizes cumulative error in statistical calculations compared to standard "Round Half Up".
* **Sample Code**:
```python
print(round(0.5)) # 0 (Nearest even)
print(round(1.5)) # 2 (Nearest even)
print(round(2.5)) # 2 (Nearest even)

```



#### 2. What is the difference between `int(x)`, `math.floor(x)`, and `math.trunc(x)`?

* **Expected Answer**: They differ significantly for negative numbers.
* `math.floor(x)`: Returns the largest integer \le x (moves left on number line).
* `math.trunc(x)`: Simply drops the decimal part (moves toward zero).
* `int(x)`: Acts like `trunc()` for floats.


* **Comparison**:
* Input `-3.5`: `floor` -> `-4`. `trunc` -> `-3`. `int` -> `-3`.


* **Why Interviewers Ask This**: Essential for financial calculations or grid mapping logic.

#### 3. What does `divmod(a, b)` return?

* **Expected Answer**: It returns a tuple containing `(quotient, remainder)`.
* **Efficiency**: It calculates both values in a single operation (mostly at the C level), which is faster and cleaner than writing `a // b` and `a % b` separately.
* **Sample Code**:
```python
q, r = divmod(10, 3)
print(q, r) # 3, 1

```



#### 4. `math.pow(x, y)` vs Built-in `pow(x, y)` vs Operator `**`.

* **Expected Answer**:
* `math.pow(x, y)`: Always converts arguments to **float** and returns **float**. It follows C-standard logic (IEEE 754).
* `pow(x, y)` and `**`: Maintain type. Integers return integers (arbitrary precision). `pow()` also supports a 3rd argument for modular exponentiation.


* **Critical Scenario**: `math.pow(2, 100)` might lose precision or overflow if the result exceeds float limits. `2 ** 100` is exact.
* **Sample Code**:
```python
import math
print(math.pow(2, 3)) # 8.0 (Float)
print(2 ** 3)         # 8   (Int)

```



#### 5. How do `min()` and `max()` work with Dictionaries?

* **Expected Answer**: By default, they iterate over the **keys**.
* **Refinement**: To find the key associated with the max value, use the `key` argument.
* **Sample Code**:
```python
d = {'a': 10, 'b': 20}
print(max(d))        # 'b' (compares strings 'a' vs 'b')
print(max(d, key=d.get)) # 'b' (compares values 10 vs 20)

```



#### 6. What constants are available in the `math` module?

* **Expected Answer**:
* `math.pi` (\pi)
* `math.e` (Euler's number)
* `math.tau` (2\pi)
* `math.inf` (Infinity)
* `math.nan` (Not a Number)


* **Why Interviewers Ask This**: To verify you don't manually type `3.14159` in production code.

---

### Medium Level (Questions 7–20)

#### 7. `math.fsum()` vs built-in `sum()`: The Precision Problem.

* **Question**: Why would you use `math.fsum([0.1, 0.1, ...])` instead of `sum()`?
* **Expected Answer**: `math.fsum` tracks lost precision using partial sums (Shewchuk's algorithm) to return a more accurate floating-point total. `sum()` simply adds step-by-step, accumulating standard IEEE 754 floating-point errors.
* **Sample Code**:
```python
import math
arr = [0.1] * 10
print(sum(arr))        # 0.9999999999999999 (Error)
print(math.fsum(arr))  # 1.0 (Exact)

```



#### 8. Calculating the Euclidean Norm (`math.hypot`).

* **Question**: Why use `math.hypot(x, y)` instead of `math.sqrt(x**2 + y**2)`?
* **Expected Answer**: `hypot` is designed to avoid **intermediate overflow/underflow**.
* **Reasoning**: If `x` is very large (10^{200}), `x**2` will overflow (return `inf` in float arithmetic) before `sqrt` can bring it back down. `hypot` handles the scaling internally.
* **New Feature**: In Python 3.8+, `hypot` accepts N-dimensions: `math.hypot(x, y, z)`.

#### 9. Integer Square Root (`math.isqrt`).

* **Question**: What is `math.isqrt(n)` and when should you use it?
* **Expected Answer**: It returns the integer part of the square root of a non-negative integer `n`.
* **Benefit**: It operates purely on integers. Standard `math.sqrt` converts to float, which loses precision for integers larger than 2^{53}. `isqrt` is exact for arbitrarily large integers.
* **Sample Code**:
```python
n = 10**100
# math.sqrt(n) would lose precision bits
print(math.isqrt(n)) # Returns exact integer

```



#### 10. `math.gcd` and `math.lcm` implementation.

* **Question**: What is the time complexity of `math.gcd(a, b)`?
* **Expected Answer**: O(\log(\min(a, b))).
* **Implementation**: It uses the Euclidean Algorithm.
* **Update**: `math.lcm` (Least Common Multiple) was added in Python 3.9. Before that, you had to calculate it as `abs(a*b) // gcd(a,b)`.

#### 11. Comparing Floating Point Numbers (`math.isclose`).

* **Question**: Why is `a == b` bad for floats, and how does `math.isclose` fix it?
* **Expected Answer**: Due to representation errors, `0.1 + 0.2` results in `0.30000000000000004`. `math.isclose(a, b)` checks if values are within a relative or absolute tolerance.
* **Parameters**: `rel_tol` (relative tolerance, default 1e-9) and `abs_tol` (absolute tolerance, useful near zero).

#### 12. Handling `NaN` (Not a Number) logic.

* **Question**: How does `nan` behave in comparisons?
* **Expected Answer**: `nan` is unordered.
* `math.nan == math.nan` is **False**.
* `math.nan != math.nan` is **True**.
* `<`, `>`, `>=` comparisons involving `nan` always return False.


* **Checking**: Must use `math.isnan(x)`.

#### 13. Complex Numbers in Python.

* **Question**: How do you calculate the square root of -1 in Python?
* **Expected Answer**:
* `math.sqrt(-1)` raises `ValueError`.
* You must use the `cmath` module (Complex Math) or complex literals.


* **Sample Code**:
```python
import cmath
print(cmath.sqrt(-1)) # 1j
print((-1) ** 0.5)    # (6.12e-17+1j) (Approximation issues)

```



#### 14. Combinatorics (`math.comb` vs `math.perm`).

* **Question**: What is the difference between `math.comb(n, k)` and `math.perm(n, k)`?
* **Expected Answer**:
* `comb(n, k)`: "n choose k". Order does not matter. Formula: \frac{n!}{k!(n-k)!}.
* `perm(n, k)`: "n arrange k". Order matters. Formula: \frac{n!}{(n-k)!}.


* **Why use them**: They are implemented in C and handle large integers much faster than writing factorial formulas manually.

#### 15. Logarithm Precision (`math.log`).

* **Question**: What base does `math.log(x)` use by default?
* **Expected Answer**: Base e (Natural Logarithm, \ln).
* **Variations**:
* `math.log(x, base)`: Arbitrary base.
* `math.log2(x)`: Base 2 (More accurate/optimized for bits).
* `math.log10(x)`: Base 10.
* `math.log1p(x)`: Calculates \ln(1+x). accurate for small x where 1+x would lose precision.



#### 16. `abs()` vs `math.fabs()`.

* **Question**: Is there a difference?
* **Expected Answer**:
* `abs(x)`: Generic built-in. Returns integer if input is integer, float if float, implements `__abs__`.
* `math.fabs(x)`: Always returns a **float**.


* **Usage**: `math.fabs(-5)` -> `5.0`. `abs(-5)` -> `5`.

#### 17. The `decimal` module vs `math` module.

* **Question**: When should you abandon `math` (float) for `decimal`?
* **Expected Answer**: When exact decimal representation is required (Finance/Accounting).
* **Problem**: `math` operations convert Decimals to floats, losing the precision you tried to gain.
* **Solution**: Use `decimal.Decimal` methods (e.g., `Decimal(2).sqrt()`) instead of `math.sqrt()`.

#### 18. Infinity in Python.

* **Question**: How do you represent infinity and what happens if you negate it?
* **Expected Answer**: `float('inf')` or `math.inf`.
* **Properties**:
* `inf + 1 = inf`
* `-inf` is valid.
* `1 / inf = 0.0`
* `inf / inf = nan`



#### 19. Python's Randomness vs Math.

* **Question**: Does `math` contain random functions?
* **Expected Answer**: No, they are in the `random` module.
* **Why**: Separation of concerns. `math` is for deterministic functions. `random` is for stochastic processes.

#### 20. `sys.float_info` and Machine Epsilon.

* **Question**: How do you find the smallest float x such that 1.0 + x \neq 1.0?
* **Expected Answer**: `sys.float_info.epsilon`.
* **Value**: Typically around 2.22 \times 10^{-16} on 64-bit machines.
* **Relevance**: Critical for writing custom convergence loops in numerical algorithms (Newton-Raphson).

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Custom Rounding to Nearest N

**Question**: Write a function `round_to_base(x, base)` that rounds a number `x` to the nearest multiple of `base`.
e.g., `round_to_base(17, 5) -> 15`, `round_to_base(18, 5) -> 20`.

* **Solution**:
```python
def round_to_base(x, base):
    if base == 0: return x
    return round(x / base) * base

# Usage
print(round_to_base(18, 5)) # 20
print(round_to_base(17, 5)) # 15

```


* **Explanation**: Divide by base to scale to "units of base", round to nearest integer, scale back up.

#### 2. Check for Prime (Using `math.isqrt`)

**Question**: Efficiently check if an integer `n` is prime using math libraries.

* **Solution**:
```python
import math

def is_prime(n: int) -> bool:
    if n <= 1: return False
    if n == 2: return True
    if n % 2 == 0: return False

    # Only check up to integer square root
    limit = math.isqrt(n)
    for i in range(3, limit + 1, 2):
        if n % i == 0:
            return False
    return True

# Usage
print(is_prime(29)) # True

```


* **Optimization**: Checking up to `isqrt(n)` reduces complexity from O(N) to O(\sqrt{N}).

#### 3. List GCD

**Question**: Given a list of integers `[12, 24, 6, 18]`, find the GCD of the entire list.

* **Solution**:
```python
import math
import functools

def list_gcd(nums: list[int]) -> int:
    if not nums: return 0
    # math.gcd only takes 2 arguments in Python < 3.9
    # In Python 3.9+, you can do: math.gcd(*nums)
    return math.gcd(*nums)

# For strict compatibility or custom logic:
# return functools.reduce(math.gcd, nums)

# Usage
print(list_gcd([12, 24, 6, 18])) # 6

```



---

### Medium Level (Questions 4–10)

#### 4. Euclidean Distance (N-Dimensions)

**Question**: Write a function `distance(p1, p2)` for two points of N-dimensions. Use `math.hypot` features.

* **Solution**:
```python
import math

def distance(p1: tuple, p2: tuple) -> float:
    # Calculate differences (deltas)
    diffs = [b - a for a, b in zip(p1, p2)]
    # Unpack diffs into hypot (Py 3.8+)
    return math.hypot(*diffs)

# Usage
p1 = (1, 2, 3)
p2 = (4, 6, 8)
print(distance(p1, p2)) # 7.07...

```


* **Manual**: Without `hypot`, \sqrt{\sum (p1_i - p2_i)^2}.

#### 5. Calculate Angle Between Vectors

**Question**: Given two vectors `v1` and `v2`, calculate the angle between them in degrees. Formula: \theta = \arccos(\frac{v1 \cdot v2}{|v1| |v2|}).

* **Solution**:
```python
import math

def vector_angle(v1: list, v2: list) -> float:
    dot_product = sum(a * b for a, b in zip(v1, v2))
    magnitude_v1 = math.hypot(*v1)
    magnitude_v2 = math.hypot(*v2)

    if magnitude_v1 == 0 or magnitude_v2 == 0:
        return 0.0

    # Clamp value to [-1, 1] to avoid domain error due to precision
    cos_theta = dot_product / (magnitude_v1 * magnitude_v2)
    cos_theta = max(min(cos_theta, 1.0), -1.0)

    rads = math.acos(cos_theta)
    return math.degrees(rads)

# Usage
v_a = [1, 0]
v_b = [0, 1]
print(vector_angle(v_a, v_b)) # 90.0

```


* **Pitfall**: Floating point errors can make `cos_theta` slightly > 1.0 (e.g., 1.0000000002), causing `math.acos` to crash. Clamping is required.

#### 6. Nth Fibonacci (Golden Ratio Formula)

**Question**: Calculate the Nth Fibonacci number in O(1) (conceptually) using Binet's Formula, and handle the precision requirement.

* **Solution**:
```python
import math

def fib_math(n: int) -> int:
    # Binet's Formula: (phi^n - psi^n) / sqrt(5)
    # phi = (1 + sqrt(5)) / 2
    phi = (1 + math.sqrt(5)) / 2
    return round(math.pow(phi, n) / math.sqrt(5))

# Usage
print(fib_math(10)) # 55

```


* **Warning**: This works for small N. For large N (>70), float precision fails. Iterative approach is better for Large Ints.

#### 7. Solve Quadratic Equation

**Question**: Solve ax^2 + bx + c = 0. Return tuple of solutions. Handle cases where roots are complex.

* **Solution**:
```python
import cmath # Use cmath to handle negative discriminants naturally

def solve_quadratic(a, b, c):
    if a == 0: return None # Not quadratic
    d = (b**2) - (4*a*c) # Discriminant

    sol1 = (-b - cmath.sqrt(d)) / (2*a)
    sol2 = (-b + cmath.sqrt(d)) / (2*a)

    return sol1, sol2

# Usage
print(solve_quadratic(1, 0, 1)) # x^2 + 1 = 0 -> (+1j, -1j)

```



#### 8. LCM of Range

**Question**: Find the smallest number divisible by all numbers from 1 to N.

* **Solution**:
```python
import math

def smallest_multiple(n: int) -> int:
    result = 1
    for i in range(1, n + 1):
        result = math.lcm(result, i) # Py 3.9+
        # Pre 3.9: result = (result * i) // math.gcd(result, i)
    return result

# Usage
print(smallest_multiple(10)) # 2520

```



#### 9. Monte Carlo Pi Estimation

**Question**: Estimate \pi using the `random` and `math` modules. (Simulate dropping points in a circle).

* **Solution**:
```python
import random
import math

def estimate_pi(num_samples: int) -> float:
    inside_circle = 0
    for _ in range(num_samples):
        x = random.random()
        y = random.random()
        # Check if point is inside unit circle: x^2 + y^2 <= 1
        if x*x + y*y <= 1.0:
            inside_circle += 1

    # Area of square = 4 (from -1 to 1, but we use 0 to 1 quadrant)
    # Ratio Area_Circle / Area_Square = (Pi*r^2) / (2r)^2 = Pi / 4
    # Here we simulated one quadrant: Area = Pi/4
    return (inside_circle / num_samples) * 4

# Usage
print(estimate_pi(100000)) # ~3.14...

```



#### 10. Sigmoid Function (Activation Function)

**Question**: Implement the Sigmoid function S(x) = \frac{1}{1 + e^{-x}}. Handle overflow for large negative X.

* **Solution**:
```python
import math

def sigmoid(x):
    # Optimization:
    # If x is very negative, exp(-x) becomes HUGE -> Overflow
    # If x is very positive, exp(-x) becomes 0 -> Result 1.0

    if x >= 0:
        z = math.exp(-x)
        return 1 / (1 + z)
    else:
        # Numerically stable alternative for negative x
        # 1 / (1 + exp(-x)) == exp(x) / (exp(x) + 1)
        z = math.exp(x)
        return z / (1 + z)

# Usage
print(sigmoid(0))   # 0.5
print(sigmoid(100)) # 1.0

```


* **Why Interviewers Ask This**: Shows knowledge of numerical stability in ML contexts using standard math libraries.
