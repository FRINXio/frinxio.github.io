---
label: UniConfig SNMP
order: 8000
---

# UniConfig SNMP

## Introduction

The SNMP (Simple Network Management Protocol) southbound plugin allows UniConfig to communicate with an SNMP agent, which is a software module installed on network devices. It collects information about the status, performance, and configuration of these devices.

The SNMP southbound plugin follows a fully model-driven approach, similar to CLI or NETCONF southbound plugins. However, the difference lies in the fact that, instead  of YANG, it uses MIB (Management Information Base) for data modeling.

## Architecture

This section provides an architectural overview of the plugin.

### SNMP topology

The SNMP topology is a dedicated topology instance where users and applications can:

- Install an SNMP agent
- Uninstall an agent
- Read device configuration settings or performance metrics

### SNMP mountpoint

The plugin relies on MD-SAL and its concept of mountpoints to expose information about a device. By exposing a mountpoint in MD-SAL, it allows the SNMP topology to access device information in a structured form.

### Local SNMP MIB repositories

It is necessary to provide the **/mibs** directory that contains the following:
- **repository** - a directory that contains mib files. You can use any name.
- **mib.metadata** file - With this file we inform UniConfig that we have added, removed, or modified an MIB file in the repository. Simply insert the repository name and any arbitrary string, and UniConfig will update the relevant context for the particular repository.

**Example - mib.metadata file**

```bash
repository1=rev1
repository1=rev2
```

## Example request

UniConfig currently supports the read operation, with plans to add support for the write operation in the future.

### GET request

```bash Reading snmpV2 container
curl --location 'http://localhost:8181/rests/data/network-topology:network-topology/topology=topology-snmp/node=agent1/yang-ext:mount/SNMPv2-SMI:iso/org/dod/internet/snmpV2'
```

```json Sample response
{
  "snmpV2": {
    "snmpModules": {
      "SNMPv2-MIB:snmpMIB": {
        "snmpMIBObjects": {
          "snmpSet": {
            "snmpSetSerialNo": "0"
          }
        }
      }
    }
  }
}
```