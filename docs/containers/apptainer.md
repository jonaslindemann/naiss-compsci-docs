# Apptainer (Singularity) on NAISS Systems

!!! note "TODO"
    * Verify all commands on NAISS systems.
    * Add explanatory texts for each section.
    * Include system-specific instructions.
    * Check for correctness.
    * Add troubleshooting section.
    * Include best practices.

Guide to using Apptainer (formerly Singularity) for containerized applications on NAISS HPC systems.

## Introduction

Apptainer is a container platform designed for HPC environments. It allows you to:
- Run applications in isolated, reproducible environments
- Use software not available as modules
- Ensure consistent environments across systems
- Package complex software stacks

## Why Apptainer (not Docker)?

Docker requires root privileges, which is not allowed on HPC systems. Apptainer:
- Runs without root privileges
- Integrates with HPC schedulers
- Supports MPI and GPU
- Can convert Docker images
- No daemon required

## Loading Apptainer

```bash
module avail apptainer
module load apptainer/1.1.0
apptainer --version
```

## Running Containers

### From SyLabs Library

```bash
# Run a command in a container
apptainer exec library://alpine cat /etc/os-release

# Interactive shell
apptainer shell library://ubuntu

# Run container's default command
apptainer run library://alpine
```

### From Docker Hub

```bash
# Run Docker container
apptainer exec docker://ubuntu:22.04 cat /etc/os-release

# Pull and convert to SIF
apptainer pull docker://ubuntu:22.04
# Creates ubuntu_22.04.sif

# Run the SIF file
apptainer exec ubuntu_22.04.sif cat /etc/os-release
```

### From Local SIF File

```bash
# Execute command
apptainer exec container.sif /path/to/command

# Interactive shell
apptainer shell container.sif

# Run default runscript
apptainer run container.sif
```

## Building Containers

### From Docker Image

```bash
# Pull and convert
apptainer pull docker://tensorflow/tensorflow:latest

# Creates tensorflow_latest.sif
```

### From Definition File

**example.def:**
```bash
Bootstrap: docker
From: ubuntu:22.04

%post
    apt-get update
    apt-get install -y python3 python3-pip
    pip3 install numpy scipy

%runscript
    python3 "$@"

%labels
    Author Your Name
    Version 1.0

%help
    This container runs Python 3 with NumPy and SciPy.
    Usage: apptainer run container.sif script.py
```

**Build:**
```bash
# Build locally (requires root/fakeroot)
apptainer build --fakeroot mycontainer.sif example.def

# Or build on build server
# (method varies by NAISS system - check documentation)
```

### Multi-Stage Build Example

```bash
Bootstrap: docker
From: ubuntu:22.04

%files
    requirements.txt /opt/

%post
    export DEBIAN_FRONTEND=noninteractive
    apt-get update && apt-get install -y \
        python3 \
        python3-pip \
        gcc \
        g++ \
        && rm -rf /var/lib/apt/lists/*
    
    pip3 install -r /opt/requirements.txt

%environment
    export PYTHONPATH=/opt/app:$PYTHONPATH

%runscript
    python3 /opt/app/main.py "$@"
```

## Binding Directories

By default, Apptainer binds:
- Current directory
- Home directory
- `/tmp`

### Bind Additional Directories

```bash
# Single bind
apptainer exec --bind /proj:/proj container.sif ls /proj

# Multiple binds
apptainer exec \
    --bind /proj:/proj \
    --bind /scratch:/scratch \
    container.sif /path/to/command

# Bind to different path inside
apptainer exec --bind /host/path:/container/path container.sif ls /container/path
```

### In SLURM Job

```bash
#!/bin/bash
#SBATCH --job-name=container_job

module load apptainer

# Bind project and scratch directories
apptainer exec \
    --bind /proj/myproject:/data \
    --bind $TMPDIR:/tmp \
    mycontainer.sif python /data/script.py
```

## MPI Applications

### MPI Hybrid Model

Use host MPI with container application (recommended):

```bash
# Load MPI module
module load openmpi/4.1.1

# Run with host MPI
mpirun -np 4 apptainer exec container.sif /path/to/mpi_app
```

**Definition file for MPI app:**
```bash
Bootstrap: docker
From: ubuntu:22.04

%post
    apt-get update
    apt-get install -y gcc g++ make
    # Don't install MPI in container - use host MPI

%files
    mpi_program.c /opt/

%post
    # Build will happen with host MPI
    # Or include pre-built binary
```

**Compile with host MPI:**
```bash
module load openmpi/4.1.1
mpicc -o mpi_program mpi_program.c
```

**Run:**
```bash
mpirun -np 8 apptainer exec \
    --bind /proj:/proj \
    container.sif /proj/mpi_program
```

### SLURM MPI Job

```bash
#!/bin/bash
#SBATCH --job-name=mpi_container
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=20

module load apptainer
module load openmpi/4.1.1

srun apptainer exec \
    --bind /proj:/proj \
    container.sif /proj/mpi_application
```

## GPU Applications

### NVIDIA GPUs

```bash
# Add --nv flag for NVIDIA GPU support
apptainer exec --nv container.sif nvidia-smi

# In job script
apptainer exec --nv \
    tensorflow_gpu.sif \
    python train_model.py
```

**Definition file for GPU app:**
```bash
Bootstrap: docker
From: nvidia/cuda:11.8.0-base-ubuntu22.04

%post
    apt-get update
    apt-get install -y python3-pip
    pip3 install tensorflow-gpu torch

%runscript
    python3 "$@"
```

### AMD GPUs

```bash
# Add --rocm flag for AMD GPUs
apptainer exec --rocm container.sif /path/to/app
```

## Python in Containers

### Simple Python Container

```bash
Bootstrap: docker
From: python:3.9

%files
    requirements.txt /opt/

%post
    pip install --no-cache-dir -r /opt/requirements.txt

%environment
    export PYTHONUNBUFFERED=1

%runscript
    python "$@"
```

### Scientific Python Container

```bash
Bootstrap: docker
From: continuumio/miniconda3

%post
    conda install -y numpy scipy pandas matplotlib scikit-learn
    conda clean -ay

%runscript
    python "$@"
```

### Using Container Python

```bash
# Run script
apptainer exec python_container.sif python script.py

# Interactive
apptainer exec python_container.sif python

# Jupyter notebook
apptainer exec python_container.sif jupyter notebook --no-browser
```

## Managing Container Images

### Pull Images

```bash
# From Docker Hub
apptainer pull docker://ubuntu:22.04

# From SyLabs Library
apptainer pull library://alpine

# Custom registry
apptainer pull docker://myregistry.com/myimage:tag
```

### Inspect Images

```bash
# Show metadata
apptainer inspect container.sif

# Show definition
apptainer inspect --deffile container.sif

# Show runscript
apptainer inspect --runscript container.sif

# Show environment
apptainer inspect --environment container.sif
```

### Cache Management

```bash
# Show cache location
echo $APPTAINER_CACHEDIR

# Clean cache
apptainer cache clean

# Clean specific types
apptainer cache clean --type=blob
apptainer cache clean --type=library
```

## Environment Variables

### Passing Environment Variables

```bash
# Single variable
apptainer exec --env VAR=value container.sif env

# Multiple variables
apptainer exec \
    --env VAR1=value1 \
    --env VAR2=value2 \
    container.sif env

# From file
apptainer exec --env-file vars.env container.sif env
```

### Cleaning Environment

```bash
# Start with clean environment
apptainer exec --cleanenv container.sif env

# Clean env + add variables
apptainer exec --cleanenv --env PATH=/custom/path container.sif env
```

## Working Directory

```bash
# Set working directory
apptainer exec --pwd /path/in/container container.sif pwd

# Default: current directory is bound and set as pwd
```

## Writable Containers

### Overlay

```bash
# Create overlay
apptainer overlay create --size 1024 overlay.img

# Use overlay
apptainer exec --overlay overlay.img container.sif touch /new_file

# Overlay persists changes
```

### Sandbox (Directory)

```bash
# Build as directory (writable)
apptainer build --sandbox mycontainer/ docker://ubuntu

# Modify
apptainer exec --writable mycontainer/ apt-get install -y vim

# Convert to SIF
apptainer build mycontainer.sif mycontainer/
```

## Best Practices

### 1. Use SIF Files in Production

```bash
# Pull once, use many times
apptainer pull docker://myimage:tag
# Creates myimage_tag.sif

# Use in jobs
apptainer exec myimage_tag.sif /path/to/app
```

### 2. Minimize Container Size

```bash
%post
    apt-get update && apt-get install -y package \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*
```

### 3. Don't Include Data in Container

```bash
# Bad: Large data in container
%files
    large_dataset.tar.gz /data/

# Good: Bind mount data
apptainer exec --bind /proj/data:/data container.sif process /data
```

### 4. Version Your Containers

```bash
%labels
    Version 1.2.3
    BuildDate 2026-02-02

# Tag images
apptainer build myapp_v1.2.3.sif myapp.def
```

### 5. Document Usage

```bash
%help
    This container runs MyApp version 1.2.3
    
    Usage:
        apptainer run container.sif input.txt output.txt
    
    Required binds:
        --bind /proj:/proj (project data)
        --bind /scratch:/scratch (temporary files)
```

## Complete Example: ML Workflow

**ml_pipeline.def:**
```bash
Bootstrap: docker
From: python:3.9-slim

%files
    requirements.txt /opt/

%post
    pip install --no-cache-dir -r /opt/requirements.txt

%environment
    export PYTHONUNBUFFERED=1
    export OMP_NUM_THREADS=1

%runscript
    python /app/train.py "$@"

%help
    Machine Learning Pipeline Container
    
    Usage: apptainer run --bind /data:/data ml_pipeline.sif
    
    Environment variables:
        OMP_NUM_THREADS - OpenMP threads (default: 1)
```

**job.sh:**
```bash
#!/bin/bash
#SBATCH --job-name=ml_training
#SBATCH --time=04:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G

module load apptainer

# Set OpenMP threads
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

# Run training
apptainer run \
    --bind /proj/myproject:/data \
    --bind $TMPDIR:/tmp \
    ml_pipeline.sif /data/config.yaml
```

## Troubleshooting

### Permission Denied

```bash
# If home directory binding causes issues
apptainer exec --no-home container.sif command
```

### Library Not Found

```bash
# Check library paths
apptainer exec container.sif ldd /path/to/binary

# May need to bind library locations
apptainer exec --bind /lib64:/lib64 container.sif command
```

### MPI Version Mismatch

Use host MPI (hybrid model) to avoid version issues.

### Out of Space

```bash
# Check cache
apptainer cache list

# Clean cache
apptainer cache clean

# Use different cache location
export APPTAINER_CACHEDIR=/proj/myproject/.apptainer_cache
```

## Converting from Docker/Singularity

### Docker → Apptainer

```bash
# Direct conversion
apptainer pull docker://image:tag

# Or from saved Docker image
docker save image:tag -o image.tar
apptainer build image.sif docker-archive://image.tar
```

### Singularity → Apptainer

Apptainer is the renamed Singularity - same format, fully compatible.

```bash
# Old: singularity exec container.sif
# New: apptainer exec container.sif
```

## See Also

- [Python Environments](../python/environments.md) - Python within containers
- [MPI Python](../python/mpi-python.md) - MPI in containers
- [Portability](../portability/naiss-systems.md) - Cross-system usage
