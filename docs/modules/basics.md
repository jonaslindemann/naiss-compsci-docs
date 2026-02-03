# Module System Basics

!!! note "TODO"
    * Include system-specific instructions.
    * Add troubleshooting section.
    * Include best practices.
    * Verify all commands on NAISS systems.
    * Add explanatory texts for each section.

This guide covers fundamental operations with the module system on NAISS HPC clusters.

## Essential Commands

### Listing Available Modules

To see all available modules:
```bash
module avail
```

To search for specific modules:
```bash
module avail gcc
module spider python
```

### Loading Modules

Load a module to make software available:
```bash
module load gcc/11.2.0
```

Load multiple modules at once:
```bash
module load gcc/11.2.0 openmpi/4.1.1 hdf5/1.12.1
```

### Viewing Loaded Modules

Check which modules are currently loaded:
```bash
module list
```

### Unloading Modules

Unload a specific module:
```bash
module unload gcc
```

Unload all modules:
```bash
module purge
```

### Module Information

Display information about a module:
```bash
module show gcc/11.2.0
module help gcc/11.2.0
```

## Module Hierarchies

On systems using Lmod, modules are organized hierarchically:

1. **Core modules** - Always available (compilers, utilities)
2. **Compiler-dependent modules** - Only visible after loading a compiler
3. **MPI-dependent modules** - Only visible after loading both compiler and MPI

### Example Workflow

```bash
# Start fresh
module purge

# Load compiler (makes compiler-dependent modules available)
module load gcc/11.2.0

# Now MPI modules become visible
module load openmpi/4.1.1

# Now MPI-dependent libraries become visible
module load hdf5/1.12.1
```

## Default Modules

Some modules are loaded by default when you log in. To see them:
```bash
module list
```

To restore default modules after `module purge`:
```bash
module reset
```

## Module Versions

### Specifying Versions

Always specify explicit versions in your job scripts for reproducibility:
```bash
module load gcc/11.2.0  # Good: explicit version
module load gcc         # Avoid: uses default version
```

### Finding Default Version

The default version is usually marked with `(D)` in `module avail` output.

## Tips

- Use `module spider <name>` to find all versions and how to load them
- Module names are case-sensitive
- Modules modify your environment; changes persist until you unload or log out
- Use `module save` to save your current module configuration

## Next Steps

- Learn [Best Practices](best-practices.md) for module usage
- Review [Common Pitfalls](common-pitfalls.md) to avoid issues
- Check the [Cheatsheet](cheatsheet.md) for quick reference
