# Lab: namespaces

## `kubectl create namespace anton`

- create a new namespace
- `kubectl apply -f namespace.yaml` also works

## `kubectl run nginx --image=nginx --namespace=anton`

- `--namespace` specify the namespace to use

## `kubectl get namespace`

- list all namespaces

## `kubectl get pods -n anton`

- list all pods in the namespace `anton`

## `kubectl delete namespace anton`

- delete a namespace
- cascading deletion -> all resources in the namespace are deleted