# MSc-Project

This is the repository for the final MSc Project and dissertation complete in part fulfilment of the requirements of the Degree of Master of Science at the University of Glasgow. This project explores the deployment of a deep neural network on a Xilinx FPGA. 

## Project Objectives

* Obtain a clear understanding of the theory and application of neural networks, with a particular focus on binarized neural networks.
* Gain insight into software tools and libraries used in the process of deep learning.
* Review the importance of hardware, particularly FPGAs, in the aid and acceleration of deep neural networks.
* Set up a working training environment which successfully trains a convolutional neural network (CNN) or a large fully connected (LFC) network on a particular dataset.
* Deploy a trained neural network on the Xilinx PYNQ-Z1 FPGA, classifying a test set of images.
* Compare performance between hardware and software inference on the PYNQ-Z1 and evaluate the accuracy of the image classification.

## Project Stages

1. Set up PYNQ-Z1 device and install BNN-PYNQ overlay.
2. Set up a training environment using Amazon Web Services (AWS).
3. Train the network on the Fashion-MNIST dataset.
4. Convert npz file from real floating point values into binary values and pack into .bin files.
5. Load binary weights onto PYNQ.
6. Create new Jupyter Notebook and using python test networks accuracy and speed.

## Acknowledgments

This project is based on the BNN-PYNQ repository project by Xilinx originally found [here](https://github.com/Xilinx/BNN-PYNQ).
