# pandas Cheat Sheet — Data Cleaning & Preparation

> Convention: `import pandas as pd`, `import numpy as np`

---

## 1. Handling Missing Values

In pandas, missing data shows up as `NaN` (float), `None`, or `NaT` (for
datetimes). The methods below treat them uniformly.

### Identifying
```python
df.isna()           # boolean DataFrame: True where missing
df.notna()          # the inverse
df.isnull()         # alias of isna()  (same thing)

df.isna().sum()             # count of NaNs per column
df.isna().sum().sum()       # total NaNs in the frame
df.isna().any(axis=1)       # which ROWS have at least one NaN
df['col'].isna().mean()     # fraction missing in a column
```

### Filtering out
```python
s.dropna()                  # Series: drop missing entries

df.dropna()                 # drop any ROW containing a NaN
df.dropna(how='all')        # drop rows where ALL values are NaN
df.dropna(axis=1)           # drop COLUMNS containing a NaN
df.dropna(thresh=2)         # keep rows with at least 2 non-NaN values
df.dropna(subset=['age'])   # only consider these columns when dropping
```

### Filling in
```python
df.fillna(0)                       # replace all NaN with 0
df.fillna({'age': 0, 'city': '?'}) # per-column fill values
df.fillna(df.mean(numeric_only=True))  # impute with column means

df.fillna(method='ffill')   # forward-fill (carry last value forward)
df.fillna(method='bfill')   # back-fill
df['col'].fillna(df['col'].median())   # common imputation

df.fillna(0, inplace=True)  # modify in place (or just reassign)
```
> Modern syntax: `df.ffill()` / `df.bfill()` replace the `method=` form.

---

## 2. Data Transformation

### Removing duplicates
```python
df.duplicated()                 # boolean: is this row a repeat of an earlier one?
df.drop_duplicates()            # drop duplicate rows (keep first)
df.drop_duplicates('col')       # judge duplicates by one column only
df.drop_duplicates(['a','b'])   # by a subset of columns
df.drop_duplicates(keep='last') # keep the last occurrence instead
df.drop_duplicates(keep=False)  # drop ALL rows that have any duplicate
```

### Transforming with functions or mappings
```python
# map on a Series — element-wise lookup or function
df['grade'] = df['score'].map({90: 'A', 80: 'B'})
df['food_lower'] = df['food'].map(str.lower)

# add a derived column via a mapping dict
meat_to_animal = {'bacon': 'pig', 'pulled pork': 'pig', 'salmon': 'fish'}
df['animal'] = df['food'].str.lower().map(meat_to_animal)

# apply for row/column-level logic
df['total'] = df.apply(lambda row: row['a'] + row['b'], axis=1)
```

### Replacing values
```python
s.replace(-999, np.nan)             # single value → NaN
s.replace([-999, -1000], np.nan)    # list of values → NaN
s.replace([-999, -1000], [np.nan, 0])   # element-wise replacement
s.replace({-999: np.nan, -1000: 0}) # dict form
df.replace('?', np.nan)             # works on whole DataFrame
```
> `replace` is for **values**; don't confuse with `rename` (for **labels**).

### Renaming axis indexes
```python
df.rename(index={'old': 'new'}, columns={'A': 'alpha'})
df.rename(columns=str.upper)        # transform all column names
df.rename(index=str.title)

df.index = df.index.map(str.upper)  # transform index in place via map
df.columns = ['a', 'b', 'c']        # wholesale reassign

df.rename(columns={'A': 'alpha'}, inplace=True)
```

### Discretization & binning
Split continuous data into discrete buckets.
```python
ages = [20, 22, 25, 27, 33, 45, 61]
bins = [18, 25, 35, 60, 100]
cats = pd.cut(ages, bins)           # → intervals (18,25], (25,35], ...
cats.codes                          # integer code for each value's bin
cats.categories                     # the interval index
pd.value_counts(cats)               # count per bin

pd.cut(ages, bins, right=False)     # make intervals left-closed [18,25)
pd.cut(ages, bins, labels=['Youth','YA','Adult','Senior'])  # name the bins
pd.cut(data, 4)                     # 4 equal-WIDTH bins (pass an int)

pd.qcut(data, 4)                    # 4 equal-SIZE bins (by quantiles)
pd.qcut(data, [0, .1, .5, .9, 1.])  # custom quantile edges
```
> `cut` = equal-width ranges; `qcut` = equal-frequency (roughly equal counts).

### Detecting & filtering outliers
```python
df.describe()                       # spot extreme min/max first

col = df['data']
col[col.abs() > 3]                  # values beyond ±3 (e.g. on standardized data)

# any row with a value exceeding 3 in absolute terms
df[(df.abs() > 3).any(axis=1)]

# cap / clip extremes instead of dropping
df['data'].clip(lower=-3, upper=3)

# IQR method
Q1, Q3 = df['x'].quantile([0.25, 0.75])
IQR = Q3 - Q1
mask = (df['x'] < Q1 - 1.5*IQR) | (df['x'] > Q3 + 1.5*IQR)
df[~mask]                           # rows without outliers
```

### Computing dummy / indicator variables
One-hot encode categorical data into 0/1 columns.
```python
pd.get_dummies(df['key'])                  # one column per category
pd.get_dummies(df['key'], prefix='key')    # prefix the new column names
pd.get_dummies(df, columns=['c1', 'c2'])   # encode chosen columns in a frame
pd.get_dummies(df['key'], drop_first=True) # drop one level (avoid collinearity)
pd.get_dummies(df['key'], dummy_na=True)   # add a column flagging NaN
```
> In pandas 2.0+ the output dtype is boolean; older versions give uint8.

### String manipulation (the `.str` accessor)
Vectorized string methods on a Series — operate on every element, skip NaN.
```python
s.str.lower()      s.str.upper()    s.str.title()
s.str.strip()      s.str.lstrip()   s.str.rstrip()
s.str.len()
s.str.contains('foo')               # boolean mask
s.str.startswith('a')  s.str.endswith('z')
s.str.replace('-', '_')             # replace within strings
s.str.replace(r'\d+', '', regex=True)   # regex replace
s.str.split(',')                    # → lists
s.str.split(',', expand=True)       # → separate columns
s.str.get(0)                        # element from each split list
s.str.cat(sep=', ')                 # join all into one string
s.str[:5]                           # slice each string
s.str.findall(r'\d+')               # regex find all matches
s.str.extract(r'(\d+)-(\d+)')       # capture groups → columns
```
Common cleaning combo:
```python
df['name'] = df['name'].str.strip().str.title()
```

---

## Quick Reminders

| Task | Tool |
|---|---|
| count missing per column | `df.isna().sum()` |
| drop rows with any NaN | `df.dropna()` |
| impute | `df.fillna(...)` / `ffill` / `bfill` |
| remove duplicate rows | `df.drop_duplicates()` |
| swap **values** | `replace` |
| swap **labels** | `rename` |
| equal-width bins | `pd.cut` |
| equal-frequency bins | `pd.qcut` |
| one-hot encode | `pd.get_dummies` |
| vectorized strings | `s.str.*` |

> `replace` ↔ values, `rename` ↔ labels, `map` ↔ element-wise transform — keep
> these three straight and most cleaning falls into place.
