# Compiler Optimization Guide

!!! note "TODO"
    * Include system-specific recommendations.
    * Add explanatory texts for each section.
    * Validate examples on NAISS systems.
    * Check for correctness.
    * Verify all flags with current compiler versions.
    * Add benchmarking section.

Learn how to optimize code compilation for performance on NAISS HPC systems.

## Optimization Levels

### Understanding -O Flags

| Flag | Description | Use Case |
|------|-------------|----------|
| `-O0` | No optimization | Debugging |
| `-O1` | Basic optimization | Quick compile + some speed |
| `-O2` | Recommended default | Production (balanced) |
| `-O3` | Aggressive optimization | Maximum performance |
| `-Os` | Optimize for size | Embedded systems |
| `-Ofast` | `-O3` + fast math | HPC (breaks strict standards) |
| `-Og` | Debug-friendly optimization | Development |

### Detailed Optimization Levels

#### -O0 (Default)
- No optimization
- Fastest compilation
- Best for debugging
- Largest binary size
- Slowest execution

#### -O2 (Recommended)
- Good performance
- Reasonable compile time
- Safe optimizations
- **Use as baseline for production**

#### -O3
- More aggressive than -O2
- Longer compile time
- May increase code size
- Can be slower than -O2 (rare)
- **Test before deploying**

#### -Ofast
- `-O3` + fast math (`-ffast-math`)
- **May violate IEEE 754**
- Can produce incorrect results for some code
- **Only use if you understand implications**

## Architecture-Specific Optimization

### CPU Feature Flags

```bash
# Optimize for current CPU
-march=native

# Specific architectures (Intel)
-march=skylake
-march=cascadelake
-march=icelake
-march=sapphirerapids

# Specific architectures (AMD)
-march=znver2  # EPYC Rome
-march=znver3  # EPYC Milan

# Generic x86-64
-march=x86-64
-march=x86-64-v2  # SSE4.2, POPCNT
-march=x86-64-v3  # AVX2, FMA
-march=x86-64-v4  # AVX512
```

### Tuning vs Architecture

```bash
# Generate code for <arch>, tune for <cpu>
-march=x86-64 -mtune=skylake

# march: determines instruction set (affects compatibility)
# mtune: optimizes scheduling (doesn't affect compatibility)
```

### Portability Considerations

```bash
# For code running on multiple systems
-march=x86-64-v2 -mtune=generic

# For single system
-march=native
```

## Vectorization

### Auto-Vectorization

GCC automatically vectorizes at `-O3`:
```bash
gcc -O3 -march=native -fopt-info-vec program.c
```

### Vectorization Reports

```bash
# GCC
-fopt-info-vec                    # What was vectorized
-fopt-info-vec-missed            # What wasn't vectorized
-fopt-info-vec-optimized         # Successfully vectorized
-fopt-info-vec-all               # All vectorization info

# Intel
-qopt-report=5                   # Detailed optimization report
-qopt-report-phase=vec           # Vectorization report only
```

### Helping the Compiler Vectorize

```c
// Use restrict keyword
void process(float *restrict a, float *restrict b, int n) {
    for (int i = 0; i < n; i++) {
        a[i] = a[i] + b[i];
    }
}

// Alignment hints
__attribute__((aligned(64))) float data[1024];

// Assume aligned
float *a = __builtin_assume_aligned(ptr, 64);
```

### OpenMP SIMD

```c
#pragma omp simd
for (int i = 0; i < n; i++) {
    a[i] = a[i] + b[i];
}
```

## Link-Time Optimization (LTO)

### Enabling LTO

```bash
# Compile
gcc -O3 -flto -c file1.c
gcc -O3 -flto -c file2.c

# Link
gcc -O3 -flto -o program file1.o file2.o
```

### LTO with Make

```makefile
CFLAGS = -O3 -flto
LDFLAGS = -O3 -flto

program: file1.o file2.o
	$(CC) $(LDFLAGS) -o $@ $^
```

### Parallel LTO

```bash
# Use multiple cores for LTO
gcc -flto=8 -o program *.o
```

## Profile-Guided Optimization (PGO)

### Three-Step Process

#### 1. Instrument
```bash
gcc -O3 -fprofile-generate program.c -o program_instrumented
```

#### 2. Profile
```bash
./program_instrumented < representative_workload.txt
# Generates .gcda files
```

#### 3. Optimize
```bash
gcc -O3 -fprofile-use program.c -o program_optimized
```

### PGO Best Practices

- Use **representative workloads** for profiling
- Profile data should cover common execution paths
- Can improve performance by 10-30%
- Especially effective for:
  - Branch prediction
  - Function inlining
  - Code layout

## Fast Math Optimizations

### -ffast-math Components

```bash
-ffast-math  # Enables all of:
  -fno-math-errno
  -funsafe-math-optimizations
  -ffinite-math-only
  -fno-rounding-math
  -fno-signaling-nans
  -fcx-limited-range
  -fexcess-precision=fast
```

### Safe Fast Math

Use individual flags instead of `-ffast-math`:
```bash
-fno-math-errno           # Usually safe
-fno-trapping-math        # Usually safe
-freciprocal-math         # May reduce precision
```

### When to Avoid Fast Math

- Financial calculations
- Algorithms sensitive to floating-point errors
- Code that relies on IEEE 754 compliance
- NaN or Inf handling is important

## Function Inlining

### Controlling Inlining

```bash
-finline-functions          # Inline small functions (enabled at -O3)
-finline-limit=<n>          # Max size to inline
-fno-inline                 # Disable inlining
```

### Explicit Inlining

```c
inline void small_function() { /* ... */ }
__attribute__((always_inline)) void must_inline() { /* ... */ }
__attribute__((noinline)) void no_inline() { /* ... */ }
```

## Loop Optimization

### Loop Unrolling

```bash
-funroll-loops              # Unroll loops (may increase code size)
-funroll-all-loops          # More aggressive
```

### Loop Optimization Pragmas

```c
#pragma GCC unroll <n>
for (int i = 0; i < 100; i++) {
    // Loop body
}

#pragma GCC optimize("unroll-loops")
```

## Memory Optimizations

### Prefetching

```bash
-fprefetch-loop-arrays      # Automatic prefetching
```

Manual prefetching:
```c
__builtin_prefetch(&data[i + 32]);  // Prefetch ahead
```

### Alignment

```bash
-falign-functions=<n>       # Align functions
-falign-loops=<n>          # Align loop starts
```

## Intel Compiler Specific

### Optimization Flags

```bash
-O3                        # Aggressive optimization
-xHost                     # Optimize for current host
-xCORE-AVX512             # Target AVX-512
-ipo                       # Interprocedural optimization
-qopt-report=5            # Detailed optimization report
```

### Intel MKL Linking

```bash
-mkl=parallel              # Link with parallel MKL
-mkl=sequential            # Link with sequential MKL
```

## Debugging Optimized Code

### Debug-Friendly Optimization

```bash
-Og                        # Optimize but keep debugging sane
-g                         # Debug symbols
-fno-omit-frame-pointer   # Better stack traces
```

### Isolating Optimization Issues

```bash
# Selectively disable optimizations
-fno-tree-vectorize
-fno-inline
-fno-unroll-loops
```

## Compiler-Specific Optimization Guides

### GCC Recommended Flags

```bash
# Development
-Og -g -Wall -Wextra

# Production
-O3 -march=native -flto -fno-math-errno

# Maximum performance (test first!)
-O3 -march=native -flto -ffast-math -funroll-loops
```

### Clang Recommended Flags

```bash
# Development
-Og -g -Wall

# Production
-O3 -march=native -flto

# With profiling
-O3 -march=native -flto -fprofile-instr-generate
```

## Benchmarking Optimizations

### Always Benchmark

```bash
# Build multiple versions
gcc -O2 -o prog_O2 program.c
gcc -O3 -o prog_O3 program.c
gcc -O3 -march=native -o prog_native program.c

# Time them
time ./prog_O2
time ./prog_O3
time ./prog_native
```

### Use Proper Benchmarking Tools

- `perf stat` for performance counters
- `gprof` for profiling
- Application-specific benchmarks

## Common Pitfalls

### Over-Optimization

- `-O3` isn't always faster than `-O2`
- `-march=native` breaks portability
- `-ffast-math` can cause incorrect results

### Ignoring Compile Time

- LTO significantly increases link time
- Balance compile time vs. runtime performance

### Not Testing

- Always test optimized binaries
- Verify numerical accuracy
- Check for performance regressions

## Optimization Checklist

- [ ] Start with `-O2`
- [ ] Profile to find hotspots
- [ ] Try `-O3` and benchmark
- [ ] Add `-march=native` if not distributing
- [ ] Consider LTO for significant projects
- [ ] Use PGO for critical applications
- [ ] Verify correctness after each optimization
- [ ] Document optimization flags in build system

## See Also

- [GCC & LLVM Guide](gcc-llvm.md) - Compiler usage
- [Programming Environments](prgenv.md) - Environment setup
- [Profiling](../profiling-debugging/profiling.md) - Find performance bottlenecks
