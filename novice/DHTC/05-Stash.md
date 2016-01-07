---
layout: lesson
root: ../..
title: Data storage and transfer
---
<div class="objectives" markdown="1">

#### Objectives
*   Discover how to transfer input and output data  
*   Learn the what, why, how, and whens of using Stash
</div>


<h2> Overview </h2>
In this lesson, we will learn the basics of data storage and transfer for DHTC jobs and on Crane. 

<h2>Introduction to Data Handling for DHTC jobs</h2>
Data handling is one of the trickiest parts of effectively using DHTC computing.
New users often run into issues in regards to effectively transferring inputs to compute
jobs or with getting output back.  However, most situations can be broken down
into a few basic use cases: 
   * Jobs without large inputs or outputs
   * Jobs with large inputs but small outputs
   * Jobs with small inputs but large outputs
   * Jobs with both large inputs and large outputs

We will consider each use case one by one.

<h3>Jobs without large inputs or outputs</h3>
Jobs without large inputs or outputs are the easiest case to handle.   If jobs
have inputs and outputs that are smaller than ~1 gigabyte then the builtin HTCondor
file transfer mechanims will work to handle transfers to and from the compute node.  
Input files can be specified using the =transfer_input_files= option in your
submit file.  Likewise output files and directions can be specified using
=transfer_output_files=.  Once the input and output files are given to HTCondor,
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

Since the output files are small (i.e. < ~1GB), using the =transfer_output_files= option
in your submit file and allowing HTCondor to manage transferring outputs from
the compute nodes to the submit node should work without problems.

<h3>Jobs with small inputs and large outputs</h3>
This use case deals with jobs that generate large outputs from a small set of
inputs.  For example, a simulation may generate a large data set showing the
evolution of a physical system based on a small set of input parameters.  

The inputs for this type of jobs can be handled using the HTCondor transfer
mechanisms.  By using the =transfer_input_files= option in your submit file,
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

<h2>Exploring the Stash system</h2>

First, we'll look at accessing Stash from the login node. You'll need to log
in to Crane:

~~~
ssh username@crane.unl.edu #Connect to the login node with your username
passwd:       # your password
~~~

Once done, you can change to the `stash` directory in your home area:

~~~
$ cd ~/stash    
~~~

This directory is an area on Stash that you can use to store files and
directories.  It functions just like any other UNIX directory although it has
additional functions that we'll go over shortly.

For future use, let's create a file in Stash:

~~~
$ cd ~/stash
$ echo "Hello world" > my_hello_world
~~~

In addition, let's create a directory as well for future use:

~~~
$ mkdir my_directory
~~~



<h2>Transferring files to and from Stash using SCP </h2> 

We can transfer files to Stash using `scp`. First, let's 
look at transferring files using `scp`.  `Scp` is a counterpart to ssh that allows for
secure, encrypted file transfers between systems using your ssh credentials.    

To transfer a file from Stash using `scp`, you'll need to run `scp` with the
source and destination.  Files on remote systems are indicated using
user@machine:/path/to/file .  Let's copy the file we just created from Stash to
our local system:

~~~
$ scp username@login.duke.ci-connect.net:~/stash/my_hello_world .
~~~

As you can see, `scp` uses similar syntax to the `cp` command that you were shown
previously.  To copy directories using `scp`, you'll just pass the `-r` option to
it.  E.g:

~~~
$ scp -r username@crane.unl.edu:~/my_directory .
~~~

> #### Challenges
>
> * Create a directory with a file called `hello_world_2` in the `~/stash` directory and copy it from Stash to your local system.
> * Create a directory called `hello_world_3` on your local system and copy it to the `data` directory.

<h2>Transferring files to and from Stash using Globus</h2>
An alternate method for accessing Stash is to use Globus.  Globus allows you
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

*   First login to [Duke Connect](http://duke.ci-connect.net) using your Duke Connect username and password
*   Next go to this [page](https://portal.duke.ci-connect.net/xfer/ManageEndpoints#)
*   Click on the add Globus Connect Personal link
*   Enter a name for your endpoint on the page (remember this!)
*   Click on "Generate setup Key"
*   Select the key and copy the key

Finally, start the Globus online personal software that you just installed.  The
installer will ask for the setup key that you obtained from the Globus website.
At this point, the install and setup of Globus Connect Personal is complete.

Now go to [http://duke.ci-connect.net](http://login.duke.ci-connect.net) and under the
Transfer menu, select Start Transfer.  For the first endpoint, enter username#name
where name is the name you choose for the endpoint above. You should now see the
files from your laptop displayed.  For the second endpoint, enter
`connect#stash` and hit enter.  You should now see the contents of your home
directory on Duke Connect.  Now double click on the `data` directory.  Select a
file on your laptop and click on the right arrow on the top of the screen to
start a transfer to Stash. You can transfer files or directories to your
laptop by selecting it in the Stash window and selecting the left arrow.

> #### Challenges 
>
> * Copy a file to Stash from your laptop using Globus.  
> * Next copy the `my_hello_world` file from Stash to your laptop using Globus.


<h2>Transferring files from Stash using HTTP</h2>
Stash also allows you to access files using your web browser.  In order to do
this, you'll need to put your file in `~/public`or `~/stash/public` (the two locations 
point to the same directory). Any file or directory that is placed 
here  will be made available in the Stash webserver.  Let's make a file
available using the Stash webserver

~~~
$ cd ~/public
$ echo "This is served over the web" > web-file
~~~

Now go to `http://stash.osgconnect.net/+username/` in your browser.  You should
see the file in the listing.  Clicking on the file should give you the contents.

> #### Challenge 
>
> * Create a file called `my-web-file` and make it available through the Stash webserver.


<h2>Using data on Stash in compute jobs</h2> 

Let us do an example calculation to understand the use of Stash and how we download 
the data from the web. We will peform a  molecular dynamics simulation of a small 
protein in implicit water. To get the necessary files, we use the *tutorial* command on 
OSG. 

Log in to Duke Connect:

~~~
$ ssh username@login.duke.ci-connect.net
~~~

Type:

~~~
$ tutorial stash-namd
$ cd ~/tutorial-stash-namd
~~~

*Aside*: [NAMD](http://www.ks.uiuc.edu/Research/namd/) is a widely used molecular dynamics simulation program. It lets users specify a molecule in some initial state and then observe its time evolution subject to forces. Essentially, it lets you go from a specifed molecular [structure](http://en.wikipedia.org/wiki/Superoxide_dismutase#mediaviewer/File:Superoxide_dismutase_2_PDB_1VAR.png) to a [simulation](https://www.youtube.com/watch?v=mk3cLd9PUPA&list=PL418E1C62DD9FC8BA&index=1) of its behavior in a particular environment.  It has been used to study polio eradication, similations of graphene, and studies of biofuels.

You should see the following files in the directory:

~~~
$ ls
namd_stash_run.sh      par_all27_prot_lipid.inp  ubq_gbis_eq.conf  ubq.psf
namd_stash_run.submit  README.md         ubq.pdb
~~~

The files 
~~~
namd_stash_run.submit #HTCondor job submission script file.
namd_stash_run.sh #Job execution script file.
ubq_gbis_eq.conf #Input configuration for NAMD.
ubq.pdb #Input pdb file for NAMD.
ubq.psf #Input file for NAMD.
par_all27_prot_lipid.inp #Parameter file for NAMD.
~~~

The file `par_all27_prot_lipid.inp` is the parameter file and is required for 
the NAMD simulations. The parameter file is common data file for the NAMD
simulations. It is a good practice to keep the common files, like  the parameter file 
in our example, in the Stash storage.  

~~~
mv par_all27_prot_lipid.inp ~/public/.  
~~~

You can view the parameter file using your web browser by going to 
`http://stash.osgconnect.net/+yourusername`.

Now we want the parameter file available on the execution (worker) machine when the 
simulation starts to run. As mentioned early, the data on the Stash is available to 
the execution machines. This means the execution machine can transfer the data from 
Stash as a part of the job execution. So we have to script this in the job execution 
script. 

You can see that the job script `namd_stash_run.sh` has the following lines:

~~~
$ cat namd_stash_run.sh
#!/bin/bash 
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/5.6.2/init/bash 
module load namd/2.9  
wget http://stash.osgconnect.net/+username/par_all27_prot_lipid.inp  
namd2 ubq_gbis_eq.conf  
~~~

In the above script, you will have to insert your "username" in URL address. The
parameter file located on Stash is downloaded using the `wget` utility.  
 

Now we submit the NAMD job. 

~~~
$ condor_submit namd_stash_run.submit 
~~~

Once the job completes, you will see non-empty `namdoutput_using_stash.dat` file where 
the standout output from the programs is written.

~~~
$ tail  namdoutput_using_stash.dat

WallClock: 6.084453  CPUTime: 6.084453  Memory: 53.500000 MB
Program finished.
~~~

The above lines indicate the NAMD simulation was successful. 

 
<div class="keypoints" markdown="1">

#### Key Points
* Stash is located at ~/stash and ~/public on login.duke.ci-connect.net.
* Data can be transferred in and out of Stash using scp, Globus, and HTTP 
* Data on Stash can be accessed by jobs running on compute nodes. 
</div>

