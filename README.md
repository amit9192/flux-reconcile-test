# Flux Reconciliation Test Setup

This repository contains a setup to test FluxCD reconciliation with HelmReleases and Kustomize resources, specifically focusing on HPA and KEDA ScaledObjects.

## Prerequisites

1.  **Kubernetes Cluster**: A running cluster (local or remote).
2.  **Flux CLI**: Installed on your machine.
3.  **GitHub Repository**: You need to push this code to a Git repository so Flux can sync from it.

## Setup Instructions

1.  **Push to Git**:
    Create a new repository on GitHub (or your preferred git provider) named `flux-reconcile-test`.
    ```bash
    # Update the remote URL in clusters/my-cluster/source.yaml
    # Replace https://github.com/REPLACE_ME/flux-reconcile-test with your actual repo URL

    git remote add origin <your-repo-url>
    git push -u origin main
    ```

2.  **Bootstrap Flux**:
    Run the bootstrap command to install Flux on your cluster and point it to your repository.
    ```bash
    flux bootstrap git \
      --url=<your-repo-url> \
      --branch=main \
      --path=clusters/my-cluster
    ```
    *Alternatively, if you don't want to bootstrap and just want to install Flux and apply manifests manually:*
    ```bash
    flux install
    kubectl apply -f clusters/my-cluster/source.yaml
    kubectl apply -f clusters/my-cluster/helm-releases/
    kubectl apply -f clusters/my-cluster/kustomize/
    ```

## Experimentation

The goal is to observe how Flux reconciles changes made manually via `kubectl` vs. the state in Git.

### Scenarios

1.  **HPA Reconciliation**:
    *   **HelmRelease**: `podinfo-hpa`
    *   **Kustomize**: `podinfo-kustomize-hpa`
    *   **Action**: Manually edit the HPA to change `maxReplicas` or `minReplicas`.
    *   **Observation**: Watch if/when Flux reverts the change.

2.  **KEDA ScaledObject Reconciliation**:
    *   **HelmRelease**: `podinfo-keda`
    *   **Kustomize**: `podinfo-kustomize-keda`
    *   **Action**: Manually edit the ScaledObject triggers.
    *   **Observation**: Watch if/when Flux reverts the change.

### Useful Commands

```bash
# Watch Flux Kustomizations
flux get kustomizations --watch

# Watch Flux HelmReleases
flux get helmreleases --watch

# Trigger immediate reconciliation
flux reconcile kustomization podinfo-kustomize-hpa --with-source
flux reconcile helmrelease podinfo-hpa
```

