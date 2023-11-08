# RPC copy-many-to-one

This RPC is used to perform operations with a configuration from multiple source paths to a single target path.

RPC input contains the following:

* **type of operation**: `merge` or `replace`
* **type of source datastore**: `CONFIGURATION` or `OPERATIONAL`
* **type of target datastore**: `CONFIGURATION` or `OPERATIONAL`
* **list of source paths** in RFC-8040 URI format
* **target path** in RFC-8040 URI format (target path denotes parent
    entities under which the configuration is copied)

Target datastore is an optional input field. By default, it is the same as source datastore. All other input fields are mandatory.

RPC output describes the result of the operation. If one path fails, the entire operation fails and the datastore is not modified (i.e., all modifications are performed in a single atomic transaction).

![RPC copy-many-to-one](copy-many-to-one.svg)

## RPC examples

### Successful example

This example demonstrates the execution of the copy-many-to-one RPC with three (3) source paths.

Data described by these source paths (`snmp`, `access`, and `ntp` containers under three different nodes) are copied under the root `system:system` container (`dev04` node).

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/subtree-manager:copy-many-to-one' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "source-datastore": "CONFIGURATION",
        "source-paths": [
            "/network-topology:network-topology/topology=uniconfig/node=dev01/configuration/system:system/snmp",
            "/network-topology:network-topology/topology=uniconfig/node=dev02/configuration/system:system/access",
            "/network-topology:network-topology/topology=uniconfig/node=dev03/configuration/system:system/ntp"
        ],
        "target-path": [
            "/network-topology:network-topology/topology=uniconfig/node=dev04/configuration/system:system"
        ],
        "operation": "replace"
    }
}'
```

```RPC Response, Status: 200
```

### Failed example

This example shows a failed copy-many-to-one RPC operation. One of the source paths points to a non-existing schema node (`invalid:invalid`).

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/subtree-manager:copy-many-to-one' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "source-datastore": "CONFIGURATION",
        "source-paths": [
            "/network-topology:network-topology/topology=uniconfig/node=dev01/configuration/invalid:invalid",
            "/network-topology:network-topology/topology=uniconfig/node=dev01/configuration/system:system/users"
        ],
        "target-path": [
            "/network-topology:network-topology/topology=uniconfig/node=dev02/configuration/system:system"
        ],
        "operation": "merge"
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-type": "protocol",
        "error-tag": "unknown-element",
        "error-message": "Cannot parse input path 'network-topology:network-topology/topology=uniconfig/node=dev01/configuration/invalid:invalid' - Failed to lookup for module with name 'invalid'."
      }
    ]
  }
}
```