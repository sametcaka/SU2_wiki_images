![Optimization Diagram](http://su2.stanford.edu/github_wiki/optimization_diagram.png)

## Goals

For this tutorial, we return to the classic NACA 0012 test case that was the subject of the [[Quick Start]] and perform aerodynamic design. Upon completing this tutorial, the user will be familiar with performing an optimal shape design of a 2D geometry. The initial geometry chosen for the tutorial is the NACA 0012 airfoil in transonic, inviscid flow. This tutorial is mean to be an introduction for using the components of SU2 for shape design in the context of a simple, unconstrained optimization problem. Consequently, the following SU2 tools will be showcased in this tutorial:
- **SU2_CFD** - performs the direct and the adjoint flow simulations
- **SU2_DOT** - projects the adjoint surface sensitivities into the design space to obtain the gradient
- **SU2_DEF** - deforms the geometry and mesh with changes in the design variables during the shape optimization process
- **shape_optimization.py** - automates the entire shape design process by executing the SU2 tools and optimizer

We will walk through the shape design process and highlight several options related to the continuous adjoint in SU2 and the configuration options for shape design.

## Resources

The resources for this tutorial can be found in the TestCases/optimization_euler/steady_naca0012/ directory. You will need the configuration file (inv_NACA0012_basic.cfg) and the mesh file (mesh_NACA0012_inv.su2). Restart files for the flow and adjoint problems are also available in this directory, which can be used as an initial state for reducing the cost of the design process.

## Tutorial

The following tutorial will walk you through the steps required when performing shape design for the transonic airfoil using SU2. It is assumed that you have already obtained and compiled SU2_CFD, SU2_DOT, and SU2_DEF. The design loop is driven by the shape_optimization.py script, and thus Python along with the NumPy and SciPy Python modules are required for this tutorial. If you have yet to complete these requirements, please see the [[Download]] and [[Installation]] pages. It may also be helpful to review the [[Quick Start]] tutorial to refamiliarize yourself with this problem.

### Background

This example uses a 2D airfoil geometry (initially the NACA 0012) in transonic inviscid flow. See the [[Quick Start]] for more information on the baseline geometry. 

The general process for performing gradient-based shape optimization with SU2 is given in the flow chart at the top of the page. We start with a baseline geometry and grid as input to our design cycle, along with a chosen objective function (J) and set of design variables (x). For this tutorial, we will use the NACA 0012 and the unstructured mesh from the [[Quick Start]] as our inputs with drag as our chosen objective and a set of Hicks-Henne bump functions to parameterize the shape. From there, everything needed for automatic shape design is provided for you in the SU2 framework! By launching the shape_optimization.py script (described below), a gradient-based optimizer will orchestrate the design cycle consisting of the flow solver, adjoint solver, and geometry/mesh deformation tools available in SU2. This iterative design loop will proceed until a minimum is found or until reaching a maximum number of optimizer iterations. Many useful output files will be available to you at the conclusion.

### Problem Setup

The flow conditions of this numerical experiment are such that transonic shocks appear on the upper and lower surfaces, which causes drag. The goal of the design process is to minimize the coefficient of drag (Cd) by changing the shape of the airfoil. In other words, we would like to eliminate the shocks along the airfoil surface. This problem will solve the Euler and adjoint Euler (drag objective) equations on the NACA0012 airfoil at an angle of attack of 1.25 degrees using air with the following freestream conditions:

- Pressure = 101325 Nm-2
- Temperature = 273.15 K
- Mach number = 0.8

While more advanced design problems can be selected, such as those containing flow and/or geoemtric constraints, we will consider a simple unconstrained drag minimization problem here.

### Mesh Description

The mesh from the Quick Start Tutorial is used again here. It consists of a far-field boundary and an Euler wall along the airfoil surface. The mesh can be seen in Figure (2).

![NACA 0012 Mesh](http://su2.stanford.edu/github_wiki/rotating_mesh.png)
Figure (2): Far-field and zoom view of the initial computational mesh.

### Configuration File Options

Several of the key configuration file options for this simulation are highlighted here. SU2 features a number of ways to compute flows on dynamic grids, including moving wall boundary conditions, calculations in a rotating frame, or unsteady flows on dynamic meshes. Specifying a simulation in a rotating frame (non-inertial) can be accomplished with the following options:
```
% ---------------- ADJOINT-FLOW NUMERICAL METHOD DEFINITION -------------------%
% Adjoint problem boundary condition (DRAG, LIFT, SIDEFORCE, MOMENT_X,
%                                     MOMENT_Y, MOMENT_Z, EFFICIENCY,
%                                     EQUIVALENT_AREA, NEARFIELD_PRESSURE,
%                                     FORCE_X, FORCE_Y, FORCE_Z, THRUST,
%                                     TORQUE, FREE_SURFACE)
OBJECTIVE_FUNCTION= DRAG
%
% Convective numerical method (JST, LAX-FRIEDRICH, ROE-1ST_ORDER,
%                              ROE-2ND_ORDER)
CONV_NUM_METHOD_ADJFLOW= JST
%
% Slope limiter (VENKATAKRISHNAN, SHARP_EDGES)
SLOPE_LIMITER_ADJFLOW= VENKATAKRISHNAN
%
% 1st, 2nd, and 4th order artificial dissipation coefficients
AD_COEFF_ADJFLOW= ( 0.15, 0.0, 0.02 )
%
% Time discretization (RUNGE-KUTTA_EXPLICIT, EULER_IMPLICIT)
TIME_DISCRE_ADJFLOW= EULER_IMPLICIT
%
% Reduction factor of the CFL coefficient in the adjoint problem
CFL_REDUCTION_ADJFLOW= 0.8
%
% Limit value for the adjoint variable
LIMIT_ADJFLOW= 1E6
```
In SU2, the governing equations (Euler, Navier-Stokes, and RANS) can be solved within a rotating reference frame which offers an efficient, steady solution method for flows around rotating bodies in axisymmetric flow. A simulation can be executed in a rotating frame by choosing the ROTATING_FRAME option for GRID_MOVEMENT_KIND. Two additional pieces of data must be supplied: the location of the rotation center (x,y,z) in the coordinate system of the computational mesh, and the angular velocity (rotation rate around the x-axis, rotation rate around the y-axis, rotation rate around the z-axis). 

For the rotating airfoil problem, the airfoil has a chord of 1 meter and the origin of the coordinate system (0,0,0) is at the leading edge of the airfoil. We set the rotation center to be 32 chord lengths below the center of the airfoil. The angular velocity is in the z-direction (out of the page) and has the units of radians per second. Lastly, since the Mach number is 0.0 for this problem, we will use the value given in MACH_MOTION to compute the non-dimensional force coefficients.

Optimal shape design specification:
```
% --------------------- OPTIMAL SHAPE DESIGN DEFINITION -----------------------%
%
% Optimization objective function with scaling factor
% ex= Objective * Scale
OPT_OBJECTIVE= DRAG * 0.001
%
% Optimization constraint functions with scaling factors, separated by semicolons
% ex= (Objective = Value ) * Scale, use '>','<','='
OPT_CONSTRAINT= NONE
%
% Maximum number of optimizer iterations
OPT_ITERATIONS= 100
%
% Requested accuracyOPT_ACCURACY= 1E-6
%
% Lower and upper bound for each design variable
BOUND_DV= 0.1
%
% Optimization design variables, separated by semicolons
DEFINITION_DV= ( 1, 1.0 | airfoil | 0, 0.05 ); ( 1, 1.0 | airfoil | 0, 0.10 ); ( 1, 1.0 | airfoil | 0, 0.15 ); ( 1, 1.0 | airfoil | 0, 0.20 ); ( 1, 1.0 | airfoil | 0, 0.25 ); ( 1, 1.0 | airfoil | 0, 0.30 ); ( 1, 1.0 | airfoil | 0, 0.35 ); ( 1, 1.0 | airfoil | 0, 0.40 ); ( 1, 1.0 | airfoil | 0, 0.45 ); ( 1, 1.0 | airfoil | 0, 0.50 ); ( 1, 1.0 | airfoil | 0, 0.55 ); ( 1, 1.0 | airfoil | 0, 0.60 ); ( 1, 1.0 | airfoil | 0, 0.65 ); ( 1, 1.0 | airfoil | 0, 0.70 ); ( 1, 1.0 | airfoil | 0, 0.75 ); ( 1, 1.0 | airfoil | 0, 0.80 ); ( 1, 1.0 | airfoil | 0, 0.85 ); ( 1, 1.0 | airfoil | 0, 0.90 ); ( 1, 1.0 | airfoil | 0, 0.95 ); ( 1, 1.0 | airfoil | 1, 0.05 ); ( 1, 1.0 | airfoil | 1, 0.10 ); ( 1, 1.0 | airfoil | 1, 0.15 ); ( 1, 1.0 | airfoil | 1, 0.20 ); ( 1, 1.0 | airfoil | 1, 0.25 ); ( 1, 1.0 | airfoil | 1, 0.30 ); ( 1, 1.0 | airfoil | 1, 0.35 ); ( 1, 1.0 | airfoil | 1, 0.40 ); ( 1, 1.0 | airfoil | 1, 0.45 ); ( 1, 1.0 | airfoil | 1, 0.50 ); ( 1, 1.0 | airfoil | 1, 0.55 ); ( 1, 1.0 | airfoil | 1, 0.60 ); ( 1, 1.0 | airfoil | 1, 0.65 ); ( 1, 1.0 | airfoil | 1, 0.70 ); ( 1, 1.0 | airfoil | 1, 0.75 ); ( 1, 1.0 | airfoil | 1, 0.80 ); ( 1, 1.0 | airfoil | 1, 0.85 ); ( 1, 1.0 | airfoil | 1, 0.90 ); ( 1, 1.0 | airfoil | 1, 0.95 )
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