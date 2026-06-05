# pandas Cheat Sheet

> Convention: `import pandas as pd`, `import numpy as np`

---

## 1. Creating Series and DataFrame

### Series — a 1D labeled array
```python
pd.Series([10, 20, 30])                      # default integer index 0,1,2
pd.Series([10, 20, 30], index=['a','b','c']) # custom index
pd.Series({'a': 1, 'b': 2, 'c': 3})          # from a dict (keys become index)

s.values        # underlying numpy array
s.index         # the index labels
s.name = 'pop'  # name the Series
```

### DataFrame — a 2D labeled table
```python
# From a dict of columns (most common)
pd.DataFrame({
    'name': ['Amy', 'Bob', 'Cal'],
    'age':  [25, 30, 35],
})

# From a list of row-dicts
pd.DataFrame([{'a': 1, 'b': 2}, {'a': 3, 'b': 4}])

# From a 2D array with explicit labels
pd.DataFrame(np.arange(6).reshape(2, 3),
             index=['r1', 'r2'],
             columns=['c1', 'c2', 'c3'])

df.index        # row labels
df.columns      # column labels
df.values       # 2D numpy array
df.dtypes       # type of each column
df.head()  df.tail()  df.info()  df.describe()
```

---

## 2. Indexing, Selection & Filtering

### Series
```python
s['a']            # by label
s[0]              # by position (works, but see pitfall below)
s[['a', 'c']]     # multiple labels → Series
s['a':'c']        # label slice — END INCLUSIVE
s[0:2]            # position slice — end exclusive
s[s > 20]         # boolean filter
```

### DataFrame — column selection
```python
df['age']         # one column → Series
df[['age','name']]# multiple columns → DataFrame
df.age            # attribute access (only if name is a valid identifier)
```

### `.loc` (labels) vs `.iloc` (positions) — the core of selection
```python
df.loc['r1']            # row by label
df.loc['r1', 'age']     # row label, column label
df.loc[:, 'age']        # all rows, one column
df.loc['r1':'r3']       # label slice — END INCLUSIVE
df.loc[df['age'] > 25, ['name', 'age']]   # boolean rows + chosen columns

df.iloc[0]              # row by position
df.iloc[0, 1]           # row 0, column 1
df.iloc[0:2]            # position slice — end EXCLUSIVE
df.iloc[[0, 2], [0, 1]] # fancy positional selection
```

### Integer indexing pitfalls
The danger: when your index is **already integers**, `s[0]` is ambiguous —
does `0` mean the *label* `0` or the *position* `0`?

```python
s = pd.Series([10, 20, 30], index=[2, 0, 1])
s[0]        # → 20  (LABEL 0, not position 0!) — confusing
```

**Rule of thumb: always be explicit.**
```python
s.loc[0]    # definitely the LABEL 0
s.iloc[0]   # definitely the POSITION 0
```

Also remember the slice asymmetry:
- **label** slices (`loc`, or `s['a':'c']`) are **end-inclusive**
- **position** slices (`iloc`, or `s[0:2]`) are **end-exclusive**

### Filtering
```python
df[df['age'] > 25]                       # rows meeting a condition
df[(df['age'] > 25) & (df['name'] != 'Bob')]   # combine with & | ~
df[df['name'].isin(['Amy', 'Cal'])]      # membership
df[df['age'].between(25, 30)]            # range
df.query('age > 25 and name != "Bob"')   # string-expression alternative
```

---

## 3. Transposing a DataFrame

```python
df.T          # swap rows and columns
```
Columns become the index and vice versa. Handy for reshaping small tables or
viewing wide data. Note: if columns have mixed dtypes, the transpose becomes
`object` dtype (everything boxed together).

---

## 4. Arithmetic on DataFrames + Filling NA

Arithmetic **aligns on labels**. Labels present in one object but not the other
produce `NaN` in the result.

```python
df1 + df2        # aligns on both index AND columns; misaligned → NaN
```

### Arithmetic methods (let you control the fill)
```python
df1.add(df2, fill_value=0)    # missing label treated as 0 before adding
df1.sub(df2, fill_value=0)    # subtract
df1.mul(df2, fill_value=1)
df1.div(df2, fill_value=1)
```
`fill_value` substitutes for labels missing in **one** frame. (If a label is
missing in *both*, it stays `NaN`.)

Method ↔ operator map: `add` `+`, `sub` `-`, `mul` `*`, `div` `/`,
`floordiv` `//`, `pow` `**`. Reversed versions exist too: `rsub`, `rdiv`, …

---

## 5. Operations Between DataFrame and Series

Default broadcasting is **row-wise** — the Series index matches the DataFrame
**columns**, and the operation applies down each row:

```python
df - df.iloc[0]      # subtract first row from every row (matches on columns)
```

To broadcast **column-wise** (match the Series on the DataFrame's index, apply
across columns), use an explicit method with `axis='index'`:

```python
df.sub(df['col'], axis='index')   # subtract a column from all columns
df.mul(series, axis=0)            # axis=0 / 'index' = match on rows
```

> Default `axis='columns'` (broadcast over rows). Use `axis='index'` (=`axis=0`)
> to broadcast over columns.

---

## 6. Sorting and Ranking

### Sorting
```python
df.sort_index()                      # by row labels
df.sort_index(axis=1)                # by column labels
df.sort_index(ascending=False)

df.sort_values('age')                # by one column
df.sort_values(['age', 'name'])      # by multiple (tie-break order)
df.sort_values('age', ascending=False)
df.sort_values('age', na_position='first')   # where NaNs go

s.sort_values()                      # Series — NaNs sorted to the end
```

### Ranking — assigns ranks 1..n
```python
s.rank()                       # average rank for ties (default)
s.rank(method='min')           # ties get the lowest rank in the group
s.rank(method='first')         # ties broken by order of appearance
s.rank(ascending=False)        # rank largest = 1
df.rank(axis='columns')        # rank across columns within each row
```

---

## 7. Summary & Descriptive Statistics (Reductions)

```python
df.sum()        df.sum(axis='columns')   # axis=0 down rows (per col); axis=1 across cols (per row)
df.mean()       df.median()
df.min()        df.max()
df.std()        df.var()
df.count()      # number of NON-NA values
df.idxmax()     df.idxmin()              # LABEL of the max/min
df.cumsum()     df.cumprod()             # cumulative
df.describe()   # count, mean, std, min, quartiles, max in one shot

s.describe()    # for non-numeric: count, unique, top, freq
```

**NA handling:** reductions **skip NaN by default** (`skipna=True`). Set
`skipna=False` to let NaN propagate.

```python
df.mean(skipna=False)
```

---

## 8. Correlation and Covariance

```python
df['a'].corr(df['b'])     # correlation between two Series
df['a'].cov(df['b'])      # covariance between two Series

df.corr()                 # full correlation matrix (all numeric columns)
df.cov()                  # full covariance matrix

df.corr(method='spearman')  # 'pearson' (default), 'kendall', 'spearman'

df.corrwith(df['a'])      # correlation of every column WITH one Series
df.corrwith(other_df)     # pairwise with another DataFrame's columns
```
NA pairs are excluded automatically. `corr` ranges -1 to 1; `cov` is unscaled.

---

## 9. Unique Values, Value Counts & Membership

```python
s.unique()                # array of distinct values (order of appearance)
s.nunique()               # COUNT of distinct values
s.value_counts()          # frequency of each value, sorted descending
s.value_counts(normalize=True)   # proportions instead of counts
s.value_counts(dropna=False)     # include NaN in the tally

s.isin(['a', 'b'])        # boolean mask: is each value in the set?
df[df['col'].isin(['x', 'y'])]   # common filtering use

s.duplicated()            # boolean: is this a repeat of an earlier value?
df.drop_duplicates()      # drop duplicate rows
df.drop_duplicates('col') # dedupe based on one column

# Apply value_counts across every column of a DataFrame:
df.apply(pd.value_counts).fillna(0)
```

---

## Quick Reminders

| Gotcha | Remember |
|---|---|
| `s[0]` with integer index | ambiguous — use `.loc` (label) or `.iloc` (position) |
| `.loc` slice | end-**inclusive** |
| `.iloc` slice | end-**exclusive** |
| `df + df2` | aligns on labels; misaligned → `NaN` (use `.add(fill_value=)`) |
| `df - series` | broadcasts row-wise (matches columns) by default |
| reductions (`sum`, `mean`) | skip `NaN` unless `skipna=False` |
| `df['col']` vs `df[['col']]` | Series vs single-column DataFrame |
