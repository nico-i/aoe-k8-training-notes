# Lab: Ingress

## `kubectl port-forward`

```bash
kubectl -n <namespace> port-forward <resource>/<resource-name> <local-port>:<resource-port>
```

## Getting the NodePort

```bash
`export NODE_PORT_HTTP=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[?(@.name=="http")].nodePort}')`
```

- `export` the `NODE_PORT_HTTP` environment variable
- `kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[?(@.name=="http")].nodePort}'`
  - get the `nodePort` of the `http` service of the `ingress-nginx-controller` service in the `ingress-nginx` namespace

## Accessing the Ingress

```bash
curl -H "Host: www.example.com" http://nodea:${NODE_PORT_HTTP}/
```
