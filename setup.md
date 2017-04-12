---
layout: page
title: Setup
permalink: /setup/
---
## Microsoft MPI

To use MPI with Windows, you will need to install the free download of Microsoft MPI. Go to the 
[installation page](https://www.microsoft.com/en-us/download/details.aspx?id=54607) and 
download `MSMpiSetup.exe`. Once downloaded, run the executable and follow the instructions.

Next, add the path `C:\Program Files\Microsoft MPI\bin` to the `PATH` environment variable. You can do this
by typing the command:

~~~
PATH=%PATH%;C:\Program Files\Microsoft MPI\bin
~~~
{: .bash}

If you want to set the `PATH` permanently, follow [these instructions](http://www.computerhope.com/issues/ch000549.htm).

Check that MPI is installed correctly by entering the command `mpiexec -help` and verifying that the output is as expected.
 
## MPI for Python

MPI for Python provides bindings of the Message Passing Interface (MPI) standard for the Python programming language, allowing any 
Python program to exploit multiple processors. 

This package is constructed on top of the MPI-1/2/3 specifications and provides an 
object oriented interface which resembles the MPI-2 C++ bindings. It supports point-to-point (sends, receives) and collective 
(broadcasts, scatters, gathers) communications of any picklable Python object, as well as optimized communications of Python 
object exposing the single-segment buffer interface (NumPy arrays, builtin bytes/string/array objects).

To install mpi4py, enter the command:

~~~
conda install mpi4py
~~~
{: .bash}

On Mac OS X and Linux, this will install *both* MPI for Python and Open MPI. On Windows, it will only install MPI for Python.
