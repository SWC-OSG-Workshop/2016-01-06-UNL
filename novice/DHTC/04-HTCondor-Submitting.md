---
layout: lesson
root: ../..
title: Job submission on HCC OSG HTCondor Pool
---
<div class="objectives" markdown="1">

#### Objectives
*   Learn how to submit HTCondor jobs.
*   Learn how to monitor the running jobs.
</div>

<h2>Overview</h2> In this section, we will learn the basics of HTCondor
in submitting and monitoring workloads, or "jobs". The jobs are
submitted through the login node of Crane. The submitted jobs are
executed on the remote worker node(s) and the outputs are transfered
back to the login node. In the HTCondor job submit file, we have to
describe how to execute the program and transfer the output data.

Note that even though we use the submit node of Crane, we are not
going to use the default scheduler for Crane, and thus not use the
rest of the cluster.

For more information, see the [HCC OSG HTCondor Documentation](https://hcc-docs.unl.edu/display/HCCDOC/HTCondor+on+the+OSG)

<h2>Login to Crane</h2>

First, we log in to Crane:

~~~
$ ssh username@crane.unl.edu    # username is your username
password:                       # enter your password
~~~

We will get our example files for all today's lessons using `tutorial`.

Let's get started with the *quickstart* tutorial:

~~~
$ tutorial quickstart       # creates a directory "tutorial-quickstart"
$ cd tutorial-quickstart    # script and input files are inside this directory
~~~

We will look at two files in detail: "short.sh" and "tutorial01.submit"

##Job execution script##

Inside the tutorial directory, open up `short.sh` in an editor.

~~~
$ nano short.sh
~~~

This is a shell script, quite ordinary and much like the ones we worked
with in Unit I.

~~~
#!/bin/bash
# short.sh: a short discovery job
printf "Start time: "; /bin/date
printf "Job is running on node: "; /bin/hostname
printf "Job running as user: "; /usr/bin/id
printf "Job is running in directory: "; /bin/pwd
echo
echo "Working hard..."
sleep ${1-15}
echo "Science complete!"
~~~

To close nano, hold down Ctrl and press X. Press Y to save, and then
Enter Now, make the script executable.  Recall that this is not
necessary for shell programs that you create and run locally.  However,
_it is extremely important for jobs running on the grid_.  So is the
"shbang" line (`#!/bin/sh`).

~~~
$ chmod +x short.sh
~~~

Since we used the tutorial command, all files are already in your
workspace. Run the job locally when setting up a new job type -- it is
important to test your job outside of HTCondor before submitting into
the Open Science Grid.

~~~
$ ./short.sh
~~~

~~~
Start time: Tue Dec 22 13:15:20 CST 2015
Job is running on node: login.crane.hcc.unl.edu
Job running as user: uid=1838(rynge) gid=11156(bockelman) groups=11156(bockelman)
Job is running in directory: /home/bockelman/rynge/tutorial-quickstart

Working hard...
Science complete!
~~~

##Job submission file##
So far, so good! Next we will create a simple (if verbose) HTCondor
submit file.  A submit file tells the grid software _how_ to run
your workload, with what properties and arguments, and how to collect
and return output to you.

~~~
$ nano tutorial01.submit
~~~

~~~
# The UNIVERSE defines an execution environment. You will almost always use VANILLA.
Universe = vanilla

# EXECUTABLE is the program your job will run It's often useful
# to create a shell script to "wrap" your actual work.
Executable = short.sh

# ERROR and OUTPUT are the error and output channels from your job
# that HTCondor returns from the remote host.
Error = job.error
Output = job.output

# The LOG file is where HTCondor places information about your
# job's status, success, and resource consumption.
Log = job.log

# QUEUE is the "start button" - it launches any jobs that have been
# specified thus far.
Queue 1
~~~

## Job submission ##

Submit the job using `condor_submit`.

~~~
$ condor_submit tutorial01.submit
Submitting job(s).
1 job(s) submitted to cluster 823.
~~~

##Job status##

Your first job is on the grid! The `condor_q` command tells the status of
currently running jobs. Generally you will want to limit it to your own
jobs by adding your own username to the command.

~~~
$ condor_q username
-- Submitter: crane.unl.edu : <128.135.158.173:43606> : crane.unl.edu
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
 823.0   username           8/21 09:46   0+00:00:06 R  0   0.0  short.sh
1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
~~~

If you want to see all jobs running on the system, use condor_q without
any extra parameters.  You can also get status on a specific job
cluster -- the number that `condor_submit` gave you.

~~~
$ condor_q 823
-- Submitter: crane.unl.edu : <128.135.158.173:43606> : crane.unl.edu
 ID      OWNER            SUBMITTED     RUN_TIME ST PRI SIZE CMD
 823.0   username           8/21 09:46   0+00:00:10 R  0   0.0  short.sh
1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
~~~

A _job cluster_ identifies one batch of jobs.  The batch could have one job
or ten thousand in it -- what matters is that each time a submit file says
"Queue", you get a cluster.  The individual jobs within a job cluster are
identified by the numbers after the dot in the job ID -- so in this example,
823 is the job cluster, and 823.0 is the job ID (or jobid) of job 0 in that
cluster.

Note the ST (state) column. Your job will be in the **I** state (idle)
if it hasn't started yet. If it's currently scheduled and running, it
will have state **R** (running). If it has completed already, it will
not appear in condor_q.

You may sometimes see jobs in **H** state.  These are _held_ jobs. Held
jobs are stalled, usually for a specific reason, and won't progress until
released.  Until you gain savvy with diagnosing why a job is held and
solving it on your own, you may contact the HCC support team for help
with held jobs.

Let's wait for your job to finish – that is, for condor_q not to show
the job in its output.

~~~
$ condor_q username | tail -1
1 jobs; 0 completed, 0 removed, 0 idle, 1 running, 0 held, 0 suspended
~~~

You could run this over and over to watch for the job to complete -- but
`watch` can make this simpler.  Let's submit the job again, and
watch condor_q output at five-second intervals. Your first
job has probably already completed by now, so submit a new one first:

~~~
$ condor_submit tutorial01.submit
Submitting job(s).
1 job(s) submitted to cluster 823
$ watch -n 5 condor_q
~~~

When your job has completed, it will disappear from the list.  To close
`watch`, press control-C -- hold down Control and press C.

##Job history##
Once your job has finished, you can get information about its execution
from the `condor_history` command:

~~~
$ condor_history 823
 ID      OWNER            SUBMITTED     RUN_TIME ST   COMPLETED CMD
 823.0   username            8/21 09:46   0+00:00:12 C   8/21 09:46 /home/username/
~~~

You can see much more information about your job's final status using
the -long option.


##Job output##

Once your job has finished, you can look at the files that HTCondor
has returned to the working directory. If everything was successful,
it should have returned:

* a log file from Condor for the job cluster: `job.log`
* an output file for each job's output: `job.output`
* an error file for each job's errors: `job.error`

Read the output file. It should be something like this:

~~~
$ cat job.output
Start time: Thu Feb 19 15:11:43 EST 2015
Job is running on node: VPA-WH505-01-S4-its-u12-nfs-20141003
Job running as user: uid=1066(osgconnect) gid=502(condoruser) groups=502(condoruser),108(fuse)
Job is running in directory: /tmp/rcc_syracuse/rcc.9jiqjUmxeB/execute.10.5.30.6-1128/dir_1380

Working hard...
Science complete!
~~~

## Unscheduling jobs ##

Once you know how to create files, you want to know how to delete
them.  And once you can schedule workloads across thousands of computers
simultaneously, you need to know how to remove them.  The command for that
is `condor_rm`, and it takes only one argument: the job cluster or job ID.

~~~
$ condor_submit tutorial01.submit
Submitting job(s).
1 job(s) submitted to cluster 829
$ condor_rm 829
Cluster 829 has been marked for removal.
~~~

Or alternately:

~~~
$ condor_submit tutorial01.submit
Submitting job(s).
1 job(s) submitted to cluster 829
$ condor_rm 829.0
Job 829.0 has been marked for removal.
~~~

## Job Requirements - A Basic OSG Job ##

HTCondor job and machines are represented as HTCondor classads. These are sets
of key/value attribute pairs. Let's examine a what a machine classad looks like.
This is a two step process, first we get a name for one of the machines, and then
we ask condor_status to give us the details for that machine (-long).

~~~
$ condor_status -pool glidein.unl.edu -af Name | head -n 1
[resource name]
$ condor_status -long -pool glidein.unl.edu [resource name] | sort
HAS_FILE_usr_lib64_libgfortran_so_3 = true
HAS_MODULES = false
OSGVO_OS_STRING = "RHEL 6"
~~~

You can make use any of these attributes to limit where your jobs go. The following
is a fairly complete OSG job which you can use when getting started on OSG. Note
the `Requirements` and `requests_*` lines.

~~~
# The UNIVERSE defines an execution environment. You will almost always use VANILLA.
Universe = vanilla

# These are good base requirements for your jobs on OSG. It is specific on OS and
# OS version, core cound and memory, and wants to use the software modules. 
Requirements = OSGVO_OS_STRING == "RHEL 6" && HAS_MODULES == True
request_cpus = 1
request_memory = 1 GB

# EXECUTABLE is the program your job will run It's often useful
# to create a shell script to "wrap" your actual work.
Executable = short.sh

# ERROR and OUTPUT are the error and output channels from your job
# that HTCondor returns from the remote host.
Error = job.$(Cluster).$(Process).error
Output = job.$(Cluster).$(Process).output

# The LOG file is where HTCondor places information about your
# job's status, success, and resource consumption.
Log = job.log

# Send the job to Held state on failure. 
on_exit_hold = (ExitBySignal == True) || (ExitCode != 0)  

# Periodically retry the jobs every 60 seconds, up to a maximum of 5 retries.
periodic_release =  (NumJobStarts < 5) && ((CurrentTime - EnteredCurrentStatus) > 60)

# QUEUE is the "start button" - it launches any jobs that have been
# specified thus far.
Queue 1

~~~

You can test this job by submitting and monitoring it as we have just covered:


~~~
$ condor_submit osg-template-job.submit
Submitting job(s).
1 job(s) submitted to cluster 830
~~~

The filenames for this job includes a job id, which means that if you submit more
than one job, they will all have unique outputs.

~~~
$ ls *.output
job.830.0.output
job.831.0.output
~~~


<div class="keypoints" markdown="1">

#### Key Points
*   HTCondor shedules and monitors your Jobs.
*   To submit a job to HTCondor, you must prepare the job execution and job submission scripts.
*   *condor_submit* - HTCondor's job submission command.
*   *condor_q* - HTCondor's job monitoring command.
*   *condor_rm* - HTCondor's job removal command.
</div>

## Challenges
<div class="challenge" markdown="1">
What happens if we leave the **Queue** line out of a submit file?
</div>

<div class="challenge" markdown="1">
What happens if we write only `Queue`, with no argument?
</div>

<div class="challenge" markdown="1">
`condor_history -long username` gives a LOT of extended information about
your past jobs, ordered as key-value pairs.  Try it with your a single
job from your last cluster:

~~~
$ condor_history -long ######.0
~~~

Included among these attributes is the `RemoteWallClockTime` parameter,
which tells how long your job ran on the remote worker.  How might you
collect this value across all your historical jobs?

(Remember that the `grep` command can be used to pick out specific
patterns from text.)
</div>
