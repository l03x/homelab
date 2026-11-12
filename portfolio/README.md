# portfolio

My portfolio site (`sefaultd`). The image is built locally and imported into k3s, never pulled. It keeps the hardening from the original Compose file: UID 65534, read-only root filesystem, all capabilities dropped, no privilege escalation, 64 Mi / 0.5 CPU.

## Deploy

```bash
# from the portfolio source repo
docker build -t portfolio-app:latest .
docker save portfolio-app:latest | sudo k3s ctr images import -

# from this directory
kubectl apply -k .
curl -sI http://127.0.0.1:8081 | head -1
```

The tunnel routes the portfolio hostname to `http://localhost:8081`. Port 8080 on the host is taken by qBittorrent.

## Update

Rebuild and import the image as above, then restart:

```bash
kubectl -n portfolio rollout restart deploy/sefaultd
```

`imagePullPolicy: Never` makes the pod fail clearly if the image wasn't imported, instead of pulling some other `portfolio-app` from Docker Hub.

## Notes

- Logs are written to the `logs` PVC (`/logs`).
- `pids_limit` from Compose has no per-pod equivalent in Kubernetes (only the node-wide `--pod-max-pids`).
