# Installing Dedalus on ARCHER2

Some initial recipes for installing
[Dedalus](https://github.com/DedalusProject/dedalus)
on the new
[ARCHER2](https://www.archer2.ac.uk/) National UK Supercomputing Service.

This version of these instructions is valid as of Nov 2024. These instructions may need to evolve a wee
bit more.

## Recipe: Create a Dedalus Python virtual environment

This creates a Python virtual environment specifically for Dedalus. This allows
you to keep your Dedalus-related Python stuff in one place, separate from your
other Python work.

```bash
module load cray-python  # Load Python module

# Create virtual environment in my /work/... directory
export WORK=${HOME/home/work}  # (This converts my home dir to corresponding work dir: /work/[proj]/[proj]/[username])
mkdir -p $WORK/venvs
python -m venv $WORK/venvs/dedalus

# Activate virtual environment
source $WORK/venvs/dedalus/bin/activate
pip install -U pip  # Upgrade pip to latest version

# Build & install h5py, linked to Cray HDF5 libraries, compiled using Cray compiler (CC=cc)
module load cray-hdf5-parallel
CC=cc pip install --no-binary=h5py h5py
module unload cray-hdf5-parallel

# Build and install mpi4py, again compiling from source rather than using a binary
MPICC=cc pip install --no-binary=mpi4py mpi4py

# Create a temporary directory for building Dedalus.
# For example:
cd $WORK
mkdir -p dedalus-build
cd dedalus-build

# Download the latest source code snapshot from Git
git clone https://github.com/DedalusProject/dedalus
cd dedalus

# if you want to install d2, select the right branch
git switch v2_master
# otherwise continue

1. Download mpi-compat.h to the mpi4py include directory
2. 

## The following steps are required to solve an MPI-4 incompatibility issue, 
# see https://github.com/DedalusProject/dedalus/issues/303 
mkdir -p $WORK/venvs/dedalus3/include/mpi4py

# download modified dedalus files
wget https://www.maths.ed.ac.uk/~linkmann/downloads/dedalus3/d3_MPI-4_compat/mpi-compat.h
wget https://www.maths.ed.ac.uk/~linkmann/downloads/dedalus3/d3_MPI-4_compat/transposes.pyx
wget https://www.maths.ed.ac.uk/~linkmann/downloads/dedalus3/d3_MPI-4_compat/fftw_wrappers.pyx

mv mpi-compat.h $WORK/venvs/dedalus3/include/mpi4py
mv transposes.pyx $WORK/dedalus3-build/dedalus3/dedalus/core
mv fftw_wrappers.pyx $WORK/dedalus3-build/dedalus3/dedalus/libraries/fftw
## end of steps required to solve MPI-4 incompatibility issue



# Patch Dedalus' setup.py to add explicit MPI library dependency
sed -i -e "/^libraries = \[/s/]/, 'mpi']/" setup.py

# Build Dedalus extension libraries
module load cray-fftw
export FFTW_INCLUDE_PATH=$FFTW_INC
export FFTW_LIBRARY_PATH=$FFTW_DIR
export MPI_PATH=$CRAY_MPICH_DIR
CC=cc pip install .
module unload cray-fftw
```

After a successful installation, you might now want to delete your
`dedalus-build` directory.


## Deleting your Dedalus installation

Simply delete your Dedalus virtual environment as follows:

```bash
rm -r $WORK/venvs/dedalus
```
