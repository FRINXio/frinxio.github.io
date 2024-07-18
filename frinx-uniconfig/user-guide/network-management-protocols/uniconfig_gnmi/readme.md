---
label: UniConfig gNMI
order: 8000
---

# UniConfig gNMI

## Introduction

The gNMI (gRPC Network Management Interface) southbound plugin allows UniConfig to communicate with devices via gNMI. It provides a mechanism to install, manipulate and delete the configuration of a network device, and to view operational data.

The plugin follows a fully model-driven approach similar to the CLI and NETCONF southbound plugins. The main difference is that it uses the protocol buffer (proto3) instead of YANG for service structure modeling within its RPCs, like the gRPC protocol does.  Note that the YANG schemas are still necessary for config data modelling on the UniConfig side. 

## gNMI southbound plugin

The gNMI southbound plugin is capable of connecting to remote gNMI devices and exposing their datastores (CONFIG, STATE) as MD-SAL mount points. These mountpoints allow applications and remote users (over RESTCONF) to interact with the mounted devices.

A new gNMI session is created by mounting a gNMI device. The session is responsible for establishing a connection via gRPC ManagedChannel. Once a connection is established, the gNMI southbound plugin reads device capabilities using the Capabilities RPC. A new schema context is built from the capabilities (once the schema context is built, it is cached for the next runs) and all necessary services are created (DOMDataBroker, DOMRpcService, DOMNotification/Subscription service). 

When all this is done, the gNMI southbound plugin marks the connection status of the particular device as READY so that the UniConfig layer can handle it.

## Important features

When a gNMI device is installed, it is possible to:

- Read (gNMI GET) and create/update/delete (gNMI SET) the configuration of the device via the following:
  - UniConfig topology
  - gNMI souhtbound topology (using `yang-ext:mount`)
  - yang-patch operations over UniConfig/gNMI souhtbound topology (using `yang-ext:mount`)
- Manipulate a device via gNOI RPCs
- Use the gNMI SET update operation for specific paths
- Use the gNMI SET replace operation for specific paths
- Order config changes (replace/delete operations) according to dependency paths
- Remove module-name from list keys for specific paths
- Read from ALL datastore for specific paths
- Subscribe to telemetry streaming for specific paths (gNMI Subscribe)
- Store failed installations
- Enable the logging-broker for gNMI messages (SET/GET)
- Use the DryRunCommit feature

### gNOI 

The gNOI (gRPC Network Operations Interface) defines gRPC-based microservices for executing operational commands on network devices. By default, the gNMI southbound plugin is capable of executing Certification, File, System and OS RPCs according to protobuf files (<https://github.com/openconfig/gnoi>). 

Apart from common ones, it is also possible to invoke SONiC-type gNOI RPCs by specifying `"gnmi-topology:device-type" : "sonic"` inside the `connection-parameters` container in the install request. It is necessary to have the **gnoi-sonic@2023-03-17.yang** schema model in the **cache** directory for this specific node to invoke it via RESTCONF. 

**Example** - gNOI RPC invocation

```bash Invocation of SONiC specific copy-config RPC
curl --location --request POST 'http://localhost:8181/rests/operations/network-topology:network-topology/topology=gnmi-topology/node=sonic/yang-ext:mount/gnoi-sonic:copy-config' \
--header 'Content-Type: application/json' \
--data-raw '{
        "input": {
            "input": {
                "source": "running-configuration",
                "destination": "startup-configuration"
            }
        }
    }'
```

```json Response
{
  "output": {
    "output": {
        "status_detail": "SUCCESS."
    }
  }
}
```

### Update paths

The update paths feature tells the gNMI southbound plugin to process the intended config changes (calculated from UniConfig diff) as a gNMI SET message - Update operation.

The plugin essentially checks if config change paths are relative to any of the update paths listed in the installation request. (For example, if you have a config change at `a/b/c/d` that is relative to the update-path `a/b/c`).

The main purpose of this feature is to send a gNMI SET message with the correct operation mode that the device supports for the specific subtree. Paths are in regexp format.

### Replace paths

The replace paths feature is a little different from update paths. With replace paths, the UniConfig topology first calculates changes using own diff implementation and, if a path is specified in the `replace-paths` list in the install request, adjusts the calculated diffs and groups them accordingly.

If there is a config change at `a/b/c/d` and `a/b/c/e`, and the replace path is specified as `a/b/c`,  the diff calculation adjustment marks the two changes as one on the path `a/b/c`. In this structure it is passed to the gNMI southbound topology, where it is processed as a gNMI SET message - Replace operation.

!!!
If a config is deleted using the DELETE REST operation on a path that is relative to some of the replace paths, 
it is processed as a gNMI SET message - Delete operation. Essentially, it is skipped by the replace paths feature. 
!!!

This feature can also be used only in the gNMI southbound plugin, for example, if we send a config change via yang-patch operation over the gNMI topology (so it goes outside of the UniConfig diff). In this scenario, it only checks if the config change path is relative to one of the replace-paths.

The main purpose of the replace paths feature is to send gNMI SET messages with the correct operation mode that the device supports for a specific subtree.

The paths are in common RESTful URL format, but a list entry can be compiled as a regexp pattern if specified with `$` sign after the `=` sign.  The list entry key is then specified like this: `interface=$.*[Ee]thernet?[0-9]+`.

### Dependency paths

The dependency paths feature helps network devices to order committed config changes if the device cannot handle this. For example, deleting an interface from the network-instance first and then deleting it from the interfaces container. This is done in the gNMI southbound topology, and it orders changes marked for the Replace/Delete operation. Config changes are sorted from their paths according to rules specified in the dependency paths.

The rule format for dependency paths:
- `before` - path without keys that is ordered before the path specified in `after`.
- `after` - path without keys that is ordered after the path specified in `before`.

### gNMI logging-broker

For more information about the gNMI logging-broker, see [Supported logging settings](https://docs.frinx.io/frinx-uniconfig/user-guide/operational-procedures/logging/#supported-logging-settings).

### Telemetry streaming

UniConfig with gNMI southbound plugin supports gNMI telemetry streaming - receiving the state of data on a target. 

Telemetry streaming in UniConfig is divided into three parts:
- Creating a subscription
- Receiving notifications
- Saving notifications to a database or publishing them to Kafka topic

To create subscriptions, the following are required in **application.properties**:
- `notifications.enabled=true`
- `notifications.kafka.gnmi-notifications-enabled=true`
- `notifications.kafka.gnmi-notifications-topic-name=gnmi-notifications`
- `notifications.kafka.embedded-kafka.enabled=true` (or its own Kafka, in which case kafka-servers needs to be adjusted)

The install request must specify the parameters necessary to create a subscription for a specific device.
Mandatory fields are `stream-name` (a marker for the particular subscription) and `paths` 
(a list of paths to which the gNMI souhtbound plugin will be subscribed, with at least one path).
Non-mandatory fields specify a time range of the subscription or subscription mode. Both `start-time` and `stop-time` are in [RFC339 format](https://datatracker.ietf.org/doc/html/rfc3339).
Subscription mode can be SAMPLE, ON_CHANGE or TARGET_DEFINED

```json stream inside of install-node RPC request
{
  ...
  "gnmi" :{
    ...
    "stream": [
      {
        "stream-name": "GNMI_if",
        "paths": [
          "openconfig-interfaces:interfaces/interface=$.*/config"
        ],
        "start-time": "2023-10-31T10:47:00+01:00",
        "stop-time": "2023-10-31T10:49:00+01:00"
      },
      {
        "stream-name": "GNMI_lldp",
        "paths": [
          "openconfig-lldp:lldp"
        ],
        "mode": "SAMPLE"
      }
    ]
  }
}
```

Subscriptions are created independently from node installation. The result of the install-node RPC relates only to node installation, and that RPC will only invoke the telemetry stream subscription process.

Subscriptions are created in the gNMI southbound plugin using the Subscribe RPC from the gNMI service defined in the proto3 file. The gNMI southbound plugin can subscribe to wildcarded paths as well as multiple paths at once. It supports only the `STREAM` subscription mode with `SAMPLE`, `ON_CHANGE` and `TARGET_DEFINED` stream mode.

If a time range is specified, it is part of the Subscribe request. If the device does not support time ranges on its server side, the gNMI southbound plugin can handle them. Note that if the timestamp of the received message is outside of the specified time range, the message is not saved to the database or published to the Kafka topic.

If the specified path is marked as disabled for the `ON_CHANGE` mode in YANG schemas, the gNMI southbound plugin fails with a log message stating that the path is marked as disabled for the subscription.

The session is open until it is closed either on the UniConfig side (releasing the subscription) or the device side (error). The result of telemetry streaming can be seen in the specified Kafka topic or in the notification database table.


### DryRun commit

For more information about dryrun commit, see [Dry-run manager](https://docs.frinx.io/frinx-uniconfig/user-guide/uniconfig-operations/dryrun-manager/#dry-run-manager).

It also supports the gNMI southbound topology, which is accessible only if the parameter `gnmi-topology:dry-run-journal-size` is set to be higher than 0. 

### gNMI testtool

The gNMI testtool is a GO language server-side simulator capable of configuring OpenConfig YANG schemas via gNMI. It currently supports openconfig-interfaces, openconfig-system, openconfig-openflow, controlling operations over gNOI and telemetry streaming. 

The tool is used for scale-testing purposes to simulate the interactions of thousands of devices with UniConfig.

Examples:
- [!ref](sonic.md)  
- [!ref](nokia.md)
- [!ref](iosxr7.md)