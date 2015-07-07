### Overview
Cygwin is a linux emulator, which allows the user to run linux programs on a Windows machine. Downloads and more information on Cygwin is available at http://www.cygwin.com/. To compile and run SU2 in Cygwin on a Windows machine, you will need to configure Cygwin with the appropriate settings, and then continue from within Cygwin as though you were using a linux machine. 

### Notes on installing Cygwin for first-time or beginner Cygwin users
* It is often recommended to install in  a directory rather than the default C:\ location.
* The Cygwin shell will only be able to access folders that are within the cygwin\ directory; you will need to install and run SU2 inside the cygwin directory.

### Cygwin packages
At the package selection step, search for the following terms and select the associated packages to install. This list is a work-in-progress, and further packages may be required or desired.

#### Basic
* g++, gcc compiler
* python: install the packages under the python sub-heading
* cpp: all debug, pre-processor, regular expression packages. 

#### Parallel
* openmpi
* metis