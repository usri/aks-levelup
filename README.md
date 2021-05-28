# AKS LevelUp workshop

## Introduction

Welcome to the _AKS LevelUp_ workshop!  The goal of this workshop is to build on your working knowledge of [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes).

This is not an _introduction_ to Kubernetes workshop.  The content in this workshop assumes that you have already completed the [Azure Kubernetes Service Workshop](https://docs.microsoft.com/en-us/learn/modules/aks-workshop/) and therefore have some working knowledge of AKS.  The modules in this workshop include adavanced topics that are not covered in the [Azure Kubernetes Service Workshop](https://docs.microsoft.com/en-us/learn/modules/aks-workshop/).

## Pre-requisites

The following are required to participate in this workshop.

- Azure Subscription (commercial or government)
- An AKS instance resulting from the [Azure Kubernetes Service Workshop](https://docs.microsoft.com/en-us/learn/modules/aks-workshop/).
  - The _Security_ module will extend your existing fruit smoothies rating application.

If you do not have an AKS instance as mentioned above, then you can still participate in the 1st three topics with another AKS instance. If you need to create a new AKS instance, then you can do so quickly using the following commands:

```bash
# Sign-in to your subscription if you're not already signed-in.
az login

# Create the resource group for AKS
az group create --name levelup-rg --location eastus

# Creat the AKS cluster
az aks create --resource-group levelup-rg --name levelup-aks --node-count 2 --generate-ssh-keys

# Merge AKS credentials into ./kube/config, so you can run kubectl commands.
az aks get-credentials --resource-group levelup-rg --name levelup-aks
```


## Agenda

The agenda for this workshop is as follows:

| Time (CST) | Topic |
| --- | --- |
| 9:00 AM | Workshop Kickoff |
| 9:10 AM | [Storage - Persistent Volumes, Claims, & Storage Classes](./storage/README.md) |
| 10:25 AM | [Kustomize](./Kustomize/README.md) |
| 11:15 AM | Break |
| 11:25 AM | [Service mesh - Istio ](./ServiceMesh/readme.md) |
| 12:15 PM | [Security](./security/readme.md) |
| 1:30 PM | End of workshop |


## Support

During the live presentation of this workshop a team of procotors are available to support you and answer questions.  Instructions on how to engage with the proctors during the live event will be conveyed at that time.
