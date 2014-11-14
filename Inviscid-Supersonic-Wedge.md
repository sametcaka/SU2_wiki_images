## Goals

Upon completing this tutorial, the user will be familiar with performing a simulation of supersonic, inviscid flow over a 2-D geometry. The specific geometry chosen for the tutorial is a simple wedge. Consequently, the following capabilities of SU2 will be showcased in this tutorial:
- Steady, 2-D Euler equations 
- Multigrid
- JST numerical scheme in space
- Euler implicit time integration
- Far-field, Outlet, and Euler Wall boundary conditions
The intent of this tutorial is to introduce a simple, inviscid flow problem that will allow users to become familiar with using a CGNS mesh. This will require SU2 to be built with CGNS support, and some new options in the configuration file related to CGNS meshes will be discussed.

## Resources

The resources for this tutorial can be found in the TestCases/euler/wedge/ directory. You will need the configuration file (inv_wedge_HLLC.cfg) and the mesh file (mesh_wedge_inv.su2). This case can also be run with a CGNS mesh (mesh_wedge_inv.cgns). The rest of this tutorial will be written assuming that the CGNS mesh is used, although the SU2 mesh is used by default in the config file.

## Tutorial

The following tutorial will walk you through the steps required when solving for the flow using SU2. It is assumed you have already obtained and compiled SU2_CFD with CGNS support. If you have yet to complete these requirements, please see the Download and Installation pages.

### Background

This example uses a 2-D geometry which features a wedge along the solid lower wall. In supersonic flow, this wedge will create an oblique shock, and its properties can be predicted from the oblique-shock relations for a perfect gas (can be found in almost any text on compressible fluids). This is a very common test case for CFD codes due to its simplicity along with the ability to verify the code against the oblique-shock relations.

### Problem Setup

This problem will solve for the flow over the wedge with these conditions:
- Freestream Pressure = 101325.0 N/m2
- Freestream Temperature = 273.15 K
- Freestream Mach number = 2.0
- Angle of attack (AoA) = 0.0 deg

### Mesh Description

The wedge mesh is a structured mesh (75x50) of rectangular elements with a total of 3,750 nodes. The lower wall of the geometry is solid and has a 10o wedge starting at x = 0.5. Figure (1) shows the mesh with the boundary markers and flow conditions highlighted.

SU2 > Tutorial 9 - Inviscid Supersonic Wedge > mesh&bcs.png (mesh&bcs.png)
Figure (1): The computational mesh with boundary conditions highlighted.

For this test case, the inlet and upper markers will be set to the far-field boundary condition, while the outlet marker will be set to the outlet condition. In supersonic flow, all characteristics point into the domain at the entrance (inlet & upper), so all flow quantities can be specified (no information travels upstream). This justifies the use of the far-field condition at these boundaries. At the exit, however, all characteristics are outgoing, meaning that no information about the exit conditions is required. Therefore, the outlet marker is set to the outlet boundary condition which, in supersonic flow, simply extrapolates the flow variables from the interior domain to the exit. In short, any back pressure can be supplied to the MARKER_OUTLET boundary condition in the configuration file, because it is ignored for this specific supersonic case.

### Configuration File Options

Several of the key configuration file options for this simulation are highlighted here.
It is recommended that you first read through the description for building and running SU2 with CGNS support. Here we will discuss a few options in the wedge configuration file as a practical example for getting your own CGNS mesh files working:
```
%
% Mesh input file
MESH_FILENAME= mesh_wedge_inv.cgns
%
% Mesh input file format (SU2, CGNS, NETCDF_ASCII)
MESH_FORMAT= CGNS
%
% Convert a CGNS mesh to SU2 format (YES, NO)
CGNS_TO_SU2= YES
%
% Mesh output file
MESH_OUT_FILENAME= mesh_out.su2
```
To use the supplied CGNS mesh, simply enter the filename and make sure that the MESH_FORMAT option is set to CGNS. A converter for creating .su2 meshes from CGNS meshes is built directly into SU2. To use this feature, set the CGNS_TO_SU2 flag to YES and provide a name for the converted mesh (for this case, it has been set to "mesh_out.su2"). SU2 will convert the mesh during the pre-processing after reading in the original CGNS mesh and print the new mesh file to the current working directory. The output written to the console during this process might look like the following for the supersonic wedge mesh:
```
---------------------- Read grid file information -----------------------
CGNS mesh file format.
Reading the CGNS file: mesh_wedge_inv.cgns
CGNS file contains 1 database(s).
Database 1, Base: 1 zone(s), cell dimension of 2, physical dimension of 3.
Zone 1, dom-1: 3750 vertices, 3626 cells, 0 boundary vertices.
Reading grid coordinates...
Number of coordinate dimensions is 3.
Reading CoordinateX values from file.
Reading CoordinateY values from file.
Reading CoordinateZ values from file.
Reading connectivity information...
Number of connectivity sections is 5.
Reading section QuadElements of element type Rectangle
 starting at 1 and ending at 3626.
Reading section inlet of element type Line
 starting at 3627 and ending at 3675.
Reading section lower of element type Line
 starting at 3676 and ending at 3749.
Reading section outlet of element type Line
 starting at 3750 and ending at 3798.
Reading section upper of element type Line
 starting at 3799 and ending at 3872.
Successfully closed the CGNS file.
Writing SU2 mesh file: mesh_out.su2.
Successfully wrote the SU2 mesh file.
```
SU2 prints out information on the CGNS mesh including the filename, the number of points, and the number of elements. Another useful piece of information is the listing of the zone sections within the mesh. These descriptions give the type of elements for the section as well as any name given to it. For instance, when the inlet boundary information is read, SU2 prints "Reading section inlet of element type Line" to the console. This information can be used to verify that your mesh is being read correctly, or even to help you remember, or learn for the first time, the names for each of the boundary markers. Lastly, if conversion to the .su2 format is requested, SU2 will communicate whether the .su2 mesh was successfully written.

### Running SU2

The wedge simulation is small and will execute quickly on a single workstation or laptop, and this case will be run in a serial fashion. To run this test case, follow these steps at a terminal command line:
 1. Move to the directory containing the config file (inv_wedge_HLLC.cfg) and the mesh file (mesh_wedge_inv.cgns or mesh_wedge_inv.su2). Make sure that the SU2 tools were compiled (with CGNS support if necessary), installed, and that their install location was added to your path.
 2. Run the executable by entering "SU2_CFD inv_wedge_HLLC.cfg" at the command line.
 3. SU2 will print residual updates with each iteration of the flow solver, and the simulation will finish after reaching the specified convergence criteria.
 4. Files containing the results will be written upon exiting SU2. The flow solution can be visualized in ParaView (.vtk) or Tecplot (.dat). The output format is specified in the config file.

### Results

The following images show some SU2 results for the supersonic wedge problem.

SU2 > Tutorial 9 - Inviscid Supersonic Wedge > wedge_mach.png (wedge_mach.png)
Figure (2): Mach contours showing the oblique shock for supersonic flow over a wedge.

SU2 > Tutorial 9 - Inviscid Supersonic Wedge > wedge_pressure.png (wedge_pressure.png)
Figure (3): Pressure contours (N/m2) for supersonic flow over a wedge.