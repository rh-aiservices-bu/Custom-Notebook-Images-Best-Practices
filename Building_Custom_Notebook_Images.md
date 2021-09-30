# Building Custom Notebook Images

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Instructions](#instructions)
  * [ImageStream Creation](#imagestream-creation)
  * [BuildConfig Creation (Optional)](#buildconfig-creation-optional)
* [Practical Examples](#practical-examples)
  * [Using a pre-built notebook image from quay.io](#using-a-pre-built-notebook-image-from-quayio)
  * [Changing one of the default notebook images](#changing-one-of-the-default-notebook-images)
  * [Expected results](#expected-results)
  * [Cleaning up the dummy examples](#cleaning-up-the-dummy-examples)

## Overview

Red Hat OpenShift Data Science (RHODS) comes bundled with a curated set of Jupyter notebook images tailored to various use cases. This document outlines the process by which RHODS customers can define additional custom notebook images to be available for use.

## Prerequisites

* Familiarity with OpenShift Build Configs and Image Streams
* Familiarity with the OpenShift console and/or oc command line client
* Access permissions to the OpenShift cluster on which RHODS is deployed. At a minimum, Edit level permissions must be granted in the redhat-ods-applications namespace. Full cluster admin level access can be granted in the [OpenShift cluster manager interface](http://cloud.redhat.com/openshift) by the cluster owner.
* An existing OpenShift build config that produces a valid Jupyter notebook image or an existing image hosted in a container image registry that can be reached by the OpenShift cluster where RHODS is deployed


## Instructions

Perform the following tasks to add your custom image to JupyterHub. Once the ImageStream has been created, it will appear in the JupyterHub spawner interface upon refreshing the page (no restart of the JuptyerHub service is necessary). If you have chosen to create a BuildConfig object to create the image, selection of the image will be disabled in the JupyterHub interface until the build has completed successfully.

### ImageStream Creation

In order for a custom image to be available in RHODS JupyterHub, an ImageStream must be created in the redhat-ods-applications namespace of the OpenShift cluster where RHODS is deployed.

A minimal example of an ImageStream is as follows (sections in all caps should be replaced):

```yaml
kind: ImageStream
apiVersion: image.openshift.io/v1
metadata:
  name: IMAGE_STREAM_NAME
  namespace: redhat-ods-applications
  annotations:
    opendatahub.io/notebook-image-desc: >-
      SOME USER FACING DESCRIPTION OF THE IMAGE
    opendatahub.io/notebook-image-name: USER FACING IMAGE NAME
  labels:
    opendatahub.io/notebook-image: 'true'
    component.opendatahub.io/name: jupyterhub
    opendatahub.io/component: 'true'
spec:
  lookupPolicy:
    local: true
  tags:
    - name: SOME_TAG_NAME
      annotations:
        # A DICTIONARY OF PYTHON PACKAGES INCLUDED IN
        # THE IMAGE. THIS INFORMATION WILL BE
        # USER-FACING. EXAMPLE BELOW:
        opendatahub.io/notebook-python-dependencies: >-
          [{"name":"JupyterLab","version": "3.0.16"}, {"name":
          "Notebook","version": "6.4.0"}]
        # A DICTIONARY OF COMPONENTS AND VERSIONS IN THE IMAGE
        opendatahub.io/notebook-software: '[{"name":"Python","version":"v3.8.6"}]'
      # INCLUDE THIS FROM BLOCK IF IMPORTING FROM A PUBLIC CONTAINER REGISTRY
      from:
        kind: DockerImage
        name: CONTAINER_PULL_URL
```

Official documentation on the format and specification of the ImageStream definition can be found [here](https://docs.openshift.com/container-platform/4.8/rest_api/image_apis/imagestream-image-openshift-io-v1.html).

After modifying required fields in the above example, save the definition to a file and then create it by running:

```bash
oc apply -f PATH_TO_IMAGESTREAM_DEFINITION.YAML
```


### BuildConfig Creation (Optional)

In some cases, you may wish to build the notebook container image locally in the OpenShift cluster. The most common reason for this is that you do not wish to expose the image on a publicly-facing container registry. In these cases, you can use an OpenShift Build object to build the container image. To do this, create a BuildConfig object in the redhat-ods-applications namespace  of the OpenShift cluster where RHODS is deployed. Ensure that the build outputs the image to the ImageStream object that was created above.

A minimal example of a BuildConfig is as follows (sections in all caps should be replaced):

```yaml
kind: BuildConfig
apiVersion: build.openshift.io/v1
metadata:
  name: BUILD_CONFIG_NAME
  namespace: redhat-ods-applications
  labels:
    opendatahub.io/build_type: notebook_image
    opendatahub.io/notebook-name: USER_FACING_NOTEBOOK_NAME_MATCHING_IMAGE_STREAM
spec:
  output:
    to:
      kind: ImageStreamTag
      # THIS MUST MATCH THE IMAGE STREAM AND TAG
      # NAME USED IN THE IMAGE STREAM THAT WAS CREATED
      name: 'IMAGE_STREAM_NAME:SOME_TAG_NAME'
  strategy: {} #CHANGEME
  source: {} #CHANGEME
```

Official documentation on the format and specification of the BuildConfig definition can be found [here](https://docs.openshift.com/container-platform/4.8/rest_api/workloads_apis/buildconfig-build-openshift-io-v1.html).

After modifying required fields in the above example, save the definition to a file and then create it by running:

```bash
oc apply -f PATH_TO_BUILDCONFIG_DEFINITION.YAML
```

## Practical Examples

### Using a pre-built notebook image from quay.io

* Viewing existing ImageStreams:

```bash
oc -n redhat-ods-applications get imagestreams
```

* Create the ImageStream

```yaml
oc apply -f - <<EOF
---
kind: ImageStream
apiVersion: image.openshift.io/v1
metadata:
  name: dummy-notebook-01
  namespace: redhat-ods-applications
  annotations:
    opendatahub.io/notebook-image-desc: Dummy Notebook Image 01 - Full Description
    opendatahub.io/notebook-image-name: Dummy Notebook Image 01
  labels:
    opendatahub.io/notebook-image: 'true'
    component.opendatahub.io/name: jupyterhub
    opendatahub.io/component: 'true'
spec:
  lookupPolicy:
    local: true
  tags:
    - name: v01
      annotations:
        opendatahub.io/notebook-python-dependencies: >-
          [{"name":"DummyPackage","version": "v1"},
          {"name":"OtherDummyPackage","version": "v2"}]
        opendatahub.io/notebook-software: '[{"name":"dummy notebook image","version":"v 01"}]'
      from:
        kind: DockerImage
        name: quay.io/llasmith/spark-notebook:spark-2.3.2_hadoop-2.8.5     ## this image is available on quay.io
EOF
```

* At this stage, you should see a new image available in JupyterHub


### Changing one of the default notebook images

* Determine which image to start from. Here, the minimal notebook.

```bash
oc -n redhat-ods-applications get imagestreams minimal-gpu
```

* Creating an ImageStream for the new version:

```yaml
oc apply -f - <<EOF
---
kind: ImageStream
apiVersion: image.openshift.io/v1
metadata:
  name: dummy-notebook-02
  namespace: redhat-ods-applications
  annotations:
    opendatahub.io/notebook-image-desc: Dummy Notebook Image 02 - Full Description
    opendatahub.io/notebook-image-name: Dummy Notebook Image 02
  labels:
    opendatahub.io/notebook-image: 'true'
    component.opendatahub.io/name: jupyterhub
    opendatahub.io/component: 'true'
spec:
  lookupPolicy:
    local: true
  tags:
    - name: v01
      annotations:
        opendatahub.io/notebook-python-dependencies: >-
          [{"name":"SoftwarePackage A","version": "v1"}, {"name":"SoftwarePackage B","version": "V2"}]
        opendatahub.io/notebook-software: '[{"name":"dummy notebook image","version":"v 02"}]'
EOF
```

* Creating a BuildConfig for it:

```yaml
oc apply -f - <<EOF
---
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: dummy-notebook-02-buildconfig
  namespace: redhat-ods-applications
  labels:
    opendatahub.io/build_type: notebook_image
    opendatahub.io/notebook-name: dummy-notebook-02
spec:
  source:
    dockerfile: |
      ## You still need a FROM line in this file, but it will be ignored because of the from: below
      FROM dummy
      RUN touch /tmp/dummy02-marker-file.txt
      RUN ls -al /tmp/dum*
  strategy:
    type: Docker
    dockerStrategy:
      ## this replaces the FROM in the dockerfile.
      from:
        kind: ImageStreamTag
        namespace: redhat-ods-applications
        name: 'minimal-gpu:py3.8-cuda-11.0.3'
  successfulBuildsHistoryLimit: 2
  failedBuildsHistoryLimit: 2
  runPolicy: Serial
  output:
    to:
      kind: ImageStreamTag
      name: 'dummy-notebook-02:v01'
  triggers:
    - type: ImageChange   ## if the source image (minimal-gpu) gets rebuilt, it will trigger a rebuild
      imageChange: {}
EOF
```

* Triggering the build

```bash
oc -n redhat-ods-applications start-build dummy-notebook-02-buildconfig
```

* Once the build completes, you should be able to see the second new notebook image

### Expected results

Your JupyterHub Spawner should look like:

  ![new notebook images](img/new_nb_images.png)

### Cleaning up the dummy examples

* Delete the first custom notebook image:

```bash
oc -n redhat-ods-applications delete imagestream dummy-notebook-01
```

* Delete the second custom notebook image:

```bash
oc -n redhat-ods-applications delete imagestream dummy-notebook-02
oc -n redhat-ods-applications delete BuildConfig dummy-notebook-02-buildconfig
```
