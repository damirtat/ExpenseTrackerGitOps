# GitOps Roadmap

This is a living implementation plan. Mark an item complete only after its validation
evidence has been recorded in `implementation-record.md`.

## Milestone 0: GitOps foundation

- [x] Create the dedicated GitOps repository.
- [x] Agree host names under `tatalovic.dev`.
- [x] Record ownership boundaries and the temporary development database policy.
- [x] Record the Auth0-protected Argo CD UI design.
- [ ] Add the Argo CD root application and AppProject.

## Milestone 1: Cluster platform bootstrap

- [ ] Verify current K3s access and node health.
- [ ] Install Argo CD once with a reviewed bootstrap command.
- [ ] Apply the root Argo CD application that watches this repository.
- [ ] Add cert-manager with a Cloudflare DNS-01 `ClusterIssuer`.
- [ ] Add Envoy Gateway and its public `Gateway`.
- [ ] Add Doppler Kubernetes secret sync.
- [ ] Configure `argocd.tatalovic.dev` with TLS, Auth0 OIDC, and deny-by-default RBAC.

## Milestone 2: Development application environment

- [ ] Create `expense-tracker-dev` namespace and resource quotas.
- [ ] Deploy single-replica PostgreSQL with a persistent volume, marked development-only.
- [ ] Store development database and application settings in Doppler.
- [ ] Add API Deployment, Service, health probes, and controlled migration Job.
- [ ] Add API `HTTPRoute` at `api.dev.expenses.tatalovic.dev`.
- [ ] Add web Deployment, Service, and `HTTPRoute` at `dev.expenses.tatalovic.dev`.
- [ ] Update Auth0 SPA callback/logout/origin settings for the development URL.
- [ ] Validate browser sign-in, household provisioning, API health, and database persistence.

## Milestone 3: Continuous delivery

- [ ] Replace the API Azure deployment workflow with immutable GHCR publishing.
- [ ] Add equivalent web image build and publishing workflow.
- [ ] Create a reviewed GitOps image-reference update flow after application `main` merges.
- [ ] Let Argo CD automatically synchronize only the merged development configuration.
- [ ] Validate rollback by reverting one GitOps image-reference commit.

## Milestone 4: Production readiness

- [ ] Choose and provision managed PostgreSQL.
- [ ] Migrate production configuration from in-cluster PostgreSQL to managed PostgreSQL.
- [ ] Add production overlays and the `expenses.tatalovic.dev` routes.
- [ ] Add backup, restore, monitoring, alerting, and access-review policies.
- [ ] Manage Auth0 tenant configuration with Auth0 Deploy CLI.
