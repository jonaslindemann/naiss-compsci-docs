# GCC and LLVM Compilers

!!! note "TODO"
    * Correct module versions.
    * Loading toolchains before CMake.
    * Compare GCC and Clang features.
    * Add cross-compilation section.
    * Add explanatory texts.

Comprehensive guide to using GCC and LLVM/Clang compilers on NAISS systems.

## GCC (GNU Compiler Collection)

### Loading GCC

```bash
module load gcc/11.2.0
```

### Compiler Commands

| Compiler | Language | Command |
|----------|----------|---------|
| `gcc` | C | `gcc -o program program.c` |
| `g++` | C++ | `g++ -o program program.cpp` |
| `gfortran` | Fortran | `gfortran -o program program.f90` |

### Common GCC Flags

#### Optimization Levels
```bash
-O0  # No optimization (default, fastest compile)
-O1  # Basic optimization
-O2  # Moderate optimization (recommended for most code)
-O3  # Aggressive optimization (may increase binary size)
-Os  # Optimize for size
-Ofast  # -O3 + fast math (may break IEEE compliance)
```

#### Debugging
```bash
-g       # Include debug symbols
-ggdb    # GDB-specific debug info
-g3      # Maximum debug info (includes macros)
```

#### Warnings
```bash
-Wall           # Enable common warnings
-Wextra         # Additional warnings
-Werror         # Treat warnings as errors
-pedantic       # Strict ISO compliance
-Wno-unused     # Suppress unused warnings
```

#### Language Standards
```bash
# C
-std=c99
-std=c11
-std=c17
-std=gnu11  # C11 + GNU extensions

# C++
-std=c++11
-std=c++14
-std=c++17
-std=c++20
-std=c++23  # Experimental (GCC 11+)
```

### Architecture-Specific Optimization

```bash
# Generic x86-64
-march=x86-64 -mtune=generic

# Enable CPU-specific optimizations
-march=native

# For specific architectures
-march=skylake
-march=cascadelake
-march=znver2  # AMD Zen 2
```

### OpenMP Support

```bash
# Enable OpenMP
gcc -fopenmp program.c -o program

# Check OpenMP version
echo | gcc -fopenmp -dM -E - | grep -i openmp
```

### Example Compilation

```bash
# Basic
gcc -o myprogram myprogram.c

# Optimized
gcc -O3 -march=native -o myprogram myprogram.c

# Debug build
gcc -g -Og -Wall -Wextra -o myprogram myprogram.c

# Production C++ build
g++ -std=c++17 -O3 -Wall -Wextra -march=native \
    -o myprogram myprogram.cpp
```

## LLVM/Clang

### Loading LLVM

```bash
module load llvm/13.0.0
```

### Compiler Commands

| Compiler | Language | Command |
|----------|----------|---------|
| `clang` | C | `clang -o program program.c` |
| `clang++` | C++ | `clang++ -o program program.cpp` |
| `flang` | Fortran | `flang -o program program.f90` |

### Clang-Specific Features

#### Better Error Messages
Clang typically provides more readable error messages than GCC.

#### Static Analysis
```bash
# Enable static analyzer
clang --analyze program.c

# Use scan-build wrapper
scan-build make
```

#### Sanitizers

**Address Sanitizer** (memory errors):
```bash
clang -fsanitize=address -g program.c -o program
```

**Undefined Behavior Sanitizer**:
```bash
clang -fsanitize=undefined -g program.c -o program
```

**Thread Sanitizer** (data races):
```bash
clang -fsanitize=thread -g program.c -o program
```

**Memory Sanitizer** (uninitialized reads):
```bash
clang -fsanitize=memory -g program.c -o program
```

### Common Clang Flags

Most GCC flags work with Clang. Additional Clang-specific flags:

```bash
-Weverything         # Enable all warnings
-Wno-c++98-compat   # Disable C++98 compatibility warnings
-fcolor-diagnostics  # Colorize output
-fdiagnostics-show-template-tree  # Better template errors
```

### Example Compilation

```bash
# Basic
clang -o myprogram myprogram.c

# With sanitizer
clang++ -std=c++17 -fsanitize=address -g \
    -o myprogram myprogram.cpp

# Production build
clang++ -std=c++20 -O3 -march=native \
    -Wall -Wextra -o myprogram myprogram.cpp
```

## Comparing GCC and Clang

| Feature | GCC | Clang |
|---------|-----|-------|
| Error messages | Good | Excellent |
| Compilation speed | Moderate | Fast |
| Optimization | Excellent | Very good |
| OpenMP support | Excellent | Good |
| Standards compliance | Excellent | Excellent |
| Sanitizers | Some | Comprehensive |
| Platform support | Widest | Growing |

## Cross-Compilation

### Using Clang
```bash
# Compile for different architecture
clang --target=aarch64-linux-gnu program.c
```

## Interprocedural Optimization (IPO/LTO)

### GCC Link-Time Optimization
```bash
# Compile
gcc -O3 -flto -c file1.c
gcc -O3 -flto -c file2.c

# Link
gcc -O3 -flto -o program file1.o file2.o
```

### Clang Link-Time Optimization
```bash
# Compile
clang -O3 -flto -c file1.c
clang -O3 -flto -c file2.c

# Link
clang -O3 -flto -o program file1.o file2.o
```

## Profile-Guided Optimization

### Step 1: Instrument
```bash
gcc -O3 -fprofile-generate program.c -o program
```

### Step 2: Run with Representative Data
```bash
./program < typical_input.txt
```

### Step 3: Optimize
```bash
gcc -O3 -fprofile-use program.c -o program
```

## Mixing Compilers

**Generally avoid mixing compilers** for different object files in the same binary. If you must:

1. Use C linkage for interfaces: `extern "C"`
2. Avoid passing C++ objects across boundaries
3. Be aware of ABI differences
4. Test thoroughly

## Debugging Compilation Issues

### Verbose Output
```bash
gcc -v program.c          # Show compilation steps
gcc -### program.c        # Show commands that would be executed
```

### Preprocessor Output
```bash
gcc -E program.c          # Show preprocessed source
gcc -E -dM empty.c        # Show predefined macros
```

### Assembly Output
```bash
gcc -S program.c          # Generate assembly (.s file)
gcc -fverbose-asm -S program.c  # Assembly with comments
```

### Linker Debugging
```bash
gcc -Wl,--verbose program.c     # Verbose linker output
gcc -Wl,--trace program.c       # Trace linker files
```

## Recommended Flags by Use Case

### Development
```bash
gcc -g -Og -Wall -Wextra -fsanitize=address
```

### Testing
```bash
gcc -O2 -Wall -Wextra -Werror
```

### Production
```bash
gcc -O3 -march=native -DNDEBUG -flto
```

### Debugging Performance
```bash
gcc -O2 -g -fno-omit-frame-pointer
```

## See Also

- [Optimization Guide](optimization.md) - Detailed optimization strategies
- [Programming Environments](prgenv.md) - Environment setup
- [CMake](../build-systems/cmake.md) - Using compilers with CMake
