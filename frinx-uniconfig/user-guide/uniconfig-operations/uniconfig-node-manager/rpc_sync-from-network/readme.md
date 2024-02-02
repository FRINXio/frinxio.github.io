# RPC sync-from-network

This RPC synchronizes configurations from network devices to UniConfig nodes in
the `Operational` datastore of the UniConfig transaction.

RPC input contains a list of UniConfig nodes whose configuration should be
refreshed within the network. RPC output describes the result and matches all
input nodes.

Calling the RPC with an empty list of target nodes syncs the configuration of
all nodes modified in the UniConfig transaction. If any node fails, the entire
RPC also fails.

If the network device was installed as southbound-only (with the
`uniconfig-config:install-uniconfig-node-enabled` parameter set to `false`), the
RPC syncs the device to the UniConfig topology and rewrites the above-mentioned
parameter to `true`.

![RPC sync-from-network](RPC_sync-from-network-RPC_sync_from_network.svg)

## RPC examples

### Successful example

RPC input contains nodes whose configuration should be refreshed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-from-network' \
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

If RPC input contains no target nodes, all nodes touched in the transaction are
synced.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-from-network' \
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

RPC input contains a list of nodes whose configuration should be refreshed. Node
R2 is not installed.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-from-network' \
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

If RPC input does not contain any target nodes and there are no touched nodes,
the request results in an error.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:sync-from-network' \
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
