# Python on NAISS Systems

This section collects best practices for running Python projects on NAISS HPC systems. It is aimed at researchers and developers who need dependable environments, reproducible jobs, and good performance without turning every project into a packaging exercise.

## Start Here

For most projects, use this baseline workflow:

```bash
module purge
module load Python/3.13.5

python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Then run the same module and environment activation commands in your batch job before launching the application.

## Guide Structure

- [Environments](environments.md): choose Python modules, create virtual environments, manage dependencies, and connect environments to batch jobs.
- [Performance](performance.md): identify bottlenecks, use NumPy and other optimized libraries effectively, reduce memory use, and choose the right parallel pattern.
- [MPI for Python](mpi-python.md): use `mpi4py` for distributed-memory workloads and run MPI Python programs through SLURM.

## Core Practices

### Prefer Project Environments

Create a separate environment for each project. This avoids dependency conflicts and gives collaborators a concrete record of what was installed.

### Keep Jobs Reproducible

Batch jobs should load modules explicitly and activate the environment from a stable path. Avoid relying on whatever shell state happened to be active when the job was submitted.

### Install Before Running at Scale

Do package installation in an interactive session or setup job. Compute jobs should spend their allocation on computation, not downloading packages or compiling dependencies.

### Profile Before Optimizing

Measure first with `cProfile`, line profilers, memory profilers, or application-specific timing. The best optimization is often replacing a Python loop with a library operation, changing an I/O format, or reducing unnecessary array copies.

### Match Parallelism to the Problem

Use vectorized libraries and threaded numerical kernels where possible. Use multiprocessing for independent CPU-bound tasks on one node, and use `mpi4py` when work must span nodes.
