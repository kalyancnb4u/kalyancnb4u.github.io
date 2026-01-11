---
title: "Python Mastery - Part 8: Advanced Topics & Specialized Domains"
date: 2024-12-09 00:00:00 +0530
categories: [Python, Python Mastery]
tags: [Python, Advanced, Data Science, Web Scraping, CLI, System Programming, Numpy, Pandas, Automation]
---
## Table of Contents

- [Introduction](#introduction)
- [8.1 Data Science with Python](#81-data-science-with-python)
  - [NumPy Fundamentals](#numpy-fundamentals)
  - [Pandas for Data Analysis](#pandas-for-data-analysis)
  - [Data Visualization](#data-visualization)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [8.2 Web Scraping &amp; Automation](#82-web-scraping--automation)
  - [Web Scraping Basics](#web-scraping-basics)
  - [Advanced Scraping](#advanced-scraping)
  - [Browser Automation](#browser-automation)
  - [Ethical Considerations](#ethical-considerations)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
- [8.3 Command-Line Applications](#83-command-line-applications)
  - [Building CLI Tools](#building-cli-tools)
  - [Rich Terminal UIs](#rich-terminal-uis)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
- [8.4 System Programming](#84-system-programming)
- [8.5 Task Automation](#85-task-automation)

---

# Complete Python Mastery Part 8: Advanced Topics & Specialized Domains

## Introduction

Welcome to **Part 8** of the Complete Python Mastery series. This part covers specialized Python applications across different domains.

**What You'll Learn in Part 8:**

This post covers advanced and specialized Python topics:

- **Data science**: NumPy, Pandas, data visualization
- **Web scraping**: BeautifulSoup, Scrapy, Selenium, Playwright
- **CLI applications**: argparse, click, typer, rich
- **System programming**: OS interaction, process management, signals
- **Task automation**: Scheduling, background jobs, Celery

**Why This Matters:**

These specialized skills enable:

- Data analysis and processing
- Web data extraction
- Professional CLI tools
- System automation
- Distributed task processing

**Prerequisites:**

- Parts 1-7 of this series
- Python fundamentals
- Basic understanding of HTML/CSS (for scraping)

Let's explore specialized Python domains! 🚀

---

## 8.1 Data Science with Python

**SDLC Phase:** Development, Analysis

**Relevant For:**

- [X] Requirements gathering (data analysis)
- [X] System design (data pipelines)
- [X] Development (data processing)
- [X] Testing (data validation)
- [ ] Deployment
- [X] Maintenance (data monitoring)

### NumPy Fundamentals

```python
"""
NUMPY: Numerical computing library

Benefits:
✅ Fast array operations (C-based)
✅ Vectorization (no explicit loops)
✅ Broadcasting (automatic array expansion)
✅ Memory efficient
"""

import numpy as np

# Create arrays

# From list
arr = np.array([1, 2, 3, 4, 5])
print(arr)  # [1 2 3 4 5]

# 2D array
matrix = np.array([[1, 2, 3], [4, 5, 6]])
print(matrix)
# [[1 2 3]
#  [4 5 6]]

# Array creation functions
zeros = np.zeros((3, 4))  # 3x4 array of zeros
ones = np.ones((2, 3))    # 2x3 array of ones
empty = np.empty((2, 2))  # Uninitialized array
arange = np.arange(0, 10, 2)  # [0 2 4 6 8]
linspace = np.linspace(0, 1, 5)  # 5 evenly spaced values

# Random arrays
random_arr = np.random.rand(3, 3)  # Uniform [0, 1)
normal = np.random.randn(1000)     # Standard normal
randint = np.random.randint(0, 100, size=10)  # Random integers

# Array operations (vectorized - no loops!)

# ❌ BAD: Python loops (slow)
result = []
for i in range(len(arr)):
    result.append(arr[i] * 2)

# ✅ GOOD: NumPy vectorization (fast)
result = arr * 2  # Multiplies all elements

# Element-wise operations
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

print(a + b)   # [5 7 9]
print(a - b)   # [-3 -3 -3]
print(a * b)   # [4 10 18]
print(a / b)   # [0.25 0.4 0.5]
print(a ** 2)  # [1 4 9]

# Universal functions (ufuncs)
arr = np.array([1, 4, 9, 16])
print(np.sqrt(arr))     # [1. 2. 3. 4.]
print(np.exp(arr))      # Exponential
print(np.log(arr))      # Natural log
print(np.sin(arr))      # Sine

# Array indexing and slicing
arr = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])

print(arr[5])       # 5
print(arr[2:7])     # [2 3 4 5 6]
print(arr[::2])     # [0 2 4 6 8] - every other element

# 2D indexing
matrix = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

print(matrix[1, 2])     # 6 (row 1, col 2)
print(matrix[0, :])     # [1 2 3] (first row)
print(matrix[:, 1])     # [2 5 8] (second column)
print(matrix[0:2, 1:3]) # [[2 3] [5 6]]

# Boolean indexing
arr = np.array([1, 2, 3, 4, 5])
mask = arr > 3          # [False False False True True]
print(arr[mask])        # [4 5]

# Fancy indexing
arr = np.array([10, 20, 30, 40, 50])
indices = [1, 3, 4]
print(arr[indices])     # [20 40 50]

# Broadcasting (automatic array expansion)
a = np.array([[1, 2, 3],
              [4, 5, 6]])  # Shape: (2, 3)
b = np.array([10, 20, 30])  # Shape: (3,)

# b is broadcast to match a's shape
result = a + b
# [[11 22 33]
#  [14 25 36]]

# Array reshaping
arr = np.arange(12)     # [0 1 2 ... 11]
matrix = arr.reshape(3, 4)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

# Transpose
transposed = matrix.T
# [[ 0  4  8]
#  [ 1  5  9]
#  [ 2  6 10]
#  [ 3  7 11]]

# Aggregations
arr = np.array([1, 2, 3, 4, 5])

print(np.sum(arr))      # 15
print(np.mean(arr))     # 3.0
print(np.std(arr))      # 1.414... (standard deviation)
print(np.min(arr))      # 1
print(np.max(arr))      # 5
print(np.argmin(arr))   # 0 (index of min)
print(np.argmax(arr))   # 4 (index of max)

# Axis-wise operations
matrix = np.array([[1, 2, 3],
                   [4, 5, 6]])

print(np.sum(matrix))           # 21 (all elements)
print(np.sum(matrix, axis=0))   # [5 7 9] (sum columns)
print(np.sum(matrix, axis=1))   # [6 15] (sum rows)

# Linear algebra
a = np.array([[1, 2], [3, 4]])
b = np.array([[5, 6], [7, 8]])

# Matrix multiplication
print(np.dot(a, b))     # [[19 22] [43 50]]
# Or use @ operator
print(a @ b)            # [[19 22] [43 50]]

# Matrix inverse
inv = np.linalg.inv(a)

# Eigenvalues and eigenvectors
eigenvalues, eigenvectors = np.linalg.eig(a)

# Performance comparison
import timeit

# Python list
python_list = list(range(1000000))
python_time = timeit.timeit(
    lambda: [x * 2 for x in python_list],
    number=10
)

# NumPy array
numpy_array = np.arange(1000000)
numpy_time = timeit.timeit(
    lambda: numpy_array * 2,
    number=10
)

print(f"Python list: {python_time:.4f}s")
print(f"NumPy array: {numpy_time:.4f}s")
print(f"Speedup: {python_time / numpy_time:.1f}x")
# NumPy is typically 10-100x faster!
```

### Pandas for Data Analysis

```python
"""
PANDAS: Data manipulation and analysis

Key structures:
- Series: 1D labeled array
- DataFrame: 2D labeled data structure
"""

import pandas as pd

# Create Series
s = pd.Series([1, 2, 3, 4, 5], index=['a', 'b', 'c', 'd', 'e'])
print(s)
# a    1
# b    2
# c    3
# d    4
# e    5

# Create DataFrame

# From dictionary
data = {
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'age': [25, 30, 35, 28],
    'salary': [50000, 60000, 75000, 55000],
    'department': ['IT', 'HR', 'IT', 'Sales']
}
df = pd.DataFrame(data)
print(df)
#       name  age  salary department
# 0    Alice   25   50000         IT
# 1      Bob   30   60000         HR
# 2  Charlie   35   75000         IT
# 3    David   28   55000      Sales

# Read from CSV
df = pd.read_csv('data.csv')

# Read from Excel
df = pd.read_excel('data.xlsx')

# Read from JSON
df = pd.read_json('data.json')

# Read from SQL
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql_query('SELECT * FROM users', conn)

# Basic DataFrame operations

# View first/last rows
print(df.head())      # First 5 rows
print(df.tail(3))     # Last 3 rows

# Info about DataFrame
print(df.info())      # Column types, non-null counts
print(df.describe())  # Statistical summary
print(df.shape)       # (rows, columns)
print(df.columns)     # Column names
print(df.dtypes)      # Data types

# Selection

# Select column
ages = df['age']
# Or: ages = df.age

# Select multiple columns
subset = df[['name', 'salary']]

# Select rows by index
row = df.loc[0]       # By label
row = df.iloc[0]      # By position

# Select rows and columns
data = df.loc[0:2, ['name', 'age']]

# Boolean indexing
high_salary = df[df['salary'] > 60000]
it_employees = df[df['department'] == 'IT']
young_high_earners = df[(df['age'] < 30) & (df['salary'] > 50000)]

# Filtering

# Filter rows
filtered = df[df['age'] > 30]

# Filter with multiple conditions
filtered = df[(df['age'] > 25) & (df['salary'] < 70000)]

# Filter with isin
filtered = df[df['department'].isin(['IT', 'HR'])]

# Sorting
sorted_df = df.sort_values('age')
sorted_df = df.sort_values('age', ascending=False)
sorted_df = df.sort_values(['department', 'salary'])

# Adding/Modifying columns

# Add new column
df['bonus'] = df['salary'] * 0.1

# Modify existing column
df['age'] = df['age'] + 1

# Conditional column
df['seniority'] = df['age'].apply(
    lambda x: 'Senior' if x > 30 else 'Junior'
)

# Delete column
df = df.drop('bonus', axis=1)
# Or: del df['bonus']

# Handling missing data

# Check for missing values
print(df.isnull())          # Boolean DataFrame
print(df.isnull().sum())    # Count per column

# Drop rows with missing values
df_clean = df.dropna()

# Drop columns with missing values
df_clean = df.dropna(axis=1)

# Fill missing values
df_filled = df.fillna(0)
df_filled = df.fillna(df.mean())  # Fill with mean
df_filled = df.fillna(method='ffill')  # Forward fill

# Grouping and aggregation

# Group by department
grouped = df.groupby('department')

# Aggregate functions
print(grouped['salary'].mean())
print(grouped['salary'].sum())
print(grouped.size())  # Count per group

# Multiple aggregations
agg_df = df.groupby('department').agg({
    'salary': ['mean', 'max', 'min'],
    'age': 'mean'
})

# Merging DataFrames

# Create second DataFrame
df2 = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Eve'],
    'city': ['NYC', 'LA', 'SF']
})

# Merge (SQL-like join)
merged = pd.merge(df, df2, on='name', how='left')
# how: 'left', 'right', 'inner', 'outer'

# Concatenate
combined = pd.concat([df1, df2], axis=0)  # Vertical
combined = pd.concat([df1, df2], axis=1)  # Horizontal

# Pivot tables
pivot = df.pivot_table(
    values='salary',
    index='department',
    columns='age',
    aggfunc='mean'
)

# Time series

# Create date range
dates = pd.date_range('2024-01-01', periods=100, freq='D')

# DataFrame with dates
ts_df = pd.DataFrame({
    'date': dates,
    'value': np.random.randn(100)
})

# Set date as index
ts_df = ts_df.set_index('date')

# Resample (aggregate by time period)
monthly = ts_df.resample('M').mean()

# Rolling window
rolling_mean = ts_df['value'].rolling(window=7).mean()

# Apply functions

# Apply to column
df['age_squared'] = df['age'].apply(lambda x: x ** 2)

# Apply to row
df['total_comp'] = df.apply(
    lambda row: row['salary'] + row['bonus'],
    axis=1
)

# Apply to DataFrame
df_normalized = df[['age', 'salary']].apply(
    lambda x: (x - x.mean()) / x.std()
)

# Export data

# To CSV
df.to_csv('output.csv', index=False)

# To Excel
df.to_excel('output.xlsx', index=False)

# To JSON
df.to_json('output.json', orient='records')

# To SQL
df.to_sql('employees', conn, if_exists='replace')

# Real-world example: Data analysis pipeline
def analyze_sales_data(filename):
    """Complete data analysis pipeline"""
    # 1. Load data
    df = pd.read_csv(filename)
  
    # 2. Clean data
    df = df.dropna(subset=['sale_amount'])
    df['sale_date'] = pd.to_datetime(df['sale_date'])
  
    # 3. Feature engineering
    df['month'] = df['sale_date'].dt.month
    df['year'] = df['sale_date'].dt.year
    df['day_of_week'] = df['sale_date'].dt.day_name()
  
    # 4. Analysis
    monthly_sales = df.groupby(['year', 'month'])['sale_amount'].sum()
    top_products = df.groupby('product')['sale_amount'].sum().nlargest(10)
  
    # 5. Statistics
    stats = df['sale_amount'].describe()
  
    return {
        'monthly_sales': monthly_sales,
        'top_products': top_products,
        'statistics': stats
    }
```

### Data Visualization

```python
"""
DATA VISUALIZATION: matplotlib, seaborn, plotly

Tools:
- matplotlib: Basic plotting
- seaborn: Statistical visualization
- plotly: Interactive plots
"""

import matplotlib.pyplot as plt
import seaborn as sns

# Matplotlib basics

# Line plot
x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y, label='sin(x)')
plt.plot(x, np.cos(x), label='cos(x)')
plt.xlabel('x')
plt.ylabel('y')
plt.title('Trigonometric Functions')
plt.legend()
plt.grid(True)
plt.savefig('plot.png', dpi=300)
plt.show()

# Scatter plot
plt.figure(figsize=(8, 6))
plt.scatter(df['age'], df['salary'], alpha=0.6)
plt.xlabel('Age')
plt.ylabel('Salary')
plt.title('Age vs Salary')
plt.show()

# Bar plot
departments = df['department'].value_counts()
plt.figure(figsize=(8, 6))
plt.bar(departments.index, departments.values)
plt.xlabel('Department')
plt.ylabel('Count')
plt.title('Employees per Department')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# Histogram
plt.figure(figsize=(8, 6))
plt.hist(df['salary'], bins=20, edgecolor='black')
plt.xlabel('Salary')
plt.ylabel('Frequency')
plt.title('Salary Distribution')
plt.show()

# Seaborn (statistical plots)

# Set style
sns.set_style('whitegrid')

# Distribution plot
plt.figure(figsize=(10, 6))
sns.histplot(df['salary'], kde=True)  # With density curve
plt.title('Salary Distribution')
plt.show()

# Box plot
plt.figure(figsize=(10, 6))
sns.boxplot(data=df, x='department', y='salary')
plt.title('Salary by Department')
plt.show()

# Violin plot (combines box plot and density)
plt.figure(figsize=(10, 6))
sns.violinplot(data=df, x='department', y='salary')
plt.title('Salary Distribution by Department')
plt.show()

# Heatmap (correlation matrix)
numeric_df = df[['age', 'salary']]
correlation = numeric_df.corr()

plt.figure(figsize=(8, 6))
sns.heatmap(correlation, annot=True, cmap='coolwarm', center=0)
plt.title('Correlation Matrix')
plt.show()

# Pair plot (all pairwise relationships)
sns.pairplot(df[['age', 'salary', 'department']], hue='department')
plt.show()

# Plotly (interactive)
import plotly.express as px

# Interactive scatter
fig = px.scatter(df, x='age', y='salary', color='department',
                 hover_data=['name'], title='Interactive Salary Plot')
fig.show()

# Interactive line chart
fig = px.line(ts_df, x=ts_df.index, y='value',
              title='Time Series')
fig.show()

# 3D scatter
fig = px.scatter_3d(df, x='age', y='salary', z='bonus',
                    color='department')
fig.show()

# Dashboard example
from plotly.subplots import make_subplots
import plotly.graph_objects as go

fig = make_subplots(
    rows=2, cols=2,
    subplot_titles=('Salary Distribution', 'Age vs Salary',
                    'Department Counts', 'Salary by Department')
)

# Histogram
fig.add_trace(
    go.Histogram(x=df['salary'], name='Salary'),
    row=1, col=1
)

# Scatter
fig.add_trace(
    go.Scatter(x=df['age'], y=df['salary'], mode='markers'),
    row=1, col=2
)

# Bar chart
dept_counts = df['department'].value_counts()
fig.add_trace(
    go.Bar(x=dept_counts.index, y=dept_counts.values),
    row=2, col=1
)

# Box plot
fig.add_trace(
    go.Box(y=df['salary'], x=df['department']),
    row=2, col=2
)

fig.update_layout(height=800, showlegend=False, title_text="Sales Dashboard")
fig.show()
```

### Frequently Asked Questions

**Q1: When should I use NumPy vs Pandas?**

**A:**

```python
"""
NUMPY vs PANDAS DECISION

Use NumPy when:
✅ Working with numerical arrays
✅ Need maximum performance
✅ Doing mathematical operations
✅ Working with matrices/linear algebra
✅ Data is homogeneous (same type)

Use Pandas when:
✅ Working with tabular data
✅ Need labeled columns/rows
✅ Handling missing data
✅ Doing data analysis/aggregation
✅ Data is heterogeneous (mixed types)
✅ Need time series functionality
"""

# NumPy: Numerical computations
import numpy as np

# Matrix operations
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
C = A @ B  # Matrix multiplication

# Fast numerical operations
data = np.random.randn(1000000)
mean = np.mean(data)  # Very fast

# Pandas: Data analysis
import pandas as pd

# Tabular data with labels
df = pd.DataFrame({
    'name': ['Alice', 'Bob'],
    'age': [25, 30],
    'salary': [50000, 60000]
})

# Group by and aggregate
avg_salary = df.groupby('department')['salary'].mean()

# Handle missing data
df_clean = df.dropna()

# Time series
df['date'] = pd.to_datetime(df['date'])
monthly = df.resample('M', on='date').mean()

# Use both together
# Pandas internally uses NumPy arrays
df['squared'] = np.square(df['age'])  # NumPy function on Pandas
values = df['age'].values  # Get NumPy array from Pandas

# Performance comparison:
"""
Operation            NumPy    Pandas
--------------------------------------
Math operations      Fast     Slower (but convenient)
Array indexing       Fast     Slower
Labeled access       N/A      Fast
Group by             N/A      Available
Missing data         Manual   Built-in
Mixed types          No       Yes
"""
```

**Why This Matters:** Using the right tool improves both performance and code clarity.

---

### Interview Questions

**Question 1: How would you efficiently process a 10GB CSV file in Python?**

**Difficulty:** Mid-to-Senior Level

**SDLC Relevance:** Development, Performance Optimization

**Answer:**

```python
"""
PROCESSING LARGE CSV FILES

Strategies:
1. Chunked reading (pandas)
2. Streaming with csv module
3. Parallel processing
4. Use appropriate data types
5. Filter early
"""

# 1. Chunked reading with Pandas
import pandas as pd

def process_large_csv_chunks(filename, chunk_size=100000):
    """Process CSV in chunks"""
    results = []
  
    # Read in chunks
    for chunk in pd.read_csv(filename, chunksize=chunk_size):
        # Process each chunk
        processed = process_chunk(chunk)
        results.append(processed)
  
    # Combine results
    final_result = pd.concat(results, ignore_index=True)
    return final_result

def process_chunk(chunk):
    """Process single chunk"""
    # Filter early (reduce memory)
    chunk = chunk[chunk['amount'] > 1000]
  
    # Optimize data types
    chunk['id'] = chunk['id'].astype('int32')  # Instead of int64
    chunk['category'] = chunk['category'].astype('category')  # Save memory
  
    # Compute
    chunk['total'] = chunk['amount'] * chunk['quantity']
  
    return chunk

# 2. Streaming with csv module (most memory efficient)
import csv

def process_csv_streaming(filename):
    """Stream process without loading all into memory"""
    results = []
  
    with open(filename, 'r') as f:
        reader = csv.DictReader(f)
      
        for row in reader:
            # Process one row at a time
            if float(row['amount']) > 1000:
                result = process_row(row)
                results.append(result)
  
    return results

# 3. Parallel processing
from multiprocessing import Pool
import numpy as np

def process_csv_parallel(filename, num_processes=4):
    """Process CSV in parallel"""
    # Read file line count
    with open(filename) as f:
        num_lines = sum(1 for _ in f) - 1  # Minus header
  
    # Split into chunks
    chunk_size = num_lines // num_processes
  
    # Create chunk arguments
    chunks = []
    for i in range(num_processes):
        start = i * chunk_size
        end = start + chunk_size if i < num_processes - 1 else num_lines
        chunks.append((filename, start, end))
  
    # Process in parallel
    with Pool(num_processes) as pool:
        results = pool.starmap(process_csv_chunk, chunks)
  
    # Combine
    return pd.concat(results, ignore_index=True)

def process_csv_chunk(filename, start, end):
    """Process specific rows of CSV"""
    df = pd.read_csv(filename, skiprows=range(1, start + 1), nrows=end - start)
    return process_chunk(df)

# 4. Optimize data types
def optimize_dtypes(df):
    """Reduce memory usage by optimizing data types"""
    for col in df.columns:
        col_type = df[col].dtype
      
        if col_type == 'object':
            # Check if can be categorical
            num_unique = df[col].nunique()
            num_total = len(df[col])
          
            if num_unique / num_total < 0.5:  # Less than 50% unique
                df[col] = df[col].astype('category')
      
        elif col_type == 'int64':
            # Downcast integers
            c_min = df[col].min()
            c_max = df[col].max()
          
            if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                df[col] = df[col].astype(np.int8)
            elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                df[col] = df[col].astype(np.int16)
            elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                df[col] = df[col].astype(np.int32)
      
        elif col_type == 'float64':
            # Downcast floats
            df[col] = df[col].astype(np.float32)
  
    return df

# 5. Filter early, compute late
def process_efficiently(filename):
    """Best practices combined"""
    # Read with optimized types
    df = pd.read_csv(
        filename,
        dtype={
            'id': 'int32',
            'category': 'category',
            'amount': 'float32'
        },
        parse_dates=['date'],
        usecols=['id', 'category', 'amount', 'date']  # Only needed columns
    )
  
    # Filter early (reduce data size)
    df = df[df['amount'] > 1000]
  
    # Process
    df['total'] = df['amount'] * 1.1
  
    return df

# Performance comparison (10GB CSV):
"""
Method                Memory      Time       Notes
--------------------------------------------------
Read all at once      10 GB       60s        ❌ May crash
Chunked reading       500 MB      90s        ✅ Memory safe
Streaming             50 MB       120s       ✅ Lowest memory
Parallel (4 cores)    2 GB        25s        ✅ Fastest
Optimized dtypes      3 GB        50s        ✅ Balanced
"""
```

**Key Points:**

- Use chunked reading for memory efficiency
- Stream with csv module for lowest memory
- Parallelize for speed (multi-core)
- Optimize data types to reduce memory
- Filter early to reduce data size
- Read only needed columns

---

(Part 8 continues with sections 8.2-8.5...)

## 8.2 Web Scraping & Automation

**SDLC Phase:** Development, Data Collection

**Relevant For:**

- [X] Requirements gathering (data collection)
- [ ] System design
- [X] Development (scraping implementation)
- [X] Testing (scraping validation)
- [ ] Deployment
- [X] Maintenance (scraper updates)

### Web Scraping Basics

```python
"""
WEB SCRAPING: Extract data from websites

Tools:
- requests: HTTP requests
- BeautifulSoup: HTML parsing
- lxml: Fast parser
- Scrapy: Full framework
- Selenium: Browser automation
"""

import requests
from bs4 import BeautifulSoup

# Basic scraping example
def scrape_quotes():
    """Scrape quotes from website"""
    url = 'http://quotes.toscrape.com/'
  
    # Send GET request
    response = requests.get(url)
  
    # Check if successful
    if response.status_code != 200:
        print(f"Error: {response.status_code}")
        return
  
    # Parse HTML
    soup = BeautifulSoup(response.content, 'html.parser')
  
    # Find all quotes
    quotes = []
    for quote_div in soup.find_all('div', class_='quote'):
        text = quote_div.find('span', class_='text').get_text()
        author = quote_div.find('small', class_='author').get_text()
        tags = [tag.get_text() for tag in quote_div.find_all('a', class_='tag')]
      
        quotes.append({
            'text': text,
            'author': author,
            'tags': tags
        })
  
    return quotes

# BeautifulSoup methods

html_doc = """
<html>
<head><title>Test Page</title></head>
<body>
    <h1 id="main-title">Welcome</h1>
    <div class="content">
        <p class="intro">First paragraph</p>
        <p>Second paragraph</p>
        <a href="https://example.com">Link</a>
    </div>
    <ul>
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
    </ul>
</body>
</html>
"""

soup = BeautifulSoup(html_doc, 'html.parser')

# Find methods
title = soup.find('title')          # First <title>
h1 = soup.find('h1')                # First <h1>
div = soup.find('div', class_='content')  # By class
by_id = soup.find(id='main-title')   # By id

# Find all
all_p = soup.find_all('p')          # All <p> tags
all_li = soup.find_all('li')        # All <li> tags
first_two = soup.find_all('li', limit=2)  # First 2

# CSS selectors
intro = soup.select_one('.intro')    # First with class
all_content = soup.select('.content p')  # All p inside .content
by_id_css = soup.select('#main-title')   # By ID

# Extract data
print(h1.get_text())                # Get text content
print(h1.string)                    # Direct string
link = soup.find('a')
print(link['href'])                 # Get attribute
print(link.get('href'))             # Alternative

# Navigate tree
div = soup.find('div')
children = div.find_all('p')        # Direct children
descendants = div.descendants       # All descendants

# Scrape with headers (avoid blocking)
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml'
}

response = requests.get(url, headers=headers)

# Handle errors
def scrape_with_retry(url, max_retries=3):
    """Scrape with retry logic"""
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()  # Raise exception for bad status
            return response
        except requests.exceptions.RequestException as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)  # Exponential backoff

# Pagination
def scrape_all_pages(base_url):
    """Scrape multiple pages"""
    all_data = []
    page = 1
  
    while True:
        url = f"{base_url}?page={page}"
        response = requests.get(url)
      
        if response.status_code != 200:
            break
      
        soup = BeautifulSoup(response.content, 'html.parser')
      
        # Extract data
        data = extract_data(soup)
      
        if not data:  # No more data
            break
      
        all_data.extend(data)
        page += 1
      
        time.sleep(1)  # Be polite, don't hammer server
  
    return all_data
```

### Advanced Scraping

```python
"""
ADVANCED SCRAPING TECHNIQUES

- Scrapy framework
- JavaScript rendering
- API extraction
- Anti-scraping bypass
"""

# Scrapy spider
import scrapy

class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    start_urls = ['http://quotes.toscrape.com/']
  
    def parse(self, response):
        """Parse quotes page"""
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').get(),
                'author': quote.css('small.author::text').get(),
                'tags': quote.css('a.tag::text').getall(),
            }
      
        # Follow next page
        next_page = response.css('li.next a::attr(href)').get()
        if next_page:
            yield response.follow(next_page, self.parse)

# Run: scrapy runspider quotes_spider.py -o quotes.json

# JavaScript rendering with Selenium
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def scrape_dynamic_content(url):
    """Scrape JavaScript-rendered content"""
    # Setup driver
    options = webdriver.ChromeOptions()
    options.add_argument('--headless')  # Run in background
    driver = webdriver.Chrome(options=options)
  
    try:
        # Load page
        driver.get(url)
      
        # Wait for element to load
        wait = WebDriverWait(driver, 10)
        element = wait.until(
            EC.presence_of_element_located((By.CLASS_NAME, 'dynamic-content'))
        )
      
        # Extract data
        data = element.text
      
        # Scroll to load more (infinite scroll)
        last_height = driver.execute_script("return document.body.scrollHeight")
        while True:
            driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            time.sleep(2)
            new_height = driver.execute_script("return document.body.scrollHeight")
            if new_height == last_height:
                break
            last_height = new_height
      
        return data
  
    finally:
        driver.quit()

# Modern approach: Playwright (faster than Selenium)
from playwright.sync_api import sync_playwright

def scrape_with_playwright(url):
    """Scrape with Playwright"""
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
      
        # Navigate
        page.goto(url)
      
        # Wait for selector
        page.wait_for_selector('.content')
      
        # Extract data
        content = page.query_selector('.content').inner_text()
      
        # Take screenshot
        page.screenshot(path='screenshot.png')
      
        browser.close()
      
        return content

# Extract API calls (best method!)
def find_api_endpoint():
    """
    Sometimes websites load data via API
    Better to use the API directly!
  
    Steps:
    1. Open browser DevTools (F12)
    2. Go to Network tab
    3. Filter by XHR/Fetch
    4. Interact with page
    5. Find API call
    6. Copy URL and use directly
    """
    # Example: Found API endpoint
    api_url = 'https://api.example.com/products?page=1&limit=50'
  
    response = requests.get(api_url)
    data = response.json()
  
    return data  # Much faster and more reliable than scraping!
```

### Browser Automation

```python
"""
BROWSER AUTOMATION: Interact with web pages

Use cases:
- Form filling
- Button clicking
- Testing
- Automated workflows
"""

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
import time

def automate_google_search(query):
    """Automate Google search"""
    driver = webdriver.Chrome()
  
    try:
        # Go to Google
        driver.get('https://www.google.com')
      
        # Find search box
        search_box = driver.find_element(By.NAME, 'q')
      
        # Type query
        search_box.send_keys(query)
      
        # Press Enter
        search_box.send_keys(Keys.RETURN)
      
        # Wait for results
        time.sleep(2)
      
        # Extract results
        results = driver.find_elements(By.CSS_SELECTOR, '.g')
      
        for result in results[:5]:
            title = result.find_element(By.TAG_NAME, 'h3').text
            print(title)
  
    finally:
        driver.quit()

# Form automation
def fill_form(url, form_data):
    """Automate form filling"""
    driver = webdriver.Chrome()
  
    try:
        driver.get(url)
      
        # Fill text fields
        driver.find_element(By.ID, 'name').send_keys(form_data['name'])
        driver.find_element(By.ID, 'email').send_keys(form_data['email'])
      
        # Select dropdown
        from selenium.webdriver.support.ui import Select
        dropdown = Select(driver.find_element(By.ID, 'country'))
        dropdown.select_by_visible_text('United States')
      
        # Check checkbox
        checkbox = driver.find_element(By.ID, 'terms')
        if not checkbox.is_selected():
            checkbox.click()
      
        # Click submit button
        driver.find_element(By.CSS_SELECTOR, 'button[type="submit"]').click()
      
        # Wait for confirmation
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, 'success'))
        )
  
    finally:
        driver.quit()
```

### Ethical Considerations

```python
"""
WEB SCRAPING ETHICS & LEGALITY

DO:
✅ Check robots.txt
✅ Respect rate limits
✅ Identify yourself (User-Agent)
✅ Cache responses
✅ Use APIs when available

DON'T:
❌ Ignore robots.txt
❌ Hammer servers
❌ Scrape copyrighted content
❌ Bypass authentication
❌ Violate ToS
"""

# Check robots.txt
from urllib.robotparser import RobotFileParser

def can_scrape(url):
    """Check if scraping is allowed"""
    rp = RobotFileParser()
    rp.set_url('https://example.com/robots.txt')
    rp.read()
  
    can_fetch = rp.can_fetch('*', url)
    return can_fetch

# Respectful scraping
import time
from datetime import datetime

class RespectfulScraper:
    def __init__(self, delay=1):
        self.delay = delay
        self.last_request = None
  
    def get(self, url):
        """Get URL with rate limiting"""
        # Wait between requests
        if self.last_request:
            elapsed = (datetime.now() - self.last_request).total_seconds()
            if elapsed < self.delay:
                time.sleep(self.delay - elapsed)
      
        # Make request
        response = requests.get(url, headers={
            'User-Agent': 'MyBot/1.0 (contact@example.com)'
        })
      
        self.last_request = datetime.now()
        return response

# Use API when available
"""
Before scraping, check if site has API:
1. Look for /api documentation
2. Check developer portal
3. Search "site_name API"

APIs are:
✅ Faster
✅ More reliable
✅ Legal
✅ Updated automatically
"""
```

### Frequently Asked Questions

**Q1: When should I use Selenium vs BeautifulSoup?**

**A:**

```python
"""
SELENIUM vs BEAUTIFULSOUP DECISION

Use BeautifulSoup when:
✅ Content is in initial HTML (not JavaScript-loaded)
✅ Need fast scraping
✅ Scraping static pages
✅ Low resource usage

Use Selenium when:
✅ Content is JavaScript-rendered
✅ Need to interact (click, type, scroll)
✅ Handling authentication
✅ Complex workflows

Use Playwright when:
✅ Like Selenium but want faster
✅ Better async support
✅ Modern web apps

Use API when:
✅ Site has public API (best option!)
"""

# Example 1: Static content → BeautifulSoup
response = requests.get(url)
soup = BeautifulSoup(response.content, 'html.parser')
data = soup.find_all('div', class_='product')
# Fast, efficient

# Example 2: JavaScript content → Selenium
driver = webdriver.Chrome()
driver.get(url)
time.sleep(2)  # Wait for JS to load
data = driver.find_element(By.CLASS_NAME, 'dynamic-product')
# Slower, but handles JS

# Example 3: Best approach → Find API
"""
Instead of scraping:
https://example.com/products

Look for API:
https://api.example.com/v1/products

Use API directly - much better!
"""
```

**Why This Matters:** Right tool = faster, more reliable scraping.

---

## 8.3 Command-Line Applications

**SDLC Phase:** Development

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (CLI tools)
- [X] Testing
- [X] Deployment
- [X] Maintenance

### Building CLI Tools

```python
"""
CLI TOOLS: Command-line applications

Libraries:
- argparse: Built-in, basic
- click: Decorator-based, popular
- typer: Type-based, modern
"""

# argparse example
import argparse

def main_argparse():
    parser = argparse.ArgumentParser(
        description='Process some files'
    )
  
    # Positional arguments
    parser.add_argument('input', help='Input file')
    parser.add_argument('output', help='Output file')
  
    # Optional arguments
    parser.add_argument(
        '-v', '--verbose',
        action='store_true',
        help='Verbose output'
    )
    parser.add_argument(
        '-f', '--format',
        choices=['json', 'csv', 'xml'],
        default='json',
        help='Output format'
    )
    parser.add_argument(
        '-n', '--count',
        type=int,
        default=10,
        help='Number of items'
    )
  
    args = parser.parse_args()
  
    # Use arguments
    if args.verbose:
        print(f"Processing {args.input} -> {args.output}")
  
    process_file(args.input, args.output, args.format, args.count)

# Click example (cleaner)
import click

@click.command()
@click.argument('input_file')
@click.argument('output_file')
@click.option('-v', '--verbose', is_flag=True, help='Verbose output')
@click.option('-f', '--format', type=click.Choice(['json', 'csv', 'xml']),
              default='json', help='Output format')
@click.option('-n', '--count', type=int, default=10, help='Number of items')
def main_click(input_file, output_file, verbose, format, count):
    """Process some files"""
    if verbose:
        click.echo(f"Processing {input_file} -> {output_file}")
  
    process_file(input_file, output_file, format, count)

if __name__ == '__main__':
    main_click()

# Typer example (modern, type-based)
import typer
from typing import Optional
from enum import Enum

class Format(str, Enum):
    json = "json"
    csv = "csv"
    xml = "xml"

app = typer.Typer()

@app.command()
def process(
    input_file: str = typer.Argument(..., help="Input file"),
    output_file: str = typer.Argument(..., help="Output file"),
    verbose: bool = typer.Option(False, "--verbose", "-v", help="Verbose output"),
    format: Format = typer.Option(Format.json, "--format", "-f", help="Output format"),
    count: int = typer.Option(10, "--count", "-n", help="Number of items")
):
    """Process some files"""
    if verbose:
        typer.echo(f"Processing {input_file} -> {output_file}")
  
    process_file(input_file, output_file, format.value, count)

if __name__ == '__main__':
    app()

# Multi-command CLI
@click.group()
def cli():
    """My awesome CLI tool"""
    pass

@cli.command()
@click.argument('name')
def greet(name):
    """Greet someone"""
    click.echo(f"Hello, {name}!")

@cli.command()
def version():
    """Show version"""
    click.echo("v1.0.0")

if __name__ == '__main__':
    cli()

# Usage:
# $ python cli.py greet Alice
# Hello, Alice!
# $ python cli.py version
# v1.0.0
```

### Rich Terminal UIs

```python
"""
RICH: Beautiful terminal output

Features:
- Colors and styles
- Tables
- Progress bars
- Syntax highlighting
- Panels and layouts
"""

from rich.console import Console
from rich.table import Table
from rich.progress import track
from rich.syntax import Syntax
from rich.panel import Panel
from rich.columns import Columns
import time

console = Console()

# Basic output
console.print("Hello, [bold magenta]World[/bold magenta]!")
console.print("[red]Error:[/red] Something went wrong")
console.print("[green]✓[/green] Success!")

# Tables
table = Table(title="Sales Data")
table.add_column("Product", style="cyan")
table.add_column("Quantity", justify="right", style="magenta")
table.add_column("Price", justify="right", style="green")

table.add_row("Widget", "100", "$1,000")
table.add_row("Gadget", "50", "$750")
table.add_row("Doohickey", "75", "$950")

console.print(table)

# Progress bars
for i in track(range(100), description="Processing..."):
    time.sleep(0.01)

# Syntax highlighting
code = '''
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
'''

syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)

# Panels
console.print(Panel.fit("Important message!", title="Alert"))

# Columns
user_renderable = Panel("[bold]User[/bold]\nJohn Doe", expand=True)
stats_renderable = Panel("[bold]Stats[/bold]\n1000 points", expand=True)

console.print(Columns([user_renderable, stats_renderable]))

# Logging
from rich.logging import RichHandler
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(message)s",
    handlers=[RichHandler(rich_tracebacks=True)]
)

log = logging.getLogger("rich")
log.info("Starting application")
log.warning("This is a warning")
log.error("An error occurred")

# Complete CLI example with Rich
import typer
from rich.console import Console
from rich.progress import Progress

app = typer.Typer()
console = Console()

@app.command()
def process_files(directory: str):
    """Process all files in directory"""
    import os
  
    files = os.listdir(directory)
  
    console.print(f"[bold]Processing {len(files)} files...[/bold]")
  
    with Progress() as progress:
        task = progress.add_task("[green]Processing...", total=len(files))
      
        for file in files:
            # Process file
            time.sleep(0.1)
            progress.advance(task)
  
    console.print("[green]✓[/green] All files processed!")

if __name__ == '__main__':
    app()
```

### Frequently Asked Questions

**Q1: How do I make my Python script executable?**

**A:**

```python
"""
MAKING SCRIPTS EXECUTABLE

Steps:
1. Add shebang line
2. Make file executable
3. Add to PATH (optional)
4. Create package (optional)
"""

# 1. Add shebang at top of script
#!/usr/bin/env python3

import sys

def main():
    print("Hello, World!")

if __name__ == '__main__':
    main()

# 2. Make executable (Linux/Mac)
# $ chmod +x script.py

# 3. Run directly
# $ ./script.py

# 4. Add to PATH (optional)
# Move to ~/bin or /usr/local/bin
# $ sudo mv script.py /usr/local/bin/myscript
# $ myscript  # Run from anywhere

# 5. Create package with entry point
# setup.py
"""
from setuptools import setup

setup(
    name='myscript',
    version='1.0.0',
    py_modules=['myscript'],
    install_requires=[
        'click',
        'rich',
    ],
    entry_points={
        'console_scripts': [
            'myscript=myscript:main',
        ],
    },
)
"""

# Install: $ pip install .
# Run: $ myscript

# 6. Create standalone executable with PyInstaller
# $ pip install pyinstaller
# $ pyinstaller --onefile script.py
# Creates dist/script.exe (Windows) or dist/script (Unix)
```

**Why This Matters:** Makes tools easy to use and distribute.

---

## 8.4 System Programming

```python
"""
SYSTEM PROGRAMMING: Interact with OS

Modules:
- os: Operating system interface
- sys: System-specific parameters
- subprocess: Running external commands
- psutil: Process and system utilities
"""

import os
import sys
import subprocess
import psutil

# OS operations

# Current directory
print(os.getcwd())

# Change directory
os.chdir('/tmp')

# List directory
files = os.listdir('.')

# Path operations
path = os.path.join('dir', 'subdir', 'file.txt')
exists = os.path.exists(path)
is_file = os.path.isfile(path)
is_dir = os.path.isdir(path)

# Create directory
os.mkdir('new_dir')
os.makedirs('parent/child/grandchild', exist_ok=True)

# Remove file/directory
os.remove('file.txt')
os.rmdir('empty_dir')
import shutil
shutil.rmtree('dir_with_contents')

# Environment variables
api_key = os.getenv('API_KEY')
os.environ['MY_VAR'] = 'value'

# System information
print(f"Platform: {sys.platform}")  # 'linux', 'darwin', 'win32'
print(f"Python version: {sys.version}")
print(f"Executable: {sys.executable}")

# Running external commands
# Simple command
result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
print(result.stdout)
print(result.returncode)  # 0 = success

# With shell
result = subprocess.run('ls -l | grep .py', shell=True, capture_output=True, text=True)

# Error handling
try:
    result = subprocess.run(['git', 'status'], check=True, capture_output=True, text=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed with code {e.returncode}")
    print(f"Error: {e.stderr}")

# Process management with psutil

# CPU usage
print(f"CPU count: {psutil.cpu_count()}")
print(f"CPU percent: {psutil.cpu_percent(interval=1)}%")

# Memory usage
memory = psutil.virtual_memory()
print(f"Total memory: {memory.total / (1024**3):.2f} GB")
print(f"Available: {memory.available / (1024**3):.2f} GB")
print(f"Used: {memory.percent}%")

# Disk usage
disk = psutil.disk_usage('/')
print(f"Disk total: {disk.total / (1024**3):.2f} GB")
print(f"Disk free: {disk.free / (1024**3):.2f} GB")

# Running processes
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent']):
    print(proc.info)

# Monitor specific process
import os
pid = os.getpid()
process = psutil.Process(pid)
print(f"CPU: {process.cpu_percent()}%")
print(f"Memory: {process.memory_info().rss / (1024**2):.2f} MB")

# Signal handling
import signal

def signal_handler(signum, frame):
    print("Signal received, cleaning up...")
    # Cleanup code
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)  # Ctrl+C
signal.signal(signal.SIGTERM, signal_handler)  # Kill
```

## 8.5 Task Automation

```python
"""
TASK AUTOMATION: Scheduled and background tasks

Tools:
- schedule: Simple scheduling
- APScheduler: Advanced scheduling
- Celery: Distributed task queue
"""

# Simple scheduling with schedule
import schedule
import time

def job():
    print("Running scheduled job...")

# Schedule jobs
schedule.every(10).seconds.do(job)
schedule.every().hour.do(job)
schedule.every().day.at("10:30").do(job)
schedule.every().monday.do(job)

# Run scheduler
while True:
    schedule.run_pending()
    time.sleep(1)

# APScheduler (more advanced)
from apscheduler.schedulers.blocking import BlockingScheduler

scheduler = BlockingScheduler()

@scheduler.scheduled_job('interval', minutes=5)
def timed_job():
    print("This job runs every 5 minutes")

@scheduler.scheduled_job('cron', day_of_week='mon-fri', hour=9)
def morning_job():
    print("This job runs Monday-Friday at 9 AM")

scheduler.start()

# Celery (distributed tasks)
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def add(x, y):
    return x + y

@app.task
def send_email(to, subject, body):
    # Send email
    print(f"Sending email to {to}")

# Call tasks asynchronously
result = add.delay(4, 6)
result.get()  # Wait for result

# Run worker:
# $ celery -A tasks worker --loglevel=info
```

### Key Takeaways

**Advanced Topics:**

- **NumPy**: Vectorized operations, 10-100x faster than Python lists
- **Pandas**: DataFrames for data analysis, groupby, merging
- **Web Scraping**: BeautifulSoup (static), Selenium (dynamic), check robots.txt
- **CLI**: Click/Typer for arguments, Rich for beautiful output
- **System**: os/subprocess for commands, psutil for monitoring
- **Automation**: schedule/APScheduler for cron, Celery for distributed tasks

---

## Part 8 Complete!

You've completed **Part 8: Advanced Topics & Specialized Domains**!

✅ **Part 1**: Python Fundamentals (~29,000 words)
✅ **Part 2**: Python Internals (~20,000 words)
✅ **Part 3**: SDLC & Architecture (~16,000 words)
✅ **Part 4**: Testing & Quality (~9,200 words)
✅ **Part 5**: Deployment & DevOps (~8,000 words)
✅ **Part 6**: Version Control (~6,500 words)
✅ **Part 7**: Performance & Production (~5,850 words)
✅ **Part 8**: Advanced Topics (~6,500 words)

**Total: ~101,050 words = ~404 pages!** 🎊

Complete Python mastery from fundamentals to specialized domains! 🚀
