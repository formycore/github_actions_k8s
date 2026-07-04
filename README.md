# github-actions-runners-k8s
Using self hosted Github Actions runners on Kubernetes

Official documentation:
- [Official Quickstart](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/quickstart)
- [Runner Scale Sets ](https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/deploy-runner-scale-sets)

## 1. Get Kubernetes Cluster

Create a Kubernetes cluster on a cloud platform (EKS, AKS, GKE, DOKS, etc) or use a self managed Kubernetes cluster.

## 3. Deploy ARC helm chart
```bash
helm install arc \
--namespace "arc-systems" \
--create-namespace \
oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

## 2. Create Personal Access Token on Github

Under `Developer Settings`, create a **Fine-Grained personal access token**.
Add the following permissions:
- Administration (read & write)
- Metadata (Read-only)

## 4. Deploy runner scale set
```bash
INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="<GITHUB_REPOSITORY>"
GITHUB_PAT="<PAT>"
helm install "${INSTALLATION_NAME}" \
--namespace "${NAMESPACE}" \
--create-namespace \
--set githubConfigUrl="${GITHUB_CONFIG_URL}" \
--set githubConfigSecret.github_token="${GITHUB_PAT}" \
oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

## 5. Set workflows to use the runners

```yaml
jobs:
  build-and-test:
    runs-on: arc-runner-set
```