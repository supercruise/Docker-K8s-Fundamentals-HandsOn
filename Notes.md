# Docker and Kubernetes Fundamentals

## Microservices Architecture

- A variant of the service-oriented architecture (SOA) structural style - arranges an application as a collection of loosely coupled services.
- In a microservices architecture, services are fine-grained and the protocols are lightweight.

---

## Kubernetes

### Introduction to K8s

- K8s is the leading container orchestration tool.
- Designed as a loosely coupled collection of components centered around deploying, maintaining, and scaling workloads.
- Vendor-neutral

### What K8s can do

- Service discovery and load balancing.
- Storage orchestration: local and cloud based.
- Automated rollouts and rollbacks.
- Self-healing.
- Secret and config management.
- Use the same API across on-premise and every cloud providers.

### What K8s cannot do

- Does not deploy source code.
- Does not build your application.
- Does not provide application-level services: message buses, databases, caches, etc.

### K8s architecture

- K8s master node (control plane)
  - kube-apiserver: REST API
  - etcd
  - kube-scheduler
  - Kube-controller manager
  - Cloud-controller manager
- K8s (worker) node
  - Kubelet
  - Kube-proxy
- Cluster
  - Node
    - Pod

### Local K8s

- Limited to 1 node:
  - Docker Desktop
- Multiple nodes
  - MicroK8s
  - MiniKube: does not need Docker Desktop
  - Kind: Kubernetes IN Docker

### K8s CLI and Context

- Kubectl:
  - communicate with the apiserver
  - config stored locally
    - ${HOME}/kube/config
    - C:\Users\\{USER}\\.kube\config
- K8s context
  - A context is a group of access parameters to a K8s cluster.
  - Contains a K8s cluster, a user, and a namespace.
  - The current context is the cluster that is currently the default for kubectl
    - All kubectl commands run against that cluster.
  - **Kubectx** - a managing tool for context

### The declarative way vs. the imperative way

- Declarative way
  - uSING kubectl and YAML manifests defining the resources that you need.
  - Reproducible and repeatable.
  - Can be saved in source control.
  - Infrastructure as Code.
- Imperative way
  - Use Kubectl commands, issue a series of commands to create resources.
  - Great for learning, testing, and troubleshooting.
  - It's like code.

---

## Kubernetes Namespaces

- Allow to group resources: Dev, Test, Production.
- K8s creates a default workspace.
- Objects in one namespace can access objects in a different one
  - Ex: objectname.prod.svc.cluster.local
- Deleting a namespace will delete all its child objects.

---

## Nodes

### Master node (Control plane)

- etcd: key-value data store for cluster state data.
  - Act as the cluster data store for storing state.
  - Not a database or a data store for applications to use.
  - The single source of truth.
- kube-apiserver: the only component communicating with etcd.
  - REST API.
  - Save states to etcd.
  - All clients interact with it, never directly to etcd.
- Kube-control-manager
  - The controller of controllers.
  - It runs controllers:
    - Node controller
    - Replication controller
    - Endpoints controller
    - Service account and token controllers
- Cloud-control-manager
  - Interact with the cloud providers controllers.
    - Node: for checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding.
    - Route: for setting up routes in the underlying cloud infrastructure.
    - Service: for creating, updating, and deleting cloud provider load balancers.
    - Volumn: for creating, attaching, and mouting volumes, and interacting with cloud provider to orchestrate volumes.
- Kube-scheduler
  - Watches newly created pods that have no node assigned, and selects a node for them to run on.
  - Factors taken into account for scheduling decisions include
    - Individual and collective resource requirements.
    - Hardware/software/policy constraints.
    - Affinity and anti-affinity specifications.
    - Data locality.
- Addons
  - DNS
  - Web UI (dashboard)
  - Cluster-level logging
  - Container resource monitoring

### Worker node

- Pods
- Kube-proxy
  - A network proxy that manages network rules on nodes.
  - All network traffic go through the kube-proxy.
- Kubelet
  - manages the pods lifecycle
  - ensures that the containers described in the pod specs are running and healthy.
- Container runtime
  - K8s supports several container runtimes.
  - Must implement the K8s Container Runtime Interface (CRI)
    - Moby
    - Containerd
    - Cri-O
    - Rkt
    - Kata
    - Virtlet

### Nodes pool

- A node pool is a group VMs, all with the same size.
- A cluster can have multiple node pools
  - These pools can host different sizes of VMs.
  - Each pool can be autoscaled independently from the other pools.
- Docker Desktop is limited to 1 node.

---

## Pods

- Atomic unit of the smallest unit of work of K8s.
- Encapsulates an applications's container.
- Represents a unit of deployment.
- Pods can run one or more containers.
- Containers within a pod share: IP address space, mounted volumes.
- Containers within a pod can communicate via: localhost, IPC
- Pods are ephemeral.
- Deploying a pod is an atomic operation, it succeed or not.
- If a pod fails, it is replaced with a new one with a new IP address.
- You don't update a pod, you replace it with an updated version.
- You scale by adding more pods, not more containers in a pod.
- Node
  - Pods: 1 or more
    - containers: 1 main worker and helper containers.

### Pod creation

### Pod deletion

- Grace period for deletion: 30 seconds.
- States are stored in etcd.

### Pod state

- Pending: accepted but not yet created.
- Running: bound to a node
- Succeeded: exited with status 0.
- Failed: all containers exit and at least one exited with non-zero status.
- Unknown: communication issues with the pod.
- CrashLoopBackOff: started, crashed, started again, and then crashed again.

### YAML for defining a pod

- apiVersion
- kind: **pod**
- metadata
  - name
  - labels
    - app: 
    - type: 
- spec
  - containers
    - name
    - _image_
    - ports
      - _containerPort_: 80
      - name
      - protocol
    - env
      - name
      - value: connection string
    - command: ["/bin/sh", "-c"]
    - args

### Init Container

- Use case: my app has a dependency (e.g., database, API, files) on [XYZ] but I don't want to clutter it with infrastructure code.
- A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.
- Always run to completion.
- Each init container must complete successfully before the next one starts.
- If it fails, the kubelet repeatedly restarts it until it succeeds (unless its restartPolicy is set to NEVER).
- Probes are not supported.
- https://kubernetes.io/docs/concepts/cluster-administration/logging/

### Pod Types
- Static pods: These are pods that are managed directly by the kubelet daemon on a specific node, without the API server observing them.
- ReplicationController (RC) pods
- Deployment or application pods: 
- DaemonSet pods: DaemonSet pods ensure that a copy of a pod runs on each node in the Kubernetes cluster. They are useful for deploying system-level agents or services that need to be present on every node, such as log collectors or monitoring agents.
- Job and CronJob pods: Jobs and CronJobs are used to run batch processes or scheduled tasks in Kubernetes. Jobs are suitable for one-time tasks, while CronJobs provide scheduling capabilities for recurring tasks. 
- Empty pods: Empty pods are pods that don't run any containers. They can be used as a placeholder or as a mechanism to reserve resources for future use.

### References
https://kubernetes.io/docs/concepts/workloads/pods/

---

## Selectors

### Labels

- Labels are key/value pairs that are attached to objects, such as Pods.
- Key-value pairs used to identify, describe, and group related sets of objects or resources.
- Labels can be attached to objects at creation time and subsequently added and modified at any time.
- Example
  - labels:
    - app: myapp
    - type: front-end
- Other example labels
  - "release" : "stable"
  - "environment" : "dev"
  - "tier" : "cache"
  - "partition" : "customerB"
  - "track" : "weekly"

### Selectors
- Selectors use labels to filter or select objects:
  -  match a set of resources based on their labels.
  - defining relationships between resources.
  - grouping related resources together.
- The API currently supports two types of selectors: _equality-based_ and _set-based_.
  - **equality-based**
  environment = production
  tier != frontend
  - **set-based**
  environment in (production, qa)
  tier notin (frontend, backend)
  partition
  !partition
- A label selector can be made of multiple requirements which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical AND (&&) operator.
- Node selector: **Make a pod to run in a specific node.**

### Reference
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

---

