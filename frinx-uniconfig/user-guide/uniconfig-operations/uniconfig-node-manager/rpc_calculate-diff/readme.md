# RPC calculate-diff

This RPC creates a diff between the actual UniConfig topology nodes and
the intended UniConfig topology nodes. The RPC input contains a list of
UniConfig nodes to calculate the diff. Output of the RPC contains a list
of statements representing the diff. It also matches all input nodes.
If RPC is called with empty list of target nodes, diff is calculated for
each modified node in the UniConfig transaction. If some node fails for
any reason, the RPC fails entirely

![RPC calculate-diff](RPC_calculate-diff-RPC_calculate_diff.svg)

## RPC Examples

### Successful Example

The RPC calculate-diff input has two target nodes and the output contains
a list of statements representing the diff.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
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
          "topology": "uniconfig",
          "updated-data": [
            {
              "path": "network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=GigabitEthernet0%2F0%2F0%2F0/config",
              "data-actual": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": true,\n    \"name\": \"GigabitEthernet0/0/0/0\"\n  }\n}",
              "data-intended": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": false,\n    \"name\": \"GigabitEthernet0/0/0/0dfhdfghd\"\n  }\n}"
            }
          ]
        },
        {
          "node-id": "R2",
          "topology": "unistore",
          "updated-data": [
            {
              "path": "network-topology:network-topology/topology=unistore/node=R2/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=GigabitEthernet0%2F0%2F0%2F0/config",
              "data-actual": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": false,\n    \"name\": \"GigabitEthernet0/0/0/0\"\n  }\n}",
              "data-intended": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": false,\n    \"name\": \"GigabitEthernet0/0/0/0dfhdfghd\"\n  }\n}"
            }
          ]
        }
      ]
    }
  }
}
```

### Successful Example

If the RPC calculate-diff input does not contain the target nodes,
calculate-diff will be invoked on top of all touched nodes in the
transaction.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

or

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
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
          "topology": "uniconfig",
          "deleted-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=R1/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=GigabitEthernet0%2F0%2F0%2F0/config",
              "data": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": false,\n    \"name\": \"GigabitEthernet0/0/0/0\"\n  }\n}"
            }
          ]
        },
        {
          "node-id": "R2",
          "topology": "uniconfig",
          "deleted-data": [
            {
              "path": "/network-topology:network-topology/topology=uniconfig/node=R2/frinx-uniconfig-topology:configuration/frinx-openconfig-interfaces:interfaces/interface=GigabitEthernet0%2F0%2F0%2F0/config",
              "data": "{\n  \"frinx-openconfig-interfaces:config\": {\n    \"type\": \"iana-if-type:ethernetCsmacd\",\n    \"enabled\": false,\n    \"name\": \"GigabitEthernet0/0/0/0\"\n  }\n}"
            }
          ]
        },
        {
          "node-id": "R3",
          "topology": "unistore",
          "reordered-lists": [
            {
              "path": "/network-topology:network-topology/topology=unistore/node=R3/frinx-uniconfig-topology:configuration/routing-policy:routing-policy/policy-definitions/policy-definition=route_policy_1/statements/statement",
              "actual-list-keys": "[\"name=1\", \"name=3\", \"name=2\"]",
              "intended-list-keys": "[\"name=1\", \"name=2\", \"name=3\"]"
            }
          ]
        }
      ]
    }
  }
}
```

### Successful Example

The RPC calculate-diff input has target node and there is no diff.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
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

The RPC calculate-diff input has target node. Nodes 'R2' has not been
installed yet. The output describes the result of the calculate-diff RPC.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
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

```json RPC Response, Status: 404
{
  "errors": {
    "error": [
      {
        "error-type": "application",
        "error-tag": "data-missing",
        "error-message": "Node 'R1' hasn't been installed in Uniconfig database",
        "error-info": {
          "node-id": "R1",
          "topology-id": "uniconfig"
        }
      }
    ]
  }
}
```

### Failed Example

The RPC calculate-diff input has two target nodes. One of the nodes,
'R2', has not been installed yet. The output describes the result
of the calculate-diff RPC.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
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
        "error-message": "Node is not installed.",
        "error-info": {
          "node-id": "R2",
          "topology-id": "uniconfig"
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
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "target-nodes": {
        }
    }
}'
```

or

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:calculate-diff' \
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
        "error-message": "There aren't any nodes specified in input RPC and there aren't any touched nodes."
      }
    ]
  }
}
```
