# Validation and test evidence

These results reflect the completed lab observations, not a continuously running demo.

| Validation | Observed result |
|---|---|
| Standard Load Balancer backend health | Both web backends displayed **100% healthy** |
| Public HTTP access | Requests reached the web tier |
| Nginx `/api` reverse proxy | Requests reached the private application VM |
| Private application response | JSON included `"status": "healthy"` |

Recorded example:

```json
{
  "message": "Hello from Azure Application Tier!",
  "server": "vm-app-01",
  "status": "healthy"
}
```

## Reproducible checks

Replace placeholders with values from a deployed lab:

```bash
curl -i http://<LOAD_BALANCER_PUBLIC_IP>/
curl -i http://<LOAD_BALANCER_PUBLIC_IP>/api
```

On a web VM:

```bash
sudo nginx -t
sudo systemctl status nginx
curl -i http://127.0.0.1/api
```

In Azure Portal, inspect Load Balancer backend health and the configured HTTP probe.

## Evidence still to attach

See [screenshot checklist](../screenshots/README.md). Screenshots are not yet present.

## Claims intentionally excluded

No demonstrated forced VM failover, no zero-downtime claim, no load-testing or SLO results, and no automated recovery workflow.
