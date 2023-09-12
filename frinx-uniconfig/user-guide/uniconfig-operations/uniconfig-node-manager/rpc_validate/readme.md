# RPC validate

The external application stores the intended configuration under nodes
in the UniConfig topology. The configuration can be validated if it is
valid or not. The trigger for execution of configuration validation is
an RPC validate. RPC input contains a list of UniConfig nodes which
configuration should be validated. Output of the RPC describes the
result of the validation and matches all input nodes. It is valid to
call this RPC with empty list of target nodes - in this case, all nodes
that have been modified in the UniConfig transaction will be validated.

The configuration of nodes consists of the following phases:

1. Open transaction to device
2. Write configuration
3. Validate configuration
4. Close transaction

If one node failed in second (validation) phase for any reason, the RPC
will fail entirely.

!!!
The validation (second phase) take place only on nodes that support
this operation.
!!!

Validate RPC is shown in the figure bellow.

![RPC validate](RPC_validation-RPC_validate.svg)

## RPC Examples

### Successful Example

RPC validate input has 2 target nodes and the output describes
the result of the successful validation.

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

### Successful Example

If the RPC input does not contain the target nodes, all touched node
in the transaction will be validated.

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

### Failed Example

RPC commit input has 1 target node and the output describes the
result of the validation. Node has failed because validation failed.

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

### Failed Example

RPC input contains 2 nodes, the first one 'R1' is valid,
the second one 'R2' has not been installed yet. If there is
one invalid node, Uniconfig will be evaluated nodes with fail.

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

### Failed Example

If the RPC input does not contain the target nodes and there
are not any touched nodes, the request will result in an error.

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
