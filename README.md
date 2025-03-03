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

A graphical interface for managing Podman containers and Kubernetes (K8s) from your desktop.  

# How to install Podman

## Install Podman on Ubuntu

- `sudo apt update`
- `sudo apt install podman`


@4/60
