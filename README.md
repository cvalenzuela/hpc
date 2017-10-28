# NYU HPC

A quick reference to access NYU's High Performance Computing Prince Cluster.

The official wiki is [here](https://wikis.nyu.edu/display/NYUHPC/Clusters+-+Prince), this is an unofficial document created as a quick-start guide for first-time users.

## Get an account

You need to be affiliated to NYU and have a sponsor. 

To get an account approved, follow [this steps.](https://wikis.nyu.edu/display/NYUHPC/Requesting+an+HPC+account+with+IIQ)

## Log in

Once you have been approved, you can access HPC from inside the NYU network:

```zsh
ssh NYUNetID@prince.hpc.nyu.edu
```

Once logged in, the root should be:
`/home/NYUNetID`, so running `pwd` should print:

```zsh
[NYUNetID@log-0 ~]$ pwd
/home/NYUNetID
```

## File Systems

You can get acces to four filesystems: `/home`, `/scratch`, `/archive`, and `/work`.

Scratch it a file system mounted on Prince that is connected to the compute nodes where we can upload files faster. The content gets periodically flushed.

```zsh
[NYUNetID@log-0 ~]$ cd /scratch/NYUNetID
[NYUNetID@log-0 ~]$ pwd
/scratch/NYUNetID
```

`/home` and `/scratch` are separate filesystems, in separate places.

## Install Anaconda

Follow [this](https://www.digitalocean.com/community/tutorials/how-to-install-the-anaconda-python-distribution-on-ubuntu-16-04) tutorial to install Anaconda.

## Install tensorflow-gpu

installing Tensorflow for GPU's requires NVidia's **CUDA® Toolkit 8.0** and **cuDNN v6** installed.
To download **CUDA® Toolkit 8.0** use 
```
wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64-deb
```

To download **cuDNN v6** you must sign in as an Nvidia developer, alternatively you can use my self-hosted version using 
```
wget orfleisher.com/downloads/cudnn-8.0-linux-x64-v6.0.tgz
```

After downloading you can install **CUDA® Toolkit 8.0** using ```sh ./cuda-repo-ubuntu1604-8-0-local-ga2_8.0.61-1_amd64-deb``` you will be prompted with EULA and questions, you **do not** need display driver or samples.

Set a path variable 
```
export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
```

Unzip **cuDNN v6** using ```tar -xvzf cudnn-8.0-linux-x64-v6.0.tgz```

To install simply copy the files from the unzipped **cuDNN v6** library to the main cuda library like so
```
cp cuda/lib64/* /usr/local/cuda/lib64/
```
```
cp cuda/include/cudnn.h /usr/local/cuda/include/
```

You should be good to go after this and can simply install tensorflow-gpu using ```pip3 install --upgrade tensorflow``` 

For a tutorial on installing **CUDA® Toolkit 8.0**, **cuDNN v6** and **tensorflow-gpu**, [follow this](https://medium.com/@acrosson/installing-nvidia-cuda-cudnn-tensorflow-and-keras-69bbf33dce8a) 

## Request CPU

You can submit batch jobs in prince to schedule jobs. This requires to write custom bash scripts. You can also run in interactive mode:

```zsh 
[NYUNetID@log-0 ~]$ srun --pty /bin/bash
```

This will run the default mode: a single CPU core and 2GB memory for 1 hour.

To request more CPU's:

```zsh
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

```zsh
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
