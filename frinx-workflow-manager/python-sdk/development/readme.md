---
order: 1000
label: Developent environment
---

# Development environment

This guide provides the step-by-step instructions for preparing develoment environment.

## Prerequisites

- `Cluster`: Make sure that you cluster is running.
- `Helm`: Make sure that Helm is installed. Follow the Helm installation guide if necessary.
- `Python`: Make sure that your environment have Python ^3.10 interpretter installed.
- `Poetry`: Make sure that you have installed Poetry. Follow the Poetry installation guide if necessary.

## Step 1: Start Frinx Machine

Install Frinx Machine with ingress enabled [!ref icon="briefcase"](/frinx-machine/installation/customization/frinx-machine-customization/readme.md)

Make sure you have enabled workflow-manager and krakend ingress, because it's required for local development.

```bash
frinx-machine:
  krakend:
    ingress:s
      enabled: true
      className: nginx
      annotations:
        # force-ssl-redirect must be disabled in case you are using a self-signed certificate
        # nginx.ingress.kubernetes.io/force-ssl-redirect: "true" 
        nginx.ingress.kubernetes.io/proxy-connect-timeout: "3600"
        nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
        nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
      hosts:
        - host: krakend.127.0.0.1.nip.io
          paths:
            - path: "/"
              pathType: ImplementationSpecific

  workflow-manager:
    ingress:
      enabled: true
      hosts:
        - host: workflow-manager.127.0.0.1.nip.io
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

In case, you using minikube, get minikube ip

```bash
minikube ip

192.168.49.2
```

add map that ip with ingres hosts to your /etc/hosts

```bash
#/etc/hosts
192.168.49.2 krakend.127.0.0.1.nip.io
192.168.49.2 workflow-manager.127.0.0.1.nip.io
```

To verify ingresses

```bash
$kubectl get ingress -n frinx 

NAME               CLASS   HOSTS                                                                          ADDRESS        PORTS   AGE
conductor-server   nginx   workflow-manager.127.0.0.1.nip.io                                              192.168.49.2   80      161m
krakend            nginx   krakend.127.0.0.1.nip.io                                                       192.168.49.2   80      161m
```

## Step 2: Clone worker-example repository

Clone repository and follow instructions in README.md

[!ref target="blank" icon="mark-github" text="frinx-workers-boilerplate"](https://github.com/FRINXio/frinx-workers-boilerplate)


## Step 3: Deploy to cluster

[!ref](/frinx-machine/installation/custom-worker-deployment/readme.md)
