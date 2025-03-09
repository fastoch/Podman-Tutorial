# Podman Tutorial
Podman Tutorial Zero to Hero - https://www.youtube.com/watch?v=YXfA5O5Mr18

# What is Podman?

Podman was written in the **Go** programming language.  

Go, also known as **Golang**, is a statically typed, compiled language developed by Google.  
It is known for its simplicity, efficiency, and strong support for **concurrent programming**, making it well-suited 
for developing containerization tools like Podman.

Podman is a daemon-less, open-source Linux-native tool designed to make it easy to find, run, build, share, and deploy applications using 
OCI (Open Container Initiative) containers and container images.  

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
podman rmi <image ID or image repo>
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

## Creating our simple application

- first, make sure that Go is installed by running `go version`
- if not, install it via `sudo apt install golang-go` (if running a Debian-based Linux distro)
- now, let's create a **project folder**: `mkdir pdm-golang`
- make it the working directory: `cd pdm-golang`
- to initialize our Go project: `go mod init pdm-golang`
- now we can create the `main.go` file to implement the application logic

```go
package main

import (
  "fmt";
  "net/http";
)

func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello Podman from Go!")
  })

  http.ListenAndServe(":8080", nil)
}
```
- The "fmt" package provides formatting functions such as `Fprintln` which we need to write the HTTP response message
- to expose the web service (to create an HTTP server), we use the standard `net/http` library provided by Go

The `main()` function is the **entry point** of the program that peforms the following tasks:
- it registers a **handler** function for the root path `"/"`
- the handler function takes 2 arguments: a ResponseWriter and a Request
  - the `ResponseWriter` creates the response that will be sent back to the client that has made an HTTP request
  - the `Request` contains information about the incoming HTTP requests
- this handler function uses the `fmt.Fprintln()` function to write the "Hello" message to the `ResponseWriter`
- finally, the program calls the `http.ListenAndServe` function to start the HTTP server and make it listen on port 8080
  - The server will listen on all available network interfaces because no specific IP address is provided before the port number
  - Using `nil` as the second parameter means you're using the default `http.DefaultServeMux` for routing requests
  - This function call is blocking, so any code after it won't execute until the HTTP server stops

This program creates an HTTP server listening on port 8080 and responds to requests with the message "Hello Podman from Go!".  

**Note**:  
For more control over our HTTP server's behavior, we would create a custom `http.Server` instance instead of using `ListenAndServe`.

---

### About the `go mod init` command

The primary purpose of `go mod init` is to initialize a Go module in our project.  

A Go module is a collection of related Go packages that are versioned together, ensuring reproducible builds by tracking dependencies and their versions.  

It simplifies dependency management by creating a `go.mod` file, which lists the module's dependencies and their versions.  
This file is crucial for maintaining reproducible builds across different environments.

---

## Running our application

To run the application, open a new terminal and run `go run main.go`.  
Then, in the first terminal, run `curl localhost:8080`.  
![image](https://github.com/user-attachments/assets/167c0afb-a746-4362-b1c3-2914a204337e)

We can also open a web browser and go to `localhost:8080` to see the same message.  

Of course, for production or more complex projects, it's better to use `go build` to create an executable.  
Here we're just running a very simple program which doesn't require the use of `go build`.  

---

# Let's containerize our application

To do this, let's create a file with all the necessary instructions.  
Even if we use Podman, we can name our file `Dockerfile`.  

This file will allow us to build the container image of our Golang application.  
And inside the container, our app will run on an Alpine Linux distribution.  

Here's our Dockerfile:
```dockerfile
FROM golang:1.22-alpine
WORKDIR /app
COPY go.mod ./
RUN go mod download
COPY *.go ./
RUN go build -o /pdm-golang
EXPOSE 8080
CMD ["/pdm-golang"]
```
- line 1 specifies the base image: a pre-configured environment with Go 1.22 installed on top of Alpine Linux
- line 2 specifies the working directory within our container filesystem
- line 3 copies the go.mod file from the host machine to the current working directory in the container (/app)
- line 4 downloads and installs any dependencies specified in the go.mod file 
- line 5 copies any .go file from the host machine to the /app folder inside the container 
- line 6 builds the app and generates an executable named "pdm-golang" in the root directory of the container filesystem
- line 7 informs Podman that the application will listen on port 8080
- line 8 specifies the command to run when the container starts, which will execute the pdm-golang file created at line 6

When this Dockerfile is used to create our container image, the resulting image will contain the compiled Go application 
and its dependencies, making it easy to deploy and run our application on any machine that has a container runtime installed.  

To build our image from our newly created Dockerfile, run the following cmd from our project folder:  
```bash
podman build -t pdm-golang .
```
- The dot notation `.` tells Podman that our Dockerfile is located in the current working directory

Here's the output of the `podman build` command:  
![image](https://github.com/user-attachments/assets/d848fc67-a556-45ab-9c34-107b74b16f4d)

The `ls` command shows that the image we've created is not visible as a file, but `podman images` allows us to see it.  
Since we haven't provided a version for our image, Podman automatically tagged it as the "latest" version.

## Running our containerized application 

`podman run -d -p 8080:8080 pdm-golang:latest`  

![image](https://github.com/user-attachments/assets/417adc57-a544-4f84-baac-3a4dc18734fa)  

---

# Sharing a container image

Our containerized application only exists locally, let's see how to share its image with others so they can use our application.  
For that, we need to push our image to a **container registry**.  
- `podman login <registry_name>`
- `podman build -t <username>/<image_name> .`
- `podman push <username>/<container_name>`

The first command is used to authenticate to the specified container registry.  
The second one is used to build a container image using the Dockerfile located in the current directory.  
The last command is used to push the image to the container registry.  

Before continuing, you need to choose a container registry and make sure you have credentials to log in to it.  
In the following example, we'll push the container image to DockerHub.  

- `podman login docker.io`

![image](https://github.com/user-attachments/assets/6f513750-f60b-45cc-9874-089a3181f6b8)  

Next, we need to rebuild the container image by adding our username (or company name) as a prefix to the image tag:  
`podman build -t fastoch/pdm-golang .`

Now, we should find this new container image on our local machine:  
![image](https://github.com/user-attachments/assets/2cd2d936-44f3-43e7-b44f-ca5c03eb3850)
Which is exactly the same image as the one we've previously created (same ID).  

This format `<username>/<image_name>` is important because it tells Podman where to publish the image in the registry.  

- To publish the image on the registry: `podman push fastoch/pdm-golang`

We have successfully published our first container image to the DockerHub registry:  
![image](https://github.com/user-attachments/assets/f74e0bf7-2ca0-491a-9140-ce3e54643f1e)

Now, we can pull our own image:  
![image](https://github.com/user-attachments/assets/aad2b6bb-76c5-40e0-8d10-a76f712cd583)  

And test it:  
![image](https://github.com/user-attachments/assets/e413ca9b-1d48-4ab6-b444-4438e19d09a0)

---

# Building Pods with Podman

## A unique and powerful Podman feature

Unlike Docker, Podman allows us to create **pods**, which are similar to **Kubernetes pods**.  
These pods provide a way for our containerized applications to be managed and scaled within a Kubernetes **cluster**.  

## About Pods and Namespaces

Pods are a group of one or more containers sharing the same Network, PID and IPC namespaces.  
- PID = process ID
- IPC = inter-process communication

**Namespaces** are particularly useful in containerization technologies, where they help create isolated environments.   
Applications deployed in such isolated environments can run without affecting the host system or other containers.  
By combining IPC namespaces with other types of namespaces (such as PID, Network, and Mount), developers can create fully isolated and secure container environments.  

## Useful commands for managing Podman pods

- `podman pod --help` provides a list of all available commands and options to use when working with pods
- `podman pod create --name <pod_name>` creates a new and empty pod
- `podman pod ls` lists all created pods
- `podman ps -a --pod` lists all containers, including stopped ones, along with information about the pods they belong to



@36/60
