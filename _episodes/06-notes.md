---
title: "Final Notes"
teaching: 0
exercises: 0
questions:
objectives:
keypoints:
---
mpi4py supports two kinds of Python data objects. 

When using the upper case version of the methods (`Send`, `Irecv`, `Gather`, etc.) the data object must support the *single-segment buffer interface*. 
This interface is a standard Python mechanism provided by some types (e.g., strings and numeric arrays), which is why we have been using NumPy 
arrays in the examples. 

It is also possible to transmit an arbitrary Python data type using the lower case version of the methods (`send`, `irecv`, `gather`, etc.) 
mpi4py will serialize the data type, send it to the remote process, then deserialize it back to the original data type (a process known as *pickling*
and *unpickling*). While this is simple, it also adds significant overhead to the MPI operation.

There are many other MPI operations available that we have not touched on here. Please refer to the 
[mpi4py documentation](http://mpi4py.readthedocs.org/en/stable/index.html) for more information 
if you are interested.
