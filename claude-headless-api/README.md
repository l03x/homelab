# claude-headless-api

OpenAI-compatible API over Claude Code headless mode (`claude -p`), using my own subscription. Runs on the **desktop** cluster.

## Deploy

```bash
cp /path/to/claude-headless-api/.env .env    # see .env.example for the k3s changes
kubectl apply -k .
kubectl -n claude-api get pods -w
```

## Access

- From the desktop: `http://localhost:8000`
- From other namespaces: `http://claude-api.claude-api:8000` (`<service>.<namespace>`)

## Notes

- The Claude login lives in `/data` on the `claude-api-data` PVC. Copy the old Docker volume in before first use. See [Copy data into a PVC](../k3s/README.md#copy-data-into-a-pvc).
- If only in-cluster apps call it, remove the `hostPort`/`hostIP` lines so it isn't reachable on the host at all.
- If the GHCR image is private, create a pull secret and add `imagePullSecrets` to the Deployment.
