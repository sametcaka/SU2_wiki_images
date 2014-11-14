## Goals

Upon completing this tutorial, the user will be familiar with performing an optimal shape design of a 3D geometry. The initial geometry chosen for the tutorial is a ONERA M6 fixed wing at transonic speed in inviscid fluid. The following SU2 tools will be showcased in this tutorial:
- **SU2_CFD** - performs the direct and the adjoint flow simulations
- **SU2_DOT** - projects the adjoint surface sensitivities into the design space to obtain the gradient
- **SU2_DEF** - deforms the geometry and mesh with changes in the design variables during the shape optimization process
- **shape_optimization.py** - automates the entire shape design process by executing the SU2 tools and optimizer

## Resources

The resources for this tutorial can be found in the TestCases/optimization_euler/steady_oneram6 directory. You will need the configuration file (inv_ONERAM6_adv.cfg) and the mesh file (mesh_ONERAM6_inv_FFD.su2), note that the mesh file contains information about the definition of the Free Form Deformation (FFD) used for the definition of 3D design variables.

## Tutorial

The following tutorial will walk you through the steps required when performing 3D shape design using SU2, and FFD tools. It is assumed that you have already obtained and compiled SU2_CFD, SU2_DOT, and SU2_DEF. The design loop is driven by the shape_optimization.py script, and thus Python along with the NumPy and SciPy Python modules are required for this tutorial. If you have yet to complete these requirements, please see the Download and Installation pages.

### Background

This example uses a 3D fixed-wing geometry (initially the ONERA M6) at transonic speed in air (inviscid calculation). The design variables are defined using the FFD methodology, and at the end of the mesh_ONERAM6_inv_FFD.su2 file, the description of the FFD box is provided:
```
FFD_NBOX= 1
FFD_NLEVEL= 1
FFD_TAG= WING
FFD_LEVEL= 0
FFD_DEGREE_I= 10
FFD_DEGREE_J= 8
FFD_DEGREE_K= 1
FFD_PARENTS= 0
FFD_CHILDREN= 0
FFD_CORNER_POINTS= 8
-0.0403 0       -0.04836
0.8463  0       -0.04836
1.209   1.2896  -0.04836
0.6851  1.2896  -0.04836
-0.0403 0       0.04836
0.8463  0       0.04836
1.209   1.2896  0.04836
0.6851  1.2896  0.04836
FFD_CONTROL_POINTS=0
FFD_SURFACE_POINTS=0
```
Note that, only the corners of the box and the polynomial degree in each direction are provided. The tag for the FFD box can be specified as a string name. Here, we choose "WING," as we are placing the FFD box around the wing. The provided mesh file is ready for optimization, but in the case that a user is specifying their own FFD box for a problem, the SU2_DEF module should be called after defining the options above (the levels, tag, degrees, and corner points) with the DV_KIND option set to FFD_SETTING in order to compute and write the FFD_CONTROL_POINTS and FFD_SURFACE_POINTS information to the grid file. Note that this mapping for the FFD variables only needs to be computed and stored once in the mesh file before performing design. We will describe this below.

### Problem Setup

The goal of the design process is to minimize the coefficient of drag (Cd) by changing the shape of the wing, and as design variables, we will use the z-coordinate of the FFD control point position. As the shock wave is located on the upper side of the wing, only the control points on the upper side will be used as design variables.

### Mesh Description and Preprocessing 

The mesh consists of a far-field boundary divided in three surfaces (XNORMAL_FACES, ZNORMAL_FACES, YNORMAL_FACES), an Euler wall divided in three surfaces (UPPER_SIDE, LOWER_SIDE, TIP) and a symmetry plane (SYMMETRY_FACE). The specific wing is the ONERA M6, and more information on this simulation can be found in the configuration file. The surface mesh can be seen in Figure (1).
 
SU2 > Tutorial 8 - Constrained Optimal Shape Design of a Fixed Wing > grid.jpg (grid.jpg)
Figure (1): View of the initial surface computational mesh.

SU2 > Tutorial 8 - Constrained Optimal Shape Design of a Fixed Wing > FFD.jpg (FFD.jpg)
Figure (2): View of the initial FFD box, control points and the surface mesh.
 
The mesh file that is provided for this test case already contains the FFD information. However, if you are interested in repeating this process for your own design cases, it is necessary to calculate the position of the control points and the parametric coordinates. To do so, follow these steps at a terminal command line after defining the tags, degrees, and corner points for your FFD box (we'll use the ONERA M6 as an example):
 1. Move to the directory containing the config file (inv_ONERAM6_adv.cfg) and the mesh file (mesh_ONERAM6_inv_FFD.su2). Make sure that the SU2 tools were compiled, installed, and that their install location was added to your path.
 2. Check that DV_KIND= FFD_SETTING in the configuration file. 
 3. Execute SU2_DEF by entering "SU2_DEF inv_ONERAM6.cfg" at the command line.
 4. After completing the FFD mapping process, a mesh file named "mesh_out.su2" is now in the directory. Rename that file to "mesh_ONERAM6_inv_FFD.su2". Note that this new mesh file contains all the details of the FFD method.

With this preprocessing, the position of the control points and the parametric coordinates have been calculated. A representative FFD box and the control points can be seen in Figure (2). 

### Configuration File Options

Several of the key configuration file options for this simulation are highlighted here. We will use a similar flow problem as that feature in the previous ONERA M6 tutorials:
```
% ----------- COMPRESSIBLE AND INCOMPRESSIBLE FREE-STREAM DEFINITION ----------%
%
% Mach number (non-dimensional, based on the free-stream values)
MACH_NUMBER= 0.8395
%
% Angle of attack (degrees)
AoA= 3.06
%
% Free-stream pressure (101325.0 N/m^2 by default, only for Euler equations)
FREESTREAM_PRESSURE= 101325.0
%
% Free-stream temperature (273.15K by default)
FREESTREAM_TEMPERATURE= 273.15
% 
% ---------------------- REFERENCE VALUE DEFINITION ---------------------------%
%
% Conversion factor for converting the grid to meters
CONVERT_TO_METER= 1.0
%
% Reference area for force coefficients (0 implies automatic calculation)
REF_AREA= 0
%
% Reference pressure (101325.0 N/m^2 by default)
REF_PRESSURE= 101325.0
%
% Reference temperature (273.15 K by default)
REF_TEMPERATURE= 273.15
Note the name of the mesh file, which includes the FFD information. Tecplot will be used for the visualization.
% Mesh input file
MESH_FILENAME= mesh_ONERAM6_inv_FFD.su2
%
% Output file format (PARAVIEW, TECPLOT)
OUTPUT_FORMAT= TECPLOT 
Optimal shape design specification:
% --------------------- OPTIMAL SHAPE DESIGN DEFINITION -----------------------%
% Optimization constraint functions with scaling factors, separated by semicolons
% ex= (Objective = Value ) * Scale, use '>','<','='
OPT_CONSTRAINT= (LIFT > 0.2864) * 0.1; (MAX_THICKNESS_SEC1 > 0.0570) * 0.1; (MAX_THICKNESS_SEC2 > 0.0513) * 0.1; (MAX_THICKNESS_SEC3 > 0.0457) * 0.1; (MAX_THICKNESS_SEC4 > 0.0399) * 0.1; (MAX_THICKNESS_SEC5 > 0.0343) * 0.1
%
% Optimization design variables, separated by semicolons
DEFINITION_DV= ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 0, 0, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 1, 0, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 2, 0, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 3, 0, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 4, 0, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 0, 1, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 1, 1, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 2, 1, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 3, 1, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 4, 1, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 0, 2, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 1, 2, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 2, 2, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 3, 2, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 4, 2, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 0, 3, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 1, 3, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 2, 3, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 3, 3, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 4, 3, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 0, 4, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 1, 4, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 2, 4, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 3, 4, 1, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | 0, 4, 4, 1, 0.0, 0.0, 1.0 )
...
```
Here, we define the objective function for the optimization as drag with a lift constraint and thickness constraints along 5 sections of the wing. The DEFINITION_DV is the list of design variables. For this problem, we want to minimize the drag by changing the position of the control points of the control box. To do so, we define the set of FFD control points. Each design variable is separated by a semicolon. The first value in the parentheses is the variable type, which is 7 for control point movement. The second value is the scale of the variable (typically left as 1.0). The name between the vertical bars is the marker tag(s) where the variable deformations will be applied. The final seven values in the parentheses are the particular information about the deformation: identification of the FFD tag, the i, j, and k index of the control point, and the allowed x, y, and z movement direction of the control point. Note that other types of design variables have their own specific input format. 

### Running SU2

A continuous adjoint methodology for obtaining surface sensitivities is implemented for several equation sets within SU2. After solving the direct flow problem, the adjoint problem is also solved. The adjoint method offers an efficient approach for calculating the gradient of an objective function with respect to a large set of design variables. This leads directly to a gradient-based optimization framework. With each design iteration, the direct and adjoint solutions are used to compute the objective function and gradient, and the optimizer drives the shape changes with this information in order to minimize the objective. Two other SU2 tools are used to compute the gradient from the adjoint solution (SU2_DOT) and deform the computational mesh (SU2_DEF) during the process. To run this design case, follow these steps at a terminal command line:
 1. Execute the shape optimization script by entering "shape_optimization.py -f inv_ONERAM6_adv.cfg" at the command line, add "-n 6" in case you want to run the optimization in parallel (6 processors). Again, note that Python, NumPy, and SciPy are all required to run the script.
 2. The python script will drive the optimization process by executing flow solutions, adjoint solutions, gradient projection, and mesh deformation in order to drive the design toward an optimum. The optimization process will cease when certain tolerances set within the SciPy optimizer are met. Note that is is possible to start the optimization from a pre-converged solution (direct and adjoint problem), in that case the following change should be done in the configuration file: RESTART_SOL= YES.
 3. Solution files containing the flow and surface data will be written for each flow solution and adjoint solution and can be found in the DESIGNS directory that is created. The file named history_project.plt (or history_project.csv for ParaView) will contain the functional values of interest resulting from each evaluation during the optimization.

### Results 

The following are representative results for this transonic shape design example with the ONERA M6 geometry as a baseline.

SU2 > Tutorial 8 - Constrained Optimal Shape Design of a Fixed Wing > original.jpg (original.jpg)
Figure (3): Pressure contours showing transonic shocks on the initial design.

SU2 > Tutorial 8 - Constrained Optimal Shape Design of a Fixed Wing > designed.jpg (designed.jpg)
Figure (4): Pressure contours around the final airfoil design (Reduced shocks).

SU2 > Tutorial 8 - Constrained Optimal Shape Design of a Fixed Wing > history.jpg (history.jpg)
Figure (5): Optimization history.
