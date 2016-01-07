---
layout: lesson
root: ../..
title: Introduction to Open Science Grid 
---
<div class="objectives" markdown="1">

#### Objectives
*   Get to know what is Open Science Grid
*   What resources are open to academic researchers
*   Computation that is a good match for OSG
*   Computation that is NOT a good match for OSG

</div>

## Introduction to Open Science Grid (OSG)  

The Open Science Grid (OSG) is a consortium of research communities who promote science 
via sharing of computing resources. The Open Science Grid (OSG)
<ul> 
<li> enables distributed computing on more than 120 institutions, </li>
<li> supports efficient data processing and  </li>
<li> provides large scale scientific computing of 2 million core CPU hours per day.   </li>
</ul> 


The resources accessible through the OSG are contributed by the community, organized by 
the OSG, and governed by the OSG Consortium. The cores that are free in the OSG shared pool 
are made available to the users. These are 
opportunistic resources for the users and the number of freely available 
cores are varying at any given time. 



## Computation that is a good match for OSG 

High throughput workflows with simple system and data dependencies are a good 
fit for OSG. Typically these workflows can be decomposed into multiple
tasks that can be carried out independently.  Ideally, these tasks will download 
data for input, run some computation on it and then return results (which may be 
used by future tasks).

Jobs submitted into the OSG will be executed on machines at several 
remote physical clusters. These machines may differ in terms of computing 
environment from the submit node. Therefore it is important that the jobs are 
as self-contained as possible by generic binaries and data that can be either 
carried with the job, or staged on demand. Please consider the following 
guidelines:
<ul>
<li>   Software should preferably be single threaded, using less than 2 GB memory and 
    each invocation should run for 1-12 hours (optimally under 4 hours). There is 
    some support for jobs with longer run time, more memory or multi-threaded codes. 
    Please contact the support listed below for more information about these 
    capabilities.</li>
<li>   Only core utilities can be expected on the remote end. There are no standard 
    versions of software such as 'gcc', 'python', 'BLAS' or others on the grid. 
    Consider using Distributed Environment Modules to manage software dependencies, 
    or read our Developing High-Throughput Applications guide.</li>
<li>   Input and output data for each job should be < 10 GB to allow them to be 
    transferred in by the jobs, processed and returned to the submit node. Note 
    that the OSG Virtual Cluster does not have a global shared file 
    system, so jobs with such dependencies will not work.</li>
<li>   No shared filesystem. Jobs must transfer all executables, input data, and 
    output data. HTCondor can transfer the files for you, but you will have to 
    identify and list the files in your HTCondor job description file. </li>
</ul>

## Computation that is NOT a good match for OSG 

The following are examples of computations that are NOT good matches for 
OSG:
<ul>
<li>   Tightly coupled computations, for example MPI based communication, will 
    not work well on OSG due to the distributed nature of the infrastructure.</li>
<li>   Computations requiring a shared file system will not work, as there is 
    no shared filesystem between the different clusters on the OSG.</li>
<li>   Computations requiring complex software deployments or proprietary software 
    are not a good fit.  There is limited support for distributing software to 
    the compute clusters, but for complex software, or licensed software, 
    deployment can be a major task.</li>
</ul>

##How to get help using OSG

Please contact user support staff at [user-support@opensciencegrid.org](mailto:user-support@opensciencegrid.org).


<h2> Available Resources on OSG </h2> 
Commonly used software and libraries on the Open Science Grid are available in a
central repository (known as OASIS) and accessed via the *module* command. We will see how to 
search for, load, and use software packages.

We will also cover the usage of the built-in *tutorial* command. Using *tutorial*,
we load a variety of job templates that cover basic usage, specific use cases, and best practices.

<h3> Software Applications </h3>

We will be submitting jobs to the OSG through HCC's Crane login node.
Log in to Crane (HCC machine) with secure shell  

~~~
$ ssh username@crane.unl.edu
~~~


The first step in using the module command is to initialize the OSG module system.  This 
step consists of sourcing a file that switches from the built-in HCC module environment to the OSG module environment.  Source the file as follows:

~~~
$ source osg_oasis_init
~~~


Once the distributed environment modules system is initialized, you can check the 
available modules: 

~~~
$ module avail
 
 
--------------------------- /cvmfs/oasis.opensciencegrid.org/osg/modules/modulefiles/Core ----------------------------
   atlas      fftw/fftw-3.3.4-gromacs    lapack              lmod/5.6.2 (D)    python/3.4
   blast      gromacs/4.6.5              lmod/SiteHook       namd/2.9          settarg/5.6.2
   blender    jpeg                       lmod/SitePackage    python/2.7 (D)
 
  Where:
   (D):  Default Module
 
Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
~~~

In order to load a module, you need to run "module load [modulename]".  Say for
example you want to load R package, 

~~~
$ module load R 
~~~

This sets up the R package for you. Now you can do some test calculations with R. 

~~~
$ R # invoke R package

> cos(45)  # simple on-screen calculation with cosine function
[1] 0.525322

~~~

If you want to unload a module, type 

~~~
$ module unload R 
~~~

<h3> Tutorial Command </h3> 

The built-in *tutorial* command assists a user in getting started on 
OSG.  To see the list of existing tutorials, type

~~~
$ tutorial # will print a list tutorials
~~~

Say for example, you are interested in learning how to run R scripts on OSG, the 
tutorial command sets up the R tutorial for you. 

~~~
$ tutorial R  # prints the following message:

Installing R (master)...
Tutorial files installed in ./tutorial-R.
Running setup in ./tutorial-R...
~~~ 

The "tutorial R" command creates a directory "osg-R" containing the neccessary script and input files. 

~~~
mciP.R      # The example R script file
R-wrapper.sh # The job execution file 
R.submit  # The job submission file (will discuss later in the lesson HTCondor scripts)
~~~

Lets focus on "mciP.R" and the "R-wrapper" scripts. The details of "R.submit" script 
will be discussed later when we learn HTCondor scripts.  

The file "mciP.R" is a R script that calculates the value of *pi* using the Monte Carlo
method.  The R-wrapper.sh essentially loads the R module and runs the "mciP.R" 
script. 

~~~
#!/bin/bash

EXPECTED_ARGS=1

if [ $# -ne $EXPECTED_ARGS ]; then
  echo "Usage: R-wrapper.sh file.R"
  exit 1
else
  source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
  module load R
  Rscript $1
fi
~~~

Similar to the R tutorial, there are other tutorials available on OSG. The available 
tutorials serve as templates to develop your own scripts and run the 
calculations on OSG. 

<div class="keypoints" markdown="1">

#### Key Points
*   OSG resources are distributed across 120 institutions and  supports scientific computing of 2 million core CPU hours per day.   
*   Many scientific applications are installed on OSG and available for the users. 
*   To use an existing application use the module load command. 
*   The command - *tutorial* helps to access the existing tutorials.  
</div>



