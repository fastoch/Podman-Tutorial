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
  
![image](https://github.com/user-attachments/assets/068cbf09-728c-4848-831b-7f76e75b9695)

- press `Esc` to exit the insert mode, type `:wq` and press `Enter` to write and quit.

## The `podman search` command

Basic syntax: `podman search <image>`  
This command searches a registry, or a list of registries, for images that match the name we specify.  

We can specify which registry to search by prefixing the search term: `podman search docker.io/library/fedora`  
By default, all unqualified searches will use the `unqualified-search-registries` list from our custom config file.  
![image](https://github.com/user-attachments/assets/6cd4ce9c-2932-432f-864c-f02a696fa39d)

---

# Pulling a container image 

- to pull an image from a registry and store it locally: `podman pull <image_URL>`
  - when we don't specify any tag, podman tries to pull the latest version
- to list images in local storage: `podman images`

![image](https://github.com/user-attachments/assets/7151e8ce-9c07-411d-a121-e493efa43cb3)  

# Running a container from an image

- `podman run -it <image>`
  - the `-it` flags enable an _interactive terminal_ session with the container
- `podman run -it --rm <image>`
  - the `--rm` option deletes the container on exit, **recommended** to avoid filling up your filesystem with ephemeral containers

To display all running containers: `podman ps`  
To see all containers and their current status: `podman ps -a`  

We can use the `--name` flag when creating a container. If we don't specify a container name, podman will give it a random name.  

---

# Working with containers

Some additional useful commands:
```bash
podman --help
podman run --name <container_name> -p <host_port>:<container_port> <image>
podman start <container_name_or_ID>
podman stop <container_name_or_ID>
podman inspect <container_name_or_ID>
podman port <container_name_or_ID>
podman rm <container_name_or_ID>
podman images
podman rmi <image ID>
```
- `podman --help` to show information about all podman commands
- `podman inspect` provides all the low-level information about the container in JSON format
- `podman port` lists all port mappings for the specified container
- `podman rm` removes the specified container
- `podman images` displays all images available on the host
- `podman rmi` removes the specified image

Let's put the previous commands into practice:
```bash
podman search nginx
podman run --name pdm-nginx -p 8080:80 nginx
```
- Note that the `podman run` command will only pull the image if it's not already present on your system
- Using the `--name` flag can lead to **naming conflicts**, it's best not to use it and let podman assign random names
- if you haven't used `-d` to run the container in detached mode, press `Ctrl + C` to stop it.
- when using the `podman start` command, your container will run in detached mode by default

## Accessing an Nginx container from the host's browser

Nginx is a very popular versatile web server.  
Now that we have started a podman nginx container with port mapping, we can access our web server via:
- a web browser:

![image](https://github.com/user-attachments/assets/069fb6bb-39a4-44b4-bc41-1c0075388ee0)

- the `curl` utility:

![image](https://github.com/user-attachments/assets/c23b9ee5-c772-4279-8545-44e79a5bcd00)

---

# Building an image

The whole point of **containerization** is to:
- make our applications run the same on any system (OS-independent)
- make our apps portable and easy to deploy
- allow us to deploy fast, and recover as fast when things go wrong
- run our apps in an isolated environment, which adds a layer of security

Up until now, we've been working with existing images such as Nginx or Busybox, but we can create our own container images.  
To do that, we need to use a specific file (`Dockerfile`) that will allow us to containerize our application and build an image out of it.  

Podman's equivalent to a `Dockerfile` is a `Containerfile`. Both Containerfiles and Dockerfiles use the same syntax internally,  
allowing us to build container images using Podman without needing to modify our existing Dockerfiles.  
Podman supports building images from either a `Containerfile` or a `Dockerfile`.  

A `Containerfile` is a script that contains instructions on how to build a container image.  
Once we have our `Containerfile`, we can use the `podman build` command to build our container image.  
The `podman build` command will build the image from the Dockerfile/Containerfile located in the current directory.  

Once the image has been built, we can use the `podman run` command to run the container from this image.  
So the steps are:
- write the dockerfile/containerfile
- build the container image from that containerfile
- run the container from that image

In this tutorial, we'll create a simple application in Go that exposes a web service with a single endpoint.  
The endpoint responds with the message "Hello Podman from Go!".  

## Creating our simple app

- first, make sure that Go is installed by running `go version`
- if not, install it via `sudo apt install golang-go` (if running a Debian-based Linux distro)
- now, let's create a **project folder**: `mkdir pdm-golang`
- make it the working directory: `cd pdm-golang`
- to initialize our Go project: `go mod init pdm-golang`

### About the `go mod init` command

The primary purpose of `go mod init` is to initialize a Go module in our project.  
A Go module is a collection of related Go packages that are versioned together, ensuring reproducible builds by tracking dependencies and their versions.  

It simplifies dependency management by creating a `go.mod` file, which lists the module's dependencies and their versions.  
This file is crucial for maintaining reproducible builds across different environments.



@25/60
