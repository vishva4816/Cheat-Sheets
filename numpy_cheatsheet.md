# NumPy Cheat Sheet

> Convention: `import numpy as np`

---

## 1. Array Creation

```python
np.array([1, 2, 3])              # from a list → 1D
np.array([[1, 2], [3, 4]])       # nested list → 2D

np.zeros((2, 3))                 # 2x3 of 0.0
np.ones((2, 3))                  # 2x3 of 1.0
np.full((2, 3), 7)               # 2x3 of 7
np.empty((2, 3))                 # uninitialized (garbage values)
np.eye(3)                        # 3x3 identity matrix

np.arange(0, 10, 2)              # [0 2 4 6 8]  (start, stop, step)
np.linspace(0, 1, 5)             # 5 evenly spaced points 0→1 inclusive

np.array([1, 2, 3], dtype=np.float64)   # set dtype at creation
np.zeros_like(arr)               # zeros matching arr's shape & dtype
```

---

## 2. Array Attributes

```python
arr.shape       # tuple of dimensions, e.g. (3, 4)
arr.ndim        # number of dimensions (axes)
arr.size        # total number of elements
arr.dtype       # element data type (int64, float64, bool, ...)
arr.itemsize    # bytes per element
arr.nbytes      # total bytes = size * itemsize
arr.T           # transpose
```

---

## 3. Arithmetic Between Arrays

Operations are **element-wise** and **vectorized** (no loops).

```python
a + b      a - b      a * b      a / b
a ** 2     a % b      a // b
a > b      a == b                 # element-wise comparison → bool array

a + 10                            # scalar broadcasts to every element
```

**Broadcasting** — smaller array is "stretched" to match the larger:

```python
a = np.array([[1, 2, 3],          # shape (2, 3)
              [4, 5, 6]])
b = np.array([10, 20, 30])        # shape (3,)
a + b                             # b applied to each row → (2, 3)
```

Rule: dimensions are compatible when they are equal or one of them is 1.

---

## 4. Indexing & Slicing

### Basic indexing (views — no copy)
```python
arr[0]           # first element (or first row if 2D)
arr[-1]          # last
arr[2:5]         # slice
arr[::2]         # every 2nd element
arr[1, 2]        # 2D: row 1, col 2
arr[1]           # 2D: entire row 1
arr[:, 0]        # 2D: entire column 0
arr[0:2, 1:3]    # sub-matrix
arr[1:3] = 0     # assignment broadcasts
```

### Boolean indexing (returns a copy)
```python
arr[arr > 5]                 # all elements > 5 (flattened)
arr[arr > 5] = 0             # set those elements to 0
mask = (arr > 2) & (arr < 8) # combine with & |  ~  (NOT  and/or/not)
arr[mask]
names == "Bob"               # boolean mask from comparison
data[names == "Bob"]         # use mask to select rows
```

### Fancy indexing (integer arrays — returns a copy)
```python
arr[[4, 3, 0, 6]]            # select rows in this order
arr[[0, 2], [1, 3]]          # picks elements (0,1) and (2,3)
arr[[1, 5, 7]][:, [0, 3, 2]] # select rows then reorder columns
```

---

## 5. Copying an ndarray

```python
b = a              # NOT a copy — same object, both names point to it
b = a.view()       # new array object, SHARES data (changes propagate)
b = a.copy()       # independent deep copy (safe to modify)

# Slices are VIEWS, so this modifies the original:
s = a[2:5]
s[:] = 0           # a is changed too!

# To avoid that:
s = a[2:5].copy()
```

---

## 6. Transposing & Swapping Axes

```python
arr.T                        # transpose (reverse all axes)
arr.transpose(1, 0, 2)       # reorder axes explicitly (by index)
arr.swapaxes(0, 1)           # swap two specific axes
np.dot(arr.T, arr)           # common use: matrix multiplication
```

For 2D, `.T` flips rows ↔ columns. Transpose returns a **view**.

---

## 7. Pseudo-Random Generator

Modern API (preferred):
```python
rng = np.random.default_rng(seed=42)   # create a generator
rng.random((2, 3))            # uniform floats in [0, 1)
rng.integers(0, 10, size=5)   # random ints, low inclusive, high exclusive
rng.normal(0, 1, size=5)      # normal: mean=0, std=1
rng.standard_normal((2, 3))   # standard normal
rng.choice([1, 2, 3], size=4) # sample from given values
rng.shuffle(arr)              # shuffle in place
rng.permutation(10)           # shuffled arange(10)
```

Legacy API (still common in books/tutorials):
```python
np.random.seed(42)
np.random.rand(2, 3)          # uniform [0,1)
np.random.randn(2, 3)         # standard normal
np.random.randint(0, 10, 5)   # random ints
```

> Seeding makes results reproducible.

---

## 8. Universal Functions (ufuncs)

Element-wise functions, fast and vectorized.

**Unary** (one array in):
```python
np.sqrt(a)   np.exp(a)   np.log(a)   np.abs(a)
np.sin(a)    np.cos(a)
np.ceil(a)   np.floor(a) np.round(a)
np.isnan(a)  np.isinf(a)
```

**Binary** (two arrays in):
```python
np.add(a, b)        np.subtract(a, b)
np.multiply(a, b)   np.divide(a, b)
np.maximum(a, b)    np.minimum(a, b)   # element-wise max/min
np.power(a, b)      np.mod(a, b)
```

**Aggregations** (reduce an array):
```python
a.sum()     a.mean()    a.std()    a.var()
a.min()     a.max()     a.argmin() a.argmax()   # arg* = index of min/max
a.cumsum()  a.cumprod()

a.sum(axis=0)    # down columns (collapse rows)
a.sum(axis=1)    # across rows (collapse columns)

np.nanmean(a)    # aggregation ignoring NaN
```

> `axis=0` → operate along rows (per column). `axis=1` → along columns (per row).

---

## 9. Conditional Logic on Arrays

### The problem with plain if/else
A Python `if` can't test a whole array at once. List comprehension works but is slow:
```python
result = [x if c else y for x, y, c in zip(xarr, yarr, cond)]
```

### np.where — vectorized if/else
```python
np.where(cond, x, y)          # where cond True → x, else → y
np.where(arr > 0, 1, -1)      # 1 if positive, else -1
np.where(arr > 0, arr, 0)     # keep positives, zero the rest (ReLU)
np.where(cond)                # with one arg → indices where True
```

Nested conditions:
```python
np.where(arr < 0, "neg",
    np.where(arr == 0, "zero", "pos"))
```

---

## 10. Sorting & np.unique

### Sorting
```python
arr.sort()            # IN-PLACE sort (modifies arr)
np.sort(arr)          # returns a SORTED COPY (arr unchanged)
np.sort(arr)[::-1]    # descending
arr.sort(axis=0)      # sort each column
arr.sort(axis=1)      # sort each row
np.argsort(arr)       # indices that would sort the array
```

### Unique & set logic
```python
np.unique(arr)                        # sorted unique values
np.unique(arr, return_counts=True)    # values + how often each appears
np.unique(arr, return_index=True)     # values + first-occurrence index

np.in1d(arr, [2, 3, 6])               # bool: is each element in the set?
np.intersect1d(a, b)                  # common values
np.union1d(a, b)                      # all unique values from both
np.setdiff1d(a, b)                    # in a but not b
```

---

## Quick Mental Model

| Concept | Returns view or copy? |
|---|---|
| `a[2:5]` (basic slice) | **view** |
| `a[a > 0]` (boolean) | copy |
| `a[[0, 2, 4]]` (fancy) | copy |
| `a.T`, `a.transpose()` | view |
| `a.copy()` | copy |

When in doubt and you need to keep the original safe → `.copy()`.
