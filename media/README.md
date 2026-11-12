# media

Jellyfin and the *arr stack: Sonarr, Radarr, Prowlarr, Bazarr, Seerr, qBittorrent and FlareSolverr, in one namespace.

## Host layout

```
/srv/data/torrents/{movies,tv}   # qBittorrent downloads
/srv/data/media/{movies,tv}      # library (hardlinked by Sonarr/Radarr)
```

`/srv/data` is a `hostPath`, mounted at the same paths inside the containers as in the original Compose file. It's one filesystem, so hardlinks work, and nothing in the apps' settings changes. App configs live on one PVC per app.

## Deploy

```bash
kubectl apply -k .
kubectl -n media get pods -w
```

## Access

Each web UI binds on all interfaces, so the same port works on the LAN and on the tailnet:

| App | LAN | Tailscale |
|---|---|---|
| Jellyfin | http://192.168.11.200:8096 | http://ilyass-server.taila311b0.ts.net:8096 |
| qBittorrent | http://192.168.11.200:8080 | http://ilyass-server.taila311b0.ts.net:8080 |
| Prowlarr | http://192.168.11.200:9696 | http://ilyass-server.taila311b0.ts.net:9696 |
| Sonarr | http://192.168.11.200:8989 | http://ilyass-server.taila311b0.ts.net:8989 |
| Radarr | http://192.168.11.200:7878 | http://ilyass-server.taila311b0.ts.net:7878 |
| Bazarr | http://192.168.11.200:6767 | http://ilyass-server.taila311b0.ts.net:6767 |
| Seerr | http://192.168.11.200:5055 | http://ilyass-server.taila311b0.ts.net:5055 |

FlareSolverr has no host port. Prowlarr reaches it at `http://flaresolverr:8191`. qBittorrent's 6881 (TCP and UDP) is open for torrent peers.

## Inside the apps

Each Service has its Compose service name, so connections configured as `http://qbittorrent:8080`, `http://flaresolverr:8191` and so on keep working. Anything configured with a Docker IP or with `localhost` must be changed to the service name.

## Stop and start

```bash
kubectl -n media scale deploy --all --replicas=0
kubectl -n media scale deploy --all --replicas=1
```

## Notes

- After changing ports, recreate the Deployments instead of applying over them. See [k3s/README.md](../k3s/README.md#after-changing-ports-in-a-deployment).
- FlareSolverr runs headless Chromium and grows towards its 768 Mi limit. It's stateless, so an OOM kill just resets it. If no Prowlarr indexer uses it, scale it to 0.
- The server sits behind the router's NAT, so "all interfaces" means the LAN and the tailnet, never the internet. Keep a DHCP reservation for `192.168.11.200`.
