# Development deployment runbook

The `expense-tracker-dev` Argo CD Application deploys the temporary development
environment into the `expense-tracker` namespace. It is intentionally separate
from production.

## Desired-state flow

1. Merge the shared-edge platform change. It creates the approved Gateway listeners
   and adds both development hostnames to the shared TLS certificate.
2. Bootstrap the read-only Doppler service token named
   `expense-tracker-dev-doppler-token` in the `doppler-operator-system` namespace.
   The token must be restricted to `expense-tracker-api/dev_k3s`.
3. Bootstrap the `ghcr-pull` Docker-registry Secret in `expense-tracker`. It is not
   stored in Git because it contains the GitHub Packages read credential.
4. Merge the application rollout. The existing `expense-tracker-root` Argo CD
   Application creates and continuously reconciles `expense-tracker-dev`.
5. Create Cloudflare DNS records for `dev.expenses.tatalovic.dev` and
   `api.dev.expenses.tatalovic.dev` pointing at the existing shared Envoy Gateway
   address. Confirm the Certificate and both HTTPRoutes are accepted before using
   the application.

## Workload order

- The Namespace and DopplerSecret resources sync first.
- PostgreSQL creates a single 10 GiB `local-path` PVC and stays internal to the
  cluster as `postgres`.
- The migration Job waits for PostgreSQL and runs the API image with `--migrate`.
- The API and Web Deployments use immutable GHCR commit tags. Normal API replicas
  explicitly keep startup migrations disabled.
- HTTP routes redirect plaintext HTTP to HTTPS and attach only to the platform
  Gateway listeners approved for this namespace.

## Development limitations

This PostgreSQL StatefulSet is a development-only, one-replica database. Its local
volume has no high-availability or backup guarantee. Do not copy this design to the
future production namespace.
