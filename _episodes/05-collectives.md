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
- "Collective operations fall into the broad categories: synchonization, communication, computation, and I/O."
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
- Collective Input/Output
  - Each member of the group reads or writes a section of a file.

## Collective communication and synchonization points

One of the things to remember about collective communication is that it implies a *synchronization point* among processes. This means that all 
processes must reach a point in their code before they can all begin executing again. 

> Collective communication routines must involve all processes within the scope of a communicator.
> Unexpected behavior, including program failure, can occur if even one task in the communicator doesn't participate. 
> It is the programmer's responsibility to ensure that all processes within a communicator participate in any collective operations.
{: .callout}

As it turns out, MPI has a special function that is dedicated to synchronizing processes: `Comm.Barrier()`.

The name of the function is quite descriptive - the function forms a barrier, and no processes in the communicator can pass the barrier 
until all of them call the function. Here's an illustration. Imagine the horizontal axis represents execution of the program and the 
circles represent different processes:

![barrier]({{ page.root }}/fig/04-barrier.png "barrier")

Process zero first calls `Barrier` at the first time snapshot (T1). While process zero is hung up at the barrier, process one and three 
eventually make it (T2). When process two finally makes it to the barrier (T3), all of the processes then begin execution again (T4).

## Broadcasting

A broadcast is one of the standard collective communication techniques. During a broadcast, one process sends the same data to all processes 
in a communicator. One of the main uses of broadcasting is to send out user input to a parallel program, or send out configuration parameters 
to all processes.

The communication pattern of a broadcast looks like this:

![broadcast pattern]({{ page.root }}/fig/04-broadcast.png "broadcast pattern")

In this example, process zero is the root process, and it has the initial copy of data. All of the other processes receive the copy of data.

Although the root process and receiver processes do different jobs, they all call the same `Comm.Bcast` function. When the root process 
(in our example, it was process zero) calls `Comm.Bcast`, the data variable will be sent to all other processes. When all of the receiver
processes call `Comm.Bcast`, the data variable will be filled in with the data from the root process.

## Scatter

Scatter is a collective operation that is very similar to broadcast. Scatter involves a designated root process sending data to all processes 
in a communicator. The primary difference between broadcast and scatter is small but important. Broadcast sends the same piece of data to all 
processes while scatter sends chunks of an array to different processes. Check out the illustration below for further clarification.

![broadcast vs scatter]({{ page.root }}/fig/04-broadcastvsscatter.png "broadcast vs scatter")

In the illustration, the broadcast takes a single data element at the root process (the red box) and copies it to all other processes. However
the scatter takes an array of elements and distributes the elements in the order of process rank. The first element (in red) goes to process zero, 
the second element (in green) goes to process one, and so on. Although the root process (process zero) contains the entire array of data, 
the scatter operation will copy the appropriate element into the receiving buffer of the process. 

The `Comm.Scatter` method takes three arguments. The first is an array of data that resides on the root process. The second 
parameter is used to hold the received data. The last parameter indicates the root process that is scattering the array of data.

## Gather

Gather is the inverse of scatter. Instead of spreading elements from one process to many processes, the gather operation takes elements from 
many processes and gathers them to one single process. This routine is highly useful to many parallel algorithms, such as parallel 
sorting and searching. Below is a simple illustration of this algorithm.

![gather]({{ page.root }}/fig/04-gather.png "gather")

Similar to scatter, gather takes elements from each process and gathers them to the root process. The elements are ordered by the rank of the 
process from which they were received. 

The `Comm.Gather` method takes the same arguments as `Comm.Scatter`. Howeverm, in the gather operation, only the root process needs to have a 
valid receive buffer.

## Reduction

Reduce is a classic concept from functional programming. Data reduction involves reducing a set of numbers into a smaller set of numbers via a 
function. For example, let's say we have a list of numbers `[1, 2, 3, 4, 5]`. Reducing this list of numbers with the sum function would produce 
`sum([1, 2, 3, 4, 5]) = 15`. Similarly, the multiplication reduction would yield `multiply([1, 2, 3, 4, 5]) = 120`.

As you might have imagined, it can be very cumbersome to apply reduction functions across a set of distributed numbers. Along with that, it is 
difficult to efficiently program non-commutative reductions, i.e. reductions that must occur in a set order. Luckily, there is a handy function 
called `Comm.Reduce` that will handle almost all of the common reductions that a programmer needs to do in a parallel application.

The `Comm.Reduce` method takes an array of input elements and returns an array of output elements to the root process. The output elements contain 
the reduced result. MPI contains a set of common reduction operations that can be used, although custom reduction operations can also be defined.

The following diagram shows the communication pattern for a reducation:

![reduction]({{ page.root }}/fig/04-reduce.png "reduction")

In the above, each process contains one integer. The reduction operation is called with a root process of 0 and using MPI_SUM as the reduction 
operation. The four numbers are summed to the result and stored on the root process.

It is also useful to see what happens when processes contain multiple elements. The illustration below shows reduction of multiple numbers per process.

![reduction with multiple elements]({{ page.root }}/fig/04-multireduce.png "reduction with multiple elements")

The processes from the above illustration each have two elements. The resulting summation happens on a per-element basis. In other words, instead 
of summing all of the elements from all the arrays into one element, the i<sup>th</sup> element from each array are summed into the i<sup>th</sup> element in result array 
of process 0.

## Example collective operations

> ## `Comm.Barrier()`
> Synchronization operation. Creates a barrier synchronization in a group. Each task, when reaching the Barrier() call, blocks until all 
> tasks in the group reach a `Barrier()` call. Then all tasks are free to proceed.
{: .callout}

> ## `Comm.Bcast(buf, root=0)`
> Data movement operation. Broadcasts (sends) a message from the process with rank "root" to all other processes in the group.
{: .callout}

> ## `Comm.Scatter(sendbuf, recvbuf, root=0)`
> Data movement operation. Distributes distinct messages from a single source task to each task in the group.
{: .callout}

> ## `Comm.Gather(sendbuf, recvbuf, root=0)`
> Data movement operation. Gathers distinct messages from each task in the group to a single destination task. This routine is the reverse 
> operation of `Scatter()`.
{: .callout}

> ## `Comm.Alltoall(sendbuf, recvbuf)`
> All-to-all Scatter/Gather, send data from all to all processes in a group.
{: .callout}

> ## `Comm.Reduce(sendbuf, recvbuf, op=MPI.SUM, root=0)`
> Reduces values on all processes to a single value by applying the operation op. Operations include:
> - `MPI.MAX` - Returns the maximum element.
> - `MPI.MIN` - Returns the minimum element.
> - `MPI.SUM` - Sums the elements.
> - `MPI.PROD` - Multiplies all elements.
> - `MPI.LAND` - Performs a logical and across the elements.
> - `MPI.LOR` - Performs a logical or across the elements.
> - `MPI.BAND` - Performs a bitwise and across the bits of the elements.
> - `MPI.BOR` - Performs a bitwise or across the bits of the elements.
> - `MPI.MAXLOC` - Returns the maximum value and the rank of the process that owns it.
> - `MPI.MINLOC` - Returns the minimum value and the rank of the process that owns it.
{: .callout}

> ## `File.Open(comm, filename, amode, info)`
> Opens the file on all processes in the communicator group.
{: .callout}

> ## `File.Write_all(buffer)`
> Collective write operation.
{: .callout}

## Parallel collective version of Mid-point rule

The example below shows how the mid-point rule can be computed using collective operations.

We choose to broadcast the number of increments per partition `n to each process, although this is not strictly necessary. Once the processes 
have received `n` they are able to compute their partition. The processes then send the values back to the root process using `Reduce` which 
automatically computes the sum of all the values and places the result in `integral_sum`.

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
print("Process ", rank, " before n = ", n[0])
comm.Bcast(n, root=0)
print("Process ", rank, " after n = ", n[0])

# Compute partition
h = (b - a) / (n * size) # calculate h *after* we receive n
a_i = a + rank * h * n
my_int[0] = integral(a_i, h ,n)

# Send partition back to root process, computing sum across all partitions
print("Process ", rank, " has the partial integral ", my_int[0])
comm.Reduce(my_int, integral_sum, MPI.SUM, dest)

# Only print the result in process 0
if rank == 0:
    print('The Integral Sum =', integral_sum[0])
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
