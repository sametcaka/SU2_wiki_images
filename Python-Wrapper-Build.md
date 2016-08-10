It is recommended to read the information at [[Build from Source]] first prior reading this section.

In order to use the Python Wrapper of SU2, an additional compilation step is required. The wrapper is based on the CDriver structure of SU2 and the compilation is performed with [Swig](http://www.swig.org/) that is required to be installed on your system.

## Configuration with the Python wrapper
The Py wrapper build configuration is still based on the `configure` script but has to be enabled by adding the option `--enable-PY_WRAPPER`. 

### Example
To configure a parallel build in a specified location with the Python wrapper, the command should be:

    ./configure --prefix=/path/to/install/SU2 CXXFLAGS="-O3" --enable-mpi --with-cc=/path/to/mpicc --with-cxx=/path/to/mpicxx --enable-PY_WRAPPER

followed by the classical `make -j N` and `make install` commands.

This will first compile the SU2 code, and all the required libraries from `./externals`, to generate the executables (SU2_CFD, SU2_SOL, SU2_DEF, ...). Then it will wrap the CDriver structure in order to create the Python module `SU2Solver.py` that is linked to the library `_SU2Solver.so` (also coming from the wrapper compilation). The `SU2Solver` module can be imported in a Python script so that any SU2 driver (single zone, multi zone, ...) can be instantiated and used as a classical Py object.

Make sure to note the **SU2_RUN** and **SU2_HOME** environment variables displayed at the conclusion of configure. It is recommended that you add the SU2_RUN and SU2_HOME variables to your ~/.bashrc file and update your PATH and PYTHONPATH variables to include the install location ($SU2_RUN, specified by --prefix).