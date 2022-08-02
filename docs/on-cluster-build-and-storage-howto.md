# Jupyter Notebook - on cluster build and storage

In this document, we go over a step-by-step walkthrough to build and store a notebook container image on the OpenShift cluster by demonstrating how to build an R Notebook image with Streamlit:

* Create an ImageStream object in YAML
* Create a BuildConfig object in YAML (optional)

## Requirements
* Admin level access to the OpenShift cluster where RHODS is deployed
* Your preferred text editor
* OpenShift Client i.e oc (Installation instructions can be [here](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html))
   
    * More information regarding oc commands can be found [here](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/developer-cli-commands.html) or alternatively, run the following command in your terminal:
     ```
    oc --help
     ```
## Prerequisties
1. In your working directory, clone this git repository.
    ```
    git clone https://github.com/rh-aiservices-bu/Custom-Notebook-Images-Best-Practices.git
    ```
2. Go into this project's directory.
    ```
    cd Custom-Notebook-Images-Best-Practices/r-notebook-streamlit
    ```
**All of the following steps will take place in this directory**

## Creating an Imagestream object definition in YAML

In order for a custom image to be available in RHODS JupyterHub, an ImageStream must be created in the redhat-ods-applications namespace of the OpenShift cluster where RHODS is deployed.

For the purposes of this tutorial, we will update one of the R-Studio default notebook image to include Streamlit. You can find the original ImageStream [here](https://github.com/rh-aiservices-bu/Custom-Notebook-Images-Best-Practices/blob/main/r-notebook/deploy/odh-minimal-data-science-r-notebook_image-stream.yml).

1. In your preferred text editor, name your ImageStream object 'r-notebook-with-streamlit_imagestream.yaml' and update it accordingly:
    ```
    kind: ImageStream
    apiVersion: image.openshift.io/v1
    metadata:
        annotations:
            opendatahub.io/notebook-image-desc: >-
                JupyterLab minimal notebook image with R, IRKernel and RStudio.
            opendatahub.io/notebook-image-name: users rstudio streamlit
             opendatahub.io/notebook-image-order: '200'
            opendatahub.io/notebook-image-url: 'https://github.com/rh-ai-services/custom-notebooks/tree/main/r-notebook'
        name: users-rstudio-streamlit 
        labels:
            component.opendatahub.io/name: jupyterhub
            opendatahub.io/component: 'true'
            opendatahub.io/notebook-image: 'true'
            app.kubernetes.io/created-by: byon 
    spec:
        lookupPolicy:
            local: true
        tags:
         - name: v1
            annotations:
                opendatahub.io/notebook-python-dependencies: >-
                [{"name":"R","version":"4.0.5"},{"name":"RStudio","version":"2022.02.0-443"}]
                opendatahub.io/notebook-software: '[{"name":"BYO Notebook Image -
                R","version":"4.0.5"},{"name":"RStudio","version":"2022.02.0"},{"name":"Python","version":"3.9.10"}, 
                {"name": "Streamlit", "version":"1.11.1"}]'
            referencePolicy: 
                type: Local

    ```
    The official documentation for configuring ImageStream objects can be found [here](https://docs.openshift.com/container-platform/4.8/rest_api/image_apis/imagestream-image-openshift-io-v1.html).

2. To save your definition to a file and then create it, run the following command:

    ```
    oc apply -n redhat-ods-applications -f r-notebook-with-streamlit_imagestream.yaml
    ```
3. You can view the newly created ImageStream object on the in the list of ImageStream objects on the OpenShift cluster by running:
   
   ```
   oc -n redhat-ods-applications get imagestreams
   ```

## Creating a BuildConfig object definition in YAML (optional)
Suppose you want to build your notebook container image locally in the OpenShift cluster. The most common reason for this is that you do not wish to expose the image on a publicly-facing container registry. In these cases, you can use an OpenShift Build object to build the container image. To do this, create a BuildConfig object in the redhat-ods-applications namespace of the OpenShift cluster where RHODS is deployed. Ensure that the build outputs the image to the ImageStream object that was created above.

1. In your preferred text editor, name your BuildConfig object 'users-rstudio-streamlit_buildconfig.yaml' and copy and paste the following lines:
    ```
    kind: BuildConfig
    apiVersion: build.openshift.io/v1
    metadata:
        name: users-rstudio-streamlit
        namespace: redhat-ods-applications
        labels:
            opendatahub.io/build_type: notebook_image 
        opendatahub.io/notebook-image: users-rstudio-streamlit
    spec:
        source: 
            type: Git
            git:
            uri: https://github.com/rh-ai-services/Custom-Notebook-Images-Best-Practices.git
           contextDir: r-notebook/container
  
        strategy:
            type: Docker
            dockerstrategy:
            dockerfilePath: Dockerfile
        output: 
            to:
            kind: ImageStreamTag
            name: users-rstudio-streamlit:v1
    ```

    The official documentation for configuring BuildConfig objects can be found [here](https://docs.openshift.com/container-platform/4.9/cicd/builds/understanding-buildconfigs.html).

2. To save your definition to a file and then create it, run the following command:
     ```
     oc -n redhat-ods-applications apply r-notebook-with-streamlit_buildconfig.yaml
     ```
3. You can view the newly created BuldConfig object on the in the list of BuildConfig objects on the OpenShift cluster by running:
    ```
    oc -n redhat-ods-applications get buildconfig
    ```

4. To trigger the build, run the following command:
    ```
    oc -n redhat-ods-applications apply -f start-build users-rstudio-streamlit
    ```

## Viewing the Build on the OpenShift Console. 
1. To track the progress of our build, head over the the OpenShift console and navigate to "Builds" on the left side bar. In the dropdown menu, click on "Builds" 

    ![builds](/img/on-cluster-build-builds.png)

2. Click on the name of your build and then click on the "Logs" tab. You will know if your build was successfully completed if at the end of the log, it says:
 **Push successful**

    ![logs](/img/on-cluster-build-logs.png)

3. Once your build is complete, you should be able to see it on RHODS listed in "Notebook Images" and view it as one of the JupyterHub notebook options.

    ![web-ui](/img/on-cluster-build-web-ui.png)

    ![jupyter-nb](/img/on-cluster-build-jupyternb.png)

## Sanity Check
1. On JupyterHub, select your notebook image and launch your newly built RStudio Notebook with Streamlit.

2. Since this is an RStudio notebook image, do a sanity check by running the following commands:

    ```
    print('hello world!')
    SessionInfo()
    ```
    ![rstudio](/img/r-studio-sanity-check.png)

3. Within your R notebook, open the terminal and run the following command to check if Strealit was properly installed:

    ```
    streamlit hello
    ```

    ![streamlit](/img/streamlit-sanity-check.png)

4. If your outputs look normal, congratulations, you have created your first custom notebook image on RHODS!






