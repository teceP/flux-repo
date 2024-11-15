flux bootstrap github --context kind-test --owner=${GITHUB_USER} --repository=${GITHUB_REPO} --branch=main --personal --path=clusters/aaamedia



1. Install Flux in Cluster
1.1 Create Git Repository
1.2 Create Git PAT (Personal Access Token) with correct permissions: see in docs
1.3 Install Flux CLI if not present
1.5 Check kubectl context, switch if needed
1.6 Create namespace: kubectl create ns flux-system
1.7 Install Flux with bootstrap command:
flux bootstrap github --owner=teceP --repository=my-flux-repo --private=false --personal=true --path=clusters/my-cluster
1.8 Watch status (in correct context): kubectl get pods -A --watch

2. Install Capacitor (Flux UI)
2.1 Get YAML: https://fluxcd.io/blog/2024/02/introducing-capacitor/
2.2 Install YAML: kubectl -f <capacitor-yaml-file>
```shell
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: capacitor
  namespace: flux-system
spec:
  interval: 12h
  url: oci://ghcr.io/gimlet-io/capacitor-manifests
  ref:
    semver: ">=0.1.0"
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: capacitor
  namespace: flux-system
spec:
  targetNamespace: flux-system
  interval: 1h
  retryInterval: 2m
  timeout: 5m
  wait: true
  prune: true
  path: "./"
  sourceRef:
    kind: OCIRepository
    name: capacitor
```
2.3 kubectl -n flux-system port-forward svc/capacitor 9000:9000

From here, we will install monitoring and cluster managment tools before we will head on to the application charts.
Alls changes in state of our cluster will be kept here: https://github.com/teceP/my-flux-repo/tree/main/clusters/my-cluster/flux-system

3 Install Infrastructure Charts/Apps

kubectl create ns cert-manager
flux create source helm cert-manager --url https://charts.bitnami.com/bitnami --namespace cert-manager --export > ./cert-manager-source.yaml
flux create helmrelease cert-manager --chart cert-manager --source HelmRepository/cert-manager --chart-version 1.3.22 --namespace cert-manager --export > ./cert-manager-release.yaml





3.1 Add a Repo as source of Definitions
flux create source git podinfo \
  --url=https://github.com/stefanprodan/podinfo \
  --branch=master \
  --interval=1m \
  --export > ./podinfo-source.yaml

3.2 Add generated podinfo-source.yaml to flux definitions
git add -A && git commit -m "Add podinfo GitRepository"
git push

3.3 Deploy podinfo
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --wait=true \
  --interval=30m \
  --retry-interval=2m \
  --health-check-timeout=3m \
  --export > ./clusters/my-cluster/podinfo-kustomization.yaml

3.4 Add to git
git add -A && git commit -m "Add podinfo Kustomization"
git push

3.5 Watch sync
flux get kustomizations --watch

3.6 Check if has synced
kubectl -n default get deployments,services








Image Automations:
https://fluxcd.io/flux/components/image/imageupdateautomations/