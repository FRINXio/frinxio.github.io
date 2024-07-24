---
label: Demo Use Cases
icon: briefcase
order: 100
---

# Demo Use Cases

The following use cases demonstrate the basic usage of FRINX Machine. 
These examples will help you explore the platform's capabilities, including executing prepared workflows, creating custom workflows via the workflow builder, and manually managing your device inventory.

#### 1. Executing Prepared Workflows
- Experience the power of automation by running pre-configured workflows.
- Learn how to efficiently manage network tasks using automated processes.

#### 2. Creating Custom Workflows
- Utilize the workflow builder to create and customize your workflows.
- Tailor processes to your specific needs, enhancing operational efficiency.

#### 3. Managing Device Inventory
- Add devices to your inventory manually, providing flexibility to experiment with different configurations.
- This feature is particularly useful for testing and deploying your own networking devices.

#### 4. Controlling Devices
- Manage devices in your inventory through the user interface (UI).
- Execute operations using prepared workflows, ensuring streamlined control and management.

#### 5. Configuring Network Services
- Set up L2VPNs between virtual devices, enhancing network connectivity and performance.
- Create loopbacks on device interfaces either manually or through automated workflows, demonstrating the versatility of FRINX Machine.

These use cases serve as a starting point for exploring the extensive capabilities of FRINX Machine. 
They illustrate how the platform can be utilized for a variety of networking tasks, from basic device management to complex network configurations.


## Install Demo Use Case workflows

Use [demo-worklows helm chart](https://artifacthub.io/packages/helm/frinx-helm-charts/demo-workflows) to run sample-topology and frinx-demo-workflows conducor worker.

### Install Frinx Machine
Make sure your Frinx Machine is running.
[!ref](../installation/basic/readme.md)

### Install Demo workflows and Sample Topology


```bash
helm repo add frinx https://FRINXio.github.io/helm-charts
helm repo update
```

```bash
helm install -n frinx frinx-demo-workflows frinx/demo-workflows
```

```bash
NAME                                    READY   STATUS    RESTARTS   AGE
sample-topology-5d677db769-zxbzs        1/1     Running   0          8h
frinx-demo-workflows-6f8c7666b8-nzmd9   1/1     Running   0          8h
```
