+++
title = "Creating Jupyterhub on Google Cloud Platform (GCP)"
date = "2018-10-24T13:51:46+02:00"
tags = ["jupyterhub", "gcp"]
categories = ["jupyterhub"]
draft = false
+++

This is a short tutorial on installing Jupyterhub on Kuberntes using the Cloud Shell and the Carme package. This is convenient as you don't have to install anything (Google CLI/Helm) locally.

<a style="cursor:pointer" onclick='javascript:window.open("https://console.cloud.google.com/cloudshell/", "_blank", "toolbar=no,scrollbars=yes,resizable=yes,menubar=no,location=no,status=no")'><image src="https://gstatic.com/cloudssh/images/open-btn.png" /></a>

## Why JupyterHub on Kubernetes

Let's say that you want to do a training session on Deep Learning for everyone here.  Well first we could spend 45 minutes getting everyones laptop setup to be able to be able to run the sample code.  Alternately, we could use cloud based computing with GCP, Kubernetes, Jupyterhub, and Carme.


## Getting Started
You must first log onto the GCP web portal and have an active subscription. Click on the button below to launch the cloud shell.



We will start by installing Carme on the Azure Shell.

```
wget https://raw.githubusercontent.com/CarmeLabs/carme/master/scripts/get/get_carme_gcloud.sh
bash get_carme_gcloud.sh
```



To verify that Carme is installed and working type `carme --help`. This will give an overview of the carme package.

```
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

```

## Create Your First Carme Project
The next thing that we are going to do is to create a new Carme project based on the `az-z2jh` package. The `az-z2jh` project includes the additional packages for creating a Kubernetes cluster (`az-cluster`), installing JupyterHub (`jupyterhub`), and customizing the singleuser container ('jupyter-cpu'). You can find the documentation for the [az-z2jh project here](https://docs.carme.ai/packages/az-z2jh.html). The az-z2jh package is based on the [Zero to JupyterHub project](https://zero-to-jupyterhub.readthedocs.io/en/latest/), which has extensive documentation on launching JupyterHub on a variety of cloud platforms. Carme just makes it a bit easier.

This will create the carme project:
```
carme new mycluster --package gcp-z2jh
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
carme cmd gcp-cluster show_config --yes
```
### Update your Email
You must now update your email in the file:
```
nano ./config/gcp-cluster.yaml
```

### Enable the Kubernetes API
In order to be able to launch Kubernetes you will have to enable the Kubernetes API.

Click the link
[https://console.cloud.google.com/apis/library/container.googleapis.com?q=Kubernetes%20Engine%20API&id=1def4230-f361-4931-b386-576c62b90799](https://console.cloud.google.com/apis/library/container.googleapis.com?q=Kubernetes%20Engine%20API&id=1def4230-f361-4931-b386-576c62b90799)

### Lauch the Cluster
Carme includes ways of templating bash commands so that the required commands are combined where necessary with the values for the config file.  This makes it easy to interactively enter a number of different commands.

```
carme cmd gcp-cluster create_all
```

This will ask you to confirm a number of commands used to launch the Cluster.  Press `y` to confirm each one.

The last command will install the cluster. This could take you up to 10 minutes to launch.


### Verify your Cluster
The `kubectl` command is the native way of controlling your cluster. Let's go ahead and verify that this works. The get node command will list out all of the servers and the kubernetes versions.

```
kubectl get node
```


Helm version will show that helm installed both locally and on the cluster.  The versions also need to be the same.
```
helm init
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
carme cmd jupyterhub install_all
```

That is it! This should return a URL.

```
carme cmd jupyterhub install_all
```

### Configuring JuptyerHub
There are a number of different configurations that you can make to JupyterHub. In particular, adjusting to the following enables a number of useful features including:
- SSL
- Github Authentication
- A customized Docker container
- A common repository which is cloned for all users
- JupyterLab by default

Of course the `secretToken`, `clientSecret`, and `clientId` fields are obscured.

```
proxy:
  secretToken: "*****************************"
  https:
    hosts:
    - lab.analyticsdojo.com
    letsencrypt:
      contactEmail: kuruzj@rpi.edu
auth:
  type: github
  github:
    clientId: "**********************"
    clientSecret: "***********************************"
    callbackUrl: "https://lab.analyticsdojo.com/hub/oauth_callback"
  admin:
    users:
      - jkuruzovich
      - jiangl4
singleuser:
  image:
    name: carmelabs/jupyter-cpu-r
    tag: c56019b
  defaultUrl: "/lab"
  lifecycleHooks:
    postStart:
      exec:
        command: ["gitpuller", "https://github.com/rpi-techfundamentals/fall2018-materials", "master", "materials-fa"]
hub:
  extraConfig: |-
    c.Spawner.cmd = ['jupyter-labhub']
```

After changing your configuration file you would just need to upgrade JupyterHub using hte following command.

```
carme cmd jupyterhub upgrade
```


## Deep Learning with Pytorch
We will have some waiting time between using our cluster. Let's use that time to go over some Deep Learning with Pytorch.

[Click this link to try the deep learning tutorial.](http://lab.analyticsdojo.com/hub/user-redirect/git-pull?repo=https://github.com/CarmeLabs/GCP-tutorial&branch=master&supPath=GCP-tutorial&app=lab)
