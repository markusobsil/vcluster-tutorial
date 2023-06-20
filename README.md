<br>
<img src="https://raw.githubusercontent.com/loft-sh/vcluster/main/docs/static/media/vcluster_horizontal_black.svg" width="500">

# Virtual Kubernetes Clusters Tutorial

## Overview

[Virtual clusters](https://www.vcluster.com/docs/what-are-virtual-clusters) are a great solution to run fully working Kubernetes clusters on top of Kubernetes clusters. As they are running their own control plane separated from the underlying Host clusters, you get a lot of new possibilities, e.g. creating development environments, running isolated tests in CI pipelines and building separated control planes in CD systems. Find out more about virtual clusters at the [offical documentation](https://www.vcluster.com/docs/what-are-virtual-clusters).

## Who is this for?

I created this tutorial to give my team, which is a team of software/cloud/devops engineers, a easy way to get familiar with vclusters and understand it's principles and architecture. This tutorial is designed to give you all you need to know to run and maintain vclusters in future.

## Documentation

Please use the [official virtual clusters](https://www.vcluster.com/docs/what-are-virtual-clusters) documentation, if you are looking for specific virtual clusters configurations.

## Labs

* [Lab1 - vcluster Introduction](/lab1)
* [Lab2 - vclusters for Development Teams](/lab2)
* [Lab3 - Fake and Real Nodes](/lab3)
* [Lab4 - Access services on the Host Cluster](/lab4)
* [Lab5 - Monitoring and Logging](/lab5)

## How to run a lab

I run all my labs on [kind](https://kind.sigs.k8s.io/) and provide my configuration for all labs. For sure you can use any other Kubernetes cluster as your host cluster. If you need specific tools for a lab, e.g. vcluster-cli, helm, Metallb, etc. you find then in the prerequisites sections of the specific lab.

Copyright (c) 2023 Markus Obsil
