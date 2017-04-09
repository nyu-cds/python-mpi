---
title: "Using Threads"
teaching: 20
exercises: 20
questions:
- "How can threads be used to improve the performance of our image downloader?"
objectives:
- "Learn about the Python `Thread` class."
- "Learn about thread safety and the GIL."
keypoints:
- "Python provides a `threading` module to easily use threads."
- "Although the GIL limits thread parallism, it is still useful to use threads in Python."
---
Threading is another well known approach to attaining concurrency. Threading is a feature usually provided by the 
operating system. Threads are typically lighter weight than processes, and they have much lower memory requirements as they share the same memory space.

In this lesson, we will use Pyton threading to increase the performance of our image downloader. To do this, we will create a pool of 8 threads, making 
a total of 9 threads including the main thread. The optimal number of threads is usually chosen based on factors such as other applications and 
services running on the same machine. We will chose 8 worker threads because many personal computers have 8 CPU cores and one worker thread per 
core seems like a good number for how many threads to run at once.

The threaded version of the program is almost the same as `simple.py` with the exception that we now have a new class, `DownloadWorker`, that inherits 
from the `Thread` class. This class provides a `run` method that should been overridden with a method that does the actual work of the thread.

> ## Thread Safety
> As mentioned earlier, each thread shares the same memory space. That is, the variables in the program are shared by all the threads and cannot be accessed
> the way you would normally access a variable. This is because the threads are executing simultaneously, and one thread may change the variable while
> another thread is reading it, or worse, two threads may try to update the variable at the same time. This is known as a *race condition*, and is one of
> the leading sources of errors in threaded programs. Instead, it is necessary to use special classes that allow multiple threads to access them
> similutaneously. These are known as *thread safe* classes.
{: .callout}

In our case, we will provide the thread with a `run` method which downloads images in an infinite loop. We will use a thread-safe data structure known 
as a `Queue` to keep track of the URLs that each thread will download. On every iteration, the thread will call `self.queue.get()` to try and fetch a
URL to download. This call suspends the thread until there is an item in the queue for the worker thread to process. Once the worker receives an item 
from the queue, it then calls the same `download_link` method that was used in the previous script to download the image to the images directory. After the download is 
finished, the worker signals the queue that that task is done. This is very important, because the `Queue` keeps track of how many tasks were enqueued. 
The call to `queue.join()` would block the main thread forever if the workers did not signal that they completed a task.

~~~
from time import time
from queue import Queue
from threading import Thread

from download import setup_download_dir, get_links, download_link

CLIENT_ID = 'replace with your client ID'

class DownloadWorker(Thread):
   def __init__(self, queue):
       Thread.__init__(self)
       self.queue = queue

   def run(self):
       while True:
           # Get the work from the queue and expand the tuple
           directory, link = self.queue.get()
           download_link(directory, link)
           self.queue.task_done()

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID)]
   # Create a queue to communicate with the worker threads
   queue = Queue()
   # Create 8 worker threads
   for x in range(8):
       worker = DownloadWorker(queue)
       # Setting daemon to True will let the main thread exit even though the workers are blocking
       worker.daemon = True
       worker.start()
   # Put the tasks into the queue as a tuple
   for link in links:
       print('Queueing {}'.format(link))
       queue.put((download_dir, link))
   # Causes the main thread to wait for the queue to finish processing all the tasks
   queue.join()
   print('Took {}'.format(time() - ts))

if __name__ == '__main__':
   main()
~~~
{: .python}

> ## Challenge
>
> Create a `threads.py` file using the program provided above. Replace the string `'replace with your client ID'` in 
> `threads.py` with the client ID you obtained from Imgur. Run `threads.py` and verify that you obtain a number of images in the `images`
> directory. Compare how long it takes to download these images with the `simple.py` and `procs.py` versions.
{: .challenge}

> ## Global Interpreter Lock (GIL)
> In multithreading, a *lock* is a mechanism for preventing multiple threads from accessing the same shared variable simultaneously. This is to avoid
> a race condition, as mentioned previously. CPython uses a global interpreter lock, otherwise known as GIL, to prevent the execution of multiple threads
> at once in the Python interpreter. This prevents two threads from accidentially corrupting internal Python data structures. Although it sounds like this
> would prevent multiple threads from being useful in Python, in practice there are still many operations that can still be done in parallel. One in
> particular is performing input/output operations, such as accessing a web site.
>
> While the de facto reference Python implementation - CPython - has a GIL, this is not true of all Python implementations. For example, Jython, a 
> Java implementation of Python, and IronPython, a Python implementation using the .NET framework both do not have a GIL. You can find a list of 
> working Python implementations [here](https://wiki.python.org/moin/PythonImplementations#Working_Implementations).
{: .callout}

Running this script on the same machine used earlier results in a download time of 3.1 seconds! That is 3 times faster than the `simple.py` example. 

While this is much faster, it is worth mentioning that only one thread was executing at a time throughout this process due to the GIL. 
Therefore, this code is concurrent but not parallel. The reason it is still faster is because this is an input/output bound task. The processor is hardly 
breaking a sweat while downloading these images, and the majority of the time is spent waiting for the network. This is why threading can provide 
a large speed increase. The processor can switch between the threads whenever one of them is ready to do some work. 

If the program was performing a task that was CPU bound, using the threading module in Python or any other interpreted language with a GIL could
actually result in reduced performance. For CPU bound tasks and truly parallel execution in Python, the `multiprocessing` module is a better option.


