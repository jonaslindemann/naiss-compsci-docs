# CMake on NAISS Systems

Comprehensive guide to using CMake for building software on NAISS HPC systems.

!!! note "TODO"
    * Check for correct module versions. 
    * Loading toolchains before CMake. 
    * System specifics.

## Introduction

CMake is a cross-platform build system generator widely used for C, C++, and Fortran projects. It generates native build files (Makefiles, Ninja files, etc.) from platform-independent configuration.

## Loading CMake

```bash
module load cmake/3.21.0
```

Check version:
```bash
cmake --version
```

## Basic Usage

### Out-of-Source Build (Recommended)

```bash
# Create and enter build directory
mkdir build
cd build

# Configure
cmake ..

# Build
cmake --build .

# Or use make directly
make -j8
```

### In-Source Build (Not Recommended)

```bash
cmake .
make
```

**Why avoid?** Pollutes source directory with build artifacts.

## CMakeLists.txt Basics

### Minimal Example

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject VERSION 1.0 LANGUAGES C CXX)

# Executable
add_executable(myprogram main.cpp)
```

### Setting C++ Standard

```cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

### Compiler Flags

```cmake
# Set flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# Build-type specific flags
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
```

## Build Types

### Specifying Build Type

```bash
# Debug build
cmake -DCMAKE_BUILD_TYPE=Debug ..

# Release build
cmake -DCMAKE_BUILD_TYPE=Release ..

# With debug info and optimization
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..

# Minimum size release
cmake -DCMAKE_BUILD_TYPE=MinSizeRel ..
```

### Custom Build Type

```cmake
set(CMAKE_CXX_FLAGS_PROFILING "-O2 -g -pg")
```

Then:
```bash
cmake -DCMAKE_BUILD_TYPE=Profiling ..
```

## Compiler Selection

### At Configuration Time

```bash
# Use specific compilers
CC=gcc CXX=g++ cmake ..

# Or with environment modules
module load gcc/11.2.0
cmake -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ ..
```

### In CMakeLists.txt (Before project())

```cmake
set(CMAKE_C_COMPILER "gcc")
set(CMAKE_CXX_COMPILER "g++")
project(MyProject)
```

## Finding and Using Libraries

### Finding Packages

```cmake
# Find MPI
find_package(MPI REQUIRED)
if(MPI_FOUND)
    include_directories(${MPI_INCLUDE_PATH})
    target_link_libraries(myprogram ${MPI_LIBRARIES})
endif()

# Find HDF5
find_package(HDF5 REQUIRED COMPONENTS C)
target_link_libraries(myprogram ${HDF5_LIBRARIES})
target_include_directories(myprogram PUBLIC ${HDF5_INCLUDE_DIRS})

# Find OpenMP
find_package(OpenMP REQUIRED)
target_link_libraries(myprogram OpenMP::OpenMP_CXX)
```

### pkg-config Integration

```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(FFTW REQUIRED fftw3)

target_include_directories(myprogram PUBLIC ${FFTW_INCLUDE_DIRS})
target_link_libraries(myprogram ${FFTW_LIBRARIES})
```

## Modern CMake (Target-Based)

### Creating Targets

```cmake
# Executable
add_executable(myapp main.cpp utils.cpp)

# Library
add_library(mylib STATIC lib.cpp)

# Shared library
add_library(mylib_shared SHARED lib.cpp)
```

### Target Properties

```cmake
# Include directories
target_include_directories(myapp PRIVATE include/)
target_include_directories(myapp PUBLIC include/)

# Compile options
target_compile_options(myapp PRIVATE -Wall -Wextra)

# Compile features
target_compile_features(myapp PRIVATE cxx_std_17)

# Link libraries
target_link_libraries(myapp PRIVATE mylib)
target_link_libraries(myapp PUBLIC pthread)
```

### Visibility: PUBLIC, PRIVATE, INTERFACE

- **PRIVATE**: Used only by this target
- **PUBLIC**: Used by this target and consumers
- **INTERFACE**: Only for consumers, not this target

## MPI Projects

### MPI Example

```cmake
cmake_minimum_required(VERSION 3.14)
project(MPIProject LANGUAGES C CXX)

find_package(MPI REQUIRED)

add_executable(mpi_program main.cpp)
target_link_libraries(mpi_program MPI::MPI_CXX)
```

Build with MPI compilers:
```bash
module load openmpi/4.1.1
cmake -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpicxx ..
make
```

## OpenMP Projects

```cmake
find_package(OpenMP REQUIRED)

add_executable(omp_program main.cpp)
target_link_libraries(omp_program OpenMP::OpenMP_CXX)
```

## HDF5 Projects

```cmake
find_package(HDF5 REQUIRED COMPONENTS C CXX)

add_executable(hdf5_app main.cpp)
target_link_libraries(hdf5_app ${HDF5_LIBRARIES})
target_include_directories(hdf5_app PUBLIC ${HDF5_INCLUDE_DIRS})
```

## Installation

### Install Rules

```cmake
# Install executable
install(TARGETS myprogram DESTINATION bin)

# Install library
install(TARGETS mylib
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib)

# Install headers
install(FILES myheader.h DESTINATION include)

# Install directory
install(DIRECTORY include/ DESTINATION include)
```

### Installing

```bash
cmake -DCMAKE_INSTALL_PREFIX=/path/to/install ..
make
make install
```

## Testing with CTest

### Enabling Tests

```cmake
enable_testing()

add_executable(test_app test.cpp)
add_test(NAME MyTest COMMAND test_app)
```

### Running Tests

```bash
make test
# or
ctest
ctest --verbose
ctest --output-on-failure
```

## Configuration Options

### Adding Options

```cmake
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(ENABLE_OPENMP "Enable OpenMP" ON)

if(ENABLE_OPENMP)
    find_package(OpenMP REQUIRED)
    target_link_libraries(myapp OpenMP::OpenMP_CXX)
endif()
```

### Using Options

```bash
cmake -DBUILD_SHARED_LIBS=ON -DENABLE_OPENMP=OFF ..
```

## Generator Selection

```bash
# Unix Makefiles (default on Linux)
cmake -G "Unix Makefiles" ..

# Ninja (faster builds)
cmake -G "Ninja" ..
make  # or ninja
```

## Parallel Builds

```bash
# With make
make -j8

# With CMake build command (recommended)
cmake --build . --parallel 8
cmake --build . -j 8
```

## Verbose Output

```bash
# See compilation commands
make VERBOSE=1

# Or with CMake
cmake --build . --verbose
```

## Common Patterns

### Finding Libraries Manually

```cmake
find_path(MYLIB_INCLUDE_DIR mylib.h
          PATHS /usr/include /usr/local/include)
find_library(MYLIB_LIBRARY NAMES mylib
             PATHS /usr/lib /usr/local/lib)

if(MYLIB_INCLUDE_DIR AND MYLIB_LIBRARY)
    set(MYLIB_FOUND TRUE)
endif()

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(MYLIB DEFAULT_MSG
                                  MYLIB_LIBRARY MYLIB_INCLUDE_DIR)
```

### Subdirectories

```cmake
# Top-level CMakeLists.txt
add_subdirectory(src)
add_subdirectory(tests)

# src/CMakeLists.txt
add_executable(myapp main.cpp)
```

### Conditional Compilation

```cmake
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(myapp PRIVATE -march=native)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    target_compile_options(myapp PRIVATE -xHost)
endif()
```

## Example: Complete CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyHPCApp VERSION 1.0 LANGUAGES C CXX)

# C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Options
option(ENABLE_MPI "Enable MPI support" ON)
option(ENABLE_OPENMP "Enable OpenMP support" ON)

# Find packages
if(ENABLE_MPI)
    find_package(MPI REQUIRED)
endif()

if(ENABLE_OPENMP)
    find_package(OpenMP REQUIRED)
endif()

find_package(HDF5 REQUIRED COMPONENTS C CXX)

# Executable
add_executable(myapp
    src/main.cpp
    src/solver.cpp
    src/io.cpp
)

# Include directories
target_include_directories(myapp PRIVATE include)

# Link libraries
target_link_libraries(myapp PRIVATE ${HDF5_LIBRARIES})

if(ENABLE_MPI)
    target_link_libraries(myapp PRIVATE MPI::MPI_CXX)
endif()

if(ENABLE_OPENMP)
    target_link_libraries(myapp PRIVATE OpenMP::OpenMP_CXX)
endif()

# Compiler flags
target_compile_options(myapp PRIVATE
    $<$<CONFIG:Debug>:-g -O0 -Wall -Wextra>
    $<$<CONFIG:Release>:-O3 -DNDEBUG>
)

# Installation
install(TARGETS myapp DESTINATION bin)

# Testing
enable_testing()
add_subdirectory(tests)
```

## Troubleshooting

### CMake Can't Find Package

```bash
# Check CMAKE_PREFIX_PATH
echo $CMAKE_PREFIX_PATH

# Set manually
cmake -DCMAKE_PREFIX_PATH=/path/to/library ..

# Or in CMakeLists.txt
set(CMAKE_PREFIX_PATH /path/to/library)
```

### Wrong Compiler Used

```bash
# Clean and reconfigure
rm -rf build/*
CC=gcc CXX=g++ cmake ..
```

### Verbose FindPackage Output

```cmake
set(CMAKE_FIND_DEBUG_MODE TRUE)
find_package(HDF5)
set(CMAKE_FIND_DEBUG_MODE FALSE)
```

### Check Configuration

```bash
cmake -L ..          # List cache variables
cmake -LA ..         # List all cache variables
cmake -LAH ..        # List all with help text
```

## Best Practices

1. **Use out-of-source builds**
2. **Specify minimum CMake version** appropriately
3. **Use modern target-based commands**
4. **Set explicit C/C++ standards**
5. **Document required packages in README**
6. **Use generator expressions** for build-type-specific settings
7. **Don't hardcode paths** - use find_package()
8. **Version your CMakeLists.txt** with your code

## See Also

- [Common Errors](common-errors.md) - Troubleshooting build issues
- [GCC & LLVM](../compilers/gcc-llvm.md) - Compiler selection
- [Modules](../modules/index.md) - Loading required modules
