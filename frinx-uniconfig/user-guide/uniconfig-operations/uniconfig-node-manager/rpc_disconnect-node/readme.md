# RPC disconnect-node

This RPC functions similar to unmount-node RPC in a sense that it closes a connection to a device.

!!!
Note that the transaction used in this RPC is created internally so a user created transaction won't be used.
!!!

Disconnect-node RPC also supports disconnecting stream nodes.

!!!
Disconnect-node RPC only works on local Uniconfig node in a cluster.
!!!

## RPC parameters

- **node-id (mandatory)** - Id of a stream node which consists of device node and stream name (\<device node>_\<stream name>) e.g.
"R1_NETCONF".

## CLI Shell

This RPC is also included in the CLI Shell implementation. As this RPC takes a node-id as an input parameter, CLI Shell
suggests to the user only nodes that are relevant to this RPC e.g. connected nodes.

## Successful RPC examples

With all examples consider that the install RPC request included the following parameters:

```json
{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "netconf": {
                    "...":  "...",
                    "stream": [
                        {
                            "stream-name": "NETCONF"
                        }
                    ]
                }
            }
        ]
    }
}
```

### Successful request for a device node.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:disconnect-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Cookie: Cookie_1=value' \
--data-raw '{
    "input": {
        "node-id": "R1"
    }
}'
```

```RPC Response, Status: 200
```

### Successful request for a stream node.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:disconnect-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Cookie: Cookie_1=value' \
--data-raw '{
    "input": {
        "node-id": "R1_NETCONF"
    }
}'
```

```RPC Response, Status: 200
```

## Failed RPC responses

### Node isn't connected

Specified node isn't connected or does not exist in database.

```json RPC Response, Status: 502
{
    "errors": {
        "error": [
            {
                "error-tag": "southbound-no-connection",
                "error-type": "application",
                "error-message": "Node vnf20_NETCONF isn't connected"
            }
        ]
    }
}
```

### Node doesn't exist
```json
{
    "errors": {
        "error": [
            {
                "error-tag": "data-missing",
                "error-type": "application",
                "error-message": "Node R1 doesn't exist in database"
            }
        ]
    }
}
```
