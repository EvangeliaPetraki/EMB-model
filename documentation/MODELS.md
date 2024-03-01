# How to train models on the HPC cluster

## Send Dermnet dataset to HPC

The first thing we will need to do is send the Dermnet dataset to our home in HPC. This can be done like so using `scp`:

 An auth [vpn](https://it.auth.gr/manuals/eduvpn/) must be active for this to work!!

After downloading the dataset locally from [https://www.kaggle.com/datasets/shubhamgoel27/dermnet](https://www.kaggle.com/datasets/shubhamgoel27/dermnet) locate the download directory and open an powershell terminal:

    ```
    scp .\archive.zip <username>@aristotle.it.auth.gr:~/
    ```
This will copy the zipped dataset to your home. You can change the `~/` directory if you want the file to copy somewhere else

After that connect to HPC via ssh and unzip the archive:
    
    ```
    ssh <username>@aristotle.it.auth.gr
    unzip archive.zip
    ```



## Send Code to HPC

Download the code from `src/`

Before sending the code you may need to make some changes to the code to suit your enviroment:

1. Edit `train_data_path` and `test_data_path` in [dataset_path.py](/src/dataset_paths.py) 
1. Edit `PATH` in [Model_train.py](/src/Model_train.py) and [Model_test.py](/src/Model_test.py) 

Alternatively you can send the code as is and use the `vim` editor inside `HPC`. You can follow the guides found here: 
[Getting started with Vim: The basics](https://opensource.com/article/19/3/getting-started-vim)

Similarly with above send it to HPC via `scp` (you may need to zip it first)

## Create python virtual enviroment for pytorch+cuda in HPC

Login via ssh as above and run the following commands:

    ```
    $ module load gcc/12.2.0 python/3.10.10
    $ python3 -m venv ~/venv/pytorch-2.1.0
    $ source ~/venv/pytorch-2.1.0/bin/activate
    $ pip install --upgrade pip
    $ pip install torch==2.1.0 torchvision==0.16.0 torchaudio==2.1.0 --index-url https://download.pytorch.org/whl/cu118
    ```

To confirm everything was install correct run the command `python` and in the python terminal that opens enter `import torch`

After that enter the directory in which you sent the code:

    ```
    $ pip install -r requirements.txt
    ```

## Sending job for model training

First with the previous enviroment activated run:

    ```
    $ python Model_train.py --help
    $ python Model_test.py --help
    ```
This will print out the command line options for each scripts

The submittion script that will tell `hpc` to run our code will be in the form of a bash (`.sh`) file as mentioned [here](https://hpc.it.auth.gr/jobs/job-submission/)

An example script will be like so:

    ```
    #!/bin/bash
    #BATCH --job-name=EMB_model
    #SBATCH --partition=ampere
    #SBATCH --gres=gpu:1
    #SBATCH --ntasks=16
    #SBATCH --time=6:00:00

    source ~/venv/pytorch-2.1.0/bin/activate

    python3 ~/EMB/Code/Model_train.py -l 5e-4 -E 200 -b 128 -s 0.96 -o train_6

    python3 ~/EMB/Code/Model_test.py -i train_6
    ```