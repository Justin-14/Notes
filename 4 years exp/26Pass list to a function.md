# Python Pass list to a function— Interview Guide (30 Questions)

**Role:** Senior Technical Interviewer

**Target Audience:** Python Developer (4 Years Experience)

**Topic:** Python Lists (Mastery & Internals)

**Focus:** Internals, Memory Management, Performance, and "Pass-by-Assignment" Mechanics.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Part A: Easy Level (6 Questions)

#### 1. How are lists passed to functions in Python? Value or Reference?

* **Expected Answer:** Python uses a mechanism known as "Call by Object Reference" or "Pass by Assignment". When you pass a list to a function, you are passing the *reference* (memory address) to the list object, not a copy of the list. Since lists are mutable, changes made to the list inside the function (like `.append()`) will affect the original list. However, reassigning the variable name inside the function will *not* affect the original variable.
* **Explanation:** The parameter inside the function becomes a new alias pointing to the same object in memory.
* **Reasoning:** Understanding this is crucial to avoid side effects where a function accidentally modifies global state.
* **Complexity:** O(1) to pass.
* **Common Mistakes:** Thinking it is "Pass by Reference" (like C++) or "Pass by Value". It is neither; it is passing the object reference by value.
* **Why Interviewers Ask:** To test understanding of mutability and scope.
* **Sample Code:**
```python
def modify_list(lst):
    lst.append(4)      # Modifies the object
    lst = [10, 11]     # Rebinds local name 'lst', original untouched

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)         # Output: [1, 2, 3, 4]

```



#### 2. What is the difference between `append()` and `extend()`?

* **Expected Answer:** `append()` adds its argument as a *single element* to the end of the list. `extend()` iterates over its argument (which must be an iterable) and adds each element to the list.
* **Explanation:** `append` increases length by 1. `extend` increases length by `len(iterable)`.
* **Reasoning:** `extend` is generally more performant than a loop with `append` because it is implemented in C.
* **Complexity:**
* `append`: O(1) amortized.
* `extend`: O(k) where k is the length of the new iterable.


* **Common Mistakes:** Using `append` when they meant to merge two lists, resulting in a nested list: `[1, 2, [3, 4]]`.
* **Why Interviewers Ask:** Basic API knowledge check.
* **Sample Code:**
```python
a = [1, 2]
b = [3, 4]
a.append(b)  # a is [1, 2, [3, 4]]

x = [1, 2]
x.extend(b)  # x is [1, 2, 3, 4]

```



#### 3. Explain negative indexing in Python lists.

* **Expected Answer:** Negative indexing allows accessing elements from the end of the list. `-1` refers to the last item, `-2` to the second to last, and so on.
* **Explanation:** Python internally translates index `i` (where `i < 0`) to `len(list) + i`.
* **Reasoning:** Syntactic sugar that simplifies getting tail elements without calculating length explicitly.
* **Complexity:** O(1).
* **Common Mistakes:** `IndexError` if the negative index exceeds the list length (e.g., `list[-1]` on an empty list).
* **Why Interviewers Ask:** Checks familiarity with Pythonic syntax.
* **Sample Code:**
```python
data = ['a', 'b', 'c', 'd']
print(data[-1]) # 'd'
print(data[-3]) # 'b'

```



#### 4. What is the difference between `list.sort()` and `sorted(list)`?

* **Expected Answer:** `list.sort()` sorts the list *in-place* and returns `None`. `sorted(list)` creates a *new* sorted list and returns it, leaving the original unchanged.
* **Explanation:** `sort()` modifies the existing object. `sorted()` accepts any iterable but always returns a list.
* **Reasoning:** Use `sort()` to save memory if you don't need the original order. Use `sorted()` if you need to preserve the original data.
* **Complexity:** O(N log N) for both (Timsort).
* **Common Mistakes:** Assigning the result of `sort()` to a variable: `x = my_list.sort()` results in `x` being `None`.
* **Why Interviewers Ask:** Tests knowledge of in-place operations vs. functional return values.
* **Sample Code:**
```python
nums = [3, 1, 2]
s_nums = sorted(nums) # s_nums=[1,2,3], nums=[3,1,2]
nums.sort()           # nums=[1,2,3], returns None

```



#### 5. How do you copy a list effectively?

* **Expected Answer:** Shallow copies can be made using slicing `[:]`, the `list()` constructor, or `copy.copy()`. Deep copies (recursive copy) require `copy.deepcopy()`.
* **Explanation:** Slicing creates a new list object containing references to the same items.
* **Reasoning:** Crucial when working with lists of mutable objects (like lists of lists).
* **Complexity:** O(N).
* **Common Mistakes:** Thinking `new_list = old_list` creates a copy. It only creates a reference alias.
* **Why Interviewers Ask:** Fundamental memory management check.
* **Sample Code:**
```python
original = [1, 2, 3]
shallow = original[:] 
alias = original

shallow[0] = 99
# original is still [1, 2, 3], alias is [1, 2, 3]

```



#### 6. What is List Comprehension?

* **Expected Answer:** A concise way to create lists based on existing iterables. It follows the syntax `[expression for item in iterable if condition]`.
* **Explanation:** generally faster than equivalent `for` loops because the iteration is optimized in C.
* **Reasoning:** Pythonic style; reduces lines of code and improves readability for simple transformations.
* **Complexity:** O(N).
* **Common Mistakes:** Overusing nested comprehensions leading to unreadable code.
* **Why Interviewers Ask:** Checks ability to write Pythonic code.
* **Sample Code:**
```python
squares = [x**2 for x in range(5) if x % 2 == 0]
# Result: [0, 4, 16]

```



---

### Part B: Medium Level (14 Questions)

#### 7. How are Python lists implemented internally?

* **Expected Answer:** Python lists are dynamic arrays (specifically, contiguous arrays of pointers to objects). They are implemented as a C struct containing `ob_item` (pointer to the array of pointers) and `allocated` (total capacity).
* **Explanation:** When you append, Python uses the available capacity. If full, it allocates a larger chunk of memory (over-allocation), copies pointers, and frees the old block.
* **Reasoning:** This explains why `append` is amortized O(1) but inserting at index 0 is O(N).
* **Complexity:** N/A (Implementation detail).
* **Common Mistakes:** Thinking they are Linked Lists. They are **NOT** linked lists.
* **Why Interviewers Ask:** To verify knowledge of data structures behind the syntax.
* **ASCII Diagram:**
```text
Python List Object
+---------------+
| Size: 3       |
| Alloc: 4      |    Memory Block (pointers)
| Items: *------+--> [ *ptr1 | *ptr2 | *ptr3 | Empty ]
+---------------+       |       |       |
                        v       v       v
                       (1)     (2)     (3)

```



#### 8. What is the growth pattern (resizing strategy) of a list?

* **Expected Answer:** Python lists use an over-allocation strategy to achieve amortized O(1) appends. The growth factor is approximately 1.125 (plus a small constant) in newer versions (CPython). Roughly: `new_allocated = new_size + (new_size >> 3) + 6`.
* **Explanation:** It does not double (2x) exactly like some C++ vectors to avoid memory wastage, but it grows exponentially to ensure resizing happens infrequently.
* **Reasoning:** Frequent reallocations would make `append` O(N).
* **Complexity:** Resizing is O(N), but happens rarely.
* **Common Mistakes:** Assuming it always doubles in size.
* **Why Interviewers Ask:** Deep dive into performance characteristics.
* **Sample Code:**
```python
import sys
l = []
print(sys.getsizeof(l)) # Size changes non-linearly as you append

```



#### 9. Why is checking `element in list` O(N)?

* **Expected Answer:** Lists are not hashed or sorted by default. To find an element, Python performs a linear scan from the beginning to the end, comparing each element using equality (`==`).
* **Explanation:** In the worst case (element at end or not present), it visits every item.
* **Reasoning:** If frequent lookups are required, a `set` or `dict` (O(1)) should be used instead.
* **Complexity:** O(N).
* **Common Mistakes:** Using lists for large whitelists/blacklists in high-traffic APIs.
* **Why Interviewers Ask:** To see if the candidate knows when *not* to use a list.

#### 10. Explain the behavior of `*` operator on lists containing mutable objects.

* **Expected Answer:** Using `[x] * n` creates a list of `n` references to the *same* object `x`. If `x` is mutable (like a list), modifying one element modifies all.
* **Explanation:** It copies the reference, not the value.
* **Reasoning:** This is a classic "gotcha" in initialization.
* **Common Mistakes:** initializing a matrix as `grid = [[0]*3]*3`.
* **Why Interviewers Ask:** Common bug source in grid/graph problems.
* **Sample Code:**
```python
# BAD
row = [0, 0]
matrix = [row] * 2  # [[0, 0], [0, 0]]
matrix[0][0] = 1
# Result: [[1, 0], [1, 0]] -> Both rows changed!

# GOOD
matrix = [[0, 0] for _ in range(2)]

```



#### 11. What happens to the memory when you `pop()` from a list?

* **Expected Answer:** `pop()` removes the item and decrements the size. If the list becomes significantly smaller than its allocated capacity, Python *may* shrink the underlying array, but CPython is generally lazy about shrinking to avoid "thrashing" (rapid grow/shrink cycles).
* **Explanation:** The reference to the object is removed from the list. If that was the last reference, the object is garbage collected.
* **Reasoning:** Memory isn't always immediately returned to the OS, but it is available for the list to grow again.
* **Complexity:** O(1) for `pop()` (last), O(N) for `pop(0)`.
* **Why Interviewers Ask:** Memory optimization awareness.

#### 12. Explain the difference between `list += iterable` and `list = list + iterable`.

* **Expected Answer:**
1. `list += iterable` (In-place): Calls `__iadd__`, which effectively runs `extend()`. It modifies the list in place and does not create a new list object.
2. `list = list + iterable` (Concatenation): Creates a *new* list object containing elements from both, then rebinds the variable.


* **Explanation:** `+=` is faster and memory efficient. `+` creates garbage (the old list).
* **Reasoning:** In `def func(l): l += [1]`, the caller sees the change. In `l = l + [1]`, the caller does not (local rebinding).
* **Complexity:** `+=` is O(N_new). `+` is O(N_total).
* **Common Mistakes:** Thinking they are identical.
* **Why Interviewers Ask:** Distinguishing in-place vs. new object creation.
* **Sample Code:**
```python
a = [1]; b = a
a += [2] # b is also [1, 2]

x = [1]; y = x
x = x + [2] # x is [1, 2], y is [1]

```



#### 13. How does Python handle list resizing during iteration?

* **Expected Answer:** Modifying a list (adding/removing items) while iterating over it using a `for` loop is dangerous and can lead to skipped items or runtime errors.
* **Explanation:** The iterator maintains an internal index. If you remove item at index `i`, the next item shifts to `i`. The iterator increments to `i+1`, skipping the shifted item.
* **Reasoning:** Always iterate over a copy (`for x in list[:]`) or iterate backwards if modifying.
* **Why Interviewers Ask:** Tests practical debugging skills regarding loops.

#### 14. What are the time complexities of `insert(0, x)` vs `append(x)`? Why?

* **Expected Answer:** `append(x)` is O(1) amortized. `insert(0, x)` is O(N).
* **Explanation:** `append` just places the item at the end. `insert(0, x)` must shift *every* existing element one spot to the right in memory to make space at the front.
* **Reasoning:** Using a list as a Queue (FIFO) is inefficient.
* **Comparison:** Use `collections.deque` for O(1) appends and pops from both ends.
* **Why Interviewers Ask:** Performance optimization checks.

#### 15. What is the limit on the size of a Python list?

* **Expected Answer:** It is limited by the addressable memory of the system and `sys.maxsize`. In a 64-bit system, the index max is roughly .
* **Explanation:** Practically, you will run out of RAM before hitting the index limit.
* **Reasoning:** Python lists deal with pointers (8 bytes on 64-bit). Storing 1 billion pointers requires ~8GB RAM just for the list structure, excluding the objects themselves.
* **Why Interviewers Ask:** Scaling limitations.

#### 16. Can you use a list as a dictionary key?

* **Expected Answer:** No, lists are unhashable because they are mutable.
* **Explanation:** Dictionary keys must be hashable so their hash value doesn't change. If a list changed content, its hash would change, breaking the hash table lookup.
* **Reasoning:** You must use a `tuple` instead if you need an ordered sequence as a key.
* **Why Interviewers Ask:** Understanding of Hash Maps and Python Data Model (`__hash__`).
* **Sample Code:**
```python
d = {}
d[[1, 2]] = "error" # TypeError: unhashable type: 'list'
d[(1, 2)] = "ok"    # Valid

```



#### 17. How does `list.reverse()` differ from `list[::-1]` in terms of memory?

* **Expected Answer:** `list.reverse()` modifies the list in-place (O(1) extra space). `list[::-1]` creates a full shallow copy (O(N) extra space).
* **Explanation:** Slicing allocates new memory for the new list pointers.
* **Reasoning:** If memory is tight, use `reverse()`. If you need to keep original, use slice.
* **Why Interviewers Ask:** Space complexity awareness.

#### 18. What is the logic behind Python's Timsort algorithm used in list sort?

* **Expected Answer:** Timsort is a hybrid stable sorting algorithm, derived from Merge Sort and Insertion Sort.
* **Explanation:** It looks for natural "runs" (already sorted subsequences) in the data. It sorts small runs with Insertion Sort (fast for small arrays) and merges them using Merge Sort logic.
* **Reasoning:** Real-world data is often partially sorted. Timsort performs in O(N) for nearly sorted data, vs O(N log N) worst case.
* **Complexity:** O(N log N) worst, O(N) best. Space O(N).
* **Why Interviewers Ask:** Advanced algorithmic knowledge.

#### 19. How do you flatten a shallow nested list `[[1,2], [3,4]]` efficiently?

* **Expected Answer:** Using `itertools.chain` or a nested comprehension.
* **Explanation:** `sum(list, [])` works but is quadratic O(N^2) because it creates a new list for every sublist addition. `itertools.chain` is O(N) and lazy.
* **Sample Code:**
```python
import itertools
nested = [[1, 2], [3, 4]]
flat = list(itertools.chain.from_iterable(nested))
# OR
flat = [item for sublist in nested for item in sublist]

```


* **Why Interviewers Ask:** Detecting inefficient patterns like `sum()` on lists.

#### 20. What are `__getitem__`, `__setitem__`, and `__delitem__`?

* **Expected Answer:** These are Magic Dunder Methods that allow objects to emulate list behavior (operator overloading for `[]`).
* **Explanation:**
* `obj[i]` calls `__getitem__(i)`.
* `obj[i] = x` calls `__setitem__(i, x)`.
* `del obj[i]` calls `__delitem__(i)`.


* **Reasoning:** Implementing these allows custom classes to behave like lists.
* **Why Interviewers Ask:** OOP and Custom Data Structure implementation.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Part A: Easy Level (3 Questions)

#### 1. Filter Even Numbers (In-place vs New List)

* **Question:** Write a function that takes a list of integers and removes all odd numbers. Do this first by returning a new list, then implement it **in-place** to save memory.
* **Scenario:** Data cleaning pipeline where invalid entries must be purged.
* **Solution:**
```python
def filter_evens_new(nums: list[int]) -> list[int]:
    return [x for x in nums if x % 2 == 0]

def filter_evens_inplace(nums: list[int]) -> None:
    # We iterate backwards to avoid index shifting issues
    for i in range(len(nums) - 1, -1, -1):
        if nums[i] % 2 != 0:
            del nums[i]

```


* **Explanation:**
* **New:** Uses list comprehension (Standard Python).
* **In-place:** Iterating forward and deleting causes index skipping. Iterating backward `range(len-1, -1, -1)` is the safe way to delete items in-place.


* **Time/Space:** New: O(N) Time, O(N) Space. In-place: O(N) Time, O(1) Space (though `del` is O(N) shifting, total is O(N^2) worst case technically, usually acceptable for easy scripts).
* **Real App Usage:** Removing invalid user IDs from a processing queue in a background worker.

#### 2. Rotate List (Slicing)

* **Question:** Given a list and an integer `k`, rotate the list to the right by `k` steps.
* **Scenario:** UI carousels (Netflix banners) cycling through content.
* **Solution:**
```python
def rotate_list(nums: list[int], k: int) -> list[int]:
    if not nums: return nums
    k = k % len(nums) # Optimize if k > len
    return nums[-k:] + nums[:-k]

```


* **Explanation:**
* `nums[-k:]`: Gets the last k elements (which move to front).
* `nums[:-k]`: Gets the rest (which move to back).
* Combine with `+`.


* **Time/Space:** O(N) Time, O(N) Space.
* **Common Mistakes:** Not handling `k > len(nums)` or empty lists.

#### 3. Merge Two Sorted Lists

* **Question:** Merge two sorted lists into a single sorted list.
* **Scenario:** Aggregating search results from two different sorted database shards.
* **Solution:**
```python
def merge_sorted(l1: list[int], l2: list[int]) -> list[int]:
    result = []
    i, j = 0, 0
    while i < len(l1) and j < len(l2):
        if l1[i] < l2[j]:
            result.append(l1[i])
            i += 1
        else:
            result.append(l2[j])
            j += 1
    # Append remaining elements
    result.extend(l1[i:])
    result.extend(l2[j:])
    return result

```


* **Explanation:** Standard "Two Pointer" approach. Compare heads, append smaller, move pointer.
* **Time/Space:** O(N+M) Time, O(N+M) Space.
* **Why not `sorted(l1+l2)`?** That is O(N log N). This merge is O(N).

---

### Part B: Medium Level (7 Questions)

#### 4. Find Duplicate Number (Floyd’s Cycle / Set)

* **Question:** Given a list of integers where every integer appears once except for one duplicate, find the duplicate.
* **Scenario:** Checking for duplicate transaction IDs in a payment batch.
* **Solution:**
```python
def find_duplicate(nums: list[int]) -> int:
    seen = set()
    for num in nums:
        if num in seen:
            return num
        seen.add(num)
    return -1

```


* **Explanation:** Uses a Hash Set for O(1) lookups.
* **Time/Space:** O(N) Time, O(N) Space.
* **Common Mistakes:** Using nested loops (O(N^2)).

#### 5. Three Sum (Indices)

* **Question:** Given an integer array nums, return all triplets `[nums[i], nums[j], nums[k]]` such that they sum to 0. No duplicate triplets.
* **Scenario:** Financial fraud detection (finding 3 transactions that net to zero).
* **Solution:**
```python
def three_sum(nums: list[int]) -> list[list[int]]:
    nums.sort()
    res = []
    for i in range(len(nums) - 2):
        # Skip duplicates for i
        if i > 0 and nums[i] == nums[i-1]:
            continue

        l, r = i + 1, len(nums) - 1
        while l < r:
            s = nums[i] + nums[l] + nums[r]
            if s < 0:
                l += 1
            elif s > 0:
                r -= 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                # Skip duplicates for l and r
                while l < r and nums[l] == nums[l+1]: l += 1
                while l < r and nums[r] == nums[r-1]: r -= 1
                l += 1; r -= 1
    return res

```


* **Explanation:**
1. Sort the list.
2. Fix one number (`i`), use two pointers (`l`, `r`) for the remaining sum.
3. Skip duplicates to ensure unique triplets.


* **Time/Space:** O(N^2) Time, O(1) Space (ignoring output).

#### 6. Product of Array Except Self

* **Question:** Given `nums`, return an array `output` where `output[i]` is the product of all elements of `nums` except `nums[i]`. **Solve without division.**
* **Scenario:** Matrix calculations in Machine Learning (normalization).
* **Solution:**
```python
def product_except_self(nums: list[int]) -> list[int]:
    n = len(nums)
    res = [1] * n

    # Left Pass: res[i] contains product of nums[0]...nums[i-1]
    prefix = 1
    for i in range(n):
        res[i] = prefix
        prefix *= nums[i]

    # Right Pass: Multiply res[i] by product of nums[i+1]...nums[n]
    postfix = 1
    for i in range(n - 1, -1, -1):
        res[i] *= postfix
        postfix *= nums[i]

    return res

```


* **Explanation:** Calculates prefix products and suffix products in two passes.
* **Time/Space:** O(N) Time, O(1) Extra Space (excluding result list).
* **Common Mistakes:** Trying to use division (fails if a zero exists).

#### 7. Maximum Subarray Sum (Kadane’s Algorithm)

* **Question:** Find the contiguous subarray which has the largest sum.
* **Scenario:** Analyzing stock prices to find the best window of profit.
* **Solution:**
```python
def max_sub_array(nums: list[int]) -> int:
    max_so_far = nums[0]
    current_max = nums[0]

    for i in range(1, len(nums)):
        # Should I start new at nums[i], or extend previous subarray?
        current_max = max(nums[i], current_max + nums[i])
        max_so_far = max(max_so_far, current_max)

    return max_so_far

```


* **Explanation:**
* Iterate through list.
* Keep a running count `current_max`. If adding the current number makes the sum smaller than the number itself, start over.


* **Time/Space:** O(N) Time, O(1) Space.

#### 8. Flatten Nested List Iterator

* **Question:** Implement an iterator to flatten a nested list of integers (arbitrary depth). `[1, [4, [6]]]` -> `1, 4, 6`.
* **Scenario:** Processing JSON responses where data might be deeply nested.
* **Solution:**
```python
def flatten_generator(nested_list):
    for item in nested_list:
        if isinstance(item, list):
            yield from flatten_generator(item)
        else:
            yield item

# Usage
data = [1, [4, [6, 7]], 8]
flat = list(flatten_generator(data)) # [1, 4, 6, 7, 8]

```


* **Explanation:** Uses Python's `yield from` for recursive generator delegation.
* **Time/Space:** O(N) Time, O(D) Space (stack depth).
* **Real App Usage:** Parsing inconsistent configuration files in DevOps tools.

#### 9. Longest Substring Without Repeating Characters (Sliding Window)

* **Question:** Given a list of characters (or string), find the length of the longest sub-segment without repeating characters.
* **Scenario:** Network packet analysis (finding unique sequences).
* **Solution:**
```python
def length_of_longest_substring(s: list[str]) -> int:
    char_map = {}
    left = 0
    max_len = 0

    for right in range(len(s)):
        if s[right] in char_map and char_map[s[right]] >= left:
            # Move left pointer just past the previous instance of this char
            left = char_map[s[right]] + 1

        char_map[s[right]] = right
        max_len = max(max_len, right - left + 1)

    return max_len

```


* **Explanation:** Sliding Window pattern. `right` expands, `left` contracts when a duplicate is found.
* **Time/Space:** O(N) Time, O(min(N, A)) Space where A is alphabet size.

#### 10. Trapping Rain Water

* **Question:** Given a list of non-negative integers representing an elevation map (width 1), compute how much water it can trap.
* **Scenario:** Image processing (filling gaps), Geographic data analysis.
* **Solution:**
```python
def trap(height: list[int]) -> int:
    if not height: return 0
    l, r = 0, len(height) - 1
    left_max, right_max = height[l], height[r]
    water = 0

    while l < r:
        if left_max < right_max:
            l += 1
            left_max = max(left_max, height[l])
            water += left_max - height[l]
        else:
            r -= 1
            right_max = max(right_max, height[r])
            water += right_max - height[r]
    return water

```


* **Explanation:** Two pointers. Water level at any point is determined by the `min(max_left, max_right)`. We process from the smaller side inward.
* **Time/Space:** O(N) Time, O(1) Space.
* **Why Interviewers Ask:** Classic hard problem requiring optimized space logic.

---
