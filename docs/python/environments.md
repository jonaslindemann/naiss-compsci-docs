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

Now we can load the Python module and its dependencies:

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

A Python environment can be compared to a self-contained python installation with a Python interpreter and a set of installed packages. There are some different ways you can create Python environments depending on what kind of Python distribution you are using. The following sections will cover how to create Python environments using the builtin ``venv`` module and using Conda-Forge if available on the resource.

### Using venv (Recommended)

``venv`` is a built-in module for creating lightweight Python environments. It is available in Python 3.3 and later and is the preferred way of creating Python environments when not using Conda. It creates an isolated environment with its own Python interpreter and site-packages directory.

#### Create Environment

An environment is created by first loading the selected Python module and then using the ``venv`` module to create a new environment. The following example creates a new environment called ``myenv`` in the current directory.

```bash
module load python/3.9.5
python3 -m venv myenv
```

#### Activate

Before using the environment it needs to be activated. Activating an environment is equivalent to setting up all the required paths for both python and associated packages. An environemtn is activated by sourcing the ``activate`` script in the ``bin`` directory of the environment.

```bash
source myenv/bin/activate
```

#### Install Packages

When the environment is activated you can use pip to install packages into the environment. The following example installs NumPy, SciPy and Matplotlib into the environment.

```bash
pip install numpy scipy matplotlib
```

#### Deactivate

When you are done working in the environment you can deactivate it to return to the system Python environment. Deactivating an environment is done by running the ``deactivate`` command and is available in the shell after activating the environment.

```bash
deactivate
```

### Complete Workflow Example

In this example we load the selected Python module, create a new virtual environment, activate it, install some common scientific packages, verify the installation, run a Python script and then deactivate the environment.

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

Conda is a popular package and environment management system that can be used to create and manage Python environments. It is especially useful for managing complex dependencies and non-Python packages. If Conda is available on the NAISS resource it can be used as an alternative to venv for creating Python environments. 

### Loading Conda

Before using Conda you need to load the Conda module if it is available on the resource. You can check if Conda is available by using the ``module avail`` command. If it is available you can load it using the ``module load`` command.

```bash
module avail conda
module load conda/4.12.0
```

### Creating Conda Environment

Creating a Conda environment is similar to using venv but with some additional features. You can specify the Python version and also include non-Python dependencies if needed. The following example creates a new Conda environment called ``myenv`` with Python 3.12 and installs some common scientific packages.

```bash
# Create with specific Python version
conda create -n myenv python=3.12

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

In many cases you will want to document the packages and versions you have installed in your environment. This is important for reproducibility and for sharing your environment with others. A common way to do this is by using a ``requirements.txt`` file which lists all the packages and their versions.

### Creating requirements.txt

Using pip you can easily create a ``requirements.txt`` file that captures all the installed packages in your environment. This is done using the ``pip freeze`` command which outputs a list of installed packages and their versions.

```bash
# Save current environment
pip freeze > requirements.txt
```

A typical ``requirements.txt`` file will look like this:

```numpy==1.21.0
scipy==1.7.0
pandas==1.3.0
matplotlib==3.4.2
scikit-learn==0.24.2
```

### Installing from requirements.txt

Installing from a ``requirements.txt`` file is straightforward using pip. This will install all the packages listed in the file with the specified versions.

```bash
pip install -r requirements.txt
```

### Pinning vs. Ranges

When specifying package versions in a requirements file you can choose to pin exact versions, specify minimum versions, compatible versions or version ranges. The choice depends on your need for reproducibility and flexibility. It can be a good practice to pin exact versions for production environments to ensure reproducibility, while using version ranges during development to allow for updates. Pinning too hard can lead to issues when packages are updated and no longer compatible, while being too flexible can lead to unexpected breakages. Here are some examples of different version specifications:

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

When running Python scripts on NAISS HPC systems you will typically submit a job script to the scheduler (e.g., SLURM). It is important to ensure that your Python environment is properly set up in the job script so that your Python code can run with the correct dependencies. Below are examples of how to integrate Python environment setup into a SLURM job script.

### Example SLURM Script

In this example we load the required Python module, activate a virtual environment, run a Python script and then deactivate the environment. Deactivation is optional as the job will end anyway, but it can be good practice to clean up the environment.

```bash
#!/bin/bash
#SBATCH --job-name=python_job
#SBATCH --time=01:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1

# Load Python module
module load python/3.12.3

# Activate virtual environment
source /path/to/myenv/bin/activate

# Run Python script
python my_script.py

# Deactivate (optional, job ends anyway)
deactivate
```

### Portable Job Script

To make your job script more portable across different NAISS resources you can include checks for the required modules and create the virtual environment if it doesn't exist. This way you can run the same script on different resources without needing to manually set up the environment each time.

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

## Jupyter Notebooks

Jupyter notebooks is an important tool in scientific research. However, running Jupyter notebooks on HPC systems can be tricky due to the need for an interactive session and the fact that HPC systems are typically accessed remotely. Below are some tips for running Jupyter notebooks on NAISS systems.

### Installing Jupyter

To be able to run Jupyter notebooks you need to have Jupyter installed in your Python environment. This can be done using pip or conda depending on how you created your environment. The following example shows how to install Jupyter using pip in a virtual environment.

```bash
pip install jupyter ipykernel
```

```bash
conda install jupyter ipykernel
```

### Register Kernel

If you want to be able to select your virtual environment as a kernel in Jupyter notebooks you need to register it using the following command:

```bash
python -m ipykernel install --user --name=myenv --display-name="Python (myenv)"
```

### Launch Jupyter (Interactive)

A notebook instance is launched by running the following command in an interactive session. This will start a Jupyter server on the compute node and print out a URL with a token that you can use to access the notebook from your local machine.

```bash
jupyter notebook --no-browser --port=8888
```

Then create SSH tunnel from your local machine:

```bash
ssh -L 8888:localhost:8888 username@naiss-system.se
```

Access at `http://localhost:8888`

### Launch Jupyter Lab (Interactive)

```bash
jupyter lab --no-browser --port=8888
```

Then create SSH tunnel from your local machine:

```bash
ssh -L 8888:localhost:8888 username@naiss-system.se
```

## Package Installation Issues

### Compiler Required

Some Python packages require compilation and thus need a compatible compiler to be loaded. If you encounter build errors during pip installation it may be because the required compiler is not available in your environment.

```bash
# Load compiler first
module load gcc/11.2.0

# Then install
pip install package_name
```

If you are using a Python module that has a specific compiler dependency make sure to load the required compiler module before loading the Python module.

### Binary Wheel Not Available

Some packages may not have pre-built binary wheels available for your platform or Python version, which can lead to build failures during installation. In such cases, you can try forcing pip to build from source or using an older version of pip that may have access to older binary wheels.

```bash
# Force build from source
pip install --no-binary :all: package_name

# Use older pip (may have binaries)
pip install package_name --prefer-binary
```

### Permission Denied

If you use module that provides a Python environment and try to install packages using pip you may encounter permission errors because the module environment is read-only. In this case you should create a virtual environment and install packages there instead of trying to install them in the module environment.

```bash
# Use user installation (within venv not usually needed)
pip install --user package_name

# Or ensure venv is activated
which python  # Should show venv path
```

## Environment Management Best Practices

Environments are the key to managing Python dependencies effectively. Here are some best practices for creating and maintaining Python environments on NAISS systems.

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

Conda environments are often created in a central location and activated per project:

```bash
conda create -n project-a-env python=3.9
conda create -n project-b-env python=3.10
conda activate project_a
```

It can be an idea to give your environments descriptive names that reflect the project they are associated with. This makes it easier to manage multiple environments and avoid confusion. For conda modules it can also be a good idea to suffix the name with ``-env`` to avoid clashes with module names.

### 2. Document Dependencies

If you are working with other people or want to ensure reproducibility it is important to document the dependencies of your project. This can be done using a ``requirements.txt`` file for pip environments or an ``environment.yml`` file for conda environments. These files should be kept up to date whenever you add or update packages in your environment.

For pip environments, use `requirements.txt`:

```bash
pip freeze > requirements.txt
```

For conda environments, use `environment.yml`:

```bash
conda env export > environment.yml
```

### 3. Use Specific Versions

If you rely on specific versions of packages it is important to specify those versions in your requirements file. This ensures that you can recreate the exact environment later and that others can do the same. Avoid using unpinned dependencies as they can lead to breakages when new versions are released.

```
# Good - reproducible
numpy==1.21.0

# Avoid - may break later
numpy
```

### 4. Separate Dev Dependencies

It can be a good idea to separate production dependencies from development dependencies. This can be done by having two requirements files, one for production and one for development. The production file should only include the packages needed to run the application, while the development file can include additional tools for testing, linting and formatting.

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

On many HPC systems there are system-wide Python modules available that include common scientific packages optimised for the system. These can be a good option for users who want to quickly get started without needing to set up their own environment. However, they may not have the latest versions of packages and may not include all the packages you need. It is important to weigh the pros and cons of using system-wide modules versus creating your own virtual environment.

### System-Wide Modules

These modules are provided by the system administrators and are available to all users. They often include common scientific packages that are optimised for the HPC system. However, they may not have the latest versions of packages and may not include all the packages you need.

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

Python environments can take up a lot of disk space, especially if you have many packages installed or if you are using conda environments which can include non-Python dependencies. It is important to be mindful of disk space when creating and managing Python environments on NAISS systems. Here are some tips for managing disk space:

### Remove Unused Environments

If you have environments that you are no longer using it is a good idea to remove them to free up disk space. For venv environments you can simply delete the environment directory. For conda environments you can use the following command to remove an environment:

```bash
conda env remove -n myenv
```

### Check Environment Size

Check the size of your environment to see how much disk space it is using. This can help you identify if you have any large packages installed that you may not need. Use the following command to check the size of your virtual environment:

```bash
du -sh venv/
```

For conda environments, use:

```bash
du -sh /path/to/conda/envs/myenv/
```

### Clean Pip Cache

Pip caches downloaded packages which can take up a lot of disk space over time. You can clear the pip cache to free up space using the following command:

```bash
pip cache purge
```

### Use --no-cache-dir

When installing packages in a containerized environment or when disk space is a concern, you can use the `--no-cache-dir` option with pip to prevent caching of downloaded packages. This can help reduce disk usage but may lead to longer installation times if you need to reinstall packages.

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

!!! warning
    Be cautious when sharing environments to avoid conflicts and ensure proper permissions.

## Conda Environment Files

Conda environments can be exported to a YAML file which captures the exact state of the environment including all packages and their versions. This is useful for sharing environments with others or for recreating the environment on a different system. The following sections cover how to export and create conda environments using environment files.

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

This section provides common troubleshooting tips for issues related to Python environments on NAISS systems.

### Wrong Python Version

Many times users may accidentally use the system Python instead of their virtual environment. This can lead to issues with missing packages or incompatible versions. To check which Python interpreter you are using, you can use the following commands:

```bash
# Check Python location
which python
which python3

# Should be in venv
# /path/to/myenv/bin/python
```

### Packages Not Found After Install

Sometimes after installing a package you may find that it is not available when you try to import it in Python. This can happen if the package was installed in a different environment or if there was an issue during installation. To troubleshoot this, you can check the location of pip and ensure that you are installing packages in the correct environment.

```bash
# Ensure venv is activated
source venv/bin/activate

# Verify pip location
which pip

# Reinstall
pip install package_name
```

### Module Import Errors

If you encounter import errors when trying to use a package in Python it may be because the package was not installed correctly or because there is a conflict with another package. To troubleshoot this, you can check the list of installed packages and their versions, check the Python path to ensure that the correct environment is being used and try reinstalling the package.

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

Some packages require compilation and may fail to install if the required compiler is not available. If you encounter build errors during pip installation it may be because the required compiler is not loaded in your environment. To fix this, you can load the required compiler module before installing the package.

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
