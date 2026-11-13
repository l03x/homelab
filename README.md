# homelab

Kubernetes (k3s) manifests for everything I self-host, on two single-node clusters:

- **ilyass-server**: an old i3-2120 box (3.7 GB RAM, 250 GB HDD) on my home network, reachable on the LAN at `192.168.11.200` and on my tailnet at `ilyass-server.taila311b0.ts.net`. Public sites go out through a Cloudflare Tunnel, so no ports are forwarded on the router.
- **desktop**: my Ubuntu PC, running a few personal tools bound to `localhost`.

Each directory is one project, deployable on its own with `kubectl apply -k <dir>`.

## Projects

| Project | Runs on | What it is | Docs |
|---|---|---|---|
| [k3s](k3s/) | both | Installing and configuring the clusters, plus common operations | [README](k3s/README.md) |
| [cloudflared](cloudflared/) | server | Cloudflare Tunnel that publishes the public sites | [README](cloudflared/README.md) |
| [portfolio](portfolio/) | server | My portfolio site (`sefaultd`) | [README](portfolio/README.md) |
| [umami](umami/) | server | Privacy-friendly web analytics for the portfolio | [README](umami/README.md) |
| [ntfy](ntfy/) | server | Push notifications (alerts from monitoring and watchers) | [README](ntfy/README.md) |
| [reactive-resume](reactive-resume/) | server | Resume builder | [README](reactive-resume/README.md) |
| [n8n](n8n/) | server | Workflow automation (webhooks come in through the tunnel) | [README](n8n/README.md) |
| [media](media/) | server | Jellyfin and the *arr stack (Sonarr, Radarr, Prowlarr, Bazarr, Seerr, qBittorrent, FlareSolverr) | [README](media/README.md) |
| [monitoring](monitoring/) | server | Minimal Prometheus for the Freelens charts | [README](monitoring/README.md) |
| [headlamp](headlamp/) | server | Optional web dashboard (Freelens is the main UI) | [README](headlamp/README.md) |
| [jupyter](jupyter/) | desktop | Jupyter notebook for gen-AI experiments | [README](jupyter/README.md) |
| [claude-headless-api](claude-headless-api/) | desktop | OpenAI-compatible API over Claude Code headless mode | [README](claude-headless-api/README.md) |
| [linkedin-outreach](linkedin-outreach/) | desktop | Backend for my LinkedIn message-drafting Chrome extension | [README](linkedin-outreach/README.md) |

## Conventions

- **One namespace per project.** Deleting a project can never take another one's data with it.
- **Split files by kind for single-service apps** (`namespace.yaml`, `pvc.yaml`, `deployment.yaml`, `service.yaml`), and **by component for multi-service apps** (`postgres.yaml`, `app.yaml`, ...). A `kustomization.yaml` ties each directory together.
- **Secrets never live in git.** Each project with secrets has a `.env.example` (or `server.yml.example`). Copy it to the real file, which is gitignored, and fill it in. kustomize turns it into a Secret with a hash suffix, so changing a value and re-applying restarts the pods that use it.
- **No quotes in `.env` files.** kustomize keeps quotes as part of the value.
- **Storage:** app data lives on `local-path` PVCs under `/var/lib/rancher/k3s/storage/`, and shared media is a `hostPath` (`/srv/data`). See [k3s/README.md](k3s/README.md#protect-the-data) to stop PVC deletion from deleting data.
- **Host ports:** each port is declared **once**. Two entries with the same `containerPort` (for example, one per `hostIP`) get merged into one by `kubectl apply`. Any Deployment with a `hostPort` uses `strategy: Recreate`, because a rolling update can't start a second pod on a port the first still holds.
- **`enableServiceLinks: false`** everywhere, so Kubernetes doesn't inject `<SERVICE>_PORT=tcp://...` variables (these crash Jupyter, for example).
- **Memory requests match real usage, and limits are caps.** The server has 3.7 GB, and requests are what the scheduler reserves.
