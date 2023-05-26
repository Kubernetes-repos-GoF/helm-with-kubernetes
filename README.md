# Helm with kubernetes

The purpose is to track all the steps and commands that are necessary to deploy a kubernetes cluster in which we can interact with several charts using Helm


# Overview

Helm is a package manager for kubernetes applications. It is a powerful tool for working with kubernetes resources. Recently, Helm has been awarded the graduated status by Cloud Native Computing Foundation (CNCF), which shows its growing popularity amongst Kubernetes users.  

Given a Kubernetes cluster:  
- A chart is a collection of files organized in a specific structure  
- The configuration information related to a chart is managed in the configuration  
- A running instance of a chart is called a release  

Helm tracks an installed chart in the Kubernetes cluster using releases. This means that it is possible to install a single chart multiple times with different releases in a cluster. Starting with Helm 3, releases are stored as Secrets by default in the namespace of the release directly.  

There is a distributed community chart repository by the name Artifact Hub but we can create our own private repository. We can add any number of chart to the repository to work with. 

## Helm Templating

# Prerequisites

1. We need a kubernetes Cluster. (Minikube: single node kubernetes cluster locally)  
2. command-line tool Kubectl  
3. An application to manage within the kubernetes cluster  


# Helm commands

Some of the commands available in Helm CLI are:   
- helm lint  
- helm template  
- helm install  
- helm get  
- helm upgrade  
- helm rollback  
- helm uninstall  
- helm package  
- helm repo  
- helm search   

choco install kubernetes-helm

kubectl cluster-info

helm init

helm create hello-world

helm lint ./hello-world
helm template ./hello-world

helm install --name hello-world ./hello-world

helm ls --all

helm upgrade hello-world ./hello-world

helm rollback hello-world 1

helm uninstall hello-world

helm package ./hello-world

helm repo index my-repo/ --url https://<username>.github.io/my-repo

helm repo add my-repo https://my-pages.github.io/my-repo

helm install my-repo/hello-world --name=hello-world

helm search repo <KEYWORD>



