# headlamp

Optional web dashboard. Freelens with [monitoring](../monitoring/) is the main UI, so deploy this only if you want a browser-based one.

## Deploy

```bash
kubectl apply -k .
kubectl -n kube-system create token headlamp --duration=720h
```

Open `http://192.168.11.200:4466` and paste the token.

## Notes

- It runs as `cluster-admin`, so anyone with the token can change anything in the cluster. Keep it off untrusted networks.
- It binds to the LAN IP only. To bind it somewhere else, change the single `hostIP` (or remove it for all interfaces). Don't add a second port entry: `kubectl apply` merges entries with the same `containerPort`.
- Without metrics-server, Headlamp's CPU/RAM graphs are empty.
