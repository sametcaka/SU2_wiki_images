## Source Install Overview

This section provides details on the various tools and requirements when building the SU2 suite from the source code on Linux/Mac OS X systems. Please see the [[Simple Build]], [[Parallel Build]], or [[Parallel and CGNS Build]]  pages for concise directions on the install process.

### GNU Autoconf / Automake Tools

These tools are widely used and frequently installed with most installations of Linux and Max OS X. Please check your system to ensure these tools are available prior to installation. Starting from release 3.1.0 and onward, the required versions of autotools are included inside the externals/ directory, and a bootstrap script is included in the SU2/ directory for quickly building them and reseting the makefile structure if they are not available on your system or are an earlier version. Simply run ./bootstrap in the SU2/ root directory (you may need to adjust your PATH after running this script) to build the appropriate dependencies and reset the makefile structure (rather than calling autoreconf on your own). 

### Compilers

Installing SU2 from source requires a C++ compiler. The GNU compilers (gcc/g++) are open-source, widely used, and reliable for building SU2. The Intel compiler set has been optimized to run on Intel hardware and has also been used successfully by the development team to build the source code, though it is commercially licensed. The Apple LLVM compiler (Clang) is also commonly used by the developers.
- GNU gcc / g++
- Intel icpc
- Apple LLVM (Clang)

### Parallel Tools

The ParMETIS graph partitioning software and an MPI implementation are required to compile and run SU2 in parallel. The source for ParMETIS is shipped with SU2 and can be found in the externals/ directory. ParMETIS will automatically be built and linked if you set 
```
--enable-mpi --with-mpicc=/path/to/mpicc --with-mpicxx=/path/to/mpicxx
``` 
in your configure options, which requests a build of the parallel version of the code with the specified MPI implementation on your machine. An implementation of the Message Passing Interface (MPI) standard is required for building the parallel version, and a number of available implementations are linked from the main installation page. In particular, you will need both a C (for ParMETIS) and C++ (for SU2) MPI implementation.

## Configuration 

Before building, SU2 must run the configuration script that will scan your system for the necessary prerequisites and generate the appropriate makefiles. The simplest version of SU2 can be configured by running configure with no arguments, or 
```
$ cd /path/to/SU2/
$ ./configure
```
This will configure serial (non-MPI) versions of all SU2 modules without external library support and a moderate level of compiler optimization. The configure tool will attempt to find a C++ compiler on your system and set some default flags if none are specified. It is strongly recommended, however, that the environment variables used by configure be set before configuring (especially CXX and CXXFLAGS). Numerous flags are available to activate or deactivate optional features. These include CGNS support, compiler flags for optimization and fine tuning, and the selection of specific programs to build or ignore (usually only required by developers). A complete list of optional features and relevant environment variables is shown by running 
```
$ ./configure --help
```
For example, to configure SU2 for parallel calculations (i.e., with ParMETIS and MPI) along with CGNS and TecIO support and a high level of compiler optimization, the configure command might look like this (replace with specific paths on your system):
```
$ ./configure --prefix=/path/to/install/SU2 --enable-mpi --with-cc=/path/to/mpicc 
--with-cxx=/path/to/mpicxx CXXFLAGS="-O3" --with-CGNS-lib=/path/to/CGNS/lib 
--with-CGNS-include=/path/to/CGNS/header
```
When defining the installation path via the --prefix option, note that you will need write access to the destination folder when installing after compiling (see below). You can switch to a privileged user (or sudo) before installing if necessary. You do not need to rerun the configuration step unless you modify the options, i.e. you would like to change compilers, dependencies, etc. Updating the source code and rebuilding does not require reconfiguring: simply rerun the make and make install commands.

## Compiling

After configuring, compile SU2 by calling
```
$ make
```
This compiles the code using the makefiles that were automatically generated from the results of the configure process. You can also use the "-j N" option of the make command in order to compile SU2 in parallel using N cores, i.e., run 
```
$ make -j 8
```
to compile using 8 cores. This can greatly reduce the compilation time if building on a multi-core laptop, workstation, or cluster head node. If no errors are encountered, you are ready to install. 

## Installing

After compiling, you are ready to install SU2. To install, enter the command 
```
$ make install
```
This will copy the programs and Python scripts comprising the SU2 suite to the folder that you selected with the `--prefix` option during configuration (/usr/local/bin by default if `--prefix` was not defined). As noted above, you will need write access to the destination folder to install. 

## Cleaning

To clean the SU2 source tree (remove all intermediate object files), enter the command 
```
make clean
```
in the root directory of the SU2 source distribution. This is recommended before rebuilding if you modify the configuration and / or update the source code.

## Environment Variables

After installing the code (but before running it), define the **SU2_HOME** and **SU2_RUN** environment variables, and update your PATH with SU2_RUN. For the basic installation, these values will be displayed at the conclusion of `./configure` from the steps above. These environment variables are useful for running SU2 from different working directories, and they are needed for some of the Python framework.

`$SU2_RUN` should point to the folder where all binaries and python scripts were installed (by default, in a folder named `bin/` within your chosen install location from `--prefix`). `$SU2_HOME` should point to the root directory of your SU2 source distribution. 

For example, add these lines to your ~/.bashrc (linux) or ~/.bash_profile (macosx) file:
```
export SU2_RUN="your/prefix/bin"
export SU2_HOME="/path/to/SU2"
export PATH=$PATH:$SU2_RUN
```
If you plan to use the Python scripts for parallel calculations or design and optimization tasks, you may also want to include $SU2_RUN in your Python path:
```
export PYTHONPATH=$PYTHONPATH:$SU2_RUN
```

That's it: you're now ready to run SU2! Check out the Quick Start and additional tutorials.

### Notes for Mac OS X Users

In order to prepare your Mac for compiling/running/developing SU2, you will need to download Xcode from the App Store. After obtaining Xcode, you should also install the Developer Tools package from inside of the Xcode distribution. This contains tools like make and the LLVM compiler, and after installing the dev tools, they will be available from within the native Terminal app. A prebuilt version of the GNU compilers for Mac OS X can be found here. Environment variables, such as SU2_RUN and SU2_HOME, on Mac OS X can be set within ~/.bash_profile (this file may not exist by default, so you can create it yourself if necessary). Lastly, note also that project files for developing the SU2 modules in Xcode are provided inside the SU2/SU2_IDE/Xcode/ directory of the SU2 source distribution. 

To have the TecIO library (the source ships with SU2) automatically built and linked on Mac, include the following configure options --enable-tecio and CPPFLAGS='-I/opt/X11/include' to make sure that it can find the correct X11 dependencies.  For platforms other than Mac OS X, the configure script must also be able to find the X11 library headers for the TecIO library to be built: if --enable-tecio is used in the configure step and the output states that TecIO will not be built, it is likely that the configure program was unable to locate the X11 header files.