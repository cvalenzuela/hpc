# NYU HPC

A quick reference to access NYU's High Performance Computing Prince Cluster.

The official wiki is [here](https://wikis.nyu.edu/display/NYUHPC/Clusters+-+Prince), this is an unofficial document created as a quick-start guide for first-time users with a focus in Python.

## Get an account

You need to be affiliated to NYU and have a sponsor. 

To get an account approved, follow [this steps.](https://wikis.nyu.edu/display/NYUHPC/Requesting+an+HPC+account+with+IIQ)

## Log in

Once you have been approved, you can access HPC from:

 1. Within the NYU network:

```bash
ssh NYUNetID@prince.hpc.nyu.edu
```

Once logged in, the root should be:
`/home/NYUNetID`, so running `pwd` should print:

```bash
[NYUNetID@log-0 ~]$ pwd
/home/NYUNetID
```

2. An off-campus location:

First, login to the bastion host:

```bash
ssh NYUNetID@gw.hpc.nyu.edu
```

Then login to the cluster:

```bash
ssh prince.hpc.nyu.edu
```

## File Systems

You can get acces to four filesystems: `/home`, `/scratch`, `/archive`, and `/work`.

Scratch is a file system mounted on Prince that is connected to the compute nodes where we can upload files faster. Notice that the content gets periodically flushed.

```bash
[NYUNetID@log-0 ~]$ cd /scratch/NYUNetID
[NYUNetID@log-0 ~]$ pwd
/scratch/NYUNetID
```

`/home` and `/scratch` are separate filesystems in separate places, but you should use `/scratch` to store your files.

## Install Anaconda

1. Download Anaconda. Check for the latest version [here](https://www.anaconda.com/download/#linux):

```bash
curl -O https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh
```

2. Install

```bash
bash Anaconda3-4.2.0-Linux-x86_64.sh
```

3. Activate the installation

```bash
source ~/.bashrc
```

4. Create an environment:

```bash
conda create --name my_env python=3
```

A complete tutorial is [here](https://www.digitalocean.com/community/tutorials/how-to-install-the-anaconda-python-distribution-on-ubuntu-16-04)

## Install tensorflow-gpu

Installing Tensorflow for GPU's requires NVidia's **CUDA® Toolkit 8.0** and **cuDNN v6** installed.

1. Download **CUDA® Toolkit 8.0**: 

```
wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run
```

2. Download **cuDNN v6**. You must sign in as an Nvidia developer, alternatively you can use my self-hosted version using 

```bash
wget orfleisher.com/downloads/cudnn-8.0-linux-x64-v6.0.tgz
```

3. After downloading you can install **CUDA® Toolkit 8.0** using 

```bash
sh ./cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64-deb
``` 
you will be prompted with EULA and questions, you **do not need display driver or samples.**

4. Set a path variable in your ./.bashrc file:

```bash
export LD_LIBRARY_PATH="/home/cv965/cuda/lib64:$LD_LIBRARY_PATH"
```

5. Unzip **cuDNN v6** using 

```bash
tar -xvzf cudnn-8.0-linux-x64-v6.0.tgz
```

6. To install simply copy the files from the unzipped **cuDNN v6** library to the main cuda library like so:

```
cp cuda/lib64/* /usr/local/cuda/lib64/
```
```
cp cuda/include/cudnn.h /usr/local/cuda/include/
```

You should be good to go after this and can simply install tensorflow-gpu using 

```bash
source activate yourEnv
pip3 install --upgrade tensorflow
``` 

For a complete tutorial on installing **CUDA® Toolkit 8.0**, **cuDNN v6** and **tensorflow-gpu**, [follow this](https://medium.com/@acrosson/installing-nvidia-cuda-cudnn-tensorflow-and-keras-69bbf33dce8a) 

## Request CPU

You can submit batch jobs in prince to schedule jobs. This requires to write custom bash scripts. You can also run in interactive mode:

```bash 
[NYUNetID@log-0 ~]$ srun --pty /bin/bash
```

This will run the default mode: a single CPU core and 2GB memory for 1 hour.

To request more CPU's:

```bash
[NYUNetID@log-0 ~]$ srun -n4 -t2:00:00 --mem=4000 --pty /bin/bash
[NYUNetID@c26-16 ~]$ 
```
That will request 4 compute nodes for 2 hours with 4 Gb of memory.


To exit a request:
```
[NYUNetID@c26-16 ~]$ exit
[NYUNetID@log-0 ~]$
```

## Request GPU

```bash
[NYUNetID@log-0 ~]$ srun --gres=gpu:1 --pty /bin/bash
[NYUNetID@gpu-25 ~]$ nvidia-smi
Mon Oct 23 17:49:19 2017
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 367.48                 Driver Version: 367.48                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           On   | 0000:12:00.0     Off |                    0 |
| N/A   37C    P8    29W / 149W |      0MiB / 11439MiB |      0%   E. Process |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```


## Submit a job

You can write a script that will be executed when the resources you requested became available.

A simple CPU demo:

```bash
## 1) Job settings

#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --time=5:00:00
#SBATCH --mem=2GB
#SBATCH --job-name=CPUDemo
#SBATCH --mail-type=END
#SBATCH --mail-user=itp@nyu.edu
#SBATCH --output=slurm_%j.out
  
## 2) Everything from here on is going to run:

cd /scratch/NYUNetID/demos
python demo.py
```

Request GPU:

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:4
#SBATCH --time=10:00:00
#SBATCH --mem=3GB
#SBATCH --job-name=GPUDemo
#SBATCH --mail-type=END
#SBATCH --mail-user=itp@nyu.edu
#SBATCH --output=slurm_%j.out

cd /scratch/NYUNetID/trainSomething
source activate ML
python train.py
```

Submit your job with:

```bash
sbatch myscript.s
```

Monitor the job:

```bash
squeue -u $USER
```

More info [here](https://wikis.nyu.edu/display/NYUHPC/Submitting+jobs+with+sbatch)

# Screen

Create a `./.screenrc` file and append this [gist](https://gist.github.com/joaopizani/2718397)