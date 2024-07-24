---
icon: rocket
expanded: false
order: 9999
---

# FRINX Machine introduction

In today's rapidly evolving digital landscape, efficient network management and automation are crucial for maintaining robust and scalable IT infrastructures. 
Frinx Machine emerges as a powerful solution tailored to meet these demands, providing an integrated platform designed to simplify and enhance network automation.

## What is Frinx Machine?

Frinx Machine is an advanced network automation platform delivering a comprehensive suite of tools for managing and automating network infrastructures. 
It is designed to streamline network operations, reduce complexity, and drive efficiency through a unified, scalable, and flexible approach.

Frinx Machine enabling seamless management of network configurations across multi-vendor environments. 
It provides a consistent interface for deploying and managing configurations, reducing the complexities associated with heterogeneous network devices.

## FRINX Machine core components


### High-Level Architecture

The following diagram outlines the main functional components in the FRINX
Machine solution:

![FM Architecture](FRINX_Machine_Architecture.png)


### UniConfig

- Connects to the devices in the network
- Retrieves and stores configuration from devices
- Pushes configuration data to devices
- Builds diffs between actual and intended config to execute atomic
  configuration changes
- Retrieves operational data from devices
- Manages transactions across one or multiple devices
- Translates between CLI, vendor native, and industry-standard data
  models (i.e. OpenConfig)
- Reads and stores vendor native data models from mounted network
  devices (i.e YANG models)
- Ensures high availability, reducing network outages and downtime
- Executes commands on multiple devices simultaneously

### Workflow manager (Conductor)

- Atomic tasks are chained together into more complex workflows
- Defines, executes and monitors workflows (via REST or UI)
- Schedule workflow executions

We chose Netflix’s conductor workflow engine since it has been proven to
be a highly scalable open-source technology that integrates very well with
FRINX UniConfig. Further information about Conductor can be found at:

- **FRINXio conductor sources:** https://github.com/FRINXio/conductor
- **FRINXio conductor community sources:** https://github.com/FRINXio/conductor-community
- **Sources:** https://github.com/conductor-oss/conductor
- **Docs:** https://conductor-oss.github.io/conductor/index.html

### Device inventory

- Store all important devices information on one place
- Maintain devices in all deployment zones from one place
- Notify about changes in inventory via Kafka

### Topology discovery

- Acquires information from live network
  - Relying on UniConfig as its primary source of network information
- Uses that information to build topology view(s):
  - Across multiple layers of the network
  - e.g. LLDP data to build physical topology view or routing data to build L3 view
- Performs reconciliation across the layers in order to provide a unified topology view
- Provides an API to query topologies
- Provides Kafka notifications about changes in the topology
- Consumes and stores device metadata events from Kafka topic


### Performance monitor

- Collects performance metrics about devices in a time-series based database
  - Relies on UniConfig as a producer of the performance metrics of devices.
  - Relies on Device Inventory as a producer of information about devices, such as device name, vendor, model, and version.
- Unifies performance metrics and produces them to a Kafka broker.
- Provides an API to query performance metrics of devices.

## Monitoring:

### Loki + Grafana + Influxdb + Telegraf - core of Monitoring

- Loki: Efficient log aggregation and querying.
- Telegraf: Data collection and reporting agent.
- InfluxDB: High-performance time-series database.
- Grafana: Powerful visualization and dashboard creation.

### Key Functions:

- Collect logs and metrics from services to provide platform observability
