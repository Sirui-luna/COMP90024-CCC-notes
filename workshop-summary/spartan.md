##Spartan

Logins to Spartan are based on POSIX identity for the system:

```ssh yourusername@spartan.hpc.unimelb.edu.au
```
###Modules Commands

Some basic module commands include the following:

```module help
```The command module help , by itself, provides a list of the switches, subcommands, and subcommand arguments that are available through the environment modules package.

```module avail
```This option lists all the modules which are available to be loaded.

```module whatis <modulefile>
```This option provides a description of the module listed.

```module display <modulefile>
```Use this command to see exactly what a given modulefile will do to your environment, such as what will be added to the PATH, MANPATH, etc. environment variables.

```
module load <modulefile>
```==This adds one or more modulefiles to the user's current environment== (some modulefiles load other modulefiles.)

```module unload <modulefile>
```This removes any listed modules from the user's current environment.

```module switch <modulefile1> <modulefile2>
```This unloads one modulefile (modulefile1) and loads another (modulefile2).

```module purge
```This removes all modules from the user's environment.
In the lmod system as used on Spartan there is also “module spider” which will search for all possible modules and not just those in the existing module path and provide descriptions.

###Slurm
--

Setup and launch consists of writing a short script that initially makes resource requests and then commands, and optionally checking queueing system.* Core command for checking queue: `squeue | less`* Alternative command for checking queue: `showq -p cloud | less`* Core command for job submission: `sbatch [jobscript]`
Check job status (by ID or user), cancel job.
* Core command for checking job in Slurm: `squeue -j [jobid]`* Detailed command in Slurm: `scontrol show job [jobid]`* Core command for deleting job in Slurm: `scancel [jobid]`
Slurm provides an error and output files They may also have files for post-job processing.Graphic visualisation is best done on the desktop.

--
**Script Example**

***Different Architecture:***

_Multinode_:

nodes or N: number of nodes/server
ntasks or n: number of cores
ntasks-per-node = ntasks divide by nodes
time : time limit for running the job (==does it impact priorities ??==)

```
#!/bin/bash#SBATCH -partition=physical#SBATCH --nodes=2
#SBATCH --ntasks=8#SBATCH --ntasks-per-node=4
#SBATCH --time=0-00:15:00module load <my-app-compiler/version>srun -N 2 -n 8 python3 <name of your python file>
```

_Multithreaded_:

> * whereby a master thread forks a number of sub-threads and divides tasks between them. The threads will then run concurrently and are then joined at a subsequent point to resume normal serial application.
> * Implementation: OpenMP (Open Multi-Processing).
> * use thread-passing

```
#SBATCH --cpus-per-task=8
```

***Array & Dependencies***

Array: A typical example is to apply the same task across multiple datasets. The following example submits 10 batch jobs with myapp running against datasets dataset1.csv, dataset2.csv, ...dataset10.csv

```#SBATCH array=1-10myapp ${SLURM_ARRAY_TASK_ID}.csv
```
The dependency directives consist of `after`, `afterok`,`afternotok`, `before`, `beforeok`, `beforenotok`. A typical use case is where the output of one job is required as the input of the next job.

```#SBATCH dependency=afterok:myfirstjobid mysecondjob
```
******

For realtime interaction, with resource requests made on the command line, an interactive job is called. This puts the user on to a compute node.
This is typically done if they user wants to run a large script (and shouldn't do it on the login node), or wants to test or debug a job. The following command would launch one node with two processors for ten minutes.

```[lev@spartan interact]$ sinteractive nodes=1 ntaskspernode=2srun: job 164 queued and waiting for resourcessrun: job 164 has been allocated resources[lev@spartanrc002 interact]$
```

