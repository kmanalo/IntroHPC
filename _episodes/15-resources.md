---
title: "Using a cluster: Using resources effectively"
teaching: 15
exercises: 15
questions:
- "How do we monitor our jobs?"
- "How can I get my jobs scheduled more easily?" 
objectives:
- "Understand how to look up job statistics and profile code."
- "Understand job size implications."
- "Be a good person and be nice to other users."
keypoints:
- "The smaller your job, the faster it will schedule."
- "Don't run stuff on the login node."
- "Again, don't run stuff on the login node."
- "Don't be a bad person and run stuff on the login node."
---

We now know virtually everything we need to know about getting stuff on a
cluster.  We can log on, submit different types of jobs, use preinstalled
software, and install and use software of our own.  What we need to do now is
use the systems effectively.

## Estimating required resources using the scheduler

Although we covered requesting resources from the scheduler earlier,
how do we know how much and what type of resources we will need in the first place?

Answer: we don't.  Not until we've tried it ourselves at least once.  We'll
need to benchmark our job and experiment with it before we know how much it
needs in the way of resources.

The most effective way of figuring out how much resources a job needs is to
submit a test job, and then ask the scheduler how many resources it used.  A
good rule of thumb is to ask the scheduler for more time and memory than your
job can use.  This value is typically two to three times what you think your
job will need.

> ## Benchmarking `bowtie2-build`
> Create a job that runs the following command 
> in the same directory as our *Drosophila* reference genome
> from earlier.
> 
> ```
> bowtie2-build Drosophila_melanogaster.BDGP6.dna.toplevel.fa dmel-index
> ```
> {: .bash}
> 
> The `bowtie2-build` command is provided by the `bowtie2` module.
> As a reference, this command could use several gigabytes of memory and up to an hour of compute time, 
> but only 1 cpu in any scenario.
> 
> You'll need to figure out a good amount of resources to ask for for this
> first "test run".  You might also want to have the scheduler email you to
> tell you when the job is done, and provide an efficiency report.
{: .challenge}

Once the job completes, we can look at the logfile for a statement of resources
used (on MARCC this only gets shown if you email yourself about job status).
An example output might look like:

```
Job ID: 31659769
Cluster: marcc
User/Group: username/groupname
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 28
CPU Utilized: 125:18:32
Job Wall-clock time: 02:14:32
Memory Utilized: 77.97 GB
Memory Efficiency: 77.97 of 100.00 GB
```
{: .bash}

* **CPU Utilized** - What is the total CPU time used?
* **Walltime** - How long did the job take?
* **Memory** - Amount of RAM used.
* **Resource Units** - RUs charged against your project for this job.

## Do not run jobs on the login nodes

The example above was a small process that can be run on the login node without
too much disruption. However, it is good practice to avoid running anything
resource intensive on the login nodes. Typically, users do get throttled, and
there is usually a hard limit of 2GB RAM and 2 cores per user.

Here is an example of a job script that would run the process above on a compute node and copy the results back.

```
#SBATCH -J bowtie-dros
#SBATCH --time=02:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=28

set echo
module load bowtie2
mkdir -p /scratch/users/$USER/mytmp
cp Drosophila_melanogaster.BDGP6.dna.toplevel.fa /scratch/users/$USER/mytmp
cd /scratch/users/$USER/mytmp
bowtie2-build Drosophila_melanogaster.BDGP6.dna.toplevel.fa dros-index
cp *.* $SLURM_SUBMIT_DIR

```
{: bash}

This job requests a full node on Blue Crab, which is not needed for this
calculation, but if you are unsure about how much memory or how many processors
your job will require, it is okay to request more than you need. As you run
jobs, you will get more comfortable identifying the amount of resources you
need.

```
cp Drosophila_melanogaster.BDGP6.dna.toplevel.fa /scratch/users/$USER/mytmp
```
{: bash}

Now, we can copy the input file to a scratch location. 

```
cd /scratch/users/$USER/mytmp 
bowtie2-build Drosophila_melanogaster.BDGP6.dna.toplevel.fa dros-index
cp *.* $SLURM_SUBMIT_DIR
```
{: bash}

Finally, we move the job to the compute node and run the software. All the
files will be read and written on the compute node. This makes your job run
faster and keeps the job traffic from impacting the network. In the final line,
we copy back any output files. I used a very general wildcard to copy
everything back, but you can be more specific, based on your job.

A note about memory: Memory (RAM) is allocated based on number of processors
requested per node. For example, if you request 12 ppn on Blue Crab, that is
half the available processors so your job will receive half the available
memory (~64GB).  

## Playing nice in the sandbox

You now have everything you need to run jobs, transfer files, use/install
software, and monitor how many resources your jobs are using.

So here are a couple final words to live by:

* Don't run jobs on the login node, though quick tests are generally fine. 
  A "quick test" is generally anything that uses less than 1GB of memory, and 20 minutes of time.
  Anything larger will be automatically killed by the system. Remember, the login node is to 
  be shared with other users. 

* Compress files before transferring to save file transfer times with large datasets.

* Use a VCS system like git to keep track of your code. Though most systems have some form
  of backup/archival system, you shouldn't rely on it for something as key as your research code.
  The best backup system is one you manage yourself.

* Before submitting a run of jobs, submit one as a test first to make sure everything works.

* The less resources you ask for, the faster your jobs will find a slot in which to run.
  The more accurate your walltime is, the sooner your job will run.

* You can generally install software yourself, but if you want a shared installation of some kind,
  it might be a good idea to email marcc-help@marcc.jhu.edu

* Always use the default compilers if possible. 

