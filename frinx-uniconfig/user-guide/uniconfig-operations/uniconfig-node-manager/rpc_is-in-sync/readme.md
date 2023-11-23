# RPC is-in-sync

This RPC is used to verify that specified nodes are in sync with the current state in the **Operational** datastore of the UniConfig transaction.

Verification is performed by comparing configuration fingerprints. The configuration fingerprint on the device is compared with the last configuration fingerprint saved in the **Operational** datastore. A fingerprint is usually represented by a configuration timestamp or the last transaction ID.

The **is-in-sync** feature is supported only for device types that have implemented translation units for the `frinx-configuration-metadata` OpenConfig module (using CLI units, NETCONF units, or UniConfig-native metadata units).

![RPC is-in-sync](RPC_is-in-sync-RPC_is_in_sync.svg)

RPC input contains a list of UniConfig nodes for which verification should be performed (the `target-nodes` field). The response contains the operation status for each node specified in the input. If the operation fails, it is because the specified node has not been successfully installed, the connection is lost or UniConfig does not support reading the configuration fingerprint from that specific device type. Calling the RPC with an empty list of target nodes invokes the RPC for each node modified in the UniConfig transaction.

Possible RPC outputs per target node:

1. `status` field with value `complete` and `is-in-sync` boolean flag set: The **is-in-sync** feature is supported and configuration fingerprints have been successfully compared.
2. `status` field with value `fail`, `error-type` field with value `no-connection` and corresponding `error-message`: The unified mountpoint does not exist because the connection was lost or the node is not mounted.
3. `status` field with value `fail`, `error-type` field with value `uniconfig-error` and corresponding `error-message`: Reading the fingerprint from the **Operational** datastore or unified mountpoint has failed, or parsing configuration metadata is not supported for the device type.

Executing the ***is-in-sync** RPC does not modify the **Operational** datastore. The configuration fingerprint stored in the **Operational** datastore is not updated. Use the **sync-from-network RPC** to update the last configuration fingerprint and the actual configuration state.

## RPC examples

### Successful example

RPC input contains valid nodes for which synchronization status should be checked (R1 is synced, R2 is not synced).

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
### Successful example

If RPC input contains no target nodes, all touched nodes are invoked.

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

### Failed example

RPC input contains an invalid node (R1), which does not support fingerprint comparisons (metadata translation unit has not been implemented for this device).

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

### Failed example

RPC input contains two nodes. Node R1 is valid and synced, R2 is not installed. If one node is invalid, the UniConfig operation will fail with an error entry in the response.

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

### Failed example

If RPC input does not contain any target nodes and there are no touched nodes, the request results in an error.

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
