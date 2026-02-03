# Module System Overview

!!! note "TODO"
    * Include system-specific instructions.
    * Add troubleshooting section.
    * Include best practices.
    * Verify all commands on NAISS systems.
    * Add explanatory texts for each section.

The module system is the primary method for managing software environments on NAISS HPC systems. It allows you to dynamically load and unload software packages, compilers, and libraries.

## What are Modules?

Modules provide a mechanism to configure your shell environment for different software packages. They modify environment variables like `PATH`, `LD_LIBRARY_PATH`, and `MANPATH` to make software available.

## Key Concepts

### Loading Modules
Modules must be loaded before you can use the associated software:
```bash
module load gcc/11.2.0
module load openmpi/4.1.1
```

### Module Hierarchy
Many NAISS systems use a hierarchical module system where certain modules only become available after loading prerequisite modules (e.g., compiler modules).

### Module Collections
You can save frequently used module combinations as collections for easy reuse.

## Common Commands

See the [Cheatsheet](cheatsheet.md) for a quick reference of module commands.

## Further Reading

- [Basics](basics.md) - Fundamental module operations
- [Best Practices](best-practices.md) - Recommended workflows
- [Common Pitfalls](common-pitfalls.md) - Avoid these mistakes
- [Cheatsheet](cheatsheet.md) - Quick reference guide
