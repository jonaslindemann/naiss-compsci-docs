# Debugging on NAISS Systems

!!! note "TODO"
    * Include system-specific debugging instructions.
    * Verify all commands on NAISS systems.
    * Add more explanatory texts for each section.

Comprehensive guide to debugging applications on NAISS HPC systems.

## Debugging Strategies

### Compilation for Debugging

```bash
# GCC - Debug build
gcc -g -O0 -Wall -Wextra -o program program.c

# With sanitizers
gcc -g -fsanitize=address -fsanitize=undefined -o program program.c

# Maximum debug info
gcc -g3 -O0 -Wall -Wextra -o program program.c
```

**Flags explained:**
- `-g`: Include debug symbols
- `-g3`: Maximum debug info (includes macros)
- `-O0`: No optimization (easier debugging)
- `-Wall -Wextra`: Enable warnings
- `-Og`: Optimize for debugging (better than -O0, still debuggable)

## GDB (GNU Debugger)

### Basic GDB Usage

```bash
# Load program
gdb ./program

# With arguments
gdb --args ./program arg1 arg2

# Run
(gdb) run

# Run with input redirection
(gdb) run < input.txt
```

### Essential GDB Commands

```gdb
# Execution control
run (r)              # Start program
continue (c)         # Continue execution
next (n)             # Next line (step over)
step (s)             # Step into function
finish               # Continue until function returns
quit (q)             # Exit GDB

# Breakpoints
break main           # Break at function
break file.c:42      # Break at line
break file.c:42 if x > 10  # Conditional breakpoint
info breakpoints     # List breakpoints
delete 1             # Delete breakpoint 1
disable 1            # Disable breakpoint 1
enable 1             # Enable breakpoint 1

# Inspection
print variable       # Print variable
print *pointer       # Dereference pointer
print array[5]       # Array element
display expr         # Auto-display expression
info locals          # Show local variables
info args            # Show function arguments

# Stack
backtrace (bt)       # Show call stack
frame 3              # Switch to frame 3
up                   # Move up stack
down                 # Move down stack

# Source code
list                 # Show source
list function        # Show function source
```

### Example GDB Session

```bash
$ gdb ./myprogram
(gdb) break main
Breakpoint 1 at 0x401136: file main.c, line 5.
(gdb) run
Starting program: ./myprogram 

Breakpoint 1, main () at main.c:5
5           int x = 42;
(gdb) next
6           int y = compute(x);
(gdb) step
compute (n=42) at compute.c:3
3           return n * n;
(gdb) print n
$1 = 42
(gdb) finish
Run till exit from #0  compute (n=42) at compute.c:3
main () at main.c:7
7           printf("Result: %d\n", y);
(gdb) print y
$2 = 1764
(gdb) continue
Result: 1764
[Inferior 1 (process 12345) exited normally]
(gdb) quit
```

### Debugging Segmentation Faults

```bash
$ gdb ./program
(gdb) run
# Program crashes

Program received signal SIGSEGV, Segmentation fault.
0x0000000000401142 in main () at program.c:8
8           *ptr = 42;

(gdb) backtrace
#0  0x0000000000401142 in main () at program.c:8

(gdb) print ptr
$1 = (int *) 0x0

(gdb) # ptr is NULL - found the bug!
```

### Core Dumps

Enable core dumps:
```bash
ulimit -c unlimited
```

Debug core dump:
```bash
gdb ./program core

(gdb) backtrace
(gdb) frame 0
(gdb) print variable
```

### Debugging Parallel Programs

#### MPI Debugging

**Serial approach:**
```bash
mpirun -np 4 xterm -e gdb ./mpi_program
```

**Attach to specific rank:**
```bash
# In code, add for rank 0:
if (rank == 0) {
    int i = 0;
    char hostname[256];
    gethostname(hostname, sizeof(hostname));
    printf("PID %d on %s ready for attach\n", getpid(), hostname);
    fflush(stdout);
    while (0 == i)
        sleep(5);
}

# Then attach:
gdb -p <PID>
(gdb) set var i=1
(gdb) continue
```

#### OpenMP Debugging

```bash
export OMP_NUM_THREADS=4
gdb ./omp_program

(gdb) break omp_function
(gdb) run

# When breakpoint hits
(gdb) info threads
(gdb) thread 2        # Switch to thread 2
(gdb) backtrace
```

## Sanitizers

### AddressSanitizer (ASan)

Detects:
- Buffer overflows
- Use-after-free
- Memory leaks
- Double-free

```bash
# Compile
gcc -g -fsanitize=address -o program program.c

# Run
./program

# Output shows exact error location
```

### UndefinedBehaviorSanitizer (UBSan)

Detects:
- Integer overflow
- Null pointer dereference
- Invalid shifts
- Alignment issues

```bash
gcc -g -fsanitize=undefined -o program program.c
./program
```

### ThreadSanitizer (TSan)

Detects data races in multithreaded code:

```bash
gcc -g -fsanitize=thread -o program program.c
./program
```

**Note:** Cannot combine with AddressSanitizer.

### MemorySanitizer (MSan)

Detects uninitialized memory reads:

```bash
clang -g -fsanitize=memory -o program program.c
./program
```

**Note:** Requires all code (including libraries) to be compiled with MSan.

### Combining Sanitizers

```bash
# ASan + UBSan (common combination)
gcc -g -fsanitize=address,undefined -o program program.c
```

## Valgrind

### Memory Error Detection

```bash
module load valgrind

# Basic memory check
valgrind --leak-check=full ./program

# More detailed
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./program

# Save to file
valgrind --leak-check=full --log-file=valgrind.log ./program
```

### Valgrind Tools

```bash
# Memcheck (default) - memory errors
valgrind --tool=memcheck ./program

# Cachegrind - cache profiling
valgrind --tool=cachegrind ./program
cg_annotate cachegrind.out.<pid>

# Callgrind - call graph profiling
valgrind --tool=callgrind ./program
callgrind_annotate callgrind.out.<pid>

# Helgrind - thread errors
valgrind --tool=helgrind ./threaded_program

# Massif - heap profiling
valgrind --tool=massif ./program
ms_print massif.out.<pid>
```

### Example Valgrind Output

```
==12345== Invalid write of size 4
==12345==    at 0x401156: main (program.c:10)
==12345==  Address 0x4a2f040 is 0 bytes after a block of size 40 alloc'd
==12345==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck.so)
==12345==    by 0x401145: main (program.c:8)
```

## Static Analysis

### Compiler Warnings

```bash
# Enable all warnings
gcc -Wall -Wextra -Wpedantic -Werror -o program program.c

# Additional useful warnings
gcc -Wall -Wextra \
    -Wconversion \
    -Wshadow \
    -Wcast-qual \
    -Wcast-align \
    -o program program.c
```

### Clang Static Analyzer

```bash
# Analyze
scan-build gcc -o program program.c

# With make
scan-build make

# View HTML report
scan-build -o /tmp/scan make
```

### Cppcheck

```bash
# Install
pip install cppcheck

# Check code
cppcheck --enable=all --inconclusive program.c

# Generate report
cppcheck --enable=all --xml program.c 2> report.xml
```

## Debugging Techniques

### Printf Debugging

```c
#include <stdio.h>

#define DEBUG_PRINT(fmt, ...) \
    fprintf(stderr, "[%s:%d] " fmt "\n", __FILE__, __LINE__, ##__VA_ARGS__)

void function(int x) {
    DEBUG_PRINT("x = %d", x);
    // ...
}
```

### Assertions

```c
#include <assert.h>

void function(int *ptr) {
    assert(ptr != NULL);
    assert(*ptr >= 0);
    // ...
}
```

Disable in production:
```bash
gcc -DNDEBUG -O3 -o program program.c
```

### Conditional Compilation

```c
#ifdef DEBUG
    printf("Debug: x = %d\n", x);
#endif
```

```bash
# Debug build
gcc -DDEBUG -g -o program program.c

# Release build
gcc -O3 -o program program.c
```

## Debugging Python

### pdb (Python Debugger)

```python
import pdb

def function(x):
    pdb.set_trace()  # Breakpoint
    y = x * 2
    return y
```

**pdb commands:**
```
n(ext)       # Next line
s(tep)       # Step into
c(ontinue)   # Continue
p variable   # Print variable
l(ist)       # Show source
w(here)      # Stack trace
q(uit)       # Quit
```

### Post-mortem Debugging

```python
import pdb
import sys

def main():
    # Your code
    pass

if __name__ == '__main__':
    try:
        main()
    except:
        type, value, tb = sys.exc_info()
        pdb.post_mortem(tb)
```

### IPython Debugger

```python
from IPython.core.debugger import set_trace

def function():
    set_trace()  # Better debugger than pdb
    # ...
```

## Common Debugging Scenarios

### Segmentation Fault

1. **Compile with debug symbols:**
   ```bash
   gcc -g -O0 -o program program.c
   ```

2. **Run in GDB:**
   ```bash
   gdb ./program
   (gdb) run
   ```

3. **Check backtrace:**
   ```gdb
   (gdb) backtrace
   (gdb) frame 0
   (gdb) print *pointer
   ```

### Memory Leak

1. **Use AddressSanitizer:**
   ```bash
   gcc -g -fsanitize=address -o program program.c
   ./program
   ```

2. **Or Valgrind:**
   ```bash
   valgrind --leak-check=full ./program
   ```

### Race Condition

1. **ThreadSanitizer:**
   ```bash
   gcc -g -fsanitize=thread -o program program.c
   ./program
   ```

2. **Helgrind:**
   ```bash
   valgrind --tool=helgrind ./program
   ```

### Wrong Results

1. **Enable sanitizers:**
   ```bash
   gcc -g -fsanitize=address,undefined -o program program.c
   ```

2. **Check intermediate values:**
   ```bash
   gdb ./program
   (gdb) break critical_function
   (gdb) run
   (gdb) print variable
   ```

3. **Compare with reference:**
   - Use known-good inputs
   - Simplify test case
   - Binary search to find error

## Debugging in SLURM Jobs

### Interactive Debugging

```bash
# Request interactive session
salloc -N 1 -t 1:00:00

# On allocated node
srun --pty bash
gdb ./program
```

### Debug Job Script

```bash
#!/bin/bash
#SBATCH --job-name=debug
#SBATCH --time=01:00:00

# Enable core dumps
ulimit -c unlimited

# Run with debugger
module load gdb
gdb -batch -ex "run" -ex "backtrace" --args ./program

# Or with sanitizers
./program  # Built with -fsanitize=address
```

## Debugging Checklist

- [ ] Compile with `-g -O0`
- [ ] Enable compiler warnings (`-Wall -Wextra`)
- [ ] Use sanitizers during development
- [ ] Run with Valgrind for memory issues
- [ ] Check with small, known inputs
- [ ] Use debugger (GDB) for crashes
- [ ] Print intermediate values
- [ ] Verify assumptions with assertions
- [ ] Test edge cases
- [ ] Simplify to minimal failing case

## Tools Summary

| Tool | Purpose | Best For |
|------|---------|----------|
| GDB | Interactive debugging | Crashes, logic errors |
| ASan | Memory errors | Buffer overflows, use-after-free |
| UBSan | Undefined behavior | Integer overflow, null deref |
| TSan | Thread issues | Data races |
| Valgrind | Memory profiling | Memory leaks, cache analysis |
| Static analyzers | Code quality | Finding potential bugs |

## See Also

- [Profiling](profiling.md) - Performance analysis
- [Optimization](../compilers/optimization.md) - Compiler optimization
- [Common Errors](../build-systems/common-errors.md) - Build issues
