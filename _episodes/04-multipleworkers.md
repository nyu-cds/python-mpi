---
title: "Multiple Workers"
teaching: 30
exercises: 20
questions:
objectives:
keypoints:
---
While the threading and multiprocessing modules are great for scripts that are running on your personal computer, what should you do if you want the 
work to be done on a different machine, or you need to scale up to more than the CPU on one machine can handle? A great use case for this is 
long-running back-end tasks for web applications. If you have some long running tasks, you don’t want to spin up a bunch of subprocesses or 
threads on the same machine that need to be running the rest of your application code. This will degrade the performance of your application for 
all of your users. What would be great is to be able to run these jobs on another machine, or many other machines.

A great Python library for this task is [RQ](http://python-rq.org/), a very simple yet powerful library. You first enqueue a function and its arguments 
using the library. This [pickles](https://docs.python.org/3.4/library/pickle.html) the function call representation, which is then appended to a 
[Redis](http://redis.io/) list. Enqueueing the job is the first step, but will not do 
anything yet. We also need at least one worker to listen on that job queue.

The first step is to install and run a Redis server on your computer, or have access to a running Redis server. After that, there are only a few small 
changes made to the existing code. We first create an instance of an RQ Queue and pass it an instance of a Redis server from the
[redis-py library](https://github.com/andymccurdy/redis-py). 
Then, instead of just calling our `download_link` method, we call `q.enqueue(download_link, download_dir, link)`. The enqueue method takes a function 
as its first argument, then any other arguments or keyword arguments are passed along to that function when the job is actually executed.

One last step we need to do is to start up some workers. RQ provides a handy script to run workers on the default queue. Just run “rqworker” in a 
terminal window and it will start a worker listening on the default queue. Please make sure your current working directory is the same as where the 
scripts reside in. If you want to listen to a different queue, you can run “rqworker queue_name” and it will listen to that named queue. The great 
thing about RQ is that as long as you can connect to Redis, you can run as many workers as you like on as many different machines as you like; 
therefore, it is very easy to scale up as your application grows. Here is the source for the RQ version:

~~~
from redis import Redis
from rq import Queue

from download import setup_download_dir, get_links, download_link

CLIENT_ID = #replace with your client id

def main():
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID)]
   q = Queue(connection=Redis(host='localhost', port=6379))
   for link in links:
       q.enqueue(download_link, download_dir, link)
~~~
{: .python}

However, RQ is not the only Python job queue solution. RQ is easy to use and covers simple use cases extremely well, but if more advanced options 
are required, other job queue solutions (such as [Celery](http://www.celeryproject.org/)) can be used.
