---
expanded: false
order: 9999
---

# Helm Chart Installation Guide with customization

This guide provides the step-by-step instructions for installing FRINX Machine on a Kubernetes cluster using Helm charts with customization via helm dependency values.

## Prerequisites

- `Cluster`: Make sure that you cluster is running.
- `Helm`: Make sure that Helm is installed. Follow the Helm installation guide if necessary.


## Step 1: Add the FRINX Helm Repository

Add the FRINX Helm repository and update the repository list:

```bash
helm repo add frinx https://FRINXio.github.io/helm-charts
helm repo update
```

## Step 2: Install Operators and CRDs

Install FRINX Machine operators and custom resource definitions (CRDs):

```bash
helm install -n frinx --create-namespace frinx-machine-operators frinx/frinx-machine-operators
```

Verify the installation by checking the pods in the frinx namespace:

```bash
kubectl get pods -n frinx
```

You should see output similar to:

```bash
NAME                                                       READY   STATUS    RESTARTS   AGE
arango-frinx-machine-operators-operator-6dfdff75bd-cnwmp   1/1     Running   0          25s
arango-frinx-machine-operators-operator-6dfdff75bd-k8kqp   1/1     Running   0          25s
frinx-machine-operators-cloudnative-pg-d9566444c-85w8g     1/1     Running   0          25s
```

## Step 3: Create Docker Registry Secret

Please complete this step before continue [!ref icon="briefcase" text="Create docker regitry secret"](../docker-registry-secret/readme.md)

## Step 4: Customize FRINX Machine

To customize the deployment of FRINX Machine, you need to create a folder that will contain the necessary Helm chart files: Chart.yaml and values.yaml. This folder will serve as the root for your customized Helm chart, with FRINX Machine as a dependency.

### Create the Chart Configuration

1. Create the Chart.yaml file:

This file contains the metadata for your Helm chart and specifies FRINX Machine as a dependency.

```yaml
# Chart.yaml
apiVersion: v2
name: frinx-machine
description: Kubernetes deployment of FRINX-machine
icon: https://avatars.githubusercontent.com/u/23452093?s=200&v=4
type: application
version: 6.1.0
maintainers:
  - name: FRINX
dependencies:
  - name: frinx-machine
    repository: https://FRINXio.github.io/helm-charts
    version: 9.0.0
```

This configuration sets up the basic information about the Helm chart and defines FRINX Machine as a dependency, pulling it from the specified repository.

2. Check Dependency Chart Details:

For mapping the Helm chart release to the product release and more details on the FRINX Machine Helm chart, refer to the [FRINX Machine Helm chart documentation on Artifact Hub](https://artifacthub.io/packages/helm/frinx-helm-charts/frinx-machine).

### Customize the Values

3. Create the values.yaml file:

    This file contains custom configurations for the FRINX Machine subcharts. Customization options are based on the documentation of each subchart, which can be found on Artifact Hub.

```yaml
# values.yaml
frinx-machine:
  krakend:
    ingress:
      enabled: true
      className: nginx
      annotations:
        nginx.ingress.kubernetes.io/proxy-connect-timeout: "3600"
        nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
        nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
      hosts:
        - host: krakend.127.0.0.1.nip.io
          paths:
            - path: "/"
              pathType: ImplementationSpecific

  uniconfig:
    image:
      repository: "frinxio/uniconfig"

  performance-monitor:
    image:
      repository: "frinxio/performance-monitor"
```

  - krakend: Configures the Ingress settings for the KrakenD API Gateway. It enables the Ingress, sets the class to nginx, and includes custom annotations for proxy timeouts. The host is set to krakend.127.0.0.1.nip.io with the path /.

  - uniconfig: Specifies the Docker image repository for the Uniconfig component.

  - performance-monitor: Specifies the Docker image repository for the Performance Monitor component.

### Customize Subcharts

4. Subchart Customization:

    For more detailed customization possibilities, refer to the [documentation for each dependency Helm chart version](https://artifacthub.io/packages/search?org=frinx&sort=relevance&page=1). This will provide information on all configurable parameters and how to adjust them to suit your deployment needs.


By following these steps, you can effectively customize the deployment of FRINX Machine using Helm. Ensure you refer to the individual subchart documentation on Artifact Hub for specific configuration options and further customization.


## Step 5: Install FRINX Machine

Install the FRINX Machine using Helm:

```bash
helm dependency update
helm install -n frinx frinx-machine . -f values.yaml
```

Verify the installation by checking the pods in the frinx namespace:

```bash
kubectl get pods -n frinx 
```

You should see output similar to:

```bash
NAME                                                       READY   STATUS    RESTARTS   AGE
arango-frinx-machine-operators-operator-6dfdff75bd-h6mxb   1/1     Running   0          21m
arango-frinx-machine-operators-operator-6dfdff75bd-xh9x6   1/1     Running   0          21m
arangodb-sngl-yxxouifa-e0f232                              1/1     Running   0          11m
conductor-server-6757754659-tss78                          2/2     Running   0          19m
device-induction-56fdd555b8-j646n                          1/1     Running   0          19m
frinx-frontend-7c596b6bfc-qgthp                            2/2     Running   0          19m
frinx-machine-operators-cloudnative-pg-d9566444c-fgp5w     1/1     Running   0          21m
grafana-64986657b8-zzc5x                                   1/1     Running   0          19m
influxdb-0                                                 1/1     Running   0          19m
inventory-57994dcd85-9v2f9                                 1/1     Running   0          19m
kafka-controller-0                                         1/1     Running   0          19m
krakend-85bb6cd88b-6ldg7                                   2/2     Running   0          19m
loki-0                                                     1/1     Running   0          19m
performance-monitor-f6885b4dc-4wrfp                        1/1     Running   0          19m
postgresql-1                                               1/1     Running   0          11m
postgresql-2                                               1/1     Running   0          11m
promtail-zfmkn                                             1/1     Running   0          19m
resource-manager-d98d6866b-w5d6x                           1/1     Running   0          19m
swagger-ui-5b9fc85b99-8tzdd                                1/1     Running   0          19m
telegraf-ds-drsh7                                          1/1     Running   0          19m
timescale-db-0                                             1/1     Running   0          19m
topology-discovery-6d8c975876-gqg79                        2/2     Running   0          19m
uc-zone-lb-9cd56dd7-x82tz                                  1/1     Running   0          19m
uniconfig-controller-75d945f9c5-lggdb                      1/1     Running   0          12m
uniconfig-postgresql-1                                     1/1     Running   0          12m
uniconfig-postgresql-2                                     1/1     Running   0          12m
```

## Step 6: Access the UI

Visit Frinx Machine page in your browser on `https://krakend.127.0.0.1.nip.io/frinxui`
