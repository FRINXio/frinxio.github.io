# RPC copy-one-to-one

This RPC is used to perform operations with a configuration from a single source path to a single target path.

RPC input contains the following:

* **type of operation**: `merge` or `replace`
* **type of source datastore**: `CONFIGURATION` or `OPERATIONAL`
* **type of target datastore**: `CONFIGURATION` or `OPERATIONAL`
* **source path** in RFC-8040 URI format
* **target path** in RFC-8040 URI format (target path denotes parent entities under which the configuration is copied)

Target datastore is an optional input field. By default, it is the same as the source datastore. All other fields are mandatory.

RPC output describes the result of the operation. If the RPC fails, no changes are made to the target datastore.

![RPC copy-one-to-one](copy-one-to-one.svg)

## RPC examples

### Successful example

This example demonstrates how to copy the entire `org:orgs` container from `dev01` to the `dev02` node under the `uniconfig` topology using the `replace` operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/subtree-manager:copy-one-to-one' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "source-datastore": "CONFIGURATION",
        "target-datastore": "OPERATIONAL",
        "source-path": "/network-topology:network-topology/topology=uniconfig/node=dev01/configuration/org:orgs",
        "target-path": "/network-topology:network-topology/topology=uniconfig/node=dev02/configuration",
        "operation": "replace"
    }
}'
```

```RPC Response, Status: 204
```

### Failed example

This example shows a failed copy-one-to-one operation.

RPC input contains the source datastore (same as the target datastore), merge operation, source path, and target path. The target path is invalid because it does not contain the `org:orgs` container in the schema tree.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/subtree-manager:copy-one-to-one' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "source-datastore": "CONFIGURATION",
        "source-path": "/network-topology:network-topology/topology=uniconfig/node=vnf01/configuration/org:orgs",
        "target-path": "/network-topology:network-topology/topology=uniconfig/node=vnf02",
        "operation": "merge"
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "unknown-element",
        "error-message": "Failed to find schema node with identifier '(example-services?revision=2010-01-01)orgs' under: NodeList(originalListSchemaNode=list node)"
      }
    ]
  }
}
```