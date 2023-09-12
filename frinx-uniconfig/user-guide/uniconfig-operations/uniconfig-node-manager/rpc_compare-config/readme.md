# RPC compare-config

This RPC is a combination of the sync-from-network and calculate-diff RPCs. 
If one of those RPCs fails, this one also fails with no changes made.

The purpose of this RPC is to synchronize configurations from network devices
to UniConfig nodes in the Configuration datastore of the UniConfig transaction.

The RPC input contains a list of UniConfig nodes which configuration should be
compared to actual configuration in the transaction The output of the RPC
describes the result of compare-config and matches all input nodes with
a list of statements representing the diff.

## RPC Examples

### Successful Example

```bash RPC Request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/uniconfig-manager:compare-config' \
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

```json RPC Response, Status: 200
{
  "output": {
    "node-results": {
      "node-result": [
        {
          "node-id": "R1",
          "topology-id": "uniconfig",
          "created-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/events:event/internal/assoc=test",
              "data": "{\n  \"assoc\": [\n    {\n      \"name\": \"test\",\n      \"id\": \"vmod-org-create\",\n      \"scripts\": [\n        \"script0.sh\"\n      ],\n      \"type\": \"ipc\"\n    }\n  ]\n}"
            }
          ],
          "updated-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/oam:alarms/alarm=all",
              "data-actual": "{\n  \"oam:destinations\": [\n    \"netconf\",\n    \"analytics\"\n  ]\n}",
              "data-intended": "{\n  \"oam:destinations\": [\n    \"netconf\"\n  ]\n}"
            }
          ]
        },
        {
          "node-id": "R2",
          "topology-id": "unistore",
          "created-data": [
            {
              "path": "/network-topology:network-topology/topology=unistore/node=R2/frinx-uniconfig-topology:configuration/oam:alarms",
              "data": "{\n  \"oam:alarms\": {\n    \"alarm\": [\n      {\n        \"name\": \"all\",\n        \"destinations\": [\n          \"netconf\"\n        ]\n      }\n    ]\n  }\n}"
            },
            {
              "path": "/network-topology:network-topology/topology=unistore/node=R2/frinx-uniconfig-topology:configuration/events:event/internal/assoc=test",
              "data": "{\n  \"assoc\": [\n    {\n      \"name\": \"test\",\n      \"scripts\": [\n        \"script0.sh\"\n      ],\n      \"id\": \"vmod-org-create\",\n      \"type\": \"ipc\"\n    }\n  ]\n}"
            }
          ]
        }
      ]
    }
  }
}
```

### Successful Example

If the RPC input does not contain the target nodes, configuration of all
touched nodes in the transaction is compared to synced device configuration.

```bash RPC Request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/uniconfig-manager:compare-config' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
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
          "node-id": "R1",
          "topology-id": "uniconfig",
          "created-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/events:event/internal/assoc=test",
              "data": "{\n  \"assoc\": [\n    {\n      \"name\": \"test\",\n      \"id\": \"vmod-org-create\",\n      \"scripts\": [\n        \"script0.sh\"\n      ],\n      \"type\": \"ipc\"\n    }\n  ]\n}"
            }
          ],
          "updated-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/oam:alarms/alarm=all",
              "data-actual": "{\n  \"oam:destinations\": [\n    \"netconf\",\n    \"analytics\"\n  ]\n}",
              "data-intended": "{\n  \"oam:destinations\": [\n    \"netconf\"\n  ]\n}"
            }
          ]
        },
        {
          "node-id": "R2",
          "topology-id": "unistore",
          "created-data": [
            {
              "path": "/network-topology:network-topology/topology=unistore/node=R2/frinx-uniconfig-topology:configuration/oam:alarms",
              "data": "{\n  \"oam:alarms\": {\n    \"alarm\": [\n      {\n        \"name\": \"all\",\n        \"destinations\": [\n          \"netconf\"\n        ]\n      }\n    ]\n  }\n}"
            },
            {
              "path": "/network-topology:network-topology/topology=unistore/node=R2/frinx-uniconfig-topology:configuration/events:event/internal/assoc=test",
              "data": "{\n  \"assoc\": [\n    {\n      \"name\": \"test\",\n      \"scripts\": [\n        \"script0.sh\"\n      ],\n      \"id\": \"vmod-org-create\",\n      \"type\": \"ipc\"\n    }\n  ]\n}"
            }
          ]
        }
      ]
    }
  }
}
```

### Successful Example

The RPC compare-config input has target node and there is no diff.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:compare-config' \
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
    "node-results": {
      "node-result": [
        {
          "node-id": "R1",
          "topology-id": "uniconfig"
        }
      ]
    }
  }
}
```

### Failed Example

The RPC compare-config input has two target nodes. One of the nodes,
'R2', has not been installed yet. The output describes the result
of the sync-from-network.

```bash RPC Request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/uniconfig-manager:compare-config' \
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

If the RPC input does not contain the target nodes and there
are not any touched nodes, the request will result in an error.

### Failed Example

```bash RPC Request
curl --location --request POST 'http://127.0.0.1:8181/rests/operations/uniconfig-manager:compare-config' \
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
