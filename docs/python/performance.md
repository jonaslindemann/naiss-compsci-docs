# Python Performance Optimization

!!! note "TODO"
    * Verify all examples on NAISS systems.
    * Add more examples for common scientific libraries.
    * Include best practices for performance tuning.
    * Check for correctness.

Techniques for improving Python performance on NAISS HPC systems.

## Understanding Python Performance

### Why is Python Slow?

- **Interpreted**: No ahead-of-time compilation
- **Dynamic typing**: Runtime type checks
- **GIL**: Global Interpreter Lock limits parallelism
- **Memory overhead**: Objects are expensive

### When Python is Fast Enough

- I/O bound operations
- Quick scripts and prototypes
- Using optimized libraries (NumPy, SciPy)
- Embarrassingly parallel workflows

## Use Optimized Libraries

### NumPy - The Foundation

NumPy operations are implemented in C and can be 10-100x faster than pure Python.

**Slow (Pure Python):**
```python
result = []
for i in range(len(a)):
    result.append(a[i] + b[i])
```

**Fast (NumPy):**
```python
import numpy as np
result = a + b  # Vectorized operation
```

### Vectorization Examples

```python
import numpy as np

# Bad: Python loops
def slow_sum(arr):
    total = 0
    for x in arr:
        total += x
    return total

# Good: NumPy
def fast_sum(arr):
    return np.sum(arr)

# Bad: Element-wise operations
result = np.zeros(len(a))
for i in range(len(a)):
    result[i] = np.sqrt(a[i] ** 2 + b[i] ** 2)

# Good: Vectorized
result = np.sqrt(a**2 + b**2)
```

### Key Scientific Libraries

- **NumPy**: Array operations
- **SciPy**: Scientific algorithms
- **Pandas**: Data manipulation
- **scikit-learn**: Machine learning
- **Numba**: JIT compilation

## Numba - JIT Compilation

### Basic Usage

```python
from numba import jit
import numpy as np

@jit(nopython=True)
def fast_function(x):
    result = 0.0
    for i in range(x.shape[0]):
        result += x[i] ** 2
    return result

# First call compiles, subsequent calls are fast
x = np.random.random(1000000)
result = fast_function(x)  # 10-100x faster than pure Python
```

### Numba Decorators

```python
from numba import jit, njit, vectorize

# Standard JIT
@jit
def func1(x):
    return x ** 2

# No Python mode (faster, more restrictive)
@njit
def func2(x):
    return x ** 2

# Vectorized function
@vectorize
def func3(x, y):
    return x + y
```

### Parallel Numba

```python
from numba import njit, prange

@njit(parallel=True)
def parallel_sum(arr):
    total = 0.0
    for i in prange(len(arr)):
        total += arr[i] ** 2
    return total
```

### When to Use Numba

- Numerical computations with loops
- Can't easily vectorize with NumPy
- Need C-like performance without writing C
- Array-heavy calculations

### Numba Limitations

- Not all Python features supported
- Limited support for classes
- No dynamic typing in nopython mode
- Some NumPy functions unavailable

## Cython - Python to C

### Basic Example

**mymodule.pyx:**
```cython
def fast_loop(int n):
    cdef int i
    cdef double total = 0.0
    
    for i in range(n):
        total += i * i
    
    return total
```

**setup.py:**
```python
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules=cythonize("mymodule.pyx")
)
```

**Build:**
```bash
python setup.py build_ext --inplace
```

### Type Annotations

```cython
import numpy as np
cimport numpy as np

def process_array(np.ndarray[np.float64_t, ndim=1] arr):
    cdef int i
    cdef double total = 0.0
    cdef int n = arr.shape[0]
    
    for i in range(n):
        total += arr[i]
    
    return total
```

## Profiling Python Code

### Using cProfile

```python
import cProfile
import pstats

# Profile code
profiler = cProfile.Profile()
profiler.enable()

# Your code here
result = expensive_function()

profiler.disable()

# Print stats
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(20)  # Top 20 functions
```

### Command-Line Profiling

```bash
python -m cProfile -o profile.stats script.py
python -m pstats profile.stats
```

### line_profiler (Line-by-Line)

```bash
pip install line_profiler
```

```python
@profile
def slow_function():
    # Function body
    pass
```

```bash
kernprof -l -v script.py
```

### memory_profiler

```bash
pip install memory_profiler
```

```python
from memory_profiler import profile

@profile
def memory_intensive():
    large_list = [i for i in range(10**7)]
    return sum(large_list)
```

```bash
python -m memory_profiler script.py
```

## Multiprocessing

### Bypassing the GIL

```python
from multiprocessing import Pool
import numpy as np

def process_chunk(data):
    # Heavy computation
    return np.sum(data ** 2)

if __name__ == '__main__':
    # Split data
    data_chunks = np.array_split(large_array, n_workers)
    
    # Process in parallel
    with Pool(processes=8) as pool:
        results = pool.map(process_chunk, data_chunks)
    
    # Combine results
    total = sum(results)
```

### Process Pool Example

```python
from multiprocessing import Pool

def compute_square(x):
    return x * x

if __name__ == '__main__':
    numbers = range(1000)
    
    with Pool(processes=4) as pool:
        results = pool.map(compute_square, numbers)
```

### Shared Memory (Python 3.8+)

```python
from multiprocessing import Pool, shared_memory
import numpy as np

def worker(shm_name, shape, dtype):
    # Attach to existing shared memory
    existing_shm = shared_memory.SharedMemory(name=shm_name)
    arr = np.ndarray(shape, dtype=dtype, buffer=existing_shm.buf)
    
    # Process array
    result = np.sum(arr ** 2)
    
    existing_shm.close()
    return result
```

## Threading (for I/O-bound tasks)

```python
from concurrent.futures import ThreadPoolExecutor
import requests

def fetch_url(url):
    response = requests.get(url)
    return len(response.content)

urls = ['http://example.com'] * 10

# I/O-bound: threading works well
with ThreadPoolExecutor(max_workers=10) as executor:
    results = executor.map(fetch_url, urls)
```

**Note:** Threads don't help with CPU-bound Python code due to GIL.

## Memory Optimization

### Generators vs Lists

```python
# Bad: Creates full list in memory
def get_numbers():
    return [i for i in range(10**6)]

# Good: Generator
def get_numbers():
    for i in range(10**6):
        yield i

# Or use generator expression
numbers = (i for i in range(10**6))
```

### NumPy Memory Views

```python
# Bad: Creates copy
subset = large_array[1000:2000]

# Good: View (no copy)
subset = large_array[1000:2000].view()
```

### Memory-Mapped Files

```python
import numpy as np

# Create memory-mapped array
data = np.memmap('large_data.dat', dtype='float32', 
                 mode='w+', shape=(10000, 10000))

# Work with it like normal array
data[0, :] = np.random.random(10000)

# Automatically saved to disk
del data
```

### Delete Unused Variables

```python
import gc

# Delete large objects
del large_array
gc.collect()  # Force garbage collection
```

## NumPy Performance Tips

### Avoid Loops

```python
# Bad
result = np.zeros(n)
for i in range(n):
    result[i] = arr[i] * 2

# Good
result = arr * 2
```

### Use In-Place Operations

```python
# Creates new array
arr = arr + 1

# In-place (saves memory)
arr += 1

# In-place functions
np.add(arr, 1, out=arr)
np.sqrt(arr, out=arr)
```

### Choose Right Data Type

```python
# Default is float64
arr = np.array([1.0, 2.0, 3.0])  # 8 bytes per element

# Use float32 when appropriate
arr = np.array([1.0, 2.0, 3.0], dtype=np.float32)  # 4 bytes per element
```

### Contiguous Arrays

```python
# Check if contiguous
arr.flags['C_CONTIGUOUS']

# Make contiguous copy if needed
arr = np.ascontiguousarray(arr)
```

## Pandas Performance

### Vectorized Operations

```python
import pandas as pd

# Bad: iterrows
for idx, row in df.iterrows():
    df.loc[idx, 'new_col'] = row['A'] + row['B']

# Good: vectorized
df['new_col'] = df['A'] + df['B']
```

### Use apply() with NumPy

```python
# Better than iterrows, but slower than vectorization
df['result'] = df.apply(lambda row: row['A'] ** 2 + row['B'], axis=1)

# Even better: use NumPy directly
df['result'] = df['A'].values ** 2 + df['B'].values
```

### Categorical Data

```python
# Memory-efficient for repeated strings
df['category'] = df['category'].astype('category')
```

## I/O Optimization

### Use Binary Formats

```python
# Slow: CSV
df.to_csv('data.csv')

# Fast: Parquet
df.to_parquet('data.parquet')

# Fast: HDF5
df.to_hdf('data.h5', key='df')

# Fast: Pickle (Python only)
df.to_pickle('data.pkl')
```

### Chunked Reading

```python
# Read large CSV in chunks
chunk_size = 10**6
for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    process(chunk)
```

## Best Practices Summary

1. **Profile first** - Don't optimize prematurely
2. **Use NumPy** for array operations
3. **Vectorize** instead of loops
4. **JIT compile** with Numba for unavoidable loops
5. **Multiprocessing** for CPU-bound parallelism
6. **Threading** for I/O-bound parallelism
7. **Memory-map** very large files
8. **Use generators** for large sequences
9. **Binary formats** for data storage
10. **Right data types** (float32 vs float64)

## Benchmarking

```python
import timeit

# Method 1
time1 = timeit.timeit('slow_function()', globals=globals(), number=100)

# Method 2
time2 = timeit.timeit('fast_function()', globals=globals(), number=100)

print(f"Speedup: {time1/time2:.2f}x")
```

## Example: Complete Optimization

```python
import numpy as np
from numba import njit, prange

# Original (slow)
def slow_distance_matrix(points):
    n = len(points)
    dist = [[0.0] * n for _ in range(n)]
    for i in range(n):
        for j in range(n):
            dx = points[i][0] - points[j][0]
            dy = points[i][1] - points[j][1]
            dist[i][j] = (dx**2 + dy**2)**0.5
    return dist

# Optimized (fast)
@njit(parallel=True)
def fast_distance_matrix(points):
    n = points.shape[0]
    dist = np.zeros((n, n))
    for i in prange(n):
        for j in range(n):
            dx = points[i, 0] - points[j, 0]
            dy = points[i, 1] - points[j, 1]
            dist[i, j] = np.sqrt(dx**2 + dy**2)
    return dist

# Even better: Pure NumPy (for this case)
def numpy_distance_matrix(points):
    return np.sqrt(((points[:, None, :] - points[None, :, :]) ** 2).sum(-1))
```

## See Also

- [Environments](environments.md) - Setting up Python
- [MPI Python](mpi-python.md) - Distributed parallelism
- [Profiling](../profiling-debugging/profiling.md) - System-level profiling
