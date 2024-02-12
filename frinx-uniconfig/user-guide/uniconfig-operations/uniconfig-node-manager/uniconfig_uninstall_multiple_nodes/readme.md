# RPC uninstall-multiple-nodes

This RPC uninstalls multiple devices at once using the default
**uninstall-node RPC**. Devices are uninstalled independently in parallel. 

If two nodes are being uninstalled and one node fails, the second node is still
uninstalled. If at least one node fails, the response describes the result for
failed nodes only.

## RPC examples

### Successful example

RPC input contains two devices (R1 and R2).

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:uninstall-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R2",
                "connection-type": "cli"
            },
            {
                "node-id": "R1",
                "connection-type": "cli"
            }
        ]
    }
}'
```

```RPC Response, Status: 204
```

### Successful example

RPC input contains two devices (R1 and R2). Device R2 is installed on two
different protocols.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:uninstall-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R2",
                "connection-type": "cli"
            },
            {
                "node-id": "R2",
                "connection-type": "netconf"
            },
            {
                "node-id": "R1",
                "connection-type": "cli"
            }
        ]
    }
}'
```

```RPC Response, Status: 204
```

### Successful example

RPC input contains two devices (R1 and R2). Device R2 is already uninstalled on
the CLI protocol.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:uninstall-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R2",
                "connection-type": "cli"
            }
            {
                "node-id": "R1",
                "connection-type": "cli"
            }
        ]
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "data-missing",
        "error-app-tag": "UniconfigError",
        "error-info": "R2",
        "error-message": "Node has already been removed from Uniconfig",
        "error-type": "application"
      }
    ]
  }
}
```

### Failed example

RPC input does not specify a `node-id`.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:uninstall-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "connection-type": "cli"
            },
            {
                "node-id": "R1",
                "connection-type": "cli"
            }
        ]
    }
}'
```

```json RPC Response, Status: 400
{
  "errors": {
    "error": [
      {
        "error-tag": "missing-element",
        "error-app-tag": "UniconfigError",
        "error-message": "Field 'node-id' must be specified in the RPC input",
        "error-type": "application"
      }
    ]
  }
}
```
