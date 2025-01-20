Welcome to our Hex cluster! Hex uses Slurm to manage your jobs, and in the following we will give you some guidance.

# What is Slurm?
[Slurm](https://slurm.schedmd.com/documentation.html) is a cluster managment software. In short: You only need to submit jobs from a single computer/node, and Slurm makes sure to schedule your jobs and distribute them among the nodes of the cluster.
We will be giving you some info about running jobs in the following, but if you need more info then there is plenty of documentation on the web.

# Quick start

The very first step: Request a cluster account from either Karen, Kevin or Jed. Follow the how-to guide on how to generate an ssh certificate for the tunnel. The first time you log into Hex you may need to set a password.

## Request User Account and set up your ssh tunnel
To get an user account, do the following steps:

- Create an ssh key https://phoenixnap.com/kb/ssh-with-key
- Send the ssh public key to Ting-Chia(Karen) Chiang
- Get an account on a ripper from Ting-Chia(Karen) Chiang
- Add the following file contents to ~/.ssh/config. Fill in your own hex/ripper username:
```
AddKeysToAgent yes

Host bigmac
Hostname 145.108.195.17
Port 22
User hopper
ServerAliveInterval 10

Host hex
Hostname 10.0.0.1
Port 22
User <your_hex_username>
ProxyJump bigmac

Host ripper#
Hostname 10.0.0.#
Port 22
User <your_ripper_user>
ProxyJump bigmac

```

You should now be able to connect to the Hex cluster with `ssh <your_hex_username>@hex`. Alternatively, if you got an individual ripper account you can use `ssh <your_ripper_user>@ripper#`

## Connecting to Hex
After following the guidance and adding the hosts file to your computer, connect to the Hex cluster via `ssh yourusername@hex`. Replace yourusername with the user name of your cluster account. You should now be on the Hex cluster in your terminal, which you can see by the prefix in your temrinal which should show: yourusername@hex:

## Creating a first slurm batch job
Now lets create a new batch job script to test your access to the cluster. Generate first in your home directory a folder with the command `mkdir out`.
Now use your favourite editor (eg vim or nano) and create a file named `sbatch_example.sh`. Copy and paste the follwoing content into the file:
```
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --time=01:00:00
#SBATCH --cpus-per-task=6
#SBATCH --partition=batch
#SBATCH --output=./out/slurm_job.%J-%A-%a.out
#SBATCH -e ./out/slurm_job.%J-%A-%a.err
#SBATCH --array=1-10
##SBATCH --gres=gpu:1
##SBATCH --mem=10G

# Each array task runs the same program
hostname
sleep 10           
```
This is a simple sbatch config file you can use to run your jobs on the cluster. The lines with `#SBATCH` are used by Slurm to configure your job, lines with `##SBATCH` are commented out and not used. This script when executed with `sbatch` will generate 10 jobs, each executing the command `hostname` and then sleep for 10 seconds before ending the job. As you see, this shell script essentially first configures the slurm job, and then executes the commands given after the last `#SBATCH` statemen.

Before we explain each line, go ahead and test this program:
Run `sbatch sbatch_example.sh` and you should see the following output: `Submitted batch job 2673`
If you now exectue the command `squeue` you will see something similar to this:
```
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
            2683_1     batch test_job   ksluck  R       0:02      1 ripper2
            2683_2     batch test_job   ksluck  R       0:02      1 ripper2
            2683_3     batch test_job   ksluck  R       0:02      1 ripper2
            2683_4     batch test_job   ksluck  R       0:02      1 ripper2
            2683_5     batch test_job   ksluck  R       0:02      1 ripper2
            2683_6     batch test_job   ksluck  R       0:02      1 ripper2
            2683_7     batch test_job   ksluck  R       0:02      1 ripper2
            2683_8     batch test_job   ksluck  R       0:02      1 ripper2
            2683_9     batch test_job   ksluck  R       0:02      1 ripper2
           2683_10     batch test_job   ksluck  R       0:02      1 ripper2
```
Here, the user ksluck, has 10 jobs running - indicated in the column ST (for State) by `R`. If the cluster is currently busy, your jobs may be in a pending state and remain in the queue until there are free ressources to run your jobs.
If you execite `squeue` again after a while, your jobs should be gone as they have ben successfully executed. You can also now check the folder `out` in your home directory, which we created earlier, where you find the output of your jobs: They should show the hostname of the node on which your job run, eg `ripper2` in this case.

Great, you have successfully run your first Slurm job, congratulatiuons!

Lets now quickly go over the different slurm settings you can use (there are more, but these are the basics you need):
* `#SBATCH --job-name=test_job` - This is simply the name of the job you can see in squeue
* `#SBATCH --time=01:00:00` The time limit for the job. If it is too low, your job may be cancelled/interrupted. But if the number is too high, your job may need more time to be scheduled. Slurm tries to balance between short jobs and long-running jobs. Try to estimate the runtime as good as possible. The format here is HH:MM:SS
* `#SBATCH --cpus-per-task=6` How many CPUs each task needs. Again the higher the number the more time it may take to have the job scheduled.
* `#SBATCH --partition=batch` - The name of the partition on which to run the job. Read more about partitions below.
* `#SBATCH --output=./out/slurm_job.%J-%A-%a.ou` - Location of the output file, which contains the things printed to the shell within the job. %J, %A and %a are job number and array numbers (the latter may not be nedded if you do not run arrays).
* `#SBATCH -e ./out/slurm_job.%J-%A-%a.err` - Same as abov,e but prints error messages.
* `#SBATCH --array=1-10` - How many array jobs to run. As you see above, you can use the array number, eg %A, and use it as a parameter in your program. Here we run 10 array jobs each numbered from 1 to 10. Array jobs generall run the same commands.
* `#SBATCH --gres=gpu:1` If you use one of the gpu partitions, you need to specify the number of gpus (or type of gpus) you want to use. This is not needed for, eg batch.

## Basic commands

Let us now list some basic commands you should know:
* `sbatch` - used to run a batch job with a sh script as detailed above.
* `squeue` - Lists all jobs in the queue
* `scancel` - Can be used to cancel a job, usually with `scancel jobid`, where jobid is the number of your job you can find with `squeue`.
* `sinfo` - can be used to list partitions and see how busy the cluster is

## Running an interactive shell
Of course it is also possible to start an interactive shell (like ssh) on one of the nodes, such that you can test and evaluate your program environment.
Simply run `salloc -p batch`. Once you are done with it, exit it by typing the command `exit`.
`-p` here refers to the name of the partition. some other examples are:
```
salloc -p batch
salloc -p short
salloc -w ripper3 -p gpu --gres=gpu:1
```

## Partitions
Partitions can be considered job queues, each of which has an assortment of constraints such as job size limit, job time limit, users permitted to use it, etc. Priority-ordered jobs are allocated nodes within a partition until the resources (nodes, processors, memory, etc.) within that partition are exhausted

The follwong partitions are available:
| Name      | Timelimit | Ressources  | Nodes          | User groups     | Notes                                                                                                    |
|-----------|-----------|-------------|----------------|-----------------|----------------------------------------------------------------------------------------------------------|
| batch     | 4 days    | CPUs        | Ripper 2-7     | Staff,Students  | Standard job queue for all jobs which require CPUs                                                       |
| short     | 2 hours   | CPUs        | Ripper 1-7     | Staff, Students | For very short jobs, or for testing setups. Has a higher priority than batch.                            |
| batch-gpu | 2 days    | CPUs + GPUs | Ripper 2-7,12  | Staff, Students | GPU partition for students. You can only request 5 CPUs per GPU.                                         |
| gpu       | 4 days    | CPUs + GPUs | Ripper 2-7, 12 | Staff           | Priority queue for GPU jobs. You can only request 5 CPUs per GPU, and you need to be in the group staff. |
| gpu-short | 2 hours   | CPUs + GPUs | Ripper 2-7, 12 | Staff           | For very short GPU jobs or testing setups. Has a higher priority than gpu, but same restrictions.        |

Some partitions are only accessible to staff memebers by default. If you need exceptions, discuss them with your supervisor and Kevin.

## Use Conda!
To install your own python packages and necessary software you can use anaconda. We have for this miniconda installed on the cluster.
When you log into the main hex node, load the conda module and follow the guidelines on the anaconda website to create your envirnments:
```
module load miniconda
```
If you add the conda setups to your `.bashrc` then you do not need to load miniconda every time.
You can find a good guide on how to use Conda [here](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html)
