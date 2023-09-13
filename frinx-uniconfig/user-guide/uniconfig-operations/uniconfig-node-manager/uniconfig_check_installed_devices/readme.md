# RPC check-installed-nodes

This RPC checks if devices included in the input are installed by looking for the database content
of each device. If content is found, the device is installed.

## RPC Examples

### Successful example

RPC input contains a device while no devices are installed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:check-installed-nodes' \
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

```json RPC Response, Status: 200
{
    "output": {
        "node-results": {}
    }
}
```

### Successful example

RPC input contains devices (R1 and R2) and device R1 is installed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:check-installed-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["R1", "R2"]
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
                    "topology-id": "netconf",
                    "node-id": "R1"
                }
            ]
        }
    }
}
```

### Successful example

RPC input contains devices (R1 and R2) and both devices are installed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:check-installed-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["R1", "R2"]
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
                    "topology-id": "netconf",
                    "node-id": "R1"
                },
                {
                    "topology-id": "netconf",
                    "node-id": "R2"
                }
            ]
        }
    }
}
```

### Failed Example

RPC input does not specify any nodes.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:check-installed-nodes' \
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
                "error-app-tag": "UniconfigError",
                "error-message": "Target nodes cannot be empty!",
                "error-type": "application"
            }
        ]
    }
}
```

### Failed Example

RPC input is missing the target-nodes container.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:check-installed-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
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
                "error-message": "Target nodes are not specified!",
                "error-type": "application"
            }
        ]
    }
}
```
