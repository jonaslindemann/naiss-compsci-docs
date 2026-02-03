# Profiling on NAISS Systems

!!! note "TODO"
    * Include system-specific profiling instructions.
    * Verify all commands on NAISS systems.
    * Add more explanatory texts for each section.

Guide to profiling and performance analysis on NAISS HPC systems.

## Why Profile?

Profiling helps you:
- Find performance bottlenecks
- Optimize hotspots effectively
- Understand resource usage
- Validate optimization impact
- Scale applications efficiently

**Remember:** Always profile before optimizing!

## Profiling Workflow

1. **Compile with profiling info**
2. **Run application** (with representative workload)
3. **Analyze results**
4. **Optimize bottlenecks**
5. **Repeat**

## Linux perf

### Basic Usage

```bash
# Profile entire program
perf record ./program

# View results
perf report

# Profile with specific events
perf record -e cycles,instructions ./program

# System-wide profiling (requires permissions)
perf record -a -g -- sleep 10
```

### Common Events

```bash
# CPU cycles
perf record -e cycles ./program

# Cache misses
perf record -e cache-misses ./program

# Branch mispredictions
perf record -e branch-misses ./program

# Multiple events
perf record -e cycles,cache-misses,branch-misses ./program
```

### Performance Counters

```bash
# Get statistics
perf stat ./program

# Detailed output
perf stat -d ./program

# Custom events
perf stat -e cycles,instructions,cache-references,cache-misses ./program
```

### Example Output

```
Performance counter stats for './myprogram':

    2,847,329,294      cycles
    4,012,345,678      instructions              #    1.41  insn per cycle
       12,456,789      cache-misses              #    0.45% of all cache refs
    2,755,432,109      cache-references

      1.234567890 seconds time elapsed
```

### Call Graph Profiling

```bash
# Record with call graph
perf record -g ./program

# Report with call graph
perf report -g

# Flame graph
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

## gprof (GNU Profiler)

### Compile and Run

```bash
# Compile with profiling
gcc -pg -o program program.c

# Run program (generates gmon.out)
./program

# View results
gprof program gmon.out

# Save to file
gprof program gmon.out > analysis.txt
```

### Example Output

```
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 60.00      0.03     0.03        1    30.00    30.00  expensive_function
 40.00      0.05     0.02   100000     0.00     0.00  helper_function
```

### Call Graph

```
                                  Call graph

granularity: each sample hit covers 2 byte(s) for 20.00% of 0.05 seconds

index % time    self  children    called     name
[1]    100.0    0.00    0.05                 main [1]
                0.03    0.00       1/1       expensive_function [2]
                0.02    0.00  100000/100000  helper_function [3]
```

## Valgrind Profiling Tools

### Cachegrind (Cache Profiler)

```bash
# Run cachegrind
valgrind --tool=cachegrind ./program

# Annotate source
cg_annotate cachegrind.out.<pid>

# Annotate specific file
cg_annotate cachegrind.out.<pid> source.c
```

### Callgrind (Call Graph Profiler)

```bash
# Run callgrind
valgrind --tool=callgrind ./program

# Analyze
callgrind_annotate callgrind.out.<pid>

# Visualize with KCachegrind (if X11 available)
kcachegrind callgrind.out.<pid>
```

### Massif (Heap Profiler)

```bash
# Run massif
valgrind --tool=massif ./program

# Analyze
ms_print massif.out.<pid>

# Detailed snapshots
valgrind --tool=massif --detailed-freq=1 ./program
```

## Intel VTune (If Available)

```bash
module load vtune

# Hotspot analysis
vtune -collect hotspots -r result_dir -- ./program

# View report
vtune -report hotspots -r result_dir

# Memory access analysis
vtune -collect memory-access -r result_dir -- ./program
```

## Profiling MPI Applications

### mpiP

```bash
# Load module (if available)
module load mpiP

# Link with mpiP
mpicc -o program program.c -lmpiP -lm -lbfd -liberty -lunwind

# Run
mpirun -np 4 ./program

# Creates *.mpiP file with MPI profiling data
```

### Score-P and Scalasca

```bash
# Load modules
module load scorep
module load scalasca

# Instrument code
scorep mpicc -o program program.c

# Profile
scalasca -analyze mpirun -np 8 ./program

# Examine results
scalasca -examine scorep_*
```

### TAU (Tuning and Analysis Utilities)

```bash
module load tau

# Compile with TAU
tau_cc.sh -o program program.c

# Run
mpirun -np 4 ./program

# Analyze
paraprof  # GUI tool
```

## Profiling Python

### cProfile

```python
import cProfile
import pstats

# Profile code
profiler = cProfile.Profile()
profiler.enable()

# Your code here
main()

profiler.disable()

# Print stats
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(20)
```

**Command line:**
```bash
python -m cProfile -o profile.stats script.py
python -m pstats profile.stats
```

### line_profiler

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

### Memory Profiler

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

### py-spy (Sampling Profiler)

```bash
pip install py-spy

# Record profile
py-spy record -o profile.svg -- python script.py

# Top (live view)
py-spy top -- python script.py

# Flame graph
py-spy record -o flamegraph.svg --format flamegraph -- python script.py
```

## GPU Profiling

### NVIDIA nsys (Nsight Systems)

```bash
module load cuda

# Profile GPU application
nsys profile -o report ./gpu_program

# Generate report
nsys stats report.qdrep
```

### NVIDIA ncu (Nsight Compute)

```bash
# Detailed kernel profiling
ncu -o profile ./gpu_program

# View report
ncu-ui profile.ncu-rep
```

### AMD rocprof

```bash
module load rocm

# Profile HIP application
rocprof ./hip_program

# With metrics
rocprof --stats ./hip_program
```

## Flame Graphs

### Generating Flame Graphs

```bash
# Using perf
perf record -g ./program
perf script > out.perf

# Convert to flame graph
git clone https://github.com/brendangregg/FlameGraph
cd FlameGraph
./stackcollapse-perf.pl ../out.perf | ./flamegraph.pl > flamegraph.svg
```

**View:** Open `flamegraph.svg` in web browser

### Interpreting Flame Graphs

- **Width:** Time spent in function
- **Depth:** Call stack depth
- **Color:** Usually random (aids distinction)
- **Hotspots:** Wide plateaus at top

## SLURM Job Profiling

### Example Job Script

```bash
#!/bin/bash
#SBATCH --job-name=profile
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=20

module load perf

# Profile with perf
perf record -g -o perf.data ./program

# Generate report
perf report -i perf.data > perf_report.txt

# Statistics
perf stat -o perf_stats.txt ./program
```

### MPI Profiling Job

```bash
#!/bin/bash
#SBATCH --job-name=mpi_profile
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=20

module load scorep

# Instrumented MPI program
srun ./instrumented_program

# scorep creates profile directories
```

## Profiling Best Practices

### 1. Representative Workload

- Use realistic inputs
- Run long enough for meaningful data
- But not so long it takes forever

### 2. Profile Production Settings

```bash
# Good: Optimize then profile
gcc -O3 -march=native -g -o program program.c

# Bad: Profile debug build
gcc -g -O0 -o program program.c  # Not representative
```

**Note:** `-g` doesn't slow down execution, just adds symbols.

### 3. Focus on Hotspots

- 80/20 rule: 80% time in 20% of code
- Optimize the biggest bottlenecks first
- Diminishing returns on small optimizations

### 4. Profile Multiple Levels

- **High-level:** Which functions are slow?
- **Low-level:** Cache misses, branch mispredictions
- **System-level:** I/O, memory bandwidth

### 5. Before and After

```bash
# Baseline
perf stat ./program_v1 > baseline.txt

# After optimization
perf stat ./program_v2 > optimized.txt

# Compare
diff baseline.txt optimized.txt
```

## Performance Metrics

### CPU Metrics

```bash
perf stat -e cycles,instructions,cache-references,cache-misses,\
branch-instructions,branch-misses ./program
```

**Key ratios:**
- **IPC (Instructions Per Cycle):** Higher is better (ideal: > 1)
- **Cache miss rate:** Lower is better (< 5% is good)
- **Branch miss rate:** Lower is better (< 10% is good)

### Memory Bandwidth

```bash
# L1/L2/L3 cache misses
perf stat -e L1-dcache-load-misses,L1-dcache-loads,\
LLC-load-misses,LLC-loads ./program
```

### Scaling Analysis

```bash
# Weak scaling
for n in 1 2 4 8 16; do
    echo "Cores: $n"
    perf stat -e cycles mpirun -np $n ./program
done

# Strong scaling
for n in 1 2 4 8 16; do
    echo "Cores: $n"
    perf stat -e cycles mpirun -np $n ./program --size=fixed
done
```

## Interpreting Results

### Good Performance Indicators

- High IPC (> 1.0)
- Low cache miss rate (< 5%)
- Low branch misprediction (< 10%)
- CPU-bound (not waiting on I/O)
- Linear scaling with cores

### Performance Problems

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Low IPC (< 0.5) | Memory stalls | Improve cache usage |
| High cache misses | Poor locality | Restructure data access |
| High branch misses | Unpredictable branches | Branchless code |
| Poor scaling | Synchronization | Reduce locks/barriers |
| Lots of minor faults | Page allocation | Pre-allocate memory |

## Example: Complete Workflow

```bash
# 1. Baseline measurement
perf stat ./program > baseline.txt

# 2. Find hotspots
perf record -g ./program
perf report > hotspots.txt

# 3. Detailed analysis of top function
perf record -e cache-misses,branch-misses -g ./program
perf report

# 4. Optimize code
# (make changes based on findings)

# 5. Verify improvement
perf stat ./program_optimized > optimized.txt
diff baseline.txt optimized.txt

# 6. Generate flame graph
perf record -g ./program_optimized
perf script | stackcollapse-perf.pl | flamegraph.pl > after.svg
```

## Advanced Techniques

### Sampling vs Instrumentation

**Sampling (perf, gprof):**
- Low overhead
- Statistical accuracy
- Good for overall view

**Instrumentation (Valgrind, Score-P):**
- Exact counts
- Higher overhead
- Detailed information

### Hardware Performance Counters

```bash
# List available events
perf list

# Profile specific event
perf stat -e cpu-cycles,instructions,L1-dcache-loads,\
L1-dcache-load-misses ./program
```

## Profiling Checklist

- [ ] Compile with `-O3 -g -march=native`
- [ ] Use representative workload
- [ ] Start with high-level profiling (perf stat)
- [ ] Identify hotspots (perf record, gprof)
- [ ] Analyze bottlenecks (cache, branches)
- [ ] Optimize top bottlenecks only
- [ ] Measure improvement
- [ ] Repeat for next bottleneck

## See Also

- [Debugging](debugging.md) - Debugging techniques
- [Optimization](../compilers/optimization.md) - Compiler optimization
- [Python Performance](../python/performance.md) - Python-specific profiling
