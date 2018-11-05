Bootstrap: docker
From: ubuntu:latest

%help
    Test container for the full-potential, all-electron FHI-aims electronic
    structure package.

%labels
    MAINTAINER William Huhn (william.paul.huhn@gmail.com)
    VERSION Stupid-User-Version

%runscript
    /opt/FHI-aims/bin/aims.scalapack.mpi.x

%post

    # Download the FHI-aims source code
    # This step requires user authentication, so do it as soon as possible
    apt-get update
    apt-get install -y git
    # As a security measure, we hardwire the known-good host public key here
    echo 'aims-git.rz-berlin.mpg.de,141.14.177.25 ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAGjz5fWz7iYO3Y+t/Bo32mVYTO3jFsyHe9L5+6Wr+LGREPN1G0dGDh8nYgsVSDV82M4NHOzRkmrDtw37vkGXIouKABJ2f5XkNgk+xyGdgVf3Bdt7P8AU2bTKY9kXpglSm6OHwOyJqDSHDXSuJNZfKrhWyhZIJsQ0D/l0KxAdy5Cy5eVXg==' >> /etc/ssh/ssh_known_hosts
    cd /opt
    rm -rf FHI-aims && mkdir FHI-aims && cd FHI-aims
    git clone https://aims-git.rz-berlin.mpg.de/aims/FHIaims.git src

    # Download and install Open MPI from source
    apt-get install -y wget \
        gcc \
        g++ \
        make \
        gfortran
    cd /tmp
    wget https://download.open-mpi.org/release/open-mpi/v2.1/openmpi-2.1.5.tar.gz
    tar -zxf openmpi-2.1.5.tar.gz && cd openmpi-2.1.5
    rm -rf build && mkdir build && cd build
    ../configure CC=gcc CXX=g++ FC=gfortran
    make && make install

    # Install Intel MKL 2019 from repo (which circumvents licensing)
    # These steps were taken from https://software.intel.com/en-us/articles/installing-intel-free-libs-and-python-apt-repo
    apt-get install -y \
        wget \
        gnupg
    wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB | apt-key add -
    sh -c 'echo deb https://apt.repos.intel.com/mkl all main > /etc/apt/sources.list.d/intel-mkl.list'
    apt-get update && apt-get install -y intel-mkl-64bit-2019.0-045

    # Create make.sys
    cd /opt/FHI-aims/src
    cat <<EOF > make.sys
###############
# Basic Flags #
###############
FC            = gfortran
FFLAGS        = -O3 -mavx -ffree-line-length-none
F90MINFLAGS   = -O0 -ffree-line-length-none
F90FLAGS      = -O3 -mavx -ffree-line-length-none
LAPACKBLAS    = -L/opt/intel/compilers_and_libraries_2019.0.117/linux/mkl/lib/intel64 \
                -lmkl_gf_lp64 -lmkl_sequential -lmkl_core

#########################
# Parallelization Flags #
#########################
USE_MPI       = yes
MPIFC         = mpif90
SCALAPACK     = -L/opt/intel/compilers_and_libraries_2019.0.117/linux/mkl/lib/intel64 \
                -lmkl_scalapack_lp64 -lmkl_blacs_openmpi_lp64

################
# C, C++ Flags #
################
USE_C_FILES   = yes
CC            = gcc
CXX           = g++
CCFLAGS       = -O3 -mavx
CXXFLAGS      = -O3 -mavx

############################
# Optional C Library Flags #
############################
USE_LIBXC     = yes
USE_SPGLIB    = yes
EOF

    # Finally, compile FHI-aims
    make -j scalapack.mpi
    make clean
    cd ../bin
    mv aims*.scalapack.mpi.x aims.scalapack.mpi.x
