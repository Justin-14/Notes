# Python Bitwise Operators — Interview Guide (30 Questions)

This guide covers Python Bitwise Operators for a Senior Technical Interview context. Unlike C or Java, Python’s handling of bitwise operations is unique due to its arbitrary-precision integers and memory model.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What are the 6 Bitwise Operators in Python?

* **Expected Answer**:
1. `&` (AND)
2. `|` (OR)
3. `^` (XOR)
4. `~` (NOT / Complement)
5. `<<` (Left Shift)
6. `>>` (Right Shift)


* **Clear Explanation**: These operators work on the binary representations of integers. They perform the operation bit-by-bit.
* **Common Mistakes**: Confusing logical operators (`and`, `or`, `not`) with bitwise operators (`&`, `|`, `~`). Logical operators evaluate truthiness (short-circuiting), bitwise operators manipulate bits.
* **Why Interviewers Ask This**: Basic syntax check.
* **Sample Code**:
```python
a, b = 5, 3  # 5=101, 3=011
print(a & b) # 1 (001)
print(a | b) # 7 (111)

```



#### 2. Explain the relationship between Left Shift (`<<`) and multiplication.

* **Question**: What is the mathematical equivalent of `x << n`?
* **Expected Answer**: `x << n` is equivalent to multiplying x by 2^n.
* **Reasoning**: Shifting binary digits to the left introduces zeros on the right, effectively doubling the number for every position shifted.
* **Complexity**: O(1) (effectively, though technically scales with number of bits).
* **ASCII Diagram**:
```text
Value: 3 (0000 0011)
<< 1
Result: 6 (0000 0110)

```


* **Sample Code**:
```python
print(3 << 2) # 3 * (2^2) = 12

```



#### 3. Explain the relationship between Right Shift (`>>`) and division.

* **Question**: What is the mathematical equivalent of `x >> n`?
* **Expected Answer**: `x >> n` is equivalent to **integer (floor) division** of x by 2^n (i.e., `x // 2**n`).
* **Reasoning**: It truncates the least significant bits.
* **Common Mistakes**: Thinking it rounds to nearest. It always floors (rounds down toward negative infinity for negative numbers).
* **Sample Code**:
```python
print(10 >> 1) # 10 // 2 = 5
print(15 >> 2) # 15 // 4 = 3

```



#### 4. How do you check if a number is Odd or Even using bitwise operators?

* **Expected Answer**: Check the Least Significant Bit (LSB) using `x & 1`.
* If `x & 1 == 0` → Even.
* If `x & 1 == 1` → Odd.


* **Clear Explanation**: Binary numbers are sums of powers of 2 (2^0, 2^1, 2^2...). Only 2^0 (1) is odd. All other powers (2, 4, 8...) are even. Therefore, the last bit determines oddness.
* **Why Interviewers Ask This**: It is slightly faster than `% 2` and demonstrates binary understanding.
* **Sample Code**:
```python
def is_odd(n):
    return (n & 1) == 1

```



#### 5. What is the unique property of XOR (`^`)?

* **Expected Answer**: XOR returns `1` only if the bits are different. Crucially, `x ^ x == 0` and `x ^ 0 == x`.
* **Reasoning**: XOR acts as a "toggle" or "difference" detector. It is reversible: if `a ^ b = c`, then `c ^ b = a`.
* **Real Usage**: Cryptography (One-time pads), RAID parity calculation, swapping values.
* **Sample Code**:
```python
# Self-inverse property
key = 123
data = 456
encrypted = data ^ key
decrypted = encrypted ^ key
print(decrypted == data) # True

```



#### 6. Why can't we Bitwise Shift a float?

* **Question**: What happens if you run `3.5 << 1`?
* **Expected Answer**: It raises a `TypeError`. Bitwise operators are only defined for integers in Python.
* **Why Interviewers Ask This**: To verify you understand that bitwise ops work on raw memory bits of *integers*, not the IEEE 754 floating-point representation.
* **Sample Code**:
```python
try:
    x = 3.5 << 1
except TypeError as e:
    print(e) # unsupported operand type(s) for <<: 'float' and 'int'

```



---

### Medium Level (Questions 7–20)

#### 7. How does Python handle Integer Overflow in Bitwise operations?

* **Question**: In C/Java, `1 << 31` might overflow signed 32-bit integers. What happens in Python?
* **Expected Answer**: Python integers have arbitrary precision. There is no overflow. `1 << 100` is simply a very large number.
* **Medium-Depth Reasoning**: Python switches underlying representation from standard machine integers to an array of digits (BigInt) automatically.
* **Implication**: You cannot rely on overflow to wrap around to 0 or negative numbers (simulating 32-bit limits requires manual masking, e.g., `& 0xFFFFFFFF`).
* **Sample Code**:
```python
x = 1 << 64
print(x) # 18446744073709551616 (No overflow)

```



#### 8. Explain the Bitwise NOT (`~`) operator result formula.

* **Question**: Why is `~5` equal to `-6`?
* **Expected Answer**: The formula is `~x = -x - 1`.
* **Clear Explanation**: Python uses an infinite stream of sign bits for negative numbers (conceptually).
* In binary, positive 5 is `...00000101`.
* Inverting bits yields `...11111010`.
* In Two's Complement arithmetic, `...11111010` represents `-6`.


* **Why Interviewers Ask This**: It confuses everyone. It proves you understand Two's Complement representation.
* **Sample Code**:
```python
print(~0)  # -1
print(~9)  # -10

```



#### 9. How do you simulate a 32-bit Unsigned Integer in Python?

* **Question**: Python ints are infinite. How do I get standard C-like 32-bit wrap-around behavior?
* **Expected Answer**: Apply a mask of 32 ones: `& 0xFFFFFFFF`.
* **Reasoning**: The AND operation zeroes out all bits higher than the 32nd position, mimicking a 32-bit register limit.
* **Real Usage**: Hashing algorithms, crypto implementations porting from C.
* **Sample Code**:
```python
val = (1 << 32) + 5 # 33rd bit is set
simulated_32 = val & 0xFFFFFFFF
print(simulated_32) # 5 (The high bit was discarded)

```



#### 10. The `x & (x - 1)` Trick.

* **Question**: What does the operation `x & (x - 1)` do?
* **Expected Answer**: It turns off (clears) the **rightmost set bit** (the lowest `1` bit).
* **ASCII Diagram**:
```text
x       = 1 1 0 0  (12)
x - 1   = 1 0 1 1  (11)
x & x-1 = 1 0 0 0  (8) -> Lowest '1' removed

```


* **Why Interviewers Ask This**: It is the standard algorithmic trick for counting set bits (Brian Kernighan’s algorithm) or checking for powers of 2.
* **Time Complexity**: O(1) operation.

#### 11. Checking for Power of 2.

* **Question**: Write a one-liner using bitwise operators to check if `n` is a power of 2.
* **Expected Answer**: `n > 0 and (n & (n - 1) == 0)`.
* **Reasoning**: Powers of 2 have exactly one bit set (`1`, `10`, `100`). Removing that single bit using `n & (n-1)` should result in `0`. We check `n > 0` because `0` is not a power of 2.
* **Sample Code**:
```python
is_power_of_two = lambda n: n > 0 and (n & (n - 1) == 0)
print(is_power_of_two(16)) # True
print(is_power_of_two(18)) # False

```



#### 12. Swapping numbers without a temporary variable.

* **Question**: Swap `a` and `b` using XOR.
* **Expected Answer**:
```python
a = a ^ b
b = a ^ b
a = a ^ b

```


* **Reasoning**:
1. `a` becomes `a^b`.
2. `b` becomes `(a^b)^b` -> `a`.
3. `a` becomes `(a^b)^a` -> `b`.


* **Practical Note**: In Python, just use `a, b = b, a`. This interview question is purely to test XOR logic properties.

#### 13. How to Set, Clear, and Toggle a specific bit?

* **Question**: Given integer `x` and position `k`:
1. Set the k-th bit.
2. Clear the k-th bit.
3. Toggle the k-th bit.


* **Expected Answer**:
1. **Set**: `x | (1 << k)` (OR with 1 at pos k)
2. **Clear**: `x & ~(1 << k)` (AND with 0 at pos k, 1 elsewhere)
3. **Toggle**: `x ^ (1 << k)` (XOR flips the bit)


* **Sample Code**:
```python
x = 10 # 1010
k = 0
print(x | (1 << k)) # 1011 (11) -> Set bit 0

```



#### 14. Bitwise Operator Precedence vs Comparison Operators.

* **Question**: How does Python evaluate `x & 1 == 0`?
* **Expected Answer**: It evaluates as `(x & 1) == 0`.
* **Reasoning**: In Python, bitwise operators (`&`, `|`, `^`) have **higher** precedence than comparison operators (`==`, `!=`).
* *Critical Note*: This is different from C/C++, where `==` is higher than `&`.


* **Common Mistake**: Assuming C-like precedence. However, explicit parentheses `(x & 1) == 0` are always recommended for readability.
* **Sample Code**:
```python
# Python Precedence
# << >> > & > ^ > | > ==
print(5 & 3 == 1) # True. (5&3) is 1. 1 == 1 is True.

```



#### 15. Negative Shifts.

* **Question**: What happens if you do `x << -1` or `x >> -1`?
* **Expected Answer**: Python raises a `ValueError: negative shift count`.
* **Reasoning**: Shifting by a negative amount is undefined behavior logically, so Python explicitly forbids it.
* **Sample Code**:
```python
# 1 << -2 # Raises ValueError

```



#### 16. Bitmasks vs. Sets for Permissions.

* **Question**: Why would you use a Bitmask instead of a Python `set` for user permissions?
* **Expected Answer**:
* **Space**: An integer (bitmask) takes ~28 bytes. A set takes ~200+ bytes.
* **Speed**: Checking `mask & PERM` is a single CPU instruction (O(1)). Set lookup is hash-based (slower overhead).
* **Storage**: Bitmasks can be stored in a single database integer column. Sets require join tables or JSON fields.


* **Real Usage**: Django's `Permission` system (historically), Linux file permissions (`755` -> `111 101 101`).

#### 17. Finding the most significant bit (MSB).

* **Question**: How do you find the bit length of an integer?
* **Expected Answer**: Use `int.bit_length()`.
* **Reasoning**: While you can loop with `>> 1`, Python integers are objects with a built-in optimized method to report the number of bits required to represent the number (excluding sign and leading zeros).
* **Sample Code**:
```python
n = 10 # 1010
print(n.bit_length()) # 4

```



#### 18. Little Endian vs Big Endian (Bytes to Int).

* **Question**: You receive raw bytes `b'\x01\x02'` from network. How do you convert this to an int using Big Endian?
* **Expected Answer**: `int.from_bytes(data, byteorder='big')`.
* **Reasoning**:
* Big Endian: `0x01` is MSB. Result: `0x0102` (258).
* Little Endian: `0x02` is MSB. Result: `0x0201` (513).


* **Why Interviewers Ask This**: Fundamental for networking/binary file parsing (images, protocol headers).
* **Sample Code**:
```python
b = b'\x01\x02'
print(int.from_bytes(b, 'big'))   # 258
print(int.from_bytes(b, 'little')) # 513

```



#### 19. Truth Table: Can you draw the XOR truth table?

* **Question**: Quick, input `A`, `B`. Output `A ^ B`.
* **Expected Answer**:
* 0, 0 -> 0
* 0, 1 -> 1
* 1, 0 -> 1
* 1, 1 -> 0


* **Mnemonic**: "One or the other, but not both." Or "Result is 1 if inputs are different."

#### 20. Efficiently Multiplying by 7.

* **Question**: How do you multiply `x` by 7 using only bitwise shifts and subtraction?
* **Expected Answer**: `(x << 3) - x`.
* **Reasoning**:
* x << 3 is 8x.
* 8x - x = 7x.


* **Why Interviewers Ask This**: Tests ability to decompose arithmetic into binary logic.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Hamming Weight (Count Set Bits)

**Question**: Write a function that takes an integer `n` and returns the number of '1' bits (set bits) it has.

* **Solution**:
```python
def count_bits(n: int) -> int:
    # Python 3.10+
    return n.bit_count()

def count_bits_manual(n: int) -> int:
    # For older python or interview constraint
    count = 0
    while n > 0:
        n &= (n - 1) # Kernighan's algorithm (skips zeros)
        count += 1
    return count

# Usage
print(count_bits(11)) # 1011 -> 3

```


* **Explanation**: `n.bit_count()` is the modern, optimized C-level implementation. Kernighan's algorithm `n &= (n-1)` is faster than standard looping because it iterates only as many times as there are set bits.
* **Complexity**: Time O(k) where k is number of set bits.

#### 2. Single Number (Find Unique)

**Question**: Given a non-empty list of integers where every element appears twice except for one, find that single one. Space O(1).

* **Solution**:
```python
def find_single_number(nums: list[int]) -> int:
    result = 0
    for num in nums:
        result ^= num
    return result

# Usage
print(find_single_number([4, 1, 2, 1, 2])) # 4

```


* **Explanation**:
* `a ^ a = 0` (Duplicates cancel out).
* `0 ^ b = b` (Identity).
* Order doesn't matter. `4 ^ 1 ^ 2 ^ 1 ^ 2` -> `4 ^ (1^1) ^ (2^2)` -> `4 ^ 0 ^ 0` -> `4`.


* **Complexity**: Time O(N), Space O(1).
* **Real App Usage**: Checksums, finding data discrepancies.

#### 3. Binary Gap

**Question**: Find the longest sequence of zeros between two ones in the binary representation of an integer.

* **Solution**:
```python
def binary_gap(n: int) -> int:
    binary_string = bin(n)[2:] # Convert to '1001' string, strip '0b'
    # Trim zeros from right because gap must be enclosed by 1s?
    # Standard definition: enclosed by 1s.
    # 9 (1001) -> gap 2. 20 (10100) -> gap 1.

    max_gap = 0
    current_gap = 0
    started = False

    # Bitwise iteration (Right to Left)
    while n > 0:
        bit = n & 1
        n >>= 1

        if bit == 1:
            if started:
                # Closing a gap
                max_gap = max(max_gap, current_gap)
            current_gap = 0
            started = True
        elif started:
            current_gap += 1

    return max_gap

# Usage
print(binary_gap(9))  # 1001 -> 2
print(binary_gap(20)) # 10100 -> 1 (Between the 1s)

```


* **Line-by-Line**: Iterates bits from LSB. Maintains a counter. Resets counter when `1` is hit. Updates max only if `started` flag is true (meaning we saw a previous `1`).
* **Complexity**: Time O(\log N) (number of bits).

---

### Medium Level (Questions 4–10)

#### 4. Reverse Bits

**Question**: Reverse the bits of a given 32-bit unsigned integer. (e.g., `00...011` -> `110...00`).

* **Solution**:
```python
def reverse_bits(n: int) -> int:
    result = 0
    for _ in range(32):
        result = (result << 1) | (n & 1)
        n >>= 1
    return result

# Usage
n = 43261596
print(f"{n:032b}")
rev = reverse_bits(n)
print(f"{rev:032b}")

```


* **Explanation**:
1. `n & 1`: Get the last bit of input.
2. `result << 1`: Shift result to make room.
3. `|`: Append the bit to result.
4. `n >> 1`: Process next bit of input.


* **Common Mistake**: Doing this in Python without a range loop will just result in the same number (since leading zeros are ignored in infinite precision integers). You must define a fixed width (32) for "reversing" to make sense.

#### 5. Missing Number

**Question**: Given an array containing `n` distinct numbers taken from `0, 1, 2, ..., n`, find the one that is missing. Usage of XOR is required for O(1) space.

* **Solution**:
```python
def missing_number(nums: list[int]) -> int:
    n = len(nums)
    xor_sum = 0

    # XOR all indices [0..n]
    for i in range(n + 1):
        xor_sum ^= i

    # XOR all values in array
    for num in nums:
        xor_sum ^= num

    return xor_sum

# Usage
print(missing_number([3, 0, 1])) # 2 (Range is 0..3)

```


* **Explanation**:
* If no number was missing: `XOR(indices) ^ XOR(values)` would be 0.
* With one missing: The result is the missing number (everything else cancels out).


* **Complexity**: Time O(N), Space O(1).

#### 6. Sum of Two Integers (No `+` or `-`)

**Question**: Calculate the sum of two integers `a` and `b` without using `+` or `-` operators.

* **Solution**:
```python
def get_sum(a: int, b: int) -> int:
    # Mask to 32-bits to handle Python's infinite recursion on negatives
    MASK = 0xFFFFFFFF
    MAX_INT = 0x7FFFFFFF

    while b != 0:
        # Carry is AND shifted left
        carry = (a & b) << 1
        # Sum is XOR (without carry)
        a = (a ^ b) & MASK
        b = carry & MASK

    # If result is negative in 32-bit sense, convert to Python negative
    return a if a <= MAX_INT else ~(a ^ MASK)

# Usage
print(get_sum(5, 7)) # 12
print(get_sum(-1, 1)) # 0

```


* **Explanation**:
* XOR (`^`) simulates addition without carry.
* AND (`&`) followed by Shift (`<<`) calculates the carry.
* The `MASK` is crucial in Python because without it, `b` (carry) would never become 0 for negative numbers (infinite 1s).


* **Why Interviewers Ask This**: To test deep understanding of how ALUs (Arithmetic Logic Units) work.

#### 7. Power Set (Subsets) via Bit Manipulation

**Question**: Given a set of distinct integers `[1, 2, 3]`, return all possible subsets.

* **Solution**:
```python
def subsets(nums: list[int]) -> list[list[int]]:
    n = len(nums)
    output = []

    # Iterate from 0 to 2^n - 1
    # Example n=3, iterate 0..7 (000 to 111)
    for i in range(1 << n):
        current_subset = []
        for j in range(n):
            # Check if j-th bit is set in i
            if (i >> j) & 1:
                current_subset.append(nums[j])
        output.append(current_subset)

    return output

# Usage
print(subsets([1, 2, 3]))

```


* **Explanation**: The binary representation of `i` acts as a mask. If the j-th bit is 1, include the j-th element.
* `000` -> `[]`
* `101` -> `[nums[0], nums[2]]`


* **Complexity**: Time O(N \cdot 2^N).

#### 8. Single Number II (Thrice Duplicates)

**Question**: Every element appears **three** times except for one, which appears exactly once. Find it. Space O(1).

* **Solution**:
```python
def single_number_three(nums: list[int]) -> int:
    ones = 0
    twos = 0

    for num in nums:
        # 'ones' tracks bits appearing once
        # 'twos' tracks bits appearing twice
        ones = (ones ^ num) & ~twos
        twos = (twos ^ num) & ~ones

    return ones

# Usage
print(single_number_three([2, 2, 3, 2])) # 3

```


* **Explanation**: This is a digital logic circuit design problem.
* We need a counter that goes 00 -> 01 -> 10 -> 00 (reset at 3).
* `ones` stores the first bit of the counter, `twos` stores the second.
* When a bit appears for the 3rd time, it clears both.


* **Why Interviewers Ask This**: Very advanced bit manipulation filtering.

#### 9. IP Address to Integer and Back

**Question**: Convert an IPv4 string "192.168.1.1" to a 32-bit integer, and back.

* **Solution**:
```python
def ip_to_int(ip: str) -> int:
    parts = [int(p) for p in ip.split('.')]
    return (parts[0] << 24) | (parts[1] << 16) | (parts[2] << 8) | parts[3]

def int_to_ip(num: int) -> str:
    return ".".join([
        str((num >> 24) & 0xFF),
        str((num >> 16) & 0xFF),
        str((num >> 8) & 0xFF),
        str(num & 0xFF)
    ])

# Usage
i = ip_to_int("192.168.1.1")
print(i) # 3232235777
print(int_to_ip(i)) # "192.168.1.1"

```


* **Real App Usage**: Databases store IPs as Integers for faster indexing and range queries (e.g., GeoIP lookups).
* **Explanation**: Each octet is 8 bits. We shift them into position.

#### 10. Compute Absolute Value (Branchless)

**Question**: Compute `abs(n)` for a 32-bit integer `n` without using `abs()` or `if/else` (branchless).

* **Solution**:
```python
def fast_abs(n: int) -> int:
    # Assuming 32-bit logic simulation
    # Mask pushes the sign bit to the far right
    # 0 for positive, -1 (all 1s) for negative
    mask = n >> 31

    # XOR handles the negation logic, subtraction handles the +1
    # If n is pos: mask=0. (n^0) - 0 = n
    # If n is neg: mask=-1. (n^-1) - (-1) -> (~n) + 1 -> -n
    return (n ^ mask) - mask

# Usage
print(fast_abs(-50)) # 50

```


* **Explanation**:
1. Get sign mask: `0` if positive, `-1` (all 1s binary) if negative.
2. `n ^ -1` is `~n` (bitwise NOT).
3. `~n - (-1)` is `~n + 1`, which is the Two's Complement negation (Absolute value of negative).


* **Why Interviewers Ask This**: Low-level optimization techniques used in graphics and physics engines.
