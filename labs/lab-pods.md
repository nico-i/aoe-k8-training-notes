# Lab: pods

## Restarting a bad container

```bash
kubectl -n pods run restart --image ubuntu -- bash -c 'sleep 60 && echo "this is an error" && exit 1'`
```

- create a pod in the namespace `pods` named `restart` and the image `ubuntu`
- additionally run the command `bash -c 'sleep 60 && echo "this is an error" && exit 1'` in the container
  - the command sleeps for 60 seconds, then prints an error message and exits with a non-zero exit code
- the pod will be in the state `Running` and the container in the state `Terminated`
- the container will be restarted according to the default restart policy `Always`

## Debugging a pod

- run a default ubuntu pod with a tail command

```bash
kubectl -n pods run debug --image ubuntu -- tail -f /dev/null
```

- now you can use `kubectl exec` to enter the pod and debug the container

```bash
kubectl -n pods exec -it debug -- /bin/bash
```

or use `sh` when using the `alpine` image

```bash
kubectl -n pods exec -it debug -- /bin/sh
```
