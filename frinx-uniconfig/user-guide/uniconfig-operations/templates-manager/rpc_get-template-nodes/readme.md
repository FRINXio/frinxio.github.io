# RPC get-template-nodes

This RPC returns all templates from the template topology. No input body
is required.

## RPC examples

### Successful example

There are no templates in the template topology.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/template-manager:get-template-nodes' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json'
```

```json RPC Response, Status: 200
{
    "output": {}
}
```

### Successful example

There is a template called 'test-template' in the template topology.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/template-manager:get-template-nodes' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json'
```

```json RPC Response, Status: 200
{
    "output": {
        "nodes": [
            "test-template"
        ]
    }
}
```
