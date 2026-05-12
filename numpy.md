# The Definitive NumPy Reference Guide: From Fundamentals to Advanced Internals

## Contents
- [1. Introduction & History](#1-introduction--history)
- [2. Installation & Setup](#2-installation--setup)
- [3. Array Creation](#3-array-creation)
- [4. Array Attributes](#4-array-attributes)
- [5. Indexing and Slicing](#5-indexing-and-slicing)
- [6. Broadcasting](#6-broadcasting)
- [7. Universal Functions](#7-universal-functions)
- [8. Array Operations](#8-array-operations)
- [9. Aggregations](#9-aggregations)
- [10. Linear Algebra](#10-linear-algebra)
- [11. Random Numbers](#11-random-numbers)
- [12. Data Types](#12-data-types)
- [13. Reshaping and Stacking](#13-reshaping-and-stacking)
- [14. Sorting and Searching](#14-sorting-and-searching)
- [15. Masking and Filtering](#15-masking-and-filtering)
- [16. Performance Tips](#16-performance-tips)
- [17. Interoperability](#17-interoperability)
- [18. Advanced Topics](#18-advanced-topics)

## 1. INTRODUCTION & HISTORY

### What is NumPy
NumPy (short for Numerical Python) is an open-source Python library that is the universal standard for working with numerical data in Python. It forms the core foundation for the entire scientific and numerical computing ecosystem. At its heart, NumPy provides the `ndarray` object—a highly optimized, multidimensional array data structure.

### Why NumPy (Performance, Vectorization, Memory Efficiency)
Python is an interpreted, dynamically-typed language. While this makes Python incredibly flexible and easy to write, it introduces significant overhead for numerical computations. Native Python loops over large datasets are notoriously slow.
NumPy solves this by:
1. **Vectorization**: NumPy delegates complex mathematical operations to highly optimized, pre-compiled C and Fortran code. You can operate on entire arrays without writing explicitly nested `for` loops in Python.
2. **Memory Efficiency**: Unlike Python lists, which store pointers to disparate objects scattered across memory, NumPy arrays store homogenous data in contiguous blocks of memory. This allows modern CPUs to utilize vectorized instructions (SIMD) and caching efficiently.

### Comparison with Python Lists

| Feature | Python `list` | NumPy `ndarray` |
| :--- | :--- | :--- |
| **Data Types** | Heterogeneous (mixed types allowed) | Homogeneous (single type required) |
| **Memory Layout** | Array of pointers to objects (scattered) | Contiguous block of memory (packed) |
| **Size Flexibility** | Dynamic (can append/remove easily) | Fixed (resizing creates a new array) |
| **Math Operations** | `+` concatenates lists | `+` performs element-wise addition |
| **Speed** | Slow for large numerical data | Orders of magnitude faster |

### Role in Data Science Ecosystem (SciPy, Pandas, ML)
NumPy is the bedrock upon which the modern data stack is built. 
- **Pandas**: DataFrames are built directly on top of NumPy arrays.
- **SciPy**: Extends NumPy with algorithms for optimization, integration, and statistics.
- **Scikit-Learn**: Uses NumPy heavily for machine learning algorithms.
- **TensorFlow / PyTorch**: Though they have their own tensor objects, they are heavily inspired by NumPy's API and interoperate seamlessly with it.

### ndarray Concept
The central object in NumPy is the `ndarray` (n-dimensional array). An `ndarray` represents a multidimensional grid of values. Its behavior is defined by:
- **Data type (`dtype`)**: The type of elements (e.g., 64-bit float).
- **Shape (`shape`)**: A tuple defining the size of the array along each dimension.
- **Strides (`strides`)**: The number of bytes to step in each dimension when traversing the array.

### Version Evolution
NumPy was created in 2005 by Travis Oliphant, combining the capabilities of two earlier libraries: `Numeric` and `Numarray`. Over the past two decades, it has seen massive improvements, integrating modern C-level optimizations. Recent versions (like 1.20+) introduced robust type hinting support, and NumPy 2.0 brings structural ABI enhancements.

> 💡 **Pro Tip:** When you are doing math in Python, if you are writing a `for` loop, you are probably doing it wrong. Always look for a vectorized NumPy solution first.

> ⚠️ **Common Pitfall:** Appending to NumPy arrays is expensive. Because they are fixed-size, `np.append` creates a completely new copy of the array every time. If you need to build an array dynamically, append to a standard Python `list` first, and convert it to a NumPy array at the end.


---

## 2. INSTALLATION & SETUP

### pip install numpy
Installing NumPy is straightforward via Python's standard package manager.

```bash
# Install the latest stable version of NumPy
pip install numpy

# If you are using Conda, which is highly recommended for data science:
conda install numpy
```

### Virtual Environments
Always install NumPy within a virtual environment to prevent dependency conflicts with system-level packages or other projects.

```bash
# Creating a virtual environment
python -m venv my_numpy_env

# Activating the virtual environment (Windows)
my_numpy_env\Scripts\activate

# Activating the virtual environment (macOS/Linux)
source my_numpy_env/bin/activate

# Install NumPy in the isolated environment
pip install numpy
```

### Import Conventions
The community standard is to import NumPy using the alias `np`. You should almost never import NumPy functions directly into your namespace (e.g., `from numpy import *`) as it clutters the namespace and overrides built-in functions like `max` and `min`.

```python
# Standard import convention
import numpy as np

# DO NOT DO THIS (clutters namespace)
# from numpy import * 
```

### Checking Version
It's often necessary to check your NumPy version to ensure compatibility with other libraries like Pandas or SciPy.

```python
import numpy as np

# Print the installed NumPy version
print(f"NumPy version: {np.__version__}")
# Example output: NumPy version: 1.26.4
```

### Basic Setup in Jupyter/IDE
When working in Jupyter notebooks, it's common to set print options early on to manage how large arrays are displayed.

```python
import numpy as np

# Suppress scientific notation for small numbers
np.set_printoptions(suppress=True)

# Limit the number of items printed in large arrays
np.set_printoptions(threshold=10)

# Set precision for floating point numbers
np.set_printoptions(precision=4)
```

> 💡 **Pro Tip:** Use `conda install numpy` if possible. Conda often installs versions of NumPy that are linked against highly optimized math libraries like Intel MKL (Math Kernel Library), which can be significantly faster than the standard PyPI version for certain operations.

> ⚠️ **Common Pitfall:** Naming your script `numpy.py`. If you name your own Python file `numpy.py`, Python will import your file instead of the actual library, leading to an `AttributeError` when you try to use `np.array`.


---

## 3. NDARRAY BASICS

### Creating Arrays
There are several ways to initialize arrays in NumPy.

```python
import numpy as np

# 1. From a standard Python list (1D array)
# We pass a list of integers to np.array()
arr_1d = np.array([1, 2, 3, 4, 5])
print("1D Array:\n", arr_1d)

# 2. From a nested Python list (2D array / Matrix)
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])
print("2D Array:\n", arr_2d)

# 3. Array of zeros
# Useful for pre-allocating memory. Pass the shape as a tuple.
zeros_arr = np.zeros((3, 4)) # 3 rows, 4 columns
print("Zeros:\n", zeros_arr)

# 4. Array of ones
# Similar to zeros, creates an array filled with 1s.
ones_arr = np.ones((2, 2))
print("Ones:\n", ones_arr)

# 5. Array filled with a specific value
# Creates a 2x3 array filled with the number 7
full_arr = np.full((2, 3), 7)
print("Full:\n", full_arr)

# 6. Sequence generation (arange)
# Works like Python's built-in range(), but returns an ndarray
# Arguments: start (inclusive), stop (exclusive), step
seq_arr = np.arange(0, 10, 2)
print("Arange:\n", seq_arr) # Output: [0 2 4 6 8]

# 7. Linearly spaced sequence (linspace)
# Extremely useful for plotting. Generates exactly 'num' points between start and stop.
# Arguments: start, stop (inclusive by default), num (number of samples)
lin_arr = np.linspace(0, 1, 5)
print("Linspace:\n", lin_arr) # Output: [0.   0.25 0.5  0.75 1.  ]
```

### Array Attributes
Every `ndarray` has several essential attributes that describe its geometry and contents.

```python
arr = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.int32)

# .ndim: Number of dimensions (axes)
# Output: 2
print("Number of dimensions:", arr.ndim)

# .shape: Tuple containing the size of each dimension
# Output: (2, 3) -> 2 rows, 3 columns
print("Shape:", arr.shape)

# .size: Total number of elements in the array
# Output: 6
print("Total elements:", arr.size)

# .dtype: Data type of the elements
# Output: int32
print("Data type:", arr.dtype)

# .itemsize: Size in bytes of each individual element
# Output: 4 (since int32 takes 4 bytes)
print("Bytes per element:", arr.itemsize)

# Total memory consumed by the elements
# Output: 24 (6 elements * 4 bytes/element)
print("Total bytes:", arr.nbytes)
```

### Data Types (int, float, bool, complex)
Unlike Python lists, NumPy arrays have a unified data type.

```python
# Integer arrays (int8, int16, int32, int64)
# Explicitly declaring an 8-bit integer to save memory
int_arr = np.array([1, 2, 3], dtype=np.int8)

# Floating point arrays (float16, float32, float64)
# Python floats default to float64 in NumPy
float_arr = np.array([1.5, 2.7, 3.1])

# Boolean arrays
bool_arr = np.array([True, False, True], dtype=np.bool_)

# Complex number arrays
complex_arr = np.array([1+2j, 3+4j], dtype=np.complex128)
```

### Type Casting
You can convert an array from one data type to another using the `.astype()` method. This always creates a copy of the array.

```python
arr_float = np.array([1.1, 2.5, 3.9])

# Cast floats to integers
# NOTE: This truncates the decimal part; it does NOT round!
arr_int = arr_float.astype(np.int32)
print("Casted to int:", arr_int) # Output: [1 2 3]

# Cast integers to strings
arr_str = arr_int.astype(str)
print("Casted to string:", arr_str) # Output: ['1' '2' '3']
```

> 💡 **Pro Tip:** By default, `np.array([1, 2, 3])` will create a 64-bit integer array (`int64`) on 64-bit systems. If you are working with massive datasets where the numbers don't exceed 255, explicitly setting `dtype=np.uint8` will reduce memory usage by 8x.

> ⚠️ **Common Pitfall:** Integer overflow. If you create an array with `dtype=np.int8` (max value 127) and try to store 200, NumPy will silently wrap around (overflow) without raising an error in older versions, or behave unexpectedly.


---

## 4. ARRAY INDEXING & SLICING

### Basic Indexing
Indexing in NumPy is zero-based and very similar to standard Python lists.

```python
import numpy as np

# 1D Array indexing
arr1d = np.array([10, 20, 30, 40, 50])
print(arr1d[0])  # Output: 10 (First element)
print(arr1d[2])  # Output: 30 (Third element)

# 2D Array indexing
arr2d = np.array([[10, 20, 30], 
                  [40, 50, 60], 
                  [70, 80, 90]])

# Accessing a specific row (returns a 1D array)
print(arr2d[1]) # Output: [40 50 60]

# Accessing a specific element: arr[row, column]
# This is much faster and cleaner than arr[1][2]
print(arr2d[1, 2]) # Output: 60
```

### Negative Indexing
You can use negative indices to count from the end of the array.

```python
print(arr1d[-1]) # Output: 50 (Last element)
print(arr2d[-1, -2]) # Output: 80 (Last row, second-to-last column)
```

### Slicing (1D, 2D, nD)
Slicing uses the syntax `start:stop:step`. It allows you to extract sub-arrays.

```python
# 1D Slicing
# Extract elements from index 1 up to (but not including) 4
print(arr1d[1:4]) # Output: [20 30 40]

# Extract every second element
print(arr1d[::2]) # Output: [10 30 50]

# 2D Slicing
# Syntax: arr[row_slice, col_slice]

# Get the first two rows, and the first two columns
print(arr2d[0:2, 0:2])
# Output:
# [[10 20]
#  [40 50]]

# Get all rows, but only the second column
print(arr2d[:, 1]) # Output: [20 50 80]

# Get the last two rows, and all columns reversed
print(arr2d[-2:, ::-1])
```

### Fancy Indexing
Fancy indexing allows you to access elements using arrays of indices.

```python
arr = np.array([10, 20, 30, 40, 50, 60, 70])

# Pass a list of indices you want to extract
indices = [1, 3, 5]
print(arr[indices]) # Output: [20 40 60]

# Works in 2D as well
arr2d = np.arange(1, 10).reshape((3, 3))
row_idx = np.array([0, 2])
col_idx = np.array([1, 2])
# Selects elements at (0,1) and (2,2)
print(arr2d[row_idx, col_idx]) # Output: [2 9]
```

### Boolean Indexing
Also known as masking. You index an array using a boolean array of the same shape.

```python
arr = np.array([1, 2, 3, 4, 5, 6])

# Create a boolean mask where the condition is True
mask = arr > 3
print(mask) # Output: [False False False  True  True  True]

# Apply the mask to the array to filter elements
filtered_arr = arr[mask]
print(filtered_arr) # Output: [4 5 6]

# This is often done in a single step
even_nums = arr[arr % 2 == 0]
print(even_nums) # Output: [2 4 6]
```

### Copy vs View
This is one of the most critical concepts in NumPy.
- **Slicing returns a VIEW**: Modifying a sliced sub-array modifies the original array.
- **Fancy/Boolean indexing returns a COPY**: Modifying the result does not affect the original array.

```python
# VIEW EXAMPLE
original = np.array([1, 2, 3, 4, 5])
view_slice = original[1:4] 
view_slice[0] = 99 # Modifying the slice
print(original) # Output: [ 1 99  3  4  5] -> Original is altered!

# COPY EXAMPLE
original = np.array([1, 2, 3, 4, 5])
copy_mask = original[original > 2]
copy_mask[0] = 99 # Modifying the copy
print(original) # Output: [1 2 3 4 5] -> Original is safe!

# If you need an explicit copy from a slice, use .copy()
safe_slice = original[1:4].copy()
```

> 💡 **Pro Tip:** Use `arr2d[:, 0]` to extract the first column. This returns a 1D array. If you want to keep it as a 2D column vector, use slicing for the column index as well: `arr2d[:, 0:1]`.

> ⚠️ **Common Pitfall:** Forgetting that basic slicing returns a view. Many bugs arise when developers modify a slice, unintentionally corrupting the underlying data. Always append `.copy()` if you intend to modify a subset of data independently.


---

## 5. ARRAY OPERATIONS

### Element-wise Operations
Unlike Python lists where `+` concatenates, NumPy applies arithmetic operators on an element-by-element basis.

```python
import numpy as np

a = np.array([1, 2, 3, 4])
b = np.array([10, 20, 30, 40])

# Element-wise addition
print("Addition:", a + b)       # [11 22 33 44]

# Element-wise subtraction
print("Subtraction:", b - a)    # [ 9 18 27 36]

# Element-wise multiplication
print("Multiplication:", a * b) # [ 10  40  90 160]

# Element-wise division
print("Division:", b / a)       # [10. 10. 10. 10.]

# Element-wise exponentiation
print("Power:", a ** 2)         # [ 1  4  9 16]

# Element-wise modulo
print("Modulo:", b % 3)         # [1 2 0 1]
```

### Scalar Arithmetic
You can perform operations between an array and a single number (scalar). NumPy handles this seamlessly via Broadcasting.

```python
a = np.array([1, 2, 3])

# The scalar '10' is added to every element in the array
print(a + 10)  # Output: [11 12 13]

# The scalar '2' multiplies every element
print(a * 2)   # Output: [2 4 6]
```

### Broadcasting Rules (Brief)
Operations between arrays of different sizes are possible due to **Broadcasting**.
NumPy compares shapes element-wise, starting from the trailing (rightmost) dimensions. Two dimensions are compatible if:
1. They are equal.
2. One of them is 1.

```python
# Array shape (3, 3)
matrix = np.ones((3, 3))

# Array shape (3,)
row_vector = np.array([1, 2, 3])

# The row_vector is "broadcast" (stretched) across all rows of the matrix
result = matrix + row_vector
print(result)
# Output:
# [[2. 3. 4.]
#  [2. 3. 4.]
#  [2. 3. 4.]]
```

### Universal Functions (ufuncs)
Universal functions are mathematically optimized operations applied element-wise. Operators like `+` are actually shorthand wrappers around ufuncs like `np.add`.

```python
arr = np.array([-1.5, 2.5, 0])

# Absolute value
print(np.abs(arr)) # Output: [1.5 2.5 0. ]

# Square root (Note: sqrt of negative numbers yields nan - Not a Number)
print(np.sqrt(np.abs(arr))) 

# Exponentiation (e^x)
print(np.exp(arr))

# Sign function (returns -1 if negative, 0 if 0, 1 if positive)
print(np.sign(arr)) # Output: [-1.  1.  0.]
```

> 💡 **Pro Tip:** Element-wise multiplication uses `*`. If you want matrix multiplication (dot product), you must use the `@` operator or `np.dot()`.

> ⚠️ **Common Pitfall:** Trying to perform operations on arrays with incompatible shapes. This raises a `ValueError: operands could not be broadcast together`. Always check your `.shape` attributes if arithmetic fails.


---

## 6. RESHAPING & MANIPULATION

### reshape()
Reshaping allows you to change the dimensions of an array without changing its underlying data. The total number of elements must remain constant.

```python
import numpy as np

# Create a 1D array of 12 elements
arr1d = np.arange(12)

# Reshape into a 3x4 matrix (3 rows, 4 columns)
# 3 * 4 = 12 elements, so this works
arr2d = arr1d.reshape(3, 4)
print(arr2d)

# You can use -1 for ONE of the dimensions.
# NumPy will automatically calculate the correct size for that dimension.
# Here, we want 2 rows, and ask NumPy to figure out the columns.
arr_auto = arr1d.reshape(2, -1)
print("Auto-calculated shape:", arr_auto.shape) # Output: (2, 6)
```

### flatten() vs ravel()
Both functions flatten a multidimensional array into a 1D array, but with a critical difference in memory management.

```python
matrix = np.array([[1, 2], [3, 4]])

# flatten() always returns a COPY of the data
flat_copy = matrix.flatten()
flat_copy[0] = 99
print("Original matrix intact:\n", matrix) 

# ravel() returns a VIEW of the data whenever possible.
# It is faster and consumes less memory, but modifying it alters the original.
ravel_view = matrix.ravel()
ravel_view[0] = 99
print("Original matrix altered:\n", matrix)
```

### transpose() and swapaxes()
Transposing flips the dimensions of an array.

```python
matrix = np.array([[1, 2, 3], 
                   [4, 5, 6]]) # Shape (2, 3)

# Transpose flips rows and columns. Shape becomes (3, 2).
transposed = matrix.transpose()
# Alternatively, use the shorthand .T attribute
transposed = matrix.T
print(transposed)

# For arrays with >2 dimensions, swapaxes() lets you flip specific axes
tensor3d = np.ones((2, 3, 4))
# Swap axis 0 (size 2) with axis 2 (size 4). New shape: (4, 3, 2)
swapped = tensor3d.swapaxes(0, 2)
print("Swapped shape:", swapped.shape)
```

### concatenate(), stack(), hstack(), vstack()
These functions join multiple arrays together.

```python
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# concatenate: Joins along an EXISTING axis
# axis=0 (default) joins rows (stacks vertically)
concat_v = np.concatenate((a, b), axis=0)

# axis=1 joins columns (stacks horizontally)
concat_h = np.concatenate((a, b), axis=1)

# hstack (Horizontal Stack): Equivalent to concatenate axis=1
h_stacked = np.hstack((a, b))

# vstack (Vertical Stack): Equivalent to concatenate axis=0
v_stacked = np.vstack((a, b))

# stack: Joins arrays along a NEW axis, increasing the dimensionality
# Here, we combine two 2D arrays into one 3D array
stacked_3d = np.stack((a, b), axis=0)
print("Stack shape:", stacked_3d.shape) # Output: (2, 2, 2)
```

### split()
Splits an array into multiple sub-arrays.

```python
arr = np.array([1, 2, 3, 4, 5, 6])

# Split into 3 equal arrays
# Note: The array size must be evenly divisible by the number of splits
splits = np.split(arr, 3)
print(splits) # Output: [array([1, 2]), array([3, 4]), array([5, 6])]

# Split at specific indices (splits before index 2 and before index 5)
idx_splits = np.split(arr, [2, 5])
print(idx_splits) # Output: [array([1, 2]), array([3, 4, 5]), array([6])]
```

> 💡 **Pro Tip:** Use `reshape(-1, 1)` to convert a 1D array of shape `(N,)` into a 2D column vector of shape `(N, 1)`. This is required by many Scikit-Learn algorithms which expect 2D feature matrices.

> ⚠️ **Common Pitfall:** `np.concatenate` expects a **tuple** of arrays, not individual arguments. `np.concatenate(a, b)` will fail. You must write `np.concatenate((a, b))`.


---

## 7. MATHEMATICAL FUNCTIONS

NumPy provides highly optimized, vectorized mathematical functions that operate element-wise.

### Basic Math
```python
import numpy as np

arr = np.array([1.1, 2.5, 3.8, 4.2])

# Element-wise addition, subtraction, etc. are handled by ufuncs
np.add(arr, 2)      # Equivalent to arr + 2
np.subtract(arr, 2) # Equivalent to arr - 2
np.multiply(arr, 2) # Equivalent to arr * 2
np.divide(arr, 2)   # Equivalent to arr / 2

# Reciprocal (1 / x)
print("Reciprocal:", np.reciprocal(arr))
```

### Trigonometric Functions
NumPy handles standard trigonometry seamlessly. Angles must be provided in **radians**.

```python
# Create an array of angles from 0 to 2*Pi
angles = np.array([0, np.pi/2, np.pi, 3*np.pi/2, 2*np.pi])

# Calculate Sine, Cosine, Tangent
print("Sine:", np.sin(angles))
print("Cosine:", np.cos(angles))
print("Tangent:", np.tan(angles))

# Inverse trigonometric functions
values = np.array([-1, 0, 1])
print("ArcSine:", np.arcsin(values))
print("ArcCosine:", np.arccos(values))

# Degrees to Radians conversion
degrees = np.array([0, 90, 180])
radians = np.deg2rad(degrees)
print("Radians:", radians)
```

### Logarithmic and Exponential Functions

```python
arr = np.array([1, 2, 3])

# Natural exponential (e^x)
print("Exp:", np.exp(arr))

# Exponential minus 1 (e^x - 1). More accurate for small x.
print("Expm1:", np.expm1(arr))

# Natural logarithm (ln)
print("Ln:", np.log(arr))

# Base 10 logarithm
print("Log10:", np.log10(arr))

# Base 2 logarithm
print("Log2:", np.log2(arr))

# Logarithm of (1 + x). More accurate for small x.
print("Log1p:", np.log1p(arr))
```

### Rounding Functions

```python
floats = np.array([-1.7, -1.5, -0.2, 0.2, 1.5, 1.7, 2.0])

# np.around: Rounds to the given number of decimals.
# Note: NumPy uses "Banker's Rounding" (rounds halves to nearest even number).
# E.g., 1.5 -> 2.0, but 2.5 -> 2.0
print("Round:", np.around(floats, decimals=0))

# np.floor: Rounds down to the nearest integer
print("Floor:", np.floor(floats)) 
# Output: [-2. -2. -1.  0.  1.  1.  2.]

# np.ceil: Rounds up to the nearest integer
print("Ceil:", np.ceil(floats))
# Output: [-1. -1. -0.  1.  2.  2.  2.]

# np.trunc: Truncates (drops) the decimal part
print("Truncate:", np.trunc(floats))
# Output: [-1. -1. -0.  0.  1.  1.  2.]
```

> 💡 **Pro Tip:** If you are dealing with very small numbers where precision is easily lost, prefer `np.expm1(x)` over `np.exp(x) - 1`, and `np.log1p(x)` over `np.log(1 + x)`.

> ⚠️ **Common Pitfall:** Forgetting that `np.sin`, `np.cos`, etc. expect radians, not degrees. Feeding `np.sin(90)` will not give you 1; it computes the sine of 90 radians. Always use `np.deg2rad()` first.


---

## 8. AGGREGATION & STATISTICS

Aggregation functions reduce an array down to a single value, or reduce the dimensions of an array based on an axis.

### The Axis Concept
Understanding `axis` is crucial in multidimensional operations:
- `axis=None`: Flattens the array and computes the operation on all elements (Default).
- `axis=0`: Computes vertically **down the rows** (column-wise). The rows collapse.
- `axis=1`: Computes horizontally **across the columns** (row-wise). The columns collapse.

```python
import numpy as np

matrix = np.array([[1, 5, 9], 
                   [2, 4, 8]])

# Sum all elements
print("Total sum:", np.sum(matrix)) # Output: 29

# Sum along axis 0 (sum columns: 1+2, 5+4, 9+8)
print("Sum axis=0:", np.sum(matrix, axis=0)) # Output: [ 3  9 17]

# Sum along axis 1 (sum rows: 1+5+9, 2+4+8)
print("Sum axis=1:", np.sum(matrix, axis=1)) # Output: [15 14]
```

### Min, Max, Argmin, Argmax

```python
arr = np.array([10, 5, 20, 15, 30, 25])

# Values
print("Minimum value:", np.min(arr)) # Output: 5
print("Maximum value:", np.max(arr)) # Output: 30

# Indices (Argmin/Argmax return the INDEX of the min/max value)
print("Index of minimum:", np.argmin(arr)) # Output: 1 (which is value 5)
print("Index of maximum:", np.argmax(arr)) # Output: 4 (which is value 30)
```

### Statistical Functions

```python
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])

# Mean (Average)
print("Mean:", np.mean(data)) # Output: 5.0

# Median (Middle value)
print("Median:", np.median(data)) # Output: 4.5

# Variance (Spread of data)
# Note: ddof=0 by default (Population variance). Use ddof=1 for Sample variance.
print("Variance:", np.var(data))

# Standard Deviation (Square root of variance)
print("Standard Deviation:", np.std(data))
```

### Cumulative Operations

```python
arr = np.array([1, 2, 3, 4])

# Cumulative Sum: [1, 1+2, 1+2+3, 1+2+3+4]
print("Cumsum:", np.cumsum(arr)) # Output: [ 1  3  6 10]

# Cumulative Product: [1, 1*2, 1*2*3, 1*2*3*4]
print("Cumprod:", np.cumprod(arr)) # Output: [ 1  2  6 24]
```

### Percentiles and Quantiles

```python
data = np.arange(1, 101) # Array from 1 to 100

# Percentile: scale of 0 to 100
# Calculate the 25th, 50th (median), and 75th percentiles
print("Percentiles:", np.percentile(data, [25, 50, 75]))
# Output: [25.75 50.5  75.25]

# Quantile: scale of 0.0 to 1.0 (identical to percentile but different scale)
print("Quantiles:", np.quantile(data, [0.25, 0.5, 0.75]))
```

> 💡 **Pro Tip:** Arrays have built-in methods for the most common aggregations, so `arr.mean()` is equivalent to `np.mean(arr)`. Using the method form is often more readable in method chaining.

> ⚠️ **Common Pitfall:** Forgetting the `axis` parameter. If you want to normalize features in a 2D matrix (samples x features), calling `matrix.mean()` calculates the mean of the ENTIRE matrix. You need `matrix.mean(axis=0)` to get the mean of each feature individually.


---

## 9. LINEAR ALGEBRA

NumPy has a dedicated submodule, `numpy.linalg`, which wraps low-level LAPACK and BLAS libraries for high-performance linear algebra computations.

### Dot Product and Matrix Multiplication
The dot product behaves differently depending on the dimensions of the input arrays.

```python
import numpy as np

# 1. 1D arrays (Vectors): Inner product
v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])
# Calculation: 1*4 + 2*5 + 3*6 = 32
print("Vector Dot Product:", np.dot(v1, v2)) 

# 2. 2D arrays (Matrices): Matrix Multiplication
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Using np.dot
print("Matrix Multiplication via dot():\n", np.dot(A, B))

# Using np.matmul
print("Matrix Multiplication via matmul():\n", np.matmul(A, B))

# Using the @ operator (Python 3.5+, Highly Recommended)
print("Matrix Multiplication via @ operator:\n", A @ B)
```

### Determinant & Inverse

```python
from numpy.linalg import det, inv

matrix = np.array([[4, 7], [2, 6]])

# Determinant
# Calculates the scaling factor of the linear transformation described by the matrix
determinant = det(matrix)
print("Determinant:", determinant) # Output: 10.0

# Inverse
# Finds matrix A^-1 such that A @ A^-1 = Identity Matrix
inverse_matrix = inv(matrix)
print("Inverse Matrix:\n", inverse_matrix)

# Verification: Should result in Identity matrix (with small float inaccuracies)
print("Verification (A @ A^-1):\n", np.around(matrix @ inverse_matrix))
```

### Eigenvalues and Eigenvectors

```python
from numpy.linalg import eig

A = np.array([[1, 2], [2, 3]])

# Returns a tuple: (eigenvalues, eigenvectors)
eigenvalues, eigenvectors = eig(A)

print("Eigenvalues:", eigenvalues)
# The columns of the eigenvectors array correspond to the eigenvalues
print("Eigenvectors:\n", eigenvectors)
```

### Solving Linear Systems
Instead of calculating the inverse of A to solve `Ax = b` (which is numerically unstable and slow), it's best to use `numpy.linalg.solve`.

```python
from numpy.linalg import solve

# We want to solve the system of equations:
# 3x + 1y = 9
# 1x + 2y = 8

# Matrix of coefficients (A)
A = np.array([[3, 1], [1, 2]])

# Vector of constants (b)
b = np.array([9, 8])

# Solve for x and y
solution = solve(A, b)
print("Solution [x, y]:", solution) # Output: [2. 3.]
```

> 💡 **Pro Tip:** Always use the `@` operator for matrix multiplication instead of `np.dot`. It is much more readable. Also, NEVER use `np.linalg.inv(A) @ b` to solve linear equations. Always use `np.linalg.solve(A, b)` because it uses optimized LU decomposition and is highly numerically stable.

> ⚠️ **Common Pitfall:** Using `*` for matrix multiplication. `A * B` performs an element-wise multiplication, NOT matrix multiplication. Always use `@` for matrix math.


---

## 10. RANDOM MODULE

NumPy provides a robust suite of pseudo-random number generators under `numpy.random`.
Note: The modern standard for generating random numbers in NumPy uses the `Generator` API, which avoids altering global state.

### Modern Random Number Generation (Generator API)

```python
import numpy as np

# Instantiate a default random number generator
# This is preferred over the legacy np.random.rand() functions
rng = np.random.default_rng()

# Generate random floats between 0.0 and 1.0
# Shape provided as a tuple or individual arguments depending on function
floats = rng.random((2, 3)) # 2x3 matrix of random floats
print("Random floats:\n", floats)

# Generate random integers
# Arguments: low (inclusive), high (exclusive), size
ints = rng.integers(0, 10, size=5)
print("Random integers:", ints)
```

### Legacy Methods (Still widely used)
You will frequently see these legacy methods in older tutorials.

```python
# Random floats in [0.0, 1.0)
print(np.random.rand(2, 3)) 

# Standard Normal Distribution (mean=0, variance=1)
print(np.random.randn(5))

# Random integers
print(np.random.randint(0, 10, size=5))
```

### Seeding (Reproducibility)
To ensure your randomized results are repeatable across different runs, you must set a seed.

```python
# Legacy seeding (Sets global state - discouraged for large applications)
np.random.seed(42)
print("Legacy Seeded rand:", np.random.rand(3))

# Modern seeding (Isolates state to the generator instance)
rng = np.random.default_rng(seed=42)
print("Modern Seeded rand:", rng.random(3))
```

### Distributions
NumPy can draw samples from a wide variety of probability distributions.

```python
rng = np.random.default_rng()

# 1. Uniform Distribution
# Every value in the range [low, high) has an equal chance of being drawn
uniform_data = rng.uniform(low=-1.0, high=1.0, size=5)

# 2. Normal (Gaussian) Distribution
# Bell curve. Arguments: loc (mean), scale (standard deviation), size
normal_data = rng.normal(loc=0.0, scale=1.0, size=5)

# 3. Binomial Distribution
# Number of successes in n independent trials with probability p
# E.g., Flipping a fair coin (p=0.5) 10 times. Do this experiment 5 times.
binomial_data = rng.binomial(n=10, p=0.5, size=5)

# 4. Poisson Distribution
# Used for counting occurrences over a fixed interval (e.g., website clicks per hour)
# Argument: lam (expected number of events)
poisson_data = rng.poisson(lam=5.0, size=5)
```

> 💡 **Pro Tip:** Always use `np.random.default_rng(seed)` for new codebases. Legacy methods modify global state, which means importing a third-party library that calls `np.random.seed()` could secretly break the reproducibility of your own scripts!

> ⚠️ **Common Pitfall:** Assuming `np.random.randint(0, 10)` includes 10. It does not. The upper bound is exclusive. It returns integers from 0 up to 9.


---

## 11. BROADCASTING (DEEP DIVE)

Broadcasting is NumPy's magical mechanism for performing mathematical operations on arrays of different shapes without manually duplicating data in memory.

### Broadcasting Rules Explained
When operating on two arrays, NumPy compares their shapes element-wise. It starts with the **trailing (rightmost) dimensions** and works its way left.
Two dimensions are compatible if:
1. They are equal, OR
2. One of them is 1.

If these conditions are not met, a `ValueError: operands could not be broadcast together` is thrown.

### Example 1: Scalar to Array
```python
import numpy as np

arr = np.array([1, 2, 3]) # Shape (3,)
scalar = 5                # Shape () -> conceptually treated as (1,)

# The scalar is virtually stretched to shape (3,) to match 'arr'
result = arr + scalar
# Result: [6, 7, 8]
```

### Example 2: 1D Array to 2D Array
```python
matrix = np.ones((3, 4))      # Shape (3, 4)
vector = np.array([1, 2, 3, 4]) # Shape (4,)

# Alignment evaluation:
# Matrix: 3 x 4
# Vector:     4
# -------------
# Result: 3 x 4 (Compatible!)

# The vector is added to every ROW of the matrix.
print(matrix + vector)
```

### Example 3: Incompatible Shapes
```python
matrix = np.ones((3, 4))      # Shape (3, 4)
vector = np.array([1, 2, 3])  # Shape (3,)

# Alignment evaluation:
# Matrix: 3 x 4
# Vector:     3
# -------------
# Dimension mismatch on the rightmost edge! 4 != 3 and neither is 1.
# np.add(matrix, vector) -> ValueError!

# FIX: We must explicitly reshape the vector to a column vector (3, 1)
# Matrix: 3 x 4
# Vector: 3 x 1
# -------------
# Result: 3 x 4 (Compatible! Vector is stretched across columns)

col_vector = vector.reshape(-1, 1)
print(matrix + col_vector)
```

### Example 4: Creating a Grid using Broadcasting
```python
# Create a row vector
x = np.array([0, 1, 2])       # Shape (3,)
# Create a column vector
y = np.array([[0], [1], [2]]) # Shape (3, 1)

# x:     3
# y: 3 x 1
# --------
# R: 3 x 3
# 'x' is stretched vertically, 'y' is stretched horizontally.
# This creates a 2D coordinate grid evaluating x + y at every point!
grid = x + y
print(grid)
# [[0 1 2]
#  [1 2 3]
#  [2 3 4]]
```

### Performance Benefits
Broadcasting happens entirely in C. It does NOT allocate new memory to stretch the smaller array into the larger shape. It simply reuses the existing memory via internal looping mechanisms (strides), saving massive amounts of RAM and execution time compared to manually duplicating rows or columns using `np.tile`.

> 💡 **Pro Tip:** You can quickly add a new dimension of size 1 to an array using `np.newaxis` (or `None`). For example, `arr[:, np.newaxis]` turns a 1D array of shape `(N,)` into a 2D column vector of shape `(N, 1)`, fixing the alignment issue in Example 3 effortlessly.

> ⚠️ **Common Pitfall:** Accidentally broadcasting when you didn't mean to. If you expect a matrix-matrix addition but mistakenly pass a row vector, NumPy won't crash; it will silently broadcast, leading to logical errors in your math that are very hard to debug.


---

## 12. ADVANCED INDEXING

Advanced indexing is triggered when the selection object is an `ndarray` of integer or boolean types, or a tuple with at least one sequence object. Advanced indexing always returns a **copy** of the data.

### Fancy Indexing with Integer Arrays
You can use integer arrays as indices to grab arbitrary elements out of order.

```python
import numpy as np

arr = np.array([10, 20, 30, 40, 50])

# Using a list of indices
indices = [1, 3, 4]
print("Selected elements:", arr[indices]) # Output: [20 40 50]

# You can even request the same index multiple times
print("Repeated selection:", arr[[0, 0, 1]]) # Output: [10 10 20]

# Multi-dimensional fancy indexing
matrix = np.arange(9).reshape(3, 3)
# matrix looks like:
# [[0, 1, 2],
#  [3, 4, 5],
#  [6, 7, 8]]

row_indices = [0, 2]
col_indices = [0, 2]

# This selects the corners: matrix[0,0] and matrix[2,2]
print("Diagonals:", matrix[row_indices, col_indices]) # Output: [0 8]

# To get the sub-matrix (intersection of rows and cols), use np.ix_
sub_matrix = matrix[np.ix_(row_indices, col_indices)]
print("Sub-matrix:\n", sub_matrix)
# [[0 2]
#  [6 8]]
```

### Boolean Masks & Filtering
Boolean indexing is the primary way to filter data.

```python
data = np.random.randint(0, 100, 10)
print("Original data:", data)

# Create a boolean array (mask)
mask = data > 50

# Apply the mask to extract values
filtered_data = data[mask]
print("Values > 50:", filtered_data)

# Combining conditions using bitwise operators: & (AND), | (OR), ~ (NOT)
# Note: Parentheses around conditions are MANDATORY due to operator precedence
complex_mask = (data > 20) & (data < 80)
print("Values between 20 and 80:", data[complex_mask])

# ~ is the logical NOT operator
print("Values NOT between 20 and 80:", data[~complex_mask])
```

### Conditional Operations (np.where)
`np.where` is an incredibly powerful vectorized version of the ternary operator `x if condition else y`.

```python
arr = np.array([1, 2, 3, 4, 5])

# Syntax: np.where(condition, value_if_true, value_if_false)

# Replace values > 3 with 10, otherwise keep original value
modified = np.where(arr > 3, 10, arr)
print("Modified array:", modified) # Output: [ 1  2  3 10 10]

# Find indices where condition is true
# If only the condition is provided, it returns a tuple of indices
indices = np.where(arr > 3)
print("Indices where > 3:", indices[0]) # Output: [3 4]
```

### np.select
When you have more than two conditions, `np.select` is better than nested `np.where` statements.

```python
arr = np.arange(10)

# Define a list of conditions
conditions = [
    arr < 3,
    (arr >= 3) & (arr < 7),
    arr >= 7
]

# Define the corresponding values for those conditions
choices = [
    0,   # If < 3
    50,  # If between 3 and 6
    100  # If >= 7
]

# Apply conditions
result = np.select(conditions, choices, default=-1)
print("np.select result:", result)
# Output: [  0   0   0  50  50  50  50 100 100 100]
```

> 💡 **Pro Tip:** In Pandas, the `.loc` and `.iloc` accessors use NumPy advanced indexing under the hood. Understanding `np.ix_` and boolean masks makes mastering Pandas much easier.

> ⚠️ **Common Pitfall:** Using `and` and `or` keywords for boolean masks instead of `&` and `|`. Python's `and` attempts to evaluate the truth value of the *entire array*, which makes no sense and raises a `ValueError`. Always use bitwise operators `&` and `|` with parentheses.


---

## 13. MEMORY MANAGEMENT

NumPy achieves its speed by interacting directly with contiguous blocks of RAM. Understanding how NumPy manages memory separates intermediate users from experts.

### Memory Layout (C vs Fortran Order)
Data in memory is strictly 1-dimensional. Multidimensional arrays are an abstraction achieved by interpreting that 1D memory sequentially.
- **C-order (Row-major, Default)**: Rows are stored contiguously in memory. Iterating through rows is fast.
- **Fortran-order (Column-major)**: Columns are stored contiguously in memory. Iterating through columns is fast.

```python
import numpy as np

# Default is C-order
c_array = np.array([[1, 2], [3, 4]], order='C')

# Force Fortran-order
f_array = np.array([[1, 2], [3, 4]], order='F')

# You can check memory layout via the flags attribute
print("Is C contiguous?:", c_array.flags.c_contiguous) # True
print("Is F contiguous?:", c_array.flags.f_contiguous) # False
```

### Strides
Strides are the number of bytes NumPy must skip in memory to move to the next element in a specific dimension.
Strides are the secret to NumPy's views. When you slice or transpose an array, NumPy doesn't move the data; it simply alters the strides metadata!

```python
# Create an array of 32-bit integers (4 bytes per element)
arr = np.array([[1, 2, 3], 
                [4, 5, 6]], dtype=np.int32)

# Shape is (2, 3)
# To move to the next row (axis 0), we must skip 3 elements (12 bytes)
# To move to the next col (axis 1), we must skip 1 element (4 bytes)
print("Strides:", arr.strides) # Output: (12, 4)

# When we transpose, NumPy just swaps the strides!
transposed = arr.T
print("Transposed Strides:", transposed.strides) # Output: (4, 12)

# Notice no new memory was allocated. Both share the same base memory.
print("Shares memory:", np.shares_memory(arr, transposed)) # Output: True
```

### Views vs Copies
Because of strides, many operations return views (pointers to the same memory).

- **Operations that return Views**: Basic slicing `arr[:]`, transposing `.T`, reshaping `reshape()` (usually), `ravel()`.
- **Operations that return Copies**: Advanced indexing `arr[[0,1]]`, boolean masking `arr[arr>0]`, `flatten()`, `.astype()`.

```python
# Detecting if an array is a view
base_arr = np.arange(10)
view_arr = base_arr[2:5]

# .base points to the original memory owner if it's a view. None if it owns its data.
print("Does view own data?", view_arr.flags.owndata) # False
print("Base object:", view_arr.base is base_arr) # True
```

### Efficient Memory Usage
When writing highly performant code, unnecessary array copies destroy performance and consume RAM rapidly.

```python
a = np.ones(10000)
b = np.ones(10000)

# INEFFCIENT: Creates an intermediate array for (a*2), then another for the sum
c = a * 2 + b

# EFFICIENT: In-place operations avoid allocating new memory
# This alters 'a' in memory directly
a *= 2
a += b 

# The out= parameter in ufuncs allows you to write results directly into pre-allocated memory
result = np.empty_like(a) # Allocate uninitialized memory
np.add(a, b, out=result)  # Write result directly into 'result' memory block
```

> 💡 **Pro Tip:** Use `np.shares_memory(a, b)` to debug whether you accidentally created a copy or a view. If your memory consumption spikes unexpectedly, you're likely creating unnecessary copies via fancy indexing.

> ⚠️ **Common Pitfall:** Memory Leaks via Views. If you load a massive 10GB array, slice a tiny 1MB piece of it, and keep that slice in memory while deleting the main array, the ENTIRE 10GB array remains in RAM because the slice's view prevents garbage collection. Always use `.copy()` when extracting small parts of huge datasets that you want to discard.


---

## 14. FILE I/O

NumPy provides fast, specialized methods for saving arrays to disk and loading them back. It supports both plain text (CSV/TXT) and highly efficient binary formats.

### Saving Arrays (save, savez)
NumPy's proprietary `.npy` format is incredibly fast and preserves all metadata (shape, dtype) perfectly.

```python
import numpy as np

arr = np.arange(100).reshape(10, 10)

# 1. Saving a single array to a binary file (.npy)
np.save("my_array.npy", arr)

# 2. Saving multiple arrays into a single compressed archive (.npz)
arr_b = np.linspace(0, 1, 10)
# Pass arrays as keyword arguments to name them in the archive
np.savez("my_archive.npz", array_a=arr, array_b=arr_b)

# 3. Save compressed (better for storage, slightly slower to read/write)
np.savez_compressed("my_archive_comp.npz", array_a=arr)
```

### Loading Arrays (load)

```python
# 1. Load single binary array
loaded_arr = np.load("my_array.npy")
print("Loaded shape:", loaded_arr.shape)

# 2. Load .npz archive
archive = np.load("my_archive.npz")
# The archive acts like a dictionary
loaded_a = archive['array_a']
loaded_b = archive['array_b']

# Important: Close the archive when done if working with large files
archive.close()
```

### Text Files (loadtxt, savetxt)
For interoperability with Excel, R, or other text-based tools, NumPy can read and write CSVs. Note: Pandas `pd.read_csv` is generally faster and more robust for text files, but NumPy handles basic cases well.

```python
data = np.random.rand(5, 3)

# 1. Save to a CSV text file
# fmt='%.4f' formats numbers to 4 decimal places
# delimiter=',' makes it a CSV
np.savetxt("data.csv", data, delimiter=",", fmt='%.4f', header="Col1,Col2,Col3")

# 2. Load from a CSV text file
# skip_header=1 skips the column names we just wrote
loaded_csv = np.loadtxt("data.csv", delimiter=",", skiprows=1)
print("Loaded CSV:\n", loaded_csv)
```

### Binary vs Text Formats
| Feature | `.npy` / `.npz` (Binary) | `.csv` / `.txt` (Text) |
| :--- | :--- | :--- |
| **Speed** | Extremely fast | Very slow (requires text parsing) |
| **File Size** | Small (especially compressed) | Large (stored as ascii strings) |
| **Metadata** | Preserves `shape` and `dtype` | Loses type info (everything read as float) |
| **Human Readable** | No | Yes |
| **Interoperability**| Python ecosystem | Universal (Excel, C++, R, Java) |

> 💡 **Pro Tip:** If your array contains objects or strings, `np.save` will set `allow_pickle=True` by default. For security reasons (pickles can execute malicious code), modern NumPy requires you to explicitly specify `allow_pickle=True` when loading it back.

> ⚠️ **Common Pitfall:** Using `np.loadtxt` for massive files with missing values. It will fail. If you have messy text data, use `np.genfromtxt` (which handles missing values) or, ideally, switch to Pandas `pd.read_csv`.


---

## 15. PERFORMANCE OPTIMIZATION

NumPy is fast, but bad code can make it slow. Understanding how to optimize array operations is key.

### Vectorization vs Loops
The golden rule of NumPy: **Never use a python `for` loop to iterate over an array.**

```python
import numpy as np
import time

# Create a huge array
size = 10_000_000
data = np.random.rand(size)

# BAD: Python Loop (Extremely Slow)
start_time = time.time()
sum_loop = 0
for val in data:
    sum_loop += val
print(f"Loop time: {time.time() - start_time:.4f} seconds") 
# e.g., ~1.2000 seconds

# GOOD: Vectorized built-in function (Extremely Fast)
start_time = time.time()
sum_vec = np.sum(data)
print(f"Vectorized time: {time.time() - start_time:.4f} seconds") 
# e.g., ~0.0050 seconds (200x faster)
```

### Avoiding Python loops with `np.vectorize`
If you have a custom Python function that only works on single scalars, you can "vectorize" it.
**Warning:** `np.vectorize` is a convenience function. It is essentially a hidden loop in Python and does *not* provide C-level speedups. It just cleans up syntax.

```python
import math

# A function that expects a single integer
def custom_math_func(x):
    if x % 2 == 0:
        return x ** 2
    else:
        return x ** 3

# Vectorize the function
vec_func = np.vectorize(custom_math_func)

arr = np.array([1, 2, 3, 4])
# Apply it to the whole array
print(vec_func(arr)) # Output: [ 1  4 27 16]

# For true performance, you should rewrite the logic using boolean masks:
# This actually runs at C-speed
result = np.where(arr % 2 == 0, arr ** 2, arr ** 3)
```

### Profiling Basics
To optimize code, you need to know what is slow. Use IPython/Jupyter magic commands.

```python
# In Jupyter or IPython, use %timeit
# %timeit np.sum(data)
```

### Using NumPy Efficiently
1. **Preallocate memory**: Don't append arrays in a loop. Create an `np.empty` array of the final size and fill it by index.
2. **In-place operations**: Use `+=`, `*=`, or the `out` parameter in ufuncs to avoid allocating intermediate arrays.
3. **Use the right dtype**: Using `float64` when `float32` provides enough precision halves your memory footprint and doubles cache efficiency.
4. **Mind the memory layout**: If doing heavy operations along columns, use Fortran-ordered arrays. Operating across non-contiguous memory causes cache misses and drastically slows down execution.

> 💡 **Pro Tip:** If you absolutely must use complex `for` loops that can't be vectorized, use `Numba`. Just adding the `@numba.jit(nopython=True)` decorator to your Python function will compile it to LLVM machine code, making your loops run as fast as C.

> ⚠️ **Common Pitfall:** Calling `np.append` or `np.concatenate` inside a loop. This requires allocating a new, slightly larger array and copying all existing data on every iteration. This leads to `O(N^2)` time complexity.


---

## 16. INTEROPERABILITY

NumPy is designed to play nicely with the rest of the Python ecosystem.

### NumPy with Pandas
Pandas is built on top of NumPy. A Pandas Series or DataFrame column is essentially a NumPy array with an index attached.

```python
import numpy as np
import pandas as pd

# Creating a DataFrame from a NumPy matrix
matrix = np.array([[1, 2], [3, 4]])
df = pd.DataFrame(matrix, columns=['A', 'B'])
print(df)

# Extracting the underlying NumPy array from a DataFrame
# (Using .to_numpy() is preferred over the older .values attribute)
extracted_arr = df['A'].to_numpy()
print(type(extracted_arr)) # <class 'numpy.ndarray'>
```

### NumPy with SciPy
SciPy (`scipy`) extends NumPy. It uses NumPy arrays natively for all its scientific algorithms.

```python
from scipy import optimize

# Example: Finding the minimum of a function
def f(x):
    return x**2 + 10*np.sin(x)

# optimize.minimize takes scalar bounds but processes NumPy arrays under the hood
result = optimize.minimize(f, x0=0)
print("Minimum found at x =", result.x[0])
```

### Conversion to/from Lists
Converting between native Python lists and NumPy arrays is a fundamental operation.

```python
# List -> Array
my_list = [[1, 2], [3, 4]]
arr = np.array(my_list)

# Array -> List
# Use .tolist() to convert back. It handles nested multidimensional arrays automatically.
converted_list = arr.tolist()
print(type(converted_list)) # <class 'list'>
print(converted_list)       # [[1, 2], [3, 4]]
```

### Interfacing with C/C++ Data (ctypes / Memory Views)
NumPy implements the Python buffer protocol. You can directly share memory between a Python bytearray, PIL image, or C++ structure without copying the data.

```python
# Convert a raw string of bytes into a NumPy array
byte_data = b'\x01\x02\x03\x04'
# Interprets the bytes as 8-bit unsigned integers
arr_from_bytes = np.frombuffer(byte_data, dtype=np.uint8)
print(arr_from_bytes) # Output: [1 2 3 4]
```

> 💡 **Pro Tip:** When passing arrays to older C/C++ extensions, they might require contiguous memory. Use `np.ascontiguousarray(arr)` before passing the array to ensure it is stored contiguously in C-order, preventing segfaults.

> ⚠️ **Common Pitfall:** Using `list(arr)` on a multidimensional array. This only converts the outer dimension to a list; the inner elements remain NumPy arrays. Always use the `.tolist()` method for a complete, deep conversion to nested Python lists.


---

## 17. UNIVERSAL FUNCTIONS (UFUNCS)

Universal functions are the engine driving NumPy's fast element-wise operations. They operate on `ndarray`s element-by-element, support array broadcasting, type casting, and several other standard features.

### Built-in Ufuncs
All standard math operators (`+`, `-`, `*`, `/`) map to ufuncs (`np.add`, `np.subtract`, etc.).
Other ufuncs include:
- `np.logical_and`, `np.logical_or`
- `np.bitwise_and`, `np.bitwise_or`
- `np.isfinite`, `np.isnan`, `np.isinf`
- `np.maximum`, `np.minimum` (element-wise comparisons)

### Ufunc Methods (reduce, accumulate, outer)
Ufuncs taking two arguments (binary ufuncs) have special methods that allow powerful array reductions.

```python
import numpy as np

arr = np.array([1, 2, 3, 4])

# 1. reduce: Applies the ufunc successively to all elements, reducing it to a single value.
# np.add.reduce is exactly what np.sum() does under the hood.
sum_val = np.add.reduce(arr)
print("Reduce (Sum):", sum_val) # Output: 10

# 2. accumulate: Similar to reduce, but stores intermediate results.
# np.add.accumulate is exactly what np.cumsum() does.
cum_sum = np.add.accumulate(arr)
print("Accumulate:", cum_sum) # Output: [ 1  3  6 10]

# 3. outer: Applies the ufunc to all possible pairs of elements from two inputs.
a = np.array([1, 2, 3])
b = np.array([10, 20])
# Creates a 3x2 matrix where result[i,j] = a[i] * b[j]
outer_prod = np.multiply.outer(a, b)
print("Outer product:\n", outer_prod)
# [[10 20]
#  [20 40]
#  [30 60]]
```

### Custom Ufuncs (`np.frompyfunc`)
You can define your own ufuncs from a standard python function. This allows your custom function to broadcast automatically. Note: This still runs at Python speed, similar to `np.vectorize`.

```python
def my_custom_add(x, y):
    return x + y + 10

# Convert to ufunc. 
# Arguments: function, number of inputs, number of outputs
my_ufunc = np.frompyfunc(my_custom_add, 2, 1)

print(my_ufunc([1, 2], [10, 20])) # Output: [21 32]
```

> 💡 **Pro Tip:** `np.maximum.reduce` is the fastest way to find the max value across an axis. It avoids the small overhead of the generic `np.max` dispatcher.

> ⚠️ **Common Pitfall:** Confusing `np.max` with `np.maximum`. `np.max` aggregates an array to find a single highest value. `np.maximum` compares two arrays element-wise and returns a new array with the largest element at each index.


---

## 18. MASKED ARRAYS

Sometimes datasets contain missing or invalid entries (like `NaN`, nulls, or sensor errors). Normal aggregations (like `np.mean`) will fail or return `NaN` if a single `NaN` is present.
The `numpy.ma` (Masked Array) module provides a way to explicitly "hide" these invalid values.

### Handling Missing Data / Mask Creation

```python
import numpy as np
import numpy.ma as ma

# Raw data with a -999 value representing a missing sensor reading
data = np.array([10.5, 12.1, -999.0, 11.2, 10.9])

# Standard mean fails because of the outlier
print("Standard Mean:", np.mean(data)) # -190.86 (Incorrect!)

# Create a masked array where values == -999 are masked (ignored)
masked_data = ma.masked_values(data, -999.0)
print("Masked Array representation:\n", masked_data)
# Output: [10.5 12.1 -- 11.2 10.9]

# Alternatively, provide an explicit boolean mask (True = hidden)
mask = [False, False, True, False, False]
masked_data_explicit = ma.masked_array(data, mask=mask)
```

### Operations on Masked Arrays
Operations on masked arrays automatically ignore the masked values.

```python
# The mean is now calculated only on the 4 valid data points
print("Masked Mean:", np.mean(masked_data)) # 11.175

# Getting the underlying mask
print("The Mask:", masked_data.mask) # [False False  True False False]

# Filling masked values with a default value (e.g., the mean)
filled_data = masked_data.filled(masked_data.mean())
print("Filled Data:", filled_data)
# Output: [10.5 12.1 11.175 11.2 10.9]
```

> 💡 **Pro Tip:** Pandas heavily relies on `np.nan` for missing data, and provides `df.mean(skipna=True)`. However, `np.nan` only works with floating-point arrays. Masked Arrays work with integers and booleans without forcing a cast to floats.

> ⚠️ **Common Pitfall:** Using `np.sum` on an array with `NaN` returns `NaN`. If you don't want to use the `ma` module, you can use NumPy's `nan`-aware functions like `np.nansum()`, `np.nanmean()`, etc.


---

## 19. STRUCTURED ARRAYS

NumPy arrays are strictly homogeneous (one data type). But what if you need to store tabular data where columns have different types (like a CSV with names, ages, and weights)?
Structured arrays allow you to define compound data types. (Note: Pandas DataFrames are generally preferred for this today, but structured arrays are highly efficient and useful in C-interfacing).

### Defining Structured Dtypes

```python
import numpy as np

# Define a custom dtype mapping field names to data types
# 'U10' = 10-character Unicode string
# 'i4'  = 32-bit integer
# 'f8'  = 64-bit float
custom_dtype = np.dtype([('name', 'U10'), ('age', 'i4'), ('weight', 'f8')])

# Initialize an empty structured array with 3 slots
people = np.zeros(3, dtype=custom_dtype)
print("Empty structured array:\n", people)
# [('', 0, 0.) ('', 0, 0.) ('', 0, 0.)]

# Populate the array
people[0] = ('Alice', 25, 65.5)
people[1] = ('Bob', 30, 80.0)
people[2] = ('Charlie', 35, 75.2)

print("Populated array:\n", people)
```

### Accessing Fields
You access columns in a structured array by their field name (like a dictionary).

```python
# Extract the 'name' column
print("Names:", people['name']) # Output: ['Alice' 'Bob' 'Charlie']

# Extract the 'age' column and compute mean
print("Average Age:", np.mean(people['age'])) # Output: 30.0

# Boolean filtering still works
# Find names of people older than 28
older_than_28 = people[people['age'] > 28]['name']
print("Over 28:", older_than_28) # Output: ['Bob' 'Charlie']
```

> 💡 **Pro Tip:** Structured arrays are extremely useful when reading binary files generated by C/C++ structs. You can map the C `struct` layout directly to a NumPy `dtype` and read thousands of records instantaneously from a file using `np.fromfile()`.

> ⚠️ **Common Pitfall:** Trying to treat a structured array exactly like a Pandas DataFrame. Structured arrays don't have indexes or fancy methods like `.groupby()`. If you need advanced data manipulation, wrap the structured array in `pd.DataFrame(people)`.


---

## 20. ADVANCED TOPICS

### Memory Mapping (memmap)
Memory mapping allows you to interact with massive files stored on disk as if they were entirely in RAM, without actually loading them into RAM. It reads only the slice of the array you request.

```python
import numpy as np
import os

# Create a massive dummy file on disk using memory mapping
# Shape: 10,000 x 10,000 floats (approx 800 MB)
filename = "massive_array.dat"
fp = np.memmap(filename, dtype='float64', mode='w+', shape=(10000, 10000))

# Write data to a small chunk. Only this chunk is loaded/written to RAM.
fp[0:5, 0:5] = np.random.rand(5, 5)

# Flush changes to disk
fp.flush()

# Later, read it back without loading 800MB into memory
read_fp = np.memmap(filename, dtype='float64', mode='r', shape=(10000, 10000))
print("Data from disk:\n", read_fp[0:2, 0:2])

# Cleanup
del fp, read_fp
os.remove(filename)
```

### NumPy Internals Overview
A NumPy array object (`PyArrayObject` in C) consists of:
1. **Pointer to data**: A memory address pointing to the first byte of raw data.
2. **Data type (`dtype`)**: Describes how to interpret fixed-size bytes (e.g., float64 = 8 bytes).
3. **Shape**: Tuple defining dimensions.
4. **Strides**: Tuple defining bytes to step per dimension.
Because it's just raw memory and metadata, reshaping and slicing only update the metadata (shape/strides), which is why they operate in `O(1)` time.

### Extending NumPy (C/Cython intro)
If vectorized operations aren't enough, you can write C or Cython code that interacts directly with NumPy's C-API.

**Conceptual Cython Example:**
```cython
# cython: boundscheck=False, wraparound=False
import numpy as np
cimport numpy as np

# Cython can interact directly with the C memory buffer of the ndarray
def fast_multiply(np.ndarray[np.float64_t, ndim=1] arr, double val):
    cdef int i
    cdef int size = arr.shape[0]
    
    # This loop is compiled to C and runs at hardware speed
    for i in range(size):
        arr[i] = arr[i] * val
```

### Parallel Computing Basics
NumPy drops the Python Global Interpreter Lock (GIL) when executing C-level routines.
- Operations like `np.dot` (matrix multiplication) are automatically parallelized across multiple CPU cores if NumPy is linked to an optimized BLAS library (like OpenBLAS or Intel MKL).
- Element-wise operations (like `+`, `*`, `np.exp`) are usually NOT parallelized by default.
- To parallelize element-wise loops, consider using **Numba** (`@njit(parallel=True)`) or **Dask** (which chunks NumPy arrays and runs operations across multiple threads/machines).

### Einstein Summation Convention (einsum)
`np.einsum` is a profoundly powerful function that can perform matrix multiplication, dot products, transposes, and traces using a concise string notation representing Einstein summation.

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Matrix Multiplication using einsum
# 'ij,jk->ik' means: multiply row i of A with col k of B, sum over j
matrix_mult = np.einsum('ij,jk->ik', A, B)
print("Einsum MatMul:\n", matrix_mult)

# Transpose using einsum
transpose_A = np.einsum('ij->ji', A)

# Trace (sum of diagonal)
trace_A = np.einsum('ii->', A)
```

> 💡 **Pro Tip:** Learn `np.einsum`. It often replaces complex chains of `reshape()`, `transpose()`, and `dot()` operations, reducing memory footprint and improving readability for advanced tensor operations (common in Deep Learning).

> ⚠️ **Common Pitfall:** Assuming NumPy utilizes all your CPU cores for everything. Simple operations like `np.sin(arr)` run on a single thread. Only specific linear algebra routines benefit from multi-threading out-of-the-box. Use profiling to verify CPU utilization.
