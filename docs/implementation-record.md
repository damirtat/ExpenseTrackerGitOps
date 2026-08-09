# Implementation Record

This chronological record captures changes to the delivery platform, its operating
assumptions, and validation evidence. It is intentionally concise so work can resume
without reconstructing decisions from chat history.

## Current status

**Active milestone:** GitOps foundation

**Next milestone:** Argo CD and cluster platform bootstrap after review of the
foundation branch.

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
