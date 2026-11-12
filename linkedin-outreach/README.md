# linkedin-outreach

Backend for my LinkedIn message-drafting Chrome extension (FastAPI with SQLite). It calls [claude-headless-api](../claude-headless-api/) for generation. Runs on the **desktop** cluster.

## Deploy

Deploy claude-headless-api first.

```bash
cp /path/to/linkedin-message-extension/.env .env    # see .env.example for the k3s changes
kubectl apply -k .
kubectl -n linkedin-outreach get pods -w
```

## Access

The extension and the dashboard keep using `http://localhost:8765`. The backend reaches the LLM at `http://claude-api.claude-api:8000`. Cluster DNS names only resolve inside the cluster, so the browser extension itself can't use them.

## Notes

- The SQLite database (system prompt, sending stats) lives in `/data` on the `backend-data` PVC. Copy the old Docker volume in while the backend is stopped.
- If the GHCR image is private, create a pull secret and add `imagePullSecrets` to the Deployment.
