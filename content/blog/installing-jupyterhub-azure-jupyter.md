+++
title = "Creating Jupyterhub on Azure using a Jupyter Notebook"
date = "2018-07-16T13:50:46+02:00"
tags = ["jupyterhub", "azure"]
categories = ["jupyterhub"]
+++


# This tutorial is currently not fully working. It launches the Cluster but the installation of Jupyterhub is not working.
Help solve the issue [here](https://github.com/CarmeLabs/packages/issues/1).

This is a short tutorial on installing Jupyterhub using a Jupyter notebook.  You must have Azure the Azure CLI, Helm, and Conda installed locally.

See instructions for installing the [Azure CLI here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
See instructions for installing the [Helm Client here](https://github.com/kubernetes/helm/blob/master/docs/install.md).
See instructions for installing the [Conda here](https://conda.io/docs/user-guide/install/index.html).

We will start by installing carme.
```
wget https://raw.githubusercontent.com/CarmeLabs/carme/master/scripts/get/get_carme_conda.sh
source get_carme_conda.sh
```

This will activate the virtual environment with Carme installed. If in the future you log in again this will not be installed but not active.  To activate the environment, just use the commands below:
```
source activate carme-env
```

To verify that Carme is installed and working type `carme --help`. This will give an overview of the carme package.


Usage: carme [OPTIONS] COMMAND [ARGS]...

Options:
  --version  Show the version and exit.

  --help     Show this message and exit.

Commands:
  build     Build project docker images.
  cleanup   Delete and remove all images.
  cmd       Runs commands from the commands folder.
  convert   Launch Jupyter Notebook (using Docker).
  git       Initializes git and performs initial...
  lab       Launch Jupyter Notebook (using Docker).
  new       Creates a new carme project in project_dir or...
  notebook  Launch Jupyter Notebook (using Docker).
  package   Manage packages on the project.
  save      A simler alias to git commit and push.
  start     Launch all installed applications.
  stop      Stop (and optionally remove, --remove) all...


## Create Your First Carme Project
The next thing that we are going to do is to create a new Carme project based on the `az-z2jh` package. The `az-z2jh` project includes the additional packages for creating a Kubernetes cluster (`az-cluster`), installing JupyterHub (`jupyterhub`), and customizing the singleuser container ('jupyter-cpu'). You can find the documentation for the [az-z2jh project here](https://docs.carme.ai/packages/az-z2jh.html). The az-z2jh package is based on the [Zero to JupyterHub project](https://zero-to-jupyterhub.readthedocs.io/en/latest/), which has extensive documentation on launching JupyterHub on a variety of cloud platforms. Carme just makes it a bit easier.

This will create the carme project:
```
carme new mycluster --package az-z2jh
```

Now change to the Carme project directory and look at the directory structure:
```
cd mycluster
ls
```

From here launch the Jupyter using:

```
jupyter lab
```

Or `jupyter notebook` if you prefer.

The notebook for launching Jupyterhub is under:
code/notebooks/az-cluster.ipynb

GO THERE NOW!  The rest of this tutorial is in the notebook.  Just press play to continue down the process of installing.
