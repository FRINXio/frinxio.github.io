# RPC connect-node

This RPC functions similar to mount-node RPC in a sense that it creates a connection to an already installed device,
but it does so outside a transaction, meaning that the connection will stay up even when there are no active
transactions.

!!!
Note that the transaction used in this RPC is created internally so a user created transaction won't be used.
!!!

Connect-node RPC also supports connecting to stream nodes. This is only possible if a stream node was already
defined in the install-node RPC. It's most relevant use case is when a subscription stream was disconnected and a
reconnection is necessary.

!!!
Connect-node RPC only works on local Uniconfig node in a cluster.
!!!

## RPC parameters

- **node-id (mandatory)** - Id of a stream node which consists of device node and stream name (\<device node>_\<stream name>) e.g.
"R1_NETCONF".
- **max-connection-attempts** - Maximum number of connection attempts. Default value is 1.
- **between-attempts-timeout** - Timeout between connection attempts in seconds. Default value is 60 seconds.

## CLI Shell

This RPC is also included in the CLI Shell implementation. As this RPC takes a node-id as an input parameter, CLI Shell
suggests to the user only nodes that are relevant to this RPC e.g. nodes that are installed in the UC but are not yet
connected.

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
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:connect-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Cookie: Cookie_1=value' \
--data-raw '{
    "input": {
        "node-id": "R1",
        "max-connection-attempts": 1,
        "between-attempts-timeout": 1
    }
}'
```

```RPC Response, Status: 200
```

### Successful request for a stream node.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:connect-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Cookie: Cookie_1=value' \
--data-raw '{
    "input": {
        "node-id": "R1_NETCONF",
        "max-connection-attempts": 1,
        "between-attempts-timeout": 1
    }
}'
```

```RPC Response, Status: 200
```

!!!
Note that we currently do not support waiting for a stream node to be connected. This means that even if the response
code is 200, the connection isn't necessary successful too. To test if the connection is successful or in the process
of connecting, use the RPC again (example below) or look at UC logs.
!!!

## Failed RPC responses

### Node is already connected
```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "Node R1_NETCONF is already connected"
            }
        ]
    }
}
```

### Node is in connection process
```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "Stream node R1_NETCONF is in connection process."
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