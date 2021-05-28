# AKS Workshop Kustomize

## Introduction

Kubernetes relies on YAML for deployment definitions. As you applications grow and more services, application and namespaces are added, these YAML definitions can become hard to manage. Add several different environments such as UAT and Production and DevOps, mistakes to your cluster are inevitable. 

[kustomize](https://kustomize.io/#overview) is built-in to kubectl and assists in managing templates.

The main concepts in Kustomize are "base" and "overlays" ([terms](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/)). The idea is to have a "base" desired state such as "UAT (User Acceptance Testing)" where the YMAL might include a small node pool and ingress. Built on top of the base template, you can then create multiple overlay templates that extend or modify the base template for a specific environment. For example, you might have a "production overlay" where you have a large cluster in a different namespace.

This module will guide you through the tutorial below to give you hands-on experience configuring and using Kustomize to templatize a YAML workload.

## Tutorial: Redeploy ngnix

_(5 minutes)_

In this tutorial, you will deploy a simple YAML from the [PVC](https://github.com/rickrain/k8s-volumes/blob/main/README.md#tutorial-pod-storage) demo. The workload is a simple NGNIX container with no customizations. Make sure you have the [01-pod-storage.yaml](https://github.com/rickrain/k8s-volumes/blob/main/01-pod-storage.yaml).

### Step-by-step instructions

Execute the commands below from a bash command shell. Assure you are in the directory with your YAML.

```bash
# Apply deployment to the cluster
kubectl apply -f 01-pod-storage.yaml

# List the pod that was created, make sure they are created and running
kubectl get pods

# Delete the deployment
kubectl delete -f 01-pod-storage.yaml
```

### Summary

In this tutorial, we redeployed the 01-pod-storage as our baseline for our kustomize project.

## Tutorial: Create a UAT environment template

_(15 minutes)_

In this tutorial you will see how a kustomize customization allows us to easily manage namespaces and other properties. Most customizations start with a "base" which mostly would be used in a UAT environment.

A quick word on [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). Namespaces are commonly used to isolate deployment and services. It is a recommended practice to isolate your workloads by namespace.

The basics steps include:

- Move your base template to a directory
- Create a kustomization.yaml file to contain changes you wish to make
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
mkdir uat

# Copy the 01-pod-storage.yaml to this directory. This will be our "base"
cp 01-pod-storage.yaml uat

# Navigate to the overlays directory
cd uat

# Create a new yaml file named kustomization.yaml and add the below
cat > kustomization.yaml

# Add the following directives to the new file. Note that type, or kind,
# is a Kustomization. The change you wish to make is renaming the namespace
# in the new deployment to UAT.
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: uat

resources:
  - 01-pod-storage.yaml

# use &lt;ctrl&gt;&lt;D&lt; to save and exit the new file
# Return to root of project
cd ..

# Confirm you are in the root of your project (one directory above the UAT folder)
# Test/Debug you customization, you should see UAT as the namespace
kubectl kustomize uat

# Write to a yaml file. This file can be deployed.
kubectl kustomize uat > out.yaml

# Apply directly to the cluster (note the -k option)
kubectl apply -k uat/

# Look for pods in the root namespace
# None should be visible
kubectl get pods

# Look for pods in the UAT namespace
# Pods are now visable, port forward if you would like to connect to pods
kubectl get pods -n uat
```

### Summary

In this step you learned about isolating deployments in their own namespaces with Kustomize. You extended the base template by creating an overlay template with a different namesapce, UAT. You also learned how to display and persist the customized template. You also deployed the customization directly to the cluster.

## Tutorial: Create production environment template, namespace and scale

_(15 minutes)_

In this step, you will create a production namespace, deploy and scale the base template.

The steps include:

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
mkdir overlays

# Navigate to the overlays directory
cd overlays

# Create a Production folder
mkdir production

# Navigate to the Production folder
cd production

# Create a kustomization file named kustomization.yaml. Add the following directives to it:
# You can also use the Visual Studio Code plug built-in to CloudShell. Simply type: "Code ."
bases:
- ../../uat

namespace: production

patchesStrategicMerge:
- scale.yaml

# Create a new yaml named scale.yaml. Add the following directives:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-storage
spec:
  replicas: 10

# Navigate back to root
cd..
cd..

# Write customizations to screen. Note that the number of replicas are 10 and the namespace is production.
kubectl kustomize overlays/production

# Deploy to cluster
kubectl apply -k overlays/production

# Look for pods in the Production namespace
# Pods are now visable and scaled to 10 in the production namespace
kubectl get pods -n production
```

### Summary

In this tutorial, you created a new production overlay template and merged it into a scale.yaml template to scale the cluster. UAT and Production are usually similar but different in scale. kustomize helps keep the namespaces and yamls more manageable.
