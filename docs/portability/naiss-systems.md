# NAISS Systems Overview

!!! note "TODO"
    * Check information for accuracy.
    * Clean up.

Guide to writing portable code across NAISS (National Academic Infrastructure for Supercomputing in Sweden) HPC systems.

## NAISS Systems

NAISS operates several major HPC systems with different architectures and capabilities.

### Major Systems (as of 2026)

#### Dardel (PDC, KTH)
- **Architecture:** HPE Cray EX
- **CPU:** AMD EPYC (Zen 2/3)
- **Accelerators:** AMD MI250X GPUs
- **Interconnect:** Cray Slingshot
- **Special features:** Large GPU partition

#### Tetralith (NSC, Linköping)
- **Architecture:** Cluster
- **CPU:** Intel Xeon (Broadwell/Cascade Lake)
- **Interconnect:** InfiniBand
- **Special features:** Large memory nodes

#### Alvis (C3SE, Chalmers)
- **Architecture:** GPU cluster
- **CPU:** AMD EPYC / Intel Xeon
- **Accelerators:** NVIDIA A100, A40, V100
- **Special features:** AI/ML focused

#### Rackham (UPPMAX, Uppsala)
- **Architecture:** Cluster
- **CPU:** Intel Xeon
- **Special features:** Bioinformatics applications

**Note:** System specifications may change. Always check current documentation.

## Writing Portable Code

### 1. Use Standard Languages

```c
// Good: ISO C standard
#include <stdio.h>
#include <stdlib.h>

int main() {
    printf("Hello, NAISS!\n");
    return 0;
}

// Avoid: Compiler-specific extensions (when possible)
#include <malloc.h>  // Non-standard
```

### 2. Avoid Architecture-Specific Code

```c
// Bad: Intel-specific intrinsics
#include <immintrin.h>
__m256d vec = _mm256_load_pd(data);

// Good: Let compiler vectorize
for (int i = 0; i < n; i++) {
    result[i] = data[i] * 2.0;
}

// Or use portable SIMD library
#include <Vc/Vc>  // Portable vectorization
```

### 3. Conditional Compilation for Optimizations

```c
void compute_array(double *data, int n) {
    #if defined(__AVX512F__)
        // AVX-512 optimized version
        compute_avx512(data, n);
    #elif defined(__AVX2__)
        // AVX2 optimized version
        compute_avx2(data, n);
    #else
        // Generic version
        for (int i = 0; i < n; i++) {
            data[i] = data[i] * 2.0;
        }
    #endif
}
```

### 4. Use Build System for Portability

**CMakeLists.txt:**
```cmake
cmake_minimum_required(VERSION 3.14)
project(PortableApp)

# Detect architecture
if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
    # x86 specific flags
    set(ARCH_FLAGS "-march=native")
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
    # ARM specific flags
    set(ARCH_FLAGS "-mcpu=native")
endif()

# Detect compiler
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ARCH_FLAGS}")
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -xHost")
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ARCH_FLAGS}")
endif()
```

## Module System Differences

### Lmod vs Environment Modules

Most NAISS systems use Lmod, but approaches vary:

```bash
# Portable module loading function
load_module() {
    if command -v module > /dev/null; then
        module load "$@"
    else
        echo "Module system not available"
        exit 1
    fi
}

# Try to load specific version, fall back to default
try_load_gcc() {
    if module avail gcc/11.2.0 2>&1 | grep -q gcc/11.2.0; then
        module load gcc/11.2.0
    elif module avail gcc/11 2>&1 | grep -q "gcc/11"; then
        module load gcc/11
    else
        module load gcc
        echo "Warning: Using default GCC version"
    fi
}
```

### System-Specific Module Names

Create environment detection:

```bash
#!/bin/bash
# detect_system.sh

case $(hostname) in
    *dardel*)
        SYSTEM="dardel"
        module load PDC/22.06
        module load gcc/11.2.0
        ;;
    *tetralith*)
        SYSTEM="tetralith"
        module load buildtool-easybuild/4.5.0
        module load gcc/11.2.0
        ;;
    *alvis*)
        SYSTEM="alvis"
        module load GCC/11.2.0
        ;;
    *kebnekaise*)
        SYSTEM="kebnekaise"
        module load GCC/11.2.0
        ;;
    *)
        SYSTEM="unknown"
        echo "Unknown system, using defaults"
        module load gcc
        ;;
esac

export NAISS_SYSTEM=$SYSTEM
```

## MPI Portability

### Use MPI Standard Features

```c
#include <mpi.h>

int main(int argc, char **argv) {
    MPI_Init(&argc, &argv);
    
    int rank, size;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    
    // Standard MPI code works everywhere
    
    MPI_Finalize();
    return 0;
}
```

### Compile with MPI Wrappers

```bash
# Works on all systems
mpicc -o program program.c
mpicxx -o program program.cpp
mpif90 -o program program.f90
```

### System-Specific MPI

```bash
# Detect and load appropriate MPI
case $NAISS_SYSTEM in
    dardel)
        module load cray-mpich
        ;;
    tetralith)
        module load buildenv-intel/2018a-eb
        module load impi/2018.1.163
        ;;
    *)
        module load openmpi/4.1.1
        ;;
esac
```

## File System Differences

### Portable Path Handling

```python
import os
from pathlib import Path

# Bad: Hardcoded paths
data_dir = "/proj/myproject/data"

# Good: Environment-based
data_dir = os.getenv('PROJECT_DATA', '/default/path')

# Better: Detect common locations
def find_project_dir(project_name):
    candidates = [
        Path(f"/proj/{project_name}"),
        Path(f"/cfs/klemming/projects/{project_name}"),
        Path(f"/mimer/NOBACKUP/groups/{project_name}"),
        Path(f"~/proj/{project_name}").expanduser(),
    ]
    
    for path in candidates:
        if path.exists():
            return path
    
    raise FileNotFoundError(f"Project {project_name} not found")
```

### Storage Tiers

```bash
# Generic approach
# $HOME - permanent, small, backed up
# $PROJECT - permanent, large, sometimes backed up  
# $SCRATCH - temporary, very large, not backed up

# Example usage
cp input_data $SCRATCH/
cd $SCRATCH
./process_data
cp results $PROJECT/results/
```

## Scheduler Differences

### Portable Job Scripts

```bash
#!/bin/bash
#SBATCH --job-name=portable_job
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=20

# Detect system
source $(dirname $0)/detect_system.sh

# Load modules based on system
case $NAISS_SYSTEM in
    dardel)
        module load PDC/22.06
        module load gcc/11.2.0
        CORES_PER_NODE=128
        ;;
    tetralith)
        module load buildtool-easybuild/4.5.0
        module load gcc/11.2.0
        CORES_PER_NODE=32
        ;;
    *)
        module load gcc
        CORES_PER_NODE=20
        ;;
esac

# Run application
srun ./my_application
```

### Core Count Portability

```bash
# Auto-detect cores per node
if [ -n "$SLURM_JOB_CPUS_PER_NODE" ]; then
    CORES=$SLURM_JOB_CPUS_PER_NODE
elif [ -f /proc/cpuinfo ]; then
    CORES=$(grep -c processor /proc/cpuinfo)
else
    CORES=1
fi

export OMP_NUM_THREADS=$CORES
```

## Compiler Portability

### Feature Detection

```c
// config.h - Generate with CMake or autoconf
#ifndef CONFIG_H
#define CONFIG_H

// Compiler detection
#if defined(__GNUC__)
    #define COMPILER_GCC
#elif defined(__INTEL_COMPILER)
    #define COMPILER_INTEL
#elif defined(__clang__)
    #define COMPILER_CLANG
#endif

// Feature detection
#ifdef __AVX2__
    #define HAVE_AVX2
#endif

#ifdef __AVX512F__
    #define HAVE_AVX512
#endif

#endif // CONFIG_H
```

### Using Feature Detection

```c
#include "config.h"

void compute(double *data, int n) {
    #if defined(HAVE_AVX512) && defined(COMPILER_INTEL)
        compute_avx512_intel(data, n);
    #elif defined(HAVE_AVX2)
        compute_avx2(data, n);
    #else
        compute_generic(data, n);
    #endif
}
```

## Testing Portability

### Multi-System Test Script

```bash
#!/bin/bash
# test_portability.sh

SYSTEMS="dardel tetralith alvis"

for system in $SYSTEMS; do
    echo "Testing on $system..."
    
    # Submit test job
    ssh $system.naiss.se "cd $PROJECT/myapp && sbatch test_job.sh"
    
    # Wait and check results
    # (implementation depends on workflow)
done
```

### Continuous Integration

Use CI/CD to test on multiple systems:

```yaml
# .github/workflows/naiss-test.yml
name: NAISS Multi-System Test

on: [push]

jobs:
  test-dardel:
    runs-on: self-hosted-dardel
    steps:
      - uses: actions/checkout@v2
      - name: Build and test
        run: |
          module load gcc/11.2.0
          mkdir build && cd build
          cmake ..
          make
          make test
  
  test-tetralith:
    runs-on: self-hosted-tetralith
    # Similar steps...
```

## Best Practices Summary

### DO:
- ✅ Use standard C/C++/Fortran
- ✅ Use portable build systems (CMake, Autotools)
- ✅ Abstract system-specific code
- ✅ Use environment variables for paths
- ✅ Test on multiple systems
- ✅ Document system requirements
- ✅ Use MPI wrappers (mpicc, mpicxx)
- ✅ Detect features at compile time
- ✅ Version control everything

### DON'T:
- ❌ Hardcode paths
- ❌ Assume specific architecture
- ❌ Use vendor-specific extensions without guards
- ❌ Rely on specific module versions
- ❌ Assume specific file system layout
- ❌ Use system-specific schedulers features only
- ❌ Mix compiler/MPI implementations

## Example: Fully Portable Application

**Directory structure:**
```
myapp/
├── CMakeLists.txt
├── README.md
├── src/
│   ├── main.c
│   └── compute.c
├── include/
│   └── config.h.in
├── scripts/
│   ├── detect_system.sh
│   └── submit_job.sh
└── tests/
    └── test_compute.c
```

**CMakeLists.txt:**
```cmake
cmake_minimum_required(VERSION 3.14)
project(PortableApp VERSION 1.0 LANGUAGES C)

# Find MPI
find_package(MPI REQUIRED)

# Executable
add_executable(myapp src/main.c src/compute.c)
target_link_libraries(myapp MPI::MPI_C)

# System detection
site_name(HOSTNAME)
if(HOSTNAME MATCHES "dardel")
    set(SYSTEM_NAME "dardel")
elseif(HOSTNAME MATCHES "tetralith")
    set(SYSTEM_NAME "tetralith")
else()
    set(SYSTEM_NAME "generic")
endif()

# Configure header
configure_file(include/config.h.in include/config.h)
target_include_directories(myapp PRIVATE ${CMAKE_BINARY_DIR}/include)

# Tests
enable_testing()
add_subdirectory(tests)
```

**submit_job.sh:**
```bash
#!/bin/bash
# Portable job submission

# Detect system
source $(dirname $0)/detect_system.sh

# Submit with system-specific options
case $NAISS_SYSTEM in
    dardel)
        sbatch --partition=main --account=your-account-dardel job.sh
        ;;
    tetralith)
        sbatch --account=your-account-tetralith job.sh
        ;;
    *)
        sbatch job.sh
        ;;
esac
```

## Documentation

Always document:
- **Tested systems:** Which NAISS systems work
- **Module requirements:** Required modules per system
- **Known limitations:** System-specific issues
- **Build instructions:** How to build on each system
- **Performance notes:** Expected performance differences

## See Also

- [Modules](../modules/index.md) - Module system usage
- [Compilers](../compilers/prgenv.md) - Compiler environments
- [CMake](../build-systems/cmake.md) - Portable build configuration
