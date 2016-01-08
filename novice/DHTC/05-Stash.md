---
layout: lesson
root: ../..
title: Data storage and transfer
---
<div class="objectives" markdown="1">

#### Objectives
*   Discover how to transfer input and output data  
</div>


<h2> Overview </h2>
In this lesson, we will learn the basics of data storage and transfer for DHTC jobs and on Crane. 

<h2>Introduction to Data Handling for DHTC jobs</h2>
Data handling is one of the trickiest parts of effectively using DHTC computing.
New users often run into issues in regards to effectively transferring inputs to compute
jobs or with getting output back.  However, most situations can be broken down
into a few basic use cases: 
   * Jobs with small inputs and outputs
   * Jobs with large inputs but small outputs
   * Jobs with small inputs but large outputs
   * Jobs with both large inputs and large outputs

We will consider each use case one by one.

<h3>Jobs without large inputs or outputs</h3>
Jobs without large inputs or outputs are the easiest case to handle.   If jobs
have inputs and outputs that are smaller than ~1 gigabyte then the builtin HTCondor
file transfer mechanims will work to handle transfers to and from the compute node.  
Input files can be specified using the `transfer_input_files` option in your
submit file.  Likewise output files and directions can be specified using
`transfer_output_files`.  Once the input and output files are given to HTCondor,
it will automatically transfer the input files to compute nodes when jobs are
scheduled to run and then transfer the output after the job completes back to
your submit node.

<h3>Jobs with large inputs but small outputs</h3>
Jobs with large inputs but small outputs are a little harder to handle.
Examples of these types of jobs, would be things such as BLAST which does
searches for similar DNA sequences.  The inputs (DNA reference databases and DNA
sequence of interest) can easily be tens or hundreds gigabytes.  However, the
output (locations of similar sequences and similarity of these sequences) tends
to be kilobytes or megabytes in size.  

If the majority of the inputs are databases or reference files a possible
solution would be to place the input files on a publicly accessible webserfer.
Then jobs would wget the appropriate files when they start running.  Since most
OSG sites have a Squid server, this will minimize the network traffic outside of
the site running your jobs.  If the majority of the input is in reference files
that are identical between jobs, then another alternative would be to use a
service such as OASIS that will make the reference files available on the
majority of OSG sites.  Jobs can then access the files as if they are on the
compute node and only the files that are accessed will be transferred.  If the
input files need to remain private or if the inputs drastically change between
jobs, you should probably discuss this with us so that we can help you come up
with a good solution.

Since the output files are small (i.e. < ~1GB), using the `transfer_output_files` option
in your submit file and allowing HTCondor to manage transferring outputs from
the compute nodes to the submit node should work without problems.

<h3>Jobs with small inputs and large outputs</h3>
This use case deals with jobs that generate large outputs from a small set of
inputs.  For example, a simulation may generate a large data set showing the
evolution of a physical system based on a small set of input parameters.  

The inputs for this type of jobs can be handled using the HTCondor transfer
mechanisms.  By using the `transfer_input_files` option in your submit file,
HTCondor will automatically handle transferring inputs to compute nodes when
your job runs.  

The outputs for this type of job are more difficult to handle.  Solutions may
range from compressing the outputs to generate smaller, more manageable files to
setting up or utilizing a service like gridftp to copy files back to your login
node or a storage system for later retrieval.  Regardless, it is probably best
to talk to us so that we can come up with the best solution for your transfer
needs. 

<h3>Jobs with both large inputs and large outputs</h3>
This use case is essentially just a combination of the prior two use cases.  As
such, inputs can be handled using a web server or OASIS and transferring outputs
will probably need some discussion with us to determine the best solution.

<h2>Intial setup on Crane</h2>

First, do some initial setup on Crane to faciliate some of the later 
exercises. You'll need to log in to Crane:

~~~
ssh username@crane.unl.edu #Connect to the login node with your username
passwd:       # your password
~~~

Once done, create a directory to work in

~~~
$ mkdir ~/transfer    
$ cd ~/transfer    
~~~

This directory is will serve as an area to create files and directories that
we'll try to transfer from Crane to your laptop and vice versa.

For future use, let's create a file:

~~~
$ echo "Hello world" > my_hello_world
~~~

In addition, let's create a directory as well for future use:

~~~
$ mkdir my_directory
~~~

Also, note that there is a separate space that should be used for I/O intensive
tasks at `/work/<group>/<username>`.

<h2>Transferring files to and from Crane using SCP (Windows)</h2>
Unlike with OS X or Linux/Unix systems, Windows doesn't come with scp installed
by default.  Instead, we will use an application called WinSCP that provides a
GUI interface for scp. The procedure to login to Crane using WinSCP can be found 
[here](https://hcc-docs.unl.edu/display/HCCDOC/For+Windows+Users).  Note, that
rather than using `tusker.unl.edu` as the hostname, you should use
`crane.unl.edu`.

<h2>Transferring files to and from Crane using SCP (OS X or Linux/Unix)</h2> 

We can transfer files to Crane using `scp`. First, let's 
look at transferring files using `scp`.  `Scp` is a counterpart to ssh that allows for
secure, encrypted file transfers between systems using your ssh credentials.    

To transfer a file from Crane using `scp`, you'll need to run `scp` with the
source and destination.  Files on remote systems are indicated using
user@machine:/path/to/file .  Let's copy the file we just created from Crane to
our local system:

~~~
$ scp username@crane.unl.edu:~/transfer/my_hello_world .
~~~

As you can see, `scp` uses similar syntax to the `cp` command that you were shown
previously.  To copy directories using `scp`, you'll just pass the `-r` option to
it.  E.g:

~~~
$ scp -r username@crane.unl.edu:~/transfer/my_directory .
~~~

> #### Challenges
>
> * Create a directory with a file called `hello_world_2` in the `~/transfer` directory and copy it from Crane to your local system.
> * Create a directory called `hello_world_3` on your local system and copy it to the `transfer` directory.

<h2>Transferring files to and from Crane using Globus</h2>
An alternate method for accessing your files on Crane is to use Globus.  Globus allows you
to initiate transfers between Globus endpoints and will handle the actual file
and directory transfers transparently without needing further input from you.
When the transfer is complete, Globus will send a notification to you indicating
this.

Let's transfer a file from your laptop to Globus.  Doing this will require an 
application called Globus Connect Personal that allows Globus to transfer files
and directories to and from your laptop. First go to the [Globus
page](https://www.globus.org/globus-connect-personal) and download and install
the globus connect personal installer specific to your system.  

While that's running, you'll need to get a setup key from Globus in order to
setup the Globus Connect Personal software.  

*   First go to the [Globus](http://globus.org) page and register or login
*   Next go to this [page](https://globus.org/xfer/ManageEndpoints#)
*   Click on the add Globus Connect Personal link
*   Enter a name for your endpoint on the page (remember this!)
*   Click on "Generate setup Key"
*   Select the key and copy the key

Finally, start the Globus online personal software that you just installed.  The
installer will ask for the setup key that you obtained from the Globus website.
At this point, the install and setup of Globus Connect Personal is complete.

Now go to [Globus](https://www.globus.org/app/transfer). 
For the first endpoint, enter username#name
where name is the name you choose for the endpoint above. You should now see the
files from your laptop displayed.  For the second endpoint, enter
`hcc#crane` and hit enter.  You should now see the contents of your home
directory on Crane.  Now double click on the `transfer` directory.  Select a
file on your laptop and click on the right arrow on the top of the screen to
start a transfer to Crane. You can transfer files or directories to your
laptop by selecting it in the Crane window and selecting the left arrow.

> #### Challenges 
>
> * Copy a file to Crane from your laptop using Globus.  
> * Next copy the `my_hello_world` file from Crane to your laptop using Globus.

<h2>Putting it all together</h2>

Let us do an example calculation to give us a taste of a what a typical
job may involve. We will peform a  molecular dynamics simulation of a
small protein in implicit water. To get the necessary files, we use the *tutorial*
command on Crane. 

Log in to Crane and setup the tutorial command:

~~~
$ ssh username@crane.unl.edu
$ source osg_oasis_init
~~~

Type:

~~~
$ tutorial namd
$ cd ~/tutorial-namd
$ rm *.inp
$ rm *.conf
~~~

*Aside*: [NAMD](http://www.ks.uiuc.edu/Research/namd/) is a widely used
molecular dynamics simulation program. It lets users specify a molecule in some
initial state and then observe its time evolution subject to forces.
Essentially, it lets you go from a specifed molecular
[structure](http://en.wikipedia.org/wiki/Superoxide_dismutase#mediaviewer/File:Superoxide_dismutase_2_PDB_1VAR.png)
to a [simulation](https://www.youtube.com/watch?v=mk3cLd9PUPA&list=PL418E1C62DD9FC8BA&index=1)
of its behavior in a particular environment.  It has been used to study polio
eradication, similations of graphene, and studies of biofuels.

You should see the following files in the directory:

~~~
$ ls
namd_stash_run.sh      ubq.psf
namd_stash_run.submit  README.md         ubq.pdb
~~~

The files 

~~~
namd_stash_run.submit #HTCondor job submission script file.
namd_stash_run.sh #Job execution script file.
ubq.pdb #Input pdb file for NAMD.
ubq.psf #Input file for NAMD.
~~~

We need two files in order to successfully run the simulation. 
The file `par_all27_prot_lipid.inp` which is the parameter file and is required for 
the NAMD simulations. The other file needed is `ubq_gbis_eq.conf` which has
the input configuration for the NAMD simulations.  

> #### Challenges
>
> * You can find the missing files at https://stash.osgconnect.net/+sthapa/tutorial-namd, 
    download the files from that website and upload it to the tutorial-namd file on Crane.

 Now submit the NAMD job. 


 ~~~
 
 $ condor_submit namd_run.submit 
 
 ~~~

 Once the job completes, you will see non-empty `ubq_gbis_eq.0.log`
 file where the standout output from the programs is written.

> #### Challenges
>
> * Download the output file from the NAMD job to your laptop and view it.  Try
> using a different method to transfer the file than before.


You should see:


 ~~~
 
 $ tail  ubq_gbis_eq.0.log

 WallClock: 6.084453  CPUTime: 6.084453  Memory: 53.500000 MB
 Program finished.
 
 ~~~

 The above lines indicate the NAMD simulation was successful. 

<div class="keypoints" markdown="1">

#### Key Points
* Data can be transferred in and out of Crane using scp and Globus
</div>

