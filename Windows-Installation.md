As of release 4.1.1, SU2 supports Windows platforms from Windows 7 through Windows 10 in (x86) 32-bit and (x64) 64-bit architectures (see details and limitations below) in serial (multi-threaded but no MPI) mode only. Please note that the executables have been built to support any version of Windows but have only been tested on Windows 10, x64 platform. The x86 versions are available for legacy support but are limited to smaller problems due to the 2 GB memory limit in x86 systems; a 64-bit architecture is recommended. Windows binaries are packaged as an installer (.exe). If you encounter a problem installing or running in Windows please contact the support team for assistance. 

**x86 (32-bit) version** 

This version is built with CGNS 3.3.0 but no Tecplot binary support since Tecplot no longer supports the x86 architecture in their Tecio library. Tecplot text file output is always available. 

**x64 (64-bit) version**

This version is built with CGNS 3.3.0 and Tecplot binary support. 

## Installation 

1. **Download and run the appropriate installer package**. Choose the installer that corresponds to your architecture (x64 or x86).  You must have administrator privileges to install. Run the installer and follow the installation wizard which will guide you through the options. The setup wizard will guide you through the setup options available. 

2. **Add SU2 environment variables**. This is done through the Environment Variables control panel.  You can access these by typing "environ" in the search/run box of the start menu.  Start a New System variable.  Assign the Variable Name "SU2_RUN", and assign the Variable Value to be the path to your SU2 Executables (the folder that contains SU2_CFD.exe for example).  If you used the default values from the installer, this could be "C:\SU2".  Any paths with white space characters must use quotes, for example: C:\"Program Files"\"Stanford ADL"\SU2\. This variable will allow you to quickly navigate to the SU2 directory using "cd %SU2_RUN%", and run the executables using "%SU2_RUN%\<executable>" if SU2\ has not been added to the system path.

3. **Add SU2 to the system path**. Access the system variables by typing "environ" in the search/run box of the start menu, select "edit the system environment variables", and then select "Environment Variables" from the System Properties menu. Edit the "Path" system variable (or select "New" if no "Path" system variable exists) and add the path to the SU2 executables. Any path with white space must use quotes as described above, and when there is more than one directory listed in "Path" they must be separated by semicolons. This will allow you to use the executables from any directory without explicitly specifying the path to the executable. 

## Running SU2 in Windows

Running SU2 in Windows is almost identical to running in Linux or Mac OS environments and is run from the command line (whether cmd.exe or the freely-available Console2 for Windows). The only difference is that ".exe" must be added to the end of the executable name, i.e. "SU2_CFD.exe" instead of "SU2_CFD". 

## What's up with MPI on Windows? 

SU2 has been successfully integrated with MS MPI, version 7 and a parallel version for Windows (including the necessary MPI package) is planned for the near future. SU2 currently relies on ParMETIS, a third-party software tool, for parallel grid partitioning and this package does not currently have Windows support nor integration with MS MPI. Once properly configured for METIS partitioning (supported in Windows and already integrated into SU2) in parallel solutions, a fully-parallel version of SU2 for Windows will be released.  