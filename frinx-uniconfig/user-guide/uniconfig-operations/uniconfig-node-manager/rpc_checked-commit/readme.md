# RPC checked-commit

The trigger for execution of the checked configuration is RPC
checked-commit. A checked commit is similar to an RPC commit, but it also
checks if nodes are in sync with the network before it starts
configuration. RPC fails if any node is out of sync. Output of the RPC
describes the result of the commit and matches all modified nodes in the
UniConfig transaction. If one node failed for any reason, RPC will fail
entirely.

In comparison to commit RPC, there is one additional phase between 'lock
and validate configured nodes' and 'write configuration into device'
phases:

1. Lock and validate configured nodes
2. Check if nodes are in-sync with state on devices
3. Write configuration into device
4. Validate configuration
5. Confirmed commit
6. Confirming commit (submit configuration)

Following diagram captures check if configuration fingerprints in the
transaction datastore and device are equal.

![RPC checked-commit](RPC_checked-commit-RPC_checked_commit.svg)

!!!
There is a difference between fingerprint-based validation in the
phases 1 and 2. The goal of the first phase is validation if other
transaction has already changed the same node by comparison of
fingerprint in the UniConfig transaction and in the database. On the
other side, the second phase validates if fingerprint in the
transaction equals to fingerprint on the device - if another system /
directly user via CLI has updated device configuration since the
beginning of the transaction.
!!!

## RPC Examples

### Successful Example

Configuration of nodes 'R1' and 'R2' has been changed in the transaction.
Both 'R1' and 'R2' are in-sync with actual state on the device.
RPC checked-commit input invoke all touched nodes.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:checked-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```RPC Response, Status: 200
```

### Failed Example

Configuration of nodes 'R1' and 'R2' has been changed in the transaction.
Both 'R1' and 'R2' are in-sync with actual state on the device. Node 'R1'
has failed due to improper configuration. The output describes the result
of the checked-commit RPC.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:checked-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 500
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "operation-failed",
        "error-message": "Supplied value \"GigabitEthernet0/0/0/1ghjfhjfhjfghj\" does not match required pattern \"^(([a-zA-Z0-9_]*\\d+/){3,4}\\d+)|(([a-zA-Z0-9_]*\\d+/){3,4}\\d+\\.\\d+)|(([a-zA-Z0-9_]*\\d+/){2}([a-zA-Z0-9_]*\\d+))|(([a-zA-Z0-9_]*\\d+/){2}([a-zA-Z0-9_]+))|([a-zA-Z0-9_-]*\\d+)|([a-zA-Z0-9_-]*\\d+\\.\\d+)|(mpls)|(dwdm)$\"\n",
        "error-info": {
          "node-id": "R1",
          "configuration-status": "fail",
          "rollback-status": "complete"
        }
      }
    ]
  }
}
```

### Failed Example

Configuration of nodes 'R1' has been changed in the transaction.
Node 'R1' is in-sync with actual state on the device. Node 'R1'
has failed on the changed fingerprint. The output describes
the result of the checked-commit.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:checked-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 500
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "validation-failed",
        "error-message": "Fingerprint from datastore: 2019-09-04T09:32:41 is different than fingerprint from device: 2019-09-04T09:49:01",
        "error-info": {
          "node-id": "R1",
          "configuration-status": "fail"
        }
      }
    ]
  }
}
```

### Failed Example

Node 'R2' has lost connection.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:checked-commit' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
    }
}'
```

```json RPC Response, Status: 502
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "southbound_no_connection",
        "error-message": "Unified Mountpoint not found.",
        "error-info": {
          "node-id": "R2",
          "configuration-status": "fail"
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
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:checked-commit' \
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
        "error-type": "application",
        "error-tag": "missing-element",
        "error-message": "There aren't any touched nodes."
      }
    ]
  }
}
```
