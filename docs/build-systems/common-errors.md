# Common Build System Errors

!!! note "TODO"
    * Check for correct module versions.
    * Relevant to NAISS resources. 
    * Loading toolchains before CMake. 
    * Test examples.
    * System specifics.

Solutions to frequent compilation and linking errors on NAISS systems.

## Module-Related Errors

### Error: Command Not Found

```
bash: cmake: command not found
bash: gcc: command not found
```

**Cause:** Required module not loaded.

**Solution:**
```bash
module avail cmake
module load cmake/3.21.0

module avail gcc
module load gcc/11.2.0
```

### Error: Wrong Version Used

**Cause:** Multiple versions loaded or wrong default.

**Solution:**
```bash
module purge
module load gcc/11.2.0
which gcc
gcc --version
```

## CMake Errors

### Error: CMake Version Too Old

```
CMake Error: CMake 3.14 or higher is required.
```

**Solution:**
```bash
module avail cmake
module load cmake/3.21.0
```

Or reduce requirement in CMakeLists.txt:
```cmake
cmake_minimum_required(VERSION 3.10)
```

### Error: Could Not Find Package

```
CMake Error: Could not find a package configuration file for "HDF5"
```

**Cause:** Library not in CMake's search path.

**Solution:**
```bash
# Load the module first
module load hdf5/1.12.1

# Check module sets CMAKE_PREFIX_PATH
echo $CMAKE_PREFIX_PATH

# Or set manually
cmake -DCMAKE_PREFIX_PATH=/path/to/hdf5 ..

# Or set HDF5_ROOT
cmake -DHDF5_ROOT=/path/to/hdf5 ..
```

### Error: No CMAKE_C_COMPILER Found

```
CMake Error: CMAKE_C_COMPILER not set
```

**Solution:**
```bash
# Load compiler module
module load gcc/11.2.0

# Verify
which gcc

# Reconfigure
rm -rf build/*
cmake ..
```

### Error: Compiler Test Failed

```
The C compiler identification is unknown
CMake Error: Compiler test failed
```

**Causes:**
- Missing compiler
- Wrong compiler module
- Library incompatibilities

**Solution:**
```bash
module purge
module load gcc/11.2.0
rm -rf build/*
CC=gcc CXX=g++ cmake ..
```

## Compilation Errors

### Error: Undefined Reference to MPI Functions

```
undefined reference to `MPI_Init'
undefined reference to `MPI_Finalize'
```

**Cause:** MPI library not linked.

**Solution in CMakeLists.txt:**
```cmake
find_package(MPI REQUIRED)
target_link_libraries(myapp MPI::MPI_CXX)
```

Or compile with MPI wrapper:
```bash
mpicxx -o program program.cpp
```

### Error: Cannot Find Header File

```
fatal error: hdf5.h: No such file or directory
```

**Cause:** Include path not set.

**Solution:**
```bash
# Load module
module load hdf5/1.12.1

# Check module sets include paths
module show hdf5/1.12.1
```

In CMakeLists.txt:
```cmake
find_package(HDF5 REQUIRED)
target_include_directories(myapp PRIVATE ${HDF5_INCLUDE_DIRS})
target_link_libraries(myapp ${HDF5_LIBRARIES})
```

### Error: Cannot Find Library

```
/usr/bin/ld: cannot find -lhdf5
```

**Cause:** Library path not set or library not available.

**Solution:**
```bash
# Load module
module load hdf5/1.12.1

# Check library location
module show hdf5/1.12.1
```

In CMakeLists.txt:
```cmake
find_package(HDF5 REQUIRED)
target_link_libraries(myapp ${HDF5_LIBRARIES})
```

## Linking Errors

### Error: Undefined Reference (General)

```
undefined reference to `function_name'
```

**Common Causes:**
1. Missing object file or library
2. Wrong link order
3. Name mangling (C vs C++)
4. Missing library module

**Solutions:**

**Missing library:**
```cmake
target_link_libraries(myapp missing_library)
```

**C/C++ linkage:**
```c
// In header
#ifdef __cplusplus
extern "C" {
#endif

void c_function(void);

#ifdef __cplusplus
}
#endif
```

**Link order (static libraries):**
```cmake
# Dependencies should come after dependents
target_link_libraries(myapp lib_user lib_provider)
```

### Error: Multiple Definition

```
multiple definition of `function_name'
```

**Cause:** Function defined in header without `inline` or `static`.

**Solution:**
```cpp
// Use inline
inline int myfunction() { return 42; }

// Or static
static int myfunction() { return 42; }

// Or move to .cpp file
```

### Error: Version Script Errors

```
warning: version script assignment of 'GLIBC_2.XX' to symbol 'X' failed
```

**Cause:** Binary compiled on newer system than runtime.

**Solution:**
Compile on the target system or use static linking for system libraries.

## OpenMP Errors

### Error: OpenMP Not Found

```
fatal error: omp.h: No such file or directory
```

**Solution:**
```bash
# GCC
gcc -fopenmp program.c

# CMake
find_package(OpenMP REQUIRED)
target_link_libraries(myapp OpenMP::OpenMP_CXX)
```

### Error: Undefined Reference to `GOMP_*`

```
undefined reference to `GOMP_parallel'
```

**Cause:** Missing `-fopenmp` at link time.

**Solution:**
```bash
gcc -fopenmp -o program program.o
```

## Dependency Issues

### Error: ABI Incompatibility

```
undefined symbol: _ZN...
```

**Cause:** Mixing objects compiled with different compilers or versions.

**Solution:**
```bash
# Clean rebuild with consistent compiler
module purge
module load gcc/11.2.0
rm -rf build/*
cmake ..
make clean
make
```

### Error: GLIBC Version Mismatch

```
version `GLIBC_2.34' not found
```

**Cause:** Binary compiled on system with newer glibc.

**Solutions:**
- Compile on target system
- Use containers
- Static linking (increases binary size)

## HDF5-Specific Errors

### Error: HDF5 Header Version Mismatch

```
Warning: HDF5 header version mismatch
```

**Cause:** Headers from one version, library from another.

**Solution:**
```bash
module purge
module load hdf5/1.12.1
rm -rf build/*
cmake ..
```

### Error: Parallel HDF5 Not Found

**Solution:**
```cmake
find_package(HDF5 REQUIRED COMPONENTS C Parallel)
```

Or:
```bash
module load hdf5-parallel/1.12.1
```

## Python Extension Errors

### Error: Python.h Not Found

```
fatal error: Python.h: No such file or directory
```

**Solution:**
```bash
module load python/3.9.5
python3-config --includes  # Check include path
```

### Error: Wrong Python Version

**Solution:**
```bash
# Use specific Python
cmake -DPYTHON_EXECUTABLE=$(which python3) ..
```

## Makefile Errors

### Error: No Rule to Make Target

```
make: *** No rule to make target 'foo.o'
```

**Causes:**
- Missing source file
- Wrong file path
- Typo in Makefile

**Solution:**
Check file exists and Makefile paths are correct.

### Error: Missing Separator

```
Makefile:10: *** missing separator. Stop.
```

**Cause:** Spaces instead of tab before command.

**Solution:**
Use TAB character (not spaces) before commands in Makefile.

## Debugging Strategies

### Verbose Compilation

```bash
# Make
make VERBOSE=1

# CMake
cmake --build . --verbose

# Direct compilation
gcc -v program.c
```

### Check Compiler Configuration

```bash
gcc -v
gcc -dumpspecs
gcc -print-search-dirs
```

### Check Library Availability

```bash
# Find library
find /path/to/search -name "libhdf5*"

# Check library symbols
nm -D /path/to/libhdf5.so | grep H5

# Check library dependencies
ldd /path/to/libhdf5.so
```

### Preprocessor Output

```bash
gcc -E program.c > preprocessed.c
```

### Linker Debugging

```bash
# Verbose linker
gcc -Wl,--verbose program.c

# Trace linker files
gcc -Wl,--trace program.c
```

### CMake Debugging

```bash
# List cache variables
cmake -L .

# Show include paths and defines
cmake --build . --target help

# Debug find_package
cmake --debug-find ..
```

## Quick Fixes Checklist

When build fails:

1. **Check modules:**
   ```bash
   module list
   module purge
   module load required_modules
   ```

2. **Clean and rebuild:**
   ```bash
   make clean
   # or
   rm -rf build/*
   cmake ..
   make
   ```

3. **Verify compiler:**
   ```bash
   which gcc
   gcc --version
   ```

4. **Check paths:**
   ```bash
   echo $PATH
   echo $LD_LIBRARY_PATH
   echo $CMAKE_PREFIX_PATH
   ```

5. **Verbose output:**
   ```bash
   make VERBOSE=1
   ```

## Prevention Tips

- **Document build requirements** (modules, versions)
- **Use consistent environment** (save module collections)
- **Version control CMakeLists.txt** with code
- **Test builds on target system** before production
- **Use explicit module versions** in build scripts
- **Keep build logs** for troubleshooting

## See Also

- [CMake Guide](cmake.md) - CMake best practices
- [Module Pitfalls](../modules/common-pitfalls.md) - Module system issues
- [Compiler Guide](../compilers/gcc-llvm.md) - Compiler usage
