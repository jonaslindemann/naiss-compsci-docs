# MPI for Python (mpi4py)

Guide to using MPI for parallel Python applications on NAISS systems.

## Introduction

MPI4Py provides Python bindings for the Message Passing Interface (MPI), enabling distributed parallel computing across multiple nodes.

## Installation

### Loading Modules

```bash
module load python/3.9.5
module load openmpi/4.1.1
```

### Installing mpi4py

```bash
# Activate your virtual environment
python3 -m venv myenv
source myenv/bin/activate

# Install mpi4py
pip install mpi4py
```

### Verify Installation

```python
from mpi4py import MPI
print(f"MPI version: {MPI.Get_version()}")
print(f"Processor name: {MPI.Get_processor_name()}")
```

## Basic Concepts

### Communicator

The communicator manages communication between processes.

```python
from mpi4py import MPI

comm = MPI.COMM_WORLD  # Default communicator
rank = comm.Get_rank()  # Process ID (0 to size-1)
size = comm.Get_size()  # Total number of processes

print(f"Hello from rank {rank} of {size}")
```

### Running MPI Programs

```bash
mpirun -np 4 python my_script.py
```

With SLURM:
```bash
srun -n 4 python my_script.py
```

## Point-to-Point Communication

### Send and Receive

```python
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

if rank == 0:
    data = {'key': 'value', 'number': 42}
    comm.send(data, dest=1, tag=11)
    print(f"Rank 0 sent: {data}")
elif rank == 1:
    data = comm.recv(source=0, tag=11)
    print(f"Rank 1 received: {data}")
```

### Uppercase Send/Recv (NumPy arrays)

```python
import numpy as np
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

if rank == 0:
    data = np.array([1, 2, 3, 4], dtype='i')
    comm.Send(data, dest=1, tag=13)
elif rank == 1:
    data = np.empty(4, dtype='i')
    comm.Recv(data, source=0, tag=13)
    print(f"Rank 1 received: {data}")
```

**Note:** 
- Lowercase (`send`/`recv`): Pickle-based, works with any Python object
- Uppercase (`Send`/`Recv`): Buffer-based, faster for NumPy arrays

### Non-blocking Communication

```python
import numpy as np
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

if rank == 0:
    data = np.array([1, 2, 3, 4], dtype='i')
    req = comm.Isend(data, dest=1, tag=0)
    # Do other work
    req.wait()  # Wait for completion
elif rank == 1:
    data = np.empty(4, dtype='i')
    req = comm.Irecv(data, source=0, tag=0)
    # Do other work
    req.wait()
    print(f"Received: {data}")
```

## Collective Communication

### Broadcast

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

if rank == 0:
    data = np.array([1, 2, 3, 4], dtype='i')
else:
    data = np.empty(4, dtype='i')

# Broadcast from root (rank 0) to all processes
comm.Bcast(data, root=0)

print(f"Rank {rank} has: {data}")
```

### Scatter

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

if rank == 0:
    # Data to distribute
    data = np.arange(size * 4, dtype='i')
else:
    data = None

# Each process receives a chunk
recvbuf = np.empty(4, dtype='i')
comm.Scatter(data, recvbuf, root=0)

print(f"Rank {rank} received: {recvbuf}")
```

### Gather

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Each process has local data
sendbuf = np.array([rank] * 4, dtype='i')

if rank == 0:
    recvbuf = np.empty(size * 4, dtype='i')
else:
    recvbuf = None

# Gather all data at root
comm.Gather(sendbuf, recvbuf, root=0)

if rank == 0:
    print(f"Gathered data: {recvbuf}")
```

### Reduce

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

# Local data
sendbuf = np.array([rank + 1], dtype='i')

# Reduce to sum at root
recvbuf = np.empty(1, dtype='i')
comm.Reduce(sendbuf, recvbuf, op=MPI.SUM, root=0)

if rank == 0:
    print(f"Sum: {recvbuf[0]}")
```

### Allreduce

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

sendbuf = np.array([rank + 1], dtype='i')
recvbuf = np.empty(1, dtype='i')

# All processes receive the result
comm.Allreduce(sendbuf, recvbuf, op=MPI.SUM)

print(f"Rank {rank}: total sum = {recvbuf[0]}")
```

### Available Reduction Operations

- `MPI.SUM` - Sum
- `MPI.PROD` - Product
- `MPI.MAX` - Maximum
- `MPI.MIN` - Minimum
- `MPI.LAND` - Logical AND
- `MPI.LOR` - Logical OR
- `MPI.BAND` - Bitwise AND
- `MPI.BOR` - Bitwise OR

## Parallel Patterns

### Embarrassingly Parallel

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Total work
N = 1000

# Divide work among processes
local_N = N // size
start = rank * local_N
end = (rank + 1) * local_N if rank < size - 1 else N

# Perform local computation
local_sum = 0.0
for i in range(start, end):
    local_sum += i * i

# Gather results
total_sum = comm.reduce(local_sum, op=MPI.SUM, root=0)

if rank == 0:
    print(f"Total sum: {total_sum}")
```

### Master-Worker Pattern

```python
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

if rank == 0:
    # Master process
    tasks = list(range(100))  # 100 tasks
    
    # Distribute tasks
    for worker in range(1, size):
        if tasks:
            task = tasks.pop(0)
            comm.send(task, dest=worker, tag=1)
        else:
            comm.send(None, dest=worker, tag=1)  # Signal completion
    
    # Collect results and send more tasks
    completed = 0
    while completed < 100:
        result = comm.recv(source=MPI.ANY_SOURCE, tag=2)
        completed += 1
        sender = result['rank']
        
        if tasks:
            task = tasks.pop(0)
            comm.send(task, dest=sender, tag=1)
        else:
            comm.send(None, dest=sender, tag=1)
else:
    # Worker processes
    while True:
        task = comm.recv(source=0, tag=1)
        if task is None:
            break
        
        # Process task
        result = task * task
        
        # Send result back
        comm.send({'rank': rank, 'result': result}, dest=0, tag=2)
```

## NumPy Integration

### Parallel Array Operations

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Global array size
N = 1000

# Local size
local_N = N // size

# Create local array
local_data = np.random.random(local_N)

# Perform local computation
local_sum = np.sum(local_data ** 2)

# Reduce to get global result
global_sum = comm.reduce(local_sum, op=MPI.SUM, root=0)

if rank == 0:
    print(f"Global sum: {global_sum}")
```

### Parallel Matrix-Vector Multiplication

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

N = 100  # Matrix size

# Each process gets rows of matrix
local_rows = N // size
local_A = np.random.random((local_rows, N))

# Vector is broadcast to all
if rank == 0:
    x = np.random.random(N)
else:
    x = np.empty(N)

comm.Bcast(x, root=0)

# Local matrix-vector product
local_y = np.dot(local_A, x)

# Gather results
if rank == 0:
    y = np.empty(N)
else:
    y = None

comm.Gather(local_y, y, root=0)

if rank == 0:
    print(f"Result shape: {y.shape}")
```

## SLURM Job Script

```bash
#!/bin/bash
#SBATCH --job-name=mpi_python
#SBATCH --time=01:00:00
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=20
#SBATCH --account=your_account

# Load modules
module purge
module load python/3.9.5
module load openmpi/4.1.1

# Activate virtual environment
source /path/to/venv/bin/activate

# Run MPI program
srun python mpi_script.py
```

## Performance Considerations

### Use Uppercase Methods for NumPy

```python
# Slow: Pickle-based
comm.send(numpy_array, dest=1)

# Fast: Buffer-based
comm.Send(numpy_array, dest=1)
```

### Non-blocking Communication

```python
# Overlap communication and computation
req = comm.Isend(data, dest=1)

# Do independent work here
perform_computation()

# Wait for communication to complete
req.wait()
```

### Collective Operations

```python
# Better: Use collective operations
total = comm.allreduce(local_sum, op=MPI.SUM)

# Avoid: Manual gathering
if rank == 0:
    total = local_sum
    for i in range(1, size):
        total += comm.recv(source=i)
```

## Common Patterns Summary

### Data Parallelism
Each process works on different data with same code.

### Task Parallelism
Master distributes different tasks to workers.

### Pipeline Parallelism
Processes form a pipeline, each doing one stage.

### Hybrid (MPI + OpenMP)
```python
from mpi4py import MPI
import numpy as np
from numba import njit, prange

@njit(parallel=True)
def local_work(data):
    result = np.zeros_like(data)
    for i in prange(len(data)):
        result[i] = data[i] ** 2
    return result

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

# MPI: distributed across nodes
# OpenMP: parallel on each node via Numba
local_data = np.random.random(1000000)
local_result = local_work(local_data)  # Uses multiple threads
```

## Debugging MPI Programs

### Print with Rank

```python
import sys
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()

def debug_print(msg):
    print(f"[Rank {rank}] {msg}", flush=True)

debug_print("Starting computation")
```

### Attach Debugger

```bash
# In job script, for one rank
if rank == 0:
    import pdb; pdb.set_trace()
```

### Check MPI Configuration

```python
from mpi4py import MPI

print(f"MPI version: {MPI.Get_version()}")
print(f"Vendor: {MPI.get_vendor()}")
print(f"Library version: {MPI.Get_library_version()}")
```

## Example: Parallel Monte Carlo

```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

# Total samples
N = 10**7
local_N = N // size

# Generate random points
np.random.seed(rank)
x = np.random.random(local_N)
y = np.random.random(local_N)

# Count points inside circle
local_inside = np.sum(x**2 + y**2 <= 1.0)

# Reduce to get total
total_inside = comm.reduce(local_inside, op=MPI.SUM, root=0)

if rank == 0:
    pi_estimate = 4.0 * total_inside / N
    print(f"Estimated π = {pi_estimate}")
    print(f"Error = {abs(pi_estimate - np.pi)}")
```

## Best Practices

1. **Load balance** - Distribute work evenly
2. **Minimize communication** - Computation >> communication
3. **Use collective operations** when possible
4. **Buffer-based methods** for NumPy arrays
5. **Non-blocking** for overlapping communication/computation
6. **Test with small ranks** before scaling up
7. **Profile** to find bottlenecks

## See Also

- [Environments](environments.md) - Python virtual environments
- [Performance](performance.md) - Python optimization
- [Profiling](../profiling-debugging/profiling.md) - Performance analysis
