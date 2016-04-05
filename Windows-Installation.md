As of release 4.1.1, SU2 supports Windows platforms from Windows 7 through Windows 10 in 32-bit and 64-bit architectures (see details and limitations below) in serial (multi-threaded but no MPI) mode only. Please note that the executables have been built to support any version of Windows but have only been tested in Windows 10, x64 platform. The 32-bit versions are available for legacy support but are limited to smaller problems due to the 2 GB memory limit in 32-bit systems; a 64-bit architecture is recommended. Windows binaries are packaged as an installer (.exe). If you encounter a problem installing or running in Windows please contact the support team for assistance. 

**The x86 (32-bit) version** 

This version is built with CGNS 3.3.0 but no Tecplot binary support since Tecplot no longer supports the x86 architecture in their Tecio library. Tecplot text file output is always available. 

**The x64 (64-bit) version**

This version is built with CGNS 3.3.0 and Tecplot binary support. 

## Installation 

1. **Download and run the appropriate installer package**. Choose the installer that corresponds to your architecture (x64 or x86).  You must have administrator privileges to install. Run the installer and follow the installation wizard which will guide you through the options. The setup wizard will guide you through the setup options available. 

2. **Add SU2 environment variables**. This is done through the Environment Variables control panel.  You can access these by typing "environ" in the search/run box of the start menu.  Start a New System variable.  Assign the Variable Name "SU2_RUN", and assign the Variable Value to be the path to your SU2 Executables (the folder that contains SU2_CFD.exe for example).  If you used the default values from the installer, this could be "C:\Program Files\Stanford ADL\SU2\bin".  Note: x64 systems might use "Program Files (x86)"  

## Running SU2 in Windows

Running SU2 in Windows is identical to running in Linux of Mac OS environments and is run from the command line (wither cmd.exe or the freely-available Console2 for Windows).