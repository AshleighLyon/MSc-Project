# Installing the Training Environment

The BNN-PYNQ Project provides a handy README guide which outlines the steps required to set up an environment and train a given dataset. However, I thought it would also be useful to outline the steps I took in more detail, to aid those with no initial experience with neural networks, as I found it to be a steep yet enjoyable learning curve.

## Setting Up AWS

As previously mentioned there are numerous ways in which to set up a successful training environment however these are the steps I took using Amazon Web Services EC2 instances.

1. Begin by reading the [AWS getting started guide](https://aws.amazon.com/ec2/getting-started/) and creating an account.
2. I had to request a limit increase from AWS for the instances used in this project, as they initially give you 0. Therefore, it is best to request a limit increase of 1 for all P2 instances. To do this select Limit from the side panel of your EC2 homepage and follow instructions. AWS usually get back to you within a few hours. Also note that some regions do not have all instances, I had to switch to another region to access the P2 instances.
3. Once limit has been authorised we can select launch an instance. 
4. First, search and select the Deep Learning AMI (ubuntu 16.04) from the AWS Marketplace option.
5. Next, select the p2.xlarge instance and then review and launch. The p2.xlarge instance comes with the following specifications:

| Name       | GPUs  | vCPUs | RAM   | ECUs  | Network Bandwidth|
| ---------- |:-----:|:-----:|:-----:|:-----:|:----------------:|
| p2.xlarge  | 1     |      4|61     |11.75  |High              |

6. Finally, create a key pair which is used to log into your instance from your computers terminal using SSH. 

## Connecting to Instance

Once your instance has been set up and key pair created, its time to connect via your computers terminal.

1. Open terminal and cd into folder where pem key is located. 
2. Connect via SSH where mykeypair.pem is the name of your key pair file and where @ec2... is the link copied from your EC2 instance homepage when your instance is running, as follows:

   ~~~~
   $ ssh -L localhost:8888:localhost:8888 -i mykeypair.pem ubuntu@ec2-00-00-000-00.eu-west-1.compute.amazonaws.com
   ~~~~
3. Once connected you are greeted with a number of environments containing frameworks such as Theano, PyTorch, Tensorflow and Caffe. We will be using the Theano framework with Python2:

   ~~~~
   $ source activate theano_p27
   ~~~~

## Install Python Packages

Now our AWS EC2 if fully set up with our instance type and environment, the next stage is to install some python packages.

1. First, install some dependencies:

   ~~~~
   $ sudo apt-get install git python-dev libopenblas-dev liblapack-dev gfortran -y
   ~~~~
2. Install latest version of pip:

   ~~~~
   $ wget https://bootstrap.pypa.io/get-pip.py && python get-pip.py --user
   ~~~~
3. Install Lasagne:

   ~~~~
   $ pip install --user https://github.com/Lasagne/Lasagne/archive/master.zip
   ~~~~
   Create a .theanorc configuration file and populate it as follows:
   
   ~~~~
   $ echo "[global]" >> ~/.theanorc
   $ echo "floatX = float32" >> ~/.theanorc
   $ echo "device = gpu" >> ~/.theanorc
   $ echo "openmp = True" >> ~/.theanorc
   $ echo "openmp_elemwise_minsize = 200000" >> ~/.theanorc
   $ echo "" >> ~/.theanorc
   $ echo "[nvcc]" >> ~/.theanorc
   $ echo "fastmath = True" >> ~/.theanorc
   $ echo "" >> ~/.theanorc
   $ echo "[blas]" >> ~/.theanorc
   $ echo "ldflags = -lopenblas" >> ~/.theanorc
   ~~~~
   Create a variable to specify the number of CPU threads you want to limit Theano to. If you want limit Theano to the number of threads you have available use the following:
   
   ~~~~
   $ export OMP_NUM_THREADS=`nproc`
   ~~~~
   
4. Install PyLearn 2 (Only required for MNIST and CIFAR-10 datasets):
   
   ~~~~
   $ pip install --user numpy==1.11.0 # Pylearn2 seems to not work with the latest version of numpy
   $ git clone https://github.com/lisa-lab/pylearn2
   $ cd pylearn2
   $ python setup.py develop --user
   $ cd ../../..
   ~~~~
   
5. Git clone the BNN-PYNQ repository:

   ~~~~
   $ git clone https://github.com/AshleighLyon/MSc-Project.git
   ~~~~
   
## Download Dataset

Now the training environment is fully up and running it is time to download and unzip the Fashion-MNIST dataset training images and labels. This can be done with the following command:

~~~~
$ wget -nc http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/train-images-idx3-ubyte.gz; gunzip -f train-images-idx3-ubyte.gz
$ wget -nc http://fashion-mnist.s3-website.eu-central-1.amazonaws.com/train-labels-idx1-ubyte.gz; gunzip -f train-labels-idx1-ubyte.gz
~~~~
Finally, navigate to the folder containing the training scripts - MSc-Project/BNN-PYNQ/bnn/src/training/ and run the following to begin training:

~~~~
$ fashion-mnist.py
~~~~

## Retrieve files from AWS EC2 instance

Once the training process has finished, we need to retreive the files from the Amazon Web Server and save onto our local drive to then load onto the PYNQ device. There are many options available, however for this I used [FileZilla](https://filezilla-project.org/). 
Once filezilla has been downloaded:
* Go to settings click on SFTP and add your pem key file. 
* Username: ubuntu
* Port: 22 
* Address: The DNS ECS address copied from your running instance. 

From here you can connect to your instance and download all the relevant files including the npz file. 

## Generating Binary Weight Files

Once the training process has finished, you'll have a file fashion-mnist.npz containing a list of numpy arrays which correspond to the real trained weights in each layer. In order to load them into the Pynq BNN Overlay they need converted from real floating point values into binary values and packed into .bin files. This is complete while on a Jupyter terminal on the PYNQ device. Navigate to BNN-PYNQ/bnn/src/training/ and run the following:

~~~~
$ python fashion-mnist-gen-binary-weights.py
~~~~

These scripts will process the weights for the given dataset and place them into a new directory. In order to load these weights on the Pynq, place the resultant folder into the /usr/local/lib/python3.6/dist-packages/bnn/params directory on the Pynq device (PYNQ version 2.3 or newer).

   
