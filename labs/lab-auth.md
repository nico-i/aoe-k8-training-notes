# Lab: Auth

## `kubectl auth can-i`

```bash
kubectl auth can-i <verb> <resource>
```

- `kubectl auth can-i get pods`
  - can I get pods?
- `kubectl auth can-i get pods --as <user>`
  - can `<user>` get pods?
  - `--as` flag is used to impersonate a user
- `kubectl auth can-i get pods --as <user> --as-group <group>`
  - can `<user>` get pods as a member of `<group>`?
  - `--as-group` flag is used to impersonate a group