---
layout: post
title:  "Using Tensorflow-GPU(CUDA and CuDNN) inside a Docker environment"
date:   2019-10-17 08:28:36 +0800
categories: Development Tensorflow CUDA
---

Nvidia provides toolkits to allow CUDA and CuDNN operating inside a Docker container, and also official Docker images based on many Linux distributions [here](https://hub.docker.com/r/nvidia/cuda/).

## Nvidia Container toolkit

First you should install [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker). For reference, key steps are taken down here. And then make sure you have installed the `NVIDIA driver` and `Docker 19.03` for your Linux distribution. There are two ways to install.

### Install from Nvidia's official Github repo

This is the method provided by Nvidia official, but may not function due to network problems.(It seems that github.io is blocked by the firewall...)

#### Debian-like(Ubuntu, LinuxMint, etc.) systems

```bash
# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Install the toolkit and restart Docker service
sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

#### RHEL-like(CentOS, Fedora, etc.) systems

```bash
# Add yum repo
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo

# Install the toolkit and restart Docker service
sudo yum install -y nvidia-container-toolkit
sudo systemctl restart docker
```

## Tensorflow-GPU inside Docker

I have already built an image based on `nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04`, you can pull it from Docker Hub.

```bash
sudo docker pull jqjiang/tf-gpu:latest
```

### Usage

```bash
# Start a container with all GPUs, proxy enabled
sudo docker run --gpus all --env http_proxy="http://child-prc.intel.com:913/" --env https_proxy="http://child-prc.intel.com:913/" jqjiang/tf-gpu:latest
```

or a basic Dockerfile:

```dockerfile
FROM nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04

VOLUME ["/work"]

RUN apt-get update && apt-get install -y --no-install-recommends python3 python3-pip && \
    pip3 install wheel setuptools && \
    pip3 install tensorflow-gpu==1.14.0 && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /work
```

For more usages, refer to [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker) page.