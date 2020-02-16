# Installing Python Packages on the HPC Clusters

This guide presents an overview of installing Python packages on the HPC clusters at Princeton.

Throughout this guide we use angular brackets `< >` to denote command line options that you should replace with a value specific to your work. Commands preceded by the $ character are to be run on the command line.

## Quick Start

If you don't want to spend the time to read this entire page (not recommended) then try the following procedure to install your package(s) (below we assume Python 3):

### pip

At the command line execute the following:

```
$ module load anaconda3
$ pip install --user <package-1> <package-2> ... <package-N>
```

Each package and its dependencies will be installed in your home directory in `~/.local/lib/python<version>/site-packages`. Below is a sample Slurm script (job.slurm) that can be used to run your Python script (myscript.py):

```bash
#!/bin/bash
#SBATCH --job-name=py-job        # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=1               # total number of tasks across all nodes
#SBATCH --cpus-per-task=1        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G per cpu-core is default)
#SBATCH --time=00:01:00          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=all          # send email when job begins, ends and fails
#SBATCH --mail-user=<YourNetID>@princeton.edu

module purge
module load anaconda3

python myscript.py
```

The job is submitted to the cluster with: `$ sbatch job.slurm`. Note that the `anaconda3` module is loaded in the Slurm script.

If the installation failed and packages were downloaded then you should remove those packages before proceeding (see contents of `~/.local/lib/`).

### conda

If the above procedure failed then try the following:

```
$ module load anaconda3
$ conda create --name myenv <package-1> <package-2> ... <package-N>
$ conda activate myenv
```

Each package and its dependencies will be installed locally in `~/.conda/pkgs`. Consider replacing `myenv` with an environment name that is more specific to your work. On the command line, use `$ conda deactivate` to leave the active environment and return to the base environment. Below is a sample Slurm script (job.slurm):

```bash
#!/bin/bash
#SBATCH --job-name=py-job        # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=1               # total number of tasks across all nodes
#SBATCH --cpus-per-task=1        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G per cpu-core is default)
#SBATCH --time=00:01:00          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=all          # send email when job begins, ends and fails
#SBATCH --mail-user=<YourNetID>@princeton.edu

module purge
module load anaconda3
conda activate myenv

python myscript.py
```

If the installation was successful then your job can be submitted to the cluster with `$ sbatch job.slurm`. If the installation failed and packages were downloaded then you should remove those packages before proceeding (see contents of `~/.conda/pkgs`). If both the pip and conda procedures failed then continue reading or see the [Getting Help](#getting_help) section at the bottom of this page.

If for some reason you are trying to install a Python 2 package then use `module load anaconda` instead of `anaconda3` in the directions above. Note that Python 2 has been unsupported since January 1, 2020.

## Introduction

When you first login to a one of our clusters, the system Python is available but this is almost always not what you want. To see the system Python, run these commands:

```
$ python --version
Python 2.7.5

$ which python
/usr/bin/python

$ python3 --version
Python 3.6.8

$ which python3
/usr/bin/python3
```

We see that `python` corresponds to version 2 and both are installed in a standard system directory.

On the Princeton HPC clusters we offer the Anaconda Python distribution. In addition to Python's vast built-in <a href="https://docs.python.org/3/library/">library</a>, Anaconda provides hundreds of additional packages which are ideal for scientific computing. In fact, many of these packages are optimized for our hardware.

To make Anaconda Python available, run the following command:

```
$ module load anaconda3
```

Let's inspect our newly loaded Python by using the same commands as above:

```
$ python --version
Python 3.7.4

$ which python
/usr/licensed/anaconda3/2019.10/bin/python

$ python3 --version
Python 3.7.4

$ which python3
/usr/licensed/anaconda3/2019.10/bin/python3
```

We now have an updated version of Python and related tools. In fact, the new `python` and `python3` commands are identical as they are in fact symbolic links that point to `python3.7`.

To see all the pre-installed Anaconda packages and their versions use the `conda list` command:

```
$ conda list
# packages in environment at /usr/licensed/anaconda3/2019.10:
#
# Name                    Version                   Build  Channel
_ipyw_jlab_nb_ext_conf    0.1.0                    py37_0  
_libgcc_mutex             0.1                        main  
alabaster                 0.7.12                   py37_0  
anaconda                  2019.10                  py37_0  
anaconda-client           1.7.2                    py37_0  
anaconda-navigator        1.9.7                    py37_0  
anaconda-project          0.8.3                      py_0  
asn1crypto                1.0.1                    py37_0  
astroid                   2.3.1                    py37_0
...
```

There are 291 packages pre-installed and ready to be used with a simple `import` statement. If the packages you need are on the list or are found in the Python standard library then you can begin your work. Otherwise, keep reading to learn how to install packages.

Anaconda Python is a system library. This means that you can use all the packages but you can't make any modifications to them (such as an upgrade) and you can't install new ones in their location. You can, however, install whatever packages you want in your home directory. This allows you to utilize both the pre-installed Anaconda packages and the new ones that you install yourself. The two most popular package managers for installing Python packages are `pip` and `conda`.

## Package and Environment Managers

### pip

pip stands for "pip installs packages". It is a package manager for Python packages only. pip installs packages that are hosted on the Python Package Index or <a href="https://pypi.org">PyPI</a>. To see if a package you want is available, run the following (with `anaconda3` module loaded):

```
$ pip search <package>
```

If you see your package then do:

```
$ pip install --user <package>
```

The `--user` option is needed above so that the package is installed in your account where you have permission to write files. If you forget to include this option then the install will eventually fail with the following error:

```
Could not install packages due to an EnvironmentError: [Errno 30] Read-only file system
```

Do not use the `pip3` command even if the directions you are following tell you to do so (use `pip` instead).

pip will search for a pre-compiled version of the package you want called a wheel. If it fails to finds this for your platform then it will attempt to build the package from source. It can take pip several minutes to build a large package from source.

One often needs to load various environment modules in addition to `anaconda3` before doing a pip install. For instance, if your package uses GPUs then you will probably need to do `$ module load cudatoolkit` or if it uses the message-passing interface (MPI) for parallelization then `$ module load openmpi`. To see all available software modules, run `$ module avail`.

### Common pip commands

View the help menu:
```
$ pip -h
```

The help menu for the install command:

```
$ pip install --help
```

Search the Python Package Index ([PyPI](https://pypi.org/)) for a given package (e.g., jax):
```
$ pip search jax
```

List all installed packages:
```
$ pip list
```

Install pairtools and pyblast for version 3.5 of Python
```
$ pip install --user python==3.5 pairtools pyblast
```

Install a set of packages listed in a text file
```
$ pip install --user -r requirements.txt
```

To see detailed information about an installed package such as sphinx:
```
$ pip show sphinx
```

Upgrade the sphinx package:
```
$ pip install --user --upgrade sphinx
```

Uninstall the pairtools package:
```
$ pip uninstall pairtools
```

For more on pip see <a href="https://pip.pypa.io/en/stable/user_guide/">here</a>.

### Isolated Python environments

Often times you will want to create isolated Python environments. This is useful, for instance, when you have two packages that require different versions of a third package. The use of environments saves one the trouble of repeatedly upgrading or downgrading the third package in this case.

We recommend using `virtualenv` to create isolated Python environments. To get started with `virtualenv` it must first be installed:

```
$ module load anaconda3
$ pip install --user virtualenv
```

Note that like pip, `virtualenv` is an executable, not a library. To create an isolated environment do:

```
$ mkdir myenv
$ virtualenv myenv
$ source </path/to>/myenv/bin/activate
```

Consider replacing `myenv` with a more suitable name for your work. Now you can install Python packages in isolation from other Python environments:

```
$ pip install slingshot bell
```

Note the `--user` option is omitted since the packages will be installed locally in the virtual environment. Make sure you `source` the environment in your Slurm script as in this example:

```
#!/bin/bash
#SBATCH --job-name=py-job        # create a short name for your job
#SBATCH --nodes=1                # node count
#SBATCH --ntasks=1               # total number of tasks across all nodes
#SBATCH --cpus-per-task=1        # cpu-cores per task (>1 if multi-threaded tasks)
#SBATCH --mem-per-cpu=4G         # memory per cpu-core (4G per cpu-core is default)
#SBATCH --time=00:01:00          # total run time limit (HH:MM:SS)
#SBATCH --mail-type=all          # send email when job begins, ends and fails
#SBATCH --mail-user=<YourNetID>@princeton.edu

module load anaconda3
source </path/to>/myenv/bin/activate

python myscript.py
```

At the command line, to leave the environment run `$ deactivate`

As an alternative to `virtualenv`, you may consider using the built-in Python 3 module `venv`. pip in combination with virtualenv serve as powerful package and environment managers. There are also combined managers such as `pipenv` and `pyenv` that you may consider.

### conda

Unlike pip, conda is both a package manager and an environment manager. It is also language-agnostic which means that in addition to Python packages, it is also used for R and Fortran, for example.

Conda looks to the main channel of <a href="https://anaconda.org">Anaconda Cloud</a> to handle installation requests but there are numerous other channels that can be searched such as `bioconda`, `intel` and `conda-forge`.

Conda always installs pre-built binary files. The software it provides often has performance advantages over other managers due to leveraging Intel MKL, for instance. Below is a typical session where an environment is created and one or more packages are installed in to it:

```
$ module load anaconda3
$ conda create --name myenv <package-1> <package-2> ... <package-N>
$ conda activate myenv
```

Note that you should specify all the packages that you need in one line so that the dependencies can be satisfied simultaneously. Installing packages at a later time is possible but should be avoided.

To exit a conda environment, run this command: `$ conda deactivate`.

If you try to install using `conda install <package>` it will fail with `EnvironmentNotWritableError: The current user does not have write permissions to the target environment`. The solution is to create an environment and do the install in the same command (as shown above).

### Common conda commands

View the help menu:
```
$ conda -h
```

To view the help menu for the install command:

```
conda install --help
```

Search the `conda-forge` channel for the fenics package:

```
$ conda search fenics --channel conda-forge
```

List all the installed packages for the present environment (consider adding `--explicit`):

```
$ conda list
```

Create the myenv environment and install pairtools into the that environment:

```
$ conda create --name myenv pairtools
```

Create an environment called myenv and install a Python version 3.6 of beaver in it:

```
$ conda create --name myenv python=3.6 beaver
```

Create an environment called biowork-env and install blast from the bioconda channel:

```
$ conda create --name biowork-env --channel bioconda blast
```

List the available environments:

```
$ conda list --envs
```

Remove the bigdata-env environment

```
$ conda remove --name bigdata-env --all
```

Much more can be done with conda as a <a href="https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-pkgs.html">package manager</a> or <a href="https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html">environment manager</a>.

### pip vs. conda

If your package exists on PyPI and Anaconda Cloud then how do you decide which to install from? The good news is that in most cases it doesn't matter. However, if you are on Windows then you should prefer conda since the packages are pre-built. For Linux if it is a simple package that doesn't do intensive computations then `pip` would be a good choice. If the package needs to be fast or if the dependencies are quite involved then favor `conda`. Because conda packages are pre-compiled, some of them cannot take full advantage of the vector units on our CPUs. However, many scientific conda packages are linked against the Intel MKL which leads to improved performance over pip installs. Conda packages may lag behind pip packages in terms of versioning. In many cases, the developers of the package will make a recommendation on their website.

## Installing Python Packages from Source

In some cases you will be provided with the source code for your package. To install from source do:

```
$ python setup.py install --prefix=</path/to/install/location>
```

For help menu use `python setup.py --help-commands`.

Be sure to update the appropriate environment variables in your `~/.bashrc` file:

```
export PATH=</path/to/install/location>/bin:$PATH
export PYTHONPATH=</path/to/install/location>/lib/python<version>/site-packages:$PYTHONPATH
```

## Packaging and Distributing Your Own Python Package

Both <a href="https://packaging.python.org/tutorials/packaging-projects/">PyPI</a> and Anaconda allow registered users to store their packages on their platforms. You must follow the instructions for doing so but once done someone can do a pip install or a conda install of your package. This makes it very easy to enable someone else to use your research software.

## Where to Store Your Files

You should run your jobs out of `/scratch/gpfs/<NetID>` on the HPC clusters. These filesystems are very fast and provide vast amounts of storage. **Do not run jobs out of `/tigress` or `/projects`. That is, you should never be writing the output of actively running jobs to those filesystems.** `/tigress` and `/projects` are slow and should only be used for backing up the files that you produce on `/scratch/gpfs`. Your `/home` directory on all clusters is small and it should only be used for storing source code and executables.

The commands below give you an idea of how to properly run a Python job:

```
$ ssh <NetID>@della.princeton.edu
$ cd /scratch/gpfs/<NetID>
$ mkdir myjob && cd myjob
# put Python script and Slurm script in myjob
$ sbatch job.slurm
```

If the run produces data that you want to backup then copy or move it to `/tigress`:

```
$ cp -r /scratch/gpfs/<NetID>/myjob /tigress/<NetID>
```

For large transfers consider using `rsync` instead of `cp`. Most users only do back-ups to `/tigress` every week or so. While `/scratch/gpfs` is not backed-up, files are never removed. However, important results should be transferred to `/tigress` or `/projects`.

The diagram below gives an overview of the filesystems:

![tigress](https://tigress-web.princeton.edu/~jdh4/hpc_princeton_filesystems.png)

## Running Jupyter Notebooks on the HPC Clusters

Jupyter is available through two web portals. If you have an account on Adroit or Della then browse to [https://myadroit.princeton.edu](https://myadroit.princeton.edu) or [https://mydella.princeton.edu](https://mydella.princeton.edu). If you need an account on Adroit then complete [this form](https://forms.rc.princeton.edu/registration/?q=adroit).

To begin a session, click on "Interactive Apps" and then "Jupyter". You will need to choose the "Number of hours", "Number of cores" and "Memory allocated". Set "Number of cores" to 1 unless you are sure that your script has been explicitly parallelized. Click "Launch" and then when your session is ready click "Connect to Jupyter". Note that the more resources you request, the more you will have to wait for your session to become available. When your session starts, click on "New" in the upper right and choose "Python 3.7 [anaconda3/2019.10]" from the drop-down menu.

![jupyter](https://tigress-web.princeton.edu/~jdh4/jupyter_notebook.png)

If you need more control over the session or if you need to use a GPU then see [this post](https://oncomputingwell.princeton.edu/2018/05/jupyter-on-the-cluster).

### Using Conda environments on MyAdroit and MyDella

First, create a Conda environment on the head node. For example, for Adroit/MyAdroit:

```
$ ssh <YourNetID>@adroit.princeton.edu
$ module load anaconda3
$ conda create --name tf-cpu tensorflow matplotlib
```

Then go to MyAdroit and launch a Jupyter notebook by entering the "Number of hours" and so on and then click on "Launch". When your session is ready click "Connect to Jupyter". On the next screen, choose "New" in the upper right and then `tf-cpu` in the drop-down menu. Your `tf-cpu` environment will be active when the notebook becomes available. To see the packages in your Conda environment, run this command in a cell:

```
%conda list
```

Note that Jupyter notebooks via OnDemand run on the compute nodes where internet access is turned off for security purposes. This means that you will not be able to install packages. All installations must be done on the head node (i.e., `ssh adroit`).

## Common Package Installation Examples

### FEniCS

<a href="https://fenicsproject.org">FEniCS</a> is an open-source computing platform for solving partial differential equations. To install:

```
$ module load anaconda3
$ conda create --name fenics-env -c conda-forge fenics
$ conda activate fenics-env
```

Make sure you include `source </path/to>/<your-fenics-env>/bin/activate` in your Slurm script. For better performance one may consider <a href="https://fenics.readthedocs.io/en/latest/installation.html#from-source">installing from source</a>.

### CuPy on Traverse:

CuPy is available via Anaconda Cloud on all our clusters except Traverse. For the `ppc64le` architecture of Traverse, use this procedure:

```
$ module load anaconda3
$ module load cudatoolkit
$ pip install --user cupy
```

Be sure to include `module load anaconda3` in your Slurm script.

### PyStan

If you get an error like "CompileError: command 'gcc' failed with exit status 1" then try again after loading the rh module:
`$ module load rh/devtoolset/7`. The rh module provides a newer compiler suite.

### Deeplabcut

```bash
$ ssh -Y <YourNetID>@tigergpu.princeton.edu
$ module load hdf5/gcc/1.8.16 anaconda3/2019.10 cudatoolkit/9.2 cudnn/cuda-9.2/7.3.1
$ export HDF5_DIR=/usr/local/hdf5/gcc/1.8.16
$ conda create --name deeplabcut-env python=3.6  wxPython=4.0.3
$ conda activate deeplabcut-env
$ pip install deeplabcut
$ pip install tensorflow-gpu==1.8
```

Note that some warnings will be produced when deeplabcut is imported in Python. You will need to add the “export HDF5_DIR …” command to your ~/.bashrc file. By using `ssh -Y` the error of `Cannont load backend 'TkAgg'` will not occur.

### Lenstools

```bash
$ module load anaconda3
$ conda create --name lenstools-env numpy scipy pandas matplotlib astropy
$ conda activate lenstools-env
$ module load rh/devtoolset/8 openmpi/gcc/3.1.5/64 gsl/2.4 
$ export MPICC=`which mpicc`
$ pip install mpi4py
$ pip install emcee==2.2.1
$ pip install lenstools
```

Note that you will receive warnings when `lenstools` is imported in Python.

### TensorFlow

See <a href="https://github.com/PrincetonUniversity/slurm_mnist">this page</a> to install TensorFlow on the HPC clusters.

### PyTorch

See <a href="https://github.com/PrincetonUniversity/install_pytorch">this page</a> to install PyTorch on the HPC clusters.

### PyTorch Cluster

Here are directions for TigerGPU:

```
$ ssh <NetID>@tigergpu.princeton.edu
$ module load anaconda3/2019.10
$ conda create --name torch-env pytorch torchvision cudatoolkit=10.1 scipy pytest-runner pytest-cov --channel pytorch
$ git clone https://github.com/rusty1s/pytorch_cluster.git
```

The build procedure for [PyTorch Cluster](https://github.com/rusty1s/pytorch_cluster) requires that GPUs be on the machine so create an interactive allocation on a GPU node (compute nodes do not have internet access so we downloaded the source code first using `git clone`):

```
$ salloc -N 1 -n 1 -t 5 --gres=gpu:1
$ module load anaconda3/2019.10 cudatoolkit/10.1 rh/devtoolset/8
$ conda activate torch-env
$ cd pytorch_cluster
$ python setup.py install test
$ exit
```

### mpi4py

MPI for Python (mpi4py) provides bindings of the Message Passing Interface (MPI) standard for the Python programming language. It can be used to parallelize Python scripts. To install:

```
$ module load anaconda3
$ conda create --name fast-mpi4py python=3.7
$ conda activate fast-mpi4py
$ module load rh/devtoolset/7 openmpi/gcc/3.1.3/64
$ export MPICC=`which mpicc`
$ pip install mpi4py
```

Be sure to include `module load anaconda3 openmpi/gcc/3.1.3/64` in your Slurm script. A complete guide on installing mpi4py on the HPC clusters is <a href="https://oncomputingwell.princeton.edu/2018/11/installing-and-running-mpi4py-on-the-cluster">here</a>.


## FAQ

1. Why does `pip install <package>` fail with an error mentioning a `Read-only file system`?

   After loading the anaconda3 module, pip will be available as part of Anaconda Python which is a system package. By default pip will try to install the files in the same locations as the Anaconda packages. Because you don't have write access to this directory the install will fail. One needs to add `--user` as discussed above.

2. What should I do if I try to install a Python package and the install fails with: `error: Disk quota exceeded`?

   You have three options. First, consider removing files within your home directory to make space available. Second, run the `checkquota` command and follow the link at the bottom to request more space. Lastly, for pip installations see the question toward the bottom of this FAQ for a third possibility i.e., setting `--location to /scratch/gpfs/<username>`. For conda installs try learning about the `--prefix` option.

3. Why do I get the following error message when I try to run pip on Della: `-bash: pip: command not found`?

   You need to do `module load anaconda3` before using pip or any of the Anaconda packages. You also need to load this module before using Python itself.

4. I read that it is a good idea to update conda before installing a package. Why do I get an error message when I try to perform the update?

   conda is a system executable. You do not have permission to update it. If you try to update it you will get this error: `EnvironmentNotWritableError: The current user does not have write permissions to the target environment`. The current version is sufficient to install any package.

5. When I run `conda list` on the base environment I see the package that I need but it is not the right version. How can I get the right version?
   
   One solution is to create a conda environment and install the version you need there. The version of NumPy on Tiger is 1.16.2. If you need version 1.16.5 for your work then do: `conda create --name myenv numpy=1.16.5`. 

6. Is it okay if I combine virtualenv and conda?

   This is highly discouraged. While in principle it can work, most users find it just causes problems. Try to stay within one environment manager. Note that if you create a conda environment you can use pip to install packages.

7. Can I combine conda and pip?

   Yes, and this tends to work well. A typical session may look like this:
   ```
   $ module load anaconda3
   $ conda create --name myenv python=3.6
   $ pip install scitools
   ```
   
   Note that `--user` is omitted when using pip within a conda environment. See the bullet points at the bottom of [this page](https://www.anaconda.com/using-pip-in-a-conda-environment/) for tips on using this approach.

8. How do I install a Python package in a custom location using pip or conda?

   For pip, first do `pip install --target=</path/to/install/location> <package>` then update the PYTHONPATH environment variable in your `~/.bashrc` file with `export PYTHONPATH=$PYTHONPATH:/path/to/install/location`.

   For conda, you use the `--prefix` option. For instance, to install cupy on `/scratch/gpfs/<NetID>`:

    ```
    $ module load anaconda3
    $ conda create --prefix /scratch/gpfs/$USER/py-gpu cupy
    ```

   Be sure to have these two lines in your Slurm script: `module load anaconda3` and `conda activate /scratch/network/$USER/py-gpu`. Note that `/scratch/gpfs` is not backed up.

9. I tried to install some packages but now none of my Python tools are working. Is it possible to delete all my Python packages and start over?

   Yes. Packages installed by pip are in `~/.local/lib` while conda packages and environments are in `~/.conda`. If you made any environments with virtualenv you should remove those as well. Removing these directories will give you a clean start. Be sure to examine the contents first. It may be wise to selectively remove sub-directories instead. You may also need remove the `~/.cache` directory and you may need to make modifications to your `.bashrc` file if you added or changed environment variables.
   
10. How are my pip packages built? Which optimization flags are used? Do I have to be careful with vectorization on Della where several different CPUs are in play?

    After loading the anaconda3 module, run this command: `python3.7-config --cflags`. To force a package to be built from source with certain optimization flags do, for example: `CFLAGS="-O1" pip install numpy -vvv --no-binary=numpy`

11. What is the Intel Python distribution and how do I get started with it?

    Intel provides their own implementation of Python as well as numerous packages optimized for Intel hardware. You may find significant performance benefits from these packages. The Intel Python interpreter does not use GNU readline so most find it awkward to use. To create a conda environment with Intel Python and some Intel-optimized numerics packages:
   
    ```
    $ module load anaconda3
    $ conda create --name my-intel --channel intel python numpy scipy
    ```
12. The installation directions that I am following say to use `pip3`. Is this okay?

    Do not use `pip3` for installing Python packages. `pip3` is a component of the system Python and it will not work properly with Anaconda. Always do `module load anaconda3` and then use `pip` for installing packages.


## <a name="getting_help">Getting Help<a>

If you encounter any difficulties while installing a Python package on the HPC clusters then please send an email to <a href="mailto:cses@princeton.edu">cses@princeton.edu</a> or attend a <a href="https://researchcomputing.princeton.edu/education/help-sessions">help session</a>.
