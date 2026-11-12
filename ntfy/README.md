# ntfy

Push notification server (alerts from Beszel, changedetection.io and similar).

## Deploy

```bash
cp server.yml.example server.yml     # set base-url, keep every path under /var/lib/ntfy
kubectl apply -k .
curl -d "hello" http://127.0.0.1:8090/test
```

The tunnel routes the ntfy hostname to `http://localhost:8090`. `server.yml` becomes a Secret, because it can hold web-push keys.

## Users and tokens

Users and tokens are stored in `/var/lib/ntfy/user.db` on the PVC:

```bash
kubectl -n ntfy exec -it deploy/ntfy -- ntfy user add --role=admin <name>
kubectl -n ntfy exec -it deploy/ntfy -- ntfy token add <name>
```

## Notes

- Editing `server.yml` and re-applying restarts ntfy automatically, because of the hash-suffixed Secret.
- Any path in `server.yml` outside `/var/lib/ntfy` isn't persisted.
