# Running DeepLoc on AWS
DeepLoc, a Deep Learning approach for Protein Subcellular Localization. 


## Introduction
This document provides step-by-step instructions on (a) establishing a connection to an AWS EC2 instance from your terminal; (b)  transferring files from your local device to the VM; (c) setting up the environment to run DeepLoc. It will also cover the prerequisites to ensure that your code runs smoothly.</br></br>



## Prerequisites (One-time)
We assume that your operating system is iOS, and that you have activated your AWS account.
1. **AWS Support for extending the vCPU Limit to 32**<br/>New AWS accounts are assigned zero virtual CPUs (vCPU) by default. However, computationally intensive tasks use use instances requiring more than 16 units. For DeepLoc, we would require instances with 32+ vCPU units. <br/><br/>Go to your AWS [console](https://console.aws.amazon.com/support/home#/) and search for [Support](https://console.aws.amazon.com/support/home#/). Raise a token by creating a case and selecting Service limit increase. Fill in the following details to submit your request. The AWS team should approve your request and increase your vCPU limit within an hour.<br/><br/>![](/images/Docim_support.png)<br/><br/>You can enter a use-case based on your task. Here’s an example:  *“We're running computationally intensive enterprise-level DL models, for a subcellular localization task (business/research use-case).  The startup that I'm working at is called NonExomics. We'd be grateful if ya'll could approve of the vCPU limit so that we could use P2/P3 instances at the earliest”.*

2. **Turn off Sleep Mode on your MAC**<br/> Go to *Energy Saver->Power Adapter*. Select the option *Prevent computer from sleeping automatically when the display is off*.

3. **Changes to SSH-related Configuration Files**<br/> We do this to ensure that the connection pipeline doesn't break (while you SSH to an Amazon EC2 instance). Two of the most common errors that you might encounter while training your model are, “Write failed: Broken pipeline” and "packet_write_wait: Connection to xx.xx.xx.xx: Broken pipe" [[Read More]](https://forums.aws.amazon.com/message.jspa?messageID=914326). We have a workaround! Open your terminal and edit the three configuration files as follows:


### Client-side Configuration
 Step 1:  Enter **$sudo nano /etc/ssh/ssh_config** on the terminal. <br/> Add and save the following lines under *Host **:

 ```shell
    ServerAliveInterval 120
    TCPKeepAlive yes
    ServerAliveCountMax 5
 ```

 Step 2:  Enter **$sudo nano ~/.ssh/config** on the terminal. <br/>
 Add and save the following lines:
 ```shell
  Host *
    ServerAliveInterval 30
    ServerAliveCountMax 5
 ```

 ### Server-side Configuration

 Step 3: Enter **$ sudo nano  /etc/ssh/sshd_config** on the terminal.<br/>
 Add the following lines under *Host **:
 ```shell
    ClientAliveInterval 600
    ClientAliveCountMax 0
```
 Check if *TCPKeepAlive yes* is present (add it if it isn't).
 Restart your system for the changes to reflect.


4. **Fork the DeepLoc Repository on Github**  
Set up your Github account and fork the [DeepLoc repo](https://github.com/ThanhTunggggg/DeepLoc). If you don’t fork the original repository, you will be denied access to the code while cloning the repository later.

Contact @sukritipaul05 if you encounter any other issues.

## Downloading the Dataset
Download the .fasta file to your local */Downloads* folder from [[Link]](http://www.cbs.dtu.dk/services/DeepLoc/data.php). Alternatively, you can rename your own dataset to *deeploc_data.fasta* and move it to the local */Downloads* folder.

## Connecting to the Running AWS Instance
Select the Amazon Deep Learning AMI (Ubuntu 16.04) and launch a <b style='color:purple'>p2.8xlarge</b> instance. Please note that you must not select any other instance (that might cause NonExomics to incur more costs than what the budget permits). You can refer to the [AWS Basics Demo](https://drive.google.com/file/d/1vVHFiewB7lBt4lWgtqfcRGpppKEMbdLP/view?usp=sharing) to get your instance running.

To securely connect to the AWS running instance, we SSH into the server (using the private key and public IPv4 on the AWS console).  Open your terminal and navigate to the folder having the private aws_deeploc1_key.pem key.

```shell
cd /Users/your_username/Downloads/
chmod 0400 aws_deeploc1_key.pem
ssh -L localhost:8888:localhost:8888 -i aws_deeploc1_key.pem ubuntu@<your instance DNS >
#Example
#ssh -L localhost:8888:localhost:8888 -i aws_deeploc1_key.pem ubuntu@ec2-13-59-192-159.us-east-2.compute.amazonaws.com
```

* For this example, we’ve named the key as *aws_deeploc1_key.pem*. You can give it any name while launching the instance. Use the same name in the commands.
* The SSH command requires the public iPV4 of the instance, which can be copied from the *Public IPv4 DNS* column in the [View Instances page](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Instances:) of the AWS Console.

<p style='color:tomato'>Remember: From this step onwards, you’ve successfully established a connection from the local machine to the virtual server. Therefore, whatever you write/create/modify/delete/configure i.e., whichever command you perform on the current terminal gets reflected on the server and not your local device! Also, remember that NonExomics is being charged (per hour) when the instance is in the running state.</p>


## Getting the DeepLoc Folder Ready on the Virtual Server

1. Set up Git on the Virtual Server by running these commands on the terminal:

```shell
$git config --global user.email "<Enter your Github E-mail>"
$git config --global user.name "<Enter your Github Username>"
```

2. Check if you’ve forked the DeepLoc Github repository. On the Github webpage, it should be in the format < *github username* >/*DeepLoc* preceded by a fork symbol.

3. To make a copy of the repository on the virtual server:
```shell
$git clone < DeepLoc repository URL>
# Example: git clone https://github.com/nonexomics09/DeepLoc
```
4. Create a */data* folder in the */DeepLoc* Folder.
```shell
$cd DeepLoc
$mkdir data
```

5. Transfer the *deeploc_data.fasta* file from the local client system to the */DeepLoc/data* folder on the virtual server using SCP. Open a new local (client) terminal window and type the command below.
```shell
$scp -i <path to .pem key> <copy file from path> user@server:<copy file to path>
# Example: scp -i /Users/nonexomics/Downloads/aws_deeploc1_key.pem /Users/nonexomics/Downloads/deeploc_data.fasta ubuntu@ec2-13-59-192-159.us-east-2.compute.amazonaws.com:/home/ubuntu/DeepLoc/data
```
The Public IPv4 DNS of the instance (corresponding to the *server* bit in the scp command) can be found in the [View Instances page](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Instances:) of the AWS Console. Cross-check if the deeploc_data.fasta file is present in the */DeepLoc/data* folder.

## Setting up a Virtual Environment to Run the Python Files

Data Science best practices emphasise the creation of virtual environments for each project. Your current project may have a set of requirements that do not satisfy those of other projects. If you install packages and libraries directly on the virtual server, there may be incompatibilities in package/library/framework versions. We create a virtual environment and activate it, so that the project requirements are installed without friction, and are independent of the installations on the main virtual server.

Fortunately, the p2.8xlarge instance comes with a set of pre-installed virtual environments that can be activated from the terminal. Although you can create your own virtual environment on Ubuntu, and install the packages/libraries in the requirements.txt one by one, we recommend that you use one of the preexisting environments  called ‘pytorch_p36’. More often than not, you may encounter clashes between the CUDA and PyTorch versions if you choose to create a new virtual environment from scratch.

The next step describe how you can activate the  <b style='color:purple'>pytorch_p36</b> environment.
```shell
$cd ~
$source activate pytorch_p36
#You can view the list of other existing virtual environments via $conda info --envs
```

## Running the Python Files in /DeepLoc

![](/images/model_architecture1.png)</br> *Model Architecture image taken from Thanh Tung Hoang's repository [1]*</br></br>
For this section, we follow the steps in [this](https://github.com/ThanhTunggggg/DeepLoc) README.md documentation. Navigate to the */DeepLoc* folder on the server, via your terminal. All the subsequent commands should be run from the */DeepLoc* directory.

### Steps
1. **Run the requirements.txt file.**
```shell
pip install -r requirements.txt
```

2. **Prepare the dataset** and build the vocabularies/parameters for the dataset.
```shell
python build_dataset.py
python build_vocab.py --data_dir data/
```

3. **Phase-1 Training** involves a simple train.
```shell
python train.py --data_dir data --model_dir experiments/base_model
```

4. **Phase-2 Training**: To select optimal hyper-parameters (learning rate) and display them.
```shell
python search_hyperparams.py --data_dir data --parent_dir experiments/learning_rate
python synthesize_results.py --parent_dir experiments/learning_rate
```

5. **Test set evaluation.**
```shell
python evaluate.py --data_dir data --model_dir experiments/base_model
```

Note: Once you’re done with the training and testing, you may transfer files to your local device or backup on Github. The server hard disk memory is temporary (ephemeral drive) and your instance will be wiped clean when you terminate it.<b style='color:tomato'> Lastly, please terminate the instance on the AWS console as soon as you’ve completed the tasks to avoid unnecessary expenses!</b>


## Additional Section (Implementation Summary)
### Instance Details

| Item| Description |
| --- | --- |
| AMI (Virtual Machine) | Amazon Deep Learning AMI (Ubuntu 16.04) |
| Instance Type | <b style='color:purple'>p2.8xlarge</b> : This is a cost-effective option, given that we need >16 GiB of GPU memory for this task, and an instance that enables accelerated computing (specs in the image below). |
| Training Runtime |  <b style='color:purple'>~4.5</b> hours (on the entire fasta file), for 20 epochs. There are two phases for training, via two training files- so the total runtime can be estimated to be 9-10 hours if it is run during the day (Indian Time Zone). |
| Testing Runtime | <1 minute |
| Instance cost (per hour) | <b style='color:tomato'>$7.20 per hour.</b> |
| Backup and External Memory |  <b style='color:tomato'>No backup</b>, at present. All the work gets wiped-off once the instance is turned off. To save our work, we would have to pay for additional EBS (external physical memory). This type of additional storage is billable even when the instance is switched off. |
| Region (instance) | East-US Ohio |

### AMI (Virtual Machine) & Instance Specs
The DeepLoc task requires NVIDIA CUDA, cuDNN, TensorFlow, and a compatible version of PyTorch. The p2.8xlarge instance is a viable *instance type*.

![](/images/instance_spec.png)<br/> *p2.8xlarge instance details*

### Snippets from the Training & Testing Phases
![](/images/implement_1.png)<br/> *Fig. 1:  Data Description (Vocabulary)*

![](/images/implement_2.png)<br/> *Fig. 2:   Train, Test & Validation Data set Creation from the Fasta file*

![](/images/implement_3.png)<br/> *Fig. 3:   Training acc=64%, loss=0.96 (without optimal hyperparams)*

![](/images/implement_4.png)<br/> *Fig. 4:   Test acc=66%, loss=0.98 (trained without optimal hyperparams)*

## References
- DeepLoc Repo by Thanh Tung Hoang. [[Link]](https://github.com/ThanhTunggggg/DeepLoc)
- [CS230 Deep Learning](https://github.com/cs230-stanford/cs230-stanford.github.io)
- Almagro Armenteros, José Juan, et al. "DeepLoc: prediction of protein subcellular localization using deep learning." Bioinformatics 33.21 (2017): 3387-3395. [[Link]](https://academic.oup.com/bioinformatics/article/33/21/3387/3931857)

###### Any changes to the model architecture in this repository are attributed to NonExomics (&copy;2021,NonExomics). Contact @sukritipaul05 while selecting your AWS EC2 instance or if you encounter any other issues.
