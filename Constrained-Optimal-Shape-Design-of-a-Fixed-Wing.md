![Opt. ONERA Orig](http://su2.stanford.edu/github_wiki/onera_opt_history.png)

## Goals

Upon completing this tutorial, the user will be familiar with performing an optimal shape design of a 3D geometry. The initial geometry chosen for the tutorial is a ONERA M6 fixed wing at transonic speed in inviscid fluid. The following SU2 tools will be showcased in this tutorial:
- **SU2_CFD** - performs the direct and the adjoint flow simulations.
- **SU2_DOT** - projects the adjoint surface sensitivities into the design space to obtain the gradient.
- **SU2_DEF** - deforms the geometry and mesh with changes in the design variables during the shape optimization process.
- **shape_optimization.py** - automates the entire shape design process by executing the SU2 tools and optimizer.

## Resources

The resources for this tutorial can be found in the TestCases/optimization_euler/steady_oneram6/ directory. You will need the configuration file (inv_ONERAM6_adv.cfg) and the mesh file (mesh_ONERAM6_inv_FFD.su2). Note that the mesh file already contains information about the definition of the Free Form Deformation (FFD) used for the definition of 3D design variables, but we will discuss how this is created below.

## Tutorial

The following tutorial will walk you through the steps required when performing 3D shape design using SU2, including the FFD tools. It is assumed that you have already obtained and compiled SU2_CFD, SU2_DOT, and SU2_DEF. The design loop is driven by the shape_optimization.py script, and thus Python along with the NumPy and SciPy Python modules are required for this tutorial. If you have yet to complete these requirements, please see the Download and Installation pages.

### Problem Setup

The goal of this wing design problem is to minimize the coefficient of drag by changing the shape while imposing lift and wing section thickness constraints. As design variables, we will use a free-form deformation approach. In this approach, a lattice of control points making up a bounding box are placed around the geometry, and the movement of these control points smoothly deforms the surface shape of the geometry inside. We begin with a 3D fixed-wing geometry (initially the ONERA M6) at transonic speed in air (inviscid). The flow conditions are the same as for the previous [[Inviscid ONERA M6]] tutorial.

![Opt. ONERA Grid](http://su2.stanford.edu/github_wiki/onera_grid.png)
Figure (1): View of the initial surface computational mesh.

### Mesh Description

The mesh consists of a far-field boundary divided in three surfaces (XNORMAL_FACES, ZNORMAL_FACES, YNORMAL_FACES), an Euler wall (flow tangency) divided into three surfaces (UPPER_SIDE, LOWER_SIDE, TIP), and a symmetry plane (SYMMETRY_FACE). The baseline mesh is the same as for the previous [[Inviscid ONERA M6]] tutorial. The surface mesh can be seen in Figure (1).

### Setting up a Free-Form Deformation Box
 
The mesh file that is provided for this test case already contains the FFD information. However, if you are interested in repeating this process for your own design cases, it is necessary to calculate the position of the control points and the parametric coordinates. The description below describes how to set up FFD boxes for deformation.

![Opt. ONERA FFD](http://su2.stanford.edu/github_wiki/onera_ffd.png)
Figure (2): View of the initial FFD box around the ONERA M6 wing, including the control points (spheres).

The design variables are defined using the FFD methodology. We will customize a set a options that specifically target FFD box creation:
 
```
% -------------------- FREE-FORM DEFORMATION PARAMETERS -----------------------%
%
% Tolerance of the Free-Form Deformation point inversion
FFD_TOLERANCE= 1E-10
%
% Maximum number of iterations in the Free-Form Deformation point inversion
FFD_ITERATIONS= 500
%
% FFD box definition: 3D case (FFD_BoxTag, X1, Y1, Z1, X2, Y2, Z2, X3, Y3, Z3, X4, Y4, Z4,
%                              X5, Y5, Z5, X6, Y6, Z6, X7, Y7, Z7, X8, Y8, Z8)
%                     2D case (FFD_BoxTag, X1, Y1, 0.0, X2, Y2, 0.0, X3, Y3, 0.0, X4, Y4, 0.0,
%                              0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0)
FFD_DEFINITION= (WING, -0.0403, 0, -0.04836, 0.8463,0, -0.04836, 1.209, 1.2896, -0.04836, 0.6851, 1.2896, -0.04836, -0.0403, 0, 0.04836, 0.8463, 0, 0.04836, 1.209, 1.2896, 0.04836, 0.6851, 1.2896, 0.04836)
%
% FFD box degree: 3D case (x_degree, y_degree, z_degree)
%                 2D case (x_degree, y_degree, 0)
FFD_DEGREE= (10, 8, 1)
%
% Surface continuity at the intersection with the FFD (1ST_DERIVATIVE, 2ND_DERIVATIVE)
FFD_CONTINUITY= 2ND_DERIVATIVE
```

As the current implementation requires each FFD box to be a quadrilaterally-faced hexahedron (6 faces, 12 edges, 8 vertices), we can simply specify the the 8 corner points of the box and the polynomial degree we would like to represent along each coordinate direction (x,y,z) in order to create the complete lattice of control points. It is convenient to think of the FFD box as a small structured mesh block with (i,j,k) indices for the control points, and note that the number of control points in each direction is the specified polynomial degree plus one. In the example above, we are creating a box with control point dimensions 11, 9, and 2 in the x-, y-, and z-directions, respectively, for a total of 198 available control points. A view of the box with the control points numbered is in Fig. 3. Note that the numbering here is 1-based, but within SU2, the control points have 0-based indexing. This is critical for specifying the design variables in the config file.

![Opt. ONERA FFD](http://su2.stanford.edu/github_wiki/onera_ffd_points.png)
Figure (3): View of the control point identifying indices, which increase in value along the positive coordinate directions. Note that the numbering here is 1-based just for visualization, but within SU2, the control points have 0-based indexing.

The tag for the FFD box can be specified as a string name. Here, we choose "WING," as we are placing the FFD box around the wing. The provided mesh file is ready for optimization, but in the case that a user is specifying their own FFD box for a problem, the SU2_DEF module should be called after defining the options above (the levels, tag, degrees, and corner points) with the DV_KIND option set to FFD_SETTING in order to compute and write the FFD_CONTROL_POINTS and FFD_SURFACE_POINTS information to the grid file. If the FFD points are already defined in the .su2 mesh file, then the FFD_SETTING is set to FFD_CONTROL_POINT. Note that this mapping for the FFD variables only needs to be computed and stored once in the mesh file before performing design. We will describe this below.

The FFD_TOLERANCE and FFD_ITERATIONS options are used internally in the iterative point inversion algorithm during the creation of the box. If you find that your particular case stalls or throws errors during the creation of the box, these parameters can be adjusted to achieve convergence of the algorithm.


Now that the FFD boxes have been set up, follow these steps at a terminal command line after defining the tags, degrees, and corner points for your FFD box (we'll use the ONERA M6 as an example):
 1. Move to the directory containing the config file (inv_ONERAM6_adv.cfg) and the mesh file (mesh_ONERAM6_inv_FFD.su2). Make sure that the SU2 tools were compiled, installed, and that their install location was added to your path.
 2. Check that DV_KIND= FFD_SETTING in the configuration file. 
 3. Execute SU2_DEF by entering "SU2_DEF inv_ONERAM6_adv.cfg" at the command line.
 4. After completing the FFD mapping process, a mesh file named "mesh_out.su2" is now in the directory. Rename that file to "mesh_ONERAM6_inv_FFD.su2". Note that this new mesh file contains all the details of the FFD method.

With this preprocessing, the position of the control points and the parametric coordinates have been calculated. A representative FFD box and the control points can be seen in Figure (2). 

### Configuration File Options

Several of the key configuration file options for this simulation are highlighted here. We will use a similar flow problem as that feature in the previous ONERA M6 tutorials and so only the design variable parameters are shown here:
```
% ----------------------- DESIGN VARIABLE PARAMETERS --------------------------%
%
% Kind of deformation (FFD_SETTING, FFD_CONTROL_POINT_2D, FFD_CAMBER_2D, FFD_THICKNESS_2D,
%                      HICKS_HENNE, COSINE_BUMP, PARABOLIC,
%                      NACA_4DIGITS, DISPLACEMENT, ROTATION, FFD_CONTROL_POINT, 
%                      FFD_DIHEDRAL_ANGLE, FFD_TWIST_ANGLE, FFD_ROTATION,
%                      FFD_CAMBER, FFD_THICKNESS, FFD_CONTROL_SURFACE, SURFACE_FILE, AIRFOIL)
DV_KIND= FFD_SETTING
%FFD_CONTROL_POINT
%
% Marker of the surface in which we are going apply the shape deformation
DV_MARKER= ( UPPER_SIDE, LOWER_SIDE, TIP )
%
% Parameters of the shape deformation 
% 	- FFD_CONTROL_POINT ( FFD_BoxTag, i_Ind, j_Ind, k_Ind, x_Disp, y_Disp, z_Disp )
% 	- FFD_DIHEDRAL_ANGLE ( FFD_BoxTag, x_Orig, y_Orig, z_Orig, x_End, y_End, z_End )
% 	- FFD_TWIST_ANGLE ( FFD_BoxTag, x_Orig, y_Orig, z_Orig, x_End, y_End, z_End )
% 	- FFD_ROTATION ( FFD_BoxTag, x_Orig, y_Orig, z_Orig, x_End, y_End, z_End )
% 	- FFD_CAMBER ( FFD_BoxTag, i_Ind, j_Ind )
% 	- FFD_THICKNESS ( FFD_BoxTag, i_Ind, j_Ind )
% 	- FFD_VOLUME ( FFD_BoxTag, i_Ind, j_Ind )
DV_PARAM= ( WING, 1, 0, 0, 0.0, 0.0, 1.0 )
%
% New value of the shape deformation
DV_VALUE= 0.0

% ------------------------ GRID DEFORMATION PARAMETERS ------------------------%
% Visualize the deformation (NO, YES)
VISUALIZE_DEFORMATION= NO

% --------------------- OPTIMAL SHAPE DESIGN DEFINITION -----------------------%
% Available flow based objective functions or constraint functions
%    DRAG, LIFT, SIDEFORCE, EFFICIENCY,
%    FORCE_X, FORCE_Y, FORCE_Z,
%    MOMENT_X, MOMENT_Y, MOMENT_Z,
%    THRUST, TORQUE, FIGURE_OF_MERIT,
%    EQUIVALENT_AREA, NEARFIELD_PRESSURE,
%    FREE_SURFACE
%
% Available geometrical based objective functions or constraint functions
%    MAX_THICKNESS, 1/4_THICKNESS, 1/2_THICKNESS, 3/4_THICKNESS, AREA, AOA, CHORD, 
%    MAX_THICKNESS_SEC1, MAX_THICKNESS_SEC2, MAX_THICKNESS_SEC3, MAX_THICKNESS_SEC4, MAX_THICKNESS_SEC5, 
%    1/4_THICKNESS_SEC1, 1/4_THICKNESS_SEC2, 1/4_THICKNESS_SEC3, 1/4_THICKNESS_SEC4, 1/4_THICKNESS_SEC5, 
%    1/2_THICKNESS_SEC1, 1/2_THICKNESS_SEC2, 1/2_THICKNESS_SEC3, 1/2_THICKNESS_SEC4, 1/2_THICKNESS_SEC5, 
%    3/4_THICKNESS_SEC1, 3/4_THICKNESS_SEC2, 3/4_THICKNESS_SEC3, 3/4_THICKNESS_SEC4, 3/4_THICKNESS_SEC5, 
%    AREA_SEC1, AREA_SEC2, AREA_SEC3, AREA_SEC4, AREA_SEC5, 
%    AOA_SEC1, AOA_SEC2, AOA_SEC3, AOA_SEC4, AOA_SEC5, 
%    CHORD_SEC1, CHORD_SEC2, CHORD_SEC3, CHORD_SEC4, CHORD_SEC5
%
% Available design variables
%    HICKS_HENNE 	(  1, Scale | Mark. List | Lower(0)/Upper(1) side, x_Loc )
%    COSINE_BUMP	(  2, Scale | Mark. List | Lower(0)/Upper(1) side, x_Loc, x_Size )
%    SPHERICAL		(  3, Scale | Mark. List | ControlPoint_Index, Theta_Disp, R_Disp )
%    NACA_4DIGITS	(  4, Scale | Mark. List |  1st digit, 2nd digit, 3rd and 4th digit )
%    DISPLACEMENT	(  5, Scale | Mark. List | x_Disp, y_Disp, z_Disp )
%    ROTATION		(  6, Scale | Mark. List | x_Axis, y_Axis, z_Axis, x_Turn, y_Turn, z_Turn )
%    FFD_CONTROL_POINT	(  7, Scale | Mark. List | FFD_BoxTag, i_Ind, j_Ind, k_Ind, x_Mov, y_Mov, z_Mov )
%    FFD_DIHEDRAL_ANGLE	(  8, Scale | Mark. List | FFD_BoxTag, x_Orig, y_Orig, z_Orig, x_End, y_End, z_End )
%    FFD_TWIST_ANGLE 	(  9, Scale | Mark. List | FFD_BoxTag, x_Orig, y_Orig, z_Orig, x_End, y_End, z_End )
%    FFD_ROTATION 	( 10, Scale | Mark. List | FFD_BoxTag, x_Orig, y_Orig, z_Orig, x_End, y_End, z_End )
%    FFD_CAMBER 	( 11, Scale | Mark. List | FFD_BoxTag, i_Ind, j_Ind )
%    FFD_THICKNESS 	( 12, Scale | Mark. List | FFD_BoxTag, i_Ind, j_Ind )
%    FFD_VOLUME 	( 13, Scale | Mark. List | FFD_BoxTag, i_Ind, j_Ind )
%    FOURIER 		( 14, Scale | Mark. List | Lower(0)/Upper(1) side, index, cos(0)/sin(1) )
%
% Optimization objective function with scaling factor
% ex= Objective * Scale
OPT_OBJECTIVE= DRAG * 0.1
%
% Optimization constraint functions with scaling factors, separated by semicolons
% ex= (Objective = Value ) * Scale, use '>','<','='
OPT_CONSTRAINT= (LIFT > 0.2864) * 0.1; (MAX_THICKNESS_SEC1 > 0.0570) * 0.1; (MAX_THICKNESS_SEC2 > 0.0513) * 0.1; (MAX_THICKNESS_SEC3 > 0.0457) * 0.1; (MAX_THICKNESS_SEC4 > 0.0399) * 0.1; (MAX_THICKNESS_SEC5 > 0.0343) * 0.1
%
% Optimization design variables, separated by semicolons
DEFINITION_DV= ( 7, 1.0 | UPPER_SIDE, LOWER_SIDE, TIP | WING, 0, 1, 0, 0.0, 0.0, 1.0 ); ( 7, 1.0 | UPPER_SIDE, 
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

![Opt. ONERA Pressure](http://su2.stanford.edu/github_wiki/onera_pressure_original.png)
Figure (4): Pressure contours showing the typical "lambda" shock on the upper surface of the initial geometry.

![Opt. ONERA Pressure](http://su2.stanford.edu/github_wiki/onera_pressure_final.png)
Figure (5): Pressure contours on the surface of the final wing design (reduced shocks).

![Opt. ONERA Pressure](http://su2.stanford.edu/github_wiki/onera_ffd_final.png)
Figure (6): View of the initial (black) and final (blue) FFD control point positions.

![Opt. ONERA History](http://su2.stanford.edu/github_wiki/onera_opt_history.png)
Figure (7): Optimization history. The drag is reduced and the lift constraint is easily met.