# Python Pandas — Interview Guide (30 Questions)

This guide covers the Pandas library, focusing on data manipulation, performance optimization with vectorized operations, and practical data cleaning scenarios relevant to a Senior Data Engineer or Backend Developer handling data pipelines.

---

## SECTION 1 — THEORY QUESTIONS (20 TOTAL)

### Easy Level (Questions 1–6)

#### 1. Series vs. DataFrame: What is the fundamental architectural difference?

* **Question**: Explain the relationship between a Series and a DataFrame in Pandas.
* **Expected Answer**: A Series is a one-dimensional labeled array (like a column). A DataFrame is a two-dimensional labeled data structure (like a table) composed of aligned Series objects sharing a common index.
* **Clear Explanation**: Think of a DataFrame as a Python dictionary where keys are column names and values are Series. They share a single Index object for row labels.
* **Medium-Depth Reasoning**: Operations on a single column of a DataFrame return a Series. Operations across columns usually return a DataFrame. This alignment allows vectorized operations to propagate correctly based on index labels, not just integer positions.
* **Time & Space Complexity**: Accessing a column is O(1) (dictionary lookup).
* **Common Mistakes**: Treating a single-column DataFrame (e.g., `df[['col']]`) as a Series (`df['col']`). They behave differently in broadcasting.
* **Why Interviewers Ask This**: To verify you understand the basic building blocks before moving to complex aggregations.
* **ASCII Diagram**:
```text
DataFrame:
+-------+-------+
| Col A | Col B |
+-------+-------+
|   1   |   X   |  --> Row Index 0
|   2   |   Y   |  --> Row Index 1
+-------+-------+
   ^       ^
   |       |
Series A  Series B

```


* **Sample Code**:
```python
import pandas as pd
s = pd.Series([1, 2, 3], name="numbers")
df = pd.DataFrame({"numbers": s, "squares": s**2})
print(type(df["numbers"])) # <class 'pandas.core.series.Series'>
print(type(df[["numbers"]])) # <class 'pandas.core.frame.DataFrame'>

```


* **Line-by-line Explanation**:
* Creates a Series `s`.
* Constructs `df` where columns align to `s`.
* Demonstrates that single bracket `[]` extracts a Series, while double brackets `[[]]` extract a DataFrame slice.



#### 2. How does Pandas handle missing data (`NaN` vs `None`)?

* **Question**: In a numeric column, how does Pandas represent missing values, and why?
* **Expected Answer**: It uses `NaN` (Not a Number) from NumPy for float compatibility.
* **Clear Explanation**: Standard Python `None` is an object. Using `None` in a list of integers forces the array to dtype `object` (slow). `NaN` is a special floating-point value (IEEE 754), allowing the array to remain numeric (`float64`) and fast.
* **Reasoning**: Recent Pandas versions introduced `Int64` (nullable int) to allow integers to have `NaN` without casting to float, but the default behavior for years has been "integers with nulls become floats".
* **Comparison**: SQL `NULL` vs Pandas `NaN`.
* **Why Interviewers Ask This**: Understanding type promotion (int -> float) when nulls are introduced is critical for data integrity.
* **Sample Code**:
```python
import numpy as np
import pandas as pd
s = pd.Series([1, 2, np.nan])
print(s.dtype) # float64 (Integers cast to float to accommodate NaN)

```



#### 3. `loc` vs `iloc`: The Slice Trap.

* **Question**: What is the precise difference between `loc` and `iloc` when slicing?
* **Expected Answer**:
* `loc`: Label-based. The slice `start:stop` is **inclusive** of both endpoints.
* `iloc`: Integer-position-based. The slice `start:stop` is **exclusive** of the stop (standard Python style).


* **Common Mistakes**: Using `loc[0:3]` expecting 3 rows but getting 4 (indices 0, 1, 2, 3).
* **Why Interviewers Ask This**: This is the most common bug in data processing scripts.
* **Sample Code**:
```python
df = pd.DataFrame({'a': range(5)}, index=[0, 1, 2, 3, 4])
print(df.loc[0:2])  # Returns rows 0, 1, 2 (3 rows)
print(df.iloc[0:2]) # Returns rows 0, 1    (2 rows)

```



#### 4. How do you check memory usage of a DataFrame?

* **Question**: You loaded a large CSV and RAM is full. How do you check the size?
* **Expected Answer**: `df.info(memory_usage='deep')`.
* **Clear Explanation**:
* `df.memory_usage()` gives memory of indices/columns.
* `deep=True` is required to accurately calculate the memory used by `object` columns (strings), which otherwise only report pointer size.


* **Why Interviewers Ask This**: Performance optimization starts with profiling memory.
* **Sample Code**:
```python
df = pd.DataFrame({'a': ['text'] * 1000})
print(df.info(memory_usage='deep'))

```



#### 5. What is the difference between `read_csv` and `read_parquet`?

* **Question**: Why would you choose Parquet over CSV for intermediate data storage?
* **Expected Answer**:
* **CSV**: Row-oriented, text-based, no type info (types inferred slowly), large disk size.
* **Parquet**: Column-oriented, binary compressed (Snappy/Gzip), stores schema/types, allows "projection pushdown" (reading only specific columns).


* **Performance**: Reading Parquet is often 10x-50x faster than CSV.
* **Why Interviewers Ask This**: Understanding file formats is key for data engineering.

#### 6. Explain the `inplace=True` parameter.

* **Question**: Does `inplace=True` strictly save memory?
* **Expected Answer**: Generally **No**.
* **Reasoning**: In Pandas, `inplace=True` often creates a copy under the hood, reassigns the pointer, and deletes the old object. It rarely modifies the actual memory buffer in-place (due to copy-on-write mechanisms in newer Pandas). It mostly prevents method chaining syntactically.
* **Comparison**: Many core devs recommend avoiding `inplace=True` in modern Pandas to support Method Chaining.
* **Sample Code**:
```python
df = pd.DataFrame({'a': [1, 2]})
df.drop(0, inplace=True) # Modifies df directly (syntactically)

```



---

### Medium Level (Questions 7–20)

#### 7. Vectorization vs. `apply()`: Performance Hierarchy.

* **Question**: You need to create a new column `C` based on `A` and `B`. Rank these methods by speed: `apply`, `iterrows`, Vectorization, List Comprehension.
* **Expected Answer**:
1. **Vectorization** (Fastest): `df['A'] + df['B']`. Uses C-level loops (NumPy).
2. **List Comprehension**: `[x+y for x, y in zip(df['A'], df['B'])]`. Fast Python loop.
3. **`apply()`**: Iterates row-by-row in Python (mostly). Slow overhead.
4. **`iterrows()`** (Slowest): Creates a Series object for every row. Extremely slow.


* **Why Interviewers Ask This**: To weed out developers who treat DataFrames like Lists of Dictionaries.
* **Sample Code**:
```python
import numpy as np
# Vectorized (Good)
df['log_val'] = np.log(df['val'])
# Apply (Bad for simple math)
df['log_val'] = df['val'].apply(lambda x: np.log(x))

```



#### 8. How does `groupby()` actually work? (Split-Apply-Combine).

* **Question**: Explain the internal steps of `df.groupby('category').sum()`.
* **Expected Answer**: It follows the **Split-Apply-Combine** pattern.
1. **Split**: Data is separated into groups based on keys (creates a mapping of key -> indices).
2. **Apply**: The function (`sum`) is applied to each group.
3. **Combine**: Results are merged back into a new DataFrame with the grouping key as the Index.


* **Medium-Depth Reasoning**: `groupby` is lazy. It returns a GroupBy object. Computation only happens when an aggregation function is called.
* **Why Interviewers Ask This**: Essential for understanding aggregation logic.
* **ASCII Diagram**:
```text
[ A, 1 ]       [ A, 1 ]
[ B, 2 ]  -->  [ A, 3 ]  --> Sum() --> [ A: 4 ]
[ A, 3 ]       [ B, 2 ]                [ B: 2 ]

```



#### 9. Merging: `merge` vs `join` vs `concat`.

* **Question**: Distinguish between these three combining methods.
* **Expected Answer**:
* `merge`: Database-style join (inner, outer, left, right) on **columns**.
* `join`: Database-style join, but defaults to joining on **Indexes**.
* `concat`: Stacks DataFrames physically (vertically or horizontally) based on axis, with simple index alignment.


* **Scenario**:
* Adding rows from this month to last month? `concat`.
* Adding user details to a transaction table? `merge`.


* **Sample Code**:
```python
# Merge on column
pd.merge(df1, df2, on='user_id')
# Concat rows
pd.concat([df1, df2], axis=0)

```



#### 10. `SettingWithCopyWarning`: What is it and how to fix it?

* **Question**: You get `SettingWithCopyWarning` doing `df[mask]['col'] = 5`. Why?
* **Expected Answer**: You are doing **Chained Assignment**.
* **Explanation**: `df[mask]` returns a subset. It is ambiguous whether this is a **View** (ref to original) or a **Copy**.
* Pandas doesn't know if you want to modify the original `df` or the temp result of `df[mask]`.


* **Fix**: Use `.loc` to access row/col in one step: `df.loc[mask, 'col'] = 5`.
* **Why Interviewers Ask This**: Shows deep knowledge of Pandas internal memory handling.

#### 11. Handling large datasets with `chunksize`.

* **Question**: A 10GB CSV file cannot fit in RAM. How do you process it to find the sum of a column?
* **Expected Answer**: Use the `chunksize` parameter in `read_csv`.
* **Reasoning**: This returns an iterator (TextFileReader) instead of a DataFrame. You iterate through chunks, process each, and aggregate the result.
* **Sample Code**:
```python
total = 0
for chunk in pd.read_csv('huge.csv', chunksize=1000):
    total += chunk['value'].sum()

```



#### 12. Categorical Data Type Optimization.

* **Question**: You have a column "Status" with 1 million rows, but only 3 unique values ("Active", "Inactive", "Pending"). How do you optimize memory?
* **Expected Answer**: Convert it to `category` dtype.
* **Reasoning**:
* **Object (String)**: Stores strings 1 million times (massive pointer overhead).
* **Category**: Stores the 3 unique strings once in a lookup table. The column itself stores tiny integers pointing to that table.


* **Memory Savings**: Often 90%+ reduction for low-cardinality string columns.
* **Sample Code**:
```python
df['status'] = df['status'].astype('category')

```



#### 13. The MultiIndex (Hierarchical Indexing).

* **Question**: How do you select data from a DataFrame with a MultiIndex `(State, City)`?
* **Expected Answer**: Use `.loc` with a tuple.
* **Explanation**: The index is a tuple.
* Select outer level: `df.loc['California']`
* Select specific row: `df.loc[('California', 'San Francisco')]`
* Slice inner level: `df.xs('San Francisco', level='City')`


* **Why Interviewers Ask This**: Complex datasets (time series, grouped data) usually result in MultiIndexes.

#### 14. `transform()` vs `apply()` in GroupBy.

* **Question**: In a GroupBy context, how does `transform` differ from `apply`?
* **Expected Answer**:
* `transform`: Returns a DataFrame/Series with the **same shape** as the original group. Used for broadcasting aggregate values back to rows (e.g., "percent of group total").
* `apply`: Can return scalar, Series, or DataFrame of **any shape**. More flexible but slower.


* **Sample Code**:
```python
# z-score within group
df['z'] = df.groupby('grp')['val'].transform(lambda x: (x - x.mean()) / x.std())

```



#### 15. Time Series: Resampling.

* **Question**: You have minute-level data. How do you convert it to 5-minute averages?
* **Expected Answer**: `df.resample('5T', on='timestamp').mean()`.
* **Explanation**: `resample` is essentially a `groupby` for time-series data. It handles frequency conversion and missing time gaps (upsampling/downsampling).
* **Comparison**: `groupby(pd.Grouper(key='ts', freq='5T'))` is the generic equivalent.

#### 16. `pivot_table` vs `pivot`.

* **Question**: What happens if you use `pivot` but there are duplicate entries for the index/column pair?
* **Expected Answer**: `pivot` will raise `ValueError`. It strictly reshapes data without aggregation.
* **Solution**: Use `pivot_table`, which accepts an `aggfunc` (default `mean`) to handle duplicates.
* **Visual**:
* `pivot`: Reorders without math.
* `pivot_table`: Reorders AND aggregates (like Excel Pivot Table).



#### 17. The Copy-on-Write (CoW) mechanism (Pandas 2.0+).

* **Question**: How is memory handling changing in modern Pandas (2.0+) regarding views vs copies?
* **Expected Answer**: Pandas is moving towards Lazy Copying (Copy-on-Write).
* **Explanation**: Previously, simple operations might defensively copy data to avoid side effects. With CoW enabled, a copy is only triggered **when you write** to the data, not when you create the derived DataFrame.
* **Why Interviewers Ask This**: Shows you follow the library's roadmap and modern optimization strategies.

#### 18. `query()` method usage.

* **Question**: Why use `df.query('A > B')` instead of `df[df['A'] > df['B']]`?
* **Expected Answer**: Readability and occasionally performance on large datasets (uses `numexpr` under the hood to avoid creating intermediate boolean arrays).
* **Sample Code**:
```python
threshold = 10
# Uses local variable via @
result = df.query('value > @threshold and status == "active"')

```



#### 19. Exploding Lists (`explode`).

* **Question**: You have a column where each cell is a list `[1, 2]`. How do you flatten this so each item gets its own row?
* **Expected Answer**: `df.explode('column_name')`.
* **Behavior**: It duplicates the index for the exploded rows.
* **Data Processing Relevance**: Handling JSON arrays parsed into a DataFrame.

#### 20. `cut` vs `qcut`.

* **Question**: Difference between `pd.cut` and `pd.qcut`?
* **Expected Answer**:
* `cut`: Bins based on **Value** edges (equal width bins). e.g., 0-10, 10-20. (Bins may have different number of items).
* `qcut`: Bins based on **Quantiles** (equal sized bins). e.g., Top 25%, Next 25%. (Bins have roughly same number of items).


* **Usage**: `qcut` is good for "High/Medium/Low" buckets where you want even distribution.

---

## SECTION 2 — CODING QUESTIONS (10 TOTAL)

### Easy Level (Questions 1–3)

#### 1. Filter and Aggregate

**Question**: Given a DataFrame with `['Department', 'Salary']`, find the average salary for the 'IT' department.

* **Solution**:
```python
import pandas as pd

def get_it_avg_salary(df: pd.DataFrame) -> float:
    # Boolean indexing to filter, then mean()
    # Returns scalar float
    return df.loc[df['Department'] == 'IT', 'Salary'].mean()

# Usage
data = {'Department': ['IT', 'HR', 'IT'], 'Salary': [100, 80, 120]}
df = pd.DataFrame(data)
print(get_it_avg_salary(df)) # 110.0

```


* **Explanation**: `df['Department'] == 'IT'` creates a boolean mask. `.loc[mask, 'Salary']` selects only the Salary column for matching rows.
* **Complexity**: O(N).
* **Mistake**: `df[df['Department'] == 'IT']['Salary'].mean()` works but creates an intermediate DataFrame.

#### 2. Filling Missing Values

**Question**: You have a DataFrame with missing values. Fill missing 'Age' with the mean Age, and missing 'City' with "Unknown".

* **Solution**:
```python
def clean_data(df: pd.DataFrame) -> pd.DataFrame:
    # Calculate mean once
    mean_age = df['Age'].mean()

    # Use fillna with a dictionary to target specific columns
    # This returns a new DataFrame (or use inplace=False by default)
    return df.fillna({
        'Age': mean_age,
        'City': 'Unknown'
    })

# Usage
df = pd.DataFrame({'Age': [25, None], 'City': [None, 'NY']})
print(clean_data(df))

```


* **Why Interviewers Ask This**: Standard data cleaning task. Passing a dict to `fillna` is cleaner than calling it twice.

#### 3. Creating a Date Column

**Question**: You have integer columns 'Year', 'Month', 'Day'. Combine them into a single Datetime column.

* **Solution**:
```python
def make_date(df: pd.DataFrame) -> pd.DataFrame:
    # pd.to_datetime can intelligently parse a DataFrame 
    # if cols are named 'year', 'month', 'day'
    df['Date'] = pd.to_datetime(df[['Year', 'Month', 'Day']])
    return df

# Usage
df = pd.DataFrame({'Year': [2023], 'Month': [1], 'Day': [15]})
print(make_date(df))

```


* **Feature**: `to_datetime` has a smart feature where it looks for standard column names to assemble the date.

---

### Medium Level (Questions 4–10)

#### 4. Find Top N per Group

**Question**: Given a DataFrame `['Category', 'Product', 'Sales']`, find the top 2 selling products in each category.

* **Solution**:
```python
def top_products(df: pd.DataFrame, n=2) -> pd.DataFrame:
    # Sort values first to define 'Top'
    # Then group and take head(n)
    return (df.sort_values(['Category', 'Sales'], ascending=[True, False])
              .groupby('Category')
              .head(n))

    # Alternative (slower but more explicit on ranking method):
    # df['rank'] = df.groupby('Category')['Sales'].rank(method='dense', ascending=False)
    # return df[df['rank'] <= n]

# Usage
data = {
    'Category': ['A', 'A', 'A', 'B', 'B'],
    'Product': ['P1', 'P2', 'P3', 'P4', 'P5'],
    'Sales': [100, 200, 50, 300, 100]
}
print(top_products(pd.DataFrame(data)))

```


* **Explanation**: `groupby().head(n)` is an optimized method to return the first `n` rows of each group according to the current sort order.
* **Real App Usage**: E-commerce "Best Sellers in Electronics".

#### 5. Rolling Window Calculation

**Question**: Calculate the 3-day moving average of stock prices. If less than 3 days of data exist, produce NaN.

* **Solution**:
```python
def moving_avg(df: pd.DataFrame) -> pd.DataFrame:
    # Assuming df is indexed by Date or sorted by date
    # min_periods=3 ensures we need 3 valid values
    df['MA_3'] = df['Price'].rolling(window=3, min_periods=3).mean()
    return df

# Usage
df = pd.DataFrame({'Price': [10, 20, 30, 40]})
# Result: [NaN, NaN, 20.0, 30.0]
print(moving_avg(df))

```


* **Scenario**: Financial dashboards.

#### 6. Pivot and Heatmap Prep

**Question**: Transform a log DataFrame `['Date', 'ErrorType', 'Count']` into a matrix where rows are Date, columns are ErrorType, and values are Count. Fill missing with 0.

* **Solution**:
```python
def prepare_heatmap(df: pd.DataFrame) -> pd.DataFrame:
    # pivot_table handles duplicate entries via aggregation (sum)
    # fill_value=0 replaces NaNs introduced by the pivot
    matrix = df.pivot_table(
        index='Date', 
        columns='ErrorType', 
        values='Count', 
        aggfunc='sum', 
        fill_value=0
    )
    return matrix

```


* **Mistake**: Using `pivot` fails if there are multiple logs for same day/type. `pivot_table` is safer.

#### 7. JSON Normalization (Nested Data)

**Question**: You have a column `metadata` containing JSON strings `{"user": {"id": 1, "loc": "NY"}}`. Extract `user.id` and `user.loc` into separate columns.

* **Solution**:
```python
import json

def extract_metadata(df: pd.DataFrame) -> pd.DataFrame:
    # 1. Parse string to dict (if strictly string)
    # If it's already a dict, skip apply(json.loads)
    # df['metadata'] = df['metadata'].apply(json.loads)

    # 2. Use pd.json_normalize
    # This converts a list of dicts into a DataFrame
    expanded = pd.json_normalize(df['metadata'])

    # 3. Join back to original (by index)
    return df.join(expanded)

# Usage
df = pd.DataFrame({'metadata': [{'user': {'id': 1, 'loc': 'NY'}}]})
print(extract_metadata(df))

```


* **Developer Scenario**: Processing API logs or NoSQL data dumps.

#### 8. Safe Division with Zero Handling

**Question**: Calculate `Revenue / Units`. If `Units` is 0, result should be 0 (not inf). Do this without `apply`.

* **Solution**:
```python
import numpy as np

def calc_unit_price(df: pd.DataFrame) -> pd.DataFrame:
    # Vectorized solution using numpy.where
    # condition: df['Units'] > 0
    # true: perform division
    # false: return 0
    df['UnitPrice'] = np.where(
        df['Units'] > 0, 
        df['Revenue'] / df['Units'], 
        0
    )
    return df

# Usage
df = pd.DataFrame({'Revenue': [100, 0], 'Units': [2, 0]})
print(calc_unit_price(df))

```


* **Comparison**: `df['Revenue'] / df['Units']` produces `inf` or `NaN` which breaks downstream ML models.

#### 9. Identifying Streaks (Consecutive Events)

**Question**: Given `['User', 'LoginDate']`, find users who logged in for 3 consecutive days.

* **Solution**:
```python
def find_streaks(df: pd.DataFrame) -> list:
    # 1. Sort
    df = df.sort_values(['User', 'LoginDate'])

    # 2. Calculate diff between current row and previous row
    # GroupBy user to ensure we don't mix users
    df['diff'] = df.groupby('User')['LoginDate'].diff()

    # 3. Identify where diff is 1 day
    df['is_consecutive'] = df['diff'] == pd.Timedelta(days=1)

    # 4. Rolling sum to check for 2 consecutive true flags (implies 3 days)
    # (Day 1 -> Day 2 is True) AND (Day 2 -> Day 3 is True)

    # Advanced Logic: Often easier to shift and compare
    # Or simply check window sums.

    # Simplified: Check if user has streak
    # Let's group and check if any sequence exists
    users_with_streak = []
    for user, grp in df.groupby('User'):
        # Convert boolean streak to int, create islands
        # (Solution simplifies complexity for interview context)
        if any(grp['LoginDate'].diff().dt.days.eq(1).rolling(2).sum().eq(2)):
            users_with_streak.append(user)

    return users_with_streak

```


* **Note**: "Gaps and Islands" problems are hard. The key is `diff()` on dates.

#### 10. Memory Optimization (Downcasting)

**Question**: Write a function that iterates through all columns. If numeric, downcast to the smallest possible type (e.g., `int64` -> `int8`, `float64` -> `float32`) to save RAM.

* **Solution**:
```python
def optimize_memory(df: pd.DataFrame) -> pd.DataFrame:
    for col in df.columns:
        col_type = df[col].dtype

        if col_type != object:
            c_min = df[col].min()
            c_max = df[col].max()

            # Integer handling
            if str(col_type).startswith('int'):
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                # ... handle int32

            # Float handling
            elif str(col_type).startswith('float'):
                if c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)

    return df

```


* **Refinement**: `pd.to_numeric(df[col], downcast='integer')` does this automatically and is safer! An interviewer wants to see if you know `pd.to_numeric`.
```python
# The Pro Move:
def optimize_memory_pro(df):
    for col in df.select_dtypes(include='number'):
        df[col] = pd.to_numeric(df[col], downcast='integer') # or 'float'
    return df

```
