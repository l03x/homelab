# monitoring

A minimal Prometheus for the Freelens charts (about 150 to 250 MB of RAM in total): Prometheus, node-exporter and kube-state-metrics (nodes and pods only). It keeps only the metrics Freelens queries, with 3 days of retention.

## Deploy

```bash
kubectl apply -k .
kubectl -n lens-metrics get pods -w
```

Check that all three scrape jobs are up:

```bash
kubectl get --raw "/api/v1/namespaces/lens-metrics/services/prometheus:80/proxy/api/v1/targets" | grep -o '"scrapePool":"[^"]*"\|"health":"[^"]*"'
```

## Freelens

In the cluster settings, under **Metrics**, set **Prometheus** to **Lens**. It finds `lens-metrics/prometheus:80` automatically. Reconnect the cluster after installing.

## Prometheus UI

```bash
kubectl -n lens-metrics port-forward svc/prometheus 9090:80     # then http://localhost:9090
```

## Notes

- Editing `prometheus.yml` and re-applying restarts Prometheus automatically, because of the hash-suffixed ConfigMap.
- With this stack, k3s's metrics-server is disabled (`config.server.yaml.example`). `kubectl top` and k9s's CPU/MEM columns depend on metrics-server, so remove it from `disable:` if you want those back.
- node-exporter runs on every node (it's a DaemonSet). On a node that's often off, expect scrape errors while it's down.
