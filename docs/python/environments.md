# Python Environments on NAISS Systems

!!! note "TODO"
    * Verify all commands on NAISS systems.
    * Add examples for common scientific packages.
    * Include best practices for environment management.


Guide to creating and managing Python environments for scientific computing on NAISS HPC systems.

## Why Use Virtual Environments?

- **Isolation**: Project-specific dependencies
- **Reproducibility**: Pin package versions
- **Flexibility**: Different Python versions per project
- **No conflicts**: Avoid system-wide package conflicts
- **User installation**: No admin rights needed

## Loading Python

### Available Python Modules

```bash
module avail python
module load python/3.9.5
python3 --version
```

### System Python (Avoid)

Don't use system Python (`/usr/bin/python`) - use module-provided Python for:
- Newer versions
- Better performance
- Scientific packages
- HPC-optimized builds

## Virtual Environments

### Using venv (Recommended)

#### Create Environment
```bash
module load python/3.9.5
python3 -m venv myenv
```

#### Activate
```bash
source myenv/bin/activate
```

#### Install Packages
```bash
pip install numpy scipy matplotlib
```

#### Deactivate
```bash
deactivate
```

### Complete Workflow Example

```bash
# Load Python module
module load python/3.9.5

# Create virtual environment
python3 -m venv ml_project_env

# Activate
source ml_project_env/bin/activate

# Upgrade pip
pip install --upgrade pip

# Install requirements
pip install numpy scipy pandas scikit-learn

# Verify installation
pip list

# Work on your project
python train_model.py

# Deactivate when done
deactivate
```

## Using Conda (If Available)

### Loading Conda

```bash
module avail conda
module load conda/4.12.0
```

### Creating Conda Environment

```bash
# Create with specific Python version
conda create -n myenv python=3.9

# Activate
conda activate myenv

# Install packages
conda install numpy scipy matplotlib

# Deactivate
conda deactivate
```

### Conda vs venv

| Feature | venv | conda |
|---------|------|-------|
| Speed | Fast | Slower |
| Packages | PyPI (pip) | conda-forge + PyPI |
| Non-Python deps | No | Yes |
| Disk space | Less | More |
| Binary packages | Limited | Extensive |

**Recommendation:** Use venv unless you need conda-specific features.

## Requirements Files

### Creating requirements.txt

```bash
# Save current environment
pip freeze > requirements.txt
```

### Installing from requirements.txt

```bash
pip install -r requirements.txt
```

### Example requirements.txt

```
numpy==1.21.0
scipy==1.7.0
pandas==1.3.0
matplotlib==3.4.2
scikit-learn==0.24.2
```

### Pinning vs. Ranges

```
# Exact version (recommended for reproducibility)
numpy==1.21.0

# Minimum version
numpy>=1.21.0

# Compatible version
numpy~=1.21.0  # >= 1.21.0, < 1.22.0

# Version range
numpy>=1.20.0,<1.22.0
```

## Job Script Integration

### Example SLURM Script

```bash
#!/bin/bash
#SBATCH --job-name=python_job
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1

# Load Python module
module load python/3.9.5

# Activate virtual environment
source /path/to/myenv/bin/activate

# Run Python script
python my_script.py

# Deactivate (optional, job ends anyway)
deactivate
```

### Portable Job Script

```bash
#!/bin/bash
#SBATCH --job-name=python_job
#SBATCH --time=01:00:00

# Load modules
module purge
module load python/3.9.5

# Create venv if it doesn't exist
if [ ! -d "venv" ]; then
    python3 -m venv venv
    source venv/bin/activate
    pip install --upgrade pip
    pip install -r requirements.txt
else
    source venv/bin/activate
fi

# Run application
python train_model.py
```

## Installing Scientific Packages

### NumPy, SciPy, Matplotlib

```bash
pip install numpy scipy matplotlib
```

### Machine Learning

```bash
# Scikit-learn
pip install scikit-learn

# TensorFlow (CPU)
pip install tensorflow

# PyTorch (CPU)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
```

### HPC-Specific Packages

```bash
# MPI for Python
module load openmpi/4.1.1
pip install mpi4py

# HDF5
module load hdf5/1.12.1
pip install h5py
```

## Jupyter Notebooks

### Installing Jupyter

```bash
pip install jupyter ipykernel
```

### Register Kernel

```bash
python -m ipykernel install --user --name=myenv --display-name="Python (myenv)"
```

### Launch Jupyter (Interactive)

```bash
jupyter notebook --no-browser --port=8888
```

Then create SSH tunnel from local machine:
```bash
ssh -L 8888:localhost:8888 username@naiss-system.se
```

Access at `http://localhost:8888`

## Package Installation Issues

### Compiler Required

Some packages need compilation:

```bash
# Load compiler first
module load gcc/11.2.0

# Then install
pip install package_name
```

### Binary Wheel Not Available

```bash
# Force build from source
pip install --no-binary :all: package_name

# Use older pip (may have binaries)
pip install package_name --prefer-binary
```

### Permission Denied

```bash
# Use user installation (within venv not usually needed)
pip install --user package_name

# Or ensure venv is activated
which python  # Should show venv path
```

## Environment Management Best Practices

### 1. One Environment Per Project

```
project_a/
  ├── venv/
  ├── requirements.txt
  └── src/

project_b/
  ├── venv/
  ├── requirements.txt
  └── src/
```

### 2. Document Dependencies

Always maintain `requirements.txt`:
```bash
pip freeze > requirements.txt
```

### 3. Use Specific Versions

```
# Good - reproducible
numpy==1.21.0

# Avoid - may break later
numpy
```

### 4. Separate Dev Dependencies

```
# requirements.txt (production)
numpy==1.21.0
scipy==1.7.0

# requirements-dev.txt (development)
pytest==6.2.4
black==21.6b0
flake8==3.9.2
```

Install both:
```bash
pip install -r requirements.txt -r requirements-dev.txt
```

## Shared Environments

### System-Wide Modules

Some packages are available as modules:
```bash
module avail python-
module load python-numpy/1.21.0
module load python-scipy/1.7.0
```

**Pros:**
- Optimized for HPC
- No installation needed
- Shared across users

**Cons:**
- Limited versions
- Less flexibility
- May not have all packages

### When to Use System Modules

- Common packages (NumPy, SciPy)
- HPC-optimized builds
- Large packages (saves disk space)
- Read-only workflows

### When to Use venv

- Project-specific versions
- Cutting-edge packages
- Custom requirements
- Frequent updates

## Disk Space Considerations

### Check Environment Size

```bash
du -sh venv/
```

### Clean Pip Cache

```bash
pip cache purge
```

### Use --no-cache-dir

```bash
pip install --no-cache-dir package_name
```

### Share Environments (Advanced)

For team projects, consider one shared environment:
```bash
# Create in project directory
python3 -m venv /proj/myproject/venv

# Team members activate
source /proj/myproject/venv/bin/activate
```

**Warning:** Coordinate to avoid conflicts.

## Conda Environment Files

### Export Environment

```bash
conda env export > environment.yml
```

### Create from File

```bash
conda env create -f environment.yml
```

### Example environment.yml

```yaml
name: myenv
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.9
  - numpy=1.21
  - scipy=1.7
  - matplotlib=3.4
  - pip:
      - mpi4py==3.1.0
```

## Troubleshooting

### Wrong Python Version

```bash
# Check Python location
which python
which python3

# Should be in venv
# /path/to/myenv/bin/python
```

### Packages Not Found After Install

```bash
# Ensure venv is activated
source venv/bin/activate

# Verify pip location
which pip

# Reinstall
pip install package_name
```

### Module Import Errors

```bash
# Check installed packages
pip list

# Check Python path
python -c "import sys; print('\n'.join(sys.path))"

# Reinstall package
pip uninstall package_name
pip install package_name
```

### Build Failures

```bash
# Load required modules
module load gcc/11.2.0
module load cmake/3.21.0

# Retry installation
pip install package_name
```

## Performance Tips

- Use system modules for optimized builds (NumPy, SciPy)
- Install packages before submitting jobs
- Use `--no-cache-dir` in containers
- Consider conda for binary packages
- Share environments to save space

## See Also

- [Performance](performance.md) - Python performance optimization
- [MPI Python](mpi-python.md) - Parallel Python with MPI
- [Modules](../modules/index.md) - Module system basics
