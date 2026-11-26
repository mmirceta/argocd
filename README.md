# argocd

Argo CD GitOps repo for the devops-training AKS clusters, side by side with
Flux ([mmirceta/fluxcd](https://github.com/mmirceta/fluxcd)). Infrastructure
(AKS, Gateway, cert-manager, external-dns, Argo CD itself) lives in
[devops-training-project](https://github.com/mmirceta/devops-training-project).

Argo CD and Flux must never manage the same objects. On dev, Argo CD owns
**nginx**; Flux owns apache and Headlamp.

## Layout (app of apps per environment)

```
<env>/<app>.yaml        Argo CD Application for one app (read by the root app)
<env>/<app>/            That app's manifests (read only by its Application)
```

| Folder | Status |
|---|---|
| `dev/` | aks-dev-training: `nginx` |
| `prod/` | Placeholder, no Argo CD on prod yet |

Every environment is self-contained: no shared base, an app is copied into
`prod/` when prod gets it. Terraform creates one root Application per
cluster that syncs the **top-level files** of `<env>/` (not recursive), so
the manifest folders next to them are never applied by the root app.

Add an app: create `<env>/<app>/` (plain YAML; every file in it is applied)
and `<env>/<app>.yaml` (copy `dev/nginx.yaml`, change name, path and
namespace). Remove an app: delete `<env>/<app>.yaml`; the root app prunes
the Application and its finalizer deletes the app's objects.

Compare with Flux: there, one Kustomization applies `<env>/` and apps are
listed in `<env>/kustomization.yaml`; here every app is its own Application
with its own sync status, history and rollback in the UI.

## Clusters

| Cluster | Root path | UI |
|---|---|---|
| aks-dev-training | `dev` | https://argo.mapko.dev |

Initial admin password (dev AKS API is private, so run it via AKS):

```bash
az aks command invoke -g rg-devops-training-dev -n aks-dev-training \
  --command "kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d"
```
