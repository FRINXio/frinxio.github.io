# RPC install-multiple-nodes

This RPC installs multiple devices at once. It uses the default
install-node RPC. Devices are installed in parallel.

## RPC Examples

### Successful example

RPC input contains two devices (R1 and R2).

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "cli": {
                    "cli-topology:host": "192.168.1.214",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "5.3.4",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:journal-size": 150,
                    "cli-topology:dry-run-journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
            },
            {
                "node-id": "R2",
                "cli": {
                    "cli-topology:host": "192.168.1.211",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "6.1.2",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:dry-run-journal-size": 180,
                    "cli-topology:journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
            }
        ]
    }
}'
```

```json RPC Response, Status: 200
```

### Successful example

RPC input contains devices (R1 and R2) and R2 uses two different protocols.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R2",
                "cli": {
                    "cli-topology:host": "192.168.1.211",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "6.1.2",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:dry-run-journal-size": 180,
                    "cli-topology:journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": false
                },
                "netconf": {
                    "netconf-node-topology:host": "192.168.1.211",
                    "netconf-node-topology:port": 830,
                    "netconf-node-topology:tcp-only": false,
                    "netconf-node-topology:username": "cisco",
                    "netconf-node-topology:password": "cisco",
                    "netconf-node-topology:session-timers": {
                        "netconf-node-topology:keepalive-delay": 0
                    },
                    "netconf-node-topology:other-parameters": {
                        "netconf-node-topology:dry-run-journal-size": 180
                    }
                }
            },
            {
                "node-id": "R1",
                "cli": {
                    "cli-topology:host": "192.168.1.214",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "5.3.4",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:journal-size": 150,
                    "cli-topology:dry-run-journal-size": 150
                }
            }
        ]
    }
}'
```

```json RPC Response, Status: 200
```

### Successful example

RPC input contains two devices (R1 and R2) and R2 is already installed using CLI protocol.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "cli": {
                    "cli-topology:host": "192.168.1.214",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "5.3.4",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:journal-size": 150,
                    "cli-topology:dry-run-journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
            },
            {
                "node-id": "R2",
                "cli": {
                    "cli-topology:host": "192.168.1.211",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "6.1.2",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:dry-run-journal-size": 180,
                    "cli-topology:journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
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
                "error-tag": "data-exists",
                "error-app-tag": "UniconfigError",
                "error-info": {
                    "topology-id": "netconf",
                    "node-id": "R2"
                },
                "error-message": "Node has already been installed using NETCONF protocol",
                "error-type": "application"
            }
        ]
    }
}
```

### Failed Example

RPC input doesn't specify node-id.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "cli": {
                    "cli-topology:host": "192.168.1.214",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "5.3.4",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:journal-size": 150,
                    "cli-topology:dry-run-journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
            },
            {
                "node-id": "R2",
                "cli": {
                    "cli-topology:host": "192.168.1.211",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "6.1.2",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:dry-run-journal-size": 180,
                    "cli-topology:journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
            }
        ]
    }
}'
```

```json RPC Response, Status: 400
{
    "errors": {
        "error": [
            {
                "error-tag": "missing-element",
                "error-app-tag": "UniconfigError",
                "error-message": "Field 'node-id' must be specified in the RPC input",
                "error-type": "application"
            }
        ]
    }
}
```

### Failed Example

RPC input contains two devices using same node-id.

```bash RPC Request
curl --location --request POST 'http://localhost:8181/rests/operations/connection-manager:install-multiple-nodes' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data-raw '{
    "input": {
        "nodes": [
            {
                "node-id": "R1",
                "cli": {
                    "cli-topology:host": "192.168.1.214",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "5.3.4",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:journal-size": 150,
                    "cli-topology:dry-run-journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                },
                "cli": {
                    "cli-topology:host": "192.168.1.211",
                    "cli-topology:port": "22",
                    "cli-topology:transport-type": "ssh",
                    "cli-topology:device-type": "ios xr",
                    "cli-topology:device-version": "6.1.2",
                    "cli-topology:username": "cisco",
                    "cli-topology:password": "cisco",
                    "cli-topology:dry-run-journal-size": 180,
                    "cli-topology:journal-size": 150,
                    "uniconfig-config:install-uniconfig-node-enabled": true
                }
            }
        ]
    }
}'
```

```json RPC Response, Status: 400
{
    "errors": {
        "error": [
            {
                "error-tag": "missing-element",
                "error-app-tag": "UniconfigError",
                "error-message": "Failed to install more than one node with same ID to Uniconfig layer.",
                "error-type": "application"
            }
        ]
    }
}
```