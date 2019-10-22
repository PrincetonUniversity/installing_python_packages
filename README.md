# Installing Python Packages on the PU HPC Clusters

This guide presents an overview of installing Python packages on the HPC clusters at Princeton.

Note: throughout this guide we use angular brackets `< >` to denote command line options that you should replace with a value specific to your work.

## Quick Start

If you don't want to spend the time to read this entire page (not recommended) then try the following procedure to install your package (below we assume Python 3):

### pip

At the command line execute the following:

```
module load anaconda3
pip install --user <package>
```

The package and its dependencies will be installed in your home directory in `~/.local/lib/python<version>/site-packages`. Below is a sample Slurm script (job.slurm) that can be used to run your Python script (myscript.py):

```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=00:01:00
#SBATCH --mem-per-cpu=4G

module load anaconda3
srun python myscript.py
```

The job is submitted to the cluster with: `sbatch job.slurm`. Note that the `anaconda3` module is loaded in the Slurm script.

If the installation failed and packages were downloaded then you should remove those packages before proceeding (see contents of `~/.local/lib/`).

### conda

If the above procedure failed then try the following:

```
module load anaconda3
conda create --name myenv <package>
conda activate myenv
```

The package will be installed locally in `~/.conda/pkgs`. On the command line, use `conda deactivate` to leave the environment. Below is a sample Slurm script:

```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=00:01:00
#SBATCH --mem-per-cpu=4G

module load anaconda3
conda activate myenv
srun python myscript.py
```

If both procedures failed then continue reading or see the [Getting Help](#getting_help) section at the bottom of this page.

If for some reason you are trying to install a Python 2 package then use `module load anaconda` instead of `anaconda3` in the directions above. Note that Python 2 will be unsupported beginning on January 1, 2020.

## Introduction

On the Princeton HPC clusters we offer the Anaconda Python distribution. In addition to Python's vast built-in <a href="https://docs.python.org/3/library/">library</a>, Anaconda provides hundreds of additional packages which are ideal for scientific computing. In fact, many of these packages are optimized for our hardware.

To see all the Anaconda packages that are available and their versions do:

```
module load anaconda3
conda list
```

The start of the output is shown below:

```
[jdh4@tigergpu ~]$ conda list
WARNING: The conda.compat module is deprecated and will be removed in a future release.
# packages in environment at /usr/licensed/anaconda3/2019.3:
#
# Name                    Version                   Build  Channel
_ipyw_jlab_nb_ext_conf    0.1.0                    py37_0  
alabaster                 0.7.12                   py37_0  
anaconda                  2019.03                  py37_0  
anaconda-client           1.7.2                    py37_0  
anaconda-navigator        1.9.7                    py37_0  
anaconda-project          0.8.2                    py37_0  
asn1crypto                0.24.0                   py37_0  
astroid                   2.2.5                    py37_0  
...
```

There are about 275 packages pre-installed and ready to be used with a simple `import` statement. If the packages you need are on the list or are found in the standard library then you can begin your work. Otherwise, you will need to do an installation.

Anaconda is a system library. This means you can use all the packages but you can't make any modifications to them (such as an upgrade) and you can't install new ones in their location. You can however install whatever packages you want in your home directory. This allows you to utilize both the pre-installed Anaconda packages and new ones that you install yourself. The two most popular package managers for installing Python packages are `pip` and `conda`.

## Package and Environment Managers

### pip

pip stands for pip installs packages. It is a package manager for Python packages only. pip installs packages that are hosted on the Python Package Index or <a href="https://pypi.org">PyPI</a>. To see if a package you want is available run the following (with the anaconda3 module loaded):

```
pip search <package>
```

If you see your package then do:

```
pip install --user <package>
```

The `--user` option is needed above so that the package is installed in your own user account where you have permission to write files. If you forget to include this option then the install will eventually fail with the following error:

```
Could not install packages due to an EnvironmentError: [Errno 30] Read-only file system
```

pip will search for a pre-compiled version of the package you want called a wheel. If it fails to finds this for your platform then it will attempt to build the package from source. It can take pip several minutes to build a large package from source.

Below is a primer which illustrates the most common uses of pip:

Search PyPI for a given package:
```
pip search scitools
```

List all installed packages:
```
pip list
```

Install pairtools and pyblast for version 3.5 of Python
```
pip install --user python==3.5 pairtools pyblast
```

Install a set of packages listed in a text file
```
pip install --user -r requirements.txt
```

To see detailed information about an installed package
```
pip show sphinx
```

Upgrade the sphinx package
```
pip install --user --upgrade sphinx
```

Uninstall the pairtools package
```
pip uninstall pairtools
```

For more on pip see <a href="https://pip.pypa.io/en/stable/user_guide/">here</a>.

### Isolated Python environments

Often times you will want to create isolated Python environments. This is useful, for instance, when you have two packages that require different versions of a third package. The use of environments saves one the trouble of repeatedly upgrading or downgrading the third package in this case.

We recommend use `virtualenv` to create isolated Python environments. To get started with virtualenv it must first be installed as follows:
```
pip install --user virtualenv
```

Note that like pip, virtualenv is an executable, not a library. To create an isolated environment do:

```
mkdir myenv
virtualenv myenv
source </path/to>/myenv/bin/activate
```

Now you can install Python packages in isolation from other environments:

```
pip install slingshot bell
```

Note the `--user` option is omitted since the packages will be installed locally in the virtual environment. Make sure you source the environment in your Slurm script as in this example:

```
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=00:01:00
#SBATCH --mem-per-cpu=4G

module load anaconda3
source </path/to>/myenv/bin/activate
srun python myscript.py
```

At the command line, to leave the environment do: `deactivate`

As an alternative to `virtualenv`, you may also consider using the built-in Python 3 module `venv`. pip in combination with virtualenv serve as powerful package and environment managers. There are also combined managers such as `pipenv` and `pyenv` that you may consider.

### conda

Unlike pip, conda is both a package manager and an environment manager. It is also language-agnostic which means that in addition to Python packages, it is also used for R and Fortran, for example.

Conda looks to <a href="https://anaconda.org">Anaconda Cloud</a> to handle installation requests but there are numerous other channels that can be searched such as bioconda.

Conda always installs pre-built binary files. The software it provides often has performance advantages over other managers due to leveraging Intel MKL, for instance. Below is a typical session where an environment is created and a package is installed in to it:

```
module load anaconda3
conda create --name myenv <package>
conda activate myenv
```

To leave a conda environment use `conda deactivate`.

If you try to install using `conda install <package>` it will fail with `EnvironmentNotWritableError: The current user does not have write permissions to the target environment`. The solution is to create an environment and do the install in the same command (as shown above).

Below is a primer which illustrates the most common uses of conda:

Search Anaconda Cloud for the fenics package:
```
conda search fenics
```

List all the installed packages for the present environment:
```
conda list
```

Create the myenv environment and install pairtools into the that environment:
```
conda create --name myenv pairtools
```

Create an environment called myenv and install a Python version 3.6 of beaver in it:
```
conda create --name myenv python=3.6 beaver
```

Create an environment called biowork-env and install blast from the bioconda channel:
```
conda create --name biowork-env --channel bioconda blast
```

List the available environments:
```
conda list --envs
```

Remove the bigdata-env environment
```
conda remove --name bigdata-env --all
```

Much more can be done with conda as a <a href="https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-pkgs.html">package manager</a> or <a href="https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html">environment manager</a>.


### pip vs. conda

If your package exists on PyPI and Anaconda Cloud then how do you decide which to install from? The good news is that in most cases it doesn't matter. However, if you are on Windows then you should prefer conda since the packages are pre-built. For Linux if it is a simple package that doesn't do intensive computations then `pip` would be a good choice. If the package needs to be fast or if the dependencies are quite involved then favor `conda`. Because conda packages are pre-compiled, they cannot take full advantage of the vector instructions of our CPUs. However, many scientific conda packages are linked to the Intel MKL which leads to improved performance over pip installs. Conda packages may lag behind pip packages in terms of versioning.


## Installing Python packages from source

In some cases you will be provided with the source code for your package. To install from source do:

```
python setup.py install --prefix=</path/to/install/location>
```

Be sure to update the appropriate environment variables:

```
export PATH=</path/to/install/location>/bin:$PATH
export PYTHONPATH=</path/to/install/location>/lib/python<version>/site-packages:$PYTHONPATH
```

## Packaging and Distributing Your Own Python Package

Both <a href="https://packaging.python.org/tutorials/packaging-projects/">PyPI</a> and Anaconda allow registered users to store their packages on their platforms. You must follow the instructions for doing so but once done someone can do a pip install or a conda install of your package. This makes it very easy to enable someone else to use your research software.

## Common Examples

### FEniCS

<a href="https://fenicsproject.org">FEniCS</a> is an open-source computing platform for solving partial differential equations. To install:

```
module load anaconda3
conda create --name fenics-env -c conda-forge fenics
conda activate fenics-env
```

For better performance one may consider <a href="https://fenics.readthedocs.io/en/latest/installation.html#from-source">installing from source</a>.

### Common pip

```
module load anaconda3
mkdir <your-fenics-env>
virtualenv <your-fenics-env>
source </path/to>/<your-fenics-env>/bin/activate
pip install <package>
```

Make sure you include `source </path/to>/<your-fenics-env>/bin/activate` in your Slurm script.

### TensorFlow

See <a href="https://github.com/PrincetonUniversity/slurm_mnist">this page</a> to install TensorFlow on the HPC clusters.

### PyTorch

<a href="https://pytorch.org">PyTorch</a> is a popular alternative to TensorFlow. To install it in your account:

```
module load anaconda3
conda create --name torch-env pytorch torchvision cudatoolkit=9.0 -c pytorch
conda activate torch-env
```

For more on getting starting with PyTorch on the HPC clusters see <a href="https://github.com/PrincetonUniversity/install_pytorch">here</a>.

### mpi4py

MPI for Python (mpi4py) provides bindings of the Message Passing Interface (MPI) standard for the Python programming language. It can be used to parallelize Python scripts. To install:

```
module load anaconda3
conda create --name fast-mpi4py python=3.7
conda activate fast-mpi4py
module load intel-mpi intel
export MPICC=`which mpicc`
pip install mpi4py
```

Be sure to include `module load anaconda3 intel-mpi intel` in your Slurm script. A complete guide on installing mpi4py on the HPC clusters is <a href="https://oncomputingwell.princeton.edu/2018/11/installing-and-running-mpi4py-on-the-cluster">here</a>.


## FAQ

1. Why does `pip install <package>` fail with an error mentioning a `Read-only file system`?

   After loading the anaconda3 module, pip will be available as part of Anaconda Python which is a system package. By default pip will try to install the files in the same locations as the Anaconda packages. Because you don't have write access to this directory the install will fail. One needs to add `--user` as discussed above.

2. What should I do if I try to install a Python package and the install fails with: `error: Disk quota exceeded`?

   You have three options. First, consider removing files within your home directory to make space available. Second, run the `checkquota` command and follow the link at the bottom to request more space. Lastly, for pip installations see the question toward the bottom of this FAQ for a third possibility i.e., setting `--location to /scratch/gpfs/<username>`.

3. Why do get the following error message when I try to run pip on Della: `-bash: pip: command not found`?

   You need to do `module load anaconda3` before using pip or any of the Anaconda packages. You also need to load this module before using Python itself.

4. I read that it is a good idea to update conda before installing a package. Why do I get an error message when I try to perform the update?

   conda is a system executable. You do not have permission to update it. If you try to update it you will get this error: `EnvironmentNotWritableError: The current user does not have write permissions to the target environment`. The current version is sufficient to install any package.

5. When I run `conda list` I see the package that I need but it is not the right version. How can I get the right version?
   
   One solution is to create a conda environment and install the version you need there. The version of NumPy on Tiger is 1.16.2. If you need version 1.16.5 for your work then do: `conda create --name myenv numpy=1.16.5`. 

6. Is it okay if I combine virtualenv and conda?

   This is highly discouraged. While in principle it can work, most users find it just causes problems. Try to stay within one environment manager. Note that if you create a conda environment you can use pip to install packages.

7. Can I combine conda and pip?

   Yes, and this tends to work well. A typical session may look like this:
   ```
   module load anaconda3
   conda create --name myenv python=3.6
   pip install scitools
   ```
   
   Note that `--user` is omitted when using pip within a conda environment.

8. How do I install a Python package using pip in a custom location?

   There is a two step procedure for this. First do `pip install --target=</path/to/install/location> <package>` then update the PYTHONPATH environment variable with `export PYTHONPATH=$PYTHONPATH:/path/to/install/location`.

9. I tried to install some packages but now none of my Python tools are working. Is it possible to delete all my Python packages and start over?

   Yes. Packages installed by pip are in `.local/lib` while conda packages and environments are in `.conda`. If you made any environments with virtualenv you should remove those as well. Removing these directories will give you a clean start. Be sure to examine the contents first. It may be wise to selectively remove sub-directories instead.
   
10. How are my pip packages built? Which optimization flags? Do I have to be careful on Della where the head node is Broadwell?

    See `/usr/bin/python3.7-config --cflags`. To force a package to be built from source with certain optimization flags do, for example: `CFLAGS="-O1" pip install numpy -vvv --no-binary=numpy`

## <a name="getting_help">Getting Help<a>

If you encounter any difficulties while installing a Python package on one of our HPC clusters then please send an email to <a href="mailto:cses@princeton.edu">cses@princeton.edu</a> or attend a <a href="https://researchcomputing.princeton.edu/education/help-sessions">help session</a>.
