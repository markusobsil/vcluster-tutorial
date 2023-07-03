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

*Fake Nodes* are the default in vcluster. When using *Fake Nodes* a node will
be created for each `spec.nodeName` encountered in a `Pod` definition. Why is
this the default?

As reading the node information from the Kube API on the host cluster is a
privileged operation, RBAC configuration would need to be in place to allow
vcluster (eg the `syncer`) to read that information.

Once the last `Pod` is removed from a *Fake Node* the node will be deleted.

### Create a vcluster

```bash
vcluster create fake-nodes
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?
3. What is the difference between a node in the Kind and the vcluster?

### Deploy a Pod

```bash
kubectl apply --filename resources/pod01
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?

### Deploy a second Pod

```bash
kubectl apply --filename resources/pod02
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?

### Delete the second Pod

```bash
kubectl delete pods pod02
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?

### Create a deployment

```bash
kubectl apply --filename resources/deployment01.yaml
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

### Fake Nodes on Kind

When using *Kind* the difference between the host (*Kind*) cluster and the
vcluster will not be significant. The reason being, that RBAC is not a
problem on *Kind* an vcluster is able to read the nodes.

To better illustrate the difference, here is an example taken from a vcluster
running on the `sandbox` DTP tooling cluster.

#### DTP sandbox node

```bash
$ kubectl get nodes ip-10-44-12-111.eu-central-1.compute.internal | \
    --output yaml | \
    --context dok-gitops-argo-sandbox | \
    jq '.metadata.labels'
```
```json
{
  "agentpool": "argo1",
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/instance-type": "m6a.xlarge",
  "beta.kubernetes.io/os": "linux",
  "dynatrace.com/tier": "system",
  "eks.amazonaws.com/capacityType": "ON_DEMAND",
  "eks.amazonaws.com/nodegroup": "dok-gitops-argo-sandbox-argo1v2-ng",
  "eks.amazonaws.com/nodegroup-image": "ami-01de92e208a15ab26",
  "eks.amazonaws.com/sourceLaunchTemplateId": "lt-09f1f5b0945e551a1",
  "eks.amazonaws.com/sourceLaunchTemplateVersion": "10",
  "failure-domain.beta.kubernetes.io/region": "eu-central-1",
  "failure-domain.beta.kubernetes.io/zone": "eu-central-1a",
  "k8s.io/cloud-provider-aws": "dc5b5acd02892ba995e2dfcdf6090d1d",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "ip-10-44-12-111.eu-central-1.compute.internal",
  "kubernetes.io/os": "linux",
  "node.kubernetes.io/instance-type": "m6a.xlarge",
  "topology.ebs.csi.aws.com/zone": "eu-central-1a",
  "topology.kubernetes.io/region": "eu-central-1",
  "topology.kubernetes.io/zone": "eu-central-1a",
  "vpc.amazonaws.com/eniConfig": "eu-central-1a"
}
```

#### vcluster node one DTP sandbox

```bash
kubectl get nodes ip-10-44-12-111.eu-central-1.compute.internal --output yaml | \
    jq '.metadata.labels'
```
```json
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "fake-ip-10-44-12-111.eu-central-1.compute.internal",
  "kubernetes.io/os": "linux",
  "vcluster.loft.sh/fake-node": "true"

```

## Real Nodes

*Real Nodes* in comparison to *Fake Nodes*, will sync the real node
information from the host cluster. As stated in
[Fake Nodes](#fake-nodes) RBAC configuration is needed for the vcluster to
be able to access the Kube API on the host cluster.

!!! note
    This is **not** true for *Kind*, as described in
    [Fake Nodes](#fake-nodes) as well.

To configure syncing the real nodes, some values need to be set when deploying
the vcluster. This can either be done on as an argument - when using the
`vcluster` CLI command - or by using a YAML file with `vcluster` and Helm.

The following values need to be set for to enable real node sync:

```yaml
sync:
  nodes:
    enabled: true
```
Pass the filename as the `--extra-values` parameter to `vcluster` when
creating the vcluster.

As is the case for [Fake Nodes](#fake-nodes), only real nodes where a Pod
is running will be synced.

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
kubectl apply --filename resources/deployment01.yaml
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

*All Real Nodes* is very similar to [Real Nodes](#real-nodes). The difference
is that with **All Real Nodes** all nodes are synced from the host cluster,
not only the ones where a Pod is running.

To sync all real nodes, an additional line must be added to the values file:

```yaml
sync:
  nodes:
    enabled: true
    syncAllNodes: true
```

You can also call `vcluster` with the `--sync-all-nodes` flag to enable this
mode.

!!! note
    You must pass the `--fake-nodes false` flag, for the `--sync-all-nodes`
    flag to work.

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
kubectl apply --filename resources/deployment01.yaml
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

## Real Nodes with Label Selector

### Add labels to Kind nodes

```bash
kubectl label --context kind-vcluster-host nodes vcluster-host-worker2 "example.com/tier=vcluster"
kubectl label --context kind-vcluster-host nodes vcluster-host-worker3 "example.com/tier=vcluster"
```

### Create vcluster

```bash
vcluster create real-nodes-selector --extra-values resources/real-nodes-selector-values.yaml
```

### Questions

1. How many nodes are in your Kind cluster?
2. How many nodes are in your vcluster?
3. How are the vcluster nodes labeled?

### Create a deployment

```bash
kubectl apply --filename resources/deployment01.yaml
```

### Questions

1. On which nodes are the pods running?
2. Can you explain the node assignment?

### Delete the vcluster

```bash
vcluster delete real-nodes-selector
```
```
info   Stopping docker proxy...
info   Delete vcluster real-nodes-selector...
done √ Successfully deleted virtual cluster real-nodes-selector in namespace vcluster-real-nodes-selector
done √ Successfully deleted virtual cluster namespace vcluster-real-nodes-selector
```
## Cleanup

### Delete the Kind cluster

```bash
kind delete cluster --name vcluster-host
Deleting cluster "vcluster-host" ...
Deleted nodes: ["vcluster-host-worker3" "vcluster-host-control-plane" "vcluster-host-worker" "vcluster-host-worker2"]
```