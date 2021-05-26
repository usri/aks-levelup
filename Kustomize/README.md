# AKS Workshop Kustomize

## Introduction


Kubernetes relies on YAML for deployment definitions. As you applications grow and more services, application and namespaces are added, these YAML definitions can become hard to manage. Add several different environments such as UAT and Production and DevOps, mistakes to your cluster are inevitable. 

[kustomize](https://kustomize.io/#overview) is built-in to kubectl and assists in managing templates.


The main concepts in Kustomize are "bases" and "overlays" ([terms](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/)). The idea is to have a "base" desired state such as "UAT (User Acceptance Testing)" where the YMAL might include a small node pool and ingress. Then you might have a "production overlay" where you have a large cluster in a different namespace.

This module will guide you through the tutorial below to give you hands-on experience configuring and using Kustomize to templatize a YAML workload.


## Tutorial: Redeploy ngnix
_(5 minutes)_

In this tutorial we will deploy a simple YAML from the [PVC](https://github.com/rickrain/k8s-volumes/blob/main/README.md#tutorial-pod-storage) demo. This is a simple NGNIX container with no customizations. Make sure you have the [01-pod-storage.yaml](https://github.com/rickrain/k8s-volumes/blob/main/01-pod-storage.yaml).

### Step-by-step instructions

Execute the commands below from a bash command shell. Assure you are in the directory with your YAML.

```bash
# Apply deployment to the cluster
kubectl apply -f 01-pod-storage.yaml

# List the pod that was created
kubectl get pods

# Setup port forwarding so you can browse to the running container.
# Replace '[pod-name]' with the name of your pod.
kubectl port-forward [pod-name] 8080:80

# Open browser to localhost:8080.
# Observe that the response is the standard 'Welcome to nginx!' response.

# Press Ctrl-c to terminate the port forwarding.

# Delete the deployment
kubectl delete -f 01-pod-storage.yaml
```


### Summary
In this tutorial, we redeployed the 01-pod-storage as our baseline for our kustomize project.


## Tutorial: Create UAT
_(15 minutes)_

In this tutorial you will see how a kustomize customization allows us to easily manage namespaces and other properties. Most customizations start with a "base" which mostly would be used in a UAT environment. 

A quick word on [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). Namespaces are commonly used to isolate deployment and services. It is a recommended practice to isolate your workloads by namespace.


The basics steps include:
- Move your base template to a directory
- Create a kustomization.yaml file to contain the changes
- Deploy to file or cluster


```bash
# View your current namespaces
# You should see defaults
kubectl get ns

# Create a new namespace for UAT
kubectl create ns uat

# View your current namespaces
# You should now see UAT in the list
kubectl get ns

# Create a directory to hold your base image
MD UAT

# Copy the 01-pod-storage.yaml to this directory. This will be our "base"

# Create a new yaml file named kustomization.yaml and add the below
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: uat

resources:
  - 01-pod-storage.yaml




# Confirm you are in the root of your project (one directory above the UAT folder)
# Test/Debug you customization, you should see UAT as the namespace
kubectl kustomize uat

# Write to a ymal file. This could be deoployed.
kubectl kustomize uat > out.yaml

# Apply directly to the cluster (note the -k option)
kubectl apply -k uat/

# Look for pods
# None should be visible
kubectl get pods

# Look for pods in the UAT namespace
# Pods are now visable, port forward if you would like to connect to pods
kubectl get pods -n uat

```

### Summary
In this step you learned about isolating deployments in their own namespaces with Kustomize. We also show a few ways to display the customized templates, screen and to file. We also deployed the customizations directly to the cluster.



## Tutorial: Create production template, namespace and scale
_(15 minutes)_


In this step we will create a production namespace, deploy and scale the base.


The basics steps include:
- Create an overlays directory
- Create a kustomization.yaml file to contain the changes
- Deploy to file or cluster

```bash
# View your current namespaces
kubectl get ns

# Create a new namespace for UAT
kubectl create ns production

# View your current namespaces
# You should now see production in the list
kubectl get ns

# Create a directory to hold overlays
MD overlays

# Navigate to the overlays directory
CD overlays

# Create a Production folder
MD Production

# Create a new yaml named kustomization.yaml
bases:
- ../../uat

namespace: production

patchesStrategicMerge:
- scale.yaml




# Create a new yaml named scale.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-storage
spec:
  replicas: 10




# Navigate back to root
cd..
cd..

# Write customizations to screen
kubectl kustomize overlays/production

# Deploy to cluster
kubectl apply -k overlays/production

# Look for pods in the Production namespace
# Pods are now visable and scaled to 10 in the production namespace
kubectl get pods -n production
```

### Summary
In this tutorial, we created a new production overlay and merged in a scale.yaml to scale our cluster. UAT and Production are usually similar but different in scale. kustomize helps keep the namespaces and yamls manageable. 



