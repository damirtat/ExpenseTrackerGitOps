# Implementation Record

This chronological record captures changes to the delivery platform, its operating
assumptions, and validation evidence. It is intentionally concise so work can resume
without reconstructing decisions from chat history.

## Current status

**Active milestone:** Cluster platform bootstrap

**Next milestone:** Validate the local Kubernetes context, then install Argo CD and
bootstrap the cluster platform.

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
