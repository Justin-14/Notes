# Python Sets — Interview Guide (30 Questions)

**Target Audience:** Python Developer (4 Years Experience)

**Topic:** Python Sets (Hashing, Mathematical Operations, Performance)

**Total Questions:** 30 (20 Theory, 10 Coding)

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Part A: Easy Level (6 Questions)

#### 1. What is a Set?

**Question:** Define a Python `set`. How does it differ from a list?

**Expected Answer:** A set is an unordered collection of unique, hashable elements. Unlike lists, sets do not allow duplicate values and do not track the order of insertion.

---

#### 2. Set Creation Syntax

**Question:** How do you create an empty set? Why is `{}` not sufficient?

**Expected Answer:** Use `set()`. Using `{}` creates an empty **dictionary**.

---

#### 3. Membership Testing Performance

**Question:** What is the time complexity of `x in my_set`? Why is it faster than a list?

**Expected Answer:** Average case O(1). It uses a hash table, meaning it jumps directly to the memory bucket where the item *should* be, rather than scanning from index 0.

---

#### 4. Adding Elements: `add()` vs `update()`

**Question:** How do these two methods differ?

**Expected Answer:** `add(item)` inserts a single element. `update(iterable)` takes an iterable (list, tuple, another set) and adds all its elements individually.

---

#### 5. Removing Elements: `remove()` vs `discard()`

**Question:** What happens if you try to remove an item that doesn't exist?

**Expected Answer:** `remove(x)` raises a `KeyError`. `discard(x)` does nothing (fails silently).

---

#### 6. Set Mutability

**Question:** Are sets mutable? Can they contain lists?

**Expected Answer:** Sets are mutable (you can add/remove items), but the **elements** within them must be immutable and hashable. Therefore, a set cannot contain a list.

---

### Part B: Medium Level (14 Questions)

#### 7. The `frozenset`

**Question:** What is a `frozenset` and when would you use one?

**Expected Answer:** It is an immutable version of a set. Because it is immutable, it is **hashable**, meaning you can use a `frozenset` as a dictionary key or as an element inside another set.

---

#### 8. Set Mathematics: Union & Intersection

**Question:** Explain `|` and `&`.

**Expected Answer:** `a | b` (Union) returns all elements from both sets. `a & b` (Intersection) returns only elements present in both.

---

#### 9. Difference vs. Symmetric Difference

**Question:** Explain `a - b` vs `a ^ b`.

**Expected Answer:** 
* `a - b`: Elements in `a` that are NOT in `b`.
* `a ^ b`: Elements in either `a` or `b`, but NOT both (exclusive OR).

---

#### 10. Internals: How Sets Store Data

**Question:** How does a set handle a "hash collision"?

**Expected Answer:** Similar to dictionaries, sets use a hash table with open addressing. If two items hash to the same bucket, Python probes for the next available slot.

---

#### 11. `isdisjoint()`, `issubset()`, and `issuperset()`

**Question:** Define these three boolean checks.

**Expected Answer:**
* `isdisjoint`: True if the sets have zero common elements.
* `issubset`: True if all elements of Set A are in Set B.
* `issuperset`: True if Set A contains all elements of Set B.

---

#### 12. List to Set Conversion Complexity

**Question:** What is the time complexity of `my_set = set(my_list)`?

**Expected Answer:** O(N), as it must iterate through the list and hash each element into the new set.

---

#### 13. Pop Behavior in Sets

**Question:** What does `my_set.pop()` do?

**Expected Answer:** It removes and returns an **arbitrary** element. Since sets are unordered, you cannot predict which item will be removed.

---

#### 14. Clearing a Set

**Question:** Difference between `s = set()` and `s.clear()`?

**Expected Answer:** `s.clear()` empties the existing set object in-place (affecting all references). `s = set()` reassigns the variable to a brand new object.

---

#### 15. Set Comprehensions

**Question:** Write a set comprehension that stores the squares of even numbers from 1 to 10.

**Expected Answer:** `{x**2 for x in range(1, 11) if x % 2 == 0}`.

---

#### 16. Memory Overhead

**Question:** Does a set use more memory than a list of the same elements?

**Expected Answer:** Yes. Sets require extra memory for the hash table (to reduce collisions and allow O(1) access).

---

#### 17. Use Case: Deduplication

**Question:** How would you remove duplicates from a list while preserving order?

**Expected Answer:** Converting to a set and back to a list loses order. To preserve order, use `dict.fromkeys(my_list).keys()`.

---

#### 18. Set Equality

**Question:** Does `{1, 2, 3} == {3, 2, 1}`?

**Expected Answer:** Yes. Order does not matter for set equality.

---

#### 19. Intersection with Non-Set Iterables

**Question:** Can you run `set_a.intersection([1, 2, 3])`?

**Expected Answer:** Yes. The method versions (like `.intersection()`) accept any iterable, whereas the operators (like `&`) require both sides to be sets.

---

#### 20. Why Interviewers Ask About Sets

**Question:** Why are sets critical for API performance?

**Expected Answer:** They are the primary tool for eliminating O(N^2) "nested loop" lookups. Checking if a User ID exists in a "Blocked" list should always be a set lookup.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Part A: Easy Level (3 Questions)

#### 21. Unique Word Count

**Question:** Given a sentence, return the number of unique words (case-insensitive).

**Solution:**
```python
def count_unique(text):
    words = text.lower().split()
    return len(set(words))
```

---

#### 22. Any Duplicates?

**Question:** Return `True` if a list contains any duplicate elements.

**Solution:**
```python
def has_duplicates(nums):
    return len(nums) != len(set(nums))
```

---

#### 23. Common Friends

**Question:** Given two lists of friends, return a set of friends they have in common.

**Solution:**
```python
def common_friends(list1, list2):
    return set(list1) & set(list2)
```

---

### Part B: Medium Level (7 Questions)

#### 24. Find the Missing Number

**Question:** Given a list containing n distinct numbers in the range [0, n], find the one that is missing.

**Solution:**
```python
def missing_number(nums):
    num_set = set(nums)
    for i in range(len(nums) + 1):
        if i not in num_set:
            return i
```

**Developer Scenario:** Checking for missing sequence IDs in a database batch upload.

---

#### 25. First Recurring Character

**Question:** Find the first character in a string that repeats.

**Solution:**
```python
def first_repeat(s):
    seen = set()
    for char in s:
        if char in seen:
            return char
        seen.add(char)
    return None
```

---

#### 26. Difference Between Two Data Streams

**Question:** You have a list of `old_ids` and `new_ids`. Return the IDs that need to be deleted (in old but not new) and added (in new but not old).

**Solution:**
```python
def sync_ids(old_ids, new_ids):
    old, new = set(old_ids), set(new_ids)
    to_delete = old - new
    to_add = new - old
    return to_delete, to_add
```

---

#### 27. Longest Consecutive Sequence

**Question:** Given an unsorted array of integers, find the length of the longest consecutive elements sequence in O(N) time.

**Solution:**
```python
def longest_consecutive(nums):
    num_set = set(nums)
    longest = 0
    for n in num_set:
        # Check if it's the start of a sequence
        if (n - 1) not in num_set:
            length = 1
            while (n + length) in num_set:
                length += 1
            longest = max(longest, length)
    return longest
```

---

#### 28. Set-Based Anagram Check

**Question:** Check if two strings are anagrams using a frequency dictionary (since sets don't handle count). Use `Counter` (which is a dict subclass):

**Solution:**
```python
from collections import Counter
def is_anagram(s1, s2):
    return Counter(s1) == Counter(s2)
```

---

#### 29. Subset Validation

**Question:** You have a list of "Required Permissions" and a list of "User Permissions." Check if the user has ALL required permissions.

**Solution:**
```python
def check_permissions(required, user_perms):
    return set(required).issubset(set(user_perms))
```

**Real App Usage:** Auth0 or AWS IAM policy validation.

---

#### 30. Find Pair with Target Sum

**Question:** Given a list and a target sum, return the two numbers that add up to the target.

**Solution:**
```python
def two_sum(nums, target):
    seen = set()
    for n in nums:
        complement = target - n
        if complement in seen:
            return (complement, n)
        seen.add(n)
    return None
```

---

## Summary

This completes the **Core Collection Series**. You now have 30 deep-dive questions for Lists, Tuples, Dictionaries, and Sets.

