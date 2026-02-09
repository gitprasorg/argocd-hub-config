Argo CD Hub Config
===================

Overview
--------

This repository contains configuration and manifests for deploying an Argo CD "hub" (control plane) and associated ApplicationSets, projects, and bootstrap apps. It includes Helm values and templates for installing Argo CD, K8s manifests for examples, and `kind` cluster configuration for local testing.

Repository layout
-----------------

- `argocd-hub-config/` — Argo CD configuration, ApplicationSets, apps, and Helm chart values.
  - `applicationsets/` — ApplicationSet definitions (example: `np-bjgs.yaml`).
  - `apps/` — Bootstrap kustomize/app manifests (example: `bootstrap.yaml`).
  - `argocd-installs/` — Helm chart values and templates for Argo CD installs (example: `np-bjgs/`).
  - `projects/` — Argo CD Project manifests.
- `kind-config/` — `kind` cluster configuration and MetalLB manifests (`kind-hub.yaml`, `metallb-native.yaml`, etc.).
- Other top-level files: `spoke1.kubeconfig`, `spoke1-cluster-config.json`, test manifests.

Quick start
-----------

Prerequisites:

- `kubectl`
- `kind` (for local testing)
- `helm`
- `argocd` or `argo` CLI (optional for UI access)

Create a local `kind` cluster (example):

```bash
kind create cluster --config kind-config/kind-hub.yaml
kubectl apply -f kind-config/metallb-native.yaml
```

Install Argo CD via Helm using the provided values (adjust release/name/namespace as needed):

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd --create-namespace \
  -f argocd-hub-config/argocd-installs/np-bjgs/values.yaml
```

Bootstrap apps and ApplicationSets:

```bash
# Apply bootstrap apps (kustomize or manifests)
kubectl apply -f argocd-hub-config/apps/bootstrap.yaml

# Apply ApplicationSets
kubectl apply -f argocd-hub-config/applicationsets/np-bjgs.yaml
```

Key files and locations
-----------------------

- Argo CD Helm values and chart: `argocd-hub-config/argocd-installs/np-bjgs/values.yaml` and `Chart.yaml`
- Argo CD application template: `argocd-hub-config/argocd-installs/np-bjgs/templates/argocd-app.yaml`
- ApplicationSet examples: `argocd-hub-config/applicationsets/np-bjgs.yaml`
- Bootstrap apps: `argocd-hub-config/apps/bootstrap.yaml`
- kind and MetalLB config: `kind-config/kind-hub.yaml`, `kind-config/metallb-native.yaml`

Usage notes
-----------

- Edit Helm values under `argocd-hub-config/argocd-installs/*/values.yaml` to customize ingress, service type, admin password, repository credentials, and other settings.
- Ensure clusters have appropriate labels/secrets if ApplicationSets target clusters by label or use external kubeconfigs.

Troubleshooting
---------------

- Access Argo CD UI (port-forward example):

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# then open https://localhost:8080
```

- Check pods and controller logs:

```bash
kubectl get pods -n argocd
kubectl logs -n argocd deploy/argocd-application-controller
kubectl logs -n argocd deploy/applicationset-controller
```

CI / Automation
---------------

- See `.github/workflows` for example workflows that automate Application creation and repository management.

Contributing
------------

PRs are welcome. Follow existing patterns for ApplicationSets and chart values. Use the issue templates under `.github/ISSUE_TEMPLATE` for feature requests or environment additions.

License
-------

See the repository `LICENSE` at the project root.

----
