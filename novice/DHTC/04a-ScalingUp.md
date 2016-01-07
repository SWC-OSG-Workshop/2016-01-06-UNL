---
layout: lesson
root: ../..
title: Scaling up compute resources - R and MATLAB runtime examples. 
---
<div class="objectives" markdown="1">

#### Objectives
*   Learn why to scale up the computational resources.
*   Learn how to scaling up the computational resources
</div>

## Overview

Scaling up the computational resources is a big advantage for doing
certain large scale calculations on OSG. Consider the extensive
sampling for a multi-dimensional Monte Carlo integration or molecular
dynamics simulation with several initial conditions. These type of
calculations require submitting lot of jobs.

In one of the previous example, we submitted the job to a single worker
machine. About a million CPU hours per day are available to OSG users
on an opportunistic basis.  Learning how to scale up and control large
number of jobs is important to realizing the full potential of distributed high
throughput computing on the OSG.

In this section, we will see how to scale up the calculations with
simple examples. The examples are based on R and MATLAB Runtime. Once we understand 
the basic HTCondor script, it is easy to scale up.

~~~
$ source osg_oasis_init
$ cd $WORK
$ tutorial ScalingUp-R
$ cd tutorial-ScalingUp-R
~~~

As we discussed in the previous section on HTCondor scripts, we need to
prepare the job execution script and the job submission file.

## Submitting jobs concurrently

If we want to submit several jobs, we need to track log, out and error  files for each
job. An easy way to do this is to add the $(Cluster) and $(Process)
macros to the HTCondor submit file. 

~~~
# The UNIVERSE defines an execution environment. You will almost always use VANILLA.
Universe = vanilla

# These are good base requirements for your jobs on OSG. It is specific on OS and
# OS version, core cound and memory, and wants to use the software modules. 
Requirements = OSGVO_OS_STRING == "RHEL 6" && HAS_MODULES == True
request_cpus = 1
request_memory = 1 GB

# executable is the program your job will run It's often useful
# to create a shell script to "wrap" your actual work.
executable = R-wrapper.sh
arguments = mcpi.R $(Process)

# files transferred into the job sandbox
transfer_input_files = mcpi.R

# error and output are the error and output channels from your job
# that HTCondor returns from the remote host.
output = Log/job.out.$(Cluster).$(Process)
error = Log/job.error.$(Cluster).$(Process)

# The log file is where HTCondor places information about your
# job's status, success, and resource consumption.
log = Log/job.log.$(Cluster).$(Process)

# Send the job to Held state on failure. 
on_exit_hold = (ExitBySignal == True) || (ExitCode != 0)  

# Periodically retry the jobs every 60 seconds, up to a maximum of 5 retries.
periodic_release =  (NumJobStarts < 5) && ((CurrentTime - EnteredCurrentStatus) > 60)

# Queue is the "start button" - it launches any jobs that have been
# specified thus far.
queue 100
~~~

Note the `Queue 100`.  This tells Condor to enqueue 100 copies of this job
as one cluster.  

Let us take a look at the execution script, R-wrapper.sh

~~~
#!/bin/bash
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
module load R
Rscript $1 > mcpi.$2.out
~~~

The wrapper loads the R module and then executes the script with Rscript
utility. From the submit file described above, the first argument is the
name of the R program - mcpi.R and the second argument is the process
number. The process number is a sequence of integers and used here to
name the output files.

You'll see something like the following upon submission:

~~~
$ condor_submit R.submit
Submitting job(s).........................
100 job(s) submitted to cluster 837.
~~~

Apply your `condor_q` knowledge to see this job
progress. Execute the following bash script to compute the average from all the jobs.


## Post process 

Once the jobs are completed, you might want to invoke the script 

~~~
$./mcpi_ave.bash
~~~

to compute the average value of pi from all the available outputs.  


## Challenges
<div class="challenge" markdown="1">
Can you get a more accurate estimation of Pi? First try to make longer jobs by increasing
the value in `montecarloPi(1000)`.
</div>
<div class="challenge" markdown="1">
Can you get a more accurate estimation of Pi by increasing the number of jobs from 100 to 1000? 
</div>


# Optimization Tool box: Simulated Annealing

## MATLAB Overview

[MATLAB®](http://www.mathworks.com/products/matlab/) is a licensed high level language and interactive modeling and development toolkit. The [MATLAB Compiler™](http://www.mathworks.com/products/compiler/) lets you share MATLAB programs as standalone applications.  All applications created with MATLAB Compiler use [MATLAB Runtime™ (MCR)](http://www.mathworks.com/products/compiler/mcr/), which enables royalty-free deployment and use. 

MATLAB Compiler is invoked with `mcc`.  Most toolboxes and user-developed interfaces are supported. For more details, check the list of [supported toolboxes](http://www.mathworks.com/products/compiler/supported/compiler_support.html) and⋅
[ineligible programs](http://www.mathworks.com/products/ineligible_programs/).  MATLAB Runtime is available on all OSG 
sites using the OASIS software service using `module` commands.


[MATLAB's Optimization Toolbox](http://www.mathworks.com/products/optimization/) tackles wide variety of solvers such as linear programming, mixed-integer linear programming, quadratic programming, nonlinear optimization. In this tutorial, we learn how to use simulated annealing to find the minimum of a function. The test function is the well known Rosenbrock function. 


![fig 1](https://raw.githubusercontent.com/OSGConnect/tutorial-matlab-SimulatedAnnealing/master/Figs/RosenBrockFunction.png)
Fig.1. Two dimensional Rosenbrock function along x-y plane.⋅⋅

## Tutorial files

It is easiest to start with the `tutorial` command. In the command prompt, type

    $ tutorial tutorial-matlab-SimulatedAnnealing

This will create a directory `tutorial-matlab-SimulatedAnnealing`. Inside the directory, you will see the following files

    SA_Opt.m               # matlab script - simulated annealing optimization of the function `simple_objective.m`
    simple_objective.m     # matlab script - defines the actual objective function
    SA_Opt                 # matlab compiled binary
    SA_Opt.submit          # Condor job description file
    SA_Opt.sh              # Executable file
    Log/                   # Directory to copy standard output, error and log files from condor jobs.



## MATLAB script -  Objective function

Lets take a look at the objective function that takes an argument of an array x. The size of the array is two.

    function y = simple_objective(x)
         y = (1-x(1))^2 + 100*(x(2)-x(1)^2)^2;

The objective function is defined in the matlab file `simple_objective.m`. The two dimensional Rosenbrock 
function defined here has a minimum at `(1,1)`. Rosenbrock function is one of the test function used to test 
the robustness of an optimization method. Now let us see how the simulated annealing performs. 

## MATLAB script - Optimization
The simulated annealing script calls the objective function and optimizes via `simulannealbnd`.⋅

    %Simulated Annealing optimization of a given function
    function SA_Optimization(fnumber)
        filenumber = num2str(fnumber);
        outfilename = sprintf ( '%s%s%s', 'rosen-sa-opt', filenumber, '.dat' );
        fileID = fopen(outfilename,'w');

        ObjectiveFunction = @simple_objective;
        rng('shuffle');
        for n = 1:1:5
            rx = rand(1:2)*100.0;
            X0 = [2.0 2.0] + rx;   % Starting point
            [x,fval,exitFlag,output] = simulannealbnd(ObjectiveFunction,X0) %
            fprintf(fileID,'f= %12.6f  x= %9.5f   y= %9.5f x0= %9.5f  y0=%9.5f\n', fval, x, X0 );
        end
        fclose(fileID);

In the above script, the optimizatoin is repeated for five times with random intial conditions.⋅

## MATLAB runtime execution
We need to compile the matlab script on a machine with license. At present, we don't have license 
for matlab on OSG. On a⋅machine with matlab license, invoke the compiler `mcc`. It is 
important to turn off all⋅ graphical options (-nodisplay), disable Java (-nojvm), and instruct MATLAB to run this⋅
program as a single-threaded application (-singleCompThread). The flag -m means `c` language 
translation during compilation.  

    mcc -m -R -singleCompThread -R -nodisplay -R -nojvm SA_Opt.m

would produce the files: `SA_Opt, run_SA_Opt.sh, mccExcludedFiles.log and readme.txt`.  The file `SA_Opt`⋅
is the compiled binary file that we would like to run on OSG.⋅

Check  further details of compilation process in the [lesson on basics of MATLAB compilation] (https://support.opensciencegrid.org/support/solutions/articles/5000660751-basics-of-compiled-matlab-applications-hello-world-example), we need to compile the matlab script on a machine with license.

## Job execution and submission files

Let us take a look at `SA_Opt.submit` file:⋅

~~~
Universe = vanilla  

# These are good base requirements for your jobs on OSG. It is specific on OS and
# OS version, core count and memory, and wants to use the software modules. 
Requirements = OSGVO_OS_STRING == "RHEL 6" && Arch == "X86_64" &&  HAS_MODULES == True
request_cpus = 1
request_memory = 1 GB

Executable = SA_Opt.sh
Arguments = $(Process)

transfer_input_files =  SA_Opt

Output = Log/job.$(Process).out 
Error =  Log/job.$(Process).err
Log =    Log/job.$(Process).log

# Send the job to Held state on failure. 
on_exit_hold = (ExitBySignal == True) || (ExitCode != 0)

# Periodically retry the jobs every 1 hour, up to a maximum of 5 retries.
periodic_release =  (NumJobStarts < 5) && ((CurrentTime - EnteredCurrentStatus) > 60)

queue 10
~~~

The above job description instructs condor to submit 10 jobs. Each job would start with different random intial conditions.

The executable is a wrapper script `SA_Opt.sh`

    #!/bin/bash⋅
    source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/current/init/bash
    module load matlab/2014b
    chmod +x SA_Opt
    ./SA_Opt $1

that loads the module `matlab/2014b` and executes the MATLAB compiled binary `SA_Opt`. The only required⋅
argument is a numerical label that would be attached with the name of the output file. 


## Job submision 

We submit the job using `condor_submit` command as follows

    $ condor_submit SA_Opt.submit  # Submit the condor job description file "SA_Opt.submit"

Now you have submitted the an ensemble of 10 jobs. The jobs should be finished quickly (less than few minutes). You can check the status of the submitted job by using the `condor_q` command as follows

    $ condor_q username  # The status of the job is printed on the screen. Here, username is your login name.

Each job produce rosen-sa-opt$(Process).dat file, where $(Process) is the process ID that runs from 0 to 9.⋅

## Post process 

After all jobs finished, we want to gather the output data. The script `post-script.bash` gathers the⋅
output values and numerically sort them according to function values.⋅

    $ ./post-script.bash 



<div class="keypoints" markdown="1">

#### Key Points
*    Scaling up the computational resources on OSG is crucial to taking full advantage of grid computing.
*    Changing the value of *Queue* allows the user to scale up the resources.
*    *Arguments* allows you to pass parameters to a job script.
*    $(Cluster) and $(Process) can be used to name log files uniquely, and as arguments
</div>
