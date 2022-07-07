##Spartan

Logins to Spartan are based on POSIX identity for the system:

```
```
###Modules Commands

Some basic module commands include the following:

```
```

```
```

```
```

```
```

```
module load <modulefile>
```

```
```

```
```

```
```


###Slurm
--

Setup and launch consists of writing a short script that initially makes resource requests and then commands, and optionally checking queueing system.




--
**Script Example**

***Different Architecture:***

_Multinode_:

nodes or N: number of nodes/server
ntasks or n: number of cores
ntasks-per-node = ntasks divide by nodes
time : time limit for running the job (==does it impact priorities ??==)

```
#!/bin/bash
#SBATCH --ntasks=8
#SBATCH --time=0-00:15:00
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

```
```
The dependency directives consist of `after`, `afterok`,

```
```
******

For realtime interaction, with resource requests made on the command line, an interactive job is called. This puts the user on to a compute node.


```
```

