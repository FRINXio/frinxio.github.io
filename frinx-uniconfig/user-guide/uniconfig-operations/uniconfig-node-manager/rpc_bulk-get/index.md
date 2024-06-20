# RPC bulk-get

This RPC provides multiple get operations on multiple nodes, paths and contents.
It is possible to get data from 'Operational' datastore of UniConfig topology if no content is specified, 
otherwise data is retrieved from the device directly (over mount-point).
Operation is executed over actual/existing UniConfig transaction (immediate-commit model / manual user transaction).
Each GET operation is executed in parallel. For mount-point read, it is enough to have node installed 
on southbound topology (if multiple mount-points with the same node-id exist, user need to specify also connection-type) 
or just stored in uniconfig database.

RPC input contains a list of node inputs where each consists of mandatory fields such as `node-id` (UniConfig node identifier) and
`path`, which points to specific subtree. Optional fields are `content` and `connection-type`. Content determines that the call will be 
over mount-point to target device datastore `config, operational, state, all`, where `state, all` are only for gnmi node. 
Connection type is needed only in a scenario where multiple southbound mount-points are defined with the same `node-id`. 

If get operation on any nodes failed (node is not installed, connection lost, data is not present, ...)  the response 
will display only failed nodes in common RFC8040 errors container. In a success case, the result is structured in 
node-results format. Each node-result consists of `node-id`, `content` (if over mount-point) and 
`data` that contains wanted subtree config.

## RPC examples

### Successful example

RPC input contains valid nodes that are installed in UniConfig topology. Bulk-get will read config on following paths 
from UniConfig operational datastore.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:bulk-get' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "path": "frinx-openconfig-interfaces:interfaces/interface"
            },
            {
                "node-id": "R2",
                "path": "Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration"
            }
        ]
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
          "data": "{\"interface\": [{\"name\": \"GigabitEthernet0/0/0/1\", \"config\": { ... } }, ... ]}"
        },
        {
          "node-id": "R2",
          "data": "{\"interface-configuration\": [{\"name\": \"GigabitEthernet0/0/0/1\", \"act\": \"active\" }, ... ]}"
        }
      ]
    }
  }
}
```

### Successful example

RPC input contains valid nodes, from which Ri is installed in UniConfig topology, and R2 is installed as southbound only. 
Bulk-get will read config on following paths from following content target datastores.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:bulk-get' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "path": "frinx-openconfig-interfaces:interfaces/interface",
                "content": "config"
            },
            {
                "node-id": "R2",
                "path": "Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration",
                "content": "operational"
            }
        ]
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
          "data": "{\"interface\": [{\"name\": \"GigabitEthernet0/0/0/1\", \"config\": { ... } }, ... ]}",
          "content": "config"
        },
        {
          "node-id": "R2",
          "data": "{\"interface-configuration\": [{\"name\": \"GigabitEthernet0/0/0/1\", \"act\": \"active\" }, ... ]}",
          "content": "operational"
        }
      ]
    }
  }
}
```

### Failed example

RPC input contains valid node R1 and an invalid node R2, which is not installed in uniconfig topology 
and content is not specified for that node.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:bulk-get' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "path": "frinx-openconfig-interfaces:interfaces/interface"
            },
            {
                "node-id": "R2",
                "path": "Cisco-IOS-XR-ifmgr-cfg:interface-configurations/interface-configuration"
            }
        ]
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
                    "node-id": "R2"
                },
                "error-message": "Node 'R2' is not installed in uniconfig topology",
                "error-type": "application"
            }
        ]
    }
}
```

### Failed example

RPC input contains one node with node-id R1 that has two mount-points (one is mounted over cli and the other over netconf) 
and no connection-type is specified.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:bulk-get' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "path": "frinx-openconfig-interfaces:interfaces/interface",
                "content": "operational"
            }
        ]
    }
}'
```

```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "protocol-conflict",
                "error-info": {
                    "node-id": "R1",
                    "error": "Specify connection-type [cli, netconf, gnmi, snmp]"
                },
                "error-message": "Node 'R1' is installed in multiple southbound topologies 'netconf, cli'",
                "error-type": "application"
            }
        ]
    }
}
```

### Failed example

RPC input contains one node with node-id R3 that is not installed in uniconfig under any southbound topology.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:bulk-get' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R3",
                "path": "frinx-openconfig-interfaces:interfaces/interface",
                "content": "operational"
            }
        ]
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
                    "node-id": "R3"
                },
                "error-message": "Node 'R3' is not installed in any southbound topology",
                "error-type": "application"
            }
        ]
    }
}
```

### Failed example

If RPC input does not contain any node inputs

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/uniconfig-manager:is-in-sync' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": []
    }
}'
```

```json RPC Response, Status: 400
{
    "errors": {
        "error": [
            {
                "error-tag": "missing-element",
                "error-type": "application",
                "error-message": "Input cannot be empty!"
            }
        ]
    }
}
```
