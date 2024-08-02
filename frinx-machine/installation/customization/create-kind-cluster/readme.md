---
expanded: false
order: 9999
label: Cluster installation
---

# Setting Up a Kind Cluster with Cilium and NGINX Ingress Controller

This guide will walk you through the process of deploying a Kubernetes (K8s) cluster using Kind (Kubernetes IN Docker), setting up the Cilium CNI (Container Network Interface), and deploying the NGINX Ingress Controller.

## Prerequisites

- `Kind`: Make sure that Kind is installed on your local machine. Follow the Kind installation guide if necessary.
- `Helm`: Make sure that Helm is installed. Follow the Helm installation guide if necessary.
- `Cilium`: Make sure that Cilium system requirements are fullfiled. Follow the [Cilium installation](https://docs.cilium.io/en/stable/operations/system_requirements/#admin-system-reqs) guide if necessary.


## Deploy Kind cluster

Create a Kind configuration file named kind-config.yaml with the following content:

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: 127.0.0.1
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    listenAddress: 127.0.0.1
    protocol: TCP
- role: worker
- role: worker
- role: worker
networking:
  disableDefaultCNI: true
  kubeProxyMode: none
```

This configuration sets up a Kind cluster with one control-plane node and three worker nodes. It also maps ports 80 and 443 from the host to the control-plane node, making the cluster ready for ingress traffic.

Deploy the cluster using Kind:

```bash
kind create cluster --config kind-config.yaml
```

Verify the cluster is running:

```bash
kubectl cluster-info
```

You should see output indicating that the Kubernetes control plane and CoreDNS are running.

```bash
Kubernetes control plane is running at https://127.0.0.1:43899
CoreDNS is running at https://127.0.0.1:43899/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Deploy Cilium

Create a Cilium configuration file named cilium-helm-values.yaml with the following content:

```yaml
# cilium-helm-values.yaml
kubeProxyReplacement: strict
k8sServiceHost: kind-control-plane 
k8sServicePort: 6443 
hostServices:
  enabled: false
externalIPs:
  enabled: true
nodePort:
  enabled: true
hostPort:
  enabled: true
image:
  pullPolicy: IfNotPresent
ipam:
  mode: kubernetes
hubble:
  enabled: true
  relay:
    enabled: true
  ui:
    enabled: true
    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: nginx
      hosts:
        - hubble-ui.127.0.0.1.nip.io
```

This configuration enables Cilium with kube-proxy replacement and various service options, including Hubble for network observability.

Install Cilium using Helm:

```bash
# Don't forget to use correct cluster context
# Add the Cilium Helm repository
helm repo add cilium https://helm.cilium.io/

# Deploy Cilium with the specified values
helm upgrade --install --namespace kube-system --repo https://helm.cilium.io cilium cilium --values cilium-helm-values.yaml
```

Check the status of the Cilium pods to ensure they are running:

```bash
kubectl get pods -n kube-system 
```

You should see the Cilium and Hubble components running without issues.

```bash
NAME                                         READY   STATUS    RESTARTS   AGE
cilium-2ldns                                 1/1     Running   0          30h
cilium-b877s                                 1/1     Running   0          30h
cilium-mhs9c                                 1/1     Running   0          30h
cilium-operator-7fc58985c4-m2kbv             1/1     Running   0          30h
cilium-operator-7fc58985c4-mq5pc             1/1     Running   0          30h
cilium-sqrdv                                 1/1     Running   0          30h
coredns-7db6d8ff4d-ltcjq                     1/1     Running   0          30h
coredns-7db6d8ff4d-s6c6f                     1/1     Running   0          30h
etcd-kind-control-plane                      1/1     Running   0          30h
hubble-relay-6d88849768-2wcjn                1/1     Running   0          30h
hubble-ui-59bb4cb67b-g79pz                   2/2     Running   0          30h
kube-apiserver-kind-control-plane            1/1     Running   0          30h
kube-controller-manager-kind-control-plane   1/1     Running   0          30h
kube-scheduler-kind-control-plane            1/1     Running   0          30h
```

## Deploy NGINX Ingress Controller

Deploy the NGINX Ingress Controller using the following command:

```bash
# Replace with the latest version from the official repository
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/kind/deploy.yaml
```

Verify the NGINX Ingress Controller is running:

```bash
kubectl get pods -n ingress-nginx
```

You should see the NGINX Ingress Controller pod running.

```bash
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-2p9jm       0/1     Completed   0          30h
ingress-nginx-admission-patch-tmnrp        0/1     Completed   0          30h
ingress-nginx-controller-d45d995d4-lqgr6   1/1     Running     0          30h
```
