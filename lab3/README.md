# Lab 3 - Fake and Real Nodes

## Overview

This lab is all about Kubernetes nodes. After completing this lab, you should
be able to answer the following questions:

* What are *Fake Nodes*?
* What are *Real Nodes*?
* What are *Node Label Selectors* used for?

## Prerequisites

The prerequisites are the same as in [Lab 1](../lab1/README.md).

## Create a kind cluster

You find the kind configuration for this lab [here](kind-config-vcluster-lab-3.yaml).

```bash
kind create cluster --config kind-config-vcluster-lab-3.yaml
```
```
Creating cluster "vcluster-host" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-vcluster-host"
You can now use your cluster with:

kubectl cluster-info --context kind-vcluster-host

Have a nice day! 👋
```

After creating the kind cluster, your kube config context should automatically
switch to the new kind cluster. If you are not using a kind cluster, please
change the context to your host cluster.

```bash
kubectl config current-context
```
```
kind-vcluster-host
```

## Fake Nodes

### Create vcluster

```bash
vcluster create fake-nodes
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?
3. What is the difference between a node in the Kind and the vcluster?

### Deploy a Pod

```bash
k apply --filename resources/pod01
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?

### Deploy a second Pod

```bash
k apply --filename resources/pod02
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?

### Delete the second Pod

```bash
k delete pods pod02
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?

### Create a deployment

```bash
k apply --filename resources/deployment01.yaml
```

### Questions

1. On which nodes are the pods running?
2. Can you explain the node assignment?

### Delete the vcluster

```bash
vcluster delete fake-nodes
```
```
info   Stopping docker proxy...
info   Delete vcluster fake-nodes...
done √ Successfully deleted virtual cluster fake-nodes in namespace vcluster-fake-nodes
done √ Successfully deleted virtual cluster namespace vcluster-fake-nodes
```

## Real Nodes

### Create vcluster

```bash
vcluster create real-nodes --extra-values resources/real-nodes-values.yaml
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?
3. What is the difference between a node in the Kind and the vcluster?

### Create a deployment

```bash
k apply --filename resources/real-nodes-deployment01.yaml
```

### Questions

1. On which nodes are the pods running?
2. Can you explain the node assignment?

### Delete the vcluster

```bash
vcluster delete real-nodes
```
```
vcluster delete real-nodes
info   Stopping docker proxy...
info   Delete vcluster real-nodes...
done √ Successfully deleted virtual cluster real-nodes in namespace vcluster-real-nodes
done √ Successfully deleted virtual cluster namespace vcluster-real-nodes
```

## All Real Nodes

### Create vcluster

```bash
vcluster create all-real-nodes --extra-values resources/all-real-nodes-values.yaml
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?
3. What is the difference between a node in the Kind and the vcluster?

### Create a deployment

```bash
k apply --filename resources/real-nodes-deployment01.yaml
```

### Questions

1. On which nodes are the pods running?
2. Can you explain the node assignment?

### Delete the vcluster

```bash
vcluster delete all-real-nodes
```
```
vcluster delete all-real-nodes
info   Stopping docker proxy...
info   Delete vcluster all-real-nodes...
done √ Successfully deleted virtual cluster all-real-nodes in namespace vcluster-all-real-nodes
done √ Successfully deleted virtual cluster namespace vcluster-all-real-nodes
```