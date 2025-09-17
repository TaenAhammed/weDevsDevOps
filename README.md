# weDevs DevOps:
**End to End checklist to run the project:**

Each task has been grouped into multiple phases, added verification points, and included a minimal example manifest/command to adapt.

## Task 1: Cluster Setup and Application Deployment:
### Phase 1: Local k3s and FluxCD:
*NB:* In Mac, a Linux VM is required for running K3s. Otherwise, k3d could be an alternative. I'm using k3s with an Ubuntu VM through the multipass VM Orchestrator.

1.1: Spin up k3s locally
```bash
# Install k3s with Traefik disabled (needed because we must deploy ingress-nginx)
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -

# Add kubeconfig export(Mac specific if we use multipass VM):
multipass exec weDevsUbuntu -- sudo cat /etc/rancher/k3s/k3s.yaml > k3s.yaml
VMIP=$(multipass info weDevsUbuntu | awk '/IPv4/{print $2}')
sed -i.bak "s/127.0.0.1/$VMIP/g" ./k3s.yaml
export KUBECONFIG=$PWD/k3s.yaml

# Verification:
kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
wedevsubuntu   Ready    control-plane,master   43m   v1.33.4+k3s1
```

1.2: Bootstrap Flux v2 via CLI (GitOps)
```bash
flux check --pre
# Export variables and credentials
export GH_OWNER=TaenAhammed
export GH_REPO=weDevsDevOps
export GH_BRANCH=main
export SYNC_PATH=cluster
export GITHUB_TOKEN=<github_pat>

# run bootstrap
flux bootstrap github \
  --owner=$GH_OWNER \
  --repository=$GH_REPO \
  --branch=$GH_BRANCH \
  --path=$SYNC_PATH \
  --personal

# Verification:
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/TaenAhammed/weDevsDevOps.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed component manifests to "main" ("a417fd976b23dbca42f5e39b7fdf92a948cca57a")
► pushing component manifests to "https://github.com/TaenAhammed/weDevsDevOps.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBOYE8K9ZCtwZXVg/nkpoXuQFIXmOvs4O/ZwfpMP/Stx/jSwpP9n73HSQwqlrRB5DdzhMdg9EwCn4sv34L8qJr/OcIF6kL7WMRfNXVPYEzaDxzqBadSZFftSHk6jJ54IWpA==
✔ configured deploy key "flux-system-main-flux-system-./cluster" for "https://github.com/TaenAhammed/weDevsDevOps"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "main" ("0d9d72dad01062e440be3366e90a3ed475ef36e7")
► pushing sync manifests to "https://github.com/TaenAhammed/weDevsDevOps.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```