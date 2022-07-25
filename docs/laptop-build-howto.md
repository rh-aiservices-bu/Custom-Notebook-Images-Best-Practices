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
5. You should recieve the following message letting you know that your notebook image was succesfully built on your local workstation:
```
Successfully tagged localhost/user:v1
```

### Using a ready-made Dockerfile
For this example, we will be using a Dockerfile containing instructions to build a RStudio notebook image which can be found [here](https://github.com/guimou/custom-notebooks/blob/main/r-notebook/container/Dockerfile)
1. Git clone the repository into your working directory. For the same of simplicity, we are working in the same directory as the previous example.
```
[user@fedora notebook-images]$ git clone https://github.com/guimou/custom-notebooks.git
```
2. Go into the custom-notebooks/r-notebook/container/ directory. 
```
[user@fedora notebook-images]$ cd custom-notebooks/r-notebook/container/
```
3. Since the Dockerfile is ready-made, we don't need to create a new one. We can skip that step and build our notebook image from the ready-made one. Let's name it user:rstudiov1.
```
[user@fedora notebook-images]$ build -t user:rstudiov1 -f Dockerfile
```
4. Again, the following message will let you know that your notebook image was succesfully built on your local workstation:
```
Successfully tagged localhost/user:rstudiov1
```

## Create Quay.io account and public repo
Now that you know how to build a notebook on your local workstation from a Dockerfile, let's go over how you can push it to a public repository via Quay.io which is an container image registy. There are other registries out there such as the Docker Registry, but for this tutorial, we are using Quay.io

For the rest of this tutorial, we will be referencing the RStudio notebook image as our example.

1. For first time users, you need to sign up for an account on Quay.io
2. Create a repository for your project. Let's name it 'rstudio'

![](/img/quay-ui.png)
![](/img/quay-create-new-repo.png)

## pushing the image to quay
1. To push your image to Quay, we first have to login to our account through the terminal.
```
[user@fedora container]$ podman login quay.io
Username:
Password:
```
  The following message will let you know that your login was successful:
```
Login Succeeded!
```
2. Let's rename our locally stored image to differentiate it from the one that we'll be pushing to Quay. This step is optional. Here is the syntax as well as an example using our RStudio image. 
```
[user@fedora container]$ podman tag <old_name> <new_name>
```
  eg. RStudio Notebook Image
```
[user@fedora container]$ podman tag localhost/user:rstudiov1 quay.io/user/user:rstudiov1
```
3. Push the RStudio image to our Quay repository.
```
[user@fedora container]$ podman push quay.io/user/user:rstudiov1
```
  The following message will let you know that you successfully pushed to your Quay.io repo:
 ```
 ...
 Writing manifest to image destination
 Storing signatures
 ```
 4. You can now view your notebook image in your Quay.io repo.

  

## adding the image to RHODS


