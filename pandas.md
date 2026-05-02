# Pandas Production-Grade Reference Guide
> An exhaustive, production-ready technical manual for the Pandas library.

---

## 1. INTRODUCTION & HISTORY

### What is Pandas
Pandas is an open-source, BSD-licensed library providing high-performance, easy-to-use data structures, and data analysis tools for the Python programming language. It is essentially the "Excel of Python" but with far more power, programmability, and scale. 

### Why Pandas
* **Data Manipulation**: Easily clean, transform, and reshape tabular data.
* **Analysis**: Compute aggregations, statistical properties, and rolling statistics.
* **Flexibility**: Handles varied data types (integers, floats, strings, booleans) and missing data seamlessly.

### Comparison with Excel, SQL, NumPy
| Feature | Pandas | Excel | SQL | NumPy |
| :--- | :--- | :--- | :--- | :--- |
| **Max Data Size** | RAM Limited (typically ~5-10GB comfortably) | ~1M rows | Disk Limited (TB/PB) | RAM Limited |
| **Data Types** | Heterogeneous (different types per column) | Heterogeneous | Heterogeneous | Homogeneous |
| **Syntax/UI** | Python Code | GUI / Formulas | Declarative SQL | Python Code |
| **Speed** | Fast (Vectorized) | Slower for large data | Optimized by engine | Extremely Fast |

### Role in the Data Science Ecosystem
Pandas is the core library for data wrangling. It typically sits after data ingestion (SQL, APIs, flat files) and before machine learning (Scikit-Learn, PyTorch) or visualization (Matplotlib, Seaborn, Plotly).

### Series and DataFrame Concepts
* **Series**: A 1-dimensional labeled array capable of holding any data type.
* **DataFrame**: A 2-dimensional labeled data structure with columns of potentially different types (like a spreadsheet or SQL table).

### Version Evolution
Pandas was created by Wes McKinney in 2008. Over the years:
* `1.0.0` (2020): Introduction of `pd.NA`, StringDtype, and stable API.
* `2.0.0` (2023): Introduction of PyArrow-backed data types for improved performance and memory efficiency.

💡 **Pro Tips:**
* Always know the memory footprint of your data. Pandas can use up to 5x-10x the memory of the file size on disk due to object overhead.

⚠️ **Common Pitfalls:**
* Assuming Pandas is the right tool for "Big Data". If your data exceeds your machine's RAM, consider PySpark, Dask, or Polars.

---

## 2. INSTALLATION & SETUP

### Installation
You can install Pandas using pip or conda.

```bash
# Using pip
pip install pandas

# Install with PyArrow for Pandas 2.0+ performance enhancements
pip install pandas[pyarrow]

# Using conda
conda install pandas
```

### Virtual Environments
Always use virtual environments (e.g., `venv`, `conda`) to avoid dependency conflicts across projects.

```bash
# Create a virtual environment
python -m venv pandas_env

# Activate it (Windows)
pandas_env\Scripts\activate

# Activate it (Unix/MacOS)
source pandas_env/bin/activate
```

### Import Conventions
The community standard is to import Pandas under the alias `pd`.

```python
import pandas as pd
import numpy as np # Usually imported alongside pandas
```

### Checking Version
It's crucial to know your Pandas version as behaviors (especially around nulls and types) change between 1.x and 2.x.

```python
# Check pandas version
print(pd.__version__)

# Show dependencies versions
pd.show_versions()
```

### Setup in Jupyter/IDE
Pandas integrates beautifully with Jupyter Notebooks. You can configure global display settings.

```python
import pandas as pd

# Set maximum columns to display
pd.set_option('display.max_columns', 100)

# Set maximum rows to display
pd.set_option('display.max_rows', 50)

# Format floats to 2 decimal places
pd.set_option('display.float_format', lambda x: f'{x:.2f}')
```

💡 **Pro Tips:**
* Use `pd.reset_option('all')` to revert to default display settings if you mess them up.

⚠️ **Common Pitfalls:**
* Setting `max_rows` to `None` on a massive DataFrame will cause your Jupyter Notebook to hang and potentially crash your browser.

---

## 3. CORE DATA STRUCTURES

### Series
A Series is a 1D array with an index.

```python
import pandas as pd
import numpy as np

# Creating a Series from a list
# The index will automatically be 0, 1, 2, 3
s = pd.Series([10, 20, 30, 40])
print(s)

# Creating a Series from a dictionary (keys become index)
sales = pd.Series({'Jan': 150, 'Feb': 200, 'Mar': 300})
print(sales)

# Accessing the index and values
print(sales.index)  # Index(['Jan', 'Feb', 'Mar'], dtype='object')
print(sales.values) # [150 200 300]
```

### DataFrame
A DataFrame is a 2D table composed of multiple Series (columns) sharing the same index.

```python
# Creating a DataFrame from a dictionary of lists
data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David'],
    'Age': [25, 30, 35, 40],
    'City': ['New York', 'Paris', 'London', 'Tokyo']
}

df = pd.DataFrame(data)
print(df)

# Creating a DataFrame from a list of dictionaries (row-oriented)
rows = [
    {'Name': 'Alice', 'Age': 25},
    {'Name': 'Bob', 'Age': 30}
]
df_rows = pd.DataFrame(rows)
```

### Index Objects
The index is the "address" of your data.

```python
# Creating a custom index
df = pd.DataFrame(data, index=['A1', 'A2', 'A3', 'A4'])

# Resetting an index (moves current index to a column and creates 0-based index)
df_reset = df.reset_index()

# Setting a specific column as the index
df_set = df.set_index('Name')
```

### Data Types (dtypes)
Pandas supports several data types:
* `int64`: Integer numbers
* `float64`: Floating point numbers
* `object`: Text/mixed types (legacy)
* `string`: Text (newer, recommended for text)
* `bool`: True/False
* `category`: Categorical data
* `datetime64[ns]`: Date and time

```python
# Check types
print(df.dtypes)
```

💡 **Pro Tips:**
* Whenever possible, avoid `object` dtypes for strings in Pandas 1.0+. Use `pd.StringDtype()` or just `"string"` for better performance and explicit intent.

⚠️ **Common Pitfalls:**
* A column containing mostly integers but one `NaN` value will automatically be cast to `float64` in legacy Pandas (pre 1.0). In Pandas 1.0+, you can use `Int64` (capital 'I') which supports `pd.NA`.

---

## 4. DATA LOADING & EXPORTING

### Reading Data
Pandas can read from almost any tabular format.

```python
# Read from CSV
df_csv = pd.read_csv('data.csv')

# Read specific columns and set an index immediately
df_csv_opt = pd.read_csv('data.csv', usecols=['id', 'name', 'price'], index_col='id')

# Read from Excel (requires openpyxl)
# pip install openpyxl
df_excel = pd.read_excel('data.xlsx', sheet_name='Sales')

# Read from JSON
df_json = pd.read_json('data.json')

# Read from SQL database (requires SQLAlchemy)
# pip install sqlalchemy
from sqlalchemy import create_engine
engine = create_engine('sqlite:///database.db')
query = "SELECT * FROM users WHERE active = 1"
df_sql = pd.read_sql(query, engine)
```

### Exporting Data

```python
# Export to CSV without the index column
df.to_csv('output.csv', index=False)

# Export to Excel
df.to_excel('output.xlsx', index=False, sheet_name='Summary')

# Export to JSON (orient='records' is usually best for APIs)
df.to_json('output.json', orient='records')
```

### Handling File Paths
Use `pathlib` for robust file path handling across OS.

```python
from pathlib import Path
data_dir = Path('./data')
file_path = data_dir / 'my_dataset.csv'
df = pd.read_csv(file_path)
```

### Encoding Issues
If you encounter `UnicodeDecodeError`, try changing the encoding.

```python
# Common alternative encodings
df = pd.read_csv('data.csv', encoding='latin1')
# or
df = pd.read_csv('data.csv', encoding='utf-16')
```

💡 **Pro Tips:**
* Use `usecols` in `read_csv` if you only need a few columns from a massive file. It saves significant memory.
* If reading dates, use `parse_dates=['date_column']` to parse them on load instead of converting them later.

⚠️ **Common Pitfalls:**
* Forgetting `index=False` in `to_csv()` will write the row numbers as an unnamed first column in your output file, which messes up subsequent reads.

---

## 5. DATA INSPECTION

Before manipulating data, you must understand its shape, types, and quality.

### Quick Looks
```python
import pandas as pd

# Assume df is loaded
df = pd.DataFrame({'A': range(100), 'B': range(100, 200)})

# View first 5 rows (default)
print(df.head())

# View last 3 rows
print(df.tail(3))

# Random sample of 5 rows
print(df.sample(5))
```

### Metadata and Structure
```python
# Returns a tuple: (rows, columns)
print(df.shape)

# View column names
print(df.columns)
# Convert to list if needed: df.columns.tolist()

# View index details
print(df.index)

# Concise summary of DataFrame (dtypes, non-null counts, memory usage)
df.info()

# Memory usage deep inspection (including object types)
df.info(memory_usage='deep')
```

### Statistical Summary
```python
# Summary stats for numeric columns (count, mean, std, min, max, quartiles)
print(df.describe())

# Summary stats for categorical/string columns
df_cat = pd.DataFrame({'color': ['red', 'blue', 'red', 'green']})
print(df_cat.describe(include=['object']))
```

### Unique Values & Counts
```python
s = pd.Series(['apple', 'banana', 'apple', 'orange', 'banana', 'apple'])

# Count frequency of each unique value
print(s.value_counts())
# Normalize to get percentages
print(s.value_counts(normalize=True))

# Get array of unique values
print(s.unique())

# Get number of unique values
print(s.nunique())
```

💡 **Pro Tips:**
* `df.info()` is the very first function you should run after reading a file to spot nulls and incorrect dtypes.

⚠️ **Common Pitfalls:**
* `describe()` silently ignores non-numeric columns by default. Use `describe(include='all')` to see everything.

---

## 6. INDEXING & SELECTION

Pandas provides several ways to slice and dice data.

### Selecting Columns
```python
df = pd.DataFrame({'Name': ['Ed', 'Al'], 'Age': [20, 22]})

# Select a single column (returns a Series)
ages = df['Age']

# Select multiple columns (returns a DataFrame, note the double brackets)
subset = df[['Name', 'Age']]
```

### `loc[]` (Label-based)
Used for selecting data by the label of the index and columns.

```python
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]}, index=['x', 'y', 'z'])

# Select row by index label
print(df.loc['y'])

# Select multiple rows
print(df.loc[['x', 'z']])

# Select row and specific columns
print(df.loc['y', 'A'])
print(df.loc['y', ['A', 'B']])

# Slice by labels (NOTE: loc slicing is INCLUSIVE of both endpoints!)
print(df.loc['x':'y', 'A':'B'])
```

### `iloc[]` (Integer-position based)
Used for selecting data by the numerical position (0-based).

```python
# Select first row
print(df.iloc[0])

# Select last row
print(df.iloc[-1])

# Slice rows and columns (NOTE: iloc slicing is EXCLUSIVE of the endpoint, like standard Python)
print(df.iloc[0:2, 0:1]) # Rows 0,1 and Column 0
```

### Boolean Indexing (Filtering)
```python
data = {'Name': ['A', 'B', 'C', 'D'], 'Age': [25, 30, 35, 40], 'City': ['NY', 'LA', 'NY', 'SF']}
df = pd.DataFrame(data)

# Simple condition
is_over_30 = df['Age'] > 30
print(df[is_over_30])
# Alternatively: df[df['Age'] > 30]

# Multiple conditions
# MUST use parenthesis around each condition!
# Use `&` for AND, `|` for OR, `~` for NOT
filtered = df[(df['Age'] > 25) & (df['City'] == 'NY')]
print(filtered)

# isin() method
cities_of_interest = ['NY', 'SF']
print(df[df['City'].isin(cities_of_interest)])
```

### Setting Values
```python
# Set value for a specific row and column
df.loc[df['Name'] == 'A', 'Age'] = 26

# Create new column based on condition using np.where
import numpy as np
df['Status'] = np.where(df['Age'] >= 30, 'Senior', 'Junior')
```

💡 **Pro Tips:**
* Avoid chained indexing like `df['Age'][0] = 99`. This causes the infamous `SettingWithCopyWarning`. Always use `df.loc[0, 'Age'] = 99`.

⚠️ **Common Pitfalls:**
* Forgetting parentheses in multi-condition boolean indexing: `df[df['A'] > 5 & df['B'] < 10]` will raise an error due to operator precedence.

---

## 7. DATA CLEANING

Data is rarely clean. Pandas excels at dealing with messy data.

### Handling Missing Values
```python
import numpy as np
df = pd.DataFrame({'A': [1, 2, np.nan, 4], 'B': [np.nan, 2, 3, 4], 'C': ['x', 'y', 'z', None]})

# Check for nulls (returns boolean mask)
print(df.isna())
# Total nulls per column
print(df.isna().sum())

# Drop any row containing at least one null
df_clean = df.dropna()

# Drop rows where specifically column 'A' is null
df_drop_A = df.dropna(subset=['A'])

# Fill missing values with a specific constant
df_filled = df.fillna(0)
df_filled_str = df.fillna('Unknown')

# Fill missing values with column mean
df['A'] = df['A'].fillna(df['A'].mean())

# Forward fill (propagate last valid observation forward)
df_ffill = df.ffill()
```

### Removing Duplicates
```python
df_dup = pd.DataFrame({'A': [1, 1, 2, 3], 'B': ['x', 'x', 'y', 'z']})

# Find duplicates (returns boolean mask)
print(df_dup.duplicated())

# Drop identical rows, keeping the first occurrence
df_nodup = df_dup.drop_duplicates()

# Drop rows based on duplicate values in a specific column, keeping the last occurrence
df_nodup_col = df_dup.drop_duplicates(subset=['A'], keep='last')
```

### Data Type Conversion
```python
df = pd.DataFrame({'price': ['10.5', '20.1', '30.9'], 'qty': ['1', '2', '3']})

# Convert types using astype
df['price'] = df['price'].astype(float)
df['qty'] = df['qty'].astype(int)

# Use to_numeric for safer conversion (handles errors)
# errors='coerce' forces invalid parsing to NaN
df['price'] = pd.to_numeric(df['price'], errors='coerce')
```

### String Operations
```python
df = pd.DataFrame({'name': [' ALICE ', 'bob', 'Charlie_Chaplin']})

# All string operations are accessed via the .str accessor
# Strip whitespace
df['name'] = df['name'].str.strip()

# Convert to uppercase
df['name'] = df['name'].str.upper()

# Split string and expand into multiple columns
df[['first', 'last']] = df['name'].str.split('_', expand=True)

# Replace substrings
df['name'] = df['name'].str.replace('bob', 'Robert')
```

### Renaming Columns
```python
# Rename using a dictionary
df = df.rename(columns={'name': 'Full_Name', 'qty': 'Quantity'})

# Lowercase all column names and replace spaces with underscores
df.columns = [col.lower().replace(' ', '_') for col in df.columns]
```

💡 **Pro Tips:**
* `errors='coerce'` in `to_numeric` or `to_datetime` is extremely useful for data containing unparseable garbage values that you just want to turn into `NaN`.

⚠️ **Common Pitfalls:**
* Running `df.dropna()` without checking might delete 90% of your data if one minor column has many nulls. Always check `df.isna().sum()` first.

---

## 8. DATA MANIPULATION

### Adding and Removing Columns
```python
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

# Add a calculated column
df['C'] = df['A'] + df['B']

# Add a scalar column
df['Status'] = 'Active'

# Insert a column at a specific position (index, name, values)
df.insert(1, 'A_squared', df['A'] ** 2)

# Drop a column (axis=1 means columns, axis=0 means rows)
df = df.drop('Status', axis=1)
# Alternatively: df.drop(columns=['Status'])
```

### `apply()`, `map()`, and Vectorization
```python
# 1. Vectorization (FASTEST - Always prefer this)
df['A_double'] = df['A'] * 2

# 2. apply() (SLOWER - Applies a function along an axis of DataFrame)
# Apply a custom function to a Series
def complex_math(x):
    if x > 2:
        return x * 10
    return x / 2

df['A_complex'] = df['A'].apply(complex_math)

# Apply a function across multiple columns (row-wise operation)
# axis=1 passes a Series representing the row to the function
df['A_B_sum'] = df.apply(lambda row: row['A'] + row['B'], axis=1)

# 3. map() (Used for substituting values in a Series)
mapping = {1: 'One', 2: 'Two', 3: 'Three'}
df['A_str'] = df['A'].map(mapping)
```

### Sorting
```python
# Sort by a single column
df_sorted = df.sort_values(by='B', ascending=False)

# Sort by multiple columns
df_sorted_multi = df.sort_values(by=['A', 'B'], ascending=[True, False])

# Sort by index
df_idx_sorted = df.sort_index(ascending=False)
```

### Replacing Values
```python
# Replace specific values across the whole DataFrame or Series
df['B'] = df['B'].replace(4, 400)
df = df.replace({'B': {5: 500, 6: 600}})
```

💡 **Pro Tips:**
* Avoid `apply(axis=1)` on large datasets. It essentially runs a Python `for` loop under the hood and is very slow. Try to use vectorized operations or NumPy `np.where`/`np.select` instead.

⚠️ **Common Pitfalls:**
* `inplace=True` is often misunderstood. It does NOT generally speed up operations or save memory (it often copies anyway). The Pandas core team recommends standard assignment: `df = df.drop(...)` instead of `df.drop(..., inplace=True)`.

---

## 9. GROUPBY OPERATIONS

The `groupby` mechanism follows the **Split-Apply-Combine** paradigm.

### Basics
```python
data = {
    'Department': ['IT', 'HR', 'IT', 'Finance', 'HR', 'IT'],
    'Employee': ['Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'],
    'Salary': [80000, 60000, 85000, 90000, 65000, 95000],
    'Experience': [5, 2, 6, 8, 3, 10]
}
df = pd.DataFrame(data)

# Group by Department
grouped = df.groupby('Department')

# Iterate through groups
for name, group in grouped:
    print(f"Group: {name}")
    print(group)
```

### Aggregation
```python
# Single aggregation on a single column
mean_salaries = df.groupby('Department')['Salary'].mean()

# Multiple aggregations on a single column
agg_salaries = df.groupby('Department')['Salary'].agg(['mean', 'sum', 'count'])

# Different aggregations for different columns
complex_agg = df.groupby('Department').agg({
    'Salary': ['mean', 'max'],
    'Experience': 'sum',
    'Employee': 'count' # Count number of employees
})

# Named aggregation (cleaner column names in output)
named_agg = df.groupby('Department').agg(
    Avg_Salary=('Salary', 'mean'),
    Total_Exp=('Experience', 'sum')
)
```

### `transform()`
`transform` returns an object that is the same shape as the input. Useful for adding group-level statistics back to the original rows.

```python
# Calculate the average department salary and assign it to every employee
df['Dept_Avg_Salary'] = df.groupby('Department')['Salary'].transform('mean')

# Calculate how far each employee is from their department average
df['Salary_Diff_From_Mean'] = df['Salary'] - df['Dept_Avg_Salary']
```

### Filtering Groups
```python
# Keep only employees in departments that have more than 2 people
df_large_depts = df.groupby('Department').filter(lambda x: len(x) > 2)
```

💡 **Pro Tips:**
* `as_index=False` is often helpful: `df.groupby('Dept', as_index=False)['Salary'].mean()`. It prevents the grouping column from becoming the index, saving you a `reset_index()` step.

⚠️ **Common Pitfalls:**
* Calling `df.groupby('A').sum()` without selecting a column will try to sum *all* numeric columns. If you have non-numeric columns, this might raise a `TypeError` in recent Pandas versions. Always select your columns: `df.groupby('A')['B'].sum()`.

---

## 10. MERGING & JOINING

Pandas provides high-performance, in-memory join operations akin to SQL.

### `merge()`
`merge` is the most versatile function for joining DataFrames on columns or indexes.

```python
users = pd.DataFrame({
    'user_id': [1, 2, 3, 4],
    'name': ['Alice', 'Bob', 'Charlie', 'David']
})

orders = pd.DataFrame({
    'order_id': [101, 102, 103],
    'user_id': [1, 2, 1],
    'amount': [250, 150, 300]
})

# Inner Join (Default: keeps only matches in both)
inner_join = pd.merge(users, orders, on='user_id', how='inner')

# Left Join (Keeps all from 'users', fills missing orders with NaN)
left_join = pd.merge(users, orders, on='user_id', how='left')

# Joining on different column names
orders_diff_name = orders.rename(columns={'user_id': 'customer_id'})
diff_col_join = pd.merge(users, orders_diff_name, left_on='user_id', right_on='customer_id', how='inner')
# Drop the redundant column after merging
diff_col_join = diff_col_join.drop('customer_id', axis=1)

# Adding suffixes for overlapping column names
users2 = pd.DataFrame({'id': [1, 2], 'status': ['A', 'B']})
orders2 = pd.DataFrame({'id': [1, 2], 'status': ['Pending', 'Completed']})
merged_suffix = pd.merge(users2, orders2, on='id', suffixes=('_user', '_order'))
```

### `join()`
`join` is primarily used for joining DataFrames on their indexes.

```python
df1 = pd.DataFrame({'A': [1, 2]}, index=['x', 'y'])
df2 = pd.DataFrame({'B': [3, 4]}, index=['x', 'y'])

# Joins on index
joined_df = df1.join(df2)
```

### `concat()`
`concat` is used to append DataFrames either vertically (adding rows) or horizontally (adding columns).

```python
df_a = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df_b = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})

# Vertically append (stack rows)
# ignore_index=True reassigns index 0,1,2,3... instead of keeping original 0,1,0,1
df_v = pd.concat([df_a, df_b], axis=0, ignore_index=True)

# Horizontally append (side-by-side columns)
df_h = pd.concat([df_a, df_b], axis=1)
```

💡 **Pro Tips:**
* Use `indicator=True` in `pd.merge()` to add a `_merge` column that tells you if a row came from `left_only`, `right_only`, or `both`. This is incredibly useful for debugging joins.

⚠️ **Common Pitfalls:**
* **Exploding Joins**: If both DataFrames have duplicate keys in the join columns, Pandas performs a Cartesian product (many-to-many join) for those keys, resulting in massive memory explosions. Use `validate='1:m'` or `validate='m:1'` to strictly enforce join expectations and catch duplicate keys.

---

## 11. RESHAPING DATA

Transforming data between "wide" and "long" formats.

### `pivot()`
Transforms long format to wide format. Does not support aggregation (must have unique index/columns combinations).

```python
df_long = pd.DataFrame({
    'date': ['2023-01-01', '2023-01-01', '2023-01-02', '2023-01-02'],
    'city': ['NY', 'LA', 'NY', 'LA'],
    'temp': [32, 75, 30, 77]
})

# Make dates the index, cities the columns, and temp the values
df_wide = df_long.pivot(index='date', columns='city', values='temp')
print(df_wide)
```

### `pivot_table()`
Like `pivot`, but handles duplicate entries by applying an aggregation function (default is mean).

```python
df_dups = pd.DataFrame({
    'date': ['2023-01-01', '2023-01-01', '2023-01-01'], # Note two NY entries on same day
    'city': ['NY', 'LA', 'NY'],
    'temp': [32, 75, 34]
})

# Averages the two NY temps on 2023-01-01
df_pt = df_dups.pivot_table(index='date', columns='city', values='temp', aggfunc='mean')

# Adding margins (totals)
df_pt_margins = df_dups.pivot_table(index='date', columns='city', values='temp', aggfunc='sum', margins=True)
```

### `melt()`
Transforms wide format to long format (the inverse of pivot). Excellent for un-pivoting datasets for visualization.

```python
# Revert df_wide back to long format
df_wide_reset = df_wide.reset_index()
df_melted = pd.melt(
    df_wide_reset,
    id_vars=['date'],         # Columns to keep as identifier variables
    value_vars=['LA', 'NY'],  # Columns to unpivot
    var_name='city',          # Name of new 'variable' column
    value_name='temperature'  # Name of new 'value' column
)
```

### `stack()` and `unstack()`
Used for pivoting MultiIndex DataFrames.
* `stack()`: "Compresses" a level in the DataFrame's columns to the index.
* `unstack()`: "Expands" a level in the index to the columns.

```python
# Given a multi-indexed dataframe df_multi
# df_stacked = df_multi.stack()
# df_unstacked = df_stacked.unstack()
```

💡 **Pro Tips:**
* Seaborn and Plotly Express often expect data in a "long" format (melted). Use `melt` extensively before visualization.

⚠️ **Common Pitfalls:**
* `pivot` will fail with a `ValueError: Index contains duplicate entries` if your index/column combination isn't unique. Switch to `pivot_table` if you encounter this.

---

## 12. TIME SERIES

Pandas was originally developed for financial time series data and possesses incredible date/time functionality.

### Converting to Datetime
```python
df = pd.DataFrame({
    'date_str': ['2023-01-01', '2023/02/01', 'March 1, 2023'],
    'value': [10, 20, 30]
})

# Convert string to datetime objects
# Pandas is smart at inferring formats, but explicitly passing `format` is much faster
df['date'] = pd.to_datetime(df['date_str'])

# Extracting components via the .dt accessor
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_name'] = df['date'].dt.day_name()
```

### DateTimeIndex
Setting a datetime column as the index unlocks powerful time-based slicing and resampling.

```python
# Create a date range
dates = pd.date_range(start='2023-01-01', periods=100, freq='D')
ts = pd.DataFrame({'sales': np.random.randn(100)}, index=dates)

# Slicing by string
print(ts['2023-01']) # All January 2023
print(ts['2023-02-15':'2023-03-15']) # Range slicing
```

### Resampling
Changing the frequency of your time series data. Think of it as a time-based `groupby`.

```python
# Downsampling: Aggregate daily data to monthly ('M') summing the values
monthly_sales = ts.resample('M').sum()

# Aggregate to weekly ('W') taking the mean
weekly_avg = ts.resample('W').mean()

# Upsampling: Convert monthly data to daily data
# Requires filling logic because we are introducing new rows
daily_from_month = monthly_sales.resample('D').ffill() # Forward fill
```

### Shifting and Lagging
Moving data backward or forward in time. Useful for calculating period-over-period differences.

```python
# Shift data down by 1 period (creates a lag of 1)
ts['prev_day_sales'] = ts['sales'].shift(1)

# Calculate day-over-day difference
ts['daily_diff'] = ts['sales'] - ts['sales'].shift(1)
# Alternatively: ts['sales'].diff()

# Calculate day-over-day percentage return
ts['pct_change'] = ts['sales'].pct_change()
```

💡 **Pro Tips:**
* Timezones: Use `dt.tz_localize('UTC')` to set a timezone on naive dates, and `dt.tz_convert('America/New_York')` to change timezones.

⚠️ **Common Pitfalls:**
* Relying on `pd.to_datetime` without a format string on massive datasets. It forces Pandas to guess the format row-by-row, which is extremely slow. Use `format='%Y-%m-%d'` when possible.

---

## 13. WINDOW FUNCTIONS

Window functions perform calculations across a sliding or expanding window of data.

### Rolling Windows
Calculates a statistic over a fixed-size window moving through the data.

```python
# Setup mock stock prices
dates = pd.date_range('2023-01-01', periods=50)
df_stock = pd.DataFrame({'price': np.cumsum(np.random.randn(50)) + 100}, index=dates)

# 7-day Simple Moving Average (SMA)
df_stock['7_day_ma'] = df_stock['price'].rolling(window=7).mean()

# 14-day rolling standard deviation (volatility)
df_stock['14_day_std'] = df_stock['price'].rolling(window=14).std()

# min_periods allows calculation even if window size isn't met (e.g., first few rows)
df_stock['7_day_ma_fluid'] = df_stock['price'].rolling(window=7, min_periods=1).mean()
```

### Expanding Windows
Calculates a statistic from the beginning of the data up to the current point.

```python
# Cumulative sum (alternative to cumsum())
df_stock['cumulative_sum'] = df_stock['price'].expanding().sum()

# Expanding max (running maximum)
df_stock['running_max'] = df_stock['price'].expanding().max()

# Drawdown calculation
df_stock['drawdown'] = df_stock['price'] / df_stock['running_max'] - 1
```

### Exponentially Weighted Windows (EWM)
Gives more weight to recent observations. Very common in finance.

```python
# Exponential moving average (EMA)
# span=10 means an N-day EMA
df_stock['10_day_ema'] = df_stock['price'].ewm(span=10, adjust=False).mean()
```

💡 **Pro Tips:**
* You can apply custom functions to rolling windows using `rolling(n).apply(custom_func)`, but it is generally slow. If possible, stick to optimized built-in methods like `mean, std, sum, min, max`.

⚠️ **Common Pitfalls:**
* Rolling operations introduce `NaN` values at the beginning of the dataset equal to `window_size - 1`. Don't forget to handle these if feeding data into an ML model.

---

## 14. STATISTICAL ANALYSIS

Pandas has a robust suite of built-in statistical methods.

### Descriptive Statistics
```python
df = pd.DataFrame({'A': np.random.rand(100), 'B': np.random.rand(100) * 10})

print("Mean:", df['A'].mean())
print("Median:", df['A'].median())
print("Variance:", df['A'].var())
print("Standard Deviation:", df['A'].std())
print("Skewness:", df['A'].skew())
print("Kurtosis:", df['A'].kurt())
```

### Correlation and Covariance
```python
# Pearson correlation matrix (linear correlation)
print(df.corr())

# Spearman correlation matrix (rank-based, robust to outliers)
print(df.corr(method='spearman'))

# Covariance matrix
print(df.cov())
```

### Ranking
```python
sales = pd.Series([100, 200, 200, 300])

# Default ranking (ties average the ranks: 1, 2.5, 2.5, 4)
print(sales.rank())

# Dense ranking (ties get same rank, next is n+1: 1, 2, 2, 3)
print(sales.rank(method='dense'))

# Min ranking (ties get lowest rank: 1, 2, 2, 4)
print(sales.rank(method='min'))
```

### Quantiles & Binning
```python
# Get specific quantiles
print(df['A'].quantile([0.25, 0.5, 0.75]))

# Bin continuous data into discrete intervals (equal width)
df['A_bins'] = pd.cut(df['A'], bins=4, labels=['Low', 'Med-Low', 'Med-High', 'High'])

# Bin data into equal-sized quantiles (e.g., quartiles)
df['A_quartiles'] = pd.qcut(df['A'], q=4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
```

💡 **Pro Tips:**
* `pd.cut` defines bins based on *value* ranges, whereas `pd.qcut` defines bins so that each bin has roughly the same *number of observations*.

⚠️ **Common Pitfalls:**
* Calculating correlation on non-stationary time series data can lead to "spurious correlation". Always difference or detrend time series data before running `.corr()`.

---

## 15. CATEGORICAL DATA

Using the `category` dtype instead of `object` (string) can save massive amounts of memory and speed up operations.

### Why Categoricals?
If a column has a limited number of unique values (e.g., 'Gender', 'Country', 'Status'), storing them as strings is inefficient. Categorical data stores the unique strings once and uses an integer array internally to reference them.

### Converting and Memory Savings
```python
# Generate 1 million rows of string data
df_large = pd.DataFrame({
    'status': np.random.choice(['Pending', 'Approved', 'Rejected'], size=1_000_000)
})

# Check memory (Object dtype)
print(df_large['status'].memory_usage(deep=True) / 1024**2, "MB") # ~60 MB

# Convert to category
df_large['status'] = df_large['status'].astype('category')

# Check memory again (Category dtype)
print(df_large['status'].memory_usage(deep=True) / 1024**2, "MB") # ~1 MB (Massive savings!)
```

### Working with Categories
```python
# Accessing the .cat accessor
print(df_large['status'].cat.categories)

# Add a new category (required before you can assign a new value)
df_large['status'] = df_large['status'].cat.add_categories(['Under Review'])

# Ordered Categories (useful for sorting: Low < Medium < High)
sizes = pd.Series(['Medium', 'Large', 'Small', 'Medium'])
size_type = pd.CategoricalDtype(categories=['Small', 'Medium', 'Large'], ordered=True)
sizes = sizes.astype(size_type)

# Now sorting works logically, not alphabetically
print(sizes.sort_values())
```

💡 **Pro Tips:**
* Groupby operations on categorical columns are significantly faster.
* Always use `category` dtype for low-cardinality string columns in production pipelines.

⚠️ **Common Pitfalls:**
* You cannot simply assign a new string value to a categorical column if that string is not in the predefined categories. You will get a `ValueError`. You must use `.cat.add_categories()` first.

---

## 16. PERFORMANCE OPTIMIZATION

Pandas can be slow if used like standard Python. You must think in vectors.

### 1. Vectorization (Do This First)
Vectorized operations push the loop into optimized C code.

```python
df = pd.DataFrame({'A': np.random.rand(100000), 'B': np.random.rand(100000)})

# BAD (Python Loop via iterrows)
# Takes seconds
for index, row in df.iterrows():
    df.loc[index, 'C'] = row['A'] + row['B']

# OK (apply with lambda)
# Takes milliseconds
df['C'] = df.apply(lambda row: row['A'] + row['B'], axis=1)

# BEST (Vectorization)
# Takes microseconds
df['C'] = df['A'] + df['B']
```

### 2. Using NumPy `where` and `select`
For conditional logic, avoid `apply`. Use NumPy.

```python
import numpy as np

# Single condition (if/else)
df['Status'] = np.where(df['A'] > 0.5, 'High', 'Low')

# Multiple conditions
conditions = [
    (df['A'] > 0.8),
    (df['A'] > 0.5) & (df['A'] <= 0.8),
    (df['A'] <= 0.5)
]
choices = ['High', 'Medium', 'Low']
df['Category'] = np.select(conditions, choices, default='Unknown')
```

### 3. Evaluating Strings (`pd.eval`)
For complex mathematical operations on very large DataFrames, `pd.eval` uses NumExpr to optimize execution.

```python
df = pd.DataFrame(np.random.randn(100000, 3), columns=['A', 'B', 'C'])

# Standard
result1 = df['A'] + df['B'] * df['C']

# Optimized via string evaluation (faster for large datasets)
result2 = pd.eval('df.A + df.B * df.C')
```

### 4. Efficient Types
* Downcast numeric types. An integer only needing 0-100 doesn't need `int64`; use `int8`.

```python
# Downcast floats
df['A'] = pd.to_numeric(df['A'], downcast='float')

# Downcast integers
df['B'] = pd.to_numeric(df['B'], downcast='integer')
```

💡 **Pro Tips:**
* NEVER use `iterrows()` or `itertuples()` unless absolutely necessary. There is almost always a vectorized alternative.
* For Pandas 2.0+, use the `pyarrow` engine backend: `df = pd.read_csv('data.csv', engine='pyarrow', dtype_backend='pyarrow')`. It is significantly faster and uses less memory.

⚠️ **Common Pitfalls:**
* Premature optimization. Vectorization is critical, but don't spend hours trying to vectorize a 100-row dataframe. Optimize where memory/speed actually matters.

---

## 17. FILE HANDLING & BIG DATA

When data exceeds RAM, you need different strategies.

### Chunking Large Files
Read a huge CSV in small chunks, process each chunk, and aggregate the results.

```python
# Initialize empty list to store aggregated results
chunk_results = []

# read_csv returns an iterator when chunksize is specified
chunk_iter = pd.read_csv('massive_data.csv', chunksize=100_000)

for chunk in chunk_iter:
    # Process the chunk (e.g., filter, aggregate)
    filtered = chunk[chunk['age'] > 30]
    agg = filtered.groupby('city')['salary'].sum()
    
    chunk_results.append(agg)

# Combine results from all chunks
final_result = pd.concat(chunk_results).groupby('city').sum()
```

### Better File Formats (Parquet / Feather)
Stop using CSVs for intermediate data storage. They are slow, large, and lose dtype information. Use Parquet.

```python
# Requires pyarrow or fastparquet: pip install pyarrow fastparquet

# Write to Parquet (Compression, fast read/write, preserves dtypes)
df.to_parquet('data.parquet', engine='pyarrow')

# Read from Parquet
df_pq = pd.read_parquet('data.parquet')

# Read specific columns from Parquet (Parquet is columnar, so this is extremely fast)
df_pq_subset = pd.read_parquet('data.parquet', columns=['city', 'salary'])
```

💡 **Pro Tips:**
* Parquet files are typically 10-20% the size of an equivalent CSV and read 10x-50x faster.

⚠️ **Common Pitfalls:**
* `to_csv` on a 10GB DataFrame might cause your OS to run out of memory during string conversion. Use `to_parquet`.

---

## 18. VISUALIZATION

Pandas has a built-in wrapper around Matplotlib for quick and easy plotting.

### Basic Plotting
```python
# Ensure matplotlib is installed: pip install matplotlib
import matplotlib.pyplot as plt

df_plot = pd.DataFrame(np.random.randn(100, 4), index=pd.date_range('1/1/2000', periods=100), columns=list('ABCD'))
df_plot = df_plot.cumsum()

# Line plot (default)
df_plot.plot()
plt.title('Random Walk')
plt.show()

# Other plot types
df_plot.plot(kind='bar')       # Bar chart
df_plot.plot(kind='hist')      # Histogram
df_plot.plot(kind='box')       # Box plot
df_plot.plot(kind='scatter', x='A', y='B') # Scatter plot

# Pandas 1.0+ introduced alternate plotting backends
# pip install plotly
# pd.options.plotting.backend = "plotly"
# df_plot.plot() # Now generates an interactive Plotly graph!
```

💡 **Pro Tips:**
* Pandas plotting is great for quick EDA. For presentation-quality graphics, pass your DataFrame to Seaborn: `import seaborn as sns; sns.boxplot(data=df, x='category', y='value')`.

---

## 19. INTEROPERABILITY

Pandas plays nicely with the entire Python ecosystem.

### Pandas and NumPy
```python
# DataFrame to NumPy array
numpy_matrix = df.to_numpy()
# Alternatively (older): df.values

# Series to NumPy array
numpy_array = df['A'].to_numpy()
```

### Pandas and Dictionaries/JSON
```python
# Convert DataFrame to a list of dictionaries (standard for APIs)
records = df.to_dict(orient='records')

# Convert DataFrame to a dictionary of series
col_dict = df.to_dict(orient='series')
```

### Pandas and PySpark
If your data outgrows Pandas, PySpark provides a pandas API.

```python
# If you are in a PySpark environment:
# import pyspark.pandas as ps
# psdf = ps.from_pandas(df) # Scalable pandas API on Spark!
```

---

## 20. ADVANCED TOPICS

### MultiIndex (Hierarchical Indexing)
MultiIndex allows you to store higher-dimensional data in 2D DataFrames.

```python
# Creating a MultiIndex from arrays
arrays = [
    ['Sales', 'Sales', 'HR', 'HR'],
    ['Q1', 'Q2', 'Q1', 'Q2']
]
index = pd.MultiIndex.from_arrays(arrays, names=('Dept', 'Quarter'))
df_mi = pd.DataFrame({'Revenue': [100, 150, 50, 60]}, index=index)

print(df_mi)
# Accessing data in MultiIndex
print(df_mi.loc['Sales'])          # All sales data
print(df_mi.loc[('Sales', 'Q1')])  # Specific tuple selection
```

### Method Chaining
Write clean, pipeline-style data transformations instead of creating multiple intermediate variables.

```python
# BAD (messy, lots of intermediate variables)
df1 = pd.read_csv('data.csv')
df2 = df1.dropna()
df3 = df2[df2['sales'] > 100]
df4 = df3.groupby('region')['sales'].sum()

# GOOD (Method Chaining)
# Wrap in parenthesis to allow multi-line chaining without \
clean_sales = (
    pd.read_csv('data.csv')
    .dropna(subset=['region', 'sales'])
    .query('sales > 100') # .query() evaluates a string expression for boolean indexing
    .assign(tax=lambda x: x['sales'] * 0.1) # .assign() creates new columns inline
    .groupby('region')
    ['sales'].sum()
    .reset_index()
)
```

### `.pipe()` Method
Allows you to pass the DataFrame through custom functions within a method chain.

```python
def remove_outliers(df_in, column, z_thresh=3):
    z_scores = np.abs((df_in[column] - df_in[column].mean()) / df_in[column].std())
    return df_in[z_scores < z_thresh]

def convert_currency(df_in, col, rate):
    df_out = df_in.copy()
    df_out[col] = df_out[col] * rate
    return df_out

# Use pipe to apply custom functions in a chain
final_df = (
    pd.read_csv('data.csv')
    .pipe(remove_outliers, column='sales')
    .pipe(convert_currency, col='sales', rate=1.2)
)
```

### Styling DataFrames (for Jupyter)
You can apply conditional formatting to DataFrames rendered in Jupyter.

```python
# Highlights the maximum value in each column yellow
# df.style.highlight_max(color='yellow')

# Apply a background gradient to numeric columns
# df.style.background_gradient(cmap='viridis')
```

### Internals Overview: How Pandas Stores Data
Historically, Pandas stored data in 2D NumPy arrays called "Blocks". A DataFrame was managed by a `BlockManager` that grouped columns of the same dtype together (e.g., all `int64` columns in one 2D array, all `float64` in another).
* **Cons**: Operations that changed types or added columns caused massive memory copies and fragmentation.
* **Modern Pandas (2.0+)**: You can now opt-in to the PyArrow backend (`dtype_backend="pyarrow"`). PyArrow uses columnar memory formats, meaning strings are stored continuously and nulls are tracked efficiently via bitmasks, circumventing the legacy `BlockManager` issues and drastically reducing memory footprints.

---
*End of Document. Refer to the official [Pandas Documentation](https://pandas.pydata.org/docs/) for exhaustive API reference.*
