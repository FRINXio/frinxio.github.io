# RPC validate

The external application stores the intended configuration under nodes in the UniConfig topology. The configuration can be checked for validity. This RPC is the trigger for configuration validation.

RPC input contains a list of UniConfig nodes whose configuration should be validated. RPC output describes the result of the validation and matches all input nodes.

If the RPC is called with an empty list of target nodes, all nodes modified in the UniConfig transaction are validated.

The configuration of nodes follows these steps:

1. Open transaction to device
2. Write configuration
3. Validate configuration
4. Close transaction

If any node fails in step 3 (validation), the entire RPC also fails.

!!!
Validation (step 3) only applies to nodes that support this operation.
!!!

The diagram below illustrates the RPC:

![RPC validate](RPC_validation-RPC_validate.svg)

## RPC examples

### Successful example

RPC input has two target nodes. RPC output describes the result of successful validation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:validate' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
            "node": ["R1","R2]
        }
    }
}'
```

```RPC Response, Status: 200
```

### Successful example

If RPC input does not specify any target nodes, all nodes touched in the transaction are validated.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:validate' \
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

RPC input has one target node. RPC output describes the result of the validation. In this case, the node has failed because validation has failed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:validate' \
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

```json RPC Response, Status: 500
{
  "errors": {
    "node-result": [
      {
        "error-type": "application",
        "error-tag": "validation-failed",
        "error-message": "RemoteDevice{R1}: Validate failed. illegal reference /orgs/org[name='TESTING-PROVIDER']/traffic-identification/using-networks\n",
        "error-info": {
          "node-id": "R1"
        }
      }
    ]
  }
}
```

### Failed example

RPC input contains two nodes. Node R1 is valid and R2 is not installed. If at least one node is invalid, the entire operation fails.

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

If RPC input does not contain any target nodes and there are no touched nodes, the request results in an error.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:validate' \
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
