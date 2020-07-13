
# Hosting RapidPro using Kubernetes

### Description

The aim of this section is to provide k8s configuration yaml files for hosting **RapidPro** on Azure or GCP cloud providers. RapidPro is a hosted service for visually building interactive messaging applications. To learn more, please visit the project site at [http://rapidpro.github.io/rapidpro](http://rapidpro.github.io/rapidpro).

Useful links: 
 - What is RapidPro? https://rapidpro.github.io/rapidpro/
 - Hosting: Highly Available Installation requirements: https://rapidpro.github.io/rapidpro/docs/hosting/
 
##### The hosted infrastructure goes by something like the general visualization below:

![general infrastructure](https://rapidpro.github.io/rapidpro/images/hosting.png)

## Getting Started

##### Requirements
First, you'll need the cloud provider's CLI tool by [Azure](https://docs.microsoft.com/pt-br/cli/azure/install-azure-cli?view=azure-cli-latest) or [Google](https://cloud.google.com/sdk/gcloud) for managing it's resources from local machine. 

Then, install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl) tool for k8s resources manager.

### Steps to create and connect to a Cluster

1. Create a kubernetes cluster on [AKS: Azure Kubernetes Service](https://docs.microsoft.com/pt-br/azure/aks/kubernetes-walkthrough#create-aks-cluster) or [GKE: Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster) with desired network/VPC conf and other cloud provider's options you'll need.

2. Get cluster credentials for connecting cloud provider resource (cluster) with kubectl commands, like below:

	**Azure**
	```
	az aks get-credentials --resource-group myResourceGroup --name myCluster
	```
	
	**GCP**
	```
	gcloud container clusters get-credentials myCluster --zone us-central1-c --project myProject
	```

3. Create kubectl namespace **prod-rapidpro** with the command below:
	```
	kubectl create namespace prod-rapidpro
	```

### Steps to create StorageClass

Create the StorageClass by applying the resource YAML according to your cloud provider, with one of the commands below:

> Edit these YAMLs as you need, adding the options your cloud provider hasand you would like to apply.

**Azure**
```
kubectl apply -f example-azure-ssd.yaml
```

or

**GCP**
```
kubectl apply -f example-gce-ssd.yaml
```

### Steps to create ElasticSearch and Redis workloads

> Edit these YAMLs as you need, adding the options your cloud provider hasand you would like to apply.

##### - ElasticSearch:
```
kubectl apply -f example-rapidpro-elasticsearch.yaml
```

##### - Redis:
```
kubectl apply -f example-rapidpro-redis.yaml
```
### Steps to create RapidPro related workloads

First, replace all occurences of *PostgreSQL*, *Redis* and *ElasticSearch* connection strings on all .envfiles which has it. 

For instance, replace all the strings below:

- ~~postgres://example_rapidpro:password@localhost:5432/example_rapidpro~~
	- for  ***postgres://USER:PSWD@HOST:PORT/DATABASE***
	> create a SQL instance and set the string above with it's connection details
> 
- ~~redis://localhost:6379/0~~
	- for ***redis://redis-master:6379/0***
	> **redis-master** is the name of the service described on example-rapidpro-redis.yaml 

- ~~http://localhost:9200~~
	- for ***http://rapidpro-elasticsearch-es-http:9200***
	> **rapidpro-elasticsearch-es-http** is the name of the service described on example-rapidpro-elasticsearch.yaml


#### 1. Create Secrets from the .envfile of each one of the pods:

##### rapidpro-courier.envfile
```
kubectl --namespace=prod-rapidpro create secret generic rapidpro-courier-environment --from-env-file=k8s-example/rapidpro-courier.envfile
```
##### rapidpro-indexer.envfile
```
kubectl --namespace=prod-rapidpro create secret generic rapidpro-indexer-environment --from-env-file=k8s-example/rapidpro-indexer.envfile
```
##### rapidpro-mailroom.envfile
```
kubectl --namespace=prod-rapidpro create secret generic rapidpro-mailroom-environment --from-env-file=k8s-example/rapidpro-mailroom.envfile
```
##### rapidpro.envfile
```
kubectl --namespace=prod-rapidpro create secret generic rapidpro-environment --from-env-file=k8s-example/rapidpro.envfile
```

> Edit these files as you need, adding or altering the environment variables needed on services

#### - Courier:
```
kubectl apply -f example-rapidpro-courier.yaml
```
#### - Indexer:
```
kubectl apply -f example-rapidpro-indexer.yaml
```
#### - Mailroom:
```
kubectl apply -f example-rapidpro-mailroom.yaml
```
#### - RapidPro:
```
kubectl apply -f example-rapidpro.yaml
```

## Exposing the service

> *(optional)* For exposing the service, a cloud provider's specific tool can be used as in Ingress/LoadBalancer on GCP

This way you could either manually config the reverse proxy on the cloud provider you use or apply the nginx config example on Cluster with the command below:

#### - Nginx:
```
kubectl apply -f example-nginx-conf.yaml
```
```
kubectl apply -f example-nginx.yaml
```


