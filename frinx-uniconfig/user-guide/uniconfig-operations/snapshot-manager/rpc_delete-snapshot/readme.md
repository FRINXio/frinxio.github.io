# RPC delete-snapshot

This RPC removes a snapshot from the `Configuration` datastore of the UniConfig
transaction.

RPC input contains the name of the snapshot topology to be removed. RPC output
contains the result of the operation.

![RPC delete-snapshot](RPC_delete-snapshot-RPC_delete_snapshot.svg)

## RPC examples

### Successful example

RPC input contains the name of the snapshot topology to be removed. RPC output
contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:delete-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot1"
    }
}'
```

```RPC Response, Status: 204
```

### Failed example

RPC input contains the name of the snapshot topology to be removed. The snapshot
name specified does not exist. RPC output contains the result of the operation.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/snapshot-manager:delete-snapshot' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "name" : "snapshot1"
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "data-missing",
        "error-type": "application",
        "error-message": "Snapshot with name snapshot1 does not exist. Cannot delete snapshot."
      }
    ]
  }
}
```
