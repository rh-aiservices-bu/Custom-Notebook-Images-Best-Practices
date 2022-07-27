# Jupyter Notebook Images - laptop build

In this document, we will do a step-by-step walkthrough of the required steps to:
* Build a notebook image on your local workstation
* Create a quay.io account and public image registry
* Push the local image into the registry
* Add the image to your RHODS environment

## Required software
* Podman (Installation instructions can be found [here](https://podman.io/getting-started/installation))

    * More information regarding podman commands can be found [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/managing_containers/finding_running_and_building_containers_with_podman_skopeo_and_buildah) or alternatively, run the following command in your terminal:

    ```
    podman --help
    ```

## Building
Yoy can either build your container based off of a Dockerfile you create from scratch or a ready-made one. We will show both options below:

### Creating a Dockerfile from scratch

1. In your terminal, create a new directory to work on building notebooks. For example, create a directory named 'notebook-images' in your Documents directory.

    ```
    cd ~/Documents
    mkdir -p ~/Documents/notebook-images
    ```

3. In your working directory, create a new file and name it. For example, let's name our file 'new.Dockerfile':

    ```
    vim ~/Documents/notebook-images/new.Dockerfile
    ```

3. Insert Docker commands into it by pressing 'i' on your keyboard. When you are done, save and quit by pressing the 'esc' key and ':wq'. More information on vim commands can be found [here](https://www.redhat.com/sysadmin/four-things-vim).

4. Build a notebook image from your Dockerfile and name it 'image:userv1' by running the following command:

    ```
    cd ~/Documents/notebook-images
    podman build -t image:userv1 -f new.Dockerfile
    ```

5. You should receive the following message letting you know that your notebook image was successfully built on your local workstation:

    **Successfully tagged localhost/image:userv1**


### Using a ready-made Dockerfile

For this example, we will be using a Dockerfile containing instructions to build a RStudio notebook image which can be found [here](https://github.com/guimou/custom-notebooks/blob/main/r-notebook/container/Dockerfile).

1. Git clone the repository into your working directory. For the sake of simplicity, we are working in the same directory as the previous example.

    ```
    cd ~/Documents/notebook-images/
    git clone https://github.com/guimou/custom-notebooks.git
    ```

2. Go into the 'custom-notebooks/r-notebook/container/' directory.

    ```
    cd ~/Documents/notebook-images/custom-notebooks/r-notebook/container/
    ```

3. Since the Dockerfile is ready-made, we don't need to create a new one. We can skip that step and build our notebook image from the ready-made one. Let's name our notebook image 'user:rstudiov1'.

    ```
    podman build -t rstudio:userv1 -f Dockerfile
    ```

4. Again, the following message will let you know that your notebook image was succesfully built on your local workstation:

    **Successfully tagged localhost/rstudio:userv1**


## Create Quay.io account and public repo

Now that you know how to build a notebook on your local workstation from a Dockerfile, let's go over how you can push it to a public container image registry.
As an example, we will user [quay.io](https://quay.io/). You can however use other registries, such as [DockerHub](https://hub.docker.com/) if you prefer.

**For the rest of this tutorial, we will be referencing the RStudio notebook image as our example.**

1. For first time users, you need to sign up for an account on Quay.io

    <p align="center"><img src=/img/quay-ui.png width=900 height=300></p>

2. Create a repository for your project. Let's name it 'rstudio'

    <p align="center"><img src='/img/quay-create-new-repo.png' width=900 height=500></p>

## Pushing the image to Quay

1. To push our image to Quay.io, we first have to login to our account through the terminal.

    ```
    podman login quay.io
    ```
    The following message will let us know that our login was successful:

    **Login Succeeded!**

2. Let's rename our locally stored image from 'localhost/rstudio:userv1' to 'quay.io/user/rstudio:userv1'. This step is NOT optional. In reality, it does not rename the image, it adds an alternative name to it. In our case, this name needs to match the name the image will have once it's pushed to quay.io.

    ```
    podman tag localhost/rstudio:userv1 quay.io/user/rstudio:userv1
    ```

3. Push the RStudio image to our Quay.io repository. Notice how the the name of the "remote" image matches exactly with the name used in the `tag` command just above.

    ```
    podman push quay.io/user/rstudio:userv1
    ```
    The following messages will let us know that we successfully pushed to your Quay.io repository:

    **Writing manifest to image destination**

    **Storing signatures**

 4. You can now view your notebook image in our Quay.io repository.

<p align="center"><img src=/img/quay-push.png width=900 height=250></p>

## Adding the image to RHODS

1. Click on the 'Fetch Tag' icon. From the 'Image Format' dropdown menu, select 'Podman Pull (by tag)' and copy the latter half of the podman pull command.

    <p align="center"><img src=/img/img-tag.png width=900 height=450></p>

2. Go to the your own RHODS environment.
1. Make sure to log in with an Account that is considered to be a RHODS administrator.
1. Navigate to the 'Settings' dropdown menu on the left and select 'Notebook Images'

    <p align="center"><img src=/img/rhods-pilot-cluster.png width=900 height=450></p>

3. Click on 'Import Notebook' and paste your notebook image tag in the 'Repository' section and give your notebook image a name in the 'Name' section.

    <p align="center"><img src=/img/import-nb-imgs.png width=900 height=500></p>

4. Make sure you enable your notebook image before navigating to the 'Applications' dropdown menu on the left and selecting 'Enabled'

    <p align="center"><img src=/img/enable-nb-img.png width=900 height=500></p>

5. Go to JupyterHub and select your notebook image and launch a RStudio Notebook.

  <p align="center"><img src=/img/rstudio-v1.png width=400 height=300></p>

  <p align="center"><img src=/img/rstudio-notebook.png width=900 height=450></p>

6. Since this is an RStudio notebook image, let's do a sanity check by running the following commands:

    ```
    > print('hello world!')
    > SessionInfo()
    ```

![r studio](/img/rstudio-sanity-check.png "Rstudio")

7. If your outputs look normal, congratulations, you have created your first notebook image on RHODS.
