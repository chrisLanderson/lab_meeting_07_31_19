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
gzip -d contigs.fa.gz
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
# commands:
module load gcc/7.2.0
/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2-build contigs.fa contigs_bt2

# create a new file with your go to text editor
nano bt2_job.sh
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

scancel <job_id> # if you want to cancel the job

ls -lhrt 
```

* Although we covered requesting resources from the scheduler earlier, how do we know how much and what type of resources we will need in the first place?

Answer: we don’t. Not until we’ve tried it ourselves at least once. We’ll need to benchmark our job and experiment with it before we know how much it needs in the way of resources.

The most effective way of figuring out how much resources a job needs is to submit a test job, and then ask the scheduler how many resources it used. A good rule of thumb is to ask the scheduler for more time and memory than your job can use. This value is typically two to three times what you think your job will need.

```bash
sacct -j <job_id> -l

sacct -j <job_id> --format=jobid,jobname,elapsed,state,maxrss,maxvmsize

sacct -u <username> --format=jobid,jobname,elapsed,state,maxrss,maxvmsize
```

* Now that we have created the bowtie2 index, lets submit a job to align one of the fastq files to the contig index - 1 node, 4 cores/threads, 2 GB RAM, and limit the job to 1 hour.

```bash
ls fastq_dir

#commands: 
module load gcc/7.2.0
/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 

nano bt2_job.sh 
vim bt2_job.sh

#example:
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --time=1:00:00
#SBATCH --mem=2000
#SBATCH --partition general

module load gcc/7.2.0
/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S1_R1.fastq -2 fastq_dir/S1_R2.fastq -S S1.sam
```

Submit and monitor the progress of the job. After it is completed, view the stdout and stderr messages. Check the resources used for the job so we can better guide our resource allocation for the remainder of the jobs.

```bash
squeue -u <username>

sacct -j <job_id> --format=jobid,jobname,elapsed,state,maxrss,maxvmsize

```


* How can we run bowtie2 for all 8 samples?

* Is this the best approach?

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --time=1:00:00
#SBATCH --mem=500
#SBATCH --partition general

module load gcc/7.2.0

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S1_R1.fastq -2 fastq_dir/S1_R2.fastq -S S1.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S2_R1.fastq -2 fastq_dir/S_R2.fastq -S S2.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S3_R1.fastq -2 fastq_dir/S3_R2.fastq -S S3.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S4_R1.fastq -2 fastq_dir/S4_R2.fastq -S S4.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S5_R1.fastq -2 fastq_dir/S5_R2.fastq -S S5.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S6_R1.fastq -2 fastq_dir/S6_R2.fastq -S S6.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S7_R1.fastq -2 fastq_dir/S7_R2.fastq -S S7.sam

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/S8_R1.fastq -2 fastq_dir/S8_R2.fastq -S S8.sam

```

How would you write a loop to do the above?

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --time=1:00:00
#SBATCH --mem=500
#SBATCH --partition general

module load gcc/7.2.0

for r1 in fastq_dir/*R1.fastq; do
  sample_id="$(basename $r1 | cut -d "_" -f 1)"
  /home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/"$sample_id"_R1.fastq -2 fastq_dir/"$sample_id"_R2.fastq -S "$sample_id".sam 
done

```

Given how short this mapping job is, a loop is likley the correct decision. However, for long jobs where we want to iterate over something, like files or genomes, and perform a common command, we could consider job arrays.

A job array is a collection of similar independent jobs which are submitted together to one of the Linux cluster job schedulers using a job template script. The advantage of using a job array is that many similar jobs can be submitted using a single job template script, and the jobs will run independently as they are able to obtain resources on the compute cluster.

The $SLURM_ARRAY_TASK_ID variable  is incremented for each job in the array.

The number of jobs in the array is indiciated with the --array parameter in the batch script.


```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --time=1:00:00
#SBATCH --mem=500
#SBATCH --partition general
#SBATCH --array=1-8

r1="$(ls fastq_dir/*R1.fastq | head -n $SLURM_ARRAY_TASK_ID | tail -n 1)"

sample_id="$(basename $r1 | cut -d "_" -f 1)"

/home/li.gua/usr/bowtie2/2.3.5.1/bin/bowtie2 --threads 4 -x contigs_bt2 -1 fastq_dir/"$sample_id"_R1.fastq -2 fastq_dir/"$sample_id"_R2.fastq -S "$sample_id".sam 

```

