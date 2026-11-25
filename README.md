# argocd

Argo CD GitOps repo for the devops-training AKS clusters, side by side with
Flux ([mmirceta/fluxcd](https://github.com/mmirceta/fluxcd)). Infrastructure
(AKS, Gateway, cert-manager, external-dns, Argo CD itself) lives in
[devops-training-project](https://github.com/mmirceta/devops-training-project).

Argo CD and Flux must never manage the same objects. On dev, Argo CD owns
**nginx**; Flux owns apache and Headlamp.

## Layout (app of apps)

```
<env>/<app>.yaml        Argo CD Application per app; the root Application syncs <env>/
apps/<app>/base/        Environment-independent manifests
apps/<app>/<env>/       Kustomize overlay: base + env specifics (hostnames, ...)
```

Terraform creates one root Application per cluster that syncs `<env>/`
recursively as plain YAML, so `<env>/` holds only Application manifests;
Kustomize folders live under `apps/`. Add an app by creating its base and
overlay under `apps/` and an Application in `<env>/` pointing at the overlay.

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
