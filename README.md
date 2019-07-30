# Lab Meeting - More on Working with the NEU Discovery Cluster

*Below is more of an outline for myself that follows some relevant ideas from HPC Carpentry materials: https://hpc-carpentry.github.io/ - good for independent study*



* Follow up and thoughts from last week's material? Struggles this week working with the cluster?

* Login to the cluster and navigate to your scratch directory

* Refresher from last week: Start an interactive job on the interactive partition with 1 node, 1 core, 1GB RAM, and a time limit of 3 hours
<!---
srun -N 1 -c 1 --mem 1G -t 3:00:00 -p interactive --pty /bin/bash
-->

* Once the job has started, clone a github repository with files we will use today:
```bash
git clone https://github.com/chrisLanderson/lab_meeting_07_31_19
cd lab_meeting_07_31_19
```

* Everyone comfortable transferring data to and from the cluster? Remember: username@**xfer**.discovery.neu.edu 

* There is not very much software made available through the cluster, but lets look at how to see what software is available and how to load it. You might not be doing this often, as we are going to have our group's software installed elsewhere - Do we need to discuss this now? - but its good to be aware of:
```bash
module avail

module list

module load bwa/0.7.17

module list

which bwa

module unload bwa/0.7.17

module list
```

* Lets try submitting a job to the general partiion. There are 8 fastq files in the fastq_files directory and the file contigs.fa in the current directory. We are going to map reads from the 8 fastq files to the contigs with bowtie2.

First, we need to create a bowtie index of the genome. We are going to use the software avaialble in Guangyu's home directory for now. Lets create a batch script to submit this as a job to the cluster. You can use nano, vim, or another text editor. Lets create a job with 1 node, 1 cpu core, 4 GB of RAM, and runs for 1 hour. An example script is below, but try to complete this without looking.

```bash
vim bt2_job.sh

# man sbatch or sbatch -h if you forget the parameters

# example:
#!/bin/bash
#SBATCH -N 1
#SBATCH --cpus-per-task=1
#SBATCH --time=1:00:00
#SBATCH --mem=4000
#SBATCH -p general

module load gcc/7.2.0
/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2-build contigs.fa contigs_bt2
```

After creating a batch script, submit the job with sbatch, look at the progress of the job, and print the stdout and stderr from the job:

```bash
sbatch bt2_job.sh

squeue -u <username>

ls -lhrt 
```

* Although we covered requesting resources from the scheduler earlier, how do we know how much and what type of resources we will need in the first place?

Answer: we don’t. Not until we’ve tried it ourselves at least once. We’ll need to benchmark our job and experiment with it before we know how much it needs in the way of resources.

The most effective way of figuring out how much resources a job needs is to submit a test job, and then ask the scheduler how many resources it used. A good rule of thumb is to ask the scheduler for more time and memory than your job can use. This value is typically two to three times what you think your job will need.

```bash
sacct -j <job_id> -l

sacct -j <job_id> --format=jobid,jobname,maxrss,elapsed,state

sacct -u <username> --format=jobid,jobname,maxrss,elapsed,state
```



