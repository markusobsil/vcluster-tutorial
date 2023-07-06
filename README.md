<br>
<img src="https://raw.githubusercontent.com/loft-sh/vcluster/main/docs/static/media/vcluster_horizontal_black.svg" width="500">

# Virtual Kubernetes Clusters Tutorial

## Overview

[Virtual clusters](https://www.vcluster.com/docs/what-are-virtual-clusters) or short vclusters, are a great solution to run fully working Kubernetes clusters on top of Kubernetes clusters. As virtual clusters are running their own control plane, separated from the underlying host clusters, you get a lot of new possibilities for your infrastructure, e.g. creating cost-effective development environments, running tests in your CI pipelines completely isolated or building separated control planes in your CD systems. Find out more about virtual clusters at the [offical documentation](https://www.vcluster.com/docs/what-are-virtual-clusters).

## Who is this for?

I created this tutorial for my team, which is a team of software/cloud/devops engineers, so that they have an easy and interactive way to get to know virtual clusters, understand it's principles and it's architecture. This tutorial is designed to give you all you need to know to run and maintain vclusters in future. I encourage everyone who is interested in virtual clusters to do run some labs.

## Documentation

Please use the [official virtual clusters](https://www.vcluster.com/docs/what-are-virtual-clusters) documentation, if you are looking for specific virtual clusters configurations. The idea of this tutorial is not to replace it. 

## Labs

* [Lab 1 - Virtual cluster introduction](/lab1)
* [Lab 2 - Multitenancy with virtual clusters](/lab2)
* [Lab 3 - Fake and Real Nodes](/lab3)
* [Lab 4 - Access services on the Host Cluster](/lab4)
* [Lab 5 - Monitoring and Logging](/lab5)

## How to run a lab?

I run all my labs on [kind](https://kind.sigs.k8s.io/) on my workstation and provide my configuration for all labs. For sure you can use any other Kubernetes cluster as your host cluster. If you need specific tools for a lab, e.g. vcluster-cli, helm, Metallb, etc. you find them in the Prerequisites section of the specific lab.

<br>
Copyright (c) 2023 Markus Obsil
