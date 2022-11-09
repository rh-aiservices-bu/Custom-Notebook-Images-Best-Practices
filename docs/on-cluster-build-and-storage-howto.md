# Jupyter Notebook - on cluster build and storage

In this document, we will go over a step-by-step walkthrough to build and store a notebook container image on the OpenShift cluster by updating an R notebook image with Streamlit.

## Table of Contents

* [Requirements](#requirements)
* [Prerequisites](#prerequisties)
* [Defining an ImageStream Object in YAML](#defining-an-imagestream-object-in-yaml)
* [Defining a BuildConfig object in YAML](#defining-a-buildconfig-object-in-yaml)
* [Viewing the Build on OpenShift](#viewing-the-build-on-the-openshift-console)
* [Sanity Checks](#sanity-checks)

## Requirements
* Admin level access to the OpenShift cluster where RHODS is deployed
* Your preferred text editor
* OpenShift Client i.e oc (Installation instructions can be [here](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html))
   
    * More information regarding oc commands can be found [here](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/developer-cli-commands.html) or alternatively, run the following command in your terminal:
     ```
    oc --help
     ```
## Prerequisites
1. In your working directory, clone this git repository.
    ```
    git clone https://github.com/rh-aiservices-bu/Custom-Notebook-Images-Best-Practices.git
    ```
2. Go into this project's directory.
    ```
    cd Custom-Notebook-Images-Best-Practices/r-notebook-streamlit/deploy/
    ```
    **All of the following steps will take place in this directory.**

3. Head over to the [Red Hat OpenShift Console](https://console-openshift-console.apps.pilot.j61u.p1.openshiftapps.com/dashboards) and copy the login command. Paste and run it in your terminal.

    ![login](/img/copy-login-command.png)


## Defining an Imagestream Object in YAML

In order for a custom notebook image to be available on RHODS JupyterHub, an ImageStream must be created in the `redhat-ods-applications` namespace of the OpenShift cluster where RHODS is deployed.

For the purposes of this tutorial, we will update one of the R-Studio default notebook images to include Streamlit. You can find the original ImageStream in the same directory [here](https://github.com/rh-aiservices-bu/Custom-Notebook-Images-Best-Practices/blob/main/r-notebook/deploy/odh-minimal-data-science-r-notebook_image-stream.yml).

1. In your preferred text editor, name your ImageStream object `rstudio-streamlit_imagestream.yaml` and update it accordingly:
    ```
    kind: ImageStream
    apiVersion: image.openshift.io/v1
    metadata: 
        name: users-rstudio-streamlit
        annotations: 
            opendatahub.io/notebook-image-desc: JupyterLab minimal notebook image with R, IRKernel, RStudio and Streamlit.
            opendatahub.io/notebook-image-name: user's rstudio streamlit
            opendatahub.io/notebook-image-order: "200"
            opendatahub.io/notebook-image-url: "https://github.com/rh-aiservices-bu/Custom-Notebook-Images-Best-Practices/tree/main/r-notebook"
        labels: 
            app.kubernetes.io/created-by: byon
            component.opendatahub.io/name: jupyterhub
            opendatahub.io/component: "true"
            opendatahub.io/notebook-image: "true"
    spec: 
        lookupPolicy: 
            local: true
        tags: 
            - name: v1
              annotations: 
                opendatahub.io/notebook-python-dependencies: >-
                  [{"name":"R","version":"4.0.5"},{"name":"RStudio","version":"2022.02.0-443"}, {"name": "Streamlit","version":"1.11.1"}]
                opendatahub.io/notebook-software: >-
                  [{"name":"BYO Notebook Image - R","version":"4.0.5"},{"name":"RStudio","version":"2022.02.0"},{"name":"Python","version":"3.9.10"}, {"name": "Streamlit","version":"1.11.1"}]
              referencePolicy: 
                  type: Local
    ```
    The official documentation for configuring ImageStream objects can be found [here](https://docs.openshift.com/container-platform/4.8/rest_api/image_apis/imagestream-image-openshift-io-v1.html).

2. To save your definition to a file and then create it, run the following command:

    ```
    oc apply -n redhat-ods-applications -f rstudio-streamlit_imagestream.yaml
    ```
3. You can view your newly created ImageStream object in the in the list of ImageStream objects on the OpenShift cluster by running:
   
   ```
   oc -n redhat-ods-applications get imagestreams
   ```

## Defining a BuildConfig Object in YAML 

1. In your preferred text editor, name your BuildConfig object `rstudio-streamlit_buildconfig.yaml` and copy and paste the following lines:

    ```
    kind: BuildConfig
    apiVersion: build.openshift.io/v1
    metadata: 
      name: users-rstudio-streamlit-buildconfig
      namespace: redhat-ods-applications
      labels: 
        opendatahub.io/build_type: notebook_image
        opendatahub.io/notebook-image: users-rstudio-streamlit
    spec: 
      source: 
        type: Git
        git: 
          uri: https://github.com/rh-aiservices-bu/Custom-Notebook-Images-Best-Practices.git
        contextDir: r-notebook-streamlit/container
      strategy: 
        type: Docker
        dockerstrategy: 
          dockerfilePath: Dockerfile
      output: 
        to:
          kind: ImageStreamTag
          name: 'users-rstudio-streamlit:v1'
    ```

    The official documentation for configuring BuildConfig objects can be found [here](https://docs.openshift.com/container-platform/4.9/cicd/builds/understanding-buildconfigs.html).

2. To save your definition to a file and then create it, run the following command:
     ```
     oc apply -n redhat-ods-applications -f rstudio-streamlit_buildconfig.yaml
     ```
3. You can view the newly created BuldConfig object on the in the list of BuildConfig objects on the OpenShift cluster by running:
    ```
    oc -n redhat-ods-applications get buildconfig
    ```

4. To trigger the build, run the following command:
    ```
    oc -n redhat-ods-applications start-build users-rstudio-streamlit
    ```

## Viewing the Build on the OpenShift Console. 
1. To track the progress of our build, head over the the OpenShift Console and navigate to `Builds` on the left side bar. In the dropdown menu, click on `Builds`. 

    ![builds](/img/on-cluster-build-builds.png)

2. Click on the name of your build and then click on the `Logs` tab. You will know if your build was successfully completed if at the end of the log, it says:
 **Push successful**

    ![logs](/img/on-cluster-build-logs.png)

3. Once your build is complete, you should be able to see it on RHODS listed in `Notebook Images`. Make sure you enable it to view it as one of the JupyterHub notebook options.

    ![web-ui](/img/on-cluster-build-web-ui.png)

    ![jupyter-nb](/img/on-cluster-build-jupyternb.png)

## Sanity Checks
1. On JupyterHub, select your notebook image and launch your newly built RStudio notebook image with Streamlit.

2. Complete a sanity check by running the following commands:

    ```
    print('hello world!')
    SessionInfo()
    ```
    ![rstudio](/img/r-studio-sanity-check.png)

3. Within your R notebook, open the terminal and run the following command to check if Streamlit was properly installed:

    ```
    streamlit hello
    ```

    ![streamlit](/img/streamlit-sanity-check.png)

4. If your outputs look normal, congratulations, you have created your first custom notebook image on RHODS!






