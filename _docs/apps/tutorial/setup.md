---
title: Setup Your Environment
overview: Setting Up Your Environment

order: 40

layout: docs
type: markdown
---

## MiniSLATE

In order to develop and test SLATE applications, you'll need a local development environment. We strongly suggest using [MiniSLATE](https://github.com/slateci/minislate). MiniSLATE acts as a self-contained instance of the SLATE platform, including all of the individual components needed to fully simulate the platform. Using this gives you access to a local dev cluster as if you are the System Admin of a SLATE Cluster. **This is the recommended environment for the tutorial.** Minislate itself runs as a container on your local machine that can be stopped or deleted with ease, and removes the need to install each of the SLATE components individually.

[The GitHub project for MiniSLATE](https://github.com/slateci/minislate) has the source and build instructions for MiniSLATE.

## Installing Each Piece Locally

While we don't recommend it, you can also install all SLATE's component pieces individually.

### Docker

Docker is the container solution that makes the rest of this stack possible. There are other containerizers, we've found that Docker has the most community support and is very accessible. [Docker's documentation](https://www.docker.com/get-started)
has a thorough installation guide. 

### Kubectl 

Kubectl is the command line interface (CLI) used to interact with Kubernetes Clusters. [The Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/) has instructions for Kubectl installation. There are many ways to install, including package managers like homebrew and snap.

We also encourage users to install the Autocomplete version for their shell to make working with Kubectl much, much easier.

### Minikube

In order to do local testing and have a single-node Kubernetes Cluster on hand, download and install Minikube. Minikube is a project directly from Kubernetes specifically designed for a testing environment, so it has a large amount of community support. [Minikube's Github repository](https://github.com/kubernetes/minikube/releases) has build and installation instructions. You will also need a VM driver such as VirtualBox that Minikube can use to start a VM for the local SLATE cluster. 

### Helm and Tiller

Helm is a package manager and templating system for Kubernetes. We use Helm in SLATE to maintain a repository of SLATE apps, and to limit what settings we need to expose to the end user to deploy an app. The value of Helm comes from a file called "values.yaml". This file exposes only the settings that a user needs to interact with to customzie a deployment, removing a lot of boilerplate work, and is used as the source for templating. [Helm's GitHub page](https://github.com/helm/helm/blob/master/docs/install.md) has build and installation instructions. 

### SLATE Client

Finally, we will need access to the SLATE client. To use SLATE, create an account on the SLATE portal and ask your SLATE-affiliated organization to add you to their approved users list. Once you've been added, there will be instructions for how to download the client, and save your user token. 

[The SLATE Portal](https://portal.slateci.io/) is an essential part of the SLATE platform.
