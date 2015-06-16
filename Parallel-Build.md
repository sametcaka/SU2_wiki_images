SU2 uses the GNU automake tools to configure and build the software from source in Linux and Mac OS X environments (or with a terminal emulator on Windows). While a basic version of the code can easily be compiled, we recognize that many users will want to extend capabilities by running calculations on multiple cores and on distributed memory clusters. Luckily, the build process is largely the same, apart from a few new options and dependencies.

To illustrate an advanced build, let's assume that you would like to build SU2 for running parallel calculations. In short, you will need to make sure some extra software is available and then execute a set of commands like the following:
```
$ cd /path/to/SU2
$ ./configure --prefix=/path/to/install/SU2 CXXFLAGS="-O3" --enable-mpi --with-cc=/path/to/mpicc --with-cxx=/path/to/mpicxx
$ make -j 8 install
```

Let's break this down and discuss the configure process and options in more detail.

### Compiler Flags
You can submit flags to your compiler for building SU2 using the `CXXFLAGS` variable. The most common choices are to impose a level of compiler optimization (e.g., `-O3` for aggressive optimization) or perhaps a debug flag with `-g`. For example, a high level of compiler optimization can be set by adding 
```
CXXFLAGS="-O3"
```
to the `configure` call.

### Parallel Support
First, to build in parallel, we need to inform the configure process by including the following options:
```
--enable-mpi --with-mpicc=/path/to/mpicc --with-mpicxx=/path/to/mpicxx
```
These three options enable parallel support and specify the MPI implementation that you would like to use for building. Note here that your machine must have an implementation of the MPI standard installed (i.e., `mpicc` and `mpicxx` must be in your path). For example, common MPI flavors used by the development team are Open MPI, MPICH, and Intel MPI. Additionally, mesh partitioning software is needed to decompose the meshes when running in parallel. In order to simplify the build process, the ParMETIS graph partitioning software ships with the SU2 source, and it will be compiled and linked automatically for you when the options above are prescribed.