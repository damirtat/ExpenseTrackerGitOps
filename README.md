# Expense Tracker GitOps

This repository is the desired-state source for Expense Tracker Kubernetes
deployments. Argo CD reconciles these application manifests onto the K3s
cluster after the shared cluster platform is available.

Application code is kept separate:

- API: [`damirtat/ExpenseTrackerAPI`](https://github.com/damirtat/ExpenseTrackerAPI)
- Web: [`damirtat/ExpenseTrackerWeb`](https://github.com/damirtat/ExpenseTrackerWeb)

## Delivery model

```text
Application repositories                    This repository                 Hetzner K3s
------------------------                    ---------------                 -----------
PR -> test -> image in GHCR  ----image ref-> reviewed desired state  ->     Argo CD syncs it
                                                                          -> shared platform routes it
                                                                          -> shared platform provides TLS
                                                                          -> Doppler syncs app secrets
```

GitHub Actions builds and tests application images. It does not receive cluster
credentials or deploy directly to Kubernetes. A reviewed GitOps change selects the
image that should run; Argo CD applies and continuously reconciles that state.

## Planned host names

| Purpose | Development | Production |
| --- | --- | --- |
| Web client | `dev.expenses.tatalovic.dev` | `expenses.tatalovic.dev` |
| API | `api.dev.expenses.tatalovic.dev` | `api.expenses.tatalovic.dev` |
| Argo CD administration | `argocd.tatalovic.dev` | `argocd.tatalovic.dev` |

`argocd.tatalovic.dev` is an administrative endpoint, not part of the public
product. It will use Auth0 sign-in and Argo CD RBAC. No unauthenticated access or
default administrator credentials will be used after bootstrap.

## Repository rules

- Do not commit credentials, `kubeconfig` files, private keys, rendered secrets, or
  Doppler tokens.
- Runtime secrets are supplied by Doppler Kubernetes secret sync.
- The temporary development PostgreSQL instance runs as a single-replica StatefulSet
  with a persistent volume. It is explicitly not a production database.
- EF Core migrations run as a controlled Kubernetes Job before the API rollout, not
  automatically in every API replica at startup.
- Deployment references use immutable image digests or commit-derived tags, never
  `latest`.

See [architecture](docs/architecture.md), [roadmap](docs/roadmap.md), and the
[Auth0 Argo CD access design](docs/auth0-argocd-access.md).

## Initial bootstrap

The shared K3s platform is bootstrapped separately. Once it is healthy, apply
[`bootstrap/root-application.yaml`](bootstrap/root-application.yaml). That
application renders `argocd/apps/` as a Kustomize entrypoint. It currently
declares only the Expense Tracker Argo Project. Add the API and web child Argo
Applications there as their deployment manifests are introduced; Argo CD will
then reconcile those applications continuously.

During the one-time platform ownership handover, reconcile the root without
pruning before re-enabling automated sync. This prevents it from deleting
Applications that the shared platform root has already adopted.
