![Turb ONERA Pressure](http://su2.stanford.edu/github_wiki/turb_onera_pressure.png)

## Goals

Upon completing this tutorial, the user will be familiar with performing a simulation of external, viscous flow around a 3D geometry using a turbulence model. The specific geometry chosen for the tutorial is the classic ONERA M6 wing. Consequently, the following capabilities of SU2 will be showcased in this tutorial:
- Steady, 3D RANS equations 
- Spalart-Allmaras turbulence model
- Roe 2nd-order numerical scheme in space
- Euler implicit time integration
- Navier-Stokes Wall, Symmetry, and Far-field boundary conditions
- Code parallelism (optional)

This tutorial also provides an explanation for properly setting up viscous, 3D flow conditions in SU2.

## Resources

The resources for this tutorial can be found in the TestCases/rans/oneram6 directory. You will need the configuration file (turb_ONERAM6.cfg) and the mesh file (mesh_ONERAM6_turb_hexa_43008.su2).

## Tutorial

The following tutorial will walk you through the steps required when solving for the flow around the ONERA M6 using SU2. The tutorial will also address procedures for both serial and parallel computations. To this end, it is assumed you have already obtained and compiled the SU2_CFD code for a serial computation or both the SU2_CFD and SU2_PRT codes for a parallel computation. If you have yet to complete these requirements, please see the Download and Installation pages.

### Background

This test case is for the ONERA M6 wing in viscous flow. The ONERA M6 wing was designed in 1972 by the ONERA Aeordynamics Department as an experimental geometry for studying three-dimensional, high Reynolds number flows with some complex flow phenomena (transonic shocks, shock-boundary layer interaction, separated flow). It has become a classic validation case for CFD codes due to the simple geometry, complicated flow physics, and availability of experimental data. This particular study will be performed at a transonic Mach number with the 3D RANS equations in SU2.

### Problem Setup

This problem will solve the for the flow past the wing with these conditions:
- Freestream Temperature = 273.15 K
- Freestream Mach number = 0.8395
- Angle of attack (AoA) = 3.06 deg
- Reynolds number = 11720000.0
- Reynolds length = 0.64607 m

These transonic flow conditions will cause the typical "lambda" shock along the upper surface of the lifting wing.

### Mesh Description

The computational domain is a large C-type mesh with the wing half-span on one boundary in the x-z plane. The mesh consists of 43,008 interior elements and 46,417 nodes. Three boundary conditions are employed: the Navier-Stokes wall condition on the wing surface, the far-field characteristic-based condition on the far-field markers, and a symmetry boundary condition for the marker where the wing half-span is attached. The symmetry condition acts to mirror the flow about the x-z plane, reducing the size of the mesh and the computational cost. Images of the entire domain and the structured, rectangular elements on the wing surface are shown below.

![Turb ONERA Mesh](http://su2.stanford.edu/github_wiki/turb_onera_mesh_bcs.png)
Figure (1): Far-field view of the computational mesh.

![Turb ONERA Surface Mesh](http://su2.stanford.edu/github_wiki/turb_onera_surface_mesh.png)
Figure (2): Close-up view of the structured surface mesh on the upper wing surface.

### Configuration File Options

Several of the key configuration file options for this simulation are highlighted here. We now discuss the proper way to prescribe 3D, viscous flow conditions in SU2:
```
% Mach number in the farfield
MACH_NUMBER= 0.8395
%
% Angle of attack (degrees)
AoA= 3.06
%
% Side-slip angle (degrees)
SIDESLIP_ANGLE= 0.0
%
% Ratio of specific heats
GAMMA_VALUE= 1.4
%
% Specific gas constant ( J/(kg*K) )
GAS_CONSTANT= 287.87
%
% Total temperature (Kelvin)
FREESTREAM_TEMPERATURE= 273.15
%
% Reynolds number (0.0 implies no definition)
REYNOLDS_NUMBER= 11720000.0
%
% Reynolds length (dimensional)
REYNOLDS_LENGTH= 0.64607
```
The options above set the conditions for a 3D, viscous flow. The MACH_NUMBER, AoA, and SIDESLIP_ANGLE options remain the same as they appeared for the inviscid ONERA M6 tutorial, which includes a description of the freestream flow direction. For the RANS equations, SU2 assumes a calorically perfect working gas. The ratio of specific heats and specific gas constant can be explicitly chosen with the GAMMA_VALUE and GAS_CONSTANT options, respectively. 

For a viscous simulation, the numerical experiment must match the physical reality. This flow similarity is achieved by matching the REYNOLDS_NUMBER and REYNOLDS_LENGTH to the original system (assuming the Mach number and the geometry already match). Upon starting a viscous simulation in SU2, the following steps are performed to set the flow conditions internally:
 1. Store the gas constants and freestream temperature, and calculate the speed of sound.
 2. Calculate and store the freestream velocity vector from the Mach number, AoA/sideslip angle, and speed of sound from step 1.
 3. Compute the freestream viscosity from Sutherland's law and the supplied freestream temperature.
 4. Use the definition of the Reynolds number to find the freestream density from the supplied Reynolds information, freestream velocity, and freestream viscosity from step 3.
 5. Calculate the freestream pressure using the perfect gas law with the freestream temperature, specific gas constant, and freestream density from step 4.
 6. Perform any required non-dimensionalization if specified reference values are not equal to 1.0, and initialize the entire flow field with these quantities.
Notice that the freestream pressure supplied in the configuration file will be ignored for viscous computations. 

Lastly, it is important to note that this method for setting similar flow conditions requires that all inputs are in SI units, including the mesh geometry. If your mesh is not in meters, or needs to be scaled in some way to match the flow conditions, the MESH_SCALE_CHANGE option can be used:
```
% Change the scale of the numerical grid (useful to change the length units
%                                         or to re-scale the grid)
MESH_SCALE_CHANGE= 1.0
%
% Write a new mesh after reading only in serial (NO, YES)
MESH_OUTPUT= NO
%
% Mesh output file
MESH_OUT_FILENAME= mesh_out.su2
```
For this ONERA M6 case, the mesh is already in meters, so no conversion is necessary. If your mesh requires conversion, enter a non-zero value for this option, and every node in the mesh will be multiplied by this factor upon reading and storing the mesh. For example, to halve the size of the mesh, enter MESH_SCALE_CHANGE = 0.5. At the end of a simulation, the mesh *will not* be converted back to the original size. Therefore, all solution files will contain the solution and geometry in the converted state. You can also output a scaled mesh using the options above.

### Running SU2

Instructions for running this test case are given here for both serial and parallel computations.

#### In Serial

The wing mesh should easily fit on a single core machine. To run this test case, follow these steps at a terminal command line:
 1. Move to the directory containing the config file (turb_ONERAM6.cfg) and the mesh file (mesh_ONERAM6_turb.su2). Make sure that the SU2 tools were compiled, installed, and that their install location was added to your path.
 2. Run the executable by entering "SU2_CFD turb_ONERAM6.cfg" at the command line.
 3. SU2 will print residual updates with each iteration of the flow solver, and the simulation will terminate after reaching the specified convergence criteria.
 4. Files containing the results will be written upon exiting SU2. The flow solution can be visualized in ParaView (.vtk) or Tecplot (.dat for ASCII).

#### In Parallel

If SU2 has been built with parallel support, the recommended method for running a parallel simulation is through the use of the parallel_computation.py Python script. This automatically handles the domain decomposition with SU2_PRT, execution of SU2_CFD, and the merging of the decomposed files using SU2_SOL. Follow these steps to run the ONERA M6 case in parallel:
 1. Move to the directory containing the config file (turb_ONERAM6.cfg) and the mesh file (mesh_ONERAM6_turb.su2). Make sure that the SU2 tools were compiled, installed, and that their install location was added to your path.
 2. Run the python script by entering "parallel_computation.py -f turb_ONERAM6.cfg -n NP" at the command line with NP being the number of processors to be used for the simulation. The python script will automatically call SU2_PRT to perform the domain decomposition, followed by SU2_CFD to perform the simulation in parallel.
 3. SU2 will print residual updates with each iteration of the flow solver, and the simulation will terminate after reaching the specified convergence criteria.
 4. The python script will automatically call the SU2_SOL executable for merging the decomposed solution files from each processor into a single file. These files containing the results will be written upon exiting SU2. The flow solution can then be visualized in ParaView (.vtk) or Tecplot (.dat for ASCII).

### Results

Results are here given for the SU2 solution of turbulent flow over the ONERA M6 wing.

![Turb ONERA Cp A](http://su2.stanford.edu/github_wiki/turb_onera_cp_a.png)
![Turb ONERA Cp B](http://su2.stanford.edu/github_wiki/turb_onera_cp_b.png)
![Turb ONERA Cp C](http://su2.stanford.edu/github_wiki/turb_onera_cp_c.png)
![Turb ONERA Cp D](http://su2.stanford.edu/github_wiki/turb_onera_cp_d.png)

Figure (3): Comparison of Cp profiles of the experimental results of Schmitt and Carpin (red squares) against SU2 computational results (blue line) at different sections along the span of the wing. (a) y/b = 0.2, (b) y/b = 0.65, (c) y/b = 0.8, (d) y/b = 0.95.