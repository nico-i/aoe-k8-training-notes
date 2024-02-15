# K8s Training Day 1

## Introduction and Concepts

### Container

- not only Docker (only founder)
  - OCI (Open Container Initiative) also support k8s
- K8s uses ContainerD (basically Docker without build and push)
- No OS, no Kernel
- Divided into layers

### History

1. Dedicated server (hardware)
2. Virtualization
   - 1 server with Hypervisor (emulates hardware such as Network, Storage, CPU)
   - VM with own Kernel and OS
   - multiple apps per VM
3. Containers
   - No Hypervisor (a lot faster)
   - 1 server with Container Runtime (Docker, ContainerD)
   - 1 app 1 container

### Advantages of containers

- Isolation
- Lightweight
- Start quickly
- Portable (can run on any OS after build)

### Disadvantages

- Sharing Kernel (can cause competition for resources)
- Orchestration (multiple containers on different servers)

### Orchestration

- Single hosts should be managed by Docker-compose to avoid unnecessary complexity
- Management of containers across multiple hosts
  - Necessary for Cloud environments (containers across multiple servers)
- Facilitates Scaling, Load Balancing, Self-Healing
- Most famous tool: Kubernetes
  - Others: Docker Swarm, Mesos, Nomad

### Lab: Container

[see lab-container.md](labs/lab-container.md)

## Kubernetes

- open source
- originally developed by Google
  - donated to CNCF (Cloud Native Computing Foundation)
- declarative configuration via YAML _resources_
  - event-based
  - listens for updates or creations of resources
  - controllers can subscribe to these events
- no GUI
- large ecosystem

## K8s Clusters

- Consists of Nodes
  - Master Node (Control Plane)
    - API Server
    - Scheduler
    - Controller Manager
    - etcd
    - Cloud Controller Manager (optional)
  - Worker Node(s)

### Node

- bare metal or virtual server
- has Kubelet, Container Runtime (Docker, ContainerD), Kube-Proxy

### Kubelet

- agent that runs on each node
- monitors the node (CPU, Memory, Disk, Network)
- communicates with the control plane about the health of the node
- receives instructions from the scheduler to start or stop containers

### Kube-Proxy

- maintains network rules on nodes
- forwards traffic to the correct container based on IP and port number
- can do simple load balancing

### Control Plane

- can be viewed as the brain of the k8s
- responsible for making global decisions about the cluster (e. g. scheduling)
- In a high availability setup, the control plane runs on multiple nodes separate from the worker nodes
- Consists of
  - API Server
  - Scheduler
  - Controller Manager
  - etcd
  - Cloud Controller Manager (optional)

### etcd

- etcd is an acronym for "extended distributed"
- comparable to a Redis database
- persistent, lightweight, distributed, and reliable key-value store
- stores the configuration data of the cluster
- always runs on the master node
- stores the resources of the cluster (e. g. Pods, Services, ConfigMaps, Secrets, etc.)

### API Server

-

### Scheduler

- watches for newly created Pods with no assigned node

## k8s Tools

### `kubeadm`

- CLI application to bootstrap a k8s cluster and setup nodes

### `kubectl`

- CLI application to interact with the k8s cluster

### Running k8s

- `minikube` for local development
- `k3s` for edge computing
- `DevSpace` for local development
- Providers
  - AWS
  - Private Cloud Managed
    - Civo (very cheap)

## Declarative configuration

- Declarative: "I want 3 replicas of my app running"
  - k8s spec YAML file is declarative
- Imperative: "Create a pod with 3 replicas"
  - Dockerfile is imperative
- Provide a system with a desired state and the system will configure itself to match that state
- YAML file with opinionated structure
  - API Version
  - Kind
  - Metadata
  - Spec

## Lab: kubectl

[see lab-kubectl.md](labs/lab-kubectl.md)

- helm can be regarded as a templating engine for k8s

## Labels and Annotations

- part of the metadata of a resource

### Labels

- usually used to **tag** resources
- no predefined labels
- can be used to select resources
  - `kubectl get pods -l app=nginx`
    - `app` is the label, while `nginx` is the label value -> includes all pods with the label app=nginx
  - `kubectl get pods -l 'app, app notin (nginx)'`
    - includes all pods with the label app, but not the label app=nginx
- key-value pairs

### Annotations

- used to store non-identifying information
- sometimes used by tools to have additional configuration space
  - e. g. Prometheus uses annotations to store additional information about the scrape configuration

## Namespaces

- used to **group / categorize** resources
  - example: namespace per team
- also allows to add physical resources (e. g. CPU, RAM) to a namespace

### Initial namespaces

These are the initial namespaces that are created when a k8s cluster is created:

- `default`
  - where resources are created if no namespace is specified
- `kube-system`
  - internal components of a k8s node
- `kube-public`
  - resources that are publicly available
- `kube-node-lease`
  - heartbeats of the nodes
  - used to determine if a node is still alive

In most cases these namespaces are not modified.

### Labels and Annotations: Tips & Tricks

- don't use the `default` namespace
- specify namespaces in the manifest
- work out the namespace strategy before deploying resources
- the default namespace k8s uses can be changed to your own namespace

## Lab: Namespaces

[see lab-namespaces.md](labs/lab-namespaces.md)

## Pods

- smallest deployable unit in k8s
- can contain one or more containers
- containers in a pod share the same network namespace (IP, port space)
- containers in a pod can communicate with each other using `localhost`
  - a use case would be to have a web server and a logging agent in the same pod
- pods cannot be modified
  - if a pod needs to be modified, it needs to be deleted and a new pod with the new configuration needs to be created
- pods are ephemeral
  - they can be deleted and recreated at any time
  - pods are not suitable for storing data

### Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: nginx
```

### Container States

- each container in a pod has a state
- `Waiting`
  - container is starting, restarting, or is in a state of waiting
- `Running`
  - container is running
- `Terminated`
  - container has stopped running
  - could be due to a failure or a successful completion

### Pod Phases

- `Pending`
  - pod has been accepted by the k8s cluster, but the container has not been created yet
- `Running`
  - at least one container is running
- `Succeeded`
  - all containers in the pod have terminated successfully
- `Failed`
  - all containers in the pod have terminated, but at least one container has terminated in failure
- `Unknown`
  - the state of the pod cannot be determined (in theory this is possible but rarely happens)

### Pod Conditions

- pods can be in have different conditions (at the same time)
- each condition gives information about the pod
- `PodScheduled`
  - the pod has been scheduled to a node
- `ContainersReady`
  - all containers in the pod are ready
- `Initialized`
  - all containers in the pod have been initialized
- `Ready`
  - the pod is ready to serve requests
  - it is included in the service's load balancer

## Restart Policy

- defines if and when a container should be restarted
- `Always` (default)
  - the container will be restarted if it terminates
- `OnFailure`
  - the container will be restarted if it terminates with a non-zero exit code
- `Never`
  - the container will not be restarted if it terminates
- The restart is tried according to an exponential delay (10s, 20s, 40s, 80s, ...) up to a maximum of 5 minutes
  - delay is reset after 10 minutes
  - after 5 minutes no more restarts are attempted

## Pods: Tips & Tricks

- pods cannot / should never be changed at runtime; they should be deleted and recreated
- `kubectl describe pod <pod-name>` shows detailed information about a resource and is vital for debugging
- pods should always be managed with a deployment or a stateful set and not manually

## Lab: Pods

[see lab-pods.md](labs/lab-pods.md)

## Multi-Container Pods

### Use-cases

- sidecar pattern
  - e. g. a logging agent that runs alongside the main application, a proxy, or a file sync, etc.
- init containers
  - e. g. a container that runs before the main application starts
  - can be used to prepare the environment for the main application
  - accessing secrets, downloading files, waiting for dependencies to be ready, etc.
  - defined in the `initContainers` field of the pod manifest
- IPC (Inter-Process Communication)
  - e. g. a shared volume between two containers
  - can be used to share data between containers
  - shared memory is an anti-pattern according to the 12-factor app methodology

## Container Probes

- k8s can check the health of a container using _health checks_ via HTTP, TCP, GRPC or shell scripts
- depending on the type of the probe, k8s can restart the container if it is unhealthy or also remove it from the load balancer
- probes are configured on a per container basis
- types
  - `StartupProbe`
    - probes after container is started
    - terminates the container if the probe fails
  - `ReadinessProbe`
    - periodically checks if the container is ready
    - only begins probing after StartupProbe is successful
  - `LivenessProbe`
    - checks if the container is still running e. g. with a HTTP request
    - if the probe fails, the container is restarted

## Resource Management

- each container can be assigned a certain amount of CPU and memory
- this is done by either specifying the resource limits or requests
- `requests` are the amount of resources the container needs to start
  - requested resources are guaranteed to be available
  - container may consume more resources than requested
- `limits` are the maximum amount of resources the container can use
  - limits are enforced and cannot be exceeded
- resources:
  - CPU
    - compute time available to the container
  - Memory
    - RAM available to the container

### Units

- CPU
  - 1 is equivalent to 1 vCPU on AWS, GCP, Azure, etc.
  - 1 is equivalent to 1 hyperthread on a physical CPU
  - 1 is equivalent to 1000m (milliCPU)
  - 1m is equivalent to 0.001 CPU
- Memory
  - 1Gi = 1024Mi
  - 1Mi = 1024Ki
  - 1Ki = 1024B
  - 1k = 1000
  - 1M = 1000k
  - 1G = 1000M
  - no unit is equivalent to bytes

### Resource Quotas

- can be used to limit the amount of resources a namespace can consume

Example namespace quota:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

### Resource Management: Tips & Tricks

- requests and limits should be close to each other
- containers should be benchmarked to determine the correct amount of resources and monitored in production to adjust the resource requests and limits if necessary
- too much requests -> unnecessary costs
- too little requests -> instability

## Lab: Multi-Container Pods

I was the presenter of this lab. Therefore I don't have a markdown file for this lab.

## ReplicaSets

- layer between a deployment and a pod
- ensures that a specified number of pod replicas are running at all times
- defined in a resource file of the type `RelplicaSet` which is reference in a deployment
- using the kubectl `scale` command just updates the `replicas` field in the manifest yaml

### ReplicaSets: Tips & Tricks

- don't use ReplicaSets directly, use Deployments instead

## StatefulSets & DaemonSets

- StatefulSets
  - used for stateful applications or applications that require sequential deployments
  - e. g. databases, key-value stores, etc.
  - each pod has a unique identity
  - pods are created in order and deleted in reverse order
  - pods are not interchangeable
  - pods have a stable network identity
  - pods have stable storage (volumes are not deleted when the pod is deleted)
- DaemonSets
  - used when a copy of a pod needs to run on all (or some) nodes
  - ensures that all (or some) nodes run a copy of a pod
  - used for system daemons (e. g. log collectors, monitoring agents, etc.)
  - pods are created on all nodes and are deleted when the node is deleted

## Configuring Applications

## `env` and `envFrom`

- containers can be provided with an `env` field in the manifest
- the `env` field is an array of key-value pairs
- the `envFrom` field can be used to provide environment variables from a `ConfigMap` or a `Secret`

### ConfigMaps

- k8s resource
- used to store non-sensitive configuration data
- can be used to store environment variables, command-line arguments, configuration files, etc.
- can be mounted as a volume

### Secrets

- k8s resource
- used to store sensitive configuration data
- not encrypted, only base64 encoded (solutions exist to encrypt secrets)
- can be mounted as a volume

## Networking: Internal Access

### Kube-proxy

- maintains network rules on nodes
- is a DaemonSet
- responsible for the routing of traffic to the correct pod

### Pod Networking

- `pod-ip-address.namespace.pod.cluster.local`
  - only internal access
- each pod is assigned an IP by the CNI (Container Network Interface)

### Services

- used to expose one or more pods via label selectors
- k8s resource
- internal access via `servicename.namespace.svc.cluster.local`
- types
  - `ClusterIP`
    - default
    - only accessible within the cluster
  - `NodePort`
    - static port on each node
    - included clusterIP
  - `LoadBalancer`
    - exposes and external load balancer
    - cloud controller manager required
    - included NodePort and ClusterIP
  - `ExternalName`
    - CNAME on external DNS record

### Networking: Tips & Tricks

- do not use the pod DNS
  - always create a service

## Lab: Networking

[see lab-networking.md](labs/lab-networking.md)

## Networking: External Access

- the previous lab covered internal access to a pod by creating a service

### `kubectl port-forward`

- used to forward a local port to a port on a pod
- useful for debugging
- not suitable for production
- supports HTTP and TCP
- forwards requests to workloads in the cluster

### Ingress

- HTTP / HTTPS from outside the cluster to services inside the cluster
- k8s resource
- requires an installed Ingress Controller (not installed by default)
  - e. g. Nginx, Traefik, HAProxy, etc.
  - multiple Ingress Controllers can be installed at the same time
  - `ingressClassName` can be used to specify which Ingress Controller should be used
- Since the spec of the Ingress resource should be the same for all Ingress Controllers, Controller providers use annotations to add additional configuration

### Ingress Controller

- listens for Ingress resources
- opens ports and routes traffic to the correct service on a node
- ofter put after a load balancer to enable routing traffic to the correct node / replica
- configures the actual http server specified in the `ingressClassName` field

## Lab: External Access

[see lab-external-access.md](labs/lab-ingress.md)

## Storage

- k8s supports different types of storage

### Volume Mounts

- used to mount storage to a container
- local fs
  - container root fs
    - use when data does not need to be persistent
    - can be set to read-only
  - HostPath
    - Access to Node
    - Security risk
    - should be avoided
    - only for Single-Node tests
  - emptyDir
- cloud fs
  - depends on the cloud provider
    - AWS EBS
    - Azure Disk
    - Google Cloud Disk
    - CephFS
    - Longhorn
  - will not be deleted when the pod is deleted
  - independent of node storage
  - integration with cloud backup solutions
- k8s objects
  - ConfigMap
  - Secret

## Autoscaling

- **HPA (Horizontal Pod Autoscaler)**
  - k8s resource
  - automatically scales the number of pods in a deployment, replica set, or stateful set
  - based on CPU or custom metrics
  - can be used to scale up or down
  - can be used to scale to a minimum and maximum number of pods
- **VPA (Vertical Pod Autoscaler)**
  - k8s resource
  - automatically adjusts the CPU and memory requests and limits of a container
  - based on the actual usage of the container
  - can be used to scale up or down
  - can be used to scale to a minimum and maximum number of pods
  - kind of an **anti-pattern**
  - deletes and recreates the pod
- VPA and HPA should not be used together when observing the same metric
- **CA (Cluster Autoscaler)**
  - after the HPA has scaled up the pods, the CA can scale up the nodes
  - can scale infinitely in theory (only limited by the cloud provider and you wallet)

## Authentication and Authorization

- It's necessary to secure the k8s cluster, k8s provides different methods for this
- **Authentication**
  - the process of verifying the identity of a user or a system
  - k8s supports different authentication methods
    - JSON Web Tokens (JWT)
    - OIDC (OpenID Connect)
- **Authorization**
  - the process of verifying if a user or a system has the necessary permissions to access a resource

### Subjects

- Identities that can be assigned roles
- types
  - User
  - Group
  - Service Account

### ServiceAccount

- k8s resource
- upon creation k8s creates credentials for the service account

### Roles

- k8s resource
- used to define a set of permissions
  - permissions are standardized in the form of verbs
  - e. g. `get`, `list`, `watch`, `create`, etc.

### ClusterRoles

- globally defined role
- mostly used for users whereas roles are used for service accounts

### RoleBindings

- Roles are assigned to one or more subjects via RoleBindings
- only applies to the ns in which it is defined
- Role must exist before it is used in a RoleBinding
- `roleRef.kind` must be `Role` or `ClusterRole`

### ClusterRoleBindings

- not namespaced
- rights apply globally
- `roleRef.kind` must be `ClusterRole`

### RBAC: Tips & Tricks

- Access Control Types
  - ABAC (Attribute-Based Access Control)
    - if you aren't an admin you probably won't have any contact points with this
  - RBAC (Role-Based Access Control)
    - recommended
- Role & RoleBinding
  - namespaced
- ClusterRole & ClusterRoleBinding
  - global

## Lab: Authentication and Authorization

[see lab-auth.md](labs/lab-auth.md)

## Deployment Strategies

### Rolling Updates

- default deployment strategy provided by k8s
- updates the pods in a rolling fashion
  - new pods are created and the old pods are deleted
- disadvantages
  - little influence on rolling behavior
- configured within a deployment resource
  - `strategy` field in spec
  - `type: RollingUpdate`
  - `rollingUpdate` field
    - `maxSurge`
      - maximum number of pods that can be created above the desired number of pods
    - `maxUnavailable`
      - maximum number of pods that can be unavailable during the update
  - doesn't need to be specified, since it's the default
- `kubectl rollout undo deployment <deployment-name>` can be used to roll back a deployment
- sufficient for most use cases

### Canary Deployments

- traffic is gradually shifted to the new version
- can be done manually or automatically
  - no native support in k8s
  - third-party tools can be used
    - e. g. Argo CD, Istio
    - also requires metric provider
- runs over the ingress
- advantages
  - can be used to test new features
  - easy to roll back
  - low impact on end users
- disadvantages
  - complex to set up (automation must be set up)
  - both versions in parallel use

### Blue-Green Deployments

- two identical environments
- one is active, the other is inactive
- the inactive environment is updated
- traffic is shifted to the updated environment
- advantages
  - easy to roll back
  - no downtime
  - easy to test
- disadvantages
  - double the resources

### Deployment Strategies: Tips & Tricks

- use a backwards-compatible database
- setup probes to ensure the new version is healthy
- `kubectl rollout restart deployment <deployment-name>` can be used to restart a deployment
- `kubectl rollout status deployment <deployment-name>` can be used to check the status of a deployment

## Helm & Kustomize

- **Helm**
  - templating engine for k8s
  - uses Go templates
  - package manager for k8s
  - uses charts to define applications
  - charts are a collection of files that define a k8s application
  - charts can be stored in a repository
  - Helm can be used to install and upgrade applications
  - standalone tool
  - suitable for complex apps
- **Kustomize**
  - extends `kubectl apply`
  - only YAML
  - integrated into kubectl
  - good for apps you created yourself