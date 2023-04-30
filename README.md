Download Link: https://assignmentchef.com/product/solved-hpc-project-7-high-performance-computing-with-python
<br>



In this project, we will continue learning about parallel programming with MPI which was introduced in project 6. Particularly, we will use the ghost cells exchange between neighboring processes towards building an MPI parallel solver for the Fisher’s equation that we discussed and parallelized with OpenMP in project 4. Furthermore, we will extend this project with several tasks related to High-Performance Computing with Python. The Python programming language is very popular in scientific computing because of the benefits it offers for fast code development. The performance of pure Python programs is often suboptimal, but there are ways to make them faster and more efficient. In this project you will learn various ways to optimize and parallelize Python programs, particularly in the context of scientific and high-performance computing. Additionally, we will introduce it through some examples. We will also look at alternative algorithms, e.g, using self-scheduling techniques for parallelizing the computation of, e.g., the Mandelbrot set.

You may do this project in groups of two or three students. In fact, we prefer that you do so.

<h1>1.      Parallel Space Solution of a nonlinear PDE using MPI [in total 40 points]</h1>

This subproject discusses domain decomposition for an MPI parallel solver of a nonlinear PDE that we discussed in detail in project 4. In project 4 we added OpenMP to the parallel space solution of a nonlinear PDE miniapplication, so that we could use all cores on one compute node on the ICS cluster . The goal of this exercise is now to use MPI, so that we are able to use multiple compute nodes. In both the serial and the OpenMP versions, there was only one process that had all the data. In the MPI version, we now have multiple processes (ranks). The computational grid on which we compute will be divided into equal subgrids, and each subgrid will be assigned to a process. Each process will only have access to its own subgrid and cannot access the data that belongs to other processes. In order to compute the

<table width="680">

 <tbody>

  <tr>

   <td width="456">new value at a given grid-point, the values at all its neighboring grid-points will be needed. In case this grid-point is on the boundary of a process’ subgrid, we would need to get the value from the neighboring process that has these data. So, before each iteration, all the MPI processes first exchange the “ghost cells” and save these values from their respective neighbors to the boundary buffers (bndN, bndS, bndE, bndW). After the exchange, each process has all the data it needs to compute the next iteration.</td>

   <td width="225">Figure 1: Ghost cell exchange: copy the north (south) row to buffN (buffS), send buffN (buffS) to the neighbor, receive to bndS (bndN).</td>

  </tr>

 </tbody>

</table>

You can find an initial incomplete version of the MPI code in the directory pde-miniapp. The source code is almost equivalent to the OpenMP version that you have already implemented in project 4. In this subproject you should modify the same files as before. There are some comments below that will guide you during the implementation process.

<h2>Hints</h2>

Note that at the beginning the initial version of the code is incomplete and it is your task to add the missing MPI functionality. As a first step you need to initialize MPI and fill in the missing parts. When you finalize the ghost cells

1

exchange in (operators.cpp), you can check the functionality by looking at the resulting final image. You might observe that only parts of the image are correct, or parts of the image are flipped in n-s or e-w directions. Think about why this happens. It will help you to find what is wrong in your code.

For the testing of your final MPI code, you can use the same parameters as for the OpenMP version, for example

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="3c494f594e7c555f4f525358596464">[email protected]</a>]$ ./main 128 100 0.01

Don’t forget that you have to use mpirun to launch the MPI application.

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="5c292f392e1c353f2f30333b35326c6d">[email protected]</a>]$ salloc -n 4

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="6712140215270e0414090803023f3f">[email protected]</a>]$ module load openmpi

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="ef9a9c8a9daf868c9c81808b8ab7b7">[email protected]</a>]$ cd pde-miniapp/

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="3441475146745d57475a5b50516c6c">[email protected]</a>]$ make

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="7104021403311812021f1e15142929">[email protected]</a>]$ mpirun -np 4 ./main 128 100 0.01

Below we will list several steps that you need to take in order to finalize this project.

<h2>1.1.     Initialize and finalize MPI [5 Points]</h2>

In the file main.cpp we need to add the initial MPI code so that the MPI environment is initialized with MPIInit. During MPIInit, all of MPI’s global and internal variables are constructed. For example, a communicator is formed around all of the processes that were spawned, and unique ranks are assigned to each process. You also need to add MPI code so that each process has its own rank as well. In particular we need to add the following:

<ul>

 <li>Initialize MPI.</li>

 <li>Get current rank and number of ranks.</li>

 <li>Finalize MPI.</li>

</ul>

<h2>1.2.     Create a Cartesian topology [10 Points]</h2>

You need to generate a 2D domain decomposition (MPI communicator) of a given grid depending on the number of ranks similar to project 6 (”Ghost cells exchange between neighboring processes”). In the file data.cpp:

<ul>

 <li>Create the dimensions of the decomposition depending on the number of ranks (using ”MPI Dims create”).</li>

 <li>Create a <em>non-periodic </em>Cartesian topology for the grid of domains (using ”MPI Cart create”).</li>

 <li>Identify the coordinates of the current rank in the domain decomposition (using ”MPI Cart coords”).</li>

 <li>Identify the neighbors of the current rank: east, west, north and south directions (using ”MPI Cart shift”).</li>

</ul>

<h2>1.3.     Extend the linear algebra functions [5 Points]</h2>

Implement the dot product and the norm computation where a vector is distributed over all ranks. Think about why this is only necessary for these two functions and not the others?

In the file linalg.cpp:

<ul>

 <li>Add a collective operation to compute the dot product (using ”MPI Allreduce”).</li>

 <li>Add a collective operation to compute the norm (using ”MPI Allreduce”).</li>

</ul>

<h2>1.4.     Exchange ghost cells [10 Points]</h2>

Use point-to-point communication to exchange ghost cells amongst neighbours.

In the file operators.cpp:

<ul>

 <li>Add point-to-point communication for all neighbours in all directions.</li>

 <li>Use <em>non-blocking </em>communication (using ”MPI Irecv” and ”MPI Isend”).</li>

 <li>Try to overlap computation and communication.</li>

</ul>

Be careful to send first row/last row/column. Before you send the data, copy it to the send buffers (buffN, buffS, buffE, buffW), and receive it in the ghost cells (bndN, nbdS, bndE, bndW). Because you copy the data to a 1D array first, you don’t need any custom data types for send and receive. Look at Figure 1 for an illustration of the ghost cells exchange.

<h2>1.5.     Scaling experiments [10 Points]</h2>

How does it scale at different resolutions? You might try both weak- and strong-scaling for this project. Plot, e.g, the time to solution using 1-32 MPI ranks on different compute nodes for the grid sizes:

<ul>

 <li>128×128</li>

 <li>256×256</li>

 <li>512×512</li>

 <li>1024×1024</li>

</ul>

Analyze and interpret your results. <em>Hint: </em>you can use the span[ptile=n] option to access different nodes.

<h1>2.      Python for High-Performance Computing (HPC) [in total 60 points]</h1>

Python is increasingly used in high-performance computing projects. It can be used either as a high-level interface to existing HPC applications and libraries, as an embedded interpreter, or directly. In this project, we will show how Python can be used on parallel architectures to parallelize the nonlinear PDE solver project using NumPy to explore the productivity gains made possible by Python for HPC.

In recent years the Python programming language has become more and more popular in scientific computing for various reasons. Users not only implement prototypes for numerical experiments on small scales, but also develop parallel production codes, thereby partly replacing compiled languages such as C or C++. However, when following this approach it is crucial to pay special attention to performance. This tutorial course teaches “High-Performance Computing with Python” approaches to use Python efficiently and reasonably in an HPC environment. We recommend to have an initial look at this course which was held from July 02–04, 2019 at CSCS:

<ul>

 <li><a href="https://www.youtube.com/watch?v=JYX4TQ_fCqY&amp;list=PL1tk5lGm7zvQ-EzsiTZ6Xv1SxZs74epzg">https://www.youtube.com/watch?v=JYX4TQ_fCqY&amp;list=PL1tk5lGm7zvQ-EzsiTZ6Xv1SxZs74epzg</a></li>

</ul>

We will use the package MPI for Python (mpi4py) for using MPI within Python. Begin by watching the lesson of the CSCS course on MPI:

<ul>

 <li><a href="https://www.youtube.com/watch?v=XeyspDaKjMM">https://www.youtube.com/watch?v=XeyspDaKjMM</a></li>

</ul>

Although the lessons use mostly IPython/Jupyter notebooks, we will use plain Python scripts. The documentation for mpi4py can be found here

<ul>

 <li><a href="https://mpi4py.readthedocs.io/en/stable/index.html">https://mpi4py.readthedocs.io/en/stable/index.html</a></li>

</ul>

Remember to use the help function within a Python interpreter:

&gt;&gt;&gt; from mpi4py import MPI

&gt;&gt;&gt; help(MPI)

ICS Cluster environment setup instructions: In order to run mpi4py on the ICS Cluster you will need to install a custom environment using anaconda that has all of the libraries that you will need to run the code. To set up this enviroment you will need the text file project7condaenv.txt that is located in the hpc-python directory. To set up the environment navigate to the directory containing the project7condaenv.txt file and run the following commands:

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="89fcfaecfbc9e0eafae5e6eee0e7b9b8">[email protected]</a>]$ module load anaconda3

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="bacfc9dfc8fad3d9c9d6d5ddd3d48a8b">[email protected]</a>]$ conda create –name project7_env –file project7_conda_env.txt

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="790c0a1c0b39101a0a15161e10174948">[email protected]</a>]$ conda init bash

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="bacfc9dfc8fad3d9c9d6d5ddd3d48a8b">[email protected]</a>]$ exit

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="0d78636c60684d7d687f7e62636c61">[email protected]</a>_computer]$ ssh ics_cluster

(base)[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="cfbabcaabd8fa6acbca3a0a8a6a1fffe">[email protected]</a>]$ conda activate project7_env

Please note that if you have the OpenMPI module loaded in your current session on the cluster you will have problems running the code. Please start a new session for running Python MPI code and do not load the OpenMPI module. For Python, we refer to the documentation

<ul>

 <li><a href="https://docs.python.org/3/">https://docs.python.org/3/</a></li>

</ul>

In order to get started, we begin with a simple Python MPI program hello.py:

<table width="675">

 <tbody>

  <tr>

   <td width="675">from mpi4py import MPI# get comm, size &amp; rank comm = MPI.COMM_WORLD size = comm.Get_size() rank = comm.Get_rank()# hello print(f”Hello world from rank {rank} out of {size} processes”)</td>

  </tr>

 </tbody>

</table>

In order to run the script, first load the following module on the ICS cluster :

[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="2550564057654c4656494a424c4b1514">[email protected]</a>]$ salloc -n 4

(base)[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="1c696f796e5c757f6f727378794444">[email protected]</a>]$ conda activate project7_env

Run the script in parallel with

(project7_env)[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="e297918790a28b81918c8d8687baba">[email protected]</a>]$ mpiexec -n 4 python hello.py

The output of the script (see the lsf.o* file) should look like (up to the order):

Hello world from rank 0 out of 4 processes

Hello world from rank 1 out of 4 processes

Hello world from rank 2 out of 4 processes

Hello world from rank 3 out of 4 processes

Now that everything is set up and working, we can get started!

<h2>2.1.     Sum of ranks: MPI collectives [5 Points]</h2>

With MPI for Python’s collective communication methods, write a script that computes the sum of all ranks:

<ul>

 <li>using the pickle-based communication of generic Python objects, i.e. the <em>all-lowercase </em>methods;</li>

 <li>using the fast, near C-speed, direct array data communication of buffer-provider objects, i.e. the method names starting with an <em>uppercase </em></li>

</ul>

<h2>2.2.     Domain decomposition: Create a Cartesian topology [5 Points]</h2>

Write a script that computes a 2D <em>periodic </em>process distribution depending on the number of processes, and creates a Cartesian topology:

<ul>

 <li>use the method MPI.Computedims, a convenience function similar to MPI’s MPIDIMSCREATE;</li>

 <li>create a Cartesian topology using MPI for Python;</li>

 <li>determine the neighbouring processes,</li>

 <li>output the topology: rank, Cartesian coordinates in decomposition, East/West/North/South neighbours.</li>

</ul>

<h2>2.3.     Exchange rank with neighbours [5 Points]</h2>

Next, we are going to exchange data within the periodic Cartesian topology from the previous task.

For each process, exchange its rank with the four east/west/north/south neighbours. Verify that you obtain the expected result.

<h2>2.4.     Parallel space solution of a nonlinear PDE using Python and MPI</h2>

In this task, we are going to complete a Python implementation of the previously introduced nonlinear PDE using C/C++ and MPI example. You can find the pdeminiapppy code on the usual git or iCorsi repositories. The code largely follows the structure of the previous C/C++ implementation. You can run the skeleton code with

(project7_env)[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="d0a5a3b5a290b9b3a3bebfb4b58888">[email protected]</a>]$ mpiexec -n 4 python main.py 128 100 0.01 verbose

In this example, the simulation is run with four MPI processes, a grid of size 128<sup>2</sup>, 100 time steps until a final time of

0.01. You can draw the solution with the draw.py script:

(project7_env)[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="b5c0c6d0c7f5dcd6c6dbdad1d0eded">[email protected]</a>]$ python draw.py

You can adapt this script for debugging purposes.

<h3>2.4.1.     Change linear algebra functions [5 Points]</h3>

In the linalg module (linalg.py):

<ul>

 <li>Complete the dot product computation, method hpcdot.</li>

 <li>Complete the norm computation, method hpcnorm2.</li>

</ul>

<h3>2.4.2.     Exchange ghost cells [5 Points]</h3>

The solution data are contained within the Field class in the data module (data.py). The class holds onedimensional (1D) Numpy arrays bdryN/E/S/W, supposed to contain ghost points from neighbouring processes, and 1D Numpy buffer arrays buffN/E/S/W, supposed to store/buffer data to be sent to neighbouring processes. Communication is started by calling the exchangestartall method, and waiting until the communication is completed is handled by the exchangewaitall method. Complete the following tasks:

<ul>

 <li>Implement the exchangestartall method using Isend and Irecv methods to initiate send and receive of operations.</li>

</ul>

<em>Remark</em>: make sure that you understand the difference between mpi4py’s <em>all-lowercase </em>and first letter <em>uppercase </em>methods.

<ul>

 <li>Implement the exchangewaitall routine.</li>

 <li>Verify that you obtain results that are consistent with your C/C++ implementation.</li>

</ul>

<h3>2.4.3.     Scaling experiments [5 Points]</h3>

Repeat the scaling experiments from 1.5 using 1-32 MPI ranks. Analyze and interpret your results, also in comparison to the behavior of the C++ implementation. <em>Remark: </em>The Python version is expected to be significantly slower.

<h2>2.5.     A self-scheduling example: Parallel Mandelbrot [30 Points]</h2>

In this task, you are asked to implement one of the most common parallel algorithm prototypes: the <em>self-scheduling</em>, or <em>manager-worker</em>, or <em>master-slave</em><a href="#_ftn1" name="_ftnref1"><sup>[1]</sup></a>, algorithm. The basic idea is that one process, known as the manager, is responsible for delegating work to other processes, known as the workers. This is particularly useful in problems where the amount of work per worker is difficult to estimate and the workers don’t have to communicate with each other in order to do their work.

As a particular example, we consider the Mandelbrot set again. Note that this is only meant as an illustration of this fundamental type of parallel algorithm, and not really as the best way to parallelize the computation of the Mandelbrot set. The manager decomposes the Mandelbrot set into a number of (rectangular) patches. Computing the Mandelbrot (sub)set on a particular patch will be called a task. The manager then delegates these tasks to the workers. Once a worker is done computing a particular task, he sends the patch back to the manager. Therewith, the worker signals to the manager that he is available to work on a new task. The manager then sends the worker another task to work on. This process is repeated until no more tasks remain, i.e. all the patches of the Mandelbrot set have been computed. Finally, the manager combines all the patches from the workers and outputs the image.

The skeleton codes for this subproject are located in the folder hpc-python/ManagerWorker available through the lecture git/ iCorsi repository. Begin by familiarizing yourself with the mandelbrottask.py module. It contains two classes. First, the class mandelbrot, which decomposes the Mandelbrot set computation in a series of subsets or patches, produces a list of tasks, and combines the tasks’ patches together. Second, the mandelbrotpatch class, which holds a subset or patch of the Mandelbrot set and contains a method dowork that performs the actual computation. This part is already fully implemented for your convenience. However, feel free to try out different implementations, e.g. domain decompositions, etc.

Complete the following:

<ul>

 <li>Implement the manager-worker algorithm in the skeleton code managerworker.py.</li>

 <li>Add a scaling study using 2,4,8, and 16 workers (or more if the ICS cluster allows) splitting the workload once into 50 and once into 100 tasks.</li>

</ul>

The program can be called as follows:

(project7_env)[<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="d9acaabcab99b0baaab7b6bdbc8181">[email protected]</a>]$ mpiexec -n 4 python manager_worker.py 4001 4001 100


