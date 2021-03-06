+++
title = "Creating Jupyterhub on Azure with Cloud Shell"
date = "2018-07-18T13:50:46+02:00"
tags = ["jupyterhub", "azure"]
categories = ["jupyterhub"]
+++

This is a short tutorial on installing Jupyterhub on Azure using the Cloud Shell and the Carme package. This is convenient as you don't have to install anything (Azure CLI/Helm) locally.

You must first log onto the Azure web portal and have an active subscription. Click on the button below to launch the cloud shell.


<a style="cursor:pointer" onclick='javascript:window.open("https://shell.azure.com", "_blank", "toolbar=no,scrollbars=yes,resizable=yes,menubar=no,location=no,status=no")'><image src="/img/clshell.png" /></a>

We will start by installing Carme on the Azure Shell.

```
wget https://raw.githubusercontent.com/CarmeLabs/carme/master/scripts/get/get_carme.sh
source get_carme.sh
```

This will activate the virtual environment with Carme installed. If in the future you log in again this will not be installed but not active.  To activate the environment, just use this easy script.
```
source activate.sh
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

The default installation includes some scaffolded commands and images.  We won't be using all of it here.

## Creating the Kubernetes Cluster
### Verifying Cluster Configuration Settings
Before we can launch the cluster, let's go ahead and look at the configuration associated with the cluster. The values here are consistent with the [Zero to Jupyterhub](https://zero-to-jupyterhub.readthedocs.io/en/latest/) documentation.

The `--yes` just doesn't ask for confirmation before running the command.

```
carme cmd az-cluster show_config --yes
```

### Lauch the Cluster
Carme includes ways of templating bash commands so that the required commands are combined where necessary with the values for the config file.  This makes it easy to interactively enter a number of different commands.

```
carme cmd az-cluster create_all --yes
```

This will ask you to confirm a number of commands used to launch the Cluster.  Press `y` to confirm each one.

The last command will install the cluster. This could take you up to 10 minutes to launch.


### Verify your Cluster
The `kubectl` command is the native way of controlling your cluster. Let's go ahead and verify that this works. The get node command will list out all of the servers and the kubernetes versions.

```
kubectl cluster-info
```

```
kubectl get nodes
```

Helm version will show that helm installed both locally and on the cluster.  The versions also need to be the same.
```
helm version
```

## Install JupyterHub
The JupyterHub installation process involves a similar process to the creation of the cluster. Let's start by verifying the configuration

### Verifying JupyterHub Configuration Settings
Before we can install Jupyterhub, let's go ahead and look at the configuration associated with the cluster. The values here are consistent with the [Zero to Jupyterhub Helm documentation](https://zero-to-jupyterhub.readthedocs.io/en/latest/setup-helm.html) documentation.

The `--yes` just doesn't ask for confirmation before running the command.

```
carme cmd jupyterhub show_config --yes
```

JupyterHub requires us to generate some configuration.

```
carme cmd jupyterhub install_all --yes
```

If you would like to confirm each command before running it, you should just skip the `--yes` command.

This second to last command will wait for 5 minutes for the public IP to show up.

The last command will indicate the IP address!

```
EXTERNAL-IP
xx.xx.xx.xx
```

OK....let us go ahead and clean everything up.
```
carme cmd az-cluster delete_group
```
More tutorials coming soon.  [Join the Slack channel](https://join.slack.com/t/carmelabs/shared_invite/enQtNDAxMDE2MjU0NzA5LTJmMGVlM2I5Zjc4Yzk2NzhjYTRlMWVhMTZlMTYzMGMyNmM0NzE1ZTMwZWFjZGUxNGRkMTc2ZjJjNTVlYThkMTQ) to stay up to date!
