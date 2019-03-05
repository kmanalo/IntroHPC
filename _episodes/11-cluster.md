---
title: "Using a cluster: Introduction"
teaching: 15
exercises: 5
questions:
- "What is a cluster?"
- "How does a cluster work?"
- "How do I log on to a cluster?"
objectives:
- "Connect to a cluster."
- "Understand the general cluster architecture."
keypoints:
- "A cluster is a set of networked machines."
- "Clusters typically provide a login node and a set of worker nodes."
- "Files saved on one node are available everywhere."
---

The words "cloud", "cluster", and "high-performance computing" get thrown
around a lot.  So what do they mean exactly?  And more importantly, how do we
use them for our work?

The "cloud" is a generic term commonly used to refer to remote computing
resources.  Cloud can refer to webservers, remote storage, API endpoints, and
as well as more traditional "raw compute" resources.  A cluster on the other
hand, is a term used to describe a network of compters.  Machines in a cluster
typically share a common purpose, and are used to accomplish tasks that might
otherwise be too substantial for any one machine. 

![The cloud is made of Linux](../files/linux-cloud.jpg)

## High performance computing

A high-performance computing cluster is a set of machines that have been 
designed to handle tasks that normal computers can't handle.
This doesn't always mean simply having super fast processors. 
High-performance computing covers a lot of use cases.
Here are a couple of use cases where high-performance computing becomes extremely useful:

* You need access to large numbers of CPUs.
* You need to run a large number of jobs.
* Your jobs are running out of memory.
* Perhaps you need to store tons and tons of data.
* You require an exceptionally high-bandwidth internet connection for data transfer.
* You need a safe archival site for your data.
* Your compute jobs require specialized GPU or FPGA hardware.
* Maybe your jobs just take a long time to run.

Chances are, you've run into one of these situations before.  Fortunately,
high-performance computing installations exist to solve these types of
problems.

## Where are we? 

So what's really happening? What machine have we logged on to?
The name of the current computer we are logged onto can be checked with the `hostname` command.

MARCC presents a single view of multiple filesystem targets, and users log into a round-robin set of login nodes.  
Users use the login nodes typically to compile codes, transfer files, and access the scheduler to submit jobs to compute nodes.

```
hostname
```
{: .bash}
```
bc-login01
```
{: .output}

Clusters have different types of machines customized for different types of
tasks.  In this case, we are on a login node.  A login node serves as a gateway
to the cluster and serves as a single point of access.  As a gateway, it is
well suited for uploading and downloading files, setting up software, and
running quick tests.  It should never be used for doing actual work.

The real work on a cluster gets done by the compute nodes.
Compute nodes come in many shapes and sizes, but generally are dedicated to doing all of the heavy lifting that needs doing. 
All interaction with the compute nodes is handled by a specialized piece of software called a scheduler. We use the SLURM scheduler.
We can view all of the worker nodes with the `pbsnodes -a` command. But this would be overwhelming since we have over 900 compute nodes, so we'll abbreviate it instead.

```
scontrol show nodes | tail -n 18
```
{: .bash}
```

NodeName=gpudev002 Arch=x86_64 CoresPerSocket=12
   CPUAlloc=2 CPUErr=0 CPUTot=24 CPULoad=1.55
   AvailableFeatures=skylake,v100
   ActiveFeatures=skylake,v100
   Gres=gpu:2
   NodeAddr=gpudev002 NodeHostName=gpudev002 Version=17.11
   OS=Linux 3.10.0-862.9.1.el7.x86_64 #1 SMP Mon Jul 16 16:29:36 UTC 2018
   RealMemory=95307 AllocMem=7100 FreeMem=82349 Sockets=2 Boards=1
   State=MIXED ThreadsPerCore=1 TmpDisk=0 Weight=2040 Owner=N/A MCS_label=N/A
   Partitions=gpuv100
   BootTime=2019-02-05T12:27:50 SlurmdStartTime=2019-02-06T08:49:33
   CfgTRES=cpu=24,mem=95307M,billing=24,gres/gpu=2
   AllocTRES=cpu=2,mem=7100M,gres/gpu=2
   CapWatts=n/a
   CurrentWatts=0 LowestJoules=0 ConsumedJoules=0
   ExtSensorsJoules=n/s ExtSensorsWatts=0 ExtSensorsTemp=n/s

```
{: .output}

There are also specialized machines used for managing disk storage, user
authentication, and other infrastructure-related tasks. 
Although we do not interact with these directly, 
but these enable a number of key features like ensuring our user account and
files are available throughout the cluster.  This is an important point to
remember: files saved on one node (computer) are available everywhere on the
cluster!

![Structure of Cluster](../files/cluster_login.jpg)

This graphic is a general view of the parts of a cluster (image courtesy Ohio
Supercomputer Center). For specific details on the clusters at MARCC, check out
our Cluster System Details:
[https://www.marcc.jhu.edu/cyberinfrastructure/hardware/](https://www.marcc.jhu.edu/cyberinfrastructure/hardware/)
