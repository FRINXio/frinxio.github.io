---
expanded: false
order: 9999
label: Helm Installation Guide
---

# Helm Chart Installation Guide

This guide provides the step-by-step instructions for installing FRINX Machine on a Kubernetes cluster using Helm charts.

## Prerequisites

- `Minikube`: Make sure that Minikube is installed on your local machine. Follow the Minikube installation guide if necessary.
- `Helm`: Make sure that Helm is installed. Follow the Helm installation guide if necessary.


## Step 1: Start Minikube

```bash
# Minimal recommended setup
minikube start --cpus=12 --memory=24G --kubernetes-version=v1.27 --addons=ingress

# Alternatively, use maximum available CPUs
minikube start --cpus=max --memory=24G --kubernetes-version=v1.27 --addons=ingress
```

## Step 2: Add the FRINX Helm Repository

Add the FRINX Helm repository and update the repository list:

```bash
helm repo add frinx https://FRINXio.github.io/helm-charts
helm repo update
```

## Step 3: Install Operators and CRDs

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

## Step 4: Create Docker Registry Secret

Please complete this step before continue [!ref icon="briefcase" text="Create docker regitry secret"](../docker-registry-secret/readme.md)


## Step 5: Install FRINX Machine

Install the FRINX Machine using Helm:

```bash
helm install -n frinx frinx-machine frinx/frinx-machine --timeout 10m
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

To access ui, use krakend-nginx ingress hostname. 

```bash
kubectl get ingress -n frinx
```

You should see output similar to:

```bash
NAME            CLASS   HOSTS                      ADDRESS        PORTS   AGE
krakend-nginx   nginx   krakend.127.0.0.1.nip.io   192.168.49.2   80      56m
```

In case of minikube is required to add the following entries to your /etc/hosts file:

```
# /etc/hosts
...
192.168.49.2 krakend.127.0.0.1.nip.io fm.127.0.0.1.nip.io
```

and visit Frinx Machine page in your browser on `https://krakend.127.0.0.1.nip.io/frinxui`
