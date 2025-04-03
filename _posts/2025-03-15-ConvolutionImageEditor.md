---
layout: post
title:  Parallel Convolution for an Image Processing System with Go 
date: 2025-03-15
---

![golang-gopher](./images/flying_gopher.jpg)

Is no secret that Golang is great for building concurrent programs. In fact, one of the main goals of the language was precisely to provide a quick intuitive way of implementing parallel algorithms.

## An image editor 

Image processing is a computational expensive task that requires processing thousands of pixels. For this specific example, I wrote a program that uses two dimentional convolution to apply filters to any input image. The program allows the user to provide an input image and specify which of the four available filters (gray scale, blur, sharpen, edges) to apply to the image. The user can choose to apply as many filters as they want. The output is a new image with all the filters applied in the order specified by the user. A sequential program, would need to go pixel by pixel at a time and apply the change, while a parallel implementation can divide the work in multiple threads that work in the same image at the same time, dramatically decreasing processing times. 

![image input](./images/parallel_input.png)

Source: Original image from UChicago Parallel Programming (MPCS 52060 1) course. 

## Different approaches 
The following diagrams 

![Barrier implementation of a parallel image editor](./images/BSP_LinkedList.png)

For the second parallel implementation, each thread is responsible for processing a complete image. To distribute the workload evenly, the program uses a lock-free queue implemented as an array of work pools (see The Art of Multiprocessor Programming section 16.5). The queue allows to initially balance the workload evenly between goroutines, and then allowing work-stealing such that those goroutines that finish all the tasks in their original queues first can help others complete their tasks.

As shown in the diagram, the parfiles implementation does not require a lot of communication or dependency between threads. The only contention for resources happens when a thread is stealing from another goroutines that is almost over. In such cases, two (or more) goroutines could try to acquire the same final task, forcing a handful of expensive CAS operations. Expensive operations such as initially loading an image can cause a bottleneck in the editing process. The throughput of each goroutine is also higher since each one deals with completeimages.


![Parallel implementation of an image editor](./images/Parfiles.png)
