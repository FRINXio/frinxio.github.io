# RPC copy-one-to-many

This RPC is used to perform operations with a configuration from a single source path to multiple target paths.

RPC input contains the following:

* **type of operation**: `merge` or `replace`
* **type of source datastore**: `CONFIGURATION` or `OPERATIONAL`
* **type of target datastore**: `CONFIGURATION` or `OPERATIONAL`
* **source path** in RFC-8040 URI format
* **list of target paths** in RFC-8040 URI format (target paths denote
    parent entities under the which configuration is copied)

Target datastore is an optional input field. By default, it is the same as source datastore. All other input fields are mandatory.

RPC output describes the result of the operation. If one path fails, the entire RPC fails and the datastore is not modified (i.e., all modifications are performed in a single atomic transaction).

![RPC copy-one-to-many](copy-one-to-many.svg)

## RPC examples

### Successful example

This example demonstrates merging the ethernet interface configuration from a single source to the following interfaces:

* `eth-0/2` (node `dev02`)
* `eth-0/3` (node `dev02`)
* `eth-0/100` (node `dev03`)
* `eth-0/200` (node `dev03`)

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/subtree-manager:copy-one-to-many' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "source-datastore": "CONFIGURATION",
        "target-datastore": "OPERATIONAL",
        "source-path": "/network-topology:network-topology/topology=uniconfig/node=dev01/configuration/interfaces:interfaces/eth=eth-0%2F2/config",
        "target-paths": [
            "/network-topology:network-topology/topology=uniconfig/node=dev02/configuration/interfaces:interfaces/eth=eth-0%2F2",
            "/network-topology:network-topology/topology=uniconfig/node=dev02/configuration/interfaces:interfaces/eth=eth-0%2F3",
            "/network-topology:network-topology/topology=uniconfig/node=dev03/configuration/interfaces:interfaces/eth=eth-0%2F100",
            "/network-topology:network-topology/topology=uniconfig/node=dev03/configuration/interfaces:interfaces/eth=eth-0%2F200"
        ],
        "operation": "merge"
    }
}'
```

```RPC Response, Status: 200
```

### Failed example

This example shows a failed copy-one-to-many RPC operation.

Both target paths are invalid, since the `ext` list schema nodes does not contain the `interfaces:interfaces` child container.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/subtree-manager:copy-one-to-many' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "source-datastore": "OPERATIONAL",
        "source-path": "/network-topology:network-topology/topology=uniconfig/node=vnf01/configuration/interfaces:interfaces",
        "target-paths": [
            "/network-topology:network-topology/topology=uniconfig/node=vnf02/configuration/interfaces:interfaces/ext=ext-0%2F2",
            "/network-topology:network-topology/topology=uniconfig/node=vnf02/configuration/interfaces:interfaces/ext=ext-0%2F3"
        ],
        "operation": "replace"
    }
}'
```

```json RPC Response, Status: 500
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "operation-failed",
        "error-message": "Failed to find schema node with identifier '(example-interfaces?revision=2019-12-01)interfaces' under: (example-interfaces?revision=2019-12-01)ext"
      }
    ]
  }
}
```