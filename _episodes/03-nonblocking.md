---
title: "Non-blocking Communication"
teaching: 0
exercises: 0
questions:
objectives:
keypoints:
---
So far we have seen how to send and receive messages using blocking communication. In this case, the sender or receiver is not able to perform any 
other actions until the corresponding message has been sent or received (to be accurate, it is actually until the buffer is safe to use.)

Blocking communication has a number of disadvantages. Potential computational time is simply wasted while waiting for the call to complete. And as we 
have seen, blocking communication can also lead to deadlock.

An alternate approach is to allow the program to continue execution while the messages is being sent or received. This is known as *non-blocking* 
communcation.

In MPI, non-blocking communication is achieved using the `Isend` and `Irecv` methods. The `Isend` and `Irecv` methods initiate a send and receive 
operation respectively, and then return immediately.

These methods return a instance of the `Request` class, which uniquely identifys the started operation. The completion can then be managed using the 
`Test`, `Wait`, and `Cancel` methods of the `Request` class. The management of `Request` objects and associated memory buffers involved in c
ommunication requires careful coordination. Users must ensure that objects exposing their memory buffers are not accessed at the Python level 
while they are involved in nonblocking message-passing operations.

The following example performs the same simple send and receive as demonstrated previously, however this time it is done with the non blocking 
versions of the send and receive methods. Create a program called `mpi5.py` with this code:

~~~
import numpy
from mpi4py import MPI
comm = MPI.COMM_WORLD
rank = comm.Get_rank()

randNum = numpy.zeros(1)

if rank == 1:
        randNum = numpy.random.random_sample(1)
        print("Process", rank, "drew the number", randNum[0])
        req = comm.Isend(randNum, dest=0)
        req.Wait()
        
if rank == 0:
        print("Process", rank, "before receiving has the number", randNum[0])
        req = comm.Irecv(randNum, source=1)
        req.Wait()
        print("Process", rank, "received the number", randNum[0])
~~~
{: .python}

Run this program using the command:

~~~
mpiexec -np 4 python mpi5.py
~~~
{: .bash}

You should see output similar to the following:

~~~     
Process 0 before receiving has the number 0.0
Process 0 received the number 0.97400925874
Process 1 drew the number 0.97400925874
~~~
{: .output}

> ## Challenge
If you commented out the `Isend` and both `Wait` calls you would see that this code will not deadlock (if you don't comment out the `Wait` calls, 
then you have effectively the same code as before and it will deadlock.) However, unless you call `Cancel`, the Python kernel will eventually 
deadlock anyway as there will be an unequal number of messages posted, so I don't recommend doing it.
{: .challenge}


The following code is a non blocking version of the send and receive program. Note there is no need to wait after process 1 sends the message, nor after process 0 sends the reply. However it is necessary for process 1 to wait for the reply so that it knows the message has been fully received before trying to print it out. Similarly, process 0 must wait for the full message before trying to compute randNum * 2. Run it to verify the program works.
In [37]:

%%px
import numpy
from mpi4py import MPI
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
​
randNum = numpy.zeros(1)
diffNum = numpy.random.random_sample(1)
​
if rank == 1:
        randNum = numpy.random.random_sample(1)
        print "Process", rank, "drew the number", randNum[0]
        comm.Isend(randNum, dest=0)
        req = comm.Irecv(randNum, source=0)
        req.Wait()
        print "Process", rank, "received the number", randNum[0]
​
if rank == 0:
        print "Process", rank, "before receiving has the number", randNum[0]
        req = comm.Irecv(randNum, source=1)
        req.Wait()
        print "Process", rank, "received the number", randNum[0]
        randNum *= 2
        comm.Isend(randNum, dest=1)
[stdout:0] 
Process 0 before receiving has the number 0.0
Process 0 received the number 0.570093200547
[stdout:1] 
Process 1 drew the number 0.570093200547
Process 1 received the number 0.623148825134
Modify this program so that process 1 overlaps a computation with sending the message and receiving the reply. The computation should be dividing diffNum by 3.14 and printing the result.
In [18]:

%%px
### BEGIN SOLUTION
import numpy
from mpi4py import MPI
comm = MPI.COMM_WORLD
rank = comm.Get_rank()
​
randNum = numpy.zeros(1)
diffNum = numpy.random.random_sample(1)
​
if rank == 1:
        randNum = numpy.random.random_sample(1)
        print "Process", rank, "drew the number", randNum[0]
        comm.Isend(randNum, dest=0)
        diffNum /= 3.14
        print "diffNum=", diffNum[0]
        req = comm.Irecv(randNum, source=0)
        req.Wait()
        print "Process", rank, "received the number", randNum[0]
​
if rank == 0:
        print "Process", rank, "before receiving has the number", randNum[0]
        req = comm.Irecv(randNum, source=1)
        req.Wait()
        print "Process", rank, "received the number", randNum[0]
        randNum *= 2
        comm.Isend(randNum, dest=1)
### END SOLUTION
[stdout:0] 
Process 0 before receiving has the number 0.0
Process 0 received the number 0.311574412567
[stdout:1] 
Process 1 drew the number 0.311574412567
diffNum= 0.00213775044313
Process 1 received the number 1.93636680934
It is possible to test without waiting using Request.Test(). This method will return True when the message operation has completed. To cancel a pending communication, call Request.Cancel().