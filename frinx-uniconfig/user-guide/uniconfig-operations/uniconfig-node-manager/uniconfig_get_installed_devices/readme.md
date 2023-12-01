# RPC get-installed-nodes

This RPC returns all installed devices from a specified topology.

If no topology is specified, the output may contain devices from multiple
topologies (CLI, NETCONF, gNMI). In this case, devices must be installed with
the install request parameter `uniconfig-config:install-uniconfig-node-enabled`
set to `true`. If no topology is specified, the RPC looks for nodes installed in
the UNICONFIG topology by default.

## RPC examples

### Successful example

RPC input does not specify a topology, and device R1 is installed in the NETCONF
topology. If the install request includes the parameter
`“uniconfig-config:install-uniconfig-node-enabled”:“true”`, the device is
installed in the UNICONFIG topology instead.


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

RPC input does not specify a topology, and device R1 is installed in the NETCONF
topology. If the install request includes the parameter
`“uniconfig-config:install-uniconfig-node-enabled”:“false”`, the device is not
installed in the UNICONFIG topology.


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

RPC input specifies the GNMI topology, and device R1 is installed in that
topology.

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

RPC input specifies the CLI topology, but no devices are installed in that
topology.

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
