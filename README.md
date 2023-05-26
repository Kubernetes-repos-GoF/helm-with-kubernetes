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

-----------------------------------------------------------------------
fuente:  https://www.baeldung.com/ops/kubernetes-helm  

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


-----------------------------------------------------------------------
brew install kubernetes-helm   (mac)
helm version

helm init (only once)



----------------------------------------------------------------------------
fuente: https://www.clearpeaks.com/deploying-apache-airflow-on-a-kubernetes-cluster/
# Overview
 

As mentioned above, the objective of this article is to demonstrate how to deploy Airflow on a K8s cluster. To do so, we will need to create and initialise a set of auxiliary resources using YAML configuration files.      

**The first items that we will need to create are the Kubernetes resources**. K8s resources define how and where Airflow should store data (Persistent Volumes and Claims, Storage class resources), and how to assign an identity – and the required permissions – to the deployed Airflow service (Service account, Role, and Role binding resources).         

**Then we will need a Postgres database**: Airflow uses an external database to store metadata about running workflows and their tasks, so we will also show how to deploy Postgres on top of the same Kubernetes cluster where we want to run Airflow. First, we will need to create a Configmap, to configure the database username and password; then, a Persistent Volume and a Claim, to store the data on a physical disk; finally, a Postgres service resource, to expose the database connection to the Airflow Scheduler.      

**Finally, we will be able to deploy the actual Airflow components**: the Airflow scheduler and the Airflow webserver. These are the two main components of the Airflow platform. The scheduler runs the task instances in the structured order defined by what is called DAG, and the webserver is the web UI to monitor all the scheduled workflows.   

## Creating Template Files

# Airflow
 
So, as we said, the first thing to do to prepare our Airflow environment on Kubernetes is to prepare YAML files for the deployment of the required resources.   

**Usually, when creating such files, we can download templates from the Helm charts repository using the Helm package manager**. Once downloaded, the files can be adapted to our specific needs, in order to reflect the desired application deployment.   

To do so for our Airflow instance, we can use the following commands to define the Airflow repository, download the Helm chart locally, and finally create the template YAML files.

helm repo add apache-airflow https://airflow.apache.org   
helm repo update apache-airflow    
mkdir airflow-yamls   
cd airflow-yamls   
helm template –output-dir ./ /root/.cache/helm/repository/airflow-1.5.0.tgz  

**For example, we can ignore those related to the redis, celery and flower services, which we won’t be using.**

# Postgres

Airflow uses an internal database to store all the metadata about running tasks and workflows. For production-ready deployments, we recommend using an external database, so we deployed a PostgreSQL database for demonstration purposes.   

To do so, the first thing we need is a ConfigMap resource. ConfigMaps are Kubernetes resources which can be used to automatically configure applications running in the container when the Kubernetes pod is starting up. For our deployment, we used ConfigMap to set the Postgres database name, username, and password.  

The file we used to create this resource is called postgres-configmap.yaml 

After the Configmap has been defined, it can be created with the command below:

kubectl apply -f postgres-configmap.yaml -n airflow

Next, we need to create PersistentVolume and PersistentVolumeClaims for the Postgres service.   

The YAML file we used to create the PersistentVolume and the PersistentVolumeClaim used by Postgres is called postgres-storage.yaml

kubectl apply -f postgres-storage.yaml -n airflow  

Once done, check that they were created successfully:  

kubectl get pv -n airflow 
kubectl get pvc -n airflow

We can see that PersistentVolume is called postgres-pv-volume and is bound by PersistentVolumeClaim postgres-pv-claim, while Reclaim Policy is defined as Retain, which means that when PersistentVolumeClaim is deleted, the volume will not be deleted but retained, thus becoming available to other Pods or applications.

At this point we can create Deployment and Kubernetes Service resources for our Postgres deployment.

Deployment is a Kubernetes resource which is used to control a set of Pods in which applications are running, so that administrators can control the scalability of the application, create ReplicaSets, rollback previous versions, and so on.


The YAML file to deploy the Postgres service is called postgres-deployment.yaml and is shown below. Note that this Postgres deployment uses a pre-built docker image.

kubectl apply -f postgres-deployment.yaml -n airflow 

kubectl get pods -n airflow

Now that the Postgres deployment is finished, we need to create its service. A service is a Kubernetes resource used to expose an application to other applications through a network port.


The YAML file to create a Kubernetes service for the Postgres deployment is postgres-service.yaml

As defined above, the Postgres service will listen on port 5432 and it will be accessible from inside the cluster via the same port. This port will be used when configuring the Airflow scheduler. We create the Postgres service with the same command:

kubectl apply -f postgres-service -n airflow 

Now that we have a Postgres service up and running, we need a database with all the schemas and tables used by the Airflow scheduler. This can be done by using the Python package for integration between the Airflow and PostgreSQL databases: postgres-airflow. First, Python must be installed inside the Postgres container. To do so, open the Postgres container bash CLI with the following command:   

kubectl exec --stdin --tty {container_name} -- /bin/bash

Once inside the container bash CLI, install Python and the postgres-airflow package with the following commands:
sudo apt-get update 
sudo apt-get install Python3.8 
AIRFLOW_VERSION=2.2.4 
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)" 
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}
/constraints-${PYTHON_VERSION}.txt" 
pip install "apache-airflow[postgres]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}" 

Now, the database inside Postgres can be initialised like this:
airflow db init