# RPC sync-from-network

The purpose of this RPC is to synchronize configuration from network
devices to the UniConfig nodes in the Operational datastore of UniConfig
transaction. The RPC input contains a list of the UniConfig nodes where
the configuration should be refreshed within the network. Output of the
RPC describes the result of sync-from-network and matches all input
nodes. Calling RPC with empty list of target nodes results in syncing
configuration of all nodes that have been modified in the UniConfig
transaction. If one node failed for any reason, the RPC will fail
entirely.

![RPC sync-from-network](RPC_sync-from-network-RPC_sync_from_network.svg)

## RPC Examples

### Successful Example

RPC input contains nodes where configuration should be refreshed.

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

```RPC Response, Status: 200
```


If RPC input does not contain the target nodes, all touched nodes
in the transaction will be synced.

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

```RPC Response, Status: 200
```

### Failed Example

RPC input contains a list of nodes where the configuration should be
refreshed. Node 'R2' has not been installed yet.

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

### Failed Example

If the RPC input does not contain the target nodes and there
are not any touched nodes, the request will result in an error.

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
