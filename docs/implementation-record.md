# Implementation Record

This chronological record captures changes to the delivery platform, its operating
assumptions, and validation evidence. It is intentionally concise so work can resume
without reconstructing decisions from chat history.

## Current status

**Active milestone:** Development application environment

**Next milestone:** Publish application images, then add the development database,
API, web, Doppler sync, and routes as GitOps-managed application manifests.

## 2026-08-15 - Shared platform bootstrap verified; development rollout bounded

### Changed

- Applied the tracked K3s Traefik disablement on the control-plane server and
  bootstrapped the shared platform from `K3sPlatformGitOps`.
- Established Envoy Gateway, cert-manager with Cloudflare DNS-01, Doppler Kubernetes
  secret sync, and Auth0-protected Argo CD at `argocd.tatalovic.dev`.
- Defined the first Expense Tracker environment as development-only:
  `expense-tracker` namespace plus the dedicated Doppler `dev_k3s` configuration.
- Reserved production for a later independent rollout; no production resources are
  declared by this repository.

### Validation

- The shared Argo CD applications are Healthy and Synced after the cert-manager
  reconciliation.
- `argocd.tatalovic.dev` is reachable over publicly trusted HTTPS and requires the
  configured Auth0 sign-in flow.

### Follow-up

- Publish immutable API and web images to GHCR without direct cluster credentials.
- Add the development PostgreSQL StatefulSet, controlled migration Job, API, web,
  Doppler secret definitions, and public development routes.
- Record end-to-end validation evidence before starting production planning.

## 2026-08-09 - K3s edge transition declared

### Changed

- Added a versioned K3s server drop-in that disables the bundled Traefik AddOn without
  replacing any earlier `disable` settings.
- Added an edge-transition runbook that keeps the one required host-level change
  reviewable and repeatable.
- Made the K3s-to-Envoy ordering explicit in the bootstrap documentation and
  architecture boundary.

### Validation

- The Hetzner cluster kubeconfig reaches K3s `v1.35.4+k3s1`; its control-plane and
  two workers are Ready, and K3s Traefik is currently installed.
- K3s documents alphabetical configuration drop-ins and the `+` append syntax.
- K3s documents that disabling an existing Traefik AddOn on this release removes
  Gateway API CRDs. The pinned Envoy Gateway chart installs compatible Gateway API
  and Envoy Gateway CRDs during the first Argo sync.

### Follow-up

- Review and merge this declaration before changing the control plane.
- Apply the tracked drop-in only while no workload relies on Traefik or Gateway API.
- Install Argo CD and apply the root application immediately afterward.

## 2026-08-09 - Platform applications declared for Argo CD bootstrap

### Changed

- Added pinned Argo CD Applications for cert-manager `v1.21.1`, Envoy Gateway
  `v1.8.3`, and Doppler Kubernetes Operator `1.7.1`.
- Added the version-controlled Argo CD Helm values and a PowerShell-first bootstrap
  runbook.
- Kept public DNS, TLS issuance, Auth0 configuration, Doppler credentials, and
  application workloads out of the first platform sync.

### Validation

- `kubectl kustomize argocd/apps` renders the Argo project and all three platform
  Applications.
- The exact Helm chart versions were checked against their official repositories.

### Follow-up

- The workstation's current `kubectl` configuration falls back to `localhost:8080`.
  Restore or select the Hetzner kubeconfig before attempting the bootstrap commands.
- Verify the K3s version, default storage class, installed Traefik state, and node
  health before exposing Envoy Gateway or scheduling PostgreSQL.

## 2026-08-09 - GitOps repository and public endpoint strategy established

### Changed

- Created `damirtat/ExpenseTrackerGitOps` as the dedicated source of desired
  Kubernetes state for the Expense Tracker platform.
- Chose `tatalovic.dev` host names for development and eventual production routes.
- Chose an Auth0-protected Argo CD UI at `argocd.tatalovic.dev`.
- Chose a temporary, single-replica in-cluster PostgreSQL StatefulSet for development
  while managed PostgreSQL is unavailable.

### Validation

- The API and web client have already completed a local, authenticated browser flow
  through Auth0 and local PostgreSQL.
- The Hetzner cluster is known to contain one K3s control-plane and two worker nodes;
  access will be re-verified before bootstrap.

### Follow-up

- Do not expose Argo CD until Cloudflare DNS, cert-manager TLS, Auth0 OIDC, and
  deny-by-default Argo RBAC are configured together.
- Do not treat the in-cluster PostgreSQL instance as production storage.

## Working-entry template

```markdown
## YYYY-MM-DD - Short work item title

### Changed
- What changed and why.

### Validation
- Commands, tests, or observable evidence.

### Follow-up
- Intentional deferrals, risks, or decisions still needed.
```
