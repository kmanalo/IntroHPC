---
title: "Using a cluster: Scheduling jobs"
teaching: 45
exercises: 15
questions:
- "What is a scheduler and why are they used?"
- "How do we submit a job?"
objectives:
- "Submit a job and have it complete successfully."
- "Understand how to make resource requests."
- "Submit an interactive job."
keypoints:
- "The scheduler handles how compute resources are shared between users."
- "Everything you do should be run through the scheduler."
- "A job is just a shell script."
- "If in doubt, request more resources than you will need."
---

An HPC system might have thousands of nodes and thousands of users.  How do we
decide who gets what and when?  How do we ensure that a task is run with the
resources it needs?  This job is handled by a special piece of software called
the scheduler.  On an HPC system, the scheduler manages which jobs run where
and when.

The scheduler used in this lesson is Slurm.  Although Slurm is not used
everywhere, running jobs is quite similar regardless of what software is being
used.  The exact syntax might change, but the concepts remain the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively.
This is also referred to as batch job submission.
In this case, we need to make a script that incorporates some arguments for SLURM such as resources needed and modules to load. 

Create a file in a `test_jobs directory`. 

```
test.job
```
```
#!/bin/bash -l
#SBATCH --partition=quickdebug
#SBATCH --job-name=test_job
#SBATCH --time=00:10:00
#SBATCH --nodes=1
#SBATCH --ntasks=1

echo 'This script is running on:'
hostname
echo 'The date is :'
date
sleep 120
```
We will talk about the parameters set above and their defaults later in this
lesson.

As this is just a bash script, you can run it as any bash script on the head
node. Please note- its very very important not to run any large or resource
heavy script on the head node, as every user relies on this head node.

To test a small SLURM script, run it as:
```
bash test.job
# alternatively you can 
# chmod +x test.job
# ./test.job
This script is running on:
bc-login01
The date is :
Mon Mar  4 12:23:40 EST 2019
```
This script will take 2 minutes to finish due to the `sleep 120` command.

There is a distinction between running the job through the scheduler and just
"running it".  To submit this job to the scheduler, we use the `sbatch`
command.

```
sbatch test.job
Submitted batch job 32962120
```

The number that first appears is your Job ID. When the job is completed, you will get a file that combines both "Output" (stdout) and "Error" (stderr). There are options in Slurm to split `stdout` and `stderr` file streams, they will be named {JobName}.o{JobID} and {JobName}.e{JobID} respectively.

And that's all we need to do to submit a job.  To check on our job's status, we
use the command `squeue`, and an option to filter out your own user.

```
~> squeue -u $USER # this usually is set to your username
Mon Mar  4 23:06:27 2019
              USER    JOBID PARTITION      NAME NODES  CPUS PRIORITY TIME_LIMIT     TIME        NODELIST ST REASON
           kmanalo 32962305 quickdebug test_job     1     1    11290      10:00     0:00     compute0678 R None
```

{: .output}

We can see all the details of our job, most importantly if it is in the "R" or
running state.  Sometimes our jobs might need to wait in a queue ("PD") or have
an error.  The best way to check our job's status is with `squeue`. It is
easiest to view just your own jobs in the queue with the `squeue -u username`
or using the MARCC alias `sqme`.  Otherwise, you get the entire queue.

### Parameters

Let's discuss the example SLURM script.
```
test.job
```
```
#!/bin/bash -l
#SBATCH --partition=quickdebug
#SBATCH --job-name=test_job
#SBATCH --time=00:10:00
#SBATCH --nodes=1
#SBATCH --ntasks=1

echo 'This script is running on:'
hostname
echo 'The date is :'
date
sleep 120
```

Comments in UNIX (denoted by `#`) are typically ignored.  But there are
exceptions.  For instance the special `#!` comment at the beginning of scripts
specifies what program should be used to run it (typically `/bin/bash`).  The
option '-l' means behave as if logging in Schedulers like Slurm also have a
special comment used to denote special scheduler-specific options.  Though
these comments differ from scheduler to scheduler, Slurm's special comment is
`#SBATCH`.  Anything following the `#SBATCH` comment is interpreted as an
instruction to the scheduler.

In our example, we have set the following parameters:
 
| Flag | Name | Example Setting | Notes|
| --- | --- | --- | --- |
| --partition | queue | quickdebug | See next section for queue info |
| --job-name | jobname| test_job | Name of your script (no spaces, alphanumeric only) |
| --nodes | number of nodes | 1 | |
| --ntasks | number of tasks | 4 | MPI tasks or CPU cores |

### Resource list
Resource list will contain a number of settings that informs the scheduler what
resources to allocate for your job and for how long (walltime).

#### Walltime
Walltime is represented by `time=00:01:01` in the format HH:MM:SS. This will be how long the job will run before timing out.  If your job exceeds this time the scheduler will terminate the job. It is recommended to find a usual runtime for the job and add some more (say 20%) to it. For example, if a job took approximately 10 hours, the walltime limit could be set to 12 hours, e.g. "-l walltime=12:00:00". By setting the walltime the scheduler can perform job scheduling more efficiently and also reduces occasions where errors can leave the job stalled but still taking up resource for the default much longer walltime limit (for queue walltime defaults run "qstat -q" command)

Resource requests are typically binding.  If you exceed them, your job will be
killed.  Let's use walltime as an example.  We will request 60 seconds of
walltime, and attempt to run a job for two minutes.

```
#!/bin/bash -l
#SBATCH --partition=quickdebug
#SBATCH --job-name=test_job
#SBATCH --time=00:00:60  ## <- altered to 60 seconds
#SBATCH --nodes=1
#SBATCH --ntasks=1

echo 'This script is running on:'
hostname
echo 'The date is :'
date
sleep 120
```

Submit the job and wait for it to finish.  Once it is has finished, check the
slurm-jobid.out file. In the log file, there will be
```
slurmstepd: error: *** JOB 32962322 ON compute0678 CANCELLED AT 2019-03-04T23:18:42 DUE TO TIME LIMIT ***
```

Our job was killed for exceeding the amount of resources it requested.
Although this appears harsh, this is actually a feature.  Strict adherence to
resource requests allows the scheduler to find the best possible placement for
jobs.  Even more importantly, it ensures that another user cannot use more
resources than specified.  

You can find out more information about these parameters by viewing the manual
page of the `sbatch` function. 

```
man sbatch
```

## Partitions or Queues

There are usually a number of available queues to use on your HPC. Remember:
Each cluster has separate queues. Right now, we are looking only at the queues
on MARCC Blue Crab. The other clusters have similar queues but they are not the same.  To
see what queues are available, you can use the command `sinfo -s`.  It is
important to specify the appropriate partition to make the most econonmical use
of your allocation and to prevent underutilization of resources.

```
~> sinfo -s 
sinfo -s
PARTITION  AVAIL  TIMELIMIT   NODES(A/I/O/T)  NODELIST
shared*       up 3-00:00:00     227/0/20/247  compute[0001-0232,0742-0756]
parallel      up 3-00:00:00    401/42/54/497  compute[0233-0666,0679-0741]
gpuk80        up 2-00:00:00        49/9/9/67  gpu[006-072]
gpup100       up 2-00:00:00          4/0/1/5  gpu[073-077]
gpuv100       up 2-00:00:00          1/0/1/2  gpudev[001-002]
lrgmem        up 3-00:00:00       24/1/20/45  bigmem[0006-0050]
debug         up    2:00:00          1/6/1/8  compute[0672-0676],gpu[001-003]
quickdebug    up    2:00:00          0/2/0/2  compute[0677-0678]
unlimited     up   infinite         9/0/3/12  bigmem[0001-0005],compute[0667-0671],gpu[004-005]
scavenger     up    6:00:00    617/40/94/751  bigmem[0001-0050],compute[0001-0671,0679-0684,0721-0744]
```
{: .bash}


> ## Submitting resource requests
>
> Submit a job that will use 1 node, 14 processors, and 5 minutes of walltime.
{: .challenge}


## Job environment variables

Slurm sets multiple environment variables at submission time. The following Slurm variables are commonly used in command files: 

| Variable Name          |  Description |
|---                     |---|
| SLURM_ARRAY_TASK_ID    |  Array ID numbers for jobs submitted with the --array flag. For example a job submitted with #SBATCH --array 1-8 will run eight identical copies of the shell script. The value of the SLURM_ARRAY_TASK_ID will be an integer between 1 and 8.|
| SLURM_JOB_ID           |  Full jobid assigned to this job. Often used to uniquely name output files for this job, for example: mpiexec -n 16 ./a.out >output.${SLURM_JOB_ID}|
| SLURM_JOB_NAME         |  Name of the job. This can be set using the -J option in the Slurm script (or from the command line). The default job name is the name of the script.|
| SLURM_NODELIST         |  Contains a list of the nodes assigned to the job. If multiple CPUs on a node have been assigned, the node will be listed in the file more than once. By default, mpirun assigns jobs to nodes in the order they are listed in this file |
| SLURM_SUBMIT_HOST      |  The name of the host upon which the qsub command is running.|
| SLURM_JOB_PARTITION    |  Partition job was submitted to.|
| SLURM_SUBMIT_DIR       |  Slurm sets the environment variable to the directory from which the batch job was submitted 

> ## Quick Reference
>A good reference for these and other Slurm variables is part of our [Queueing System (Slurm) - MARCC](https://www.marcc.jhu.edu/getting-started/running-jobs/).
{: .callout}

## Canceling a job

Sometimes we'll make a mistake and need to cancel a job.  This can be done with
the `scancel` command.  Let's submit a job and then cancel it using its job
number.

```
~> sbatch test.job

~> sqme
Mon Mar  4 13:30:27 2019
              USER    JOBID PARTITION      NAME NODES  CPUS PRIORITY TIME_LIMIT     TIME        NODELIST ST REASON
           kmanalo 32962473 quickdebug test_job     1     1    11290       1:00     0:00     compute0678 R None

```

Now cancel the job with it's job number.  Absence of any job info indicates
that the job has been successfully canceled.

```
> scancel 32962473
> sqme
>
```
### Interactive jobs

Sometimes, you will need a lot of resource for interactive use.  Perhaps it's
the first time running an analysis or we are attempting to debug something that
went wrong with a previous job.  At MARCC we provide a convenient command called
`interact`: running it alone provides the help options.

```
interact -p debug -t2:0:0 -N1 -n4 # 4 tasks on 1 node for 2 hours on the debug partition
```
{: .bash}
