# Podman Tutorial
Podman Tutorial Zero to Hero - https://www.youtube.com/watch?v=YXfA5O5Mr18

# What is Podman?

Podman is a daemon-less, open-source Linux-native tool designed to make it easy to find, run, build, share, and deploy applications using OCI (Open Container Initiative) containers 
and container images.  

The OCI aims to create industry standards around container formats and runtimes.  

Podman offer many features:
- Rootless containers (more secure than Docker containers)
- Multi-container pods (similar to K8s pods, another feature that Docker does not offer)
- Daemonless (another difference with Docker)

With Podman, you can create multi-container pods locally, and export them as a K8s manifest.  
Being daemonless means that Podman interacts directly with `runc`, a CLI tool for running containers on Linux.  

## Podman Desktop

is a graphical interface for managing Podman containers and Kubernetes (K8s) from your desktop.  

---

# How to install Podman

## Install Podman on Ubuntu

- `sudo apt update`
- `sudo apt install podman`

We'll use an **Ubuntu** machine until the end of this tutorial.

## Podman on non-Linux machines

As containers are a Linux technology, you can't run them on Mac or Windows.  
The only way to run containers on these operating systems is to rely on a Linux VM.  
For this reason, after you have installed Podman, you need to run `podman machine init` then `podman machine start` in order to run Podman on non-Linux systems.  

- The default machine name will be `podman-machine-default`.  
- Only one Podman VM can be active.

---

# Custom Container Registry Configuration

One of the most popular container registries is **Docker Hub**.  

But thanks to the **Podman Container Registry Configuration**, we can use other registries such as:
- GitHub Container Registry (**ghcr**)
- RedHat Container Registry (**quay**)
- or your company's private registry

The file for the Podman custom container registry configuration is located in `/etc/containers/registries.conf`.  
Podman uses this file when you pull or push an image, and every time you attempt to connect to a container registry.  

Instead of changing the default registry configuration, you can have a different config file for your user.  
For that, just create a separate config file in `$HOME/.config/containers/registries.conf`

- run `cd` to place yourself in your home directory, which is represented by `~` or `$HOME`
- create the 'containers' folder: `mkdir .config/containers`
- create and edit the config file with **Vim**: `vim .config/containers/registries.conf`
- press `i` to enter insert mode
  
![image](https://github.com/user-attachments/assets/b196afc3-a2c5-48d4-8153-1a2f99f2b2a2)

- press `Esc` to exit the insert mode, type `:wq` and press `Enter` to write and quit.

## The `podman search` command

Basic syntax: `podman search <image_name>`  
This command searches a registry, or a list of registries, for images that match the name we specify.  

We can specify which registry to search by prefixing the search term: `podman search docker.io/library/fedora`  
By default, all unqualified searches will use the `unqualified-search-registries` list from our custom config file.  

---

# Pulling a container image 

- to pull an image from a registry and store it locally: `podman pull <image_name>`
  - when we don't specify any tag, podman tries to pull the latest version
- to list images in local storage: `podman images`

![image](https://github.com/user-attachments/assets/7151e8ce-9c07-411d-a121-e493efa43cb3)  






@13/60
