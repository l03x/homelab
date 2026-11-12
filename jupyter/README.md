# jupyter

Jupyter (minimal-notebook, Python 3.12) for gen-AI experiments. Runs on the **desktop** cluster.

## Deploy

```bash
cp .env.example .env           # set JUPYTER_TOKEN
kubectl apply -k .
kubectl -n jupyter get pods -w
```

Open `http://localhost:8888` and log in with the token.

## Notes

- `enableServiceLinks: false` is required. Otherwise Kubernetes injects `JUPYTER_PORT=tcp://...` for the `jupyter` Service, and Jupyter crashes on startup trying to parse it as a port.
- Everything, including `work/`, lives in the `home` PVC mounted at `/home/jovyan`. Unlike a Docker volume, a new PVC starts empty: it isn't pre-filled from the image.
- Copy notebooks in with `kubectl -n jupyter cp ./notebooks/. deploy/jupyter:/home/jovyan/work/`.
