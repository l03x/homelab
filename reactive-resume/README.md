# reactive-resume

Resume builder: the app, Postgres 18, Redis, and SeaweedFS (S3 storage for uploads).

## Deploy

```bash
cp .env.example .env           # fill in, then add the storage/Redis keys from upstream's .env.example
kubectl apply -k .
kubectl -n reactive-resume get pods -w
curl -sI http://127.0.0.1:4000 | head -1
```

The tunnel routes the resume hostname to `http://localhost:4000`.

## Migrating or restoring data

**Database** (stop the app first):

```bash
kubectl -n reactive-resume scale deploy reactive-resume --replicas=0
kubectl -n reactive-resume exec -i deploy/postgres -- pg_restore -U postgres -d postgres --clean --if-exists --no-owner < rr.dump
kubectl -n reactive-resume scale deploy reactive-resume --replicas=1
```

**SeaweedFS:** copy into an **empty** PVC directory while SeaweedFS is stopped. SeaweedFS writes a fresh topology on its first start, and old data copied on top of it fails with `Split-brain detected`. See [Copy data into a PVC](../k3s/README.md#copy-data-into-a-pvc), using namespace `reactive-resume` and PVC `seaweedfs-data`.

## Notes

- Postgres and SeaweedFS have **no host ports**, so only the app reaches them. In the old Compose setup, both were open to the LAN, with the superuser password and the default S3 keys.
- Postgres is pinned to major version 18, because the data on disk is tied to it.
- Keep `FLAG_DISABLE_SIGNUPS=true`, since the site is public.
- Pin the app image to a release tag once it works. The tag of the running image is in `kubectl -n reactive-resume get pod -l app=reactive-resume -o jsonpath='{.items[0].status.imageID}'`.
- Redis uses an `emptyDir`: its data (sessions, streams) is transient.
