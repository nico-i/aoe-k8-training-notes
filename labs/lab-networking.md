# Lab: Networking

## `kubectl expose`

- expose a resource as a new Kubernetes Service
- shortcut command to create a service manifest

```bash
kubectl -n <namespace> expose <resource> --port <port> --name <service-name>
```

- `--port` specify the port of the service
- `--name` specify the name of the service
- `--dry-run=client` print the manifest `kubectl` will send to the cluster's API server to the console
  - `-o yaml` change the output format to yaml
  - `=server` prints the manifest that will be stored in the cluster's etcd