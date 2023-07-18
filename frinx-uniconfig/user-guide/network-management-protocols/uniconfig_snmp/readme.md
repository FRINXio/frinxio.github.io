---
label: UniConfig SNMP
order: 8000
---

# UniConfig SNMP

## Introduction

The SNMP (Simple Network Management Protocol) southbound plugin enables Frinx 
UniConfig to communicate with an SNMP agent, which is a software module installed 
on network devices. It collects information about the status, performance, 
and configuration of these devices.

The SNMP southbound plugin follows a fully model-driven approach, similar 
to CLI or NETCONF southbound plugins. However, the difference lies in the 
fact that it uses MIB (Management Information Base) for data modeling instead 
of YANG.

## Architecture

This section provides an architectural overview of the plugin, focusing
on the main components.

### SNMP topology

The SNMP topology is a dedicated topology instance where users and
applications can:

- install an SNMP agent,
- uninstall an agent,
- read device configuration settings or performance metrics

### SNMP mountpoint

The plugin relies on MD-SAL and its concept of mountpoints to expose information 
about a device. By exposing a mountpoint in MD-SAL, it enables the SNMP topology
to access device information in a structured form.

### Local SNMP MIB repositories

It is necessary to provide /mibs directory that has to contain:
- repository - it is directory that contains mib files. It is possible to 
    use any name.
- mib.metadata file - through this file, we inform UniConfig that we have 
    added, removed, or modified some MIB file in the repository. Just insert 
    the repository name and any arbitrary string and UniConfig will update
    the relevant context for particular repository.

Example of mib.metadata file
----------------------------

```bash
repository1=rev1
repository1=rev2
```

Example of requests
-------------------

UniConfig currently supports read operation, with plans to add write operation in the future.

## GET request

```bash
curl --location 'http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-snmp/node=agent1/yang-ext:mount/SNMPv2-SMI:iso'
```
