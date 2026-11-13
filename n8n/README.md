# n8n

Workflow automation, with SQLite on a PVC. Runs on the **server**, published through the tunnel so webhooks from outside can reach it.

## Deploy

```bash
cp .env.example .env           # set the key (openssl rand -hex 32) and the public URL
kubectl apply -k .
kubectl -n n8n get pods -w
curl -s http://127.0.0.1:5678/healthz
```

**Create the owner account before adding the tunnel route.** Until someone does, the first visitor to the editor becomes the owner. From the client:

```bash
ssh -L 5678:127.0.0.1:5678 192.168.11.200
```

Open `http://localhost:5678`, create the owner, then add the route: the tunnel routes the n8n hostname to `http://localhost:5678`.

## Backup and restore

Everything (the SQLite DB, binary data, the generated `config`) lives in the `data` PVC at `/home/node/.n8n`. Workflows and credentials can also be exported with the CLI:

```bash
kubectl -n n8n exec deploy/n8n -- n8n export:workflow --all --output=/home/node/.n8n/backup/workflows.json
kubectl -n n8n exec deploy/n8n -- n8n export:credentials --all --output=/home/node/.n8n/backup/credentials.json
kubectl -n n8n cp $(kubectl -n n8n get pod -l app=n8n -o name | cut -d/ -f2):/home/node/.n8n/backup ./n8n-backup
```

Exported credentials are still encrypted, so they only import into an instance with the same `N8N_ENCRYPTION_KEY`.

## Notes

- **`N8N_ENCRYPTION_KEY` must stay the same** across restores and migrations. Credentials in the database are encrypted with it. If the PVC already has a `config` file with a different key, n8n refuses to start with a key mismatch.
- `enableServiceLinks: false` is required. Otherwise Kubernetes injects `N8N_PORT=tcp://...` for the `n8n` Service, which overrides the real port setting.
- `WEBHOOK_URL` and `N8N_EDITOR_BASE_URL` are the public `https://` URL. Don't set `N8N_PROTOCOL=https`: that makes n8n serve TLS itself, while the tunnel terminates TLS and talks plain HTTP to `localhost:5678`.
- The editor is public along with the webhooks. To lock it down, put a Cloudflare Access policy on the hostname with a bypass for `/webhook/*` and `/webhook-test/*`.
- Pin the image to a release tag once it works. The tag of the running image is in `kubectl -n n8n get pod -l app=n8n -o jsonpath='{.items[0].status.imageID}'`.
- Other pods can call webhooks in-cluster at `http://n8n.n8n:5678/webhook/...`.
