---
title: "Using a cluster: Accessing software"
teaching: 20
exercises: 15
questions:
- "How do we load and unload software packages?"
objectives:
- "Understand how to load and use a software package."
keypoints:
- "Load software with `module load softwareName`"
- "The module system handles software versioning and package conflicts for you automatically."
- "You can edit your `.bashrc` file to automatically load a software package."
- "When using software on any HPC system, check the software documents for details on how to use it effectively."
---

On a high-performance computing system, no software is loaded by default.
If we want to use a software package, we will need to "load" it ourselves.

Before we start using individual software packages, however, 
we should understand the reasoning behind this approach.
The two biggest factors are software incompatibilities and versioning.

Software incompatibility is a major headache for programmers.  Sometimes the
presence (or absence) of a software package will break others that depend on
it.  Two of the most famous examples are Python 2 and 3 and C compiler
versions.  Python 3 famously provides a `python` command that conflicts with
that provided by Python 2.  Software compiled against a newer version of the C
libraries and then used when they are not present will result in a nasty
`'GLIBCXX_3.4.20' not found` error, for instance. 

Software versioning is another common issue.  A team might depend on a certain
package version for their research project - if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions allow a set of researchers to take
software version out of the equation if results are weird.

## Environment modules (Lmod)

Environment modules are the solution to these problems.
A module is a self-contained software package - 
it contains all of the files required to run a software packace 
and loads required dependencies.

To see available software modules, use `module avail`

```
module avail
```
{: .bash}
```

MARCC - Blue Crab Cluster - CentOS 7
Search for software with 'module spider'

------------------------------------------------------------ Core Modules -------------------------------------------------------------
   MARCC/summer-2018 (L)    cuda/9.0         gcc/5.5.0  (D)    intel/18.0    (L,D)    openmpi/3.0.0          openmpi/3.1.4    use.own
   centos7/current   (L)    cuda/9.2  (D)    gcc/6.4.0         intelmpi/2018          openmpi/3.1   (L,D)    openmpi/4.0.0
   cuda/8.0                 gcc/4.8.5        intel/17.0        mvapich2/2.3           openmpi/3.1.3          pgi/18.10

-------------------------------------------------------------- Base Apps --------------------------------------------------------------
   R/3.4.2-velox            (c)    eog/3.20.5               julia/0.6.3           (c)      python/3.6
   abaqus/6.14                     fastqc/0.11.7            julia/0.7.0           (c)      python/3.7                      (D)
   abaqus/2016              (D)    fastqtl/2.184            julia/1.0.1           (c)      pytorch/0.3.1-gpu-py3
   aspera/3.7                      ffmpeg/18.09             julia/1.0.2           (c)      pytorch/0.4.1-gpu-py3           (D)
   atlas/3.10.3                    freesurfer/6.0.0         julia/1.0.3           (D)      qctool/2.0.1
   augustus/3.3             (c)    fsl/5.0.11               kraken/1.1                     qiime2/2018.8
   bbmap/38.23                     gatk/4.0.0        (c)    lammps/2016-ICMS               qt5/5.12.1
   bcl2fastq2/2.20                 gaussian/g09             ls-dyna/10.1.0                 rnaseqc/1.18
   bedtools/2.27.0                 gaussian16/g16           mathematica/11.3               rsem/1.3.1
   bioperl/1.7.2                   gaussview/5.0            matlab/R2016a                  ruby/2.5.1
   biopython/1.72-py2              gdal/2.2.3               matlab/R2017a                  sbank/1.1.1.2
   biopython/1.72-py3       (D)    gdal/2.3.1               matlab/R2018a         (D)      settarg/7.7.14
   blast/2.7.1                     gdal/2.4.0        (D)    moby/2.0              (c)      singularity/2.6.0
   blender/2.79b                   gdc-client/1.3.0         moby/2.0.1            (c,D)    snp-sites/2.4.1                 (c)
   bowtie/1.1.1                    gedit/3.22.0             openbabel/2.4.1                spm/12
   bwa/0.7.17                      geos/3.8.0               opensees/3.0.0        (c)      sra-tools/2.9.2                 (c)
   cdhit/4.6.8.1                   giggle/0.6.3             orca/3.0.3                     star/2.4.2a
   cdo/1.9.5                (c)    git/2.9.4                orca/4.0.1.2                   subread/1.6.3
   cellranger/2.2.0                git/2.18.0        (D)    orca/4.1              (D)      tecplot360ex/2015.R2
   cmake/3.8.2                     globus-cli/1.7.0         ovito/3.0.0           (c)      tensorflow/1.10.1-gpu-py3-keras (g,c)
   cmake/3.12.1             (D)    gnuplot/4.6.2            parallel/20180822              tensorflow/1.10.1-gpu-py3       (g,c)
   comsol/5.4                      gsl/2.5                  picard/2.18                    tensorflow/1.10.1-gpu           (g,c,D)
   connectome_workbench/1.0        gurobi/702               plink/1.90b6.4                 tophat/2.1.1
   cpanminus/latest                gv/gv                    povray/3.7                     trim_galore/0.5.0
   cpu_features/latest             hail/0.1                 proj/5.1.0                     trimmomatic/0.38
   cufflinks/2.2.1                 haskell/7.6.3            proot/5.1.0                    vcf2db/2018.10.26               (c)
   cutadapt/1.18                   hisat2/2.1.0             python/2.7-anaconda            vcftools/0.1.17
   deeptools/3.1.2                 hwloc/1.11               python/2.7-anaconda53          vim/8.1
   dos2unix/7.2.3                  imagemagick/6.7.8        python/2.7                     vmd/1.93
   emacs/26.1                      java/1.8.0_181           python/3.6-anaconda            xpdf/3.0.4

---------------------------------------------------- Intel Compiler Dependent Apps ----------------------------------------------------
   R/3.4.0                        hdf5/1.10                 meshalyzer/99a3a10                 pflotran/2018.10
   R/3.5.1                 (D)    hdf5/1.10.4        (D)    moby2-deps/0.1                     quantumespresso/6.2.1
   bamtools/2.5.1                 htslib/1.9                namd/2.12-ibverbs-smp-cuda         quantumespresso/6.8   (D)
   bcftools/1.9                   hypre/2.11.2              namd/2.12-ibverbs                  samtools/0.1.19
   boost/1.68.0                   jags/4.3.0                namd/2.12-mpi                      samtools/1.9          (D)
   bowtie2/2.3.4.2                jsoncpp/1.8.4             namd/2.13b-ibverbs-smp-cuda        stringtie/1.3.4d
   carp/1.8.0                     lammps/20180822    (D)    namd/2.13b2-ibverbs         (D)    swig/3.0.12
   charmm/42                      lapack/3.8.0              netcdf-fortran/4.4.4               velvet/1.2.10
   fftw3/3.3.8                    libdrm/2.4.83             netcdf/4.6.1                       xcb-proto/1.13
   gcc-compatibility/5.5.0 (D)    libxshmfence/1.2.1        petsc/3.6.1
   gcc-compatibility/6.4.0        meme/5.0.3                petsc/3.9.3                 (D)

  Where:
   g:  Built for GPU
   L:  Module is loaded
   c:  Singularity Container
   D:  Default Module
```
{: .output}

## Loading and unloading software

To load a software module, use `module load`.
In this example we will use Python 3.

Initially, a default Python 3 is loaded but it's the system installed version. Perhaps you may want a newer one.

```
which python3
```
{: .bash}
```
/software/centos7/bin/python3
```
{: .output}

We can load the `python/3.6` software with `module load`.

```
module load python/3.6
which python3
```
{: .bash}
```
/software/apps/python/3.6/bin/python3
```
{: .output}

So what just happened?

To understand the output, first we need to understand the nature of the 
`$PATH` environment variable.
`$PATH` is a special ennvironment variable that controls where a UNIX system looks for software.
Specifically `$PATH` is a list of directories (separated by `:`)
that the OS searches through for a command before giving up and telling us it can't find it.
As with all environment variables we can print it out using `echo`.

```
echo $PATH
```
{: .bash}
```
/software/apps/python/3.6/bin:/software/apps/marcc/bin:/software/centos7/bin:/software/apps/slurm/current/sbin:/software/apps/slurm/current/bin:/home-0/kmanalo/perl5/bin:/software/apps/mpi/openmpi/3.1/intel/18.0/bin:/software/apps/compilers/intel/itac/2018.3.022/intel64/bin:/software/apps/compilers/intel/clck/2018.3/bin/intel64:/software/apps/compilers/intel/compilers_and_libraries_2018.3.222/linux/bin/intel64:/software/apps/compilers/intel/compilers_and_libraries_2018.3.222/linux/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/software/apps/compilers/intel/parallel_studio_xe_2018.3.051/bin:/home-0/kmanalo/bin
```
{: .output}

You'll notice a similarity to the output of the `which` command. 
In this case, there's only one difference:
the `/software/apps/python/3.6/bin` directory at the beginning.
When we ran `module load python/3.5.2`, 
it added this directory to the beginning of our `$PATH`.
Let's examine what's there:

```
ls /software/apps/python/3.6/bin
```
{: .bash}
```
2to3      easy_install-3.6  idle3.6  pip3.6  pydoc3.6  python3.6         python3.6m         python3-config  pyvenv-3.6
2to3-3.6  idle3             pip3     pydoc3  python3   python3.6-config  python3.6m-config  pyvenv
```
{: .output}

`module load` will add software to your `$PATH`.  It "loads" software.
A special note on this - 
depending on which version of the `module` program that is installed at your site,
`module load` will also load required software dependencies.
To demonstrate, let's use `module list`.
`module list` shows all loaded software modules.

```
module list
```
{: .bash}
```

Currently Loaded Modules:
  1) intel/18.0   2) openmpi/3.1   3) MARCC/summer-2018   4) centos7/current   5) python/3.6
  
```
{: .output}



```
module unload python/3.6
module list
```
{: .bash}
```

Currently Loaded Modules:
  1) intel/18.0   2) openmpi/3.1   3) MARCC/summer-2018   4) centos7/current

```
{: .output}

So using `module unload` "un-loads" a module along with it's dependencies.
If we wanted to unload everything at once, we could run `module purge` (unloads everything).

At MARCC, we recommend keeping the MARCC meta-module loaded after a purge, `module purge; module load MARCC`
This takes us back to the default recommendations.

## Software versioning

So far, we've learned how to load and unload software packages.
But let's examine the output of `module avail` more closely.

```
module avail
```
{: .bash}
```

MARCC - Blue Crab Cluster - CentOS 7
Search for software with 'module spider'

------------------------------------------------------------ Core Modules -------------------------------------------------------------
   MARCC/summer-2018 (L)    cuda/9.0         gcc/5.5.0  (D)    intel/18.0    (L,D)    openmpi/3.0.0          openmpi/3.1.4    use.own
   centos7/current   (L)    cuda/9.2  (D)    gcc/6.4.0         intelmpi/2018          openmpi/3.1   (L,D)    openmpi/4.0.0
   cuda/8.0                 gcc/4.8.5        intel/17.0        mvapich2/2.3           openmpi/3.1.3          pgi/18.10

-------------------------------------------------------------- Base Apps --------------------------------------------------------------
   R/3.4.2-velox            (c)    eog/3.20.5               julia/0.6.3           (c)      python/3.6
   abaqus/6.14                     fastqc/0.11.7            julia/0.7.0           (c)      python/3.7                      (D)
   abaqus/2016              (D)    fastqtl/2.184            julia/1.0.1           (c)      pytorch/0.3.1-gpu-py3
   aspera/3.7                      ffmpeg/18.09             julia/1.0.2           (c)      pytorch/0.4.1-gpu-py3           (D)
   atlas/3.10.3                    freesurfer/6.0.0         julia/1.0.3           (D)      qctool/2.0.1
   augustus/3.3             (c)    fsl/5.0.11               kraken/1.1                     qiime2/2018.8
   bbmap/38.23                     gatk/4.0.0        (c)    lammps/2016-ICMS               qt5/5.12.1
```
{: .output}

Software with multiple versions will be marked with a (D).  Please engage with us at marcc-help@marcc.jhu.edu to help us shape software needs.

> ## Using software modules in scripts
>
> Create a job that is able to run `python3 --version`.
> (you should not assume a module loaded on the login node is loaded on a compute node).
{: .challenge}

> ## Loading a module by default
> 
> Adding a set of `module load` commands to all of your scripts and
> having to manually load modules every time you log on can be tiresome.
> Fortunately, there is a way of specifying a set of "default modules"
> that always get loaded, regardless of whether or not you're logged on or running a job.
>
> Every user has two hidden files in their home directory: `.bashrc` and `.bash_profile`
> (you can see these files with `ls -la ~`).
> These scripts are run every time you log on or run a job.
> Adding a `module load` command to one of these shell scripts means that
> that module will always be loaded.
> Modify either your `.bashrc` or `.bash_profile` scripts to load a commonly used module like Python.
> Does your `python3 --version` job from before still need `module load` to run?
{: .challenge}

## Installing software of our own

>The following tutorial is from our HOWTO guide on our website. While we are MARCC, OSC has a great set of tutorial materials,
>please have a look at [https://www.osc.edu/resources/getting_started/howto](https://www.osc.edu/resources/getting_started/howto)
>
{: .callout}

Most HPC clusters have a pretty large set of preinstalled software.
Nonetheless, it's unlikely that all of the software we'll need will be available.
Sooner or later, we'll need to install some software of our own. 

Though software installation differs from package to package, the general
process is the same: download the software, read the installation instructions
(important!), install dependencies, compile, then start using our software.

### Getting Started
Before installing your software, you should first prepare a place for it to live. We recommend the following directory structure, which you should create in the top-level of your home directory:

```
    local
    |-- src
    |-- share
        `-- lmodfiles
```
{: .bash}

This structure is one way to organizes software. Each directory serves a specific purpose:

| directory | description |
| --- | --- |
|local  |Gathers all the files related to your local installs into one directory, rather than cluttering your home directory. Applications will be installed into this directory with the format "appname/version". This allows you to easily store multiple versions of a particular software install if necessary.|
|local/src | Stores the installers -- generally source directories -- for your software. Also, stores the compressed archives ("tarballs") of your installers; useful if you want to reinstall later using different build options.|
|local/share/lmodfiles  |The standard place to store module files, which will allow you to dynamically add or remove locally installed applications from your environment.|

You can create this structure with one command.

NOTE: Ensure $HOME is the full path of your home directory. You can identify this from the command line with the command echo $HOME.

After navigating to where you want to create the directory structure, run:

```
    mkdir -p $HOME/local/src $HOME/local/share/lmodfiles
```
{: .bash}


### Installing Software
Now that you have your directory structure created, you can install your software. For demonstration purposes, we will install a local copy of Git.

First, we need to get the source code onto the HPC filesystem. The easiest thing to do is find a download link, copy it, and use the wget tool to download it on the HPC. We'll download this into $HOME/local/src:

```
    cd $HOME/local/src
    wget https://github.com/git/git/archive/v2.9.0.tar.gz
```
{: .bash}


Now extract the tar file:

```
    tar zxvf v2.9.0.tar.gz
```
{: .bash}

Next, we'll go into the source directory and build the program. Consult your application's documentation to determine how to install into `$HOME/local/"software_name"/"version"`. Replace "software_name" with the software's name and "version" with the version you are installing, as demonstrated below. In this case, we'll use the configure tool's --prefix option to specify the install location.

You'll also want to specify a few variables to help make your application more compatible with our systems. We recommend specifying that you wish to use the Intel compilers and that you want to link the Intel libraries statically. This will prevent you from having to have the Intel module loaded in order to use your program. To accomplish this, add `CC=icc CFLAGS=-static-intel` to the end of your invocation of configure. If your application does not use configure, you can generally still set these variables somewhere in its Makefile or build script.

Then, we can build Git using the following commands:

```
    cd git-2.9.0
    autoconf # this creates the configure file
    ./configure --prefix=$HOME/local/git/2.9.0 CC=icc CFLAGS=-static-intel
    make && make install
```
{: .bash}

We've successfully installed our first piece of software!

See the HOWTO for details on creating your own modules for locally installed software.
