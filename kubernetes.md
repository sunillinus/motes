# Kubernetes
## Overview
- Kubernetes is a system for managing containerized applications across multiple hosts.
- Provides basic mechanisms for deployment, maintenance, and scaling of applications.
- Also Provides self-healing mechanisms, such as auto-restarting, re-scheduling, and replicating containers.
- Kubernetes enables users to ask a cluster to run a set of containers. The system automatically chooses hosts to run those containers on.

## Key Concepts

### Pods
- Tightly coupled group of containers that are scheduled to run on the same host.
- Pods serve as units of deployment and horizontal scaling/replication.
- In addition to defining the containers that run in the pod, the pod specifies a set of shared storage volumes. Volumes enable data to survive container restarts and to be shared among the containers within the pod.
- While Pods can be used to host vertically integrated application stacks, their primary motivation is to support co-located, co-managed helper programs.

### Labels
- Labels are key/value pairs that are attached to Kubernetes objects, such as pods.
- Via a label selector the user can identify a set of objects.
- Can be used to loosely couple cooperating pods Example - key: `environment` (with values `dev`, `staging`, or `production`...).

The `service` and `replicationController` objects use selectors to match sets of pods that they operate on - the set of pods that a service targets is defined with a label selector. Similarly, the population of pods that a replicationController is monitoring is also defined with a label selector.

### Services
- A Kubernetes service is an abstraction which defines a logical set of pods.
- A service offers clients an IP and port pair which redirects to the appropriate pods  determined by a label selector.
- It enables connectivity between the layers of your application even when the pods move around.
- A service is a configuration unit for the proxies that run on every worker node. It is named and points to one or more pods.

### ReplicationController
- A Replication Controller ensures that there is a specified number of "replicas" of a pod running at any given time. If there are too many, it'll kill some. If there are too few, it'll start more.
- Replication controller makes it easy to scale the number of replicas up or down, either manually or by an auto-scaling.
- Replication controller also facilitate rolling updates to a service by replacing pods one-by-one.
- The population of pods that a replicationController is monitoring is defined by a label, which creates a loosely coupled relationship between the controller and the pods controlled.
- A replication controller creates new pods from a template, which is currently inline in the replicationController manifest. 

## Networking
Kubernetes gives every pod its own IP address allocated from an internal network, so you do not need to explicitly create links between communicating pods. However, since pods can fail and be scheduled to different nodes, it is not recommended having a pod directly talk to the IP address of another Pod. Instead, if a pod, or collection of pods, provide some service, then you should create a service object spanning those pods, and clients should connect to the IP of the service object.

## Architecture
Consists of master node that run services that comprise the cluster-level control plane and a worker node that run on services necessary to run Docker containers and be managed from the master systems.

## Kubernetes Worker/Minion Node
The services that run on the worker node are:
1. Docker
2. Kubelet
3. Kubernetes Proxy

### 1. Docker
Duh!

### 2. Kubelet
The Kubelet takes a set of manifests that are provided in various mechanisms and ensures that the containers described in those manifests are started and continue running.
There are 4 ways that a container manifest can be provided to the Kubelet:
1. File Path passed as a flag on the command line. This file is rechecked every 20 seconds (configurable with a flag).
2. HTTP endpoint HTTP endpoint passed as a parameter on the command line. This endpoint is checked every 20 seconds (also configurable with a flag.)
3. etcd server The Kubelet will reach out and do a watch on an etcd server. The etcd path that is watched is /registry/hosts/$(hostname -f). As this is a watch, changes are noticed and acted upon very quickly.
4. HTTP server The kubelet can also listen for HTTP and respond to a simple API (underspec'd currently) to submit a new manifest.

### 3. Kubernetes Proxy
Simple network proxy that reflects services (see here for more details) as defined in the Kubernetes API on each node and can do simple TCP and UDP stream forwarding (round robin) across a set of backends.

Service endpoints are  found through environment variables (both Docker-links-compatible and Kubernetes {FOO}_SERVICE_HOST and {FOO}_SERVICE_PORT variables are supported). These variables resolve to ports managed by the service proxy.

## Kunernetes Master Node
The services that run on the master node are:
1. etcd
2. Kubernetes API Server
3. Kubernetes Controller Manager Server

### 1. etcd
Used to store master state.

### 2. Kubernetes API Server
-  Server Kubernetest REST API
- Validates and stores (in etcd) configuration data for 3 types of objects: pods, services, and replicationControllers.
- Schedules pods to worker nodes.
- Synchronize pod information (where they are, what ports they are exposing) with the service configuration.

### 3. Kubernetes Controller Manager Server
This server watches etcd for changes to replicationController objects and then uses the public Kubernetes API to implement the replication algorithm.

## References
1. [Kubernetes Design Overview](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/DESIGN.md#kubernetes-controller-manager-server
)
2. [Kubernetes 101](https://github.com/GoogleCloudPlatform/kubernetes/tree/master/examples/walkthrough)
3. [Kubernetes 201](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/examples/walkthrough/k8s201.md)
4. [Pods](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/pods.md)
5. [Labels](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/labels.md)
6. [Volumes](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/volumes.md)
7. [Services](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/services.md)
8. [Replication](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/replication-controller.md)
9. [Getting Started](https://github.com/GoogleCloudPlatform/kubernetes)
10. [Networking](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/design/networking.md)
11. [Misc](https://www.digitalocean.com/community/tutorials/an-introduction-to-kubernetes)
