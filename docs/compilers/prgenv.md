# Programming Environments on NAISS Systems

!!! note "TODO"
    * Clarify which systems use PrgEnv.
    * Add examples for each environment.
    * Explain module loading best practices.
    * Verify commands on NAISS systems.
    * Add explanatory texts.
    * Check for correctness.

Some NAISS HPC systems (particularly those based on Cray architecture like PDC's Dardel) use Programming Environments (PrgEnv) to manage compiler toolchains and associated libraries. Understanding these environments is crucial for successful compilation and linking on these systems.

**Note:** Not all NAISS systems use the Cray Programming Environment. Check your specific system's documentation (e.g., PDC, NSC) for details.

## What are Programming Environments?

A Programming Environment is a collection of:
- A compiler suite (GCC, Intel, Cray, etc.)
- Compatible MPI implementation
- Optimized scientific libraries
- Build tools and utilities

## The Cray Programming Environment (CPE)

On Cray systems like PDC's Dardel, you can load the `cpe` module to enable a specific version of the CPE:

```bash
module load cpe/24.11
```

The `cpe` module ensures that corresponding versions of Cray libraries are loaded, such as `cray-libsci` and `cray-mpich`.

## Available Programming Environments

### Cray Compilers (CCE)
```bash
module load PrgEnv-cray
```

**Default on:** PDC Dardel  
**Use for:**
- System-specific optimizations
- Best performance on Cray hardware
- Scientific computing
- Production runs on Cray systems

### GCC (GNU Compiler Collection)
```bash
module load PrgEnv-gnu
```

**Use for:**
- General-purpose development
- Open-source software
- Maximum portability
- C++20/C++23 features
- Code development and testing

### AMD AOCC Compilers
```bash
module load PrgEnv-aocc
```

**Use for:**
- AMD CPU optimization (e.g., on Dardel's AMD nodes)
- AMD-specific optimizations
- High-performance computing on AMD hardware

### Intel Compilers
```bash
# Available on some systems - check with module avail
module load PrgEnv-intel
# or on non-Cray systems:
module load intel/2021.4.0
```

**Use for:**
- Intel CPU optimization
- Math-intensive code
- Intel-specific optimizations
- Commercial software requiring Intel compilers

**Note:** Intel compilers may not be available on all Cray systems.

## Switching Programming Environments

### Method 1: Module Swap (Recommended on Cray systems)
```bash
module swap PrgEnv-cray PrgEnv-gnu
module swap PrgEnv-gnu PrgEnv-aocc
```

### Method 2: Clean Load
```bash
module purge
module load cpe/24.11
module load PrgEnv-gnu
```

**Recommendation:** Use `module swap` on Cray systems to maintain consistency with loaded libraries. Use clean load for complete reproducibility in scripts.

## Compiler Wrappers

On Cray systems, use compiler wrappers that automatically link MPI and libraries:

| Wrapper | Compiles |
|---------|----------|
| `cc` | C code |
| `CC` | C++ code |
| `ftn` | Fortran code |

The wrapper automatically uses the active programming environment's compiler. For example:

```bash
cc -o myexe.x mycode.c      # C compiler wrapper
CC -o myexe.x mycode.cpp    # C++ compiler wrapper  
ftn -o myexe.x mycode.f90   # Fortran compiler wrapper
```

**Important:** The compiler wrappers automatically:
- Choose the correct compiler based on loaded PrgEnv
- Add target architecture options
- Link to MPI libraries (via `cray-mpich`)
- Link to scientific libraries (via `cray-libsci`)
- No need for `-I`, `-l`, or `-L` flags for Cray-provided libraries

## Environment Variables

Key variables set by programming environments:

```bash
CC          # C compiler
CXX         # C++ compiler
FC          # Fortran compiler
CFLAGS      # C compiler flags
CXXFLAGS    # C++ compiler flags
FFLAGS      # Fortran compiler flags
```

### Check Your Environment
```bash
echo $CC
echo $CFLAGS
module show PrgEnv-gnu
```

## Best Practices

1. **Choose the right environment** for your code and hardware
2. **Be consistent** - use same environment for build and run
3. **Load CPE version** explicitly on Cray systems for reproducibility
4. **Test with multiple compilers** - PDC recommends testing with both `PrgEnv-cray` and `PrgEnv-gnu`
5. **Document requirements** in your README, including system type
6. **Use explicit versions** in production scripts
7. **Check system documentation** - practices vary between NAISS centers (PDC, NSC, etc.)

## Example: Complete Setup

### On Cray Systems (e.g., PDC Dardel)
```bash
#!/bin/bash
# Setup for GCC-based development on Dardel

module load cpe/24.11
module swap PrgEnv-cray PrgEnv-gnu
module load cmake/3.21.0

# Verify
cc --version  # Shows GCC version via wrapper
which cc      # Shows compiler wrapper path
module list   # Check all loaded modules
```

### On Non-Cray Systems
```bash
#!/bin/bash
# Setup for GCC-based development

module purge
module load gcc/11.2.0
module load openmpi/4.1.1
module load cmake/3.21.0

# Verify
which gcc
gcc --version
which mpicc
mpicc --version
```

## Troubleshooting

### Wrong Compiler Used
```bash
module list  # Check loaded environment
which gcc    # Verify compiler path
```

### Library Linking Issues
```bash
module show <library>  # Check library paths
echo $LD_LIBRARY_PATH
```

### ABI Incompatibility
Ensure all components use the same programming environment.

## System-Specific Resources

- **PDC (Dardel):** [Compilers and libraries](https://support.pdc.kth.se/doc/software_development/development/)
- **NSC:** Check [NSC support pages](https://nsc.liu.se/support/) for system-specific guidance

## See Also

- [GCC & LLVM](gcc-llvm.md) - Detailed compiler usage
- [Optimization](optimization.md) - Compiler optimization flags
- [Modules Basics](../modules/basics.md) - Module system fundamentals
- [NAISS Systems](../portability/naiss-systems.md) - Overview of different NAISS systems
