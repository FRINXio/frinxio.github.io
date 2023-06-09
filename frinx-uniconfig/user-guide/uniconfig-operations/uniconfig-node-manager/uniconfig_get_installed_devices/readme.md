# RPC get-installed-nodes

This RPC returns all installed devices from a selected topology.
If no topology is specified, the output may contain devices from
multiple topologies (CLI, NETCONF, gNMI). 
In this case devices must be installed with install request parameter
"uniconfig-config:install-uniconfig-node-enabled" set to "true".
Rpc with no topology is looking for nodes installed under UNICONFIG topology
by default.

## RPC Examples

### Successful example

The RPC contains **no topology** defined in input and device called 'R1'
is installed in the NETCONF topology. With parameter
"uniconfig-config:install-uniconfig-node-enabled":"true" in install
request is **installed** under UNICONFIG topology.


```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:get-installed-nodes' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {}
}'
```

```json RPC Response, Status: 200
{
    "output": {
        "node-results": {
            "node-result": [
                {
                    "topology-id": "netconf",
                    "node-id": "R1"
                }
            ]
        }
    }
}
```

### Successful example

The RPC input contains **no topology** and device called 'R1' is installed in the NETCONF topology.
With parameter "uniconfig-config:install-uniconfig-node-enabled":"false" in install
request is **not installed** under UNICONFIG topology.


```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:get-installed-nodes' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {}
}'
```

```json RPC Response, Status: 200
{
    "output": {
        "node-results": {}
    }
}
```

### Successful example

The RPC input contains the GNMI topology and device called 'R1' is installed in the topology.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:get-installed-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "mount-type": "gnmi"
    }
}'
```

```json RPC Response, Status: 200
{
    "output": {
        "node-results": {
            "node-result": [
                {
                    "topology-id": "gnmi",
                    "node-id": "R1"
                } 
            ]
        }
    }
}
```

### Successful example

The RPC input contains the CLI topology, but no devices are installed in the topology.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:get-installed-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "mount-type": "cli"
    }
}'
```

```json RPC Response, Status: 200
{
    "output": {
        "node-results": {}
    }
}
```
