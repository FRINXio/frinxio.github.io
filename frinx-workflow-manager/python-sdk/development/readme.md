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
- `Mirrord`: Make sure that you have installed Mirrord. Follow the [mirrord installation quide ](https://github.com/metalbear-co/mirrord) if necessary

## Step 1: Start Frinx Machine

Install Frinx Machine [!ref icon="briefcase"](/frinx-machine/installation/customization/frinx-machine-customization/readme.md)

Check out gitops-boilerplate repository to run Frinx Machine locally [!ref target="blank" icon="icon-mark-github" text="frinx-workers-boilerplate"](https://github.com/FRINXio/gitops-boilerplate)

## Step 2: Clone worker-example repository

Clone repository and follow instructions in README.md

[!ref icon="icon-mark-github" target="blank" text="frinx-workers-boilerplate"](https://github.com/FRINXio/frinx-workers-boilerplate)

### Step 3: Local development with Mirrord

Mirrord is great tool, which simplifies development of workflows on local/remove kubernetes cluster.
Is only needed to install it to the IDE (CLI), set connection to the correct kubeconfig/namespace and run python worker in targetless pod. 
No more need to workaround connection via ingress.

## Step 3: Deploy to cluster

[!ref](/frinx-machine/installation/custom-worker-deployment/readme.md)
