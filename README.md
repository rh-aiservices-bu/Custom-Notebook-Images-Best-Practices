<<<<<<< HEAD
# custom-images

* Some starting instructions on ways a customer could create/leverage custom notebook images: [Building Custom Images](Building_Custom_Notebook_Images.md)... this is a bit out of date now that there is a GUI.

## simple diagram to set the stage

`to be whiteboarded`

## Introductions to the various options

* build on laptop and push to quay.io. See the [laptop-build-howto](docs/laptop-build-howto.md) page.
* on-cluster build, stored only in cluster
* on-cluster build, stored on quay.io, with a pipeline that makes the updates (stretch goal)

Include links to `./docs/` for more details/examples for each option

## Building Jupyter Notebook Images ... considerations

* starting image
* adding software
* user id / root considerations
* versionning / pinning of versions
* entrypoints
* persistent storage
* if a sub-process is using a different port

## Reading/reference materials

* <https://github.com/guimou/custom-notebooks>
* <https://github.com/rh-aiservices-bu/byoni>





=======
# R Notebook

Custom notebook based on RHODS standard data science notebook, adding:

* R
* R Kernel
* R Studio
* Streamlit
>>>>>>> cfa0a5dc159fb054a27fa932302f0325d3177700
