# RPC disconnect-node

This RPC terminates connection to all notification streams belonging to an installed nodes.

!!!
Note that the transaction used in this RPC is created internally, so no
user-created transactions are used.
!!!

The **disconnect-node RPC** also supports terminating connection of a specific stream node by specifying
the stream node name in the RPC. Examples are shown below.

!!!
Note that the **disconnect-node RPC** only works on local Uniconfig nodes in
a cluster.
!!!

## RPC parameters

- `node-id` (mandatory) - ID of a node.
- `stream-name` (optional) - Name of a stream.

## UniConfig Shell

The **disconnect-node RPC** is also included in 
[UniConfig shell](https://docs.frinx.io/frinx-uniconfig/user-guide/uniconfig-operations/uniconfig-shell/). 
As it takes `node-id` as input, the shell only suggests nodes that are relevant
to this RPC (nodes that are installed in UniConfig topology).

## RPC examples

For all examples, assume that the **install-node RPC** request included the following
parameters:

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

### Successful example - Request to disconnect all notification streams

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

### Failed example - Already disconnected

The specified node is not connected or does not exist in the database.

```json RPC Response, Status: 502
{
    "errors": {
        "error": [
            {
                "error-tag": "southbound-no-connection",
                "error-type": "application",
                "error-message": "All connections for node R1 are already disconnected."
            }
        ]
    }
}
```

### Failed example - Can't disconnect because some connections are in the process of connecting

The specified node is not connected or does not exist in the database.

```json RPC Response, Status: 502
{
    "errors": {
        "error": [
            {
                "error-tag": "southbound-no-connection",
                "error-type": "application",
                "error-message": "Some connections are in the process of connecting or aren't connected. Try again later."
            }
        ]
    }
}
```

### Successful example - Request to disconnect a specific notification stream

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:disconnect-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Cookie: Cookie_1=value' \
--data-raw '{
    "input": {
        "node-id": "R1",
        "stream-name": "NETCONF"
    }
}'
```

```RPC Response, Status: 200
```

### Failed example - Node not connected

The specified node is not connected or does not exist in the database.

```json RPC Response, Status: 502
{
    "errors": {
        "error": [
            {
                "error-tag": "southbound-no-connection",
                "error-type": "application",
                "error-message": "Notification stream NETCONF for node R1 is already disconnected."
            }
        ]
    }
}
```
