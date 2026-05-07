# Python Best Practice Guide

!!! info
    This is a living guide for Python development on NAISS systems. Please contribute corrections, examples, and system-specific notes as the guidance improves.

Welcome to the Python best practice guide for NAISS HPC systems. The guide focuses on practical, reproducible Python workflows for scientific computing: choosing a Python module, managing project environments, installing packages, improving performance, and scaling Python programs with MPI.

## Overview

Python is often the fastest way to express scientific ideas, but HPC systems add constraints that are easy to miss: module environments, compiled dependencies, shared filesystems, batch jobs, and distributed execution. These pages collect patterns that make Python projects easier to run, reproduce, debug, and scale on NAISS resources.

## What You'll Find

- **[Python overview](python/index.md)**: Recommended workflow for Python projects on NAISS
- **[Environments](python/environments.md)**: Create isolated, reproducible Python environments
- **[Performance](python/performance.md)**: Profile code, use optimized libraries, and reduce memory pressure
- **[MPI for Python](python/mpi-python.md)**: Run distributed Python applications with `mpi4py`

## Recommended Workflow

1. Load a suitable Python module for the system you are using.
2. Create one virtual environment per project.
3. Record dependencies in a requirements file or environment file.
4. Install and test packages before submitting batch jobs.
5. Profile before optimizing.
6. Scale out with multiprocessing, threaded libraries, or MPI only after the serial workflow is correct.

## Contributing

These guides are continuously updated to reflect Python packaging, performance, and NAISS system changes. If a command behaves differently on a specific system, add the system name and a short tested example.

---

*Last updated: May 2026*
