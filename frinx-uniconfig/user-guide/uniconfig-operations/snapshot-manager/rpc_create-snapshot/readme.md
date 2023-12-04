# RPC create-snapshot

This RPC creates a snapshot of nodes in the UniConfig topology. The snapshot can
be used later for manual rollback.

RPC input contains the name of the snapshot topology and nodes that the snapshot
will contain. RPC output describes the result of the operation and matches all
input nodes.

The RPC cannot be called with an empty list of target nodes. If a node fails for
any reason, the entire RPC fails.

![RPC create-snapshot](RPC_create-snapshot-RPC_create_snapshot.svg)

## RPC examples

### Successful example

RPC input contains the name for the topology snapshot and nodes that the
snapshot contains. RPC output contains the result of operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:create-snapshot' \
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

RPC input includes nodes that will be contained in the snapshot, but a snapshot
name is missing. RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:create-snapshot' \
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

RPC input contains a name for the topology snapshot and a node that will be
contained in the snapshot. The node has not been mounted. RPC output contains
the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:create-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot1",
        "target-nodes": {
            "node": ["AAA"]
        }
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "data-missing",
        "error-info": {
          "node-id": "AAA"
        },
        "error-type": "application",
        "error-message": "Node is missing in uniconfig topology OPERATIONAL datastore."
      }
    ]
  }
}
```

### Failed example

RPC input does not contain target nodes, so the RPC cannot be executed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:create-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot1",
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
