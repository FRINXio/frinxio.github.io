---
label: UniConfig gNMI
order: 8000
---

# UniConfig gNMI

## Introduction

The gNMI (gRPC Network Management Interface) southbound plugin enables Frinx 
UniConfig to communicate with a device through gNMI. It provides the mechanism to
install, manipulate, and delete the configuration of network devices, and also to view operational data.

The gNMI southbound plugin follows a fully model-driven approach, similar 
to CLI or NETCONF southbound plugins. However, the difference is that it uses protocol buffer (proto3) 
for service structure modeling within it's RPCs instead of YANG.

## gNMI southbound plugin

The gNMI southbound plugin is capable of connecting to remote gNMI devices and exposing their 
datastores (CONFIG, STATE) as MD-SAL mount points. These mount points allow applications and remote users 
(over RESTCONF) to interact with the mounted devices. By mounting of gNMI device a new session is gNMI session is created.
The gNMI session is responsible for establishing a connection via gRPC ManagedChannel. After the connection is established, 
gNMI southbound plugin read the device capabilities via Capabilities RPC. From the capabilities, a new schema context is built 
(once the schema context is built, it is cached for next runs) and all necessary services are created (DOMDataBroker, 
DOMRpcService, DOMNotification/Subscription service). After all of this is done, gNMI southbound plugin will mark 
the connection status of the particular device as READY, so UniConfig layer can handle it.

## Important features

When gNMI device is installed, it is possible to do:

- read (gNMI GET), create/update/delete (gNMI SET) the configuration of device:
  - via UniConfig topology
  - via gNMI souhtbound topology (using `yang-ext:mount`)
  - via yang-patch operations over UniConfig/gNMI souhtbound topology (using `yang-ext:mount`)
- manipulate with a device via gNOI RPCs
- use gNMI SET update operation for specific paths
- use gNMI SET replace operation for specific paths
- order config changes (replace/delete operation) according to dependency paths
- remove module-name from list keys for specific paths
- read from ALL datastore for specific paths
- subscribe to telemetry streaming for specific paths (gNMI Subscribe)
- store failed installations
- enable logging-broker for gNMI messages (SET/GET)
- use DryRunCommit feature

### gNOI 

The gNOI (gRPC Network Operations Interface) defines gRPC-based microservices 
for executing operational commands on network devices. By default gNMI southbound plugin is capable of 
executing Certification, File, System and OS RPCs according to protobuf files <https://github.com/openconfig/gnoi>. 
Apart from the common ones, it is possible to invoke also SONiC type gNOI RPCs by specifying 
`"gnmi-topology:device-type" : "sonic"` in install request inside of `connection-parameters` container. 
It's necessary to have `gnoi-sonic@2023-03-17.yang` schema model in cache directory for this specific node, 
to be able to invoke it via RESTCONF. 

Example of gNOI RPC invocation
------------------------------

```bash Invocation of SONiC specific copy-config RPC
curl --location --request POST 'http://localhost:8181/rests/operations/network-topology:network-topology/topology=gnmi-topology/node=sonic/yang-ext:mount/gnoi-sonic:copy-config' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
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

Update paths feature tells gNMI southbound plugin to process the intended config 
changes (calculated from UniConfig Diff) as gNMI SET message - Update operation. 
The gNMI southbound plugin basically checks for config changes paths if they are relative to those specified in the install request.
Each path is whole in regexp format.

### Replace paths

Replace paths feature is different from Update paths one. The main difference is that UniConfig topology
will at first calculate changes by it's own diff implementation and afterwards 
if some path is specified in replace-paths list in the install request, UniConfig topology will adjust the calculated 
diffs and group them according to replace paths.

If we have config change at `a/b/c/d` and `a/b/c/e` and the replace path is specified as `a/b/c`, 
the diff calucalation adjustment will mark those 2 changes as one on the path `a/b/c` and in this structure 
it will be passed to gNMI southbound topology where it will be processed as gNMI SET message - Replace operation.

!!!
If we delete some config on path using DELETE REST operation, that is relative to some of the replace paths, 
it will be processed as gNMI SET message - Delete operation. It will be basically skipped from the replace paths feature. 
!!!

This feature can be used also only in gNMI southbound plugin, 
for example if we send config change via yang-patch operation over gNMI topology (so it goes outside of UniConfig Diff). 
In this scenario it will just check if the config change path is relative to one of the replace-paths

The paths are in common RESTful URL format, but the list entry can be compiled 
as regexp pattern if it is specified with `$` sign after `=` sign. 
Then the list entry key be specified like this : `interface=$.*[Ee]thernet?[0-9]+`.

### Dependency paths

Dependency paths feature helps network device to order committed config changes, if the device can't handle it by it's own. 
For example, firstly delete interface from the network-instance and secondly delete it from the interfaces container.
This is done in gNMI southbound topology and it orders changes marked as for Replace/Delete operation. 
It will sort config changes from their paths according to the rules specified in the dependency paths

The rule format of dependency paths is: 
- `before` - path without keys that will be ordered before the path specified in `after`.
- `after` - path withou keys that will be ordered after the path specified in `before`.

### gNMI logging-broker

The gNMI logging-broker is explained here <https://docs.frinx.io/frinx-uniconfig/user-guide/operational-procedures/logging/#supported-logging-settings>.

### Telemetry streaming

UniConfig with gNMI southbound plugin supports gNMI telemetry streaming - receiving state of data on a target. 
Telemetry streaming in UniConfig is divided into three parts:
- creation of the subscription
- receiving notifications
- saving notifications to DB / publishing it to Kafka topic

To be able to create subscription it is necessary to have these pre-requirements in application.properties:
- `notifications.enabled=true`
- `notifications.kafka.gnmi-notifications-enabled=true`
- `notifications.kafka.gnmi-notifications-topic-name=gnmi-notifications`
- `notifications.kafka.embedded-kafka.enabled=true` (or have own Kafka -> then kafka-servers needs to be adjusted)

In install request we need to specify parameters needed for the creation of the subscription for specific device.
The mandatory fields are `stream-name`, which is a marker for the particular subscription and `paths` which 
is list of paths to which gNMI souhtbound plugin will be subscribed (at least 1 path needs to be specified).
There are also non-mandatory fields that are responsible for the time range of the subscription. 
Both `start-time` and `stop-time` are in RFC339 format <https://datatracker.ietf.org/doc/html/rfc3339>.

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
        ]
      }
    ]
  }
}
```

Creation of subscription is done independently from the installation of the node. 
The result of the install-node RPC is related only to the node installation. 
The install-node rpc will just invoke telemetry stream subscription process. 
Creation of the subscription is done in gNMI southbound plugin using Subscribe RPC from gNMI service defined in proto3 file.
The gNMI southbound plugin is capable of subscription to wildcarded paths as well as to multiple paths at once. 
It only support STREAM subscription mode and ON_CHANGE stream mode. If time range is specified it will be part of the Subscribe request and 
if the device doesn't support time range on it's server side, gNMI southbound plugin can handle it on it's own 
(if timestamp of the received message is outside of the time range it will not be saved to DB or published to Kafka topic).
If the specified path is marked as disabled in YANG schemas for ON_CHANGE mode, gNMI southbound plugin will check it before the creation 
of the subscription and it will fail with logged message that some path is marked as disabled for the subscription.
The session will be opened until it will be closed either from UniConfig side (releasing of the subscription) 
of device side (error). The result of telemetry streaming can be seen in specified kafka topic or in notification DB table.


### DryRun commit

The Dry-run manager is explained here <https://docs.frinx.io/frinx-uniconfig/user-guide/uniconfig-operations/dryrun-manager/#dry-run-manager>.
It also support gNMI southbound topology. It will be accessible only if the parameter `gnmi-topology:dry-run-journal-size` 
will be set to higher number than 0. 

### gNMI testtool

The gNMI testtool is a GO language server-side simulator capable of configuring via gNMI over OpenConfig YANG schemas 
(currently supported only openconfig-interfaces, openconfig-system, openconfig-openflow), 
controlling operations over gNOI, telemetry streaming. 
It is used for scale-testing purpose to simulate interaction of thousands of devices with UniConfig.


Examples
--------

[!ref](sonic.md)
[!ref](nokia.md)
