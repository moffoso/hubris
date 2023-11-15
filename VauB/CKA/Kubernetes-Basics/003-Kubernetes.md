### What Kubernetes is

According to https://kuberenetes.io
	_"Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications"._

Kubernetes comes from the Greek word for "helmsman" or more directly translated "ship pilot" Google donated it to the Cloud Native Computing Foundation (CNCF) in 2015

### Core Kubernetes Features

- Automatic bin packing
	Automatic scheduling of containers based on resource needs and constraints, to maximize utilization without sacrificing availablity

- Extensibility
	Clusters can be extended with custom resources without modifying the upstream source code

- Self-healing
	Automatic replacing and rescheduling of containers from failed nodes. Termination and restarting of containers that become unresponsive to health checks based on rules/policies. This prevents traffic from being routed to unresponsive containers

- Horizontal Scaling
	Applications can be scaled manually or automatically based on custom metrics utilization

- Service discovery and load-balancing
	Containers recieve an IP address from k8s, while it assigns a single DNS name to a set of containers to aid in load-balancing

- Automated rollouts and rollbacks
	Seemless rollouts and rollbacks of application updates and configuration changes, while monitoring application health to prevent downtime

- Secret and configuration management
	Management of sensitive data and config details for an application seperatetly from the container image

- Storage Orchestration
	Automatic mounting of software-defined storage (SDS) solutions to containers from local storage, external cloud providers, distributed storage, or network storage systems

- Batch execution
	Support for batch execution, long-running jobs and replaces failed containers

- IPv4/IPv6 dual-stack
	Support for both IPv4 and IPv6 addresses


### Kubernetes Architecture

At a high level Kubernetes is a cluster of compute systems with distinct roles:
- One or more control planes
- One or more worker nodes

![](Kubernetes_architecture.png)

- Control plane node overview
	- provides a running environment for the control plane agents, these agents are have distinct roles in the cluster's management
	- to persist the cluster's state, all configuration data is saved to a distributed key-value store which only holds cluster state related data
-  Control plane node components
	- API Server
	- Scheduler
	- Controller Manager
	- Key-Value Data Store