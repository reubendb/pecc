# pecc
Compiler Wrapper for Automated Options Generation with pkg-config


## Description
`pecc` is a compiler wrapper utilizing [pkg-config] (https://www.freedesktop.org/wiki/Software/pkg-config/) to automatically generate the correct compiler options on the command line. This is particularly useful when combined with [Environment Module] (http://modules.sourceforge.net/), typicaly available in High-Performance Computing (HPC) and clusters. 

An example works much better to clarify the above description. Typically, without the compiler wrapper, a user needs to do something like this:
```
module load fftw
gcc ${FFTW_DIR}/include my_fft_code.c ${FFTW_DIR}/lib -lfftw3
```
or, with MPI compiler:
```       
module load fftw
mpicc ${FFTW_DIR}/include my_fft_mpicode.c ${FFTW_DIR}/lib -lfftw3_mpi -lfftw3
```
This is assuming the environment variable `$(FFTW_DIR)` is set in the modulefile for FFTW.

With `pecc`, the above compile and linking lines are much simplified for the user:
```
module load fftw
pecc my_fft_code.c
pecc my_fft_mpicode.c
```
because `pecc` automatically generates the correct compiler options behind the scene.

### How It Works
`pecc` makes compiler and linking much easier for end-users by shifting the burden to setting up the correct environment to the software installers and system administrators. However, the work needed is relatively simple. 

Using our example with FFTW above, the modulefile for FFTW only needs to few more extra environment variables:
```
prepend-path PE_PKGCONFIG_LIBS fftw
prepend-path PKG_CONFIG_PATH   [directory containing fftw.pc]
```
The file `fftw.pc` needs to be created during installation. Many software and libraries provide this already, and its content can simply be customized. The file is typically located in `${PACKAGE_DIR}/lib/pkgconfig` where $PACKAGE_DIR is the installation directory. 

For example, here is the content of `fftw.pc` that would work with our example above:
```
prefix=/software/cs400/fftw/3.3.5/centos7.2_gnu5.3.0
includedir=${prefix}/include
libdir=${prefix}/lib

Name: fftw
Description: fftw Library
URL: http://www.fftw.org
Version: 3.3.5
Cflags: -I${includedir}  
Libs: -L${libdir} -lfftw3_mpi -lfftw3_omp -lfftw3_threads -lfftw3 -lfftw3f_mpi -lfftw3f_omp -lfftw3f_threads -lfftw3f
Libs.private: -lpthreads
```
In this file we include both the douple precision,single precision, threading, and MPI libraries in a single link line to make it easier for users. Overlinking is avoided because `pecc` by default added `--as-needed` flag to the linker. But of course system administrator can choose how to customize or split the FFTW package into multiple versions (for example, it is conceivable to have separate `fftw`, `fftw-threaded`, `fftw-mpi` modules instead). 

