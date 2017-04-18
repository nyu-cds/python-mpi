---
title: "Final Notes"
teaching: 10
exercises: 0
questions:
- "What are the other imporatant aspects of mpi4py?"
objectives:
- "Learn about how mpi4py can handle different types of data."
keypoints:
- "mpi4py provides different methods to handle buffer-like and generic objects.
---
## Communication of buffer-like objects 

When using the upper case version of the methods (`Send`, `Irecv`, `Gather`, etc.) the data object must support the *single-segment buffer interface*. 
This interface is a standard Python mechanism provided by some types (e.g., strings and numeric arrays), which is why we have been using NumPy 
arrays in the examples. 

## Communication of generic Python objects

It is also possible to transmit an arbitrary Python data type using the lower case version of the methods (`send`, `irecv`, `gather`, etc.) 
mpi4py will serialize the data type, send it to the remote process, then deserialize it back to the original data type (a process known as *pickling*
and *unpickling*). While this is simple, it also adds significant overhead to the MPI operation.

## Further information

There are many other MPI operations available that we have not touched on here. Please refer to the 
[mpi4py documentation](http://mpi4py.readthedocs.org/en/stable/index.html) for more information 
if you are interested.
