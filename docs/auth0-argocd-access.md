# Auth0 Access for Argo CD

## Decision

Expose the Argo CD web UI at `https://argocd.tatalovic.dev` and use a separate Auth0
**Regular Web Application** for its sign-in. This application is distinct from the
Expense Tracker browser SPA because Argo CD is a server-side confidential client and
needs a client secret. A browser SPA must never receive that secret.

## Intended sign-in flow

```text
Browser -> argocd.tatalovic.dev -> Argo CD -> Auth0 Universal Login
        <- authenticated session <- Auth0
        -> Argo CD RBAC checks the signed-in administrator identity
        <- authorized Argo CD UI
```

Argo CD will use Auth0's OpenID Connect support directly. Its client secret will be
stored in Doppler and synced only to the `argocd` Kubernetes namespace. It will not
be committed to Git or exposed to the Expense Tracker web client.

## Auth0 application configuration

Create this after DNS and TLS work:

| Auth0 setting | Value |
| --- | --- |
| Name | `ExpenseTracker Argo CD` |
| Application type | Regular Web Application |
| Allowed Login URL | `https://argocd.tatalovic.dev/login` |
| Allowed Callback URL | `https://argocd.tatalovic.dev/auth/callback` |
| Allowed Logout URL | `https://argocd.tatalovic.dev` |
| Allowed Web Origin | `https://argocd.tatalovic.dev` |

The exact tenant domain and client ID are non-secret configuration. The client secret
is a Doppler value, such as `ARGOCD_AUTH0_CLIENT_SECRET` in the `dev` configuration.

## Authorization policy

Authentication alone is not authorization. The first deployment maps the owner email
to Argo CD's administrator role and gives no default application permissions to other
authenticated users. Before sharing access, create an Auth0 role/group claim and map
that group to a deliberately smaller Argo CD role.

This avoids a public administrative dashboard where any Auth0 user can see cluster
or deployment metadata.

## Deployment configuration

Argo CD will receive:

- Auth0 issuer: `https://dev-xn7atexont37avle.us.auth0.com/`
- the Argo CD client ID
- requested scopes: `openid`, `profile`, and `email`
- a secret reference for the Auth0 client secret
- `https://argocd.tatalovic.dev` as its externally visible URL

The future manifest must keep the default Argo CD policy empty or deny-by-default,
then map the owner identity explicitly. It must not give `role:admin` to every Auth0
user.

## Bootstrap safety

The initial local Argo CD `admin` password is only a bootstrap recovery mechanism.
After Auth0 works, disable or rotate it according to the Argo CD installation
procedure. Do not expose the UI until TLS and Auth0 configuration both work.

## References

- [Argo CD Auth0 guidance](https://argo-cd.readthedocs.io/en/release-3.5/operator-manual/user-management/auth0/)
- [Argo CD OIDC and secret references](https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/)
