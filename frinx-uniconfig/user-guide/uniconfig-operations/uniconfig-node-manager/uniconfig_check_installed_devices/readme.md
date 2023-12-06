# RPC check-installed-nodes

This RPC checks if devices specified in the input are installed by looking for
the database content of each device. If content is found, the device is
installed.

Output of the RPC contains only a list of devices that are present in the UniConfig database.
List of devices contains:
- topology-id - identifier of the topology in which the device is installed
- node-id - device identifier
- uniconfig-layer - boolean value indicating whether the device is installed in the UniConfig layer

## RPC examples

### Successful example

RPC input contains a device, while no devices are installed.

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

RPC input contains two devices (R1 and R2), one of which (R1) is installed.

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
          "node-id": "R1",
          "uniconfig-layer": true
        }
      ]
    }
  }
}
```

### Successful example

RPC input contains two devices (R1 and R2), both of which are installed.

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
          "node-id": "R1",
          "uniconfig-layer": true
        },
        {
          "topology-id": "netconf",
          "node-id": "R2",
          "uniconfig-layer": false
        }
      ]
    }
  }
}
```

### Failed example

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

### Failed example

RPC input is missing the `target-nodes` container.

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
