# Module System Cheatsheet

!!! note "TODO"
    * Check for correct module versions.
    * Check for correctness.

Quick reference for module commands on NAISS HPC systems.

## Basic Operations

| Command | Description |
|---------|-------------|
| `module avail` | List all available modules |
| `module avail gcc` | Search for modules matching "gcc" |
| `module list` | Show currently loaded modules |
| `module load <module>` | Load a module |
| `module unload <module>` | Unload a module |
| `module purge` | Unload all modules |
| `module reset` | Reset to default modules |

## Information Commands

| Command | Description |
|---------|-------------|
| `module show <module>` | Display module configuration |
| `module help <module>` | Show module help text |
| `module spider <module>` | Search all modules and show how to load |
| `module whatis <module>` | Brief description of module |

## Advanced Operations

| Command | Description |
|---------|-------------|
| `module swap <old> <new>` | Replace one module with another |
| `module save <name>` | Save current modules as a collection |
| `module restore <name>` | Restore a saved collection |
| `module savelist` | List saved collections |
| `module describe <collection>` | Show modules in a collection |
| `module disable <collection>` | Disable a collection |

## Lmod-Specific Commands

| Command | Description |
|---------|-------------|
| `module spider` | Show all possible modules |
| `module spider <name>` | Find all versions and dependencies |
| `module keyword <keyword>` | Search modules by keyword |
| `module overview` | Show module hierarchy overview |

## Common Usage Patterns

### Load Multiple Modules
```bash
module load gcc/11.2.0 openmpi/4.1.1 hdf5/1.12.1
```

### Check and Load
```bash
module avail gcc/11.2.0 && module load gcc/11.2.0
```

### Clean Slate
```bash
module purge
module load gcc/11.2.0
```

### Switch Compilers
```bash
module swap gcc/11.2.0 intel/2021.4.0
```

### Save Your Setup
```bash
module save my_project
# Later...
module restore my_project
```

## Job Script Template

```bash
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --time=01:00:00
#SBATCH --nodes=1

# Clean environment
module purge

# Load required modules
module load gcc/11.2.0
module load openmpi/4.1.1

# Optional: verify environment
module list

# Run application
./my_program
```

## Environment Variables

Common variables modified by modules:

| Variable | Purpose |
|----------|---------|
| `PATH` | Executable search path |
| `LD_LIBRARY_PATH` | Shared library search path |
| `MANPATH` | Manual page search path |
| `CPATH` | C/C++ header search path |
| `PKG_CONFIG_PATH` | pkg-config search path |
| `MODULEPATH` | Module file search path |

### Check Environment Changes
```bash
module show gcc/11.2.0
```

## Troubleshooting Commands

```bash
# Module not found?
module spider <module>

# Which modules are loaded?
module list

# What does this module do?
module show <module>

# What version is default?
module avail <module>  # Look for (D)

# Find conflicts
module list
# Check for multiple compilers or MPI implementations
```

## Tips

- **Always specify versions** in scripts: `module load gcc/11.2.0`
- **Start job scripts** with `module purge`
- **Use `module spider`** to find hidden modules
- **Save collections** for frequently used setups
- **Module names** are case-sensitive
- **Check dependencies** with `module spider <module>/<version>`

## Quick Diagnostics

```bash
# Full environment check
module list
echo $PATH | tr ':' '\n'
echo $LD_LIBRARY_PATH | tr ':' '\n'
which gcc
gcc --version
```

## System-Specific Notes

### Lmod (most NAISS systems)
- Uses hierarchical modules
- `module spider` is your friend
- Modules appear/disappear based on hierarchy

### Environment Modules (older systems)
- Flat module namespace
- All modules always visible
- May require manual conflict resolution

## See Also

- [Basics](basics.md) - Detailed explanations
- [Best Practices](best-practices.md) - Recommended workflows
- [Common Pitfalls](common-pitfalls.md) - Avoid these mistakes
