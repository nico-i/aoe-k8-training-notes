# Lab: kubectl

## `kubectl api-resources`

```bash
kubectl api-resources
```

- list all resources in the cluster
- `kubectl api-resources -o wide` show additional information
- Headers
  - `NAME`: name of the resource
  - `SHORTNAMES`: short names of the resource
  - `APIVERSION`: version of the resource
  - `NAMESPACED`: if the resource has a namespace
  - `KIND`: kind of the resource

## `kubectl explain`

```bash
kubectl explain pod
```

- show the documentation of a resource
- `--recursive` show all fields of the resource
- `--api-version <version>` show the documentation of a specific version

## `kubectl get`

```bash
kubectl get <kind>
```

- list all resources of a kind
- `kubctl get pods` list all pods in default namespace
  - `--all-namespaces` list all pods in all namespaces
  - `-n` or `--namespace` list all pods in a specific namespace
  - `-o <output>` change the output format (e. g. wide, json, yaml)

## `kubectl describe`

```bash
kubectl describe pod <pod-name>
```

- show detailed information about a resource

## `kubectl run`

```bash
kubectl run nginx --image=nginx
```

- create a new deployment with a pod
- `--image` specify the image to use
  - the image is pulled from the default registry (Docker Hub)

## `watch kubectl get pods`

```bash
watch kubectl get pods
```

- watch the output of a command
- `watch` is a command that runs a command repeatedly
  - `watch -n 1 kubectl get pods` run the command every second
  - `watch -d kubectl get pods` highlight the differences between the outputs
- watch is a third-party command and not part of kubectl

## `kubectl logs`

```bash
kubectl logs <pod-name>
```

- show the logs of a pod
- `-f` follow the logs (live output)

## `kubectl exec`

```bash
kubectl exec -it <pod-name> -- /bin/bash
```

- execute a command in a running container
- `-it` interactive mode
  - can be used to enter a shell in the container

## `kubectl create`

```bash
kubectl create <kind> <name>
```

- create a new resource
- `kubectl run` must be used to create a pod

## `kubectl run`

```bash
kubectl run <name> --image=<image>
```

- create a new deployment with a pod
- `--image` specify the image to use
  - the image is pulled from the default registry (Docker Hub)

## `kubectl edit`

```bash
kubectl edit <kind> <name>
```

- edit a resource in the default editor (usually vi)

## `kubectl apply`

```bash
kubectl apply -f <file>
```

- should be the preferred way to create or update resources
- apply a configuration file to the cluster
- updates or creates a resource
- `kubectl apply -f <file>` apply a configuration file
- `kubectl apply -f <directory>` apply all configuration files in a directory
- `kubectl apply -f <url>` apply a configuration file from a URL
- `kubectl apply -f -` apply a configuration file from stdin
- **sends a PATCH request to the API server, which in turn updates the etcd database with the new configuration**

## `kubectl delete`

```bash
kubectl delete <kind> <name>
```

- delete a resource
