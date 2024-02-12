# RPC sync-to-network

This RPC is a combination of the **sync-from-network** and **commit** RPCs. If
one of those RPCs fails, this RPC also fails without making any changes.

The purpose of this RPC is to synchronize configurations from UniConfig nodes in
the `Configuration` datastore of the UniConfig transaction to network devices.

RPC input contains a list of UniConfig nodes to be updated on a network device.
RPC output describes the results and matches all input nodes.

Calling this RPC with an empty list of target nodes syncs the configuration of
all nodes modified in the UniConfig transaction. If any node fails, the entire
RPC also fails. The `admin-state` of UniConfig nodes specified in the input must
be set to `unlocked`.

## RPC examples

### Successful example

RPC input contains a list of nodes to be updated on the corresponding network
device.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-to-network' \
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

```RPC Response, Status: 204
```

### Successful example

If RPC input contains no target nodes, the operation is invoked for all nodes
touched in the transaction.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-to-network' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

```RPC Response, Status: 204
```

### Failed example

If the `admin-state` for some input nodes is not set to `unlocked`, the request
results in an error that includes a list of nodes with the wrong state.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-to-network' \
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

```json RPC Response, Status: 409
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "lock-denied",
        "error-message": "Node is currently in admin-state 'SouthboundLocked'.",
        "error-info": {
          "node-id": "R1"
        }
      },
      {
        "error-type": "application",
        "error-tag": "lock-denied",
        "error-message": "Node is currently in admin-state 'Locked'.",
        "error-info": {
          "node-id": "R2"
        }
      }
    ]
  }
}
```

### Failed example

RPC input contains one node with an incorrect `admin-state`.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-to-network' \
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

```json RPC Response, Status: 409
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "lock-denied",
        "error-message": "Node is currently in admin-state 'SouthboundLocked'.",
        "error-info": {
          "node-id": "R2"
        }
      }
    ]
  }
}
```

### Failed example

RPC input contains two nodes. Node R1 is valid and R2 has not been installed. If
at least one node is invalid, the entire operation fails.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-to-network' \
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

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "data-missing",
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

If RPC input contains no target nodes and there are no touched nodes, the
request results in an error.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-to-network' \
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
