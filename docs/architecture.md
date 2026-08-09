# Deployment Architecture

## Scope

The initial outcome is a real development environment reachable through
`dev.expenses.tatalovic.dev`, deployed through Argo CD. The repository contains a
production-shaped layout from the outset, but production rollout is deferred until
the development path is proven.

```text
Browser
  |
  | HTTPS, Cloudflare DNS
  v
Envoy Gateway on Hetzner K3s
  |                              |
  | dev.expenses...              | api.dev.expenses...
  v                              v
React web client               ExpenseTracker API
                                      |
                                      v
                         PostgreSQL StatefulSet (development only)

Auth0
  |                              |
  | browser login                | administrator login
  v                              v
Web SPA                       Argo CD at argocd.tatalovic.dev

Doppler -> Kubernetes Secrets -> API, PostgreSQL, cert-manager, Argo CD
GitHub Actions -> GHCR -> GitOps image reference -> Argo CD -> K3s
```

## Responsibility boundaries

| Component | Responsibility | Managed by |
| --- | --- | --- |
| GitHub Actions | test and publish immutable images | application repositories |
| GHCR | retain published images | GitHub |
| GitOps repository | desired platform and workload configuration | this repository |
| Argo CD | reconcile declared Git state to the cluster | K3s, bootstrapped once |
| Envoy Gateway | public HTTP(S) routing | Argo CD after bootstrap |
| cert-manager | TLS certificates using Cloudflare DNS validation | Argo CD after bootstrap |
| Doppler operator | turn Doppler configuration into Kubernetes Secrets | Argo CD after bootstrap |
| PostgreSQL | temporary stateful development database | Argo CD after bootstrap |
| Cloudflare | DNS records for the public hosts | manual initial setup |
| Auth0 | browser authentication and Argo CD administrator sign-in | tenant configuration, later managed as code |

## Argo CD model

Argo CD watches this repository. Its `Application` resources say which repository
paths or Helm charts belong in the cluster. When Git says an API image or a platform
setting should change, Argo CD compares that desired state with the running cluster
and applies the difference. If a manual cluster edit drifts from Git, Argo can report
or correct that drift.

Argo CD does **not** build images, create Cloudflare DNS records, create Hetzner
servers, or invent secrets. Those responsibilities remain with GitHub Actions,
Cloudflare, Hetzner, and Doppler respectively.

## Development database policy

Development starts with the official PostgreSQL container image as a one-replica
StatefulSet and persistent volume. This is suitable for experimentation, migrations,
and real end-to-end development, but it has one node-local volume and no high
availability or backup guarantee.

Before production, the API will move to managed PostgreSQL. The API configuration
will continue to consume a connection string from Doppler, so the application and
deployment shape do not need to change when the provider changes.

## Deliberately deferred

- Production managed PostgreSQL and backup/restore policy.
- Highly available cluster control plane, database, and edge.
- Mobile delivery.
- Monitoring and alerting stack beyond workload health checks and Argo status.
- Auth0 Deploy CLI tenant management. The version-controlled Auth0 Action source in
  the API repository remains the recovery path until deployment credentials exist.
