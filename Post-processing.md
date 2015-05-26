SU2 is capable of outputting solution files and other result files that can be visualized in a number of formats, including ParaView (.vtk), Tecplot (.dat for ASCII, .plt for binary), CGNS (.cgns), comma-separated values (.csv), or STL (.stl) depending on the problem.

At the end of each simulation (or at a a frequency specified by the user), SU2 will output several files that contain all of the necessary information for post-processing of results, visualization, and a restart. **Note that when the code is compiled and running in parallel, only restart files will be generated during runtime to avoid overheads.** The restart files can then be used as input to generate the visualization files using SU2_SOL. This will be done automatically if the parallel_computation.py script is used, but it can also be done manually at any time. In fact, this offers additional flexibility, as solution files in multiple formats can always be generated/regenerated as long as the original mesh and a restart file are available. SU2_SOL can be called from the common line like any of the other modules, in serial or parallel.

For a PDE analysis (direct solution), these files might look like the following:
- **flow.dat** or **flow.vtk**: full volume flow solution.
- **surface_flow.dat** or **surface_flow.vtk**: flow solution along the surface of the geometry.
- **surface_flow.csv**: comma-separated values (.csv) file containing values along the surface of the geometry.
- **history.dat** or **history.csv**: file containing the convergence history information.
- **restart_flow.dat**: restart file in an internal format for restarting simulations in SU2. Contains the flow solution at each node (description below).

For adjoint solutions, the file names are slightly different and depend on the particular objective chosen:
- **adjoint.dat** or **adjoint.vtk**: full volume adjoint solution.
- **surface_adjoint.dat** or **surface_adjoint.vtk**: adjoint solution along the surface of the geometry.
- **surface_adjoint.csv**: comma-separated values (.csv) file containing values along the surface of the geometry.
- **history.dat** or **history.csv**: file containing the convergence history information.
- **restart_adj_cd.dat**: readable restart file in an internal format for restarting this simulation in SU2. Note that the name of the objective appears in the file name. In this example, it is the coefficient of drag.

SU2 uses a simple format for the restart files. The SU2 solution format has the extension .dat, and the files are in readable ASCII format. After a header line with variable names, the solution (conservative variables) is provided at each vertex of the numerical grid, along with other meaningful quantities, such as Pressure or Mach number, and no information about the mesh elements or connectivity is included in the file. **Note: the global point index is provided at the beginning of each line, and the ordering of the points is (and must be kept) the same as in the mesh file.**

The restart files are used to restart the code from a previous solution and also to run the adjoint simulations, which require a flow solution as input. In order to run an adjoint simulation, the user must first change the name of the restart_flow.dat file to solution_flow.dat in the execution directory (these default file names can be adjusted in the config file). It is important to note that the adjoint solver will create a different adjoint restart file for each objective function, e.g. restart_adj_cd.dat.