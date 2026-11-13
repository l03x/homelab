# cloudflared

Cloudflare Tunnel connector. It runs with `hostNetwork`, so every public hostname is routed to `http://localhost:<port>` on the server, wherever the app runs. No router ports are forwarded.

## Deploy

```bash
cp .env.example .env           # paste the tunnel token
kubectl apply -k .
kubectl -n cloudflared logs deploy/cloudflared | grep -i "registered tunnel connection"
```

Expect about 4 `Registered tunnel connection` lines.

## Routes (Zero Trust dashboard -> Networks -> Tunnels -> Public Hostnames)

| Hostname | Service |
|---|---|
| portfolio | `http://localhost:8081` |
| umami | `http://localhost:3000` |
| ntfy | `http://localhost:8090` |
| resume | `http://localhost:4000` |
| n8n | `http://localhost:5678` |

The tunnel is token-run, so routes live in the dashboard and not in a config file.

## Notes

- The tunnel accepts several connectors at once, so a new connector can join before the old one is removed, with zero downtime.
- Metrics listen on `127.0.0.1:20241` only.
- If k3s is down, every public site is down.
