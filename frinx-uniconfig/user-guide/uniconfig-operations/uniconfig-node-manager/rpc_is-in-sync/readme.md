# RPC is-in-sync

This RPC can be used for verification whether the specified nodes
are in-sync with the current state in the Operational datastore of
UniConfig transaction. This verification is done by comparison of
configuration fingerprints. The configuration fingerprint on the
device is compared with the last configuration fingerprint saved
in the Operational datastore. A fingerprint is usually represented
by a configuration timestamp or the last transaction ID. The is-in-sync
feature is supported only for device types that have implemented
translation units for the 'frinx-configuration-metadata' OpenConfig
module (using cli units, netconf units, or uniconfig-native metadata
units).

![RPC is-in-sync](RPC_is-in-sync-RPC_is_in_sync.svg)

The RPC input contains a list of UniConfig nodes for which the
verification should be completed ('target-nodes' field). Response
comprises the operation status for each of the nodes that was
specified in the RPC input. If the operation failed it is because
the specified node has not been successfully installed or connection
has been lost or uniconfig doesn't have support for reading of
configuration fingerprint from specific device type. Calling RPC
with empty list of target nodes will result in invocation of RPC
for each node that has been modified in the UniConfig transaction.

Possible RPC outputs per target node:

1. 'status' field with value 'complete' with set 'is-in-sync' boolean
    flag; is-in-sync feature is supported and the configuration
    fingerprints have been successfully compared.
2. 'status' field with value 'fail' with set 'error-type' to
    'no-connection' and corresponding 'error-message'; Unified
    mountpoint doesn't exist because the connection has been lost or the
    node has not been mounted yet.
3. 'status' field with value 'fail' with set 'error-type' to
    'uniconfig-error' and corresponding 'error-message'; reading of the
    fingerprint from the Operational datastore or Unified mountpoint has
    failed, or the configuration metadata parsing is not supported for
    the device type.

Execution of the 'is-in-sync' RPC doesn't modify the Operational
datastore. The configuration fingerprint that is stored in the
Operational datastore is not updated. 'Sync-from-network' RPC must be
used for updating the last configuration fingerprint and the actual
configuration state.

## RPC Examples

### Successful Example

the RPC input contains valid nodes for which the synchronization status
must be checked ('R1' is synced while 'R2' is not synced):

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:is-in-sync' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["R1","R2"]
        }
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "node-results": {
      "node-result": [
        {
          "node-id": "R1",
          "is-in-sync": true
        },
        {
          "node-id": "R2",
          "is-in-sync": false
        }
      ]
    }
  }
}
```
### Successful Example

If the RPC input does not contain the target nodes, all touched nodes
will be invoked.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:is-in-sync' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

```json RPC Response, Status: 200
{
  "output": {
    "node-results": {
      "node-result": [
        {
          "node-id": "R1",
          "is-in-sync": true
        },
        {
          "node-id": "R2",
          "is-in-sync": false
        }
      ]
    }
  }
}
```

### Failed Example

RPC input contains invalid node, the 'R1' doesn't support comparison of fingerprints
(metadata translation unit has not been implemented for this device).

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:is-in-sync' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["R1"]
        }
    }
}'
```

```json RPC Response, Status: 501
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "operation-not-supported",
        "error-message": "Unable to check configuration fingerprint - parsing of configuration fingerprint is not implemented for this device type.",
        "error-info": {
          "node-id": "R1"
        }
      }
    ]
  }
}
```

### Failed Example

RPC input contains 2 nodes, the first one 'R1' is valid and synced,
the second one ('R2') has not been installed yet. If there is one
invalid node, Uniconfig operation will fail with 1 error entry in the
response.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:is-in-sync' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["R1","R2"]
        }
    }
}'
```

```json RPC Response, Status: 502
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "southbound-no-connection",
        "error-message": "Node 'R2' hasn't been installed in Uniconfig database",
        "error-info": {
          "node-id": "R2"
        }
      }
    ]
  }
}
```

### Failed Example

If the RPC input does not contain the target nodes and there
are not any touched nodes, the request will result in an error.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:is-in-sync' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "missing-element",
        "error-message": "There aren't any nodes specified in input RPC and there aren't any touched nodes."
      }
    ]
  }
}
```
