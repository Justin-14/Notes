# Python Sets — Interview Guide (30 Questions)

This guide covers Python Sets with a focus on hash table mechanics, algorithmic efficiency, and mathematical set theory applied to backend logic. It assumes a 4-year developer level, moving quickly past basics into performance characteristics.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. What are the three defining characteristics of a Python Set?

* **Expected Answer**:
1. **Unordered**: Elements have no defined position or index.
2. **Unique**: No duplicate elements are allowed.
3. **Mutable**: You can add/remove items (though elements themselves must be immutable/hashable).


* **Clear Explanation**: Sets are implemented as hash tables (dictionaries with dummy values). This structure dictates their behavior.
* **Common Mistakes**: Trying to access a set by index `s[0]` (raises `TypeError`).
* **Why Interviewers Ask This**: To ensure you understand when to use a Set vs. a List.
* **Sample Code**:
```python
s = {1, 2, 2, 3}
print(s) # {1, 2, 3} (Duplicates removed, order not guaranteed)
# print(s[0]) # TypeError: 'set' object is not subscriptable

```



#### 2. How do you create an empty set? Why not `{}`?

* **Expected Answer**: You must use the constructor `set()`. Using `{}` creates an empty **Dictionary**.
* **Clear Explanation**: Python prioritizes dictionaries for the `{}` syntax because they are more fundamental to the language internals.
* **Sample Code**:
```python
a = {}
b = set()
print(type(a)) # <class 'dict'>
print(type(b)) # <class 'set'>

```



#### 3. `remove()` vs `discard()`: What is the difference?

* **Expected Answer**:
* `remove(x)`: Removes `x`. Raises **KeyError** if `x` is missing.
* `discard(x)`: Removes `x`. Does **nothing** if `x` is missing (no error).


* **Why Interviewers Ask This**: Error handling capability. If you are cleaning data and don't care if the item was already gone, `discard` avoids a `try/except` block.
* **Sample Code**:
```python
s = {1, 2}
s.discard(99) # No error
# s.remove(99) # Raises KeyError

```



#### 4. Can a Set contain a List? Why or why not?

* **Expected Answer**: No. Sets require all elements to be **hashable** (immutable). Lists are mutable, so they are not hashable.
* **Reasoning**: If a list inside a set changed (e.g., appended an item), its hash would change, and the set would lose track of which "bucket" the item is stored in.
* **Workaround**: Convert the list to a `tuple` first.
* **Sample Code**:
```python
# s = {[1, 2]} # TypeError: unhashable type: 'list'
s = {(1, 2)}   # Valid (Tuple is immutable)

```



#### 5. What is the time complexity of adding or checking membership in a Set?

* **Expected Answer**: **Average Case O(1)**. Worst Case O(N).
* **Clear Explanation**: Sets use a hash function to map a value directly to a memory address (bucket). In 99.9% of cases, this is instant.
* **Reasoning for Worst Case**: Hash Collisions. If all elements hash to the same bucket, the set degrades into a linked list, requiring linear search.
* **Comparison**: Lists are O(N) for membership checks (`x in list`). Sets are significantly faster for lookups.

#### 6. What does `set.pop()` do?

* **Expected Answer**: It removes and returns an arbitrary element from the set. If the set is empty, it raises `KeyError`.
* **Clear Explanation**: Since sets are unordered, "popping" does not mean "remove the last item" (there is no last item). It usually removes the first item found in the internal memory structure.
* **Warning**: Do not rely on `pop()` returning random elements for cryptography; it is deterministic based on hash values.
* **Sample Code**:
```python
s = {1, 2, 3}
val = s.pop()
print(val) # Could be 1, 2, or 3 depending on hash seed

```



---

### Medium Level (Questions 7–20)

#### 7. `union()` vs `update()`: Return values and Mutability.

* **Question**: What is the difference between `s1.union(s2)` and `s1.update(s2)`?
* **Expected Answer**:
* `union()` (or `|`): Returns a **new set**. The original sets are unchanged.
* `update()` (or `|=`): Modifies `s1` **in-place** to include elements from `s2`. Returns `None`.


* **Memory Implication**: `union` creates a copy, consuming more memory. `update` is more efficient if you don't need the original version of `s1`.
* **Sample Code**:
```python
s1 = {1, 2}
s2 = {3}
s3 = s1.union(s2)
print(s1) # {1, 2} (Unchanged)
s1.update(s2)
print(s1) # {1, 2, 3} (Changed)

```



#### 8. How does Python implement Set Intersection internally?

* **Question**: When running `small_set.intersection(huge_set)`, how does Python optimize?
* **Expected Answer**: Python iterates over the **smaller** set and checks for membership in the **larger** set.
* **Reasoning**:
* Let N = size of small set, M = size of large set.
* Iterating small set: O(N) iterations \times O(1) lookup = O(N).
* Iterating large set: O(M) iterations \times O(1) lookup = O(M).
* Since N < M, Python chooses the O(N) path automatically.


* **Why Interviewers Ask This**: Algorithmic intuition. It shows you understand cost optimization.

#### 9. Why is lookup in a Set O(1)? (Hash Table Mechanics)

* **Expected Answer**:
1. Python calculates `hash(item)`.
2. It uses the last few bits of the hash to find an index in an internal array (e.g., `index = hash & (size - 1)`).
3. It jumps directly to that memory slot.
4. If the slot is empty, the item isn't there. If occupied, it checks equality.


* **Diagram**:
```text
Value "Apple" -> Hash(123987) -> Index 3
Array: [ Empty, Empty, Empty, "Apple", Empty ]
                                 ^
                           Direct Access

```



#### 10. `True` vs `1` in Sets (The Equivalence Trap).

* **Question**: What is the length of `s = {True, 1, 1.0}`?
* **Expected Answer**: Length is **1**.
* **Clear Explanation**: In Python, `True == 1` and `True == 1.0`. They also have the same hash value.
* **Mechanism**: The set sees `True`. Next, it sees `1`. Since `1 == True` and `hash(1) == hash(True)`, the set treats `1` as a duplicate and discards it (or keeps the original `True` but ignores the new insertion).
* **Sample Code**:
```python
s = {True, 1, 1.0}
print(s) # {True}

```



#### 11. Can you modify a set while iterating over it?

* **Question**: What happens if you run `for item in s: s.remove(item)`?
* **Expected Answer**: `RuntimeError: Set changed size during iteration`.
* **Reasoning**: Python's iterators are not thread-safe or mutation-safe. Modifying the structure invalidates the internal pointers of the iterator.
* **Fix**: Iterate over a copy (`list(s)` or `s.copy()`).
```python
for item in list(s):
    s.remove(item)

```



#### 12. Set Comprehension Syntax.

* **Question**: Write a one-liner to create a set of squares of even numbers from 0 to 9.
* **Expected Answer**: `{x**2 for x in range(10) if x % 2 == 0}`.
* **Comparison**: It looks exactly like a list comprehension but uses curly braces `{}`.
* **Why Interviewers Ask This**: Idiomatic Python check.

#### 13. Is the order of elements in a set truly random?

* **Expected Answer**: It is "arbitrary" but deterministic within a single process run (mostly).
* **Nuance**: For integers, `hash(n) == n`. So `{1, 2, 3}` usually prints as `{1, 2, 3}`. However, for strings, Python enables **Hash Randomization** by default (salted hashes) to prevent DoS attacks. So `{"a", "b"}` might print differently in different runs of the script.
* **Why Interviewers Ask This**: Security awareness (Hash DoS).

#### 14. `issubset()` vs `<=` Operator.

* **Question**: Is there a difference between `A.issubset(B)` and `A <= B`?
* **Expected Answer**:
* `A <= B`: Requires both operands to be **Sets**.
* `A.issubset(B)`: Accepts any **Iterable** as the argument (automatically converts B to a temporary set or iterates it).


* **Sample Code**:
```python
s = {1, 2}
l = [1, 2, 3]
# print(s <= l)       # TypeError
print(s.issubset(l))  # True

```



#### 15. The Symmetric Difference Operator (`^`).

* **Question**: What does `A ^ B` return?
* **Expected Answer**: It returns elements that are in **either** A or B, but **not in both**.
* **Formula**: `(A | B) - (A & B)`.
* **Usage**: Finding discrepancies between two datasets (e.g., items in DB but not in Cache, AND items in Cache but not in DB).
* **Diagram**:
```text
Set A: [ 1, 2 ]
Set B: [ 2, 3 ]
A ^ B: { 1, 3 }  (2 is excluded because it's shared)

```



#### 16. What is a `frozenset` used for?

* **Question**: Give a concrete use case for `frozenset`.
* **Expected Answer**:
1. Using a set as a key in a dictionary (e.g., mapping a group of users `{u1, u2}` to a chat room ID).
2. Creating a set of sets (nested sets).


* **Sample Code**:
```python
groups = {}
users = frozenset(["alice", "bob"])
groups[users] = "Room 101"

```



#### 17. How to efficiently check if two sets have NO common elements?

* **Question**: Which method is optimized for this?
* **Expected Answer**: `s1.isdisjoint(s2)`.
* **Performance**: It returns `True` as soon as the first common element is found (Short-circuiting). It does not construct a new intersection set.
* **Mistake**: `len(s1 & s2) == 0`. This is slower because it constructs the entire intersection set in memory before checking length.

#### 18. Set implementation in CPython (Open Addressing).

* **Question**: Does Python use Chaining or Open Addressing for Set collisions?
* **Expected Answer**: **Open Addressing** (specifically, Quadratic Probing / Pseudo-random probing).
* **Mechanism**: If the target bucket is occupied (collision), Python checks a specific sequence of other buckets until an empty one is found. It does not create linked lists (chaining) like Java's HashMap.
* **Why Interviewers Ask This**: Deep internals knowledge.

#### 19. Removing duplicates from a list while preserving order.

* **Question**: `list(set(data))` removes duplicates but destroys order. How do you keep the order?
* **Expected Answer**: Since Python 3.7, dictionaries preserve insertion order.
* `list(dict.fromkeys(data))`


* **Explanation**: `dict.fromkeys` creates a dictionary where keys are the list items (deduplicated) and order is preserved.
* **Time Complexity**: O(N).

#### 20. Arithmetic Operations on Sets?

* **Question**: Can you add two sets `s1 + s2`?
* **Expected Answer**: No, the `+` operator is not defined for sets because "addition" is ambiguous (Union? concatenation?). You must use `|` (OR) for union.
* **Note**: The `-` operator *is* defined (Difference). `s1 - s2`.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Unique Character Check

**Question**: Write a function `has_unique_chars(s: str) -> bool` that returns True if a string has all unique characters. Ignore whitespace.

* **Solution**:
```python
def has_unique_chars(text: str) -> bool:
    # Filter out whitespace
    clean_text = text.replace(" ", "")
    # Compare length of string vs length of set
    return len(clean_text) == len(set(clean_text))

# Usage
print(has_unique_chars("a b c")) # True
print(has_unique_chars("a b a")) # False

```


* **Explanation**: Converting to a set removes duplicates. If the lengths match, no duplicates existed.
* **Complexity**: Time O(N), Space O(N).

#### 2. Find Missing Element

**Question**: You have a list of numbers `1` to `N`, but one is missing. Find the missing number using sets.

* **Solution**:
```python
def find_missing(nums: list[int]) -> int:
    if not nums: return 1
    n = len(nums) + 1
    # Create full set of expected numbers
    expected = set(range(1, n + 1))
    # Create set of actual numbers
    actual = set(nums)
    # The difference is the missing one
    return (expected - actual).pop()

# Usage
print(find_missing([1, 3, 4])) # 2

```


* **Explanation**: Set difference `expected - actual` returns items present in expected but not in actual.
* **Note**: Math approach `sum(expected) - sum(actual)` is O(1) space, but Set approach handles multiple missing items if needed.

#### 3. Common Elements (Intersection)

**Question**: Given three lists, find the elements common to all three.

* **Solution**:
```python
def common_elements(l1, l2, l3):
    # Convert first to set, intersect with others
    return list(set(l1).intersection(l2, l3))

# Usage
a = [1, 2, 3, 4]
b = [2, 4, 6]
c = [2, 8, 4]
print(common_elements(a, b, c)) # [2, 4]

```


* **Why Interviewers Ask This**: Testing knowledge that `intersection` accepts multiple arguments.

---

### Medium Level (Questions 4–10)

#### 4. List to Unique Tuple List

**Question**: You have a list of lists `[[1, 2], [1, 2], [3, 4]]`. Remove duplicates. (Recall: Lists are not hashable).

* **Solution**:
```python
def dedup_list_of_lists(data: list[list[int]]) -> list[list[int]]:
    # Strategy: Convert inner lists to tuples to make them hashable
    unique_tuples = set(tuple(x) for x in data)

    # Convert back to lists
    return [list(x) for x in unique_tuples]

# Usage
data = [[1, 2], [1, 2], [3, 4]]
print(dedup_list_of_lists(data)) # [[1, 2], [3, 4]]

```


* **Common Mistake**: Trying `set(data)` directly crashes.
* **Developer Scenario**: Cleaning JSON logs where entries might be duplicated arrays.

#### 5. Longest Consecutive Sequence

**Question**: Given an unsorted array of integers, find the length of the longest consecutive elements sequence. Algorithm must be O(N).
`[100, 4, 200, 1, 3, 2]` -> Result 4 (`1, 2, 3, 4`).

* **Solution**:
```python
def longest_consecutive(nums: list[int]) -> int:
    num_set = set(nums)
    longest = 0

    for num in num_set:
        # Only start counting if 'num' is the start of a sequence
        # (i.e., num-1 is NOT in the set)
        if (num - 1) not in num_set:
            current_num = num
            current_streak = 1

            while (current_num + 1) in num_set:
                current_num += 1
                current_streak += 1

            longest = max(longest, current_streak)

    return longest

# Usage
print(longest_consecutive([100, 4, 200, 1, 3, 2])) # 4

```


* **Explanation**:
1. Convert to set for O(1) lookups.
2. Iterate. If `num-1` exists, this number is part of an existing sequence (not the start), so skip it. This ensures we process each sequence only once.
3. If `num-1` doesn't exist, this is the start. While loop checks `num+1`, `num+2`...


* **Time Complexity**: O(N). Although there is a nested while loop, each number is visited at most twice (once in for loop, once in while loop).

#### 6. Data Reconciliation (Added, Removed, Unchanged)

**Question**: You have `old_state` (list of IDs) and `new_state` (list of IDs). Return three lists: `added`, `removed`, `kept`.

* **Solution**:
```python
def reconcile_data(old: list[int], new: list[int]):
    old_s = set(old)
    new_s = set(new)

    added = list(new_s - old_s)
    removed = list(old_s - new_s)
    kept = list(old_s & new_s)

    return added, removed, kept

# Usage
old = [1, 2, 3]
new = [3, 4, 5]
print(reconcile_data(old, new))
# Added: [4, 5], Removed: [1, 2], Kept: [3]

```


* **Real App Usage**: Syncing a local frontend cache with a backend API response. Determining which DOM elements to insert/delete.

#### 7. Check if Array Pairs are Divisible by k

**Question**: Given an array of integers and integer `k`, check if the array can be divided into pairs such that the sum of every pair is divisible by `k`.

* **Solution**:
```python
def can_arrange(arr: list[int], k: int) -> bool:
    # We need to track frequencies of remainders
    # While sets handle uniqueness, here we need counts,
    # but understanding the set-logic of remainders is key.
    remainder_counts = {}

    for x in arr:
        rem = x % k
        remainder_counts[rem] = remainder_counts.get(rem, 0) + 1

    for rem in remainder_counts:
        count = remainder_counts[rem]
        if rem == 0:
            # Pairs of (0, 0) -> Count must be even
            if count % 2 != 0: return False
        elif 2 * rem == k:
            # Remainder is exactly half of k (e.g., k=10, rem=5).
            # Pairs of (5, 5) -> Count must be even
            if count % 2 != 0: return False
        else:
            # Count of 'rem' must match count of 'k - rem'
            complement = k - rem
            if remainder_counts.get(complement, 0) != count:
                return False

    return True

```


* **Why this is here**: It tests the boundary of Sets. A pure `Set` cannot solve this because duplicates matter (frequencies). The candidate must realize they need a Frequency Map (Counter), which is conceptually a "Multiset".

#### 8. Jaccard Similarity Coefficient

**Question**: Calculate the Jaccard Similarity between two text documents (lists of words). Formula: `Size(Intersection) / Size(Union)`.

* **Solution**:
```python
def jaccard_similarity(doc1: list[str], doc2: list[str]) -> float:
    s1 = set(doc1)
    s2 = set(doc2)

    intersection = len(s1 & s2)
    union = len(s1 | s2)

    if union == 0: return 1.0 # Both empty
    return intersection / union

# Usage
d1 = ["ai", "model", "learning"]
d2 = ["machine", "learning", "model"]
print(jaccard_similarity(d1, d2)) # 2/4 = 0.5

```


* **Real App Usage**: Search engines, recommendation systems (finding similar users).

#### 9. Filtering Blocklist in Stream

**Question**: You are processing a stream of 1 million user comments. You have a `blocklist` of 5,000 banned words. Efficiently censor the comments.

* **Solution**:
```python
def censor_comments(comments: list[str], blocklist: list[str]) -> list[str]:
    # CRITICAL: Convert list to set for O(1) lookups.
    # If we kept blocklist as list, complexity would be O(N*M).
    # With set, it is O(N).
    banned_set = set(blocklist)

    censored = []
    for comment in comments:
        words = comment.split()
        clean_words = [
            "****" if word in banned_set else word
            for word in words
        ]
        censored.append(" ".join(clean_words))
    return censored

```


* **Performance Note**: Looking up a word in a list of 5000 items is 5000x slower than looking it up in a set. For 1 million comments, this is the difference between running in 2 seconds vs 2 hours.

#### 10. Power Set (All Subsets)

**Question**: Generate the Power Set of `[1, 2, 3]` (all possible subsets).

* **Solution**:
```python
def power_set(nums: list[int]) -> list[list[int]]:
    # Iterative approach using cascading additions
    subsets = [[]]

    for num in nums:
        # For every existing subset, create a new one adding the current num
        new_subsets = [curr + [num] for curr in subsets]
        subsets.extend(new_subsets)

    return subsets

# Usage
# []
# [], [1]
# [], [1], [2], [1, 2]
# [], [1], [2], [1, 2], [3], [1, 3], [2, 3], [1, 2, 3]
print(power_set([1, 2, 3]))

```


* **Connection to Sets**: While the output is a list of lists, logically this represents the mathematical power set 2^S.
* **Complexity**: Time O(2^N).
