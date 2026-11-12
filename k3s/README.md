# k3s

Cluster setup for both machines, plus the operations shared by every project.

## Install

**Server** (Traefik and ServiceLB disabled: cloudflared handles ingress):

```bash
sudo mkdir -p /etc/rancher/k3s
sudo cp config.server.yaml.example /etc/rancher/k3s/config.yaml   # edit the tls-san names first
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable traefik --disable servicelb" sh -
```

**Desktop:**

```bash
sudo mkdir -p /etc/rancher/k3s
sudo cp config.desktop.yaml.example /etc/rancher/k3s/config.yaml
curl -sfL https://get.k3s.io | sh -
```

k3s runs as a systemd service and starts at boot. After changing `/etc/rancher/k3s/config.yaml`, run `sudo systemctl restart k3s`.

## kubectl as your user

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config && chmod 600 ~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.zshrc     # or ~/.bashrc
```

## Reaching the server from another machine (Freelens, kubectl)

Make two kubeconfigs on the client: one for the LAN, one for Tailscale.

```bash
ssh -t ilyass@192.168.11.200 'sudo cp /etc/rancher/k3s/k3s.yaml ~/k3s.yaml && sudo chown ilyass: ~/k3s.yaml'
scp ilyass@192.168.11.200:~/k3s.yaml ~/.kube/server.yaml && ssh ilyass@192.168.11.200 'rm ~/k3s.yaml'
sed -i 's/127.0.0.1/192.168.11.200/' ~/.kube/server.yaml && chmod 600 ~/.kube/server.yaml
cp ~/.kube/server.yaml ~/.kube/server-ts.yaml
sed -i 's#https://192.168.11.200:6443#https://ilyass-server.taila311b0.ts.net:6443#' ~/.kube/server-ts.yaml
kubectl --kubeconfig ~/.kube/server.yaml    config rename-context default server-lan
kubectl --kubeconfig ~/.kube/server-ts.yaml config rename-context default server-ts
```

The Tailscale name only works because it's in `tls-san` in the server config. These files give full admin access, so keep them only on trusted machines.

## Common operations

### Find a PVC's directory on disk

```bash
pvdir() { kubectl get pv $(kubectl -n "$1" get pvc "$2" -o jsonpath='{.spec.volumeName}') -o jsonpath='{.spec.hostPath.path}{.spec.local.path}'; }
pvdir umami db-data
```

Always check the result isn't empty before copying into it: `cp -a src/. "$(pvdir ns pvc)/"` copies into `/` when the lookup fails.

### Copy data into a PVC

Copy only while the app is stopped, and into an **empty** directory. Some apps, SeaweedFS for example, initialize fresh state on their first start, and copying old data on top of that corrupts it.

```bash
kubectl -n <ns> scale deploy <app> --replicas=0
D=$(pvdir <ns> <pvc>); echo "$D"
[[ "$D" == /var/lib/rancher/k3s/storage/* ]] && sudo find "$D" -mindepth 1 -delete && sudo cp -a /path/to/old/data/. "$D/"
kubectl -n <ns> scale deploy <app> --replicas=1
```

### Protect the data

`local-path` deletes a PV's directory when its PVC is deleted (`kubectl delete -k .` deletes the PVCs too). Switch every PV to `Retain` once:

```bash
kubectl get pv -o name | xargs -I{} kubectl patch {} -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

### Stop or start a project without deleting anything

```bash
kubectl -n <ns> scale deploy --all --replicas=0
kubectl -n <ns> scale deploy --all --replicas=1
```

### After changing ports in a Deployment

`kubectl apply` merges port lists by `containerPort`, so removing or renaming port entries can leave stale fields, such as a dropped `hostPort`. Recreate the Deployments instead of merging. PVCs, Services and data are untouched:

```bash
kubectl -n <ns> delete deploy --all
kubectl apply -k <project-dir>
```

### Clean up old generated Secrets

Each `.env` change creates a new hash-suffixed Secret. The old ones are harmless, and you can delete them:

```bash
kubectl -n <ns> get secrets
kubectl -n <ns> delete secret <old-name>
```

### Disk pressure

If pods sit in `Pending` with `untolerated taint ... disk-pressure`, free disk space (`docker builder prune -a`, `docker image prune -a`) or lower the eviction thresholds as in `config.desktop.yaml.example`.
