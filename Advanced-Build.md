SU2 uses the GNU automake tools to configure and build the software from source in Linux and Mac OS X environments (or with a terminal emulator on Windows). While a basic version of the code can easily be compiled, we recognize that many users will want to extend capabilities of the suite by including external libraries or by running calculations on distributed memory clusters. Luckily, the build process is largely the same, apart from a few new options and dependencies.

To illustrate an advanced build, let's assume that you would like to build SU2 for running parallel calculations and that you will need CGNS support for mesh input. In short, you will need to make sure some extra software is available and then execute a set of commands like the following:
```
$ cd /path/to/SU2
$ ./configure --prefix=/path/to/install/SU2 CXXFLAGS="-O3" --enable-mpi 
--with-cc=/path/to/mpicc --with-cxx=/path/to/mpicxx 
--with-CGNS-lib=/path/to/CGNS/lib --with-CGNS-include=/path/to/CGNS/header
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

### CGNS Support

Support for the open, widely-used CGNS data standard has been including within SU2 to make it easy to get your own meshes into the solver. Many third-party mesh packages support export to the binary CGNS format. Moreover, the size of the mesh file (particularly for large test cases) can be significantly reduced. Users should obtain and follow the instructions supplied for building the CGNS library (Version 3.1.4 recommended currently) from the [official CGNS site](http://cgns.github.io). After compiling and installing the CGNS library on your machine, one must inform the configure process of the locations of the library file and header file(s) so that the CGNS library can be linked to the SU2 build. The following two configure options accomplish this:
```
--with-CGNS-lib=/path/to/CGNS/lib --with-CGNS-include=/path/to/CGNS/header
```

### Putting It All Together
Now that you have your dependencies and configure options all lined up, let's return to the big picture. You can perform an advanced build by executing the commands from above, which are repeated here for simplicity:
```
$ cd /path/to/SU2
$ ./configure --prefix=/path/to/install/SU2 CXXFLAGS="-O3" --enable-mpi --with-cc=/path/to/mpicc --with-cxx=/path/to/mpicxx --with-CGNS-lib=/path/to/CGNS/lib --with-CGNS-include=/path/to/CGNS/header
$ make -j 8 install
```
The `--prefix` option defines the location that the executables will be installed (in a folder named bin/ within your chosen install location from --prefix). If the `--prefix` option is not specified, the code will be installed in `/usr/local/bin`, which may require admin access. Note that we are also using the "-j N" option of the make command in order to compile SU2 in parallel using N cores. This can greatly reduce the compilation time if building on a multicore laptop, workstation, or cluster head node. While not required, here we are combining the `make` and `install` commands into one.

Make sure to note the **SU2_RUN** and **SU2_HOME** environment variables displayed at the conclusion of configure. It is recommended that you add the **SU2_RUN** and **SU2_HOME** variables to your ~/.bashrc file and update your PATH variable to include the install location ($SU2_RUN, specified by `--prefix`).