# RPC replace-config-with-snapshot

This RPC replaces nodes in the UniConfig topology of the `Configuration` datastore with
selected nodes from a specified snapshot.

RPC input contains the name of the snapshot topology and target nodes that
should replace UniConfig nodes in the `Configuration` datastore. RPC output describes
the result of the operation and matches all input nodes.

This RPC cannot be called with an empty `target-nodes` list. If a node fails for
any reason, the entire RPC fails.

![RPC replace-config-with-snapshot](RPC_replace-config-with-snapshot-RPC_replace_config_with_snapshot.svg)

## Examples

### Successful example

RPC input contains the name of the snapshot topology that should replace nodes
in the UniConfig topology of the `Configuration` datastore, as well as a list of nodes
from that snapshot. RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:replace-config-with-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot1",
        "target-nodes": {
            "node": ["IOSXR","IOSXRN"]
        }
    }
}'
```

```RPC Response, Status: 204
```

### Failed example

RPC input contains the name of the snapshot topology that should replace nodes
in the UniConfig topology of the `Configuration` datastore, as well as a list of nodes
from that snapshot. This particular snapshot (`snapshot2`) has not been created
yet.

RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:replace-config-with-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot2",
        "target-nodes": {
            "node": ["IOSXR"]
        }
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "missing-element",
        "error-info": {
          "node-id": "IOSXR"
        },
        "error-type": "application",
        "error-message": "Snapshot with name 'snapshot2' does not exist."
      }
    ]
  }
}
```

### Failed example

RPC input contains the name of the snapshot topology that should replace nodes
in the UniConfig topology of the `Configuration` datastore, as well as a list of nodes
from that snapshot. RPC input is missing the snapshot name. RPC output contains
the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:replace-config-with-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["IOSXR"]
        }
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-tag": "missing-element",
        "error-type": "application",
        "error-message": "Snapshot name cannot be empty. "
      }
    ]
  }
}
```

### Failed example

RPC input contains the name of the snapshot topology that should replace nodes
in the UniConfig topology of the `Configuration` datastore, as well as a list of nodes
from that snapshot. One node is missing in snapshot1 (`IOSXRN`). RPC output
contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:replace-config-with-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot1",
        "target-nodes": {
            "node": ["IOSXR","IOSXRN"]
        }
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "missing-element",
        "error-info": {
          "node-id": "IOSXRN"
        },
        "error-type": "application",
        "error-message": "UniConfig node does not exist in snapshot 'snapshot1'."
      }
    ]
  }
}
```

### Failed example

RPC input does not contain target nodes, so the RPC can not be executed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:replace-config-with-snapshot' \
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
        "error-tag": "missing-element",
        "error-type": "application",
        "error-message": "Nodes are not specified in input request"
      }
    ]
  }
}
```
