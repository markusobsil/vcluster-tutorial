# Lab 1 - Virtual cluster Introduction

## 1. Overview

After completing this lab you should have a basic understanding of the architecture of virtual clusters and how to interact with them with the vcluster CLI.

## 2. Prerequisites

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

## 3. Create a kind cluster

You find the kind configuration for this lab [here](kind-config-vcluster-lab-1.yaml).

```
kind create cluster --config kind-config-vcluster-lab-1.yaml
```

After creating the kind cluster, your kube config context should automatically switch to the new kind cluster. If you are not using a kind cluster, please change the context to your host cluster.

```
> kubectl config current-context
kind-vcluster-host
```

## 4. Deploy a virtual cluster

The easiest way to deploy a virtual cluster is by using the vcluster CLI.

```
vcluster create my-first-vcluster
```

### Questions

1. To which Kubernetes cluster are you now connected? Hint: Take a look at the current-context again.
2. How can you switch between host cluster and virtual cluster by using the vcluster CLI?
3. In which namespace on the host cluster was the virtual cluster deployed?

## 5. Analyse your virtual cluster

### K3s and vcluster Syncer

Let's have a look at the pods the vcluster CLI created on the host cluster. By default, when using the vcluster CLI, it will install a [K3s](https://k3s.io/) powered Kubernetes cluster. It is a very small and lightweight, and runs in only one process. That's why the list of pods is very short and looks like this.

```
> kubectl get pods -n vcluster-my-first-vcluster
NAME                                                         READY   STATUS    RESTARTS   AGE
coredns-56bfc489cf-gtfkj-x-kube-system-x-my-first-vcluster   1/1     Running   0          3h26m
my-first-vcluster-0                                          2/2     Running   0          3h26m

> kubectl get pod my-first-vcluster-0 -n vcluster-my-first-vcluster -o jsonpath='{.spec.containers[*].image}'
rancher/k3s:v1.26.0-k3s1 ghcr.io/loft-sh/vcluster:0.15.0
```

Take a look at [this section](https://www.vcluster.com/docs/architecture/basics) vcluster documentation to understand the components of a vcluster. My virtual cluster runs K3s in the version 1.26.0 and next to it a vcluster container, which hosts the vcluster control plane. To fully understand who the vcluster works under the hood, let's take a look at the K3s configuration.

```
> kubectl get pod my-first-vcluster-0 -n vcluster-my-first-vcluster -o jsonpath='{.spec.containers[0].args[1]}'
/bin/k3s server --write-kubeconfig=/data/k3s-config/kube-config.yaml --data-dir=/data --disable=traefik,servicelb,metrics-server,local-storage,coredns --disable-network-policy --disable-agent --disable-cloud-controller --flannel-backend=none --disable-scheduler --kube-controller-manager-arg=controllers=*,-nodeipam,-nodelifecycle,-persistentvolume-binder,-attachdetach,-persistentvolume-expander,-cloud-node-lifecycle,-ttl --kube-apiserver-arg=endpoint-reconciler-type=none --service-cidr=10.96.0.0/12 && true
```

The important part I want to show you here is, that the Kubernetes Scheduler is disabled. And this is crucial in the virtual cluster architecture. Instead of using a scheduler in the vcluster, the scheduler of the host cluster is used. But how is this working? Let's go through this step by step.

1. You want to run a Pod within the virtual cluster and e.g. use the *kubectl run* command. So you are interacting with the Kubernetes API server of the vcluster, add your new Pod definition and wait for scheduling.
2. Normally now the Kubernetes Scheduler would takes care about scheduling the Pod by calculation the *best* node and add the selected node to the Pod object, so that the Pod will get started on that node. But as we show before, the Kubernetes Scheduler is disabled. This is where the vcluster control plane comes into play. The virtual cluster Syncer process interacts on both clusters, the virtual cluster and the host cluster. It watches e.g. Pods on the vcluster and applies the same Pod manifests with small adjustments, e.g. the Pod name, on the host cluster.
3. The Kubernetes Scheduler on the host cluster will choose a node for the Pod, and the Kubelet will start the Pod on this node. The Syncer will watch the Pods on the host cluster too and syncs the state of the Pod into the vcluster. For within the vcluster, you will not realize that no Scheduler is running within the vcluster, and that the host cluster will schedule and run your pod.

Within the virtual cluster you do not see these control plane components. You are interaction with the Kubernetes API only, which is accessible via the Kubernetes Service object (as on every other Kubernetes Cluster).

```
> kubectl get svc -n default
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.143.146   <none>        443/TCP   10m
```


### Core DNS

The second Pod which runs in the virtual cluster namespace is the DNS server of the virtual cluster. [Core DNS](https://coredns.io/) is widely used on Kubernetes clusters and a first choice in such environments. By using a DNS server within every vcluster, you are getting a separated service discovery and do not mix services between virtual clusters or the host cluster. If you want to know more about that, you can take a look at the [Network and DNS section](https://www.vcluster.com/docs/architecture/networking) of the official documentation.

```
> kubectl get deploy,pod,svc -n kube-system
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           10m

NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-56bfc489cf-pjmxg   1/1     Running   0          10m

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.111.169.140   <none>        53/UDP,53/TCP,9153/TCP   10m
```

### Questions
1. Which Kubernetes version is your host cluster and your virtual cluster running?


## 6. Run your first pods within the vcluster

Connect to the vcluster, create two namespaces and run one pod in each namespace.

```
> vcluster connect my-first-vcluster
  --output omitted--

> kubectl config current-context
vcluster_my-first-vcluster_vcluster-my-first-vcluster_kind-vcluster-host

> kubectl create ns my-first-ns
namespace/my-first-ns created

> kubectl create ns my-second-ns
namespace/my-second-ns created

> kubectl run busybox --image busybox:1.36.1 --namespace my-first-ns --command -- sh -c "sleep 3600"
 pod/busybox created

> kubectl run busybox --image busybox:1.36.1 --namespace my-second-ns --command -- sh -c "sleep 3600"
pod/busybox created
```

### Questions
1. Take a look at the namespaces on the host cluster. Are your newly created namespaces synced or not?
2. Compare the name of the Pods within the virtual cluster with the name of the Pods on the host cluster. Can you see the naming schema the Syncer is introducing?

## 7. High and Low-Level resources

The last step in this lab is to find out which resources are synced and which not by the Syncer. So let's make some tests.
We create one deployment, a service object, a secret and a network policy. You can find all resources I am using in this repository.

```
> kubectl apply -f resources/deployment.yaml -n my-first-ns
deployment.apps/my-webserver created

> kubectl apply -f resources/service.yaml -n my-first-ns
service/my-webserver created

> kubectl apply -f resources/secret.yaml -n my-first-ns
secret/my-secret created

> kubectl apply -f resources/network-policy.yaml -n my-first-ns
networkpolicy.networking.k8s.io/allow-all-ingress-my-webserver created
```

### Questions

1. After creating these resources, compare the resources between virtual cluster and host cluster. Which resources are synced to the host cluster?

A full list of resources and how they are synced, can be found in the [offical documentation](https://www.vcluster.com/docs/architecture/synced-resources).

## 8. Cleanup

Delete at first the virtual cluster with the vcluster CLI.

```
vcluster delete my-first-vcluster
info   Stopping docker proxy...
info   Delete vcluster my-first-vcluster...
done √ Successfully deleted virtual cluster my-first-vcluster in namespace vcluster-my-first-vcluster
done √ Successfully deleted virtual cluster namespace vcluster-my-first-vcluster
```

And then delete the kind cluster.

```
> kind delete cluster --name vcluster-host
Deleting cluster "vcluster-host" ...
Deleted nodes: ["vcluster-host-control-plane"]
```
