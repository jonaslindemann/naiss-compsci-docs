# Module System Best Practices

!!! note "TODO"
    * Include system-specific recommendations.
    * Add explanatory texts for each section.
    * Verify all commands on NAISS systems.
    * Check for correctness.

Follow these recommendations to work effectively with modules on NAISS systems.

## Job Scripts

### Always Specify Explicit Versions

**Do:**
```bash
#!/bin/bash
module load gcc/11.2.0
module load openmpi/4.1.1
module load hdf5/1.12.1
```

**Don't:**
```bash
#!/bin/bash
module load gcc
module load openmpi
module load hdf5
```

**Why:** Default versions can change, breaking reproducibility. Explicit versions ensure your code runs the same way months or years later.

### Start with a Clean Environment

Begin job scripts with `module purge`:
```bash
#!/bin/bash
#SBATCH ...

module purge
module load gcc/11.2.0
module load openmpi/4.1.1

./my_application
```

**Why:** This ensures you don't have conflicting modules from your login environment.

## Interactive Work

### Use Module Collections

Save commonly used module combinations:
```bash
# Load your modules
module load gcc/11.2.0 openmpi/4.1.1 hdf5/1.12.1

# Save as a collection
module save my_project
```

Restore later:
```bash
module restore my_project
```

### List Available Collections
```bash
module savelist
```

### Create Different Collections for Different Projects

```bash
module purge
module load gcc/11.2.0 openmpi/4.1.1
module save project_a

module purge
module load intel/2021.4.0 impi/2021.4.0
module save project_b
```

## Documentation

### Document Module Requirements

In your README or documentation:
```markdown
## Required Modules

This software requires:
- gcc/11.2.0
- openmpi/4.1.1
- hdf5/1.12.1

Load with:
    module load gcc/11.2.0 openmpi/4.1.1 hdf5/1.12.1
```

### Include Module Info in Build Scripts

```bash
#!/bin/bash
# build.sh - Build script for MyProject
# Requires: gcc/11.2.0, cmake/3.21.0

module purge
module load gcc/11.2.0
module load cmake/3.21.0

mkdir -p build
cd build
cmake ..
make -j8
```

## Troubleshooting

### Check for Conflicts

If you encounter issues, check for module conflicts:
```bash
module list  # See what's loaded
module show <module>  # Check what a module does
```

### Use Spider for Dependencies

Find out how to load a module:
```bash
module spider netcdf/4.8.1
```

This will tell you which prerequisite modules need to be loaded first.

## Cross-System Portability

### Use Conditional Loading

For scripts that run on multiple systems:
```bash
# Detect system
case $(hostname) in
  tetralith*)
    module load buildtool-easybuild/4.5.0
    module load gcc/11.2.0
    ;;
  dardel*)
    module load PDC/22.06
    module load gcc/11.2.0
    ;;
esac
```

### Document System-Specific Requirements

Keep notes on which modules are available on which systems.

## Performance Considerations

### Module Switching vs Fresh Load

When changing compilers, prefer:
```bash
module purge
module load new_compiler/version
```

over:
```bash
module swap old_compiler new_compiler
```

**Why:** `purge` + `load` ensures clean dependency resolution.

## Summary

- **Always** use explicit module versions in scripts
- **Always** start job scripts with `module purge`
- Use module collections for frequently used combinations
- Document module requirements in your code
- Check dependencies with `module spider`
- Test on multiple systems if portability is needed

## See Also

- [Basics](basics.md) - Fundamental module operations
- [Common Pitfalls](common-pitfalls.md) - Mistakes to avoid
- [Cheatsheet](cheatsheet.md) - Quick reference
