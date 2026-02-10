# ArgoCD Hub Config - Overview

This folder contains the GitOps configuration for a hub-managed ArgoCD (non-prod), using an App-of-App pattern to provision per-team applications into spoke clusters.

Key concepts and file mappings

- ApplicationSet (`applicationsets/hub-config.yaml`)
  - Runs in the hub ArgoCD and watches `argocd-hub-config/apps/*` in this repo.
  - For each manifest it creates an Application on the hub which then applies that manifest into the spoke cluster's `argocd` namespace.
  - Uses `project: hub-np-bjgs` (a hub-side AppProject) so hub permissions are limited.
  - `syncOptions: [CreateNamespace=true]` is enabled so the `argocd` namespace on the spoke is created if missing.

- Hub AppProject (`projects/hub-np-bjgs.yaml`)
  - Controls what the hub-generated Applications can do (which repos and destinations are allowed).

- `argocd-hub-config/apps/np-bjgs.yaml` (App-of-App bundle)
  - Contains two CRs that the hub will apply into the spoke `argocd` namespace:
    1. `AppProject` named `np-bjgs` (spoke-side project) — this must exist in the spoke ArgoCD so the spoke enforces team boundaries.
    2. `Application` named `team1-sample-app` — this is the App-of-App which the spoke ArgoCD will manage; it points at the team repo.
  - Both use `destination.server: https://kubernetes.default.svc` so the spoke ArgoCD treats the target cluster as in-cluster.

- `argocd-hub-config/apps/sample-app-hub.yaml`
  - Example Application that the hub can also manage directly (for hub-owned resources).

Operational flow and repo layout

Recommended split: two repos (infra and hub-config)

Repo A — infra / argocd-install (platform-only)
- Purpose: manages installation and upgrades of the hub ArgoCD itself (Helm chart + `values.yaml`).
- Access: restricted to platform engineers. Use PRs, CI checks, and manual promotion before applying updates to non-prod or prod.

Repo B — hub-config (this repository)
- Purpose: manages hub ArgoCD configuration: `applicationsets/`, `apps/` (App-of-App bundles), and `projects/`.
- This repo should *not* contain the ArgoCD helm chart used to install/upgrade the hub ArgoCD; that lives in Repo A.

Flow summary

1. Manage hub ArgoCD installation via Repo A (infrastructure). Promotion/merge in Repo A triggers a controlled upgrade of the hub ArgoCD (manual sync or platform pipeline).
2. The hub ArgoCD reads Repo B (`gitprasorg/argocd-hub-config`) and applies configuration (ApplicationSets, AppProjects, App-of-App manifests) that manage spoke ArgoCDs and team apps.
3. ApplicationSet in the hub generates Applications that apply App-of-App bundles into each spoke's `argocd` namespace; the spoke ArgoCD then manages team apps in-cluster.

Notes on sync policy

- `root-apps` (bootstrap Application in this repo) is intentionally configured for manual sync. That prevents this repo from being used to upgrade the hub ArgoCD itself; upgrades are performed from Repo A.
- The hub-config repo may have automated sync for non-risky configuration (ApplicationSets, projects), but hub ArgoCD software upgrades should be gated.

Webhook / auto-sync recommendations

- Expose spoke ArgoCD `repo-server` (LoadBalancer/Ingress) so team repo webhooks can POST to `/api/webhook`.
- The App-of-App should have `spec.syncPolicy.automated` enabled so the spoke ArgoCD auto-syncs after a webhook triggers a repo refresh.

Verification commands

```bash
# on hub: show the ApplicationSet
kubectl -n argocd get applicationsets hub-config -o yaml

# on hub: list applications created by ApplicationSet
argocd app list

# on spoke (using spoke kubeconfig): check resources applied by hub
kubectl --kubeconfig=spoke1.kubeconfig -n argocd get appprojects,applications
kubectl --kubeconfig=spoke1.kubeconfig -n argocd get appprojects np-bjgs -o yaml
kubectl --kubeconfig=spoke1.kubeconfig -n argocd get application team1-sample-app -o yaml
```

Security notes

- Use a dedicated hub AppProject for hub-generated Applications to limit repo/destination scope.
- Prefer GitHub App + private-key authentication for repo access.
- Keep spoke AppProjects scoped to the minimal allowed repos and namespaces.
- Only register cluster credentials where necessary and avoid granting cluster-admin to ArgoCD service accounts.

Next steps (phase 2)

- Add the GitHub Actions workflow to create ApplicationSet/App-of-App artifacts when a team registers an app. (Not yet implemented here.)

