# Python Environments on NAISS Systems

!!! note "TODO"
    * Verify all commands on NAISS systems.
    * Add examples for common scientific packages.
    * Include best practices for environment management.


Guide to creating and managing Python environments for scientific computing on NAISS HPC systems.

## Why Use Virtual Environments?

If you have used Python for scientific computing you have probarbly encountered a situation where maintaining your Python install gets more difficult the more packages you install. Perhaps a package requires a specific Python version than you have installed. It can also be that you need to document your current setup for reproducibility so that others can recreate the exact setup you used when running your workflow. It is here the Python Virtual Environments can solve many of these problems such as:

- **Isolation**: Project-specific dependencies
- **Reproducibility**: Pin package versions
- **Flexibility**: Different Python versions per project
- **No conflicts**: Avoid system-wide package conflicts
- **User installation**: No admin rights needed

!!! success
    Whenever starting a new Python-based project consider creating a Python Virtual environment.

## Loading available Python Modules

On NAISS resources all software is provided as modules. A software module sets up the shell environment for using a special software. Read more on this in the NAISS documentation on modules. Python can come in many forms on NAISS systems. Base versions, containing only the standard Python interpreter and associated run time library. Prepackages versions containing common scientific Python libraries such as NumPy, SciPy, Matplotlib, Pandas and an optimised version of MPI for Python. Python can also be provided using a conda distribution such as CondaForge. Please see the documentation for more information on what modules are available on NAISS resources.

### Available Python Modules

An easy way of querying what Python modules are available can be done by using the ``module avail`` command this will produce a list of available Python modules.

```bash
module spider Python

------------------------------------------------------------------------------------
  Python:
------------------------------------------------------------------------------------
    Description:
      Python is a programming language that lets you work more quickly and integrate your systems more effectively.

     Versions:
        Python/2.7.18-bare
        Python/2.7.18
        Python/3.8.6
        Python/3.9.5-bare
        Python/3.9.5
        Python/3.9.6-bare
        Python/3.9.6
        Python/3.10.4-bare
        Python/3.10.4
        Python/3.10.8-bare
        Python/3.10.8
        Python/3.11.3
        Python/3.11.5
        Python/3.12.3
        Python/3.13.1
        Python/3.13.5
```

Then you need to figure out any module dependencies required for the Python module:

```bash
module spider Python/3.13.5
```

You will then get a list of required dependencies:

```bash
------------------------------------------------------------------------------------
  Python: Python/3.13.5
------------------------------------------------------------------------------------
    Description:
      Python is a programming language that lets you work more quickly and integrate your systems more effectively.


    You will need to load all module(s) on any one of the lines below before the "Python/3.13.5" module is available to load.

      GCCcore/14.3.0
 
    This module provides the following extensions:

       flit_core/3.12.0 (E), packaging/25.0 (E), pip/25.1.1 (E), setuptools/80.9.0 (E), setuptools_scm/8.3.
1 (E), tomli/2.2.1 (E), typing_extensions/4.14.0 (E), wheel/0.45.1 (E)
```

Now we can load the Python environment:

```bash
module load GCCcore/14.3.0
module load Python/3.13.5
```

The actual Python environment can be queried:

```bash
$ which python
/sw/easybuild_milan/software/Python/3.13.5-GCCcore-14.3.0/bin/python
$ which python3
/sw/easybuild_milan/software/Python/3.13.5-GCCcore-14.3.0/bin/python3
$ python -V
Python 3.13.5
```

### System Python (Avoid)

Why bother using these complicated modules when we just can use the builtin Python-version (```/usr/bin/python```)? The problem with this approach is that the system provided Python interpreter never gets any new major version only bugfixes. As it doesn't change over the life time of a resource it will be harder and harder to use newer packages or new language features. The system provided scientific packages are often also not highly optimised to be able to be used on a more diverse hardware ecosystem. 

!!! warning
    Never use the system provided Python interpreter for scientific work.

## Virtual Environments

A Python environment can be compared to a self-contained python installation with a Python interpreter and a set of installed packages. There are some different ways you can create Python environments depending on what kind of Python distribution you are using. The first way of creating environment 

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
