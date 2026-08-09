# Cluster Platform Bootstrap

This runbook is intentionally split into a small manual bootstrap and ongoing GitOps
reconciliation. It must be executed only after the local Kubernetes context points to
the Hetzner K3s cluster.

## Preconditions

Run these read-only checks first. They must show the Hetzner nodes, not
`localhost:8080`.

```powershell
kubectl config get-contexts
kubectl cluster-info
kubectl get nodes -o wide
kubectl get storageclass
kubectl get pods -A
```

Also verify the firewall currently permits this workstation to reach the K3s API on
port `6443`. A dynamic home IP may require updating the Hetzner firewall before these
commands work.

## One-time Argo CD installation

The values file is version-controlled, but this initial install is manual because
Argo CD cannot reconcile itself before it exists.

```powershell
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade --install argocd argo/argo-cd --version 10.3.0 --namespace argocd --create-namespace --values .\bootstrap\argocd-values.yaml
kubectl -n argocd rollout status deployment/argocd-server --timeout=5m
kubectl apply -f .\bootstrap\root-application.yaml
kubectl -n argocd get applications
```

The root application creates the dedicated `expense-tracker` Argo project, then its
child applications install cert-manager, Envoy Gateway, and the Doppler Kubernetes
Operator. The dependency order is declared through Argo sync waves.

## What the first sync does not do

It does not create public DNS records, issue a certificate, expose the Argo CD UI,
create a Doppler token Secret, configure Auth0, or deploy PostgreSQL. Those require
separate reviewable manifests and credentials that do not belong in Git.

## Validation after the first sync

```powershell
kubectl -n argocd get applications
kubectl -n cert-manager get pods
kubectl -n envoy-gateway-system get pods
kubectl -n doppler-operator-system get pods
kubectl get crd | Select-String -Pattern 'cert-manager.io|gateway.networking.k8s.io|secrets.doppler.com'
```

At this point the platform control plane is installed, but it is not yet reachable
from the internet. Use a temporary port-forward for bootstrap-only Argo CD access;
do not rely on the initial Argo `admin` account for normal operation.

```powershell
kubectl -n argocd port-forward service/argocd-server 8080:80
```

The next milestone adds Cloudflare DNS validation, the Envoy `Gateway`, Auth0 OIDC
configuration, and deny-by-default Argo RBAC before `argocd.tatalovic.dev` is made
public.
