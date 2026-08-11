# K3s Edge Transition

K3s installs Traefik by default. Expense Tracker uses Envoy Gateway instead, so
K3s Traefik must be disabled before the Argo root application installs Envoy Gateway.

The K3s server configuration is kept in this repository, but it cannot be applied by
Argo CD: Argo itself depends on the K3s control plane. Applying this one host-level
file is therefore a deliberate, reviewed bootstrap action, not a console-only change.

## Important compatibility note

The Hetzner cluster currently runs K3s `v1.35.4+k3s1`. This K3s release removes its
Gateway API CRDs when an already-installed Traefik AddOn is disabled. The cluster has
no Expense Tracker Gateway API resources yet, so the brief CRD absence is safe. The
first Argo sync immediately reinstalls compatible Gateway API and Envoy Gateway CRDs
through the pinned Envoy Gateway chart.

Do not apply this transition after creating `Gateway`, `HTTPRoute`, or other Gateway
API resources without planning their migration first.

## Preflight

Run these checks with the Hetzner kubeconfig. Stop if any existing route relies on
Traefik or Gateway API.

```powershell
kubectl get nodes
kubectl get ingress,httproute,gateway --all-namespaces
```

This configuration must be installed on every K3s **server** node. The current
cluster has one control-plane server. If a second server is added later, install the
same file there before it joins the cluster.

## Apply the versioned K3s configuration

From the root of this repository, replace the placeholder with the SSH user and host
of the Hetzner K3s control plane. The source file stays under version control; only
the copy and service restart happen on the host.

```powershell
$ControlPlane = '<ssh-user>@<control-plane-host>'
scp .\bootstrap\k3s\config.yaml.d\90-expense-tracker-edge.yaml "${ControlPlane}:/tmp/90-expense-tracker-edge.yaml"
ssh $ControlPlane 'sudo install -D -m 0644 /tmp/90-expense-tracker-edge.yaml /etc/rancher/k3s/config.yaml.d/90-expense-tracker-edge.yaml'
ssh $ControlPlane 'sudo systemctl restart k3s'
ssh $ControlPlane 'sudo systemctl is-active k3s'
```

The drop-in uses `disable+` instead of `disable` so it augments rather than replaces
any existing disabled packaged components in earlier K3s configuration files. K3s
loads drop-ins alphabetically from `/etc/rancher/k3s/config.yaml.d/` and documents
this append behavior in its [configuration reference](https://docs.k3s.io/installation/configuration).

## Validate and continue immediately

Wait for the control plane to recover, then confirm Traefik is gone. A `NotFound`
result for Traefik is expected.

```powershell
kubectl get nodes
kubectl -n kube-system get deployment traefik
```

Do not edit K3s's generated Traefik manifest. K3s recreates generated manifests at
startup. The supported removal mechanism is the `disable: traefik` server setting,
as described in the [K3s packaged-components documentation](https://docs.k3s.io/installation/packaged-components).

Next, follow [Cluster Platform Bootstrap](cluster-bootstrap.md) without delay. Its
first Argo sync installs Envoy Gateway and restores the Gateway API CRDs needed for
future `Gateway` and `HTTPRoute` manifests.
