![Rotating NACA 0012 Pressure](http://su2.stanford.edu/github_wiki/rotating_pressure.png)

## Goals

Upon completing this tutorial, the user will be familiar with performing an optimal shape design of a 2D geometry. The initial geometry chosen for the tutorial is a NACA 0012 airfoil that is rotating at transonic speed in inviscid fluid. This tutorial is mean to be an introduction for using the components of SU2 for shape design. Consequently, the following SU2 tools will be showcased in this tutorial:
- **SU2_CFD** - performs the direct and the adjoint flow simulations
- **SU2_DOT** - projects the adjoint surface sensitivities into the design space to obtain the gradient
- **SU2_DEF** - deforms the geometry and mesh with changes in the design variables during the shape optimization process
- **shape_optimization.py** - automates the entire shape design process by executing the SU2 tools and optimizer

## Resources

The resources for this tutorial can be found in the TestCases/optimization_euler/rotating_naca0012 directory. You will need the configuration file (rotating_NACA0012.cfg) and the mesh file (mesh_NACA0012_rot.su2).

## Tutorial

The following tutorial will walk you through the steps required when performing shape design for the rotating airfoil using SU2. It is assumed that you have already obtained and compiled SU2_CFD, SU2_DOT, and SU2_DEF. The design loop is driven by the shape_optimization.py script, and thus Python along with the NumPy and SciPy Python modules are required for this tutorial. If you have yet to complete these requirements, please see the Download and Installation pages.

### Background

This example uses a 2D airfoil geometry (initially the NACA 0012) which is rotating counter-clockwise in still air.

### Problem Setup

This numerical experiment for the rotating airfoil was set up such that transonic shocks would appear on the upper and lower surfaces causing drag. The goal of the design process is to minimize the coefficient of drag (Cd) by changing the shape of the airfoil with a constraint on the thickness of the airfoil. In other words, we would like to eliminate the shocks along the airfoil surface. The details of the rotating airfoil experiment are given in Figure (1).

![Rotating NACA 0012 Experiment](http://su2.stanford.edu/github_wiki/rotating_experiment.png)
Figure (1): Details for the rotating airfoil numerical experiment.

### Mesh Description

The mesh from the Quick Start Tutorial is used again here as the initial geometry. It consists of a far-field boundary and an Euler wall along the airfoil surface. The specific airfoil is the NACA 0012, and more information on this airfoil can be found in the Quick Start Tutorial. The mesh can be seen in Figure (2).

![Rotating NACA 0012 Mesh](http://su2.stanford.edu/github_wiki/rotating_mesh.png)
Figure (2): Far-field and zoom view of the initial computational mesh.

### Configuration File Options

Several of the key configuration file options for this simulation are highlighted here. SU2 features a number of ways to compute flows on dynamic grids, including moving wall boundary conditions, calculations in a rotating frame, or unsteady flows on dynamic meshes. Specifying a simulation in a rotating frame (non-inertial) can be accomplished with the following options:
```
% ----------------------- DYNAMIC MESH DEFINITION -----------------------------%
%
% Dynamic mesh simulation (NO, YES)
GRID_MOVEMENT= YES
%
% Type of dynamic mesh (NONE, RIGID_MOTION, DEFORMING, ROTATING_FRAME,
%                       MOVING_WALL)
GRID_MOVEMENT_KIND= ROTATING_FRAME
%
% Motion mach number (non-dimensional). Used for intitializing a viscous flow
% with the Reynolds number and for computing force coeffs. with dynamic meshes.
MACH_MOTION= 0.7958
%
% Coordinates of the motion origin
MOTION_ORIGIN_X= 0.5
MOTION_ORIGIN_Y= -32.0
MOTION_ORIGIN_Z= 0.0
%
% Angular velocity vector (rad/s) about the motion origin
ROTATION_RATE_X = 0.0
ROTATION_RATE_Y = 0.0
ROTATION_RATE_Z = 8.25
```
In SU2, the governing equations (Euler, Navier-Stokes, and RANS) can be solved within a rotating reference frame which offers an efficient, steady solution method for flows around rotating bodies in axisymmetric flow. A simulation can be executed in a rotating frame by choosing the ROTATING_FRAME option for GRID_MOVEMENT_KIND. Two additional pieces of data must be supplied: the location of the rotation center (x,y,z) in the coordinate system of the computational mesh, and the angular velocity (rotation rate around the x-axis, rotation rate around the y-axis, rotation rate around the z-axis). 

For the rotating airfoil problem, the airfoil has a chord of 1 meter and the origin of the coordinate system (0,0,0) is at the leading edge of the airfoil. We set the rotation center to be 32 chord lengths below the center of the airfoil. The angular velocity is in the z-direction (out of the page) and has the units of radians per second. Lastly, since the Mach number is 0.0 for this problem, we will use the value given in MACH_MOTION to compute the non-dimensional force coefficients.

Optimal shape design specification:
```
% --------------------- OPTIMAL SHAPE DESIGN DEFINITION -----------------------%
% Optimization objective function with optional scaling factor
% ex= Objective * Scale
OPT_OBJECTIVE= DRAG * 0.001
%
% Optimization constraint functions with scaling factors, separated by semicolons
% ex= (Objective = Value ) * Scale, use '>','<','='
OPT_CONSTRAINT= ( MAX_THICKNESS > 0.12 ) * 0.001
%
% List of design variables (Design variables are separated by semicolons)
DEFINITION_DV= ( 1, 1.0 | airfoil | 0, 0.961538461538 ); ( 1, 1.0 | airfoil | 0, 0.923076923077 ); ( 1, 1.0 | airfoil | 0, 0.884615384615 ); ( 1, 1.0 | airfoil | 0, 0.846153846154 ); ( 1, 1.0 | airfoil | 0, 0.807692307692 ); ( 1, 1.0 | airfoil | 0, 0.769230769231 ); ( 1, 1.0 | airfoil | 0, 0.730769230769 ); ( 1, 1.0 | airfoil | 0, 0.692307692308 ); ( 1, 1.0 | airfoil | 0, 0.653846153846 ); ( 1, 1.0 | airfoil | 0, 0.615384615385 ); ( 1, 1.0 | airfoil | 0, 0.576923076923 ); ( 1, 1.0 | airfoil | 0, 0.538461538462 ); ( 1, 1.0 | airfoil | 0, 0.5 ); ( 1, 1.0 | airfoil | 0, 0.461538461538 ); ( 1, 1.0 | airfoil | 0, 0.423076923077 ); ( 1, 1.0 | airfoil | 0, 0.384615384615 ); ( 1, 1.0 | airfoil | 0, 0.346153846154 ); ( 1, 1.0 | airfoil | 0, 0.307692307692 ); ( 1, 1.0 | airfoil | 0, 0.269230769231 ); ( 1, 1.0 | airfoil | 0, 0.230769230769 ); ( 1, 1.0 | airfoil | 0, 0.192307692308 ); ( 1, 1.0 | airfoil | 0, 0.153846153846 ); ( 1, 1.0 | airfoil | 0, 0.115384615385 ); ( 1, 1.0 | airfoil | 0, 0.0769230769231 ); ( 1, 1.0 | airfoil | 0, 0.0384615384615 ); ( 1, 1.0 | airfoil | 1, 0.0384615384615 ); ( 1, 1.0 | airfoil | 1, 0.0769230769231 ); ( 1, 1.0 | airfoil | 1, 0.115384615385 ); ( 1, 1.0 | airfoil | 1, 0.153846153846 ); ( 1, 1.0 | airfoil | 1, 0.192307692308 ); ( 1, 1.0 | airfoil | 1, 0.230769230769 ); ( 1, 1.0 | airfoil | 1, 0.269230769231 ); ( 1, 1.0 | airfoil | 1, 0.307692307692 ); ( 1, 1.0 | airfoil | 1, 0.346153846154 ); ( 1, 1.0 | airfoil | 1, 0.384615384615 ); ( 1, 1.0 | airfoil | 1, 0.423076923077 ); ( 1, 1.0 | airfoil | 1, 0.461538461538 ); ( 1, 1.0 | airfoil | 1, 0.5 ); ( 1, 1.0 | airfoil | 1, 0.538461538462 ); ( 1, 1.0 | airfoil | 1, 0.576923076923 ); ( 1, 1.0 | airfoil | 1, 0.615384615385 ); ( 1, 1.0 | airfoil | 1, 0.653846153846 ); ( 1, 1.0 | airfoil | 1, 0.692307692308 ); ( 1, 1.0 | airfoil | 1, 0.730769230769 ); ( 1, 1.0 | airfoil | 1, 0.769230769231 ); ( 1, 1.0 | airfoil | 1, 0.807692307692 ); ( 1, 1.0 | airfoil | 1, 0.846153846154 ); ( 1, 1.0 | airfoil | 1, 0.884615384615 ); ( 1, 1.0 | airfoil | 1, 0.923076923077 ); ( 1, 1.0 | airfoil | 1, 0.961538461538 )
```
Here, we define the objective function for the optimization as drag without any constraints. The scale value of 0.01 is chosen to aid the optimizer in taking a physically appropriate first step (i.e., not too large that the subsequent calculations go unstable due to a non-physical deformation). We are imposing a constraint on the maximum thickness, but it is also possible, for instance, to add a lift constraint.

The DEFINITION_DV is the list of design variables. For the rotating airfoil problem, we want to minimize the drag by changing the surface profile shape. To do so, we define a set of Hicks-Henne bump functions. Each design variable is separated by a semicolon, although note that there is no final semicolon at the end of the list. The first value in the parentheses is the variable type, which is 1 for a Hicks-Henne bump function. The second value is the scale of the variable (typically left as 1.0). The name between the vertical bars is the marker tag where the variable deformations will be applied. Only the airfoil surface will be deformed in this problem. The final two values in the parentheses specify whether the bump function is applied to the upper (1) or lower (0) side and the x-location of the bump, respectively. Note that there are many other types of design variables available in SU2, and each has their own specific input format.

### Running SU2

A continuous adjoint methodology for obtaining surface sensitivities is implemented for several equation sets within SU2. For this problem, a formulation based on the Euler equations in a rotating reference frame is used. After solving the direct flow problem, the adjoint problem is also solved which offers an efficient approach for calculating the gradient of an objective function with respect to a large set of design variables. This leads directly to a gradient-based optimization framework. With each design iteration, the direct and adjoint solutions are used to compute the objective function and gradient, and the optimizer drives the shape changes with this information in order to minimize the objective. Two other SU2 tools are used to compute the gradient from the adjoint solution (SU2_DOT) and deform the computational mesh (SU2_DEF) during the process.

To run this design case, follow these steps at a terminal command line:
 1. Move to the directory containing the config file (rot_NACA0012.cfg) and the mesh file (mesh_NACA0012_rot.su2). Assuming that SU2 tools were compiled, installed, and that their install location was added to your path, the shape_optimization.py script, SU2_CFD, SU2_DOT, and SU2_DEF should all be available.
 2. Execute the shape optimization script by entering "python shape_optimization.py -f rot_NACA0012.cfg" at the command line. Again, note that Python, NumPy, and SciPy are all required to run the script.
 3. The python script will drive the optimization process by executing flow solutions, adjoint solutions, gradient projection, and mesh deformation in order to drive the design toward an optimum. The optimization process will cease when certain tolerances set within the SciPy optimizer are met.
 4. Solution files containing the flow and surface data will be written for each flow solution and adjoint solution and can be found in the DESIGNS directory that is created. The flow solutions are in the DESIGNS/DSN_*/DIRECT/ directories. The file named history_project.plt (or history_project.csv for ParaView) will contain the functional values of interest resulting from each evaluation during the optimization.

### Results

![Rotating NACA 0012 Mach](http://su2.stanford.edu/github_wiki/rotating_mach.png)
Figure (3): Mach number contours for the airfoil rotating in still air.

![Rotating NACA 0012 Adjoint](http://su2.stanford.edu/github_wiki/rotating_psi_density.png)
Figure (4): Adjoint density contours on the initial design.

![Rotating NACA 0012 Pressure](http://su2.stanford.edu/github_wiki/rotating_pressure.png)
Figure (5): Pressure contours showing transonic shocks on the initial design.

![Rotating NACA 0012 Final Contour](http://su2.stanford.edu/github_wiki/rotating_final_contour.png)
Figure (6): Pressure contours around the final airfoil design with MAX_THICKNESS > 0.12. Note that the shocks have essentially been removed during the design process.

![Rotating NACA 0012 Final Cp](http://su2.stanford.edu/github_wiki/rotating_final_cp.png)
Figure (7): Cp distribution and profile shape comparison for the initial and final airfoil designs.

![Rotating NACA 0012 Final History](http://su2.stanford.edu/github_wiki/rotating_final_history.png)
Figure (8): Function evaluation history.