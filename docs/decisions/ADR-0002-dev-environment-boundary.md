# ADR-0002: Keep the first K3s environment development-only

- Status: Accepted
- Date: 2026-08-15

## Context

Expense Tracker needs a publicly reachable, end-to-end K3s deployment before a
production rollout is safe. The cluster's shared services (Argo CD, Envoy Gateway,
cert-manager, and the Doppler Kubernetes Operator) are already owned by the separate
`K3sPlatformGitOps` repository. This repository owns only Expense Tracker workloads
and their application-specific configuration.

Creating production resources now would duplicate secrets, database state, DNS
records, and operational work before the development path has been proven.

## Decision

- Use the `expense-tracker` namespace as the first, explicitly development-only
  environment. Every workload in it carries an `environment: dev` label.
- Use the dedicated Doppler `expense-tracker-api/dev_k3s` configuration and a
  read-only service token for this environment. The token is stored only in the
  Doppler Operator namespace; synced target Secrets contain runtime values and are
  never committed here.
- Deploy no production namespace, database, Doppler configuration, route, or Argo CD
  Application during this rollout.
- When production is approved, create it as a separate `expense-tracker-prod`
  environment with independent Doppler credentials, Secrets, database, certificates,
  and routes. It must not reuse development data or credentials.
- Keep shared cluster services in `K3sPlatformGitOps`; do not install a separate
  cert-manager, Envoy Gateway, or Doppler Operator for Expense Tracker.

## Consequences

- `expense-tracker` is concise but must not be mistaken for production; labels,
  Application names, Doppler configuration names, and public development host names
  make its purpose explicit.
- Promotion to production is a deliberate, later rollout rather than a namespace
  copy. This avoids silently carrying development secrets or node-local database
  assumptions into production.
- Application manifests can rely on the already-running shared platform without
  taking ownership of it.
