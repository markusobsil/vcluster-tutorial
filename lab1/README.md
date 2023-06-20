# Lab1 - vcluster Introduction

## Prerequisites

### vcluster CLI

In this lab we are using the vcluster CLI, which provides an easy way to manage virtual clusters. Please follow the instructions on the [official documentation](https://www.vcluster.com/docs/getting-started/setup) for the installation.
I am using following version in this lab.

```
> vcluster --version
vcluster version 0.15.0
```

### kind

As already mentioned I am using kind as a host cluster in all labs. Please follow the instructions on the [official documentation](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) for the installation.
I am using following version in this lab.

```
> kind --version
kind version 0.19.0
```

### kubectl

To interact with your Kubernetes clusters you need kubectl. Please follow the instructions on the [official documentation](https://kubernetes.io/docs/tasks/tools/) for the installation.

```
> kubectl version --output=yaml
clientVersion:
  buildDate: "2023-04-14T13:14:41Z"
  compiler: gc
  gitCommit: 4c9411232e10168d7b050c49a1b59f6df9d7ea4b
  gitTreeState: clean
  gitVersion: v1.27.1
  goVersion: go1.20.3
  major: "1"
  minor: "27"
  platform: darwin/amd64
kustomizeVersion: v5.0.1
serverVersion:
  buildDate: "2023-05-12T19:03:40Z"
  compiler: gc
  gitCommit: 4c9411232e10168d7b050c49a1b59f6df9d7ea4b
  gitTreeState: clean
  gitVersion: v1.27.1
  goVersion: go1.20.3
  major: "1"
  minor: "27"
  platform: linux/amd64
```


## Create a kind cluster

You can find the used kind configuration in the [config](../config) directory.

```
> kind create cluster --config configs/kind-vcluster-1-node.yaml
```

After creating the kind cluster, the context of your kubeconfig should automatically switch to your kind cluster. If you are not using a kind cluster, please change your context to your host cluster.

```
> kubectl config current-context
kind-vcluster-host
```

## Deploy a virtual cluster

The easist way to deploy a virtual cluster is by using the vcluster CLI.

```
> vcluster create my-first-vcluster
```

### Questions

1. To which Kubernetes cluster are you now connected? Hint: Take a look at the current-context.
2. How can you switch between host cluster and virtual cluster by using the vcluster CLI?
3. In which namespace on the host cluster was the virtual cluster deployed?


## Analyse your virtual cluster


