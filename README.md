# follow this 
```
https://docs.github.com/en/actions/tutorials/use-actions-runner-controller/get-started?search-overlay-input=authentication&search-overlay-ask-ai=true
```


# GitHub Actions Runners on Kubernetes

> This guide explains how to deploy self-hosted GitHub Actions runners on a Kubernetes cluster using the Actions Runner Controller (ARC).

## Overview

- Create or use an existing Kubernetes cluster.
- Deploy the ARC controller.
- Create a GitHub personal access token.
- Deploy the runner scale set.
- Use the runner in your workflow.

## 1. Prerequisites

- A working Kubernetes cluster.
- `kubectl` configured to talk to that cluster.
- `helm` installed locally.
- A GitHub repository where you want to use self-hosted runners.

## 2. Deploy the ARC controller

```bash
# Set the namespace where the ARC controller will run
NAMESPACE="arc-systems"

# Install the ARC controller using the Helm chart
helm install arc \
  --namespace "$NAMESPACE" \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

## 3. Create a GitHub personal access token

- Open GitHub and go to Settings -> Developer settings -> Personal access tokens.
- Create a token with the required repository permissions.
- Store it safely because it will be used during runner setup.

## 4. Deploy the runner scale set

```bash
# Name of the runner installation
INSTALLATION_NAME="arc-runner-set"

# Namespace for the runner pods
NAMESPACE="arc-runners"

# GitHub repository URL where the runner will register
GITHUB_CONFIG_URL="https://github.com/<OWNER>/<REPO>"

#copy the github token to a .env file for security

cat <<EOF > .env
token=<YOUR_PERSONAL_ACCESS_TOKEN>
EOF


# Load variables from .env first if you prefer to keep secrets out of the shell history
set -a
source .env
set +a

# Read the GitHub token from the .env file
GITHUB_PAT="${token}"

# Install the runner scale set and connect it to GitHub
helm install "${INSTALLATION_NAME}" \
  --namespace "${NAMESPACE}" \
  --create-namespace \
  --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
  --set githubConfigSecret.github_token="${GITHUB_PAT}" \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

## 5. Use the runner in GitHub Actions

```yaml
jobs:
  build-and-test:
    runs-on: arc-runner-set
```

## 6. Helpful verification commands

```bash
# Check that the ARC controller is running
kubectl get pods -n arc-systems

# Check that runner pods are created
kubectl get pods -n arc-runners
```

## Extra note

- If the installation fails, verify that your Kubernetes cluster is reachable and that Helm can access the chart repository.
- For production use, prefer storing secrets in a secret manager or a `.env` file that is not committed to source control.


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