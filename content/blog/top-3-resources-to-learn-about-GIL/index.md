---
title: "Top 3 resources to learn about GIL"
published: true
date: "20190311"
tags: ["GIL"]
---

In this blog post I will share with you easily in one place the best sources to learn about the Python Global Interpretation Lock ([GIL](https://wiki.python.org/moin/GlobalInterpreterLock)). If you come from a language like Golang, the chances are that you aren't familiar with GIL and what limitation may come with it.

The GIL is a mechanism that makes all new pythonistas to scratch their head. It is a gatekeeper that protects access to Python objects. In practice this means that only one thread in allowed to run and access memory at a time and thus GIL makes Python effectively a single-threaded programming language.

## Why is this important?

Python has various libraries that are used to simulate multi-theaded processing. They make Python seem like a multi-threaded language even though it is not so. The programmer has to know which library to use in what kind of situation. If you try to run parallel tasks with an incorrect library, the processing time can increase instead of decreasing.

### Available libraries

Two popular libraries are `threading` and `multiprocessing`. To put it short, the `threading` library should be used when the program needs to carry various simultaneous IO-bound tasks. An IO-bound task is a task that the processor has to wait to be ready. For example, downloading a file from a server is an IO-bound task. The program requests the file and then waits for the task to finish. The `multiprocessing` library should be used for CPU-bound tasks. CPU-bound taks are tasks that require a lot of calculating power from the CPU. In this case this processor does not need to wait, for example, for an input from outside, but it is itself very busy solving a problem. These two aren't the only libraries out there, but the scope of this post is not to go through all of them. The newest library is `asyncio`, that it can be considered in place of the `threading` library in many cases.

### Why are the libraries different

The `threading` uses threads and the `multiprocessing` uses processes. The difference is that threads are created in the same memory space within the Python interpreter, but a new process is created by another Python interpreter. This means that the process has its own memory space and it is not limited by the GIL. This makes sharing data between processes a bit more complicated and the total memory usage goes up. Creating new processes is also slower than creating new threads. This adds a bit overhead to the processing times if your program needs many new processes.

## Top 3 resources to learn about GIL

Finally we've reached the point of this post. I will list here the best resources that I have found on the internet to learn about GIL. The YouTube videos are quite long, but especially the video by Larry Hastings is very easy to follow.

1. [Understanding the Python GIL by David Beazley](https://www.youtube.com/watch?v=Obt-vMVdM8s)

This is a fundamental video about the GIL and how it works by David Beazley. He did an extensive research on the GIL and it performance. The video has also some cool visualizations included. I suggest you to check this first. You should also check his [website](http://www.dabeaz.com) for other interesting talks.

2. [PyCon 2015 - Python's Infamous GIL by Larry Hastings](https://www.youtube.com/watch?v=4zeHStBowEk)

This video is more like an overview about the reasons behind the GIL. Why was it added in the first place? What are the future plans? Larry talks a lot about reference counting which is interesting if you want to learn about carbage collection. This is a good video to get a good grasp about the terminology surrounding the topics.

3. [Answer in StackExchange](https://softwareengineering.stackexchange.com/a/186909)

This good post in StackExchange explains the good aspects of the GIL in an easy-to-understand way. If you don't have the time for the videos, at least take a look at this post.

Actually you can find a lot of profound articles in a few minutes of googling, but I feel like these three sources are the ones that sunk in my case. Especially if you find the time to watch both of the videos you should be good to go.
