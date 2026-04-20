---
name: pandas-pro
description: Performs pandas DataFrame operations for data analysis, manipulation, and transformation. Use when working with pandas DataFrames, data cleaning, aggregation, merging, or time series analysis. Invoke for data manipulation tasks such as joining DataFrames on multiple keys, pivoting tables, resampling time series, handling NaN values with interpolation or forward-fill, groupby aggregations, type conversion, or performance optimization of large datasets.
---

# Pandas Pro

Expert pandas developer specializing in efficient data manipulation, analysis, and transformation workflows with production-grade performance patterns.

## Core Workflow

1. **Assess data structure** — Examine dtypes, memory usage, missing values, data quality:
   ```python
   print(df.dtypes)
   print(df.memory_usage(deep=True).sum() / 1e6, "MB")
   print(df.isna().sum())
   print(df.describe(include="all"))
   ```
2. **Design transformation** — Plan vectorized operations, avoid loops, identify indexing strategy
3. **Implement efficiently** — Use vectorized methods, method chaining, proper indexing
4. **Validate results** — Check dtypes, shapes, null counts, and row counts
5. **Optimize** — Profile memory, apply categorical types, use chunking if needed

## Code Patterns

### Vectorized Operations

```python
# ❌ AVOID: row-by-row iteration
for i, row in df.iterrows():
    df.at[i, 'tax'] = row['price'] * 0.2

# ✅ USE: vectorized assignment
df['tax'] = df['price'] * 0.2
```

### GroupBy Aggregation

```python
summary = (
    df.groupby(['region', 'category'], observed=True)
    .agg(
        total_sales=('revenue', 'sum'),
        avg_price=('price', 'mean'),
        order_count=('order_id', 'nunique'),
    )
    .reset_index()
)
```

### Merge with Validation

```python
merged = pd.merge(
    left_df, right_df,
    on=['customer_id', 'date'],
    how='left',
    validate='m:1',
    indicator=True,
)
unmatched = merged[merged['_merge'] != 'both']
merged.drop(columns=['_merge'], inplace=True)
```

### Missing Value Handling

```python
# Forward-fill then interpolate numeric gaps
df['price'] = df['price'].ffill().interpolate(method='linear')

# Fill categoricals with mode, numerics with median
for col in df.select_dtypes(include='object'):
    df[col] = df[col].fillna(df[col].mode()[0])
for col in df.select_dtypes(include='number'):
    df[col] = df[col].fillna(df[col].median())
```

### Time Series Resampling

```python
daily = (
    df.set_index('timestamp')
    .resample('D')
    .agg({'revenue': 'sum', 'sessions': 'count'})
    .fillna(0)
)
```

### Memory Optimization

```python
df['category'] = df['category'].astype('category')
df['count'] = pd.to_numeric(df['count'], downcast='integer')
df['score'] = pd.to_numeric(df['score'], downcast='float')
```

## Constraints

**MUST DO:**
- Use vectorized operations instead of loops
- Set appropriate dtypes (categorical for low-cardinality strings)
- Handle missing values explicitly (don't silently drop)
- Use `.copy()` when modifying subsets to avoid SettingWithCopyWarning
- Validate data quality before and after transformations

**MUST NOT:**
- Iterate with `.iterrows()` unless absolutely necessary
- Use chained indexing (`df['A']['B']`) — use `.loc[]` or `.iloc[]`
- Load entire large datasets without chunking
- Use deprecated methods (`.append()` — use `pd.concat()`)
