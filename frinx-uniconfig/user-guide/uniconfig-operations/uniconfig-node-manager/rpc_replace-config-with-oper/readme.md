# RPC replace-config-with-operational

This RPC replaces UniConfig topology nodes in the **Config** datastore with nodes from the **Operational** datastore.

RPC input contains a list of UniConfig nodes to replace. RPC output describes the result of the operation and matches all input nodes.

If the RPC is invoked with an empty list of target nodes, the operation is invoked for all nodes modified in the UniConfig transaction. If any node fails, the entire RPC also fails.

![RPC replace-config-with-operational](RPC_replace-config-with-operational-RPC_replace_config_with_operational.svg)

## RPC examples

### Successful example

RPC input has two target nodes. RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:replace-config-with-operational' \
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

```RPC Response, Status: 200
```

### Successful Example

If the RPC input does not contain the target nodes, configuration of all touched nodes will be replaced by operational state.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:replace-config-with-operational' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

```RPC Response, Status: 200
```

### Failed example

RPC input contains a list of target nodes. Node R1 has not been installed. RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:replace-config-with-operational' \
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
        "error-message": "Node is not installed",
        "error-info": {
          "node-id": "R1"
        }
      }
    ]
  }
}
```

### Failed example

If RPC input does not contain any target nodes and there are no any touched nodes, the request results in an error.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:replace-config-with-operational' \
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
