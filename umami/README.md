# umami

Web analytics for the portfolio: Umami and Postgres 15.

## Deploy

```bash
cp .env.example .env           # fill in; generate values with openssl rand -hex 32
kubectl apply -k .
kubectl -n umami get pods -w
curl -s http://127.0.0.1:3000/api/heartbeat
```

The tunnel routes the umami hostname to `http://localhost:3000`.

## Restore from a dump

```bash
kubectl -n umami scale deploy umami --replicas=0
kubectl -n umami exec -i deploy/db -- pg_restore -U umami -d umami --clean --if-exists --no-owner < umami.dump
kubectl -n umami scale deploy umami --replicas=1
```

To take a dump: `kubectl -n umami exec deploy/db -- pg_dump -U umami -d umami -Fc > umami.dump`

## Notes

- **`TWO_FACTOR_ENCRYPTION_KEY` must stay the same** across migrations and restores. 2FA secrets in the database are encrypted with it. To rotate it: disable 2FA in Umami, change the key, re-apply, then re-enable 2FA.
- Postgres is pinned to major version 15, because the data on disk is tied to it. Upgrade with a dump and restore, never by changing the tag.
- The password must be URL-safe, because it's embedded in `DATABASE_URL`.
