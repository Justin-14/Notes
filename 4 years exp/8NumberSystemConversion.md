# Python Number System Conversion — Interview Guide (30 Questions)

This guide covers conversion between different number systems (Binary, Octal, Decimal, Hexadecimal) and custom bases using Python. It explores built-in functions, manual algorithmic implementations, and the nuances of Python's memory model regarding integer representation.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What are the four standard Python Integer Literals?

* **Expected Answer**:
1. **Decimal** (Base 10): `10`, `255`
2. **Binary** (Base 2): Starts with `0b`. e.g., `0b1010`
3. **Hexadecimal** (Base 16): Starts with `0x`. e.g., `0xFF`
4. **Octal** (Base 8): Starts with `0o`. e.g., `0o755`


* **Clear Explanation**: These prefixes tell the Python parser how to interpret the digits that follow. They all result in a standard `int` object.
* **Sample Code**:
```python
a = 0b101
b = 5
print(a == b) # True

```



#### 2. How do you convert a String to an Integer with a specific base?

* **Expected Answer**: Use `int(string, base)`.
* **Clear Explanation**: The `int()` constructor takes a second argument `base`.
* `int("101", 2)` -> `5`
* `int("FF", 16)` -> `255`


* **Edge Case**: If the base is `0`, Python attempts to interpret the base from the string's prefix (like `0x` or `0b`).
* **Sample Code**:
```python
print(int("10", 2))  # 2
print(int("10", 8))  # 8
print(int("10", 16)) # 16

```



#### 3. What do `bin()`, `oct()`, and `hex()` return?

* **Expected Answer**: They return a **string** representation prefixed with `0b`, `0o`, or `0x` respectively.
* **Common Mistake**: Thinking they return a number. They return `str`.
* **Why Interviewers Ask This**: To verify you know how to format output for logging or debugging.
* **Sample Code**:
```python
x = 10
print(type(hex(x))) # <class 'str'>
print(hex(x))       # "0xa"

```



#### 4. The "Leading Zero" Trap in Decimal Literals.

* **Question**: What happens if you type `x = 010` in Python 3?
* **Expected Answer**: `SyntaxError`.
* **Reasoning**: In Python 2, `010` was Octal (8). In Python 3, to avoid confusion between "Zero-padded decimal" and "Octal", explicit prefixes (`0o`) are required. Leading zeros on non-zero decimal numbers are forbidden.
* **Sample Code**:
```python
# x = 05 # SyntaxError: leading zeros in decimal integer...
x = 0o5 # Correct Octal

```



#### 5. Case Sensitivity in Hexadecimal.

* **Question**: Is `0xFF` the same as `0xff`? What about `int('A', 16)` vs `int('a', 16)`?
* **Expected Answer**: Yes, they are identical.
* **Explanation**: Hexadecimal digits `A-F` are case-insensitive in Python, both in literals and in string conversion functions.
* **Sample Code**:
```python
print(0xA == 0xa) # True

```



#### 6. Converting integers back to characters (ASCII/Unicode).

* **Question**: How do you convert an integer `65` to `'A'` and back?
* **Expected Answer**:
* `chr(65)` -> `'A'`
* `ord('A')` -> `65`


* **Context**: This is essentially Base 256 (or Base 1,114,112 for Unicode) conversion, fundamental for cryptography and encoding.

---

### Medium Level (Questions 7–20)

#### 7. Algorithm: Converting Decimal to Any Base.

* **Question**: Mathematically, how do you convert a decimal number N to Base B?
* **Expected Answer**: Repeated Division-Remainder method.
* **Steps**:
1. N \% B gives the least significant digit (LSD).
2. N // B gives the new quotient.
3. Repeat until quotient is 0.
4. The result is the remainders collected in reverse order (LIFO).


* **ASCII Diagram**:
```text
Convert 6 to Binary:
6 / 2 = 3 remainder 0  (LSD)
3 / 2 = 1 remainder 1
1 / 2 = 0 remainder 1  (MSB)
Result: 110

```



#### 8. Algorithm: Converting Any Base to Decimal.

* **Question**: How do you convert string "110" (Base 2) to Decimal manually?
* **Expected Answer**: Polynomial Expansion (Positional Notation).
* **Formula**: \sum (digit \times base^{position}).
* **Example**: `1*2^2 + 1*2^1 + 0*2^0` = `4 + 2 + 0` = `6`.
* **Time Complexity**: O(N) where N is the number of digits.

#### 9. Format Strings vs Built-in Functions.

* **Question**: How do you get a binary string *without* the `0b` prefix using f-strings?
* **Expected Answer**: `f"{value:b}"`.
* **Comparison**:
* `bin(5)` -> `"0b101"`
* `format(5, 'b')` -> `"101"`
* `f"{5:08b}"` -> `"00000101"` (Padded).


* **Why Interviewers Ask This**: String formatting mastery is essential for generating clean reports or network packets.

#### 10. Negative Numbers in Binary (Python's Two's Complement).

* **Question**: `bin(-5)` returns `'-0b101'`. Why doesn't it return the Two's Complement bit sequence?
* **Expected Answer**: Python integers are arbitrary-precision. There is no fixed "32-bit" width, so there is no "sign bit" at a fixed position to flip. Conceptually, negative numbers have an infinite string of leading 1s.
* **Workaround**: To get a 32-bit Two's Complement representation, apply a mask: `bin(-5 & 0xFFFFFFFF)`.

#### 11. `int.to_bytes` and `int.from_bytes`.

* **Question**: How do you convert a large integer into a raw byte array for network transmission?
* **Expected Answer**: Use `val.to_bytes(length, byteorder)`.
* **Parameters**:
* `length`: Number of bytes (e.g., 4 for 32-bit).
* `byteorder`: `'big'` (Network standard) or `'little'`.


* **Sample Code**:
```python
x = 1024
b = x.to_bytes(2, 'big') # b'\x04\x00'

```



#### 12. Floating Point to Hexadecimal.

* **Question**: Can you convert a float to hex? What is `float.hex()`?
* **Expected Answer**: Yes. `float.hex()` returns the exact hexadecimal representation of the floating-point number.
* **Format**: `0x1.m...p+n` (Hex digit + Mantissa + Power of 2 exponent).
* **Why Interviewers Ask This**: Debugging precision issues. `0.1` in decimal is distinct in binary.
* **Sample Code**:
```python
print((0.1).hex()) # 0x1.999999999999ap-4

```



#### 13. Why use Octal (`0o755`) in Linux Permissions?

* **Question**: Why is Base 8 used for file permissions?
* **Expected Answer**: Each Octal digit represents exactly 3 bits.
* Permissions group: Read (4), Write (2), Execute (1).
* Max sum is 7 (111).
* `755` aligns perfectly with `rwx r-x r-x` (User, Group, Others).


* **Relevance**: Understanding how to manipulate mode bits in Python (`os.chmod`).

#### 14. Efficiently stripping prefixes.

* **Question**: You have a list of strings `['0b10', '0xAF']`. How do you reliably strip prefixes to get raw digits?
* **Expected Answer**: `s.removeprefix('0b')` (Python 3.9+).
* **Legacy**: `s[2:]` (Risky if the string doesn't actually have the prefix).

#### 15. Base64 vs Base16 (Hex).

* **Question**: Why do we use Base64 for email attachments instead of Base16?
* **Expected Answer**: **Efficiency**.
* Base16 (Hex): 2 characters per byte (100% overhead).
* Base64: 4 characters per 3 bytes (~33% overhead).


* **Python Lib**: `base64` module vs `binascii.hexlify`.

#### 16. What is the max base for `int(s, base)`?

* **Question**: Can I do `int("Z", 36)`? Can I do Base 62?
* **Expected Answer**: Max base is **36** (Digits 0-9 + Letters A-Z).
* **Reasoning**: English alphabet has 26 letters + 10 digits = 36 symbols.
* **Base 62**: Standard `int()` does not support Base 62 (case-sensitive A-Z, a-z). You must write a custom function.

#### 17. Time Complexity of `int(str, base)`.

* **Question**: If the string has N digits, what is the complexity?
* **Expected Answer**: O(N^2) generally, or O(N^{1.585}) with Karatsuba multiplication for very large numbers.
* **Reasoning**: Converting string to integer involves large number multiplication (Accumulator \times Base + Digit).

#### 18. Bit Length calculation.

* **Question**: How many bits are required to store the number 16?
* **Expected Answer**: 5 bits (`10000`).
* **Method**: `(16).bit_length()` returns 5.
* **Edge Case**: `(0).bit_length()` is 0.

#### 19. Using `eval()` for conversions.

* **Question**: Can `eval("0xFF")` convert hex? Is it safe?
* **Expected Answer**: Yes, it works, but **never use it** on user input. It poses a massive security risk (Code Injection). Always use `int("0xFF", 16)`.

#### 20. Does Endianness affect bitwise operations?

* **Question**: If I shift `x << 1`, does it matter if my CPU is Little Endian or Big Endian?
* **Expected Answer**: No. Bitwise operations are an abstraction over the *value*, not the *storage layout*. Python handles the abstraction. Endianness only matters when converting to/from raw bytes (`to_bytes`).

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Decimal to Binary (Manual String Construction)

**Question**: Write a function `to_binary(n)` that converts a positive integer to a binary string without using `bin()`.

* **Solution**:
```python
def to_binary(n: int) -> str:
    if n == 0: return "0"
    bits = []
    while n > 0:
        # Append remainder (0 or 1)
        bits.append(str(n % 2))
        # Floor divide
        n //= 2
    # Reverse to get MSB first
    return "".join(reversed(bits))

# Usage
print(to_binary(10)) # "1010"

```


* **Complexity**: Time O(\log N).
* **Why Interviewers Ask This**: To verify you understand the modulo/division algorithm.

#### 2. Hex String to Integer (Manual)

**Question**: Convert a hex string (e.g., "1A") to decimal manually. Assume valid input.

* **Solution**:
```python
def hex_to_int(s: str) -> int:
    hex_map = {
        '0':0, '1':1, '2':2, '3':3, '4':4, '5':5, '6':6, '7':7, '8':8, '9':9,
        'A':10, 'B':11, 'C':12, 'D':13, 'E':14, 'F':15
    }
    s = s.upper()
    result = 0
    power = 0

    # Iterate from right to left
    for char in reversed(s):
        result += hex_map[char] * (16 ** power)
        power += 1

    return result

# Usage
print(hex_to_int("1A")) # 16*1 + 10 = 26

```


* **Optimization**: `result = result * 16 + val` (Horner's Method) avoids calculating powers repeatedly.

#### 3. RGB to Hex Color Code

**Question**: Convert an RGB tuple `(255, 0, 128)` to a CSS hex string `"#FF0080"`.

* **Solution**:
```python
def rgb_to_hex(r: int, g: int, b: int) -> str:
    # :02X formats as Hex, Uppercase, padded to 2 chars
    return f"#{r:02X}{g:02X}{b:02X}"

# Usage
print(rgb_to_hex(255, 0, 128)) # "#FF0080"

```


* **Real App Usage**: Frontend CSS generation, Image processing.

---

### Medium Level (Questions 4–10)

#### 4. Excel Sheet Column Title (Base 26)

**Question**: Convert a column number to an Excel title. `1 -> A`, `26 -> Z`, `27 -> AA`.
*Note*: This is a "bijective" base-26 system (no zero).

* **Solution**:
```python
def convert_to_title(n: int) -> str:
    result = []
    while n > 0:
        n -= 1 # Adjust 1-based index to 0-based
        remainder = n % 26
        char = chr(ord('A') + remainder)
        result.append(char)
        n //= 26
    return "".join(reversed(result))

# Usage
print(convert_to_title(28)) # AB

```


* **Tricky Part**: The `n -= 1` is crucial because 'A' corresponds to 1, not 0. In standard Base 26, 0 would be 'A' and 10 would be 'BA'. Here, 26 is 'Z' and 27 is 'AA'.

#### 5. Base 62 Converter (URL Shortener Logic)

**Question**: Convert an integer ID (e.g., database Primary Key) to a Short URL string using characters `0-9`, `a-z`, `A-Z`.

* **Solution**:
```python
import string

# "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
BASE62 = string.digits + string.ascii_letters

def encode_base62(num: int) -> str:
    if num == 0: return BASE62[0]
    arr = []
    base = len(BASE62)
    while num:
        num, rem = divmod(num, base)
        arr.append(BASE62[rem])
    arr.reverse()
    return "".join(arr)

def decode_base62(s: str) -> int:
    base = len(BASE62)
    num = 0
    for char in s:
        num = num * base + BASE62.index(char)
        # Note: For production, use a dict for O(1) index lookup
    return num

# Usage
short = encode_base62(12345)
print(short) # "3D7" (Example)
print(decode_base62(short)) # 12345

```


* **Real App Usage**: TinyURL, YouTube video IDs (`dQw4w9WgXcQ`).

#### 6. IP Address to Integer (and back)

**Question**: Convert `"192.168.1.1"` to a 32-bit integer. Do not use `ipaddress` module (implement logic).

* **Solution**:
```python
def ip_to_int(ip: str) -> int:
    parts = ip.split('.')
    # Shift: 1st octet << 24, 2nd << 16, etc.
    return (int(parts[0]) << 24) + (int(parts[1]) << 16) + \
           (int(parts[2]) << 8) + int(parts[3])

def int_to_ip(num: int) -> str:
    return ".".join([
        str((num >> 24) & 0xFF),
        str((num >> 16) & 0xFF),
        str((num >> 8) & 0xFF),
        str(num & 0xFF)
    ])

# Usage
val = ip_to_int("192.168.1.1")
print(val) # 3232235777

```


* **Why Interviewers Ask This**: Networking fundamentals + bitwise shifting.

#### 7. Add Binary Strings

**Question**: Given two binary strings `a = "11"` and `b = "1"`, return their sum as a binary string `"100"`. Do **not** convert to int, add, and convert back (simulating large integer arithmetic).

* **Solution**:
```python
def add_binary(a: str, b: str) -> str:
    res = []
    i, j = len(a) - 1, len(b) - 1
    carry = 0

    while i >= 0 or j >= 0 or carry:
        val_a = int(a[i]) if i >= 0 else 0
        val_b = int(b[j]) if j >= 0 else 0

        total = val_a + val_b + carry
        carry = total // 2  # 2 becomes carry 1
        digit = total % 2   # 2 becomes digit 0

        res.append(str(digit))
        i -= 1
        j -= 1

    return "".join(reversed(res))

# Usage
print(add_binary("11", "1")) # "100"

```


* **Complexity**: Time O(\max(N, M)).

#### 8. Palindrome Number (in Binary)

**Question**: Check if the binary representation of a number is a palindrome. `9` (`1001`) -> True. `10` (`1010`) -> False.

* **Solution**:
```python
def is_binary_palindrome(n: int) -> bool:
    # Get binary string without '0b'
    b_str = bin(n)[2:]
    # Compare with reverse
    return b_str == b_str[::-1]

# Usage
print(is_binary_palindrome(9)) # True

```


* **Bitwise Optimization**: You can also construct the reverse integer using bitwise shifts to avoid string allocation (Advanced).

#### 9. Arbitrary Base Converter (Base N to Base M)

**Question**: Write a generic function `convert_base(num_str, from_base, to_base)`.

* **Solution**:
```python
def convert_base(num_str: str, from_base: int, to_base: int) -> str:
    # Step 1: Convert from_base to Decimal (Intermediate)
    decimal_val = int(num_str, from_base)

    # Step 2: Convert Decimal to to_base
    if decimal_val == 0: return "0"
    digits = []
    chars = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"

    while decimal_val:
        decimal_val, rem = divmod(decimal_val, to_base)
        digits.append(chars[rem])

    return "".join(reversed(digits))

# Usage
# Convert Hex "FF" (255) to Binary
print(convert_base("FF", 16, 2)) # "11111111"

```



#### 10. Manual `atoi` (String to Integer)

**Question**: Implement `my_atoi(s)` that converts a string to an integer, handling optional sign `+`/`-` and ignoring leading whitespace. (Like C++ `atoi`).

* **Solution**:
```python
def my_atoi(s: str) -> int:
    s = s.strip()
    if not s: return 0

    sign = 1
    index = 0

    if s[0] == '-':
        sign = -1
        index += 1
    elif s[0] == '+':
        index += 1

    result = 0
    while index < len(s) and s[index].isdigit():
        digit = int(s[index]) # or ord(s[index]) - ord('0')
        result = result * 10 + digit
        index += 1

    return sign * result

# Usage
print(my_atoi("   -42")) # -42
print(my_atoi("4193 with words")) # 4193 (Stops at non-digit)

```


* **Note**: Standard `int()` raises Error on "4193 with words". `atoi` logic traditionally parses until invalid char.
