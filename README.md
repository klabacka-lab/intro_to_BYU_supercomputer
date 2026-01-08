# Genomics Pipeline Intro

An introduction to the CHPC - specifically, logging onto a cluster and running scripts on compute nodes using the slurm workload manager.

---

# Contents

-   [Objectives](#objectives)
-   [Getting Set Up](#getting-set-up)
-   [Genomic Filetypes](#genomic-filetypes)
-   [Basic Processing Steps](#basic-processing-steps)
-   [Exercise](#exercise)

---

# <a name="objectives"></a>
# Objectives 

- Logon to a CHPC cluster
- Explore the login node
- Submit a batch script

---

# <a name="getting-set-up"></a>
# Getting set up
If you don't have an account with the BYU Office of Research Computing, you will need to create one here:
[BYU Office of Research Computing](https://rc.byu.edu/)
You will need to enable two-factor authentication to login to your account (on the website and onto the cluster). This will be independent from your other two-factor authentication accounts.

---

# <a name="introduction"></a>
# Introduction
The BYU Supercomputer is owned and operated by the BYU Office of Research Computing. This guide will help you login to the supercomputer and learn some of the basic commands for running scripts.

# <a name="logging-on"></a>
# Logging on
```ssh``` stands for 'secure shell'. By using ssh, you establish a secure connection between you and a remote computer. To use ssh for logging into the CHPC, do the following:

```
ssh <your-netid>@ssh.rc.byu.edu
```

This should prompt you to insert your password. This is your password for your account with the BYU Office of Research Computing (not your general BYU profile). This will take you to your personal directory on the login node for the cluster. Look at your location using the following command:

```
pwd
```

You'll see that you are within a directory named after your netid. This is your home directory on the login node. Any time you use the '~' alias, you'll see that it is referring to your home directory. Try this out by moving to the root directory and then move back to your home directory:

```
cd /
pwd
cd ~
pwd
```

***IMPORTANT:*** You should only read/create/edit files within your home directory. You shouldn't have access to anything else, but just-in-case I felt I should mention this here.

Within your home directory you can create your own environment with directories and script files. Create a directory within your home directory called 'LonePeakLearner'.

```
mkdir -p ~/LizardLover
```

Now the directory 'LizardLover' is within your home directory. Check it out (make sure you are in your home directory when you do this)

```
ls
```

Now create a bash script within the 'LizardLover' directory called 'practice_script.sh' that writes the text 'Hello World' to a text file called 'output.txt' (you can do this using a linux terminal text editor [such as vim] or you can use ```echo``` to create it with one line)

```
cd ~/LizardLover
echo 'echo "hello world\n" >> output.txt' > practice_script.sh
```

You can then run this script as you would on your local computer, but you are doing it on the cluster!

```
bash practice_script.sh
```

***IMPORTANT:*** You should never run demanding commands from the login node, regardless of the directory you are in. The login node is for editing files, submitting jobs, and analyzing output. Very inexpensive computation is okay (e.g., installing software packages is usually okay), but be careful. If you notice a job is taking a long time to run, you should kill it (using ```cmd+c```). Any computation that requires lots of time/power should be conducted on the compute nodes- not the login nodes. The compute nodes are introduced below. 

The Supercomputer is a cluster of nodes that include the user login node (where your home directory is located) and compute nodes for high performance computing. These can all be seen on the [Supercomputer Website]([https://chpc.utah.edu/documentation/gettingstarted.php](https://rc.byu.edu/documentation/resources)).


# <a name="creating-a-batch-script"></a>
# Creating a Batch Script

As stated above, when you login to the supercomputer, you land at the login node. On this node, you should never run programs that are computationally intensive (i.e., require lots of power or long run times). The set cutoff is 15 minutes- and a command takes longer than this, you will be penalized with a 30-minute 'time out' (seriously, you won't be able to submit anything on the cluster for 30 minutes).

You might be asking yourself, "They why the heck are we using the supercomputer if we can't run big jobs? That doesn't sound very super!" Computationally intensive jobs will be submitted from your login node to a compute node. The software on the cluster that enables these submissions is called 'slurm'. Slurm is a workload manager used for job submission on HPC clusters so that multiple people can access compute nodes for their high-intensity computing needs.

Move to the 'LizardLover' directory you created earlier.

```
cd ~/LizardLover
```

You should have a bash script here called 'practice_script.sh' that appends 'hello world\n' to a an output file. You can run this script on a compute node by converting your simple script to a batch script as described below. For more detailed instructions, see the [ORC slurm page]([https://chpc.utah.edu/documentation/software/slurm.php#submit](https://rc.byu.edu/wiki/?id=Slurm)). To make your batch script, create a new bash script called 'slurm_newbie.sh' and add the following lines to the top of the file:

```
#!/bin/sh
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --time=1:00:00
#SBATCH -o slurm-%j.out-%N
#SBATCH -e slurm-%j.err-%N

cd ~/LizardLover
bash practice_script.sh
```

Every line that begins with ```#SBATCH``` is an option that will be interpreted by slurm. Any line that starts with ```#``` and is not imediately followed with ```SBATCH``` will simply be interpreted as a comment. To see the values you can include for ```partition```, use the ```myallocation``` command below.

```
myallocation
```

The ```--nodes``` (```-N```) option specifies the number of nodes you are requesting. You can think of a node as a single computer. When we refer to the supercomputer as "the cluster", we are referring to the "cluster" of nodes that are used. In other words, the collection of interconnected computers that can be accessed for computing tasks. For computationally intensive jobs, you may want to use multiple nodes. The login node is essentially a node with low memory dedicated to user interaction (e.g., writing scripts). The compute nodes have more memory and are dedicated to computation. A node is made up of multiple processors.

The '''---time''' option specifies the maximum runtime for the script. If the job goes over this time limit, the script will be terminated. The hours are broken up into 'hours':'minutes':'seconds'.

The ```-o``` option specifies the name of the standard output file (stdout). In this example, we use the job number (```%j```) and the first node (```%N```) as components of the filename (this is a good practice, but is not necessary). Anytime you use a command such as ```echo``` that typically prints something to your terminal screen, it will be appended to this file.

The ```-e``` option specifies the name of the error file. In this example, we use the job number (```%j```) and the first node (```%N```) as components of the filename (this is a good practice, but is not necessary). If your script runs into errors, the error messages will be printed to this file.

The ```--ntasks``` (```-n```) option specifies the number of parallelized tasks you will utilize.

When running a command in parallel, the SLURM built in ```$SLURM_NTASKS``` variable can then be used to denote the number of MPI tasks to run.

In case of a plain MPI job, this number should equal number of nodes (```$SLURM_NODES```) times number of cores per node.

# <a name="submitting a batch script to the queue"></a>
# Submitting a batch script to the queue

Rather than running your script locally, you're now ready to submit the script as a batch to the slurm queue. Submit the script using the following command:

```
sbatch slurm_newbie.sh
```

It may take some time to run if you aren't first in the queue
Check back periodically for the output files


# <a name="using-an-interactive-node"></a>
# Using an interactive node

Sometimes submitting a batch script is not efficient (you submit, then wait, sometimes wait some more, get distracted, and then check your output and see that you forgot a semi-colon). It is helpful to verify things interactively before submitting a full job, but you can't do this on the login node. Instead, you will work interactively on a compute node (similar to how you are accustomed to working in your local environment).

To open an interactive node, you will simply run an interactive job using the following command:

```
salloc --time=1:00:00 --ntasks 1 --nodes=1
```

This provides you with a computing space with a one-hour time limit where you can run commands/scripts you would run on a compute node using a batch submission.

