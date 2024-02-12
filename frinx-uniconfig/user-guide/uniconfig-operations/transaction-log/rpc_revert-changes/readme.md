# RPC revert-changes

This RPC is used to revert previously configured changes. To revert one or more
transactions, their transaction ids must be included in the body of the RPC. The
transaction id is part of transaction metadata created by the transaction
tracker after the commit/checked-commit RPC.

!!!
This RPC only updates data in the `CONFIGURATION` snapshot. To write reverted
data to the device, use **RPC commit** after **RPC revert-changes**.
!!!


![RPC revert-changes](RPC_revert-changes.png)

## Ignore non-existing nodes

When reverting multiple transactions, some transaction metadata may contain
nodes that do not currently exist in UniConfig. In this case, the RPC fails.

There are two options for handling non-existing nodes:

1.  Remove transactions that contain non-existing nodes from the request
    body.
2.  Add the `ignore-non-existing-nodes` parameter to the RPC request body
    with a value of `true` (the default value is `false`).

!!!
If the `ignore-non-existing-nodes` parameter is not specified, the default value
is used.
!!!

## RPC examples

### Successful examples

To revert a transaction, we need to find its ID. Let's use a `GET` request to
display all stored transaction metadata.

```bash RPC Request
curl --location --request GET 'http://192.168.56.11:8181/rests/data/transaction-log:transactions-metadata' \
--header 'Accept: application/json'
```

```json RPC Response, Status: 200
{
  "transactions-metadata": {
    "transaction-metadata": [
      {
        "transaction-id": "221aa4a5-e32e-46fd-921a-83314b190e89",
        "username": "admin",
        "metadata": [
          {
            "node-id": "xr6",
            "diff": [
              {
                "path": "/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Bundle-Ether1/description",
                "data-after": "{\n  \"Cisco-IOS-XR-ifmgr-cfg:description\": \"bundle-ether1-description-create\"\n}"
              },
              {
                "path": "/Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration=act,Bundle-Ether2/description",
                "data-before": "{\n  \"Cisco-IOS-XR-ifmgr-cfg:description\": \"bundle-ether2-description-before\"\n}",
                "data-after": "{\n  \"Cisco-IOS-XR-ifmgr-cfg:description\": \"bundle-ether2-description-after\"\n}"
              }
            ],
            "topology": "uniconfig"
          }
        ],
        "commit-time": "2021-Mar-09 10:53:59.102 +0100"
      },
      {
        "transaction-id": "869df9d6-9025-4849-b30b-9db4d8fb26ec",
        "username": "admin",
        "metadata": [
          {
            "node-id": "xr5",
            "diff": [
              {
                "path": "/frinx-openconfig-interfaces:interfaces/interface=Loopback123/config",
                "data-before": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:softwareLoopback\",\n    \"enabled\": true,\n    \"name\": \"Loopback123\"\n  }\n}",
                "data-after": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:softwareLoopback\",\n    \"enabled\": true,\n    \"description\": \"test-description\",\n    \"name\": \"Loopback123\"\n  }\n}"
              }
            ],
            "topology": "uniconfig"
          }
        ],
        "commit-time": "2021-Mar-09 11:06:58.000 +0100"
      },
      {
        "transaction-id": "2c4c1eb5-185a-4204-8021-2ea05ba2c2c1",
        "username": "admin",
        "metadata": [
          {
            "node-id": "R1",
            "diff": [
              {
                "path": "/frinx-openconfig-interfaces:interfaces/interface=Bundle-Ether1",
                "data-after": "{\n  \"interface\": [\n    {\n      \"name\": \"Bundle-Ether1\",\n      \"config\": {\n        \"type\": \"iana-if-type:ieee8023adLag\",\n        \"enabled\": false,\n        \"name\": \"Bundle-Ether1\"\n      }\n    }\n  ]\n}"
              }
            ],
            "topology": "uniconfig"
          },
          {
            "node-id": "xr5",
            "diff": [
              {
                "path": "/frinx-openconfig-interfaces:interfaces/interface=Loopback1/config",
                "data-before": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:softwareLoopback\",\n    \"enabled\": true,\n    \"name\": \"Loopback1\"\n  }\n}"
              }
            ],
            "topology": "uniconfig"
          }
        ],
        "commit-time": "2021-Mar-09 11:10:54.104 +0100"
      }
    ]
  }
}
```

Revert changes for a single transaction.

```bash RPC Request
curl --location --request POST 'http://192.168.56.11:8181/rests/operations/transaction-log:revert-changes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-transactions": {
            "transaction": [
                "221aa4a5-e32e-46fd-921a-83314b190e89"
            ]
        }
    }
}'
```

```RPC Response, Status: 204
```

Revert changes for multiple transactions.

```bash RPC Request
curl --location --request POST 'http://192.168.56.11:8181/rests/operations/transaction-log:revert-changes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-transactions": {
            "transaction": [
                "221aa4a5-e32e-46fd-921a-83314b190e89",
                "869df9d6-9025-4849-b30b-9db4d8fb26ec"
            ]
        }
    }
}'
```

```RPC Response, Status: 204
```

Revert changes for multiple transactions. The transaction with id
`2c4c1eb5-185a-4204-8021-2ea05ba2c2c1` contains the non-existing node `R1`.
Since the `ignore-non-existing-nodes` parameter is set to `true`, the RPC is
successful.

```bash RPC Request
curl --location --request POST 'http://192.168.56.11:8181/rests/operations/transaction-log:revert-changes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "ignore-non-existing-nodes": true,
        "target-transactions": {
            "transaction": [
                "221aa4a5-e32e-46fd-921a-83314b190e89",
                "2c4c1eb5-185a-4204-8021-2ea05ba2c2c1"
            ]
        }
    }
}'
```

```json RPC Response, Status: 204
```

### Failed example

The request contains a non-existing transaction in the request body.

```bash RPC Request
curl --location --request POST 'http://192.168.56.11:8181/rests/operations/transaction-log:revert-changes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-transactions": {
            "transaction": [
                "82b4e916-e1ed-4a54-97bc-067699842af6"
            ]
        }
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "data-missing",
        "error-info": {
          "transaction-id": "2c4c1eb5-185a-4204-8021-2ea05ba2c2c1"
        },
        "error-type": "application",
        "error-message": "Failed to find transaction in log with ID: 82b4e916-e1ed-4a54-97bc-067699842af6"
      }
    ]
  }
}
```

Reverting changes for multiple transactions. The transaction metadata with id
`2c4c1eb5-185a-4204-8021-2ea05ba2c2c1` contains a non-existing node. Since the
`ignore-non-existing-nodes` parameter is set to `false`, the RPC fails.

```bash RPC Request
curl --location --request POST 'http://192.168.56.11:8181/rests/operations/transaction-log:revert-changes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "ignore-non-existing-nodes": false,
        "target-transactions": {
            "transaction": [
                "221aa4a5-e32e-46fd-921a-83314b190e89",
                "2c4c1eb5-185a-4204-8021-2ea05ba2c2c1"
            ]
        }
    }
}'
```

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-tag": "data-missing",
        "error-info": {
          "transaction-id": "2c4c1eb5-185a-4204-8021-2ea05ba2c2c1"
        },
        "error-message": "Transaction metadata 2c4c1eb5-185a-4204-8021-2ea05ba2c2c1 contains non-existent uniconfig nodes: [R1]",
        "error-type": "application"
      }
    ]
  }
}
```
