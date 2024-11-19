# RPC connect-node

This RPC establishes connection to all notification streams belonging to an installed nodes.
The connection is created outside a transaction,
so that the connection stays up even without active transactions.

!!!
Note that the transaction used in this RPC is created internally, so no
user-created transactions are used.
!!!

The **connect-node RPC** also supports connection to a specific stream node by specifying
the stream node name in the RPC. Examples are shown below.

!!!
The **connect-node RPC** only works on local Uniconfig nodes in a cluster.
!!!

## RPC parameters

- `node-id` (mandatory) - ID of a node.
- `stream-name` (optional) - Name of a stream.
- `max-connection-attempts` (optional) - Maximum number of connection attempts. The default
  value is 1.
- `between-attempts-timeout` (optional) - Timeout between connection attempts in seconds.
  The default value is 60 seconds.

## UniConfig Shell

The **connect-node RPC** is also included in
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

### Successful example - Request to establish connection to all notification streams.

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

### Failed example - All notification streams are already connected

```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "All connections for node R1 are already established."
            }
        ]
    }
}
```

### Failed example - No available subscriptions to notification streams

```json RPC Response, Status: 404
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "There are no available connections for node: R1"
            }
        ]
    }
}
```

### Successful example - Request for a specific stream node

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:connect-node' \
--header 'Authorization: Basic YWRtaW46YWRtaW4=' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Cookie: Cookie_1=value' \
--data-raw '{
    "input": {
        "node-id": "R1",
        "stream-name": "NETCONF",
        "max-connection-attempts": 1,
        "between-attempts-timeout": 1
    }
}'
```

```RPC Response, Status: 200
```

!!!
Note that waiting for a stream node to be connected is currently not supported.
This means that even if the response code is 200, the connection is not
necessarily successful as well. To test if the connection is successful or in
the process of connecting, call the RPC again (see example below) or examine the
UniConfig logs.
!!!

### Failed example - Node is already connected

```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "Notification stream NETCONF for node R1 is already connected."
            }
        ]
    }
}
```

### Failed example - Node is in the process of connecting

```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "Some connections are in the process of connecting. Try again later."
            }
        ]
    }
}
```
```json RPC Response, Status: 409
{
    "errors": {
        "error": [
            {
                "error-tag": "in-use",
                "error-type": "application",
                "error-message": "Some connections are in the process of connecting or failed to connect. Try again later."
            }
        ]
    }
}
```