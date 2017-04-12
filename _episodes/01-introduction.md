---
title: "Introduction"
teaching: 30
exercises: 0
questions:
- "What is parallel computing?"
- "What are the different parallel computing approaches?"
- "What are the pro's and con's of the different approaches?"
objectives:
- "Explain the basics of parallel computing."
keypoints:
- "Parallel computing is used to solve problems that are too large for traditional approaches."
- "There are a variety of techniques for parallel computing."
- "Different problems may require different techniques."
- "Amdahl's law defines the limits of parallel computing performance improvements."
---
## Definitions

Program
: An executable file residing on disk

Process
: One or more executing instances of a program. Processes have separate address spaces.

Task
: In MPI, a process is sometimes called a *task*, but these are used interchangably. We will always use "process" rather than "task".

Thread (or lightweight process)
: Ane or more threads of control within a process. Threads share the same address space.

![definitions]({{ page.root }}/fig/01-prog-proc-thread.png "definitions")

## What is parallel computing?

Traditionally, software has been written for serial computation:
- A problem is broken into a discrete series of instructions
- Instructions are executed sequentially one after another
- Executed on a single processor
- Only one instruction may execute at any moment in time

![serial computer]({{ page.root }}/fig/01-serial.png "serial computer")

In the simplest sense, parallel computing is the simultaneous use of multiple compute resources to solve a computational problem:
- A problem is broken into discrete parts that can be solved concurrently
- Each part is further broken down to a series of instructions
- Instructions from each part execute simultaneously on different processors
- An overall control/coordination mechanism is employed

![parallel computer]({{ page.root }}/fig/01-parallel.png "parallel computer")

## Why do we need parallel programming?

Need to solve larger problems
Require more memory
Computation takes much longer
Huge amounts of data may be required

Parallel programming provides
- More CPU resources
- More memory resources
- The ability to solve problems that were not possible with serial program
- The ability to solve problems more quickly

## Two basic approaches

### Shared Memory Computer

- Used by most laptops/PCs
- Multiple cores (CPUs)
- Share a global memory space
- Cores can efficiently exchange/share data

![shared memory computer]({{ page.root }}/fig/01-shared-mem.png "shared memory computer")

### Distributed Memory (ex. Compute cluster)

- Collection of nodes which have multiple cores
- Each node uses its own local memory
- Work together to solve a problem
- Communicate between nodes and cores via messages
- Nodes are networked together

![distributed memory computer]({{ page.root }}/fig/01-distributed-mem.png "distributed memory computer")

## Parallel programming models

Directive-based parallel programming language
- OpenMP (most widely used)
- High Performance Fortran (HPF) is another example
- Directives tell processor how to distribute data and work across the processors
- Directives appear as comments in the serial code
- Implemented on shared memory architectures

Message Passing
- MPI (most widely used)
- Pass messages to send/receive data between processes
- Each process has its own local variables
- Can be used on either shared or distributed memory architectures

## Pros and cons

Pros of OpenMP
- Easier to program and debug than MPI
- Directives can be added incrementally - gradual parallelization
- Can still run the program as a serial code
- Serial code statements usually don't need modification
- Code is easier to understand and maybe more easily maintained

Cons of OpenMP
- Can only be run in shared memory computers
- Requires a compiler that supports OpenMP
- Mostly used for loop parallelization

Pros of MPI
- Runs on either shared or distributed memory architectures
- Can be used on a wider range of problems than OpenMP
- Each process has its own local variables
- Distributed memory computers are less expensive than large shared memory computers

Cons of MPI
- Requires more programming changes to go from serial to parallel version
- Can be harder to debug
- Performance is limited by the communcation network between the nodes

## Parallel programming issues

Goal is to reduce execution time
- Computation time
- Idle time - waiting for data from other processors
- Communication time - time the processors take to send and receive messages

Load Balancing
- Divide the work equally among the available processors

Minimizing Communication
- Reduce the number of messages passed
- Reduce amount of data passed in messages

Where possible - overlap communication and computation

Many problems scale well to only a limited number of processors

## Amdahl's law

![Amdahl's law]({{ page.root }}/fig/01-speedup.png "Amdahl's law")

where
- *speedup* is the theoretical speedup of the execution of the whole program
- *n* is the number of parallel threads/processes
- *P* is the fraction of the algorithm that can be made parallel

Basically this is saying that the amount of speedup a program will see by using *n* cores is based on how much of the program is serial 
(can only be run on a single CPU core) and how much of it is parallel (can be split up among multiple CPU cores).

![amdahl's law]({{ page.root }}/fig/01-amdahls-law.png "amdahl's law")

## Programming approaches

Two main approaches:

SPMD - Single Program, Multiple Data Streams
- Each processor is executing the same program on different data
- A parallel execution model that assumes multiple cooperating processes executing a program
- The most common style of parallel programming and the one used by MPI

MPMD - Multiple Programs, Multiple Data Streams
- Multiple processors executing at least two independent programs
- Manager/worker strategies fit into this category
- Web browser and web server is another example

## What is MPI?

MPI stands for Message Passing Interface
- Library of functions (C/C++) or subroutines (Fortran)

History
- Early message passing Argonne's P4 and Oak Ridge PVM in 1980s
- MPI-1 completed in 1994 (1.1 - 1995, 1.2 - ?, 1.3 - 2008)
- MPI-2 completed in 1998 (2.1 - 2008, 2.2 - 2009)
- MPI-3 completed in 2012 (3.1 - 2015)
- MPI-3 features gradually added to MPI implementations

## Version differences

Examples of Different Implementations
- MPICH - developed by Argonne Nationa Labs (freeware)
- MPI/LAM - developed by Indiana, OSC, Notre Dame (freeware)
- MPI/Pro - commerical product
- Apple's X Grid
- OpenMPI - MPI-2 compliant, thread safe

Similiarities in Various Implementations
- Source code compatibility (except parallel I/O)
- Programs should compile and run as is
- Support for heterogeneous parallel architectures

Difference in Various Implementations
- Commands for compiling and linking
- How to launch an MPI program
- Parallel I/O (from MPI-2)
- Debugging