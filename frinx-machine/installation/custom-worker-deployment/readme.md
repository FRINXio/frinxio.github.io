---
expanded: false
order: 1995
label: Custom worker deployment
---

# Custom worker deployment

To deploy a custom worker to Frinx Machine, utilize the generic worker Helm chart. 
Select the Helm chart version that corresponds with your application version, ensuring it is aligned with the Frinx Machine version.

You can find the necessary Helm chart at: [Helm Chart for Frinx Workers](https://artifacthub.io/packages/helm/frinx-helm-charts/worker).

### Create the Chart Configuration

Create the Chart.yaml file in new folder:

```yaml
# Chart.yaml
apiVersion: v2
name: custom-worker
description: Kubernetes deployment of custom worker
type: application
version: 0.1.0
maintainers:
  - name: FRINX
dependencies:
  - condition: custom-worker.enabled
    name: worker
    alias: custom-worker
    repository: https://FRINXio.github.io/helm-charts
    version: 4.0.0
  - condition: another-worker.enabled
    name: worker
    alias: anothers-worker
    repository: https://FRINXio.github.io/helm-charts
    version: 4.0.0
```

### Customize the Values

Create the values.yaml file next to Chart.yaml:

```yaml
# values.yaml
x-frinx-rbac-admin-role: &frinx-rbac-admin-role "FRINXio"

# -- use dependency alias from Chart.yaml
custom-worker:
  enabled: true

  # -- override deployment name
  fullnameOverride: "custom-workers"

  image:
    # -- use your image name
    repository: your/custom-worker
    # -- use your image tag
    tag: "tag"

  env:
    # -- set worker RBAC based on Frinx Machine RBAC settings
    # -- yaml anchor can be used to set same value for multiple workers
    X_AUTH_USER_GROUP: *frinx-rbac-admin-role

another-worker:
  enabled: true

  fullnameOverride: "another-workers"

  image:
    repository: your/another-worker
    tag: "tag"

  env:
    X_AUTH_USER_GROUP: *frinx-rbac-admin-role

```

### Deploy charts

```
helm dependency build
helm upgrade --install -n frinx custom-worker . -f values.yaml
```

### Useful Links
[!ref minikube image load](https://minikube.sigs.k8s.io/docs/commands/image/)

[!ref kind image load](https://minikube.sigs.k8s.io/docs/commands/image/)