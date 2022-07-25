# Jupyter Notebook Images - laptop build

In this document, we will do a step-by-step walkthrough of the required steps to: 
* Build a notebook image on your local workstation
* Create a quay.io account and public repo
* Push the local image into the registry
* Add the image to your RHODS environment

## Required software
* Podman (Installation instructions can be found [here](https://podman.io/getting-started/installation))


## Building
Yoy can either build your container based off of a Dockerfile you create from scratch or a ready-made one. We show both options below:
### Creating a Dockerfile from scratch 
1. In your terminal, create a new directory to work on building notebooks. For example, create a directory named notebook-images in your Documents directory.
```
[user@fedora ~]$ cd Documents
[user@fedora Documents]$ mkdir notebook-images
```
3. In your working directory, create a new file and name it. For example, let's name our file 'new.Dockerfile':
```
[user@fedora notebook-images]$ vim new.Dockerfile
```
3. Insert Docker commands into it before writing and quitting.

4. Build a notebook image from your Dockerfile and name it user:v1 by running the following command:
```
[user@fedora notebook-images]$ podman build -t user:v1 -f new.Dockerfile
```
### Using a ready-made Dockerfile
For this example, we will be using a Dockerfile containing instructions to build a RStudio notebook image which can be found [here](https://github.com/guimou/custom-notebooks/blob/main/r-notebook/container/Dockerfile)
1. Git clone the repository into your working directory. For the same of simplicity, we are working in the same directory as the previous example.
```
[user@fedora notebook-images]$ git clone https://github.com/guimou/custom-notebooks.git
```
2. Go into the r-notebook/container directory. 
```
[user@fedora notebook-images] cd r-notebook/container
```


## Create Quay.io account and public repo


## pushing the image to quay


## adding the image to RHODS


