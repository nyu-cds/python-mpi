---
title: "Collective Operations"
teaching: 30
exercises: 20
questions:
- "What is the difference between point-to-point and collective communication?" 
objectives:
- "Understand the basics of collective communication."
- "Learn about the different types of collective communication."
- "See how collective communication can be used in practice."
keypoints:
- "Collective communication allows data to be sent or received from multiple processes simultaneously."
- "Collective operations fall into three broad categories: synchonization, communication, and computation."
---
There are many situations in parallel programming when groups of processes need to exchange messages. Rather than explicitly sending and receiving 
such messages as we have been doing, the real power of MPI comes from group operations known as *collectives*.

Collective communications allow the sending of data between multiple processes of a group simultaneously. Collective functions come in blocking and 
non-blocking versions.

The more commonly used collective communication operations are the following:

- Synchronization
  - Processes wait until all members of the group have reached the synchronization point
- Global communication functions
  - Broadcast data from one member to all members of a group
  - Gather data from all members to one member of a group
  - Scatter data from one member to all members of a group
- Collective computation (reductions)
  - One member of the group collects data from the other members and performs an operation (min, max, add, multiply, etc.) on that data.

Collective communication routines must involve all processes within the scope of a communicator.

All processes are by default, members in the communicator MPI.COMM_WORLD, however additional communicators can be defined by the programmer 
(beyond the scope of this course).

> Unexpected behavior, including program failure, can occur if even one task in the communicator doesn't participate. 
> It is the programmer's responsibility to ensure that all processes within a communicator participate in any collective operations.
{: .callout}

## Example collective operations

> ## Comm.Barrier()
> Synchronization operation. Creates a barrier synchronization in a group. Each task, when reaching the Barrier() call, blocks until all 
> tasks in the group reach a Barrier() call. Then all tasks are free to proceed.
{: .callout}

> ## Comm.Bcast(buf, root=0)
> Data movement operation. Broadcasts (sends) a message from the process with rank "root" to all other processes in the group.
{: .callout}

> ## Comm.Scatter(sendbuf, recvbuf, root=0)
> Data movement operation. Distributes distinct messages from a single source task to each task in the group.
{: .callout}

> ## Comm.Gather(sendbuf, recvbuf, root=0)
> Data movement operation. Gathers distinct messages from each task in the group to a single destination task. This routine is the reverse 
> operation of Scatter().
{: .callout}

> ## Comm.Alltoall(sendbuf, recvbuf)
> All-to-all Scatter/Gather, send data from all to all processes in a group.
{: .callout}

> ## Comm.Reduce(sendbuf, recvbuf, op=MPI.SUM, root=0)
> Reduces values on all processes to a single value by applying the operation op. Operations include:
> - MPI.MAX - Returns the maximum element.
> - MPI.MIN - Returns the minimum element.
> - MPI.SUM - Sums the elements.
> - MPI.PROD - Multiplies all elements.
> - MPI.LAND - Performs a logical and across the elements.
> - MPI.LOR - Performs a logical or across the elements.
> - MPI.BAND - Performs a bitwise and across the bits of the elements.
> - MPI.BOR - Performs a bitwise or across the bits of the elements.
> - MPI.MAXLOC - Returns the maximum value and the rank of the process that owns it.
> - MPI.MINLOC - Returns the minimum value and the rank of the process that owns it.
{: .callout}

## Parallel collective version of Mid-point rule

The example below shows how the mid-point rule can be computed using collective operations.

We choose to broadcast the number of increments per partition n to each process, although this is not strictly necessary. Once the processes 
have received n they are able to compute their partition. The processes then send the values back to the root process using `Reduce` which 
automatically computes the sum of all the values and places the result in integral_sum.

~~~
import numpy
from math import acos, cos
from mpi4py import MPI
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

def integral(a_i, h, n):
    integ = 0.0
    for j in range(n):
        a_ij = a_i + (j + 0.5) * h
        integ += cos(a_ij) * h
    return integ

pi = 3.14159265359
a = 0.0
b = pi / 2.0
dest = 0
n = numpy.zeros(1)
my_int = numpy.zeros(1)
integral_sum = numpy.zeros(1)

# Initialize value of n only if this is rank 0
if rank == 0:
    n = numpy.full(1, 500) # default value
    
# Broadcast n to all processes
print "Process ", rank, " before n = ", n[0]
comm.Bcast(n, root=0)
print "Process ", rank, " after n = ", n[0]

# Compute partition
h = (b - a) / (n * size) # calculate h *after* we receive n
a_i = a + rank * h * n
my_int[0] = integral(a_i, h ,n)

# Send partition back to root process, computing sum across all partitions
print "Process ", rank, " has the partial integral ", my_int[0]
comm.Reduce(my_int, integral_sum, MPI.SUM, dest)

# Only print the result in process 0
if rank == 0:
    print 'The Integral Sum =', integral_sum[0]
~~~
{: .python}

This program is run with the command:

~~~
mpiexec -n 4 python midpoint_coll.py
~~~
{: .bash}

The following is an example of the output generated:

~~~
Process  0  before n =  500.0
Process  0  after n =  500.0
Process  0  has the partial integral  0.382683442201
The Integral Sum = 1.0000000257
Process  1  before n =  0.0
Process  1  after n =  500.0
Process  1  has the partial integral  0.32442335716
Process  2  before n =  0.0
Process  2  after n =  500.0
Process  2  has the partial integral  0.216772756896
Process  3  before n =  0.0
Process  3  after n =  500.0
Process  3  has the partial integral  0.0761204694451
~~~
{: .output}

> ## Challenge
> Modify the above code to broadcast both the number of increments `n` and the increment width `h` to each process. 
> Hint: `h` will need to be a NumPy array.
{: .challenge}
