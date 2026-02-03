# Common Module System Pitfalls

!!! note "TODO"
    * Verify all examples on NAISS systems.
    * Add explanatory texts for each pitfall.
    * Include system-specific instructions.
    * Check for correctness.

Learn from these common mistakes when working with modules on NAISS systems.

## Pitfall 1: Forgetting Module Purge

### The Problem

```bash
#!/bin/bash
#SBATCH ...

module load gcc/11.2.0
./my_program
```

If you had different modules loaded interactively, they persist in the job.

### The Solution

Always start with a clean slate:
```bash
#!/bin/bash
#SBATCH ...

module purge
module load gcc/11.2.0
./my_program
```

## Pitfall 2: Using Default Versions

### The Problem

```bash
module load gcc
module load python
```

Default versions can change, breaking your code later.

### The Solution

Always specify versions:
```bash
module load gcc/11.2.0
module load python/3.9.5
```

## Pitfall 3: Ignoring Module Hierarchies

### The Problem

```bash
module load hdf5/1.12.1
# Error: Module not found
```

Some modules only appear after loading their dependencies.

### The Solution

Use `module spider` to find dependencies:
```bash
module spider hdf5/1.12.1
# Output tells you to load gcc and openmpi first

module load gcc/11.2.0
module load openmpi/4.1.1
module load hdf5/1.12.1  # Now it works
```

## Pitfall 4: Module Conflicts

### The Problem

```bash
module load gcc/11.2.0
module load intel/2021.4.0
# Warning: Conflicting compilers!
```

Loading conflicting modules can cause unpredictable behavior.

### The Solution

Unload or purge first:
```bash
module purge
module load intel/2021.4.0
```

Or use `module swap`:
```bash
module swap gcc/11.2.0 intel/2021.4.0
```

## Pitfall 5: Not Checking Loaded Modules

### The Problem

Compilation or execution fails mysteriously, and you don't check what's loaded.

### The Solution

Regularly verify your environment:
```bash
module list
which gcc
echo $LD_LIBRARY_PATH
```

Add to job scripts for debugging:
```bash
#!/bin/bash
#SBATCH ...

module purge
module load gcc/11.2.0

echo "=== Loaded Modules ==="
module list
echo "=== GCC Version ==="
gcc --version
```

## Pitfall 6: Inconsistent Build and Run Environments

### The Problem

Build with one environment:
```bash
module load gcc/11.2.0
make
```

Run with a different environment:
```bash
module purge
./my_program  # May fail or behave strangely
```

### The Solution

Use the same modules for build and run:
```bash
# Save build environment
module load gcc/11.2.0 openmpi/4.1.1
module save my_project

# In job script
module restore my_project
./my_program
```

## Pitfall 7: Not Understanding PATH Changes

### The Problem

After loading modules, programs aren't found or wrong versions run.

### The Solution

Check how modules modify your environment:
```bash
module show gcc/11.2.0
# Shows all environment variable changes

# Verify paths
which gcc
echo $PATH
```

## Pitfall 8: Assuming Modules Work Across Systems

### The Problem

A script works on one NAISS system but fails on another:
```bash
module load gcc/11.2.0  # Version not available on system B
```

### The Solution

Check module availability and adapt:
```bash
module avail gcc  # See available versions

# Or write portable scripts
if module avail gcc/11.2.0 2>&1 | grep -q gcc/11.2.0; then
    module load gcc/11.2.0
elif module avail gcc/10.3.0 2>&1 | grep -q gcc/10.3.0; then
    module load gcc/10.3.0
else
    echo "No suitable GCC version found"
    exit 1
fi
```

## Pitfall 9: Not Reading Module Help

### The Problem

Missing important usage information or special requirements.

### The Solution

Always check module documentation:
```bash
module help gcc/11.2.0
module show gcc/11.2.0
```

Some modules print important messages when loaded - read them!

## Pitfall 10: Mixing System and Module Software

### The Problem

```bash
module load python/3.9.5
pip install numpy  # Installs to wrong location
```

### The Solution

Understand where module software is installed and use appropriate tools:
```bash
module load python/3.9.5
# Use virtual environments
python -m venv myenv
source myenv/bin/activate
pip install numpy
```

## Quick Recovery Commands

When things go wrong:

```bash
# Reset to default modules
module reset

# Start completely fresh
module purge

# See what happened
module list
module show <module_name>

# Find the right module
module spider <software>
module avail <software>
```

## Prevention Checklist

- [ ] Start job scripts with `module purge`
- [ ] Use explicit module versions
- [ ] Check dependencies with `module spider`
- [ ] Verify loaded modules with `module list`
- [ ] Use same modules for build and run
- [ ] Test on target system before production runs
- [ ] Document module requirements

## See Also

- [Basics](basics.md) - Learn fundamental operations
- [Best Practices](best-practices.md) - Follow recommended workflows
- [Cheatsheet](cheatsheet.md) - Quick command reference
